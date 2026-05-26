# Architecture Guide

**Project**: Social Media Microservice Backend
**Style**: Microservices (event-driven)
**Language**: Python | **Framework**: FastAPI | **Package Manager**: uv

---

## Architecture Overview

```
Client




**Key Decision**: Zero inter-service REST on hot path — async Redis Streams replace synchronous calls. Only one cross-service REST call exists (feed-service→content-service for batch post hydration). JWT validation is stateless in each service — no per-request call to user-service.

---

## Service Map

| Service | Port | DB | Responsibility |
|---------|------|----|----------------|
| user-service | 8001 | users DB | Auth, profiles, admin role management |
| content-service | 8002 | content DB | Posts, comments, likes, views, WS rooms |
| feed-service | 8003 | feed DB | Personalized feed, tags, tag weights |
| notification-service | 8004 | notif DB | Notification persistence, WS delivery |

Each service owns its database — **no shared DB, no cross-service FKs**.

---

## Component Structure (per service)

```
services/<service-name>/
├── pyproject.toml
├── src/<service_pkg>/
│   ├── main.py          # FastAPI app, routers, lifespan
│   ├── core/config.py   # pydantic-settings env config
│   ├── api/
│   │   ├── deps.py      # get_db, user context from X-User-* headers
│   │   └── v1/          # routers matching api.md base paths
│   ├── schemas/         # Pydantic request/response models
│   ├── models/          # SQLAlchemy ORM models
│   ├── db/session.py    # engine, session factory, Base
│   └── services/        # business logic (routers stay thin)
├── alembic/             # migrations — this service only
└── tests/
```

**Service-specific additions:**

| Service | Extra |
|---------|-------|
| user-service | `services/auth.py` — token issuance (RS256 sign), Redis refresh token management |
| content-service | `ws/` — WebSocket handlers, room registry, broadcast helpers |
| feed-service | `workers/` — Redis Streams consumer (group: `feed_svc`) + HTTP client to content-service |
| notification-service | `workers/` — Redis Streams consumer (group: `notif_svc`) + `ws/` for live delivery |

---

## Design Patterns

| Pattern | Usage | Where |
|---------|-------|-------|
| Thin Routers | Routers delegate to `services/`, no business logic inline | `api/v1/*.py` |
| Repository-like Services | `services/` layer owns all DB queries | `services/*.py` |
| Denormalization | Write-time copies of user fields to avoid cross-service reads | design.md |
| Event Sourcing (light) | Content Service emits all events; consumers react | `content.events` stream |
| Observer (Redis Streams) | Feed and Notif services are event consumers | `workers/` |
| Suppression Check | WS room membership checked before adding to `notify_user_ids[]` | ws/ + redis |

---

## Layer Responsibilities

| Layer | Responsibility | Rule |
|-------|----------------|------|
| `api/v1/` (routers) | Parse request, call service, return response | No DB queries, no business logic |
| `services/` | Business logic, side effects, event emission | Owns DB + Redis calls |
| `models/` | SQLAlchemy table definitions | No logic |
| `schemas/` | Pydantic validation and serialization | No DB access |
| `core/config.py` | Typed env config via pydantic-settings | Read-only singleton |
| `db/session.py` | DB engine, session factory | Injected via `deps.py` |

---

## Data Flow (typical write request)

```
Client POST /api/v1/posts/42/like
  │
  ├─► Nginx routes request (pure proxy)
  │
  ├─► content-service decodes Bearer JWT locally (RS256 public key)
  │     └─► get_current_user() → UserContext(id, role, uname, avatar)
  │
  ├─► content-service router (api/v1/posts.py)
  │     └─► PostService.like_post()
  │           ├─► INSERT post_likes + UPDATE post.like_count (atomic)
  │           ├─► WS broadcast to room post:42 {event: like_update, like_count: N}
  │           └─► XADD content.events {type: post.liked, ...}
  │
  ├─► feed-service worker (async, Redis consumer)
  │     └─► UPSERT tag_weight +1 for each tag on post 42
  │
  └─► notification-service worker (async, Redis consumer)
        └─► INSERT notification for post author (if not suppressed)
              └─► Push live via ws/notifications if author connected
```

---

## Nginx Gateway Route Map

| Route Pattern | Upstream |
|---------------|----------|
| `/api/v1/user/*` | user-service :8001 |
| `/api/v1/post/*`, `/api/v1/comment/*` | content-service :8002 |
| `/ws/post/{id}`, `/ws/post/{id}/thread/{id}` | content-service :8002 |
| `/api/v1/feed/*`, `/api/v1/tag/*` | feed-service :8003 |
| `/api/v1/notifications/*` | notification-service :8004 |
| `/ws/notifications` | notification-service :8004 |
| `/health` | each service (own endpoint) |

---

## Cross-Service REST Calls (minimal)

| Call | Who | When |
|------|-----|------|
| `POST /internal/posts/batch` | feed-service → content-service | Feed hydration only |

All other integration = Redis Streams (async).

> **Removed**: `POST /internal/auth/validate` (gateway→user-service).
> JWT validation is now stateless — each service decodes and verifies the token locally using the public key.

---

## Boundaries

| ✅ DO | ❌ DON'T |
|-------|---------|
| Each service owns its DB exclusively | Share DB tables across services |
| Use Redis Stream for cross-service events | Add synchronous REST calls on hot path |
| Denormalize user fields at write time (from JWT claims) | Call user-service at read time for username/avatar |
| Keep business logic in `services/` layer | Put logic in routers or models |
| Validate JWT in each service using the public key | Re-validate JWT by calling user-service per request |
| Pass only public key to non-auth services | Share private key outside user-service |

---

## Adding a New Service

1. Create `services/<new-service>/` with standard layout
2. Add `pyproject.toml` as uv workspace member
3. Register route prefix in Nginx config
4. Subscribe to relevant `content.events` in `workers/` if needed
5. Never add DB-level FKs pointing to another service's DB
