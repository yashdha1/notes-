# Workflows Guide

**Source**: feed_algorithm.md, notification_plan.md, design.md
**Services involved**: feed-service, content-service, notification-service

---

## Feed Build Workflow

**Endpoint**: `GET /api/v1/feed?limit=25&cursor=...`

### High-Level Flow

```
GET /feed
  │
  ├─► Check Redis list feed:{user_id}
  │     Has items? → Pop 25 post_ids → go to BATCH step
  │     Empty?     → REBUILD pool
  │
  ├─► REBUILD (if cache empty):
  │     tag_weight rows exist? → FULL build (200 ids: 60% preference + 40% random)
  │     New user (no weights)?  → COLD start (50 random post_ids)
  │     → LPUSH ids into feed:{user_id} (TTL: 10 min)
  │
  ├─► Pop 25 post_ids from Redis list
  │     → INSERT feed_post_seen rows (prevents re-showing)
  │
  └─► POST /internal/posts/batch (content-service)
        → Returns hydrated post objects
        → Return to client
```

### Step 1: Cache Check

```
Redis key:  feed:{user_id}
Type:       List (LPUSH / RPOP)
TTL:        10 minutes
```

- `LLEN feed:{user_id} > 0` → skip rebuild, go to pop
- Empty or missing → trigger rebuild

### Step 2: Top-K Tags for User

```sql
SELECT tag_id, weight
FROM tag_weight
WHERE user_id = :user_id
ORDER BY weight DESC
LIMIT 10;
```

### Step 3: Full Build (warm user)

```sql
-- 60% preference: posts tagged with user's top tags, not yet seen
SELECT DISTINCT pt.post_id
FROM post_tags pt
JOIN tag_weight tw ON tw.tag_id = pt.tag_id AND tw.user_id = :user_id
LEFT JOIN feed_post_seen fps ON fps.post_id = pt.post_id AND fps.user_id = :user_id
WHERE fps.post_id IS NULL
ORDER BY tw.weight DESC
LIMIT 120;

-- 40% random: any unseen post
SELECT id AS post_id
FROM post_universe  -- post_tags doubles as universe
WHERE id NOT IN (SELECT post_id FROM feed_post_seen WHERE user_id = :user_id)
ORDER BY RANDOM()
LIMIT 80;

-- Shuffle merged list → LPUSH 200 ids
```

### Step 4: Cold Start (new user, no tag_weight rows)

```sql
SELECT DISTINCT post_id
FROM post_tags
WHERE post_id NOT IN (SELECT post_id FROM feed_post_seen WHERE user_id = :user_id)
ORDER BY RANDOM()
LIMIT 50;
```

### Step 5: Serve

1. `RPOP feed:{user_id} COUNT 25` — pop next 25 post_ids
2. `INSERT INTO feed_post_seen (user_id, post_id, seen_at)` for all 25 (ON CONFLICT DO NOTHING)
3. `POST {CONTENT_SERVICE_URL}/internal/posts/batch` with list of 25 post_ids
4. Return hydrated posts to client

### Tag Weight Update (via Redis consumer)

```
post.liked event   → UPSERT tag_weight SET weight = weight + 1
comment.created    → UPSERT tag_weight SET weight = weight + 5
post.unliked       → UPSERT tag_weight SET weight = GREATEST(weight - 1, 0)
post.viewed        → INSERT feed_post_seen ON CONFLICT DO NOTHING
post.deleted       → DELETE post_tags WHERE post_id = ?
                     UPDATE tags SET post_count = post_count - 1
```

---

## Notification Workflow

**Trigger**: Any content event with `notify_user_ids[]` → notification-service `notif_svc` consumer

### Flow

```
Redis event received by notif_svc
  │
  ├─► For each user_id in notify_user_ids[]:
  │     1. INSERT notifications row (always — for Activity tab)
  │     2. Is user_id connected to ws/notifications?
  │           YES → push live: { type, actor_uname, post_title, comment_id, ... }
  │           NO  → row stays in DB; user sees it on Activity tab next visit
  │
  └─► Return (no HTTP response needed — async consumer)
```

### Notification Types

| Type | Triggered By | notify_user_ids[] |
|------|-------------|-------------------|
| `like` | `post.liked` | `[post_author_id]` (suppressed if in WS room) |
| `comment` | `comment.created` (top-level) | `[post_author_id]` (suppressed if in WS room) |
| `reply` | `comment.created` (parent_id set) | `[parent_comment_author_id]` (suppressed if in thread WS room) |
| `mention` | `comment.created` (@ mention detected) | `[mentioned_user_ids[]]` |
| `post_deleted` | `post.deleted` by mod/admin | `[post_author_id]` (never suppressed) |
| `comment_deleted` | `comment.deleted` by mod/admin | `[comment_author_id]` (never suppressed) |

### Suppression Rule

```
Content Service performs suppression check BEFORE publishing:

For reply/mention/like:
  → Check WS room membership for affected users
  → Users already viewing the content live → excluded from notify_user_ids[]

For post_deleted / comment_deleted:
  → Always include post/comment author in notify_user_ids[]
  → No suppression (moderation must always notify)
```

### Live Push Payload (ws/notifications)

```json
{
  "type": "reply",
  "actor_id": 5,
  "actor_uname": "alice",
  "post_id": 42,
  "post_title": "...",
  "comment_id": 7,
  "created_at": "2024-01-01T12:00:00Z"
}
```

### Activity Tab (REST)

```
GET /api/v1/notifications?cursor=<last_id>&limit=20

Response:
{
  "notifications": [
    { "id": 15, "type": "like", "actor_uname": "bob", "post_title": "...", "is_read": false, ... }
  ],
  "next_cursor": "10",
  "unread_count": 3
}
```

Unread count query: `SELECT COUNT(*) FROM notifications WHERE user_id = ? AND is_read = false`

---

## WS Room Lifecycle (content-service)

```
Client opens post detail page
  └─► WS connect to ws/post/{post_id}
        → joined room post:{post_id}

Client expands comment thread
  └─► WS connect to ws/post/{post_id}/thread/{comment_id}
        → joined room post:{post_id}/thread:{comment_id}

Client closes page
  └─► WS disconnect → removed from room registry
```

**Post deletion WS flow** (mod/admin deletes post):
1. `DELETE /posts/{post_id}` called
2. content-service pushes `{ "event": "post_deleted" }` to all clients in room `post:{post_id}`
3. Room closed
4. `post.deleted` event emitted to Redis → notif_svc notifies author

---

## Rules

| ✅ DO | ❌ DON'T |
|-------|---------|
| Always INSERT notification row (persistence), then push if connected | Skip DB insert when user is online (breaks Activity tab) |
| Suppress notifications for users in WS rooms (except moderation) | Notify users who are already watching content live |
| Use RPOP (not LPOP) to drain feed in order | Randomly sample from Redis list |
| Reset feed cache (delete key) when user has no more unseen posts | Serve empty/stale list forever |
| Seed tag_weight from first interactions for cold-start users | Block feed until user has tag history |
