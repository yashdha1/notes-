---
epic: "Search System"
title: "Meilisearch Service + SQS Indexing Pipeline"
status: in-progress
assignee: ""
created: 2026-04-24
---

# Meilisearch Service + SQS Indexing Pipeline

## 1. Vision
> Build a search capability for issues and projects using Meilisearch, kept in sync via an SQS-driven consumer. The project-service publishes events; a dedicated FastAPI consumer indexes them.

---

## 2. Goals & Success Metrics

| Goal                                     | Success Metric                                     |
| ---------------------------------------- | -------------------------------------------------- |
| Fast full-text search on issues/projects | p99 < 50ms on query                                |
| Decoupled write path                     | project-service never waits on Meilisearch         |
| Reliable indexing                        | No event loss; dead-letter queue captures failures |

---

## 4. Functional Requirements (MoSCoW)

### Must Have (M)
- Meilisearch running in Docker with persistent volume
- Light FastAPI search-proxy service (`GET /search?q=&index=&filters=`)
- SQS queue: project-service publishes `{event, entity_type, entity_id, data}` on create/update/delete
- FastAPI SQS consumer that reads from queue and upserts/deletes in Meilisearch
- Searchable indexes: `issues` (title, description, status, assignee) and `projects` (name, description)

### Should Have (S)
- Dead-letter queue (DLQ) for failed indexing events
- Search-only restricted API key exposed via `/search/key` endpoint
- Highlight support (`<mark>`) in search results
- Filter by `project_id`, `status`, `assignee_id`

### Could Have (C)
- Pagination via `offset`/`limit`
- Re-index endpoint (`POST /admin/reindex`) to bulk-seed from Postgres

### Won't Have (W)
- Direct frontend → Meilisearch calls (all search goes through FastAPI proxy)
- Real-time WebSocket search updates
--- 
## 5. Non-Functional Requirements

| Category     | Requirement |
|--------------|-------------|
| Performance  | Consumer processes event < 500ms after SQS receive |
| Security     | Master key never exposed; proxy adds JWT auth check |
| Code Quality | Ruff + TDD; docstrings on all public functions |
| Infra        | All services in docker-compose; env vars via `.env` |

---

## 6. Dependencies

- **Runtime:** `meilisearch-python`, `boto3`, `fastapi`, `uvicorn`, `pydantic`
- **Infra:** AWS SQS (or LocalStack for local dev), Docker, `getmeili/meilisearch:v1.7`
- **Backend:** project-service must publish SQS events on issue/project mutations (publisher hook)

---

## 8. Acceptance Criteria

- [ ] `docker compose up` starts meilisearch + search-proxy + sqs-consumer
- [ ] Creating an issue in project-service → event on SQS → indexed in Meilisearch within 2s
- [ ] `GET /search?q=bug&index=issues` returns ranked hits with highlight
- [ ] DLQ captures events where indexing fails; no silent drops
- [ ] All consumer logic covered by pytest with mocked SQS + Meilisearch client
- [ ] Functions named `snake_case`; module-level docstrings present

---

## 9. Build Order

| Stage | Scope                                                                                   |
| ----- | --------------------------------------------------------------------------------------- |
| S0    | Docker-compose: meilisearch container + volume + env vars                               |
| S1    | SQS publisher hook in project-service (create/update/delete events)                     |
| S2    | FastAPI search-proxy: `/search` endpoint + restricted key provisioning on startup       |
| S3    | SQS consumer service: poll → parse event → upsert/delete in Meilisearch                 |
| S4    | Index settings (searchable, filterable, sortable attributes) seeded on consumer startup |
| S5    | DLQ wiring + error handling                                                             |
| S6    | Re-index admin endpoint (bulk seed from Postgres)                                       |
| S7    | Integration tests (LocalStack + Meilisearch test instance)                              |

---

## 11. Out of Scope (v1)

- Semantic / vector search
- Multi-tenancy per-project Meilisearch indexes
- Autocomplete / instant-search UI widget (frontend uses standard debounced query)
--- 

