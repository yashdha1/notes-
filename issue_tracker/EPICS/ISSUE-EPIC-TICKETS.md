# 🧱 EPIC: Issue Management System

> **Epic Category:** Backend + Frontend  
> **Tables:** `issue`, `issue_labels`  
> **Stack:** FastAPI · SQLAlchemy · PostgreSQL · Elasticsearch · Redis Pub/Sub · React + TypeScript  
> **Total Tickets:** 18  

---

## Ticket Index : 

| #   | Title                                        | Type  | Layer    | Dependencies         |
| --- | -------------------------------------------- | ----- | -------- | -------------------- |
| 1   | Issue SQLAlchemy Model & Migration           | Task  | Backend  | Project model exists |
| 2   | Issue Labels Model & Migration               | Task  | Backend  | Ticket 1             |
| 3   | Issue Domain Entity & Enums                  | Task  | Backend  | —                    |
| 4   | Issue Pydantic Schemas (Request/Response)    | Task  | Backend  | Ticket 1, 3          |
| 5   | Issue CRUD API Endpoints                     | Story | Backend  | Ticket 1, 4          |
| 6   | Issue Labels CRUD API                        | Task  | Backend  | Ticket 2, 5          |
| 7   | Issue Filtering (status, priority, assignee) | Story | Backend  | Ticket 5             |
| 8   | Issue Pagination                             | Task  | Backend  | Ticket 5             |
| 9   | Issue Assignment Logic                       | Story | Backend  | Ticket 5             |
| 10  | Issue Status Workflow                        | Story | Backend  | Ticket 5             |
| 13  | Issue Soft Delete                            | Task  | Backend  | Ticket 5             |
| 15  | Issue Event Publishing (Redis Pub/Sub)       | Task  | Backend  | Ticket 5, Redis      |
| 16  | Issue List Page + Filter Bar (Frontend)      | Story | Frontend | Ticket 5, 7          |
| 17  | Create Issue Form (Frontend)                 | Story | Frontend | Ticket 5, 6          |
| 18  | Issue Detail Page (Frontend)                 | Story | Frontend | Ticket 5, 11         |
| 19  | Issue API Tests (Backend)                    | Test  | Backend  | Ticket 5–15          |
|     |                                              |       |          |                      |
|     |                                              |       |          |                      |

---

## 🟣 Ticket 1 — Issue SQLAlchemy Model & Migration

---
epic: "Issue Management System"
title: "Design and implement Issue model (SQLAlchemy + Alembic)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Define the core `issue` table as a SQLAlchemy ORM model inside the project-service, matching the DB schema. Generate and verify Alembic migration.

---

### 2. Goals & Success Metrics

| Goal                            | Success Metric                                 |
| ------------------------------- | ---------------------------------------------- |
| Model matches DB schema exactly | All columns, types, constraints match the spec |
| Migration runs cleanly          | `alembic upgrade head` succeeds on fresh DB    |
| ORM usable                      | CRUD works via Python shell / pytest fixture   |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- SQLAlchemy model `Issue` in `src/infrastructure/persistence/models/issue_model.py`
- Columns: `id` (UUID, PK, default `gen_random_uuid()`), `project_id` (Integer, FK → project.id, NOT NULL), `title` (VARCHAR, NOT NULL), `description` (Text, nullable), `type` (VARCHAR, NOT NULL), `status` (VARCHAR, NOT NULL, default `backlog`), `priority` (VARCHAR, default `none`), `reporter_id` (Integer, NOT NULL), `assignee_id` (Integer, nullable), `parent_id` (UUID, FK → issue.id, nullable), `due_date` (TIMESTAMPTZ, nullable), `is_deleted` (Boolean, default false), `created_at` (TIMESTAMPTZ, default now), `updated_at` (TIMESTAMPTZ, default now, onupdate now)
- Denormalized columns: `reporter_username` (VARCHAR 100, NOT NULL), `assignee_username` (VARCHAR 100, nullable)
- FK constraint: `project_id` → `project(id)` ON DELETE CASCADE
- Self-referential FK: `parent_id` → `issue(id)` nullable
- Alembic migration file generated and tested

#### Should Have (S)
- `attachment_url` (VARCHAR 500, nullable)
- `epic_category` (VARCHAR 50, nullable — only for issue_type = `epic`)

#### Won't Have (W)
- Full-text search columns (handled by Elasticsearch)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| DB Indexes | `idx_issue_project` on `(project_id, status)`, `idx_issue_assignee` on `(assignee_id)`, `idx_issue_priority` on `(priority)`, `idx_issue_type` on `(type)`, `idx_issue_parent` on `(parent_id) WHERE parent_id IS NOT NULL` |
| Code Quality | Model class follows Onion Architecture — lives in infrastructure/persistence layer |
| Naming | Table name: `issue`, model class: `IssueModel` |

---

### 6. Dependencies

- **Runtime:** PostgreSQL (project-db), SQLAlchemy 2.x, asyncpg
- **Dev / Build:** Alembic for migrations
- **Backend:** Project model must exist (FK reference)

---

### 8. Acceptance Criteria

- [ ] `IssueModel` class exists with all columns matching schema
- [ ] Alembic migration generated (`alembic revision --autogenerate`)
- [ ] `alembic upgrade head` runs without errors on clean DB
- [ ] Can create an Issue row via Python shell / test
- [ ] All indexes created (verify via `\d+ issue` in psql)
- [ ] Foreign keys enforced (inserting invalid `project_id` raises IntegrityError)
- [ ] Code follows project naming conventions; docstring on model class

---

### 9. Build Order

| Stage | Scope                                 |
| ----- | ------------------------------------- |
| S0    | Project model & migration must exist  |
| S1    | This ticket — Issue model + migration |

---

### 11. Out of Scope (v1)

- Comment model (separate ticket)
- Elasticsearch indexing (Ticket 14)
- API endpoints (Ticket 5)
- Notification events (Ticket 15)

---

## 🟣 Ticket 2 — Issue Labels Model & Migration

---
epic: "Issue Management System"
title: "Implement Issue Labels model (SQLAlchemy + Alembic)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Create the `issue_labels` table to support multiple labels per issue for categorization, filtering, and search.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Label mapping works | One issue can have multiple labels |
| Query support | Can filter issues by label via ORM join |
| Data integrity | Deleting an issue cascades to its labels |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- SQLAlchemy model `IssueLabelModel` in `src/infrastructure/persistence/models/issue_label_model.py`
- Columns: `id` (UUID, PK, default `gen_random_uuid()`), `issue_id` (UUID, FK → issue.id, ON DELETE CASCADE), `label` (VARCHAR, NOT NULL)
- Relationship on `IssueModel`: `labels = relationship("IssueLabelModel", back_populates="issue", cascade="all, delete-orphan")`
- Alembic migration generated and tested

#### Should Have (S)
- Unique constraint on `(issue_id, label)` — prevent duplicate labels per issue
- Index on `issue_id` for join performance

#### Could Have (C)
- Predefined label values via enum or config (e.g., `groomed`, `ready`, `blocked`)

#### Won't Have (W)
- Label color or metadata — plain string labels only in v1

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| DB | Index on `issue_id` for FK join queries |
| Code Quality | Model in infrastructure layer, relationship bidirectional |

---

### 6. Dependencies

- **Runtime:** PostgreSQL, SQLAlchemy 2.x
- **Backend:** Issue model (Ticket 1) must exist

---

### 8. Acceptance Criteria

- [ ] `IssueLabelModel` exists with all columns
- [ ] Alembic migration runs successfully
- [ ] Can add multiple labels to a single issue
- [ ] Duplicate `(issue_id, label)` pair is rejected
- [ ] Deleting an issue cascades to delete its labels
- [ ] ORM relationship works: `issue.labels` returns list of `IssueLabelModel`

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue model (Ticket 1) |
| S1 | This ticket — Issue Labels model |

---

### 11. Out of Scope (v1)

- Label CRUD API (Ticket 6)
- Label colors, icons, or metadata
- Global label management (project-wide labels)

---

## 🟣 Ticket 3 — Issue Domain Entity & Enums

---
epic: "Issue Management System"
title: "Define Issue domain entity, value objects, and enums"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Define pure domain-layer entities and enums for Issue, following Onion Architecture. These have no ORM or framework imports — purely business logic types.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Clean domain layer | No SQLAlchemy imports in domain/ |
| Type safety | Enums prevent invalid type/status/priority values |
| Reusability | Used by use-cases and schemas |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `IssueType` enum: `story`, `epic`, `task`, `bug`, `test`, `incident`
- `IssueStatus` enum: `backlog`, `open`, `in_progress`, `in_review`, `done`, `closed`
- `IssuePriority` enum: `none`, `low`, `medium`, `high`, `critical`
- Domain entity dataclass `Issue` in `src/domain/entities/issue.py` (no ORM dependency)
- Domain exceptions: `IssueNotFound`, `InvalidIssueTransition`, `InvalidParentIssue` in `src/domain/exceptions/issue.py`

#### Should Have (S)
- `EpicCategory` enum: `frontend`, `backend`, `authentication` (only for epic type)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Code Quality | Zero infrastructure imports in domain layer |
| State | Enums used consistently across schemas, models, and use-cases |

---

### 6. Dependencies

- **Runtime:** Python stdlib only (dataclasses, enum)
- **Backend:** None — this is the innermost layer

---

### 8. Acceptance Criteria

- [ ] All enums defined with correct values
- [ ] Domain entity `Issue` is a plain dataclass
- [ ] Domain exceptions defined
- [ ] No SQLAlchemy, Pydantic, or FastAPI imports in domain layer
- [ ] Enums importable from `src.domain.enums`

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | None — no dependencies |
| S1 | This ticket |

---

### 11. Out of Scope (v1)

- Domain services / business rules (handled in use-cases)
- Status transition validation logic (Ticket 10)

---

## 🟣 Ticket 4 — Issue Pydantic Schemas (Request/Response)

---
epic: "Issue Management System"
title: "Implement Pydantic v2 schemas for Issue API request/response"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Define all Pydantic v2 schemas for Issue create, update, list, and detail responses. These serve as the API contract and validation layer.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Validation | Invalid payloads rejected with clear error messages |
| API contract | Response shape matches frontend expectations |
| Type safety | Enums validated at schema level |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `IssueCreateSchema`: title (str, required, max 500), description (str, optional), type (IssueType, required), priority (IssuePriority, default `none`), assignee_id (int, optional), parent_id (UUID, optional), labels (list[str], optional), due_date (datetime, optional)
- `IssueUpdateSchema`: all fields optional (partial update support)
- `IssueResponseSchema`: all fields + id, project_id, reporter_id, reporter_username, assignee_username, labels, created_at, updated_at
- `IssueListResponseSchema`: paginated wrapper — results (list), count (int), next (str|null), previous (str|null)
- `IssueLabelSchema`: label (str)

#### Should Have (S)
- `IssueAssignSchema`: assignee_id (int, required) — dedicated schema for assignment endpoint
- `IssueStatusUpdateSchema`: status (IssueStatus, required)
- Validators: `parent_id` only valid when type = `task`; `epic_category` only valid when type = `epic`; `incident` auto-sets priority ≥ `high` if not explicitly higher

#### Could Have (C)
- `IssueBriefSchema` — lightweight schema for list view (fewer fields)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Code Quality | Pydantic v2 `model_config = ConfigDict(from_attributes=True)` for ORM mode |
| Security | Input sanitization — strip HTML from title/description |

---

### 6. Dependencies

- **Runtime:** Pydantic v2
- **Backend:** Domain enums (Ticket 3), Issue model (Ticket 1)

---

### 8. Acceptance Criteria

- [ ] All schemas defined in `src/api/v1/schemas/issue_schema.py`
- [ ] Creating an issue with missing `title` returns 422 with clear message
- [ ] Enum fields reject invalid values
- [ ] `parent_id` validation: rejected if type ≠ `task`
- [ ] `IssueResponseSchema` includes nested labels list
- [ ] All schemas have docstrings

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Domain enums (Ticket 3), Issue model (Ticket 1) |
| S1 | This ticket — Pydantic schemas |

---

### 11. Out of Scope (v1)

- Comment schemas (separate epic)
- File upload / attachment validation
- Bulk operation schemas

---

## 🟣 Ticket 5 — Issue CRUD API Endpoints

---
epic: "Issue Management System"
title: "Implement Issue CRUD API endpoints (FastAPI)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Provide full REST API for creating, reading, updating, and deleting issues within a project. This is the core ticket of the epic — all other Issue tickets depend on it.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| All CRUD operations work | POST, GET, GET/{id}, PUT, PATCH, DELETE all return correct responses |
| Proper validation | Invalid payloads return 422; not-found returns 404 |
| Role-based access | Permissions enforced per LLD role matrix |
| Clean architecture | Router → Use-case → Repository → DB (no business logic in router) |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `POST /api/v1/projects/{project_id}/tickets` — Create issue (JWT required); sets reporter_id from token; creates labels if provided; returns 201
- `GET /api/v1/projects/{project_id}/tickets` — List issues for project (JWT required); returns paginated list; excludes soft-deleted
- `GET /api/v1/tickets/{ticket_id}` — Get single issue with labels (JWT required); returns 404 if not found or deleted
- `PUT /api/v1/tickets/{ticket_id}` — Full update (reporter/assignee/lead/admin)
- `PATCH /api/v1/tickets/{ticket_id}` — Partial update (status change, reassign, etc.)
- `DELETE /api/v1/tickets/{ticket_id}` — Soft-delete (reporter/admin only); sets `is_deleted=true`

#### Should Have (S)
- Denormalize `reporter_username` and `assignee_username` at write time from JWT claims
- Return `labels` as list of strings in response (not raw label objects)
- `GET /api/v1/tickets/{ticket_id}/subtasks` — List sub-tasks where `parent_id = ticket_id`

#### Could Have (C)
- Bulk status update endpoint

#### Won't Have (W)
- File attachment upload (future ticket)
- Webhook triggers (future)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | List endpoint < 200ms for 100 issues (indexed queries) |
| Security | JWT required on all endpoints; role checks enforced |
| Code Quality | Onion Architecture: router → use-case → repository → DB |
| Routing | All routes registered under `api/v1` prefix |

---

### 6. Dependencies

- **Runtime:** FastAPI, SQLAlchemy 2.x (async), asyncpg
- **Backend:** Issue model (Ticket 1), Labels model (Ticket 2), Pydantic schemas (Ticket 4), auth dependency (JWT decode)
- **Functional:** Project must exist (validated before issue creation)

---

### 8. Acceptance Criteria

- [ ] `POST /api/v1/projects/{project_id}/tickets` creates issue and returns 201 with full response
- [ ] `GET /api/v1/projects/{project_id}/tickets` returns paginated list
- [ ] `GET /api/v1/tickets/{ticket_id}` returns single issue with labels
- [ ] `PUT /api/v1/tickets/{ticket_id}` updates all fields
- [ ] `PATCH /api/v1/tickets/{ticket_id}` updates partial fields
- [ ] `DELETE /api/v1/tickets/{ticket_id}` soft-deletes (sets is_deleted=true, returns 204)
- [ ] 404 returned for non-existent or deleted issues
- [ ] 403 returned for unauthorized updates/deletes
- [ ] 422 returned for invalid payloads
- [ ] All endpoints tested via Swagger / httpie / pytest
- [ ] Functions follow naming conventions; docstrings present

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue model (T1), Labels model (T2), Enums (T3), Schemas (T4) |
| S1 | Repository layer — `IssueRepository` (async CRUD queries) |
| S2 | Use-case layer — `CreateIssue`, `ListIssues`, `GetIssue`, `UpdateIssue`, `DeleteIssue` |
| S3 | Router layer — wire endpoints to use-cases via FastAPI dependency injection |

---

### 11. Out of Scope (v1)

- Filtering (Ticket 7)
- Pagination logic detail (Ticket 8)
- Elasticsearch indexing (Ticket 14)
- Event publishing (Ticket 15)
- Comment endpoints (separate epic)

---

## 🟣 Ticket 6 — Issue Labels CRUD API

---
epic: "Issue Management System"
title: "Implement Issue Labels CRUD API"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Allow adding, listing, and removing labels on issues through dedicated API endpoints or as part of issue create/update payloads.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Labels manageable via API | Can add/remove labels on existing issues |
| Integrated with issue CRUD | Labels sent on create/update are persisted |
| No duplicates | Same label on same issue is rejected or idempotent |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- Labels passed in `IssueCreateSchema.labels` (list[str]) are inserted into `issue_labels` table on issue creation
- Labels passed in `IssueUpdateSchema.labels` replace existing labels (full replace strategy)
- `GET /api/v1/tickets/{ticket_id}` response includes `labels: ["groomed", "blocked"]` as flat string list

#### Should Have (S)
- `POST /api/v1/tickets/{ticket_id}/labels` — Add a label to an issue (idempotent)
- `DELETE /api/v1/tickets/{ticket_id}/labels/{label}` — Remove a specific label
- `GET /api/v1/tickets/{ticket_id}/labels` — List all labels for an issue

#### Could Have (C)
- `GET /api/v1/projects/{project_id}/labels` — List all unique labels used in a project (for autocomplete)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| DB | Unique constraint on (issue_id, label) prevents duplicates |
| Performance | Labels loaded eagerly with issue (joinedload or subqueryload) |

---

### 6. Dependencies

- **Backend:** Issue model (Ticket 1), Labels model (Ticket 2), Issue CRUD (Ticket 5)

---

### 8. Acceptance Criteria

- [ ] Creating issue with `labels: ["groomed", "ready"]` persists both labels
- [ ] Updating issue with `labels: ["blocked"]` replaces all previous labels
- [ ] GET issue response includes `labels` as flat string list
- [ ] Duplicate label on same issue is handled gracefully (no 500)
- [ ] Deleting issue cascades to delete its labels
- [ ] Label endpoints return correct HTTP status codes

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5), Labels model (Ticket 2) |
| S1 | Integrate label handling into issue create/update use-cases |
| S2 | (Optional) Dedicated label endpoints |

---

### 11. Out of Scope (v1)

- Label colors, descriptions, or metadata
- Project-wide label management / presets
- Label-based access control

---

## 🟣 Ticket 7 — Issue Filtering (status, priority, assignee)

---
epic: "Issue Management System"
title: "Implement issue filtering via query parameters"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Allow users to filter the issue list by status, priority, assignee, and issue type using query parameters. Combined filters must be supported.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Filter accuracy | Returned issues match all applied filters |
| Combined filters | Multiple filters AND together correctly |
| Performance | Filtered queries use DB indexes |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `GET /api/v1/projects/{project_id}/tickets?status=open` — filter by status
- `GET /api/v1/projects/{project_id}/tickets?priority=high` — filter by priority
- `GET /api/v1/projects/{project_id}/tickets?assignee_id=5` — filter by assignee
- Combined: `?status=open&priority=high&assignee_id=5` — all filters AND together
- Exclude soft-deleted issues (`is_deleted=false`) always

#### Should Have (S)
- `?issue_type=bug` — filter by issue type
- `?labels=groomed` — filter by label (JOIN on issue_labels)
- `?reporter_id=3` — filter by reporter

#### Could Have (C)
- `?due_before=2026-05-01` — filter by due date range
- `?created_after=2026-04-01` — filter by creation date

#### Won't Have (W)
- Full-text search via query params (that's Ticket 14 — Elasticsearch)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | All filter columns are indexed (see Ticket 1 index list) |
| Security | `assignee_id` / `reporter_id` are integers — validate input type |

---

### 6. Dependencies

- **Backend:** Issue CRUD list endpoint (Ticket 5), DB indexes (Ticket 1)
- **Functional:** Labels model (Ticket 2) for label filtering

---

### 8. Acceptance Criteria

- [ ] `?status=open` returns only issues with status `open`
- [ ] `?priority=high` returns only high-priority issues
- [ ] `?assignee_id=5` returns only issues assigned to user 5
- [ ] `?status=open&priority=high` returns issues matching BOTH filters
- [ ] Invalid filter values return 422 (e.g., `?status=invalid`)
- [ ] Soft-deleted issues never appear in filtered results
- [ ] Query uses indexes (verify via EXPLAIN ANALYZE)

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD list endpoint (Ticket 5) |
| S1 | Add filter params to list use-case and repository query builder |

---

### 11. Out of Scope (v1)

- Elasticsearch-based search (Ticket 14)
- Saved filters / user filter presets
- OR-based filter logic

---

## 🟣 Ticket 8 — Issue Pagination

---
epic: "Issue Management System"
title: "Implement cursor/offset pagination for issue list"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Support paginated issue listing to handle large projects efficiently. Provide consistent pagination metadata in response.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Pagination works | Large datasets return correct pages |
| Metadata correct | `count`, `next`, `previous` fields accurate |
| Composable with filters | Pagination + filters work together |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- Query params: `?page=1&page_size=20` (offset-based pagination)
- Default page size: 20, max page size: 100
- Response shape: `{ results: [...], count: 150, next: "/api/v1/.../tickets?page=2", previous: null }`
- Works with all filter combinations (Ticket 7)

#### Should Have (S)
- Sorting: `?sort_by=created_at&order=desc` (default: newest first)
- Sort options: `created_at`, `updated_at`, `priority`, `due_date`

#### Could Have (C)
- Cursor-based pagination for real-time consistency

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Use `COUNT(*)` over filtered query for total; LIMIT/OFFSET with indexed ORDER BY |

---

### 6. Dependencies

- **Backend:** Issue CRUD list endpoint (Ticket 5)

---

### 8. Acceptance Criteria

- [ ] `?page=1&page_size=10` returns first 10 issues
- [ ] `?page=2&page_size=10` returns next 10 issues
- [ ] `count` reflects total matching issues (not page count)
- [ ] `next` / `previous` URLs are correct or null at boundaries
- [ ] `page_size > 100` is clamped or returns 422
- [ ] Pagination + filters work together correctly

---

### 11. Out of Scope (v1)

- Infinite scroll support (frontend concern)
- Cursor-based pagination (offset is sufficient for MVP)

---

## 🟣 Ticket 9 — Issue Assignment Logic

---
epic: "Issue Management System"
title: "Implement issue assignment and reassignment"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Allow assigning and reassigning issues to users. Denormalize the assignee username at write time. Trigger notification event on assignment change.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Assignment works | Assignee saved correctly on create and update |
| Denormalization | `assignee_username` populated from auth data |
| Event published | `TicketAssigned` event fires on assignment change |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `assignee_id` set during issue creation (optional field)
- `PATCH /api/v1/tickets/{ticket_id}` with `{ assignee_id: 5 }` — reassign issue
- Denormalize `assignee_username` at write time (from JWT claims or user lookup)
- Validate `assignee_id` is a valid user (via auth-service user list or cached user data)
- Permissions: member can assign self, lead/admin can assign anyone

#### Should Have (S)
- Unassign: `PATCH` with `{ assignee_id: null }` clears assignment
- Publish `TicketAssigned` event to Redis Pub/Sub on assignment change (consumed by notification-service)

#### Could Have (C)
- `GET /api/v1/users` proxy endpoint in project-service for assignee picker dropdown

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Security | Members can only self-assign; leads/admins can assign anyone |
| Performance | Index on `assignee_id` for filtered queries |

---

### 6. Dependencies

- **Backend:** Issue CRUD (Ticket 5), Auth service (user validation)
- **Functional:** Redis Pub/Sub setup (Ticket 15) for event publishing

---

### 8. Acceptance Criteria

- [ ] Issue can be created with `assignee_id`
- [ ] PATCH updates `assignee_id` and `assignee_username`
- [ ] Setting `assignee_id: null` unassigns the issue
- [ ] Invalid `assignee_id` returns 400/404
- [ ] Member cannot assign to someone else (returns 403)
- [ ] `TicketAssigned` event published to Redis on reassignment
- [ ] API response includes `assignee_id` and `assignee_username`

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5) |
| S1 | Assignment logic in update use-case |
| S2 | Event publishing (after Ticket 15) |

---

### 11. Out of Scope (v1)

- Auto-assignment / load balancing
- Assignment history / audit log
- Watchers / CC list

---

## 🟣 Ticket 10 — Issue Status Workflow

---
epic: "Issue Management System"
title: "Implement issue status workflow with transition validation"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Track issue lifecycle from creation to completion. Enforce valid status transitions to prevent invalid state changes.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Status updates work | PATCH changes status correctly |
| Transitions enforced | Invalid transitions return 400 |
| Event published | `TicketStatusChanged` event fires |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- Statuses: `backlog` → `open` → `in_progress` → `in_review` → `done` → `closed`
- Default status on creation: `backlog`
- `PATCH /api/v1/tickets/{ticket_id}` with `{ status: "in_progress" }` — update status
- Basic transition validation:
  - `backlog` → `open`
  - `open` → `in_progress`
  - `in_progress` → `in_review` | `open` (back to open if blocked)
  - `in_review` → `done` | `in_progress` (back if changes needed)
  - `done` → `closed`
  - `closed` → `open` (reopen)
- Permissions: assigned user or lead/admin can change status

#### Should Have (S)
- Publish `TicketStatusChanged` event to Redis Pub/Sub
- Store `updated_at` timestamp on every status change

#### Could Have (C)
- Status change comment (auto-generated: "Status changed from X to Y by User")

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Code Quality | Transition map defined in domain layer (not hardcoded in router) |
| State | Status transitions are deterministic and documented |

---

### 6. Dependencies

- **Backend:** Issue CRUD (Ticket 5), Domain enums (Ticket 3)
- **Functional:** Redis Pub/Sub (Ticket 15) for event publishing

---

### 8. Acceptance Criteria

- [ ] New issue defaults to `backlog` status
- [ ] Valid transitions succeed (e.g., `open` → `in_progress`)
- [ ] Invalid transitions return 400 with message (e.g., `backlog` → `done`)
- [ ] Only assigned user / lead / admin can change status
- [ ] `TicketStatusChanged` event published on status change
- [ ] `updated_at` updated on status change
- [ ] Transition map is defined in domain layer, not in router

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5), Domain enums (Ticket 3) |
| S1 | Define transition map in domain layer |
| S2 | Integrate validation into update use-case |

---

### 11. Out of Scope (v1)

- Custom workflows per project
- SLA timers on status transitions
- Status change history / audit log

---

## 🟣 Ticket 11 — Parent-Child Issue Linking

---
epic: "Issue Management System"
title: "Implement parent-child issue relationship (epic → task)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Support hierarchical linking of issues — an epic can have tasks, a task can be a sub-task of another task. Enable fetching child issues for a given parent.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Parent-child linkage works | `parent_id` saved and queryable |
| Hierarchy fetched | API returns children for a parent issue |
| Constraint enforced | Only `task` type can have a `parent_id` |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `parent_id` (UUID, nullable, FK → issue.id) already on model (Ticket 1)
- On issue creation: if `parent_id` is provided, validate parent exists and is not deleted
- Constraint: `parent_id` only valid when `issue_type = 'task'` — reject otherwise with 400
- `GET /api/v1/tickets/{ticket_id}/subtasks` — returns list of child issues where `parent_id = ticket_id`
- `GET /api/v1/tickets/{ticket_id}` response includes `parent_id` field

#### Should Have (S)
- Response includes `subtask_count` on parent issue detail
- Prevent circular references: a ticket cannot be its own parent
- Prevent deep nesting: only 1 level (task → subtask, no subtask-of-subtask)

#### Could Have (C)
- Bulk move subtasks to a different parent

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| DB | Partial index `idx_issue_parent ON (parent_id) WHERE parent_id IS NOT NULL` |
| Code Quality | Validation in use-case layer, not in router |

---

### 6. Dependencies

- **Backend:** Issue model (Ticket 1) — `parent_id` column, Issue CRUD (Ticket 5)

---

### 8. Acceptance Criteria

- [ ] Issue created with `parent_id` links to parent correctly
- [ ] `parent_id` rejected if issue type ≠ `task` (returns 400)
- [ ] `parent_id` referencing non-existent issue returns 404
- [ ] Self-referencing `parent_id` rejected
- [ ] `GET /api/v1/tickets/{id}/subtasks` returns correct children
- [ ] Subtasks count included in parent detail response
- [ ] Deleting parent does NOT cascade-delete children (children become orphaned or blocked)

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5) |
| S1 | Add parent validation to create/update use-cases |
| S2 | Add subtasks endpoint |

---

### 11. Out of Scope (v1)

- Multi-level nesting (grandchild issues)
- Dependency links (blocked-by, relates-to)
- Epic progress calculation from child statuses

---

## 🟣 Ticket 12 — Issue Due Date Support

---
epic: "Issue Management System"
title: "Add due date support for issues"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Track deadlines for issues. Allow setting and filtering by due date to surface overdue or upcoming work.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Due date stored | `due_date` persisted and returned in API |
| Filtering works | Can filter by due date range |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `due_date` (TIMESTAMPTZ, nullable) already on model (Ticket 1)
- Settable on create and update via `IssueCreateSchema` / `IssueUpdateSchema`
- Returned in `IssueResponseSchema`
- Clear due date: `PATCH` with `{ due_date: null }`

#### Should Have (S)
- Filter: `?due_before=2026-05-01` — issues due before date
- Filter: `?due_after=2026-04-20` — issues due after date
- Sort: `?sort_by=due_date&order=asc` — sort by due date

#### Could Have (C)
- `?overdue=true` — filter for issues past due date and not `done`/`closed`
- Visual indicator in API response: `is_overdue: true/false` computed field

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| DB | Index on `due_date` if filtering is implemented |
| Code Quality | Timezone-aware datetime handling (TIMESTAMPTZ) |

---

### 6. Dependencies

- **Backend:** Issue model (Ticket 1), Issue CRUD (Ticket 5), Filtering (Ticket 7)

---

### 8. Acceptance Criteria

- [ ] `due_date` can be set on issue create
- [ ] `due_date` can be updated via PATCH
- [ ] `due_date: null` clears the due date
- [ ] `due_date` returned in API response (ISO 8601 format)
- [ ] Due date filters work correctly
- [ ] Timezone handled correctly (stored as TIMESTAMPTZ)

---

### 11. Out of Scope (v1)

- Due date reminders / notifications
- SLA enforcement
- Calendar view integration

---

## 🟣 Ticket 13 — Issue Soft Delete

---
epic: "Issue Management System"
title: "Implement soft delete for issues"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Issues should never be hard-deleted. Soft-delete preserves audit trail and enables future restore/moderation features.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Soft delete works | `is_deleted=true` set on delete |
| Filtered out | Soft-deleted issues excluded from all list/search queries |
| Reversible | Admins can restore deleted issues |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `DELETE /api/v1/tickets/{ticket_id}` sets `is_deleted = true` (does NOT delete row)
- All list queries filter `WHERE is_deleted = false` by default
- `GET /api/v1/tickets/{ticket_id}` returns 404 for soft-deleted issues
- Permissions: reporter can delete own; admin can delete any

#### Should Have (S)
- `PATCH /api/v1/tickets/{ticket_id}/restore` — admin-only restore (sets `is_deleted = false`)
- Soft-deleted issues still accessible via admin endpoint (for moderation)

#### Could Have (C)
- Hard-delete after 30 days (background job)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Security | Only reporter or admin can soft-delete |
| DB | Partial index on `is_deleted = false` for query performance |

---

### 6. Dependencies

- **Backend:** Issue model (Ticket 1 — `is_deleted` column), Issue CRUD (Ticket 5)

---

### 8. Acceptance Criteria

- [ ] DELETE endpoint sets `is_deleted = true`, returns 204
- [ ] Soft-deleted issues do NOT appear in list queries
- [ ] Soft-deleted issues return 404 on direct GET
- [ ] Non-authorized delete returns 403
- [ ] (Should) Admin can restore a soft-deleted issue
- [ ] Labels of soft-deleted issues remain in DB (not cascade-deleted)

---

### 11. Out of Scope (v1)

- Hard-delete / purge
- Trash view for users
- Undo-delete grace period in UI

---

## 🟣 Ticket 14 — Issue Elasticsearch Indexing & Search

---
epic: "Issue Management System"
title: "Implement Elasticsearch indexing and search for issues"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Enable fast, relevance-ranked full-text search across issues using Elasticsearch (Amazon OpenSearch). Index issues on create/update; query via dedicated search endpoint.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Indexing works | Issues indexed in ES on create/update |
| Search returns results | `?q=login bug` returns relevant issues |
| Label search works | `?labels=groomed` filters in ES |
| Relevance ranking | Title matches rank higher than description |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- ES index: `tickets` with mapping: `ticket_id` (integer), `project_id` (integer), `title` (text, analyzed), `description` (text, analyzed), `status` (keyword), `priority` (keyword), `issue_type` (keyword), `labels` (keyword[]), `assignee_id` (integer), `reporter_username` (keyword), `created_at` (date)
- On issue create: index document in ES (async, non-blocking — do not fail the create if ES is down)
- On issue update: re-index document in ES
- On issue soft-delete: remove document from ES index
- `GET /api/v1/search?q=login+bug` — full-text search using `multi_match` on `title^3 + description` with `fuzziness: AUTO`
- Filter in ES: `?q=bug&status=open&labels=groomed&priority=high`
- Search returns paginated results with relevance score

#### Should Have (S)
- `elasticsearch_client.py` as infrastructure adapter implementing `SearchPort` interface
- Graceful degradation: if ES is unavailable, log error and skip (don't crash the service)

#### Could Have (C)
- Autocomplete / suggest endpoint for title search
- Highlight matching terms in search results

#### Won't Have (W)
- PostgreSQL `tsvector` / `tsquery` — ES is the only search path

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | Search response < 300ms for typical queries |
| Reliability | ES failure does not block issue CRUD operations |
| Code Quality | ES client in infrastructure layer, behind port interface |

---

### 6. Dependencies

- **Runtime:** Elasticsearch / Amazon OpenSearch cluster
- **Backend:** Issue model (Ticket 1), Labels model (Ticket 2), Issue CRUD (Ticket 5)
- **Dev / Build:** `elasticsearch-py` or `opensearch-py` library

---

### 8. Acceptance Criteria

- [ ] ES index `tickets` created with correct mapping
- [ ] Creating an issue indexes it in ES
- [ ] Updating an issue re-indexes it in ES
- [ ] Soft-deleting an issue removes it from ES index
- [ ] `GET /api/v1/search?q=login` returns matching issues
- [ ] Title matches rank higher than description matches
- [ ] `?q=bug&labels=groomed` filters correctly
- [ ] ES failure does not cause 500 on issue create/update
- [ ] Search results are paginated

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5), ES cluster provisioned |
| S1 | ES client infrastructure adapter + index mapping |
| S2 | Hook indexing into issue create/update/delete use-cases |
| S3 | Search endpoint + router |

---

### 11. Out of Scope (v1)

- Bulk reindexing command (future ops tooling)
- Search analytics / popular queries
- Cross-project search

---

## 🟣 Ticket 15 — Issue Event Publishing (Redis Pub/Sub)

---
epic: "Issue Management System"
title: "Publish issue lifecycle events to Redis Pub/Sub"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Publish domain events (created, assigned, status changed) to Redis Pub/Sub channel `ticket:events` so notification-service can subscribe and create notifications.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Events published | All specified events fire on corresponding actions |
| Notification-service consumes | Subscriber receives events and persists notifications |
| Non-blocking | Event publish failure does not fail the API request |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- Redis Pub/Sub channel: `ticket:events`
- Events:
  - `TicketCreated` — on issue create (if assignee set): `{ type, ticket_id, project_id, reporter_id, assignee_id }`
  - `TicketAssigned` — on assignee change: `{ type, ticket_id, assignee_id, reporter_id, assigner_id }`
  - `TicketStatusChanged` — on status update: `{ type, ticket_id, new_status, reporter_id, changed_by_id }`
- Event publisher in `src/infrastructure/pubsub/event_publisher.py` implementing `EventPublisherPort`
- Publish is async and non-blocking — wrapped in try/except, logged on failure

#### Should Have (S)
- `CommentAdded` event: `{ type, ticket_id, comment_id, author_id, ticket_reporter_id, ticket_assignee_id }` (when comments epic lands)
- Correlation ID in event payload for tracing

#### Could Have (C)
- Event payload versioning (`event_version: 1`)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Reliability | Redis publish failure is logged but does not return 500 to client |
| Code Quality | Event publisher behind port/adapter interface (Onion Architecture) |
| Performance | Publish is async — no blocking the API response |

---

### 6. Dependencies

- **Runtime:** Redis (ElastiCache), `redis-py` async client
- **Backend:** Issue CRUD (Ticket 5), Issue Assignment (Ticket 9), Status Workflow (Ticket 10)
- **Functional:** notification-service must be subscribed to `ticket:events` channel

---

### 8. Acceptance Criteria

- [ ] `TicketCreated` published when issue created with assignee
- [ ] `TicketAssigned` published when assignee changes
- [ ] `TicketStatusChanged` published when status changes
- [ ] Event payloads contain all specified fields
- [ ] Redis failure does not cause API 500
- [ ] Events logged at DEBUG level for traceability
- [ ] Event publisher implements abstract `EventPublisherPort`

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Issue CRUD (Ticket 5), Redis connection setup |
| S1 | Event publisher adapter |
| S2 | Hook publishing into relevant use-cases |

---

### 11. Out of Scope (v1)

- Redis Streams (Pub/Sub is sufficient for MVP)
- Event sourcing / event store
- Retry queue for failed publishes

---

## 🟣 Ticket 16 — Issue List Page + Filter Bar (Frontend)

---
epic: "Issue Management System"
title: "Build Issue List page with filter bar (React + TypeScript)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Display a paginated, filterable list of issues for a project. Users can filter by status, priority, assignee, and search by title. This is the primary landing page after project selection.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Issues displayed | Paginated list renders correctly |
| Filters work | UI filters update query params and refetch |
| Search works | Title search triggers ES-backed search |
| Responsive | Usable on desktop and tablet |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `TicketList.tsx` — paginated list of issue cards
- `TicketCard.tsx` — displays: title, status badge, priority badge, assignee avatar/name, labels, issue type icon
- `FilterBar.tsx` — dropdowns for status, priority, assignee + search input for title
- Fetch from `GET /api/v1/projects/{project_id}/tickets?status=&priority=&assignee_id=&q=`
- Pagination controls (previous / next / page number)
- URL query params synced with filter state (shareable filtered URLs)
- Loading state and empty state ("No issues match your filters")

#### Should Have (S)
- Debounced search input (300ms) for title search
- Sort dropdown (newest, priority, due date)
- Label pills displayed on each issue card
- Click issue card → navigate to Issue Detail page

#### Could Have (C)
- Kanban board toggle (group by status columns)
- Bulk selection + bulk status update

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Performance | List renders < 1s for 50 issues |
| State | React Query / TanStack Query for server state management |
| Routing | Route: `/projects/:projectId/issues` |
| Code Quality | TypeScript strict mode; components are typed with interfaces |

---

### 6. Dependencies

- **Runtime:** React 18, TypeScript, TanStack Query, React Router
- **Backend:** Issue List API (Ticket 5), Filtering API (Ticket 7), Pagination (Ticket 8)
- **Dev / Build:** Vite, Tailwind CSS or CSS modules

---

### 8. Acceptance Criteria

- [ ] Issue list renders with all required fields per card
- [ ] Status filter dropdown works and updates list
- [ ] Priority filter dropdown works and updates list
- [ ] Assignee filter dropdown works and updates list
- [ ] Search input triggers search (debounced)
- [ ] Pagination controls navigate between pages
- [ ] Combined filters work (status + priority + search)
- [ ] URL query params reflect current filter state
- [ ] Loading spinner shown during fetch
- [ ] Empty state displayed when no results
- [ ] Components are TypeScript with proper interfaces
- [ ] All component functions have JSDoc comments

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Backend APIs ready (Tickets 5, 7, 8) |
| S1 | `ticketsApi.ts` — API client functions |
| S2 | `useTickets.ts` — React Query hook |
| S3 | `TicketCard.tsx`, `FilterBar.tsx`, `TicketList.tsx` |
| S4 | Wire routing and query param sync |

---

### 11. Out of Scope (v1)

- Kanban board view
- Drag-and-drop status changes
- Real-time updates (polling is acceptable)
- Bulk operations

---

## 🟣 Ticket 17 — Create/Edit Issue Form (Frontend)

---
epic: "Issue Management System"
title: "Build Create and Edit Issue form (React + TypeScript)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Provide a form for creating new issues and editing existing ones. Support all issue fields including type, priority, assignee, labels, parent linking, and due date.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Create issue works | Form submits and creates issue via API |
| Edit issue works | Form pre-fills and updates issue via API |
| Validation | Client-side validation matches backend schema |
| UX | Clear error messages, loading states, success feedback |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `TicketForm.tsx` — reusable for create and edit modes
- Fields: title (text, required), description (textarea), issue_type (dropdown: story/epic/task/bug/test/incident), priority (dropdown: none/low/medium/high/critical), assignee_id (user picker dropdown), labels (multi-select tag input), parent_id (issue picker — only when type=task), due_date (date picker)
- Create mode: `POST /api/v1/projects/{project_id}/tickets`
- Edit mode: pre-fill from `GET /api/v1/tickets/{ticket_id}`, submit via `PUT /api/v1/tickets/{ticket_id}`
- Client-side validation: title required (max 500 chars), type required
- Error display: field-level errors from 422 response
- Success: redirect to issue detail page after create/edit

#### Should Have (S)
- `epic_category` dropdown (shown only when type = `epic`)
- Assignee picker fetches users from `GET /api/v1/users`
- Label input with autocomplete from existing project labels
- Confirm discard on navigation away with unsaved changes

#### Could Have (C)
- Markdown preview for description field
- Attachment URL input field

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| State | React Hook Form or controlled form with local state |
| Routing | Create: `/projects/:projectId/issues/new`; Edit: `/issues/:ticketId/edit` |
| Code Quality | TypeScript interfaces for form values; reusable form component |

---

### 6. Dependencies

- **Runtime:** React 18, TypeScript, React Hook Form (or equivalent)
- **Backend:** Issue CRUD API (Ticket 5), Labels API (Ticket 6), Users list endpoint (auth-service)

---

### 8. Acceptance Criteria

- [ ] Create form submits and creates issue successfully
- [ ] Edit form pre-fills all fields from existing issue
- [ ] Edit form submits and updates issue successfully
- [ ] Title required validation prevents empty submit
- [ ] Issue type dropdown shows all 6 types
- [ ] Priority dropdown shows all 5 levels
- [ ] Assignee picker lists available users
- [ ] Labels can be added/removed as tags
- [ ] `parent_id` picker shown only for `task` type
- [ ] `epic_category` shown only for `epic` type
- [ ] Due date picker works correctly
- [ ] 422 errors displayed per field
- [ ] Redirect to detail page on success

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Backend CRUD API ready (Ticket 5) |
| S1 | Form component with all fields |
| S2 | Create mode wiring (API call + routing) |
| S3 | Edit mode wiring (prefill + API call) |
| S4 | Validation + error handling |

---

### 11. Out of Scope (v1)

- Rich text editor for description
- Image/file attachment upload
- Draft auto-save
- Issue templates

---

## 🟣 Ticket 18 — Issue Detail Page (Frontend)

---
epic: "Issue Management System"
title: "Build Issue Detail page (React + TypeScript)"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Display full issue details including metadata, labels, parent/child links, and action buttons (edit, delete, assign, change status). This page is the hub for issue interaction.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| All details rendered | All issue fields displayed correctly |
| Actions work | Edit, delete, status change, assignment work from this page |
| Hierarchy visible | Parent link and subtask list displayed |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- `TicketDetail.tsx` — full issue view page
- Display: title, description, type, status (with badge), priority (with badge), reporter, assignee, labels (as pills), due date, created/updated timestamps
- Action buttons: Edit (→ edit form), Delete (confirm modal → soft delete), Change Status (dropdown)
- Assignee change: inline dropdown or modal
- Parent issue link (if `parent_id` set — clickable to navigate)
- Subtasks list (fetch from `GET /api/v1/tickets/{ticket_id}/subtasks`)
- Fetch from `GET /api/v1/tickets/{ticket_id}`
- 404 page if issue not found

#### Should Have (S)
- Comments section (integrates with Comments epic — placeholder or stub in v1)
- Breadcrumb: Project → Issue List → Issue #ID
- Activity log stub (status changes, assignments)

#### Could Have (C)
- Side panel layout (detail left, activity right)
- Print-friendly view

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Routing | Route: `/issues/:ticketId` |
| State | React Query for data fetching; mutations for actions |
| Code Quality | TypeScript strict; proper loading/error/404 states |

---

### 6. Dependencies

- **Runtime:** React 18, TypeScript, TanStack Query
- **Backend:** Issue CRUD (Ticket 5), Subtasks endpoint (Ticket 11), Status workflow (Ticket 10)

---

### 8. Acceptance Criteria

- [ ] All issue fields rendered correctly
- [ ] Status and priority displayed as colored badges
- [ ] Labels displayed as pills/tags
- [ ] Edit button navigates to edit form
- [ ] Delete button shows confirmation modal, then soft-deletes
- [ ] Status change dropdown triggers PATCH and updates UI
- [ ] Assignee change works inline
- [ ] Parent issue is clickable link
- [ ] Subtasks listed with links
- [ ] Due date displayed (with overdue indicator if past)
- [ ] 404 page shown for invalid/deleted issue
- [ ] Loading state while fetching

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Backend APIs ready (Tickets 5, 10, 11) |
| S1 | Detail page layout + data fetching |
| S2 | Action buttons (edit, delete, status, assign) |
| S3 | Parent/child links + subtask list |

---

### 11. Out of Scope (v1)

- Inline editing of fields
- Real-time updates (manual refresh acceptable)
- Comments section (separate epic, placeholder OK)
- File attachments display

---

## 🟣 Ticket 19 — Issue API Tests (Backend)

---
epic: "Issue Management System"
title: "Write comprehensive pytest tests for Issue APIs"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Ensure reliability and correctness of all issue API endpoints through automated tests. Cover CRUD, filters, search, assignment, status workflow, and edge cases.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| All endpoints tested | Every endpoint has happy-path + error test |
| Edge cases covered | Invalid data, not found, unauthorized, soft-deleted |
| CI integration | Tests run in GitLab CI pipeline |
| Coverage | ≥ 80% line coverage on issue-related code |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- **CRUD tests:**
  - Create issue → 201, correct response shape
  - Create issue with missing title → 422
  - Get issue → 200, correct data
  - Get deleted issue → 404
  - Update issue → 200, fields changed
  - Delete issue → 204, is_deleted = true
  - Unauthorized delete → 403
- **Filter tests:**
  - Filter by status → correct results
  - Filter by priority → correct results
  - Filter by assignee → correct results
  - Combined filters → correct AND behavior
  - Invalid filter value → 422
- **Label tests:**
  - Create issue with labels → labels persisted
  - Update issue labels → labels replaced
  - Duplicate label → handled gracefully
- **Assignment tests:**
  - Assign on create → saved
  - Reassign → updated + event published
  - Member self-assign → allowed
  - Member assign other → 403
- **Status workflow tests:**
  - Valid transition → 200
  - Invalid transition → 400
- **Parent-child tests:**
  - Create with parent → linked
  - Non-task with parent → 400
  - Get subtasks → correct children
- **Pagination tests:**
  - Page 1 → correct items
  - Page 2 → next items
  - Beyond last page → empty

#### Should Have (S)
- Elasticsearch search tests (mocked ES or test ES instance)
- Event publishing tests (mock Redis, verify PUBLISH called)
- Performance: list endpoint < 200ms for 100 issues (benchmark test)

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Code Quality | pytest with fixtures, factories (factory_boy), async tests |
| CI | Tests run in GitLab CI backend job |

---

### 6. Dependencies

- **Dev / Build:** pytest, pytest-asyncio, httpx (for async test client), factory_boy
- **Backend:** All issue API tickets (Tickets 1–15)

---

### 8. Acceptance Criteria

- [ ] All test categories above have passing tests
- [ ] `pytest` passes with 0 failures
- [ ] Coverage ≥ 80% on issue-related modules
- [ ] Tests use fixtures and factories (no hard-coded test data)
- [ ] Async tests use `pytest-asyncio`
- [ ] Tests documented with clear test names (e.g., `test_create_issue_missing_title_returns_422`)
- [ ] Tests run in CI pipeline

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | All backend issue tickets complete |
| S1 | Test fixtures + factories |
| S2 | CRUD tests |
| S3 | Filter, pagination, assignment, status tests |
| S4 | Search + event tests (mocked) |

---

### 11. Out of Scope (v1)

- Load testing / stress testing
- E2E tests (separate ticket)
- Frontend test coverage (Ticket 20)

---

## 🟣 Ticket 20 — Issue Frontend Tests

---
epic: "Issue Management System"
title: "Write tests for Issue frontend components"
status: todo
assignee: ""
created: 2026-04-24
---

### 1. Vision
> Ensure frontend issue components work correctly through unit and integration tests.

---

### 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Component tests pass | All key components tested |
| Interaction tested | Filter changes, form submissions, navigation |
| CI integration | Tests run in GitLab CI frontend job |

---

### 4. Functional Requirements (MoSCoW)

#### Must Have (M)
- **TicketCard** — renders title, status badge, priority badge, labels
- **TicketList** — renders list, handles empty state, handles loading
- **FilterBar** — filter changes trigger callback with correct params
- **TicketForm** — create mode submits correct payload; edit mode pre-fills; validation prevents empty title
- **TicketDetail** — renders all fields; action buttons call correct APIs

#### Should Have (S)
- API mock layer (MSW — Mock Service Worker) for integration tests
- Snapshot tests for badge/label components

#### Could Have (C)
- Playwright/Cypress smoke test for issue create → list → detail flow

---

### 5. Non-Functional Requirements

| Category | Requirement |
|----------|-------------|
| Code Quality | Vitest + React Testing Library |
| CI | `npm run test` passes in CI |

---

### 6. Dependencies

- **Dev / Build:** Vitest, React Testing Library, MSW
- **Frontend:** Issue frontend tickets (Tickets 16–18)

---

### 8. Acceptance Criteria

- [ ] All component test suites pass
- [ ] Form validation tested (empty title rejected)
- [ ] Filter interactions tested
- [ ] API calls mocked (no real backend needed)
- [ ] Tests run in CI frontend job
- [ ] Test file names follow `*.test.tsx` convention

---

### 9. Build Order

| Stage | Scope |
|-------|-------|
| S0 | Frontend issue tickets complete (Tickets 16–18) |
| S1 | MSW setup + API mocks |
| S2 | Component unit tests |
| S3 | Integration tests (form submit, filter apply) |

---

### 11. Out of Scope (v1)

- E2E tests across full stack
- Visual regression tests
- Accessibility audit (a11y)
- Performance profiling

---

## 📋 Overall Build Order (Recommended Sprint Sequence)

```
Week 1 — Backend Foundation
├── S0: Ticket 3 (Domain Entity & Enums) — no dependencies
├── S1: Ticket 1 (Issue Model & Migration)
├── S1: Ticket 2 (Issue Labels Model & Migration)
├── S2: Ticket 4 (Pydantic Schemas)
├── S3: Ticket 5 (Issue CRUD APIs) ← CORE
├── S3: Ticket 6 (Labels CRUD API)
├── S4: Ticket 7 (Filtering)
├── S4: Ticket 8 (Pagination)
├── S4: Ticket 9 (Assignment Logic)
├── S4: Ticket 10 (Status Workflow)
├── S4: Ticket 11 (Parent-Child Linking)
├── S4: Ticket 12 (Due Date Support)
├── S4: Ticket 13 (Soft Delete)

Week 2 — Integration + Frontend + Tests
├── S5: Ticket 14 (Elasticsearch Indexing & Search)
├── S5: Ticket 15 (Redis Pub/Sub Events)
├── S6: Ticket 16 (Issue List + Filter Bar — Frontend)
├── S6: Ticket 17 (Create/Edit Form — Frontend)
├── S6: Ticket 18 (Issue Detail Page — Frontend)
├── S7: Ticket 19 (Backend API Tests)
├── S7: Ticket 20 (Frontend Tests)
```

---

## 📌 Notes for Scrum Master

1. **Ticket 5 (Issue CRUD) is the critical path** — all other backend tickets depend on it. Prioritize and assign to strongest backend dev.
2. **Tickets 3, 1, 2 can be parallelized** — Enums have no dependency; Models 1 & 2 depend on project model but not each other.
3. **Tickets 7–13 are independent of each other** — can be assigned to different team members after Ticket 5 is done.
4. **Frontend tickets (16–18) can start once Ticket 5 + 7 APIs are deployed** — use mock API if backend is delayed.
5. **Ticket 14 (Elasticsearch) requires ES cluster** — ensure infra is provisioned before sprint starts.
6. **Ticket 15 (Redis Pub/Sub) requires Redis** — ensure ElastiCache is provisioned.
7. **Test tickets (19, 20) should NOT be deferred** — write tests as features land, track with these tickets for coverage gates.
8. **Stack clarification:** Backend = FastAPI + SQLAlchemy (NOT Django). Frontend = React + TypeScript (NOT plain JSX).
