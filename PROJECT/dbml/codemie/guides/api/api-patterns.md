# API Patterns Guide

**Source**: api.md
**Base prefix**: `/api/v1/`
**Auth mechanism**: RS256 JWT — each service validates the Bearer token independently

---

## Auth Convention

Each microservice decodes and verifies the JWT locally using the shared public key.
Nginx is a pure router — it does **not** validate JWT or inject headers.

**In deps.py** — decode JWT with public key, extract user context:
```python
# src/<pkg>/api/deps.py
from dataclasses import dataclass
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import jwt, JWTError
from ..core.config import settings

security = HTTPBearer()

@dataclass
class UserContext:
    id: int
    role: str
    uname: str
    avatar: str | None

def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> UserContext:
    try:
        payload = jwt.decode(
            credentials.credentials,
            settings.JWT_PUBLIC_KEY,
            algorithms=["RS256"],
            audience=settings.JWT_AUDIENCE,
            issuer=settings.JWT_ISSUER,
        )
    except JWTError:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid or expired token")

    return UserContext(
        id=int(payload["sub"]),
        role=payload["role"],
        uname=payload["uname"],
        avatar=payload.get("avatar"),
    )
```

**Config required in every service** (`core/config.py`):
```python
JWT_PUBLIC_KEY: str   # PEM string from env var or mounted file
JWT_ISSUER: str = "user-service"
JWT_AUDIENCE: str = "social-media-api"
```

---

## Pagination

All list endpoints use **cursor-based pagination** (not page/offset).

```
GET /posts/search?q=python&cursor=<last_id>&limit=10

Response:
{
  "posts": [...],
  "next_cursor": "35"   # null if no more results
}
```

- Default limit: 10 · Max limit: 50
- Cursor = last returned item's `id`
- SQL pattern: `WHERE id < :cursor ORDER BY id DESC LIMIT :limit`

---

## Service Base Paths

| Service | Base Path |
|---------|-----------|
| user-service | `/api/v1/users` |
| content-service (posts) | `/api/v1/posts` |
| content-service (comments) | `/api/v1/comments` |
| feed-service (feed) | `/api/v1/feed` |
| feed-service (tags) | `/api/v1/tags` |
| notification-service | `/api/v1/notifications` |

---

## User Service Endpoints (`:8001`)

### Auth (public — no gateway auth required)

| Method | Path                           | Description                                |
| ------ | ------------------------------ | ------------------------------------------ |
| POST   | `/auth/register`               | Register → returns access + refresh tokens |
| POST   | `/auth/login`                  | Login → returns access + refresh tokens    |
| POST   | `/auth/logout`                 | Invalidate refresh token                   |
| POST   | `/auth/refresh`                | Exchange refresh token → new access token  |


### Profile (auth required)

| Method | Path                 | Description                               |     |
| ------ | -------------------- | ----------------------------------------- | --- |
| GET    | `/profile`           | Own full profile                          |     |
| GET    | `/profile/{user_id}` | Public profile (uname, avatar, bio, role) |     |
| PATCH  | `/profile`           | Update bio, avatar, uname                 |     |
| DELETE | `/profile`           | Delete own account                        |     |
|        |                      |                                           |     |

### Admin (role: admin)

| Method | Path                          | Description                |     |
| ------ | ----------------------------- | -------------------------- | --- |
| GET    | `/admin/users`                | List all users (paginated) |     |
| PATCH  | `/admin/users/{user_id}/role` | Promote/demote role        |     |

**add tags** -> in the tags and the feed database.

---

## Content Service Endpoints (`:8002`)

### Posts :-> 

| Method | Path                    | Auth            | Description                  |
| ------ | ----------------------- | --------------- | ---------------------------- |
| POST   | `/posts`                | required        | Create post with tags        |
| GET    | `/posts/search?q=...`   | required        | FTS: title + content + tags  |
| GET    | `/posts/{post_id}`      | required        | Get post; marks as viewed    |
| PATCH  | `/posts/{post_id}`      | owner/mod/admin | Edit post                    |
| DELETE | `/posts/{post_id}`      | owner/mod/admin | Delete post                  |
| POST   | `/posts/{post_id}/like` | required        | Like (composite PK enforced) |
| DELETE | `/posts/{post_id}/like` | required        | Unlike                       |
|        |                         |                 |                              |

**POST /posts request**:
```json
{ "title": "...", "content": "...", "image_link": "...", "tag_names": ["python", "webdev"] }
```

**GET /posts/search response**:
```json
{ "query": "...", "posts": [...], "next_cursor": "35" }
```

### Comments

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | `/comments` | required | Create comment (top-level or reply via `parent_id`) |
| PATCH | `/comments/{comment_id}` | owner/mod/admin | Edit comment |
| DELETE | `/comments/{comment_id}` | owner/mod/admin | Delete comment |
| POST | `/comments/{comment_id}/like` | required | Like comment |
| DELETE | `/comments/{comment_id}/like` | required | Unlike comment |

### Internal (NOT exposed via gateway)

| Method | Path | Description |
|--------|------|-------------|
| POST | `/internal/posts/batch` | Given post IDs → return post details (for feed-service) |

---

## Feed Service Endpoints (`:8003`)

| Method | Path | Description |
|--------|------|-------------|
| GET | `/feed` | Paginated personalized feed (`?cursor=&limit=`) |
| GET | `/tags` | List tags (paginated, sorted by post_count) |
| GET | `/tags/search?q=...` | Fuzzy tag search |

---

## Notification Service Endpoints (`:8004`)
https://fastapi.tiangolo.com/tutorial/background-tasks/ 
for the backround tasks in the fastapi : 

| Method | Path | Description |
|--------|------|-------------|
| GET | `/notifications` | Activity tab (paginated, cursor-based) |
| PATCH | `/notifications/{id}/read` | Mark single notification as read |
| PATCH | `/notifications/read-all` | Mark all as read |

---

## WebSocket Routes

| Path | Service | Who joins | Receives |
|------|---------|-----------|----------|
| `ws/post/{post_id}` | content-service | Users viewing post detail | `like_update`, new top-level comments |
| `ws/post/{post_id}/thread/{comment_id}` | content-service | Users in a comment thread | Live replies at any depth |
| `ws/notifications` | notification-service | Logged-in users (global) | `{type, actor_uname, post_title, ...}` |

**WS message format** (content service broadcasts):
```json
{ "event": "like_update", "like_count": 16 }
{ "event": "new_comment", "comment": { "id": 7, "content": "...", ... } }
{ "event": "post_deleted" }
```

---

## Role-Based Access

| Role    | What they can do                                           |
| ------- | ---------------------------------------------------------- |
| `user`  | Own content only (create, edit, delete own posts/comments) |
| `mod`   | Delete/edit any post or comment (moderation)               |
| `admin` | Everything + promote/demote roles                          |

Ownership check: `post.user_id == user.id` OR `user.role in ["mod", "admin"]` (user context from JWT via `get_current_user`).

---

## HTTP Status Codes

| Status | When |
|--------|------|
| 200 | Success (GET, PATCH) |
| 201 | Created (POST creating a resource) |
| 204 | Success, no body (DELETE) |
| 400 | Bad request / validation error |
| 401 | Missing or invalid auth |
| 403 | Authenticated but insufficient role |
| 404 | Resource not found |
| 409 | Conflict (duplicate email, duplicate like) |

---

## Side Effects Pattern

Every write endpoint declares side effects explicitly. Example for `POST /posts/{id}/like`:

1. `INSERT post_likes` + `UPDATE post.like_count` (atomic)
2. WS broadcast to room `post:{id}`
3. Check suppression (is author in WS room?)
4. `XADD content.events` with `notify_user_ids[]`

Always implement side effects in the `services/` layer, not in the router.
