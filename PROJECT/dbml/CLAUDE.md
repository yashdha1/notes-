# CLAUDE.md

**Purpose**: AI-optimized execution guide for Claude Code agents working with the Social Media Microservice Backend

---

## CRITICAL RULES (Check Every Task)

| Rule | Trigger | Action |
|------|---------|--------|
| **Check Guides First** | ANY task/prompt | ALWAYS check relevant guides BEFORE searching docs |
| **Testing** | User says "test", "write tests", "run tests" | ONLY then work on tests |
| **Git Ops** | User says "commit", "push", "PR", "branch" | ONLY then do git operations |
| **Shell** | ANY shell command | ONLY bash/Linux syntax |
| **Service isolation** | ANY DB or migration task | Each service owns its own DB and Alembic history |

**Recovery**: If stuck → Check [Troubleshooting](#troubleshooting)

---

## GUIDE IMPORTS

| Category | Guide Path | Purpose |
|----------|------------|---------|
| Architecture | .codemie/guides/architecture/architecture.md | Microservice layout, data flow, Nginx routing, boundaries |
| API Development | .codemie/guides/api/api-patterns.md | REST endpoints, gateway headers, WebSocket routes, pagination |
| Data & Database | .codemie/guides/data/database-patterns.md | DBML schemas, denormalization, FTS, composite PKs, migrations |
| Development Practices | .codemie/guides/development/development-practices.md | Service layout, FastAPI patterns, layers, uv commands |
| Integrations | .codemie/guides/integrations/redis-streams.md | Redis Stream event contract, consumer groups, suppression |
| Workflows | .codemie/guides/workflows/feed-workflow.md | Feed algorithm, notification flow, WS room lifecycle |
| Security | .codemie/guides/security/security-patterns.md | JWT auth, roles, password reset, input validation |

---

## TASK CLASSIFIER

**Analyze request intent → Match category → Load appropriate guides**

| Category | User Intent / Purpose | Example Requests | P0 Guide | P1 Guide |
|----------|----------------------|------------------|----------|----------|
| **Architecture** | System structure, where things belong, design decisions | "Where should I put X?", "How do services connect?" | .codemie/guides/architecture/architecture.md | - |
| **API Development** | Endpoints, routing, request/response, WebSocket | "Add endpoint for...", "How does the feed API work?" | .codemie/guides/api/api-patterns.md | .codemie/guides/security/security-patterns.md |
| **Data & Database** | Schemas, models, migrations, queries, indexes | "Add new table", "How is FTS implemented?", "Schema for X" | .codemie/guides/data/database-patterns.md | .codemie/guides/architecture/architecture.md |
| **Development Practices** | Code structure, service layout, FastAPI patterns | "How should I structure the service?", "Show layer pattern" | .codemie/guides/development/development-practices.md | .codemie/guides/architecture/architecture.md |
| **Integrations** | Redis events, consumer workers, WS broadcasting | "Add Redis event", "How does feed consume events?" | .codemie/guides/integrations/redis-streams.md | .codemie/guides/workflows/feed-workflow.md |
| **Workflows** | Feed algorithm, notification logic, business flows | "How is the feed built?", "When is notification sent?" | .codemie/guides/workflows/feed-workflow.md | .codemie/guides/integrations/redis-streams.md |
| **Security** | Auth, roles, JWT, password reset, access control | "How does auth work?", "Add role check", "Password reset flow" | .codemie/guides/security/security-patterns.md | .codemie/guides/api/api-patterns.md |

### Intent Detection

```
USER REQUEST
    ├─> What is the PRIMARY deliverable? → Primary Category (load P0)
    ├─> What standards must be followed? → Secondary Categories (load P0s)
    └─> How many services affected? → Complexity
```

### Complexity Guide

| Level | Indicators | Action |
|-------|------------|--------|
| **Simple** | Single endpoint or schema | P0 guide only |
| **Medium** | 1 service, multiple layers | Load P0 guides for affected categories |
| **High** | Cross-service, Redis events involved | Load all P0+P1 guides, check architecture guide first |

---

## EXECUTION WORKFLOW

```
START
  ├─> STEP 1: Parse Request
  │   └─ Match intent to Category → Assess complexity
  │
  ├─> STEP 2: Load Guides
  │   └─ Load P0 guides → Confidence < 80%? → Load P1 or ask user
  │
  ├─> STEP 3: Execute
  │   └─ Apply patterns from guides → Follow Critical Rules
  │
  └─> STEP 4: Validate & Deliver
      └─ Run checklist → All pass? → Deliver
```

### Pre-Delivery Checklist

- [ ] Meets user's request requirements?
- [ ] Follows patterns from loaded guides?
- [ ] Service isolation respected (no cross-DB queries, no shared Alembic)?
- [ ] Side effects declared explicitly (Redis events, WS broadcasts)?
- [ ] No hardcoded secrets or credentials?

---

## COMMANDS

| Task | Command | Notes |
|------|---------|-------|
| **Setup** | `uv sync` | Install all workspace dependencies |
| **Run service** | `uv run uvicorn src.<pkg>.main:app --reload --port <port>` | Run from service directory |
| **Migrate** | `uv run alembic upgrade head` | From inside service directory |
| **New migration** | `uv run alembic revision --autogenerate -m "description"` | From inside service directory |
| **Test** ⚠️ | `uv run pytest tests/` | ONLY when user requests tests |

**Service ports**: user-service :8001 · content-service :8002 · feed-service :8003 · notification-service :8004

---

## PROJECT CONTEXT

### Technology Stack

| Component | Technology |
|-----------|------------|
| Language | Python (3.12+) |
| Framework | FastAPI (ASGI) |
| Package Manager | uv (workspace monorepo) |
| Database | PostgreSQL (one per service) |
| ORM | SQLAlchemy (async) |
| Migrations | Alembic (per service) |
| Schemas | Pydantic v2 |
| Async messaging | Redis Streams |
| Real-time | WebSockets (FastAPI native) |
| Gateway | Nginx (JWT validation + routing) |

### Project Structure

```
practice/
├── pyproject.toml                  # uv workspace root
├── services/
│   ├── user-service/               # Auth, profiles (port 8001)
│   ├── content-service/            # Posts, comments, WS (port 8002)
│   ├── feed-service/               # Feed, tags, weights (port 8003)
│   └── notification-service/       # Notifications, WS (port 8004)
├── packages/common/                # Shared constants/event types only
├── infra/                          # docker-compose, nginx, env templates
├── .codemie/guides/                # Architecture and pattern guides
└── *.md                            # Design documents (source of truth)
```

### Key Design Documents

| File | Contents |
|------|----------|
| design.md | DBML schemas for all 4 services |
| api.md | Full REST + WebSocket endpoint specification |
| structure.md | Monorepo layout, service structure, tech stack |
| feed_algorithm.md | Feed build algorithm (Redis cache + tag weights) |
| notification_plan.md | Notification delivery flow |
| stories.md | User stories with acceptance criteria |

### Key Integrations

| Integration | Purpose | Guide |
|-------------|---------|-------|
| Redis Streams (`content.events`) | Async event bus between services | .codemie/guides/integrations/redis-streams.md |
| WebSockets | Live content updates + notification bell | .codemie/guides/workflows/feed-workflow.md |
| Nginx gateway | JWT validation + service routing | .codemie/guides/security/security-patterns.md |

---

## TROUBLESHOOTING

| Symptom | Cause | Solution |
|---------|-------|----------|
| Cross-service data missing | Denormalization not applied at write time | Read design.md denormalization table; copy fields from `X-User-*` headers at create |
| Duplicate notification delivered | Suppression logic missing | Check WS room membership before adding to `notify_user_ids[]`; see integrations guide |
| Feed shows seen posts | `feed_post_seen` not populated on serve | INSERT seen rows at serve time AND on `post.viewed` event |
| FTS returns no results | `search_vector` not populated | Ensure DB trigger is applied; `tag_names` field set on post create |
| Migration runs against wrong DB | Wrong `DATABASE_URL` in env | Each service has its own `.env`; never share Alembic across services |
| Auth headers missing in service | Nginx not injecting headers | Verify gateway JWT validation config; check `X-User-Id` injection |
| Consumer not receiving events | Wrong consumer group name | `feed_svc` for feed-service, `notif_svc` for notification-service |

---

## REMEMBER

### Workflow
1. **Parse** → Match intent to Category (Task Classifier)
2. **Load** → Read P0 guides for matched categories
3. **Check** → Confidence ≥ 80%? No → load P1 or ask user
4. **Execute** → Apply patterns from guides
5. **Validate** → Checklist must pass
6. **Deliver**

### Key Invariants
- One DB per service — never cross-DB queries
- Content Service is the only Redis Stream producer
- Always read design docs (*.md) as the source of truth for schemas and APIs
- Denormalize at write time using `X-User-*` headers; never call user-service at read time
- Suppression = check WS room before adding to `notify_user_ids[]`

### When to Ask User
- Ambiguous service boundary for a new feature
- New event type needed that crosses multiple services
- Unsure if a field should be denormalized or fetched live
