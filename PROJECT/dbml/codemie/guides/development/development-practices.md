# Development Practices Guide

**Source**: structure.md
**Stack**: Python + uv + FastAPI + Pydantic + SQLAlchemy
**Workspace**: uv monorepo — each service is a member package under `services/`

---

## Monorepo Layout

```
practice/                          # repo root
├── pyproject.toml                 # uv workspace root
├── README.md
├── services/
│   ├── user-service/
│   ├── content-service/
│   ├── feed-service/
│   └── notification-service/
├── packages/
│   └── common/                    # shared constants/event types ONLY — no "god library"
└── infra/                         # docker-compose, nginx config, env templates
```

**Rule**: `packages/common/` stays minimal. Avoid sharing business logic.

---

## Standard Service Layout

Every service follows the **same directory shape**:

```
services/<service-name>/
├── pyproject.toml          # declares service as uv workspace member
├── README.md               # how to run migrations and dev server
├── src/
│   └── <service_pkg>/      # e.g., user_service, content_service
│       ├── main.py         # FastAPI app, register routers, lifespan
│       ├── core/
│       │   └── config.py   # typed env config via pydantic-settings
│       ├── api/
│       │   ├── deps.py     # dependency injection: get_db, get_current_user
│       │   └── v1/         # routers — one file per resource
│       ├── schemas/        # Pydantic request/response models
│       ├── models/         # SQLAlchemy ORM models
│       ├── db/
│       │   └── session.py  # engine, AsyncSession factory, Base
│       └── services/       # business logic
├── alembic/                # migrations — this service only
├── alembic.ini
└── tests/                  # pytest + httpx AsyncClient
```

---

## Layer Rules

| Layer | File Location | Responsibility | Must NOT |
|-------|---------------|----------------|----------|
| **Router** | `api/v1/*.py` | Parse request, call service, return schema | Contain business logic, query DB directly |
| **Service** | `services/*.py` | Business logic, DB queries, Redis calls, events | Call other service's HTTP endpoints (except batch) |
| **Model** | `models/*.py` | SQLAlchemy table definition | Contain methods with business logic |
| **Schema** | `schemas/*.py` | Pydantic request/response validation | Access DB |
| **Config** | `core/config.py` | Typed env config (pydantic-settings) | Be instantiated multiple times |
| **Deps** | `api/deps.py` | DI: DB session, user context from headers | Contain business logic |

---

## main.py Pattern

```python
# src/<pkg>/main.py
from fastapi import FastAPI
from contextlib import asynccontextmanager
from .db.session import create_db_engine
from .api.v1 import auth, profile, posts  # import routers

@asynccontextmanager
async def lifespan(app: FastAPI):
    # startup: init DB engine, Redis connection pool
    yield
    # shutdown: close connections

app = FastAPI(lifespan=lifespan)
app.include_router(auth.router, prefix="/api/v1/users/auth", tags=["auth"])
app.include_router(posts.router, prefix="/api/v1/posts", tags=["posts"])
```

---

## Config Pattern (pydantic-settings)

```python
# src/<pkg>/core/config.py
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    database_url: str
    redis_url: str
    jwt_secret: str
    content_service_url: str = ""  # feed-service only

    class Config:
        env_file = ".env"

settings = Settings()  # singleton — import this, don't re-instantiate
```

---

## Dependency Injection Pattern

```python
# src/<pkg>/api/deps.py
from fastapi import Header, Depends
from sqlalchemy.ext.asyncio import AsyncSession
from .db.session import async_session_factory

async def get_db() -> AsyncSession:
    async with async_session_factory() as session:
        yield session

def get_current_user(
    x_user_id: int = Header(...),
    x_user_name: str = Header(...),
    x_user_avatar: str = Header(None),
    x_user_role: str = Header(...),
):
    return {"id": x_user_id, "uname": x_user_name, "avatar": x_user_avatar, "role": x_user_role}
```

---

## Router Pattern (thin)

```python
# src/<pkg>/api/v1/posts.py
from fastapi import APIRouter, Depends
from ..deps import get_db, get_current_user
from ..services.post_service import PostService
from ..schemas.post import PostCreate, PostResponse

router = APIRouter()

@router.post("/", response_model=PostResponse, status_code=201)
async def create_post(
    payload: PostCreate,
    db = Depends(get_db),
    user = Depends(get_current_user),
):
    return await PostService(db).create(payload, user)
```

---

## Service-Specific Additions

| Service | Additional Directories |
|---------|----------------------|
| user-service | `api/v1/internal.py` — JWT validate endpoint (no gateway exposure) |
| content-service | `ws/` — WebSocket handlers, room registry (`post:{id}`, `post:{id}/thread:{cid}`), broadcast |
| feed-service | `workers/` — Redis consumer (`feed_svc` group), `clients/content.py` for batch HTTP call |
| notification-service | `workers/` — Redis consumer (`notif_svc` group), `ws/` for live notification delivery |

---

## Environment Variables (per service)

```env
# Common
DATABASE_URL=postgresql+asyncpg://user:pass@localhost:5432/db
SECRET_KEY=...

# content-service only
REDIS_URL=redis://localhost:6379

# feed-service only
CONTENT_SERVICE_URL=http://localhost:8002
REDIS_URL=redis://localhost:6379

# notification-service only
REDIS_URL=redis://localhost:6379
```

---

## Running Services (uv workspace)

```bash
# From service directory
uv run uvicorn src.<pkg>.main:app --reload --port 8001

# Migrations
uv run alembic upgrade head

# Tests
uv run pytest tests/

# Install deps for entire workspace
uv sync
```

---

## Implementation Order

Follow this order to respect dependencies:

1. **user-service** — auth + profile (no upstream dependencies)
2. **content-service** — posts/comments (depends on user JWT headers)
3. **feed-service** — tags/feed (consumes content.events, calls content batch API)
4. **notification-service** — notifications (consumes content.events)

---

## Code Quality Rules

| Rule | Detail |
|------|--------|
| Async everywhere | Use `async/await` for all DB and HTTP calls (asyncpg, httpx) |
| Thin routers | No logic beyond request parsing and service delegation |
| Explicit schemas | Separate request schema from response schema (never expose ORM model directly) |
| Config singleton | Import `settings` object; never call `Settings()` more than once |
| No cross-DB queries | Never JOIN across service databases |
| No shared Alembic | Each service runs its own migrations against its own DB |
