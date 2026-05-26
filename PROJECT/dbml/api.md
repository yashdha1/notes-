# API Endpoint Specification

> All endpoints use JSON request/response bodies.
> Auth-required endpoints receive user context from the decoded JWT Bearer token.
> Each service validates the RS256 JWT locally using the shared public key.
> JWT claims available to services: `sub` (user_id), `role`, `uname`, `avatar`, `iss`, `aud`, `exp`.
> Pagination uses cursor-based approach: `?cursor=<last_id>&limit=<n>`.

---

API gateway :- 
## 1. User Service `:8001`

Base path: `/api/v1/users`

### Auth (public)

| Method | Path                           | Auth     | Description                                                          |
| ------ | ------------------------------ | -------- | -------------------------------------------------------------------- |
| POST   | `/auth/register`               | none     | Register with email, password. Returns access + refresh tokens.      |
| POST   | `/auth/login`                  | none     | Login with email + password. Returns access + refresh tokens.        |
| POST   | `/auth/logout`                 | required | Invalidate the refresh token (server-side revocation).               |
| POST   | `/auth/refresh`                | none     | Exchange a valid refresh token (in body) for a new access token.     |
| POST   | `/auth/password-reset/request` | none     | Send OTP/reset link to email. Creates a `password_reset_tokens` row. |


**`POST /auth/register`**
```
Request:  { "email": "...", "uname": "...", "password": "..." }
Response: { "access_token": "...", "refresh_token": "..." }
```

**`POST /auth/login`**
```
Request:  { "email": "...", "password": "..." }
Response: { "access_token": "...", "refresh_token": "..." }
```

**`POST /auth/refresh`**
```
Request:  { "refresh_token": "..." }
Response: { "access_token": "..." }
```

**`POST /auth/password-reset/request`**
```
Request:  { "email": "..." }
Response: { "message": "Reset link sent to email" }
```

**`POST /auth/password-reset/confirm`**
```
Request:  { "token": "...", "new_password": "..." }
Response: { "message": "Password updated" }
```

# **Postal service :->** 
### Profile (auth required)

| Method | Path                 | Auth     | Description                                                   |
| ------ | -------------------- | -------- | ------------------------------------------------------------- |
| GET    | `/profile`           | required | Get own profile (all fields).                                 |
| GET    | `/profile/{user_id}` | required | Get another user's public profile (uname, avatar, bio, role). |
| PATCH  | `/profile`           | required | Update own bio, avatar, or uname.                             |
| DELETE | `/profile`           | required | Delete own account.                                           |
| POST   | ```/profile```       | req      | uname, bio, avatar etc.                                       |
| PATCH  | /profile             | req      | update the bio uname, avatar, bio                             |

**`PATCH /profile`**
```
Request:  { "uname": "new_name", "avatar": "url", "bio": "text" }  (all optional)
Response: { "id": 1, "uname": "new_name", "avatar": "url", "bio": "text" }
```

### Admin (role: admin)

| Method | Path                          | Auth  | Description                                  |
| ------ | ----------------------------- | ----- | -------------------------------------------- |
| GET    | `/admin/users`                | admin | List all users (paginated). Admin dashboard. |
| PATCH  | `/admin/users/{user_id}/role` | admin | Promote or demote user role.                 |

**`PATCH /admin/users/{user_id}/role`**
```
Request:  { "role": "mod" }          (one of: admin, mod, user)
Response: { "id": 5, "role": "mod" }
```

---

## 2. Content Service `:8002`

Base paths: `/api/v1/posts`, `/api/v1/comments`

### Posts

| Method | Path               | Auth            | Description                                      |
| ------ | ------------------ | --------------- | ------------------------------------------------ |
| POST   | `/posts`           | required        | Create a post with tags.                         |
| GET    | `/posts/search`    | required        | Full-text search by title, content, and tags.    |
| GET    | `/posts/{post_id}` | required        | Get single post. Marks as viewed if first visit. |
| PATCH  | `/posts/{post_id}` | owner/mod/admin | Edit post title, content, or image_link.         |
| DELETE | `/posts/{post_id}` | owner/mod/admin | Delete a post.                                   |
|        |                    |                 |                                                  |

**`POST /posts`**
```
Request:  { "title": "...", "content": "...", "image_link": "...", "tag_names": ["python", "webdev"] }
Response: { "id": 42, "title": "...", ... }
```
Side effects:
- Writes `tag_names` as space-separated string onto `post.tag_names` (used by FTS trigger to build `search_vector`)
- Emits `post.created` to Redis (Feed Service resolves/creates tags, inserts `post_tags`)
- Emits `post.viewed` (author has seen their own post)

**`GET /posts/search`**
```
Query params:
  ?q=rust+async        (required -- the search query)
  ?cursor=<last_id>    (cursor-based pagination)
  ?limit=10            (default 10, max 50)

Response: {
  "query": "rust async",
  "posts": [
    { "id": 42, "title": "...", "author_uname": "alice", "author_avatar": "url",
      "like_count": 15, "comment_count": 7, "created_at": "...", "rank": 0.85 },
    ...
  ],
  "next_cursor": "35"
}
```
Internal logic:
- Queries `search_vector @@ to_tsquery('english', :q)` on the `post` table
- Orders by `ts_rank(search_vector, query) DESC, created_at DESC`
- Cursor pagination on post id
- `rank` field shows relevance score (0.0 to 1.0)
- Title and tag matches rank higher than content matches (weight A vs B)
- Returns 400 if `?q` is missing or empty

**`GET /posts/{post_id}`**
```
Response: {
  "id": 42,
  "user_id": 1,
  "author_uname": "alice",
  "author_avatar": "url",
  "title": "...",
  "content": "...",
  "image_link": "...",
  "like_count": 15,
  "comment_count": 7,
  "liked_by_me": true,
  "created_at": "...",
  "updated_at": "..."
}
```
Side effects:
- Inserts `post_views` row if user hasn't viewed before
- Emits `post.viewed` to Redis (Feed Service uses this for "unseen" filtering)

**`DELETE /posts/{post_id}`**
```
Response: { "message": "Post deleted" }
```
Side effects:
- If deleted by mod/admin (not owner): closes WS room `post:{post_id}`, pushes `{ "event": "post_deleted" }` to connected clients before closing
- Emits `post.deleted` to Redis with `deleted_by_id`, `deleted_by_uname`, `notify_user_ids: [post_author_id]`
  - Feed Service: removes `post_tags` rows, decrements `tags.post_count`
  - Notification Service: inserts notification (type=`post_deleted`) for the post author if deleted by mod/admin

### Post Likes

| Method | Path                    | Auth     | Description                                        |
| ------ | ----------------------- | -------- | -------------------------------------------------- |
| POST   | `/posts/{post_id}/like` | required | Like a post. One per user (composite PK enforced). |
| DELETE | `/posts/{post_id}/like` | required | Unlike a post.                                     |

**`POST /posts/{post_id}/like`**
```
Response: { "like_count": 16 }
```
Side effects:
- Increments `post.like_count` atomically
- WS broadcast to room `post:{post_id}`: `{ "event": "like_update", "like_count": 16 }`
- Suppression check: is post author in room `post:{post_id}`? If yes → `notify_user_ids = []`. If no → `notify_user_ids = [post_author_id]`
- Emits `post.liked` to Redis with `notify_user_ids[]` (Feed Service updates `tag_weight` +1; Notification Service creates like notification if not suppressed)

**`DELETE /posts/{post_id}/like`**
```
Response: { "like_count": 15 }
```
Side effects:
- Decrements `post.like_count` atomically
- Emits `post.unliked` to Redis
- WS broadcast to room `post:{post_id}`: `{ "event": "like_update", "like_count": 15 }`

### Comments

| Method | Path                             | Auth            | Description                                             |
| ------ | -------------------------------- | --------------- | ------------------------------------------------------- |
| GET    | `/posts/{post_id}/comments`      | required        | Get top-level comments (paginated). `parent_id = null`. |
| POST   | `/posts/{post_id}/comments`      | required        | Create a top-level comment on a post.                   |
| GET    | `/comments/{comment_id}/replies` | required        | Get replies to a comment (paginated, "load more").      |
| POST   | `/comments/{comment_id}/replies` | required        | Reply to a specific comment.                            |
| PATCH  | `/comments/{comment_id}`         | owner/mod/admin | Edit a comment's content.                               |
| DELETE | `/comments/{comment_id}`         | owner/mod/admin | Delete a comment.                                       |

**`GET /posts/{post_id}/comments`**
```
Query:    ?cursor=<last_comment_id>&limit=10
Response: {
  "comments": [
    { "id": 1, "post_id": 42, "parent_id": null, "user_id": 2, "author_uname": "bob",
      "author_avatar": "url", "content": "Great post!", "like_count": 3,
      "reply_count": 4, "created_at": "...", "updated_at": "..." },
    ...
  ],
  "next_cursor": "5"
}
```

**`POST /posts/{post_id}/comments`**
```
Request:  { "content": "Great post!" }
Response: { "id": 10, "post_id": 42, "parent_id": null, "content": "Great post!", ... }
```
Side effects:
- Increments `post.comment_count`
- WS broadcast to room `post:{post_id}`: `{ "event": "new_comment", "comment": {...} }`
- Parse content for @mentions → `mentioned_user_ids`
- Suppression check: is post author in room `post:{post_id}`? If yes → skip from notify list. Same check for each mentioned user.
- Emits `comment.created` to Redis with `notify_user_ids[]` and `notify_types{}` (Feed Service updates `tag_weight` +5; Notification Service creates comment/mention notifications for non-suppressed users)

**`POST /comments/{comment_id}/replies`**
```
Request:  { "content": "I agree!" }
Response: { "id": 11, "post_id": 42, "parent_id": 10, "content": "I agree!", ... }
```
Side effects:
- Increments `post.comment_count`
- WS broadcast to room `post:{post_id}/thread:{root_comment_id}`: `{ "event": "new_reply", "comment": {...} }`
- Parse content for @mentions → `mentioned_user_ids`
- Suppression checks (each potential recipient checked against the relevant room):
  - Post author: in room `post:{post_id}`? → skip/include (type=`comment`)
  - Parent comment author: in room `post:{post_id}/thread:{root_comment_id}`? → skip/include (type=`reply`)
  - @mentioned users: in room `post:{post_id}` OR thread room? → skip/include (type=`mention`)
- Emits `comment.created` to Redis with `notify_user_ids[]` and `notify_types{}` (Feed Service updates `tag_weight` +5; Notification Service creates notifications per type for non-suppressed users)

**`GET /comments/{comment_id}/replies`**
```
Query:    ?cursor=<last_reply_id>&limit=5
Response: {
  "replies": [ ... ],
  "next_cursor": "15"
}
```

**`DELETE /comments/{comment_id}`**
```
Response: { "message": "Comment deleted" }
```
Side effects:
- Decrements `post.comment_count`
- If deleted by mod/admin (not owner): emits `comment.deleted` to Redis with `deleted_by_id`, `deleted_by_uname`, `notify_user_ids: [comment_author_id]`
  - Notification Service: inserts notification (type=`comment_deleted`) for the comment author

### Comment Likes

| Method | Path                          | Auth     | Description                                  |
| ------ | ----------------------------- | -------- | -------------------------------------------- |
| POST   | `/comments/{comment_id}/like` | required | Like a comment. One per user (composite PK). |
| DELETE | `/comments/{comment_id}/like` | required | Unlike a comment.                            |

**`POST /comments/{comment_id}/like`**
```
Response: { "like_count": 4 }
```
Side effects:
- Increments `comment.like_count` atomically

**`DELETE /comments/{comment_id}/like`**
```
Response: { "like_count": 3 }
```
Side effects:
- Decrements `comment.like_count` atomically

### Internal (not exposed via gateway)

| Method | Path                    | Description                                                                 |
| ------ | ----------------------- | --------------------------------------------------------------------------- |
| POST   | `/internal/posts/batch` | Given `{ "post_ids": [1, 2, 3] }`, return post summaries. For Feed Service. |

**`POST /internal/posts/batch`**
```
Request:  { "post_ids": [42, 43, 44] }
Response: {
  "posts": [
    { "id": 42, "user_id": 1, "author_uname": "alice", "author_avatar": "url",
      "title": "...", "image_link": "...", "like_count": 15, "comment_count": 7,
      "created_at": "..." },
    ...
  ]
}
```

### WebSocket

| Endpoint                                    | Description                                  |
|---------------------------------------------|----------------------------------------------|
| `WS /ws/post/{post_id}`                     | Post room. Live like counts + new top-level comments. |
| `WS /ws/post/{post_id}/thread/{comment_id}` | Thread room. Live replies within a comment thread.    |

**Post room events pushed to client:**
```
{ "event": "like_update",  "like_count": 16 }
{ "event": "new_comment",  "comment": { "id": 10, "author_uname": "bob", "content": "...", ... } }
```

**Thread room events pushed to client:**
```
{ "event": "new_reply", "comment": { "id": 11, "parent_id": 10, "author_uname": "carol", "content": "...", ... } }
```

---

## 3. Feed / Tag Service `:8003`

Base paths: `/api/v1/feed`, `/api/v1/tags`

### Feed

| Method | Path    | Auth     | Description                               |
| ------ | ------- | -------- | ----------------------------------------- |
| GET    | `/feed` | required | Build personalized feed for current user. |

**`GET /feed`**
```
Query:    ?cursor=<last_post_id>&limit=10
Response: {
  "posts": [
    { "id": 42, "user_id": 1, "author_uname": "alice", "author_avatar": "url",
      "title": "...", "image_link": "...", "like_count": 15, "comment_count": 7,
      "created_at": "..." },
    ...
  ],
  "next_cursor": "38"
}
```
Internal logic:
1. Get user's top-K tags from `tag_weight` (ordered by weight DESC)
2. Find `post_ids` from `post_tags` for those tags
3. Exclude posts already in `post_views` for this user (via `post.viewed` events)
4. Shuffle results with recency bias
5. Call Content Service `POST /internal/posts/batch` with selected `post_ids`
6. Return paginated result

### Tags

| Method | Path                     | Auth      | Description                                                     |
| ------ | ------------------------ | --------- | --------------------------------------------------------------- |
| GET    | `/tags`                  | required  | Search/list tags for autocomplete. Sorted by `post_count DESC`. |
| POST   | `/tags`                  | mod/admin | Create a tag manually.                                          |
| DELETE | `/tags/{tag_id}`         | mod/admin | Delete a tag. Cascades post_tags + tag_weight rows.             |
| GET    | `/tags/{tag_name}/posts` | required  | Get posts tagged with this tag (paginated).                     |

**`GET /tags`**
```
Query:    ?q=pyt&limit=10
Response: {
  "tags": [
    { "id": 1, "name": "python", "post_count": 42 },
    { "id": 7, "name": "pytorch", "post_count": 18 },
    ...
  ]
}
```

**`POST /tags`**
```
Request:  { "name": "rust" }
Response: { "id": 12, "name": "rust", "post_count": 0 }
```

**`GET /tags/{tag_name}/posts`**
```
Query:    ?cursor=<last_post_id>&limit=10
Response: {
  "tag": { "id": 1, "name": "python", "post_count": 42 },
  "posts": [ ... ],
  "next_cursor": "35"
}
```
Internal logic: get `post_ids` from `post_tags` -> call Content Service `/internal/posts/batch`.

Note: tags are also auto-created when Feed Service consumes `post.created` events containing new tag names.

---

## 4. Notification Service `:8004`

Base path: `/api/v1/notifications`

### Activity Tab

| Method | Path                                    | Auth     | Description                                      |
| ------ | --------------------------------------- | -------- | ------------------------------------------------ |
| GET    | `/notifications`                        | required | List notifications (paginated, newest first).    |
| GET    | `/notifications/unread-count`           | required | Get unread count for badge.                      |
| PATCH  | `/notifications/{notification_id}/read` | required | Mark a single notification as read.              |
| PATCH  | `/notifications/read-all`               | required | Mark all notifications as read for current user. |

**`GET /notifications`**
```
Query:    ?cursor=<last_notification_id>&limit=20
Response: {
  "notifications": [
    { "id": 99, "actor_uname": "bob", "type": "reply",
      "post_id": 42, "post_title": "How to learn Rust",
      "comment_id": 11, "is_read": false, "created_at": "..." },
    { "id": 98, "actor_uname": "carol", "type": "mention",
      "post_id": 42, "post_title": "How to learn Rust",
      "comment_id": 15, "is_read": false, "created_at": "..." },
    { "id": 97, "actor_uname": "dave", "type": "comment",
      "post_id": 42, "post_title": "How to learn Rust",
      "comment_id": 20, "is_read": false, "created_at": "..." },
    { "id": 96, "actor_uname": "mod_x", "type": "post_deleted",
      "post_id": 55, "post_title": "Old post",
      "comment_id": null, "is_read": false, "created_at": "..." },
    ...
  ],
  "next_cursor": "85"
}
```
Notification types: `reply`, `mention`, `like`, `comment`, `post_deleted`, `comment_deleted`

**`GET /notifications/unread-count`**
```
Response: { "count": 7 }
```

**`PATCH /notifications/{notification_id}/read`**
```
Response: { "id": 99, "is_read": true }
```

**`PATCH /notifications/read-all`**
```
Response: { "message": "All notifications marked as read" }
```

### WebSocket

| Endpoint              | Description                                                          |
|-----------------------|----------------------------------------------------------------------|
| `WS /ws/notifications`| Global per-user channel. Opened on login. Pushes live notifications. |

**Notification events pushed to client:**
```
{ "event": "notification", "data": { "id": 100, "actor_uname": "dave", "type": "reply",
  "post_id": 42, "post_title": "How to learn Rust", "comment_id": 20, "created_at": "..." } }
{ "event": "unread_count", "count": 8 }
```

---

## Inter-Service Communication

```
Feed Service ──POST /internal/posts/batch──► Content Service
Content Service ──XADD content.events──► Redis Streams
Redis Streams ──consumer group: feed_svc──► Feed / Tag Service
Redis Streams ──consumer group: notif_svc──► Notification Service
```

Only **1 synchronous REST call** between services. Everything else is async via Redis Streams.

> JWT validation is stateless — each service verifies the Bearer token locally using the RS256 public key.
> There is no per-request call to User Service for auth validation.

---

## Auth / Permission Matrix

| Permission Level | Endpoints                                                                 |
|------------------|---------------------------------------------------------------------------|
| **Public**       | register, login, refresh, password-reset                                  |
| **Auth required**| profile, create/edit/delete own content, like/unlike, feed, notifications |
| **Mod + Admin**  | edit/delete any post/comment, create/delete tags                          |
| **Admin only**   | list users, promote/demote roles                                          |

Ownership checks: `user.id` (from JWT `sub`) is matched against `post.user_id` or `comment.user_id` for edit/delete of own content.
Role checks: `user.role` (from JWT `role` claim) is checked for mod/admin-only operations.


api specs for the project and the frontend : 

## Repository layout (monorepo) backend imp refer the notes too

One repository keeps paths predictable and matches how [stories.md](stories.md) epics map to code.

```text
practice/                          # repository root (name may vary)
├── pyproject.toml                 # uv workspace root: tool.uv.workspace members
├── README.md
├── services/
│   ├── user-service/
│   ├── content-service/
│   ├── feed-service/
│   └── notification-service/
├── packages/                      # optional — keep small
│   └── common/                    # shared constants or event payload types only; avoid a “god library”
└── infra/                         # optional: docker-compose, sample Nginx, env templates
```

---

## Layout inside each service

Use the **same shape** in every `services/<name>/` so beginners always know where to look.

```text
services/<service-name>/
├── pyproject.toml                 # dependencies; member of uv workspace
├── README.md                      # how to run migrations and the dev server
├── src/
│   └── <service_pkg>/             # e.g. user_service, content_service (single import root)
│       ├── main.py                # FastAPI app, include routers, lifespan (DB engine, Redis if needed)
│       ├── core/
│       │   └── config.py          # env-based settings (DATABASE_URL, REDIS_URL, secrets, URLs)
│       ├── api/
│       │   ├── deps.py            # get_db, optional user context from X-User-* headers
│       │   └── v1/                # routers: auth.py, posts.py, ... matching api.md base paths
│       ├── schemas/               # Pydantic models for JSON in/out
│       ├── models/                # SQLAlchemy models
│       ├── db/
│       │   └── session.py         # engine, session factory, Base
│       └── services/              # business logic (routers stay thin)
├── tests/                         # pytest; httpx AsyncClient against the app
``` 
--- 


