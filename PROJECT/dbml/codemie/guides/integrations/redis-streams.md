# Redis Streams Integration Guide

**Source**: design.md, paln_imp_cursor.md
**Stream key**: `content.events`
**Producer**: content-service (only)
**Consumers**: feed-service (`feed_svc` group), notification-service (`notif_svc` group)

---

## Architecture

```
content-service
  └─► XADD content.events {type, ...payload}
                │
                ├─► Consumer group: feed_svc  (feed-service/workers/)
                │     └─► tag_weight updates, post_count, feed_post_seen
                │
                └─► Consumer group: notif_svc (notification-service/workers/)
                      └─► INSERT notifications + live WS push
```

**Rule**: Content Service always publishes to Redis; it never calls feed-service or notification-service via HTTP.

---

## Event Contract

Stream: `content.events`

| Event | Payload Fields | Consumers |
|-------|----------------|-----------|
| `post.created` | `post_id, user_id, author_uname, title, tag_names[], tag_ids[]` | feed_svc |
| `post.liked` | `post_id, user_id, author_uname, like_count, post_author_id, post_title, tag_ids[], notify_user_ids[]` | feed_svc, notif_svc |
| `post.unliked` | `post_id, user_id, like_count, tag_ids[]` | feed_svc |
| `post.viewed` | `post_id, user_id` | feed_svc |
| `post.deleted` | `post_id, post_author_id, post_title, deleted_by_id, deleted_by_uname, tag_ids[], notify_user_ids[]` | feed_svc, notif_svc |
| `comment.created` | `post_id, comment_id, parent_id, user_id, author_uname, content, post_author_id, post_title, tag_ids[], notify_user_ids[], notify_types{}` | feed_svc, notif_svc |
| `comment.deleted` | `post_id, comment_id, comment_author_id, post_title, deleted_by_id, deleted_by_uname, notify_user_ids[]` | notif_svc |

---

## Suppression Logic (content-service responsibility)

Before publishing an event, content-service checks WS room membership to avoid duplicate notifications for users already watching the content.

```
Example: User A likes post 42

content-service checks: is post author in WS room post:42?
  YES → post author sees like_count update live in the thread
        → notify_user_ids = []  (skip notification)
  NO  → notify_user_ids = [post_author_id]
        → notif_svc creates notification row + live push
```

**Exception**: Moderation events (`post.deleted`, `comment.deleted`) **always** notify the content author — no suppression regardless of WS room membership.

---

## Consumer Group Responsibilities

### feed_svc (feed-service/workers/)

| Event | Action |
|-------|--------|
| `post.created` | Insert `post_tags` rows, increment `tags.post_count` |
| `post.liked` | `UPSERT tag_weight (user_id, tag_id) weight += 1` for each `tag_id` in payload |
| `comment.created` | `UPSERT tag_weight weight += 5` for each `tag_id` (comment = stronger engagement) |
| `post.viewed` | Insert `feed_post_seen (user_id, post_id)` |
| `post.deleted` | Delete `post_tags` rows, decrement `tags.post_count` |
| `post.unliked` | `UPSERT tag_weight weight -= 1` (floor at 0) |

### notif_svc (notification-service/workers/)

For each `user_id` in `notify_user_ids[]`:
1. **Always**: `INSERT notifications` row (persistence for Activity tab)
2. **If recipient has open `ws/notifications` connection**: push live payload

```
Notification types mapped from events:
  post.liked       → type: "like"
  comment.created  → type: "comment" | "reply" | "mention" (from notify_types{})
  post.deleted     → type: "post_deleted"
  comment.deleted  → type: "comment_deleted"
```

---

## Redis Key Conventions

| Key | Type | Purpose | TTL |
|-----|------|---------|-----|
| `content.events` | Stream | Event bus | No TTL (consumer group manages offset) |
| `feed:{user_id}` | List | Cached feed post IDs | 10 minutes |

---

## WebSocket Room Keys

Content-service manages two WS room types (in-memory registry):

| Room Key | Who Joins | Events Broadcast |
|----------|-----------|------------------|
| `post:{post_id}` | Users viewing post detail page | `like_update`, new top-level comments |
| `post:{post_id}/thread:{comment_id}` | Users expanded into a thread | New replies at any depth under `comment_id` |

Notification-service manages one channel per connected user (not post-scoped):

| Channel | Who Joins | Events |
|---------|-----------|--------|
| `ws/notifications` (user-scoped) | Any logged-in user | Live notification payloads |

---

## Publishing Pattern (content-service)

```python
# services/post_service.py — example: publish like event
async def like_post(self, post_id: int, user: dict) -> dict:
    # 1. DB write
    await self.db.execute(insert(PostLikes).values(post_id=post_id, user_id=user["id"]))
    new_count = await self._increment_like_count(post_id)

    # 2. WS broadcast (in-room update)
    await ws_manager.broadcast(f"post:{post_id}", {"event": "like_update", "like_count": new_count})

    # 3. Suppression check
    notify_ids = [] if ws_manager.is_connected(post_author_id, f"post:{post_id}") else [post_author_id]

    # 4. Emit to Redis
    await redis.xadd("content.events", {
        "type": "post.liked",
        "post_id": post_id,
        "user_id": user["id"],
        "author_uname": user["uname"],
        "like_count": new_count,
        "post_author_id": post_author_id,
        "post_title": post_title,
        "tag_ids": json.dumps(tag_ids),
        "notify_user_ids": json.dumps(notify_ids),
    })
    return {"like_count": new_count}
```

---

## Consumer Worker Pattern (feed-service example)

```python
# workers/feed_consumer.py
async def run():
    await redis.xgroup_create("content.events", "feed_svc", id="0", mkstream=True)
    while True:
        messages = await redis.xreadgroup("feed_svc", "worker-1", {"content.events": ">"}, count=10)
        for stream, entries in messages:
            for msg_id, fields in entries:
                await handle_event(fields)
                await redis.xack("content.events", "feed_svc", msg_id)
```

---

## Rules

| ✅ DO | ❌ DON'T |
|-------|---------|
| Always XACK after successful processing | Leave messages unacknowledged (causes redelivery) |
| Handle idempotency (upsert, not insert blindly) | Assume messages arrive exactly once |
| Include all consuming fields in the event payload | Let consumers make HTTP calls to fetch missing data |
| Suppress notifications for users in WS rooms (except mod actions) | Always notify regardless of WS presence |
| consumer group per service, one group = one logical consumer | Multiple services sharing one consumer group |
