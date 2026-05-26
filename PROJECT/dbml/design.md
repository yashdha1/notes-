# Microservice Schema Design  NEW -> IMPORTANT : 

> 4 services, each with its own database.
> Cross-service IDs are logical references (not DB-level FKs).
> Denormalized fields marked with `[note: 'denormalized ...']`.

---

## 1. User Service (`:8001`)

```dbml
Table users {
  id           int       [pk, increment]
  email        varchar   [unique, not null]
  uname        varchar   [unique, not null]
  password     varchar   [not null]
  avatar       varchar
  bio          text
  role         varchar   [not null, default: 'user', note: 'admin | mod | user']
  created_at   timestamp
}

Table password_reset_tokens {
  id           int       [pk, increment]
  user_id      int       [not null]
  token        varchar   [not null]
  expires_at   timestamp [not null]
  used         boolean   [not null, default: false]
  created_at   timestamp
}
```

**Refresh tokens are stored in Redis (not a DB table)**:
```
Key:   refresh:{jti}          # jti = UUID from the refresh token's JWT claim
Value: {"user_id": 1}         # JSON
TTL:   604800 seconds (7 days)
```
Revocation = `DEL refresh:{jti}`. Validation = Redis key must exist after JWT signature check.

---

## 2. Content Service (`:8002`)

```dbml
Table post {
  id             int       [pk, increment]
  user_id        int       [not null]
  author_uname   varchar   [not null, note: 'denormalized from User Service']
  author_avatar  varchar   [note: 'denormalized from User Service']
  title          varchar   [not null]
  content        text
  image_link     varchar
  like_count     int       [not null, default: 0, note: 'denormalized from post_likes']
  comment_count  int       [not null, default: 0, note: 'denormalized from comment']
  tag_names      varchar   [note: 'space-separated tag names, denormalized at write time for FTS']
  search_vector  tsvector  [note: 'GIN indexed; title+tag_names weight A, content weight B; maintained by DB trigger']
  created_at     timestamp
  updated_at     timestamp

  indexes {
    search_vector [type: gin, name: 'idx_post_fts', note: 'PostgreSQL FTS index']
  }
}

// search_vector is maintained by a BEFORE INSERT OR UPDATE trigger:
//   setweight(to_tsvector('english', title),     'A') ||
//   setweight(to_tsvector('english', tag_names), 'A') ||
//   setweight(to_tsvector('english', content),   'B')

Table comment {
  id             int       [pk, increment]
  post_id        int       [not null, ref: > post.id]
  parent_id      int       [ref: > comment.id, note: 'null = top-level comment']
  user_id        int       [not null]
  author_uname   varchar   [not null, note: 'denormalized from User Service']
  author_avatar  varchar   [note: 'denormalized from User Service']
  content        text      [not null]
  like_count     int       [not null, default: 0, note: 'denormalized from comment_likes']
  created_at     timestamp
  updated_at     timestamp
}

Table post_likes {
  post_id      int       [not null, ref: > post.id]
  user_id      int       [not null]
  created_at   timestamp

  indexes {
    (post_id, user_id) [pk]
  }
}

Table comment_likes {
  comment_id   int       [not null, ref: > comment.id]
  user_id      int       [not null]
  created_at   timestamp

  indexes {
    (comment_id, user_id) [pk]
  }
}
```

---

## 3. Feed / Tag Service (`:8003`)

```dbml
Table tags {
  id           int       [pk, increment]
  name         varchar   [unique, not null]
  post_count   int       [not null, default: 0, note: 'denormalized; updated via content.events']
  created_at   timestamp
}

Table post_tags {
  post_id      int       [not null, note: 'logical ref to Content Service post.id']
  tag_id       int       [not null, ref: > tags.id]
  indexes {
    (post_id, tag_id) [pk]
    (tag_id, post_id) [note: 'reverse index for feed query']
  }
}

Table tag_weight {
  user_id      int       [not null, note: 'logical ref to User Service users.id']
  tag_id       int       [not null, ref: > tags.id]
  weight       float     [not null, default: 0, note: 'like: +1, comment: +5']
  updated_at   timestamp
  indexes {
    (user_id, tag_id) [pk]
    (user_id, weight) [name: 'idx_user_top_tags', note: 'fast top-K lookup for feed']
  }
}

Table post_views {
  post_id      int       [not null, ref: > post.id]
  user_id      int       [not null]
  viewed_at    timestamp
  indexes {
    (post_id, user_id) [pk]
  }
}
```

---

## 4. Notification Service (`:8004`)

```dbml
Table notifications {
  id           int       [pk, increment]
  user_id      int       [not null, note: 'recipient; logical ref to User Service']
  actor_id     int       [not null, note: 'who triggered it; logical ref to User Service']
  actor_uname  varchar   [not null, note: 'denormalized from event payload']
  type         varchar   [not null, note: 'reply | mention | like | comment | post_deleted | comment_deleted']
  post_id      int       [note: 'thread context; logical ref to Content Service']
  post_title   varchar   [note: 'denormalized from event payload']
  comment_id   int       [note: 'set for reply/mention, null for post-level like']
  is_read      boolean   [not null, default: false]
  created_at   timestamp

  indexes {
    (user_id, created_at) [note: 'Activity tab pagination']
    (user_id, is_read)    [note: 'unread count badge']
  }
}
```

---

## WebSocket Room Strategy (Content Service)

### Two levels of rooms

1. **Post room** — `ws/post/{post_id}`
   - Client joins when they open a post detail page.
   - Receives: live `like_count` updates, new top-level comments.
   - On the feed/listing page, `like_count` is served statically from `post.like_count` (no WS).

2. **Thread room** — `ws/post/{post_id}/thread/{comment_id}`
   - Client joins when they expand or engage with a comment thread.
   - Receives: live replies within that thread (any depth under `comment_id`).
   - Anyone can join this room (not just the parent commenter).

### Live content vs live notifications (two separate concerns)

- **Post/thread rooms** (Content Service) = live content updates (like counts, new comments appearing inline).
- **Notification channel** (Notification Service) = live notification bell/badge for active users.

These are independent. A user in a thread room sees replies inline. A user with a notification channel gets badge updates. Both can happen at the same time.

### 3. Notification channel — `ws/notifications`

- Client opens when the user logs in / loads any page (global connection, not per-post).
- Receives: live notification payloads (reply, mention, like) for the badge/popup.
- Managed by Notification Service.

### Notification flow

```
Event happens (reply, mention, like)
  → Content Service publishes to Redis Stream (always, for all affected users)
  → Notification Service consumes the event:
      1. ALWAYS INSERT into notifications table (persistence for Activity tab)
      2. Check: is recipient connected to ws/notifications?
         YES → push notification live over WS (badge count + payload)
         NO  → row sits in DB, user sees it in Activity tab on next visit
```

Content Service still checks thread room presence to decide whether to **suppress** the notification for users who are already watching:

```
User A replies to User B's comment in thread #7

Content Service checks: is User B in room post:42/thread:7?
  YES → User B sees the reply live in the thread
        skip User B in notify_user_ids[] (no redundant notification)
  NO  → include User B in notify_user_ids[]
        Notification Service persists row + pushes live if B has ws/notifications open
```

This avoids the UX problem of getting both an inline reply AND a notification bell for the same event.

---

## Redis Stream Event Contract

Stream: `content.events`

| Event             | Payload fields                                                                                                                             | Consumers           |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------ | ------------------- |
| `post.created`    | `post_id, user_id, author_uname, title, tag_names[], tag_ids[]`                                                                            | feed_svc            |
| `post.liked`      | `post_id, user_id, author_uname, like_count, post_author_id, post_title, tag_ids[], notify_user_ids[]`                                     | feed_svc, notif_svc |
| `post.unliked`    | `post_id, user_id, like_count, tag_ids[]`                                                                                                  | feed_svc            |
| `comment.created` | `post_id, comment_id, parent_id, user_id, author_uname, content, post_author_id, post_title, tag_ids[], notify_user_ids[], notify_types{}` | feed_svc, notif_svc |
| `post.viewed`     | `post_id, user_id`                                                                                                                         | feed_svc            |
| `post.deleted`    | `post_id, post_author_id, post_title, deleted_by_id, deleted_by_uname, tag_ids[], notify_user_ids[]`                                       | feed_svc, notif_svc |
| `comment.deleted` | `post_id, comment_id, comment_author_id, post_title, deleted_by_id, deleted_by_uname, notify_user_ids[]`                                   | notif_svc           |

Content Service performs **suppression checks** before publishing: users in the relevant WS room are excluded from `notify_user_ids[]` (they see content live, no duplicate bell). Moderation events (`post.deleted`, `comment.deleted`) **always** notify the content author — no suppression.

Consumer groups:
- `feed_svc` — reads like/comment/view/delete events to UPSERT `tag_weight`, update `tags.post_count`, and track views
- `notif_svc` — reads events with `notify_user_ids[]`: INSERT `notifications` row per user, push live via `ws/notifications` if recipient is connected

---

## Nginx Gateway Route Map

| Route pattern              | Upstream                        |
|----------------------------|---------------------------------|
| `/api/v1/user/*`           | User Service `:8001`            |
| `/api/v1/post/*`           | Content Service `:8002`         |
| `/api/v1/comment/*`        | Content Service `:8002`         |
| `/ws/post/{id}`            | Content Service `:8002` (WS post room)        |
| `/ws/post/{id}/thread/{id}`| Content Service `:8002` (WS thread room)      |
| `/ws/notifications`        | Notification Service `:8004` (WS notif channel)|
| `/api/v1/feed/*`           | Feed/Tag Service `:8003`        |
| `/api/v1/tag/*`            | Feed/Tag Service `:8003`        |
| `/api/v1/notifications/*`  | Notification Service `:8004`    |
| `/health`                  | Each service (own health check) |

JWT validation: Each service independently validates the RS256 Bearer token using the public key. Nginx is a pure router. No per-request call to User Service.

---

## Cross-Service REST Calls (minimal)

1. **Feed Service → Content Service**: `POST /internal/posts/batch` — given post IDs from feed query, return post details. Feed build only.

All other communication flows through Redis Streams events.

> **Removed**: `Gateway → User Service` JWT validation call.
> JWT validation is now stateless — each service decodes the RS256 Bearer token locally.

 