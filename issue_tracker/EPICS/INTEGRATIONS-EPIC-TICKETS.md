---
epic: "Frontend Integration"
title: "Wire All Frontend Pages to Backend APIs"
status: in-progress
assignee: ""
created: 2026-04-24
---

# Wire All Frontend Pages to Backend APIs

## 1. Vision
> Connect every completed frontend page and component to its corresponding backend API, replacing all mock/local state with real server data via Axios + Zustand stores.

---

## 2. Goals & Success Metrics

| Goal | Success Metric |
|------|----------------|
| Full end-to-end auth flow | User can register, login, and logout; JWT persisted and cleared correctly |
| Pages reflect live data | No hardcoded fixtures anywhere in production code |
| Protected routes enforced | Unauthenticated requests redirect to `/login` |

---

## 4. Functional Requirements (MoSCoW)

### Must Have (M)
- **Auth**  - Login, Signup, Logout wired to backend; JWT stored in `authStore`; axios interceptor attaches token
- **Navbar + Search** -  Search bar calls `GET /search?q=` (search-proxy); results rendered in dropdown
- **Activity Feed**  `GET /activity` feeds `activityStore`; page renders real events
- **Left Division** — `GET /projects` populates project list; `POST /projects` (CreateProject modal) creates project; filters call `GET /projects?status=&assignee=`
- **Right Division** — `GET /projects/:id` loads project details + members; `GET /projects/:id/issues` loads issue list; role-based UI (only authorized users see edit/delete)
- **Issue Detail Page** — `GET /issues/:id` loads issue; `GET /issues/:id/comments` loads comments; `POST /issues/:id/comments` submits comment; `PATCH /issues/:id` changes status/assignee/priority
- **Create Issue Modal** — `POST /issues` wired; project + epic dropdowns populated from store
- **Create Project Modal** — `POST /projects` wired
- **Create Epic** — `POST /epics` wired

### Should Have (S)
- Optimistic UI updates on issue state change
- Error toasts on API failures (axios response interceptor)
- Loading skeletons while fetching

### Could Have (C)
- Infinite scroll / pagination on issue list
- Real-time activity via polling (5s interval)

### Won't Have (W)
- WebSocket live updates (v1)
- Offline support

---

## 5. Non-Functional Requirements

| Category     | Requirement |
|--------------|-------------|
| Performance  | No redundant API calls; cache in Zustand; invalidate on mutation |
| Security     | JWT never in localStorage if XSS risk; use httpOnly cookie or memory store |
| Code Quality | Axios instance in `api/client.js`; one file per resource in `api/`; no raw fetch |
| State        | All server state in Zustand stores; no prop-drilling of API data |
| Routing      | React Router protected route wrapper guards all pages except `/login` and `/signup` |

---

## 6. Dependencies

- **Runtime:** `axios`, `zustand`, `react-router-dom`
- **Dev / Build:** Vite proxy config for local dev (`/api → localhost:8000`)
- **Backend:** All backend APIs must be complete and tested before integration begins
- **Search:** Search-proxy service (ticket-search-system) must be running for search integration

---

## 8. Acceptance Criteria

- [ ] User can sign up, log in, and log out; JWT attached to all subsequent requests
- [ ] Unauthenticated navigation to any protected route redirects to `/login`
- [ ] Project list in LeftDivision updates immediately after CreateProject modal submits
- [ ] RightDivision issue list fetches from backend; creating an issue appends it without full reload
- [ ] Issue Detail page loads comments and allows posting a new one; status change reflects via PATCH
- [ ] Search bar returns real results from the search-proxy
- [ ] Activity page shows real events from backend
- [ ] No `console.error` API failures in happy path; errors surface as UI toasts
- [ ] All `api/*.js` files have JSDoc; function names `camelCase`

---

## 9. Build Order

| Stage | Scope                                                                                   |
| ----- | --------------------------------------------------------------------------------------- |
| S0    | `api/client.js` — axios instance, base URL from env, JWT interceptor, error interceptor |
| S1    | Auth integration — `authStore` wired; Login + Signup + Logout; protected route          |
| S2    | `api/projects.js` + `projectStore` — fetch list, fetch by id, create project            |
| S3    | LeftDivision — project list, filters, CreateProject modal wired                         |
| S4    | RightDivision — project detail, issue list, role-gated actions                          |
| S5    | `api/issues.js` + `issueStore` — fetch issue, create, patch status/assignee             |
| S6    | Issue Detail Page — comments fetch + post, status change PATCH                          |
| S7    | Create Issue + Create Epic modals wired                                                 |
| S8    | Navbar search wired to search-proxy                                                     |
| S9    | Activity page wired to `GET /activity`                                                  |
| S10   | Manual smoke test all flows; fix edge cases                                             |

---

## 11. Out of Scope (v1)

- File/attachment upload on issues
- Bulk issue operations
- Real-time collaborative editing




---

# Tickets

## Ticket Index

| #       | Title                                                     | Type  | Layer      | Dependencies        |
| ------- | --------------------------------------------------------- | ----- | ---------- | ------------------- |
| INTG-00 | Axios client + Vite proxy setup + interceptor             | Task  | Frontend   | —                   |
| INTG-01 | Auth integration — Login / Signup / Logout                | Story | Frontend   | Auth API done       |
| INTG-02 | Project API layer + projectStore                          | Task  | Frontend   | Project APIs done   |
| INTG-03 | LeftDivision — project list, filters, CreateProject modal | Story | Frontend   | INTG-02             |
| INTG-04 | RightDivision — project detail, issue list, role-gated    | Story | Frontend   | INTG-02, Issue APIs |
| INTG-05 | Issue API layer + issueStore                              | Task  | Frontend   | Issue APIs done     |
| INTG-06 | Issue Detail — comments + status PATCH                    | Story | Frontend   | INTG-05             |
| INTG-07 | Create Issue + Create Epic modals wired                   | Story | Frontend   | INTG-05             |
| INTG-08 | Navbar search → search-proxy                              | Task  | Frontend   | SRCH-03 done        |
| INTG-09 | Activity page → `/activity` API                           | Task  | Frontend   | Activity API done   |
| INTG-10 | Full-stack smoke test + bug sweep                         | Test  | Full Stack | All INTG above      |

---

#### INTG-00 · Axios client + Vite proxy setup
**Stage:** S0 | **Size:** S | **Type:** Task

###### Description
Create `src/api/client.js` — a configured Axios instance with base URL from `VITE_API_URL`, a request interceptor that attaches the JWT from `authStore`, and a response interceptor that handles 401 (clear token, redirect to `/login`) and shows a toast on other errors. Add Vite dev proxy so `/api → localhost:8000`.

###### Acceptance Criteria
- [ ] `api/client.js` exports a single Axios instance used by all other API files
- [ ] JWT attached automatically on every request if present in `authStore`
- [ ] 401 response clears token and redirects to `/login`
- [ ] Non-401 errors trigger a UI toast (error message from response body)
- [ ] Vite proxy configured in `vite.config.js` for `/api` prefix
- [ ] `VITE_API_URL` read from `.env`; never hard-coded

---

#### INTG-01 · Auth integration — Login / Signup / Logout
**Stage:** S1 | **Size:** M | **Type:** Story

###### Description
Wire the Login, Signup, and Logout flows to the auth backend. `authStore` holds `{ user, token, isAuthenticated }`. On login/signup success store the JWT and user object; on logout clear both. Add a `ProtectedRoute` wrapper component that redirects unauthenticated users to `/login`.

###### Acceptance Criteria
- [ ] `POST /api/auth/login` called on Login form submit; JWT + user stored in `authStore`
- [ ] `POST /api/auth/signup` called on Signup form submit; auto-login after success
- [ ] Logout clears `authStore` and redirects to `/login`
- [ ] `ProtectedRoute` wraps all non-auth routes; unauthenticated access redirects to `/login`
- [ ] Form shows field-level errors from 422 / 400 responses
- [ ] JWT persisted in memory (not localStorage) for XSS safety
- [ ] `authStore` exported from `stores/authStore.js`

---

#### INTG-02 · Project API layer + projectStore
**Stage:** S2 | **Size:** S | **Type:** Task

###### Description
Create `src/api/projects.js` with functions: `fetchProjects()`, `fetchProjectById(id)`, `createProject(payload)`. Wire these into `projectStore` (Zustand) with actions `loadProjects`, `loadProject`, `addProject`. Store holds `{ projects, currentProject, loading, error }`.

###### Acceptance Criteria
- [ ] `GET /api/projects` populates `projectStore.projects`
- [ ] `GET /api/projects/:id` populates `projectStore.currentProject`
- [ ] `POST /api/projects` appends new project to store without full refetch
- [ ] All functions in `api/projects.js` have JSDoc
- [ ] Store `loading` flag set true during fetch; cleared on success or error

---

#### INTG-03 · LeftDivision — project list, filters, CreateProject modal
**Stage:** S3 | **Size:** M | **Type:** Story

###### Description
Wire `LeftDivision` to `projectStore`. On mount call `loadProjects`. Filter controls (`status`, `assignee`) call `GET /api/projects?status=&assignee=` and update the list. CreateProject modal submits `POST /api/projects` via `addProject` action; closes and shows the new project in the list on success.

###### Acceptance Criteria
- [ ] Project list renders from `projectStore.projects` on page load
- [ ] Status filter dropdown updates the displayed list (query param on request)
- [ ] Assignee filter updates the displayed list
- [ ] CreateProject modal submits and new project appears in list immediately
- [ ] CreateProject modal closes and clears form on success
- [ ] API error in modal shows inline error message (not just console)
- [ ] Loading skeleton shown while `projectStore.loading` is true

---

#### INTG-04 · RightDivision — project detail, issue list, role-gated actions
**Stage:** S4 | **Size:** M | **Type:** Story

###### Description
When a project is selected in LeftDivision, call `loadProject(id)` to populate `projectStore.currentProject`, then fetch `GET /api/projects/:id/issues` for the issue list. Role-based UI: only project lead / admin see edit / delete buttons. Member role hides those controls.

###### Acceptance Criteria
- [ ] Selecting a project fetches and displays project details (name, description, members)
- [ ] Issue list loads from `GET /api/projects/:id/issues`
- [ ] Edit / delete project controls hidden for `member` role; visible for `lead` / `admin`
- [ ] Clicking an issue navigates to Issue Detail page (`/issues/:id`)
- [ ] Empty state shown when project has no issues
- [ ] Loading state shown while fetching issues

---

#### INTG-05 · Issue API layer + issueStore
**Stage:** S5 | **Size:** S | **Type:** Task

###### Description
Create `src/api/issues.js` with: `fetchIssue(id)`, `createIssue(projectId, payload)`, `updateIssue(id, patch)`, `fetchComments(issueId)`, `postComment(issueId, body)`. Wire into `issueStore` with actions `loadIssue`, `addIssue`, `patchIssue`, `loadComments`, `addComment`.

###### Acceptance Criteria
- [ ] All five API functions exported from `api/issues.js` with JSDoc
- [ ] `issueStore` holds `{ currentIssue, comments, loading, error }`
- [ ] `patchIssue` does optimistic update in store then confirms with server response
- [ ] Failed optimistic update rolls back store state
- [ ] All functions use the shared Axios `client` from INTG-00

---

#### INTG-06 · Issue Detail — comments + status PATCH
**Stage:** S6 | **Size:** M | **Type:** Story

###### Description
Wire the Issue Detail page. On mount call `loadIssue(id)` and `loadComments(id)`. Comment form calls `postComment` and appends to list. Status change dropdown calls `patchIssue({ status })` — optimistic update with rollback on failure.

###### Acceptance Criteria
- [ ] Issue detail fetches and renders all fields from `GET /api/issues/:id`
- [ ] Comments load from `GET /api/issues/:id/comments`
- [ ] Comment form submits `POST /api/issues/:id/comments`; new comment appends to list without reload
- [ ] Status dropdown PATCH updates status optimistically; badge changes immediately
- [ ] Failed status change shows error toast and reverts badge
- [ ] 404 page shown if issue not found or deleted
- [ ] Loading skeleton while fetching

---

#### INTG-07 · Create Issue + Create Epic modals wired
**Stage:** S7 | **Size:** M | **Type:** Story

###### Description
Wire CreateIssue modal (`POST /api/projects/:id/issues`) and CreateEpic modal (`POST /api/epics`). Both are controlled by `uiStore`. On success: close modal, append item to the relevant store, show success toast. Epic dropdown in CreateIssue populates from `projectStore.currentProject.epics`.

###### Acceptance Criteria
- [ ] CreateIssue modal submits and new issue appears in RightDivision issue list
- [ ] CreateEpic modal submits and new epic appears in epic dropdown immediately
- [ ] Both modals show field-level 422 errors
- [ ] Both modals clear form and close on success
- [ ] `uiStore.openModal` / `uiStore.closeModal` control visibility (no local state)
- [ ] Epic picker in CreateIssue fetches options from store (no extra API call if already loaded)

---

#### INTG-08 · Navbar search → search-proxy
**Stage:** S8 | **Size:** S | **Type:** Task

###### Description
Wire the Navbar search input to `GET /api/search?q=&index=issues,projects`. Debounce input 300ms. Display results in a dropdown — issues and projects grouped separately. Clicking a result navigates to the correct detail page.

###### Acceptance Criteria
- [ ] Typing in search input triggers `GET /api/search?q=` after 300ms debounce
- [ ] Results grouped by type (Issues / Projects) in dropdown
- [ ] Clicking an issue result navigates to `/issues/:id`
- [ ] Clicking a project result selects it in LeftDivision / navigates to project view
- [ ] Empty state ("No results") shown for zero hits
- [ ] Search request uses shared Axios client (JWT attached)
- [ ] Dropdown closes on Escape or outside click

---

#### INTG-09 · Activity page → `/activity` API
**Stage:** S9 | **Size:** S | **Type:** Task

###### Description
Wire the Activity page to `GET /api/activity`. Feed response into `activityStore`. Display a chronological feed of events (issue created, status changed, comment added, etc.). Poll every 30s to pick up new events.

###### Acceptance Criteria
- [ ] Activity page fetches from `GET /api/activity` on mount
- [ ] Events rendered in reverse-chronological order with timestamp
- [ ] `activityStore` holds `{ events, loading }` — no prop drilling
- [ ] Auto-polls every 30s; stops polling when component unmounts
- [ ] Loading skeleton on first load; no skeleton on poll refresh
- [ ] Empty state shown when no activity exists

---

#### INTG-10 · Full-stack smoke test + bug sweep
**Stage:** S10 | **Size:** L | **Type:** Test

###### Description
Manual end-to-end smoke test of all integrated flows. Fix any integration bugs found. Document any known gaps as follow-up issues. Covers every acceptance flow in INTG-01 through INTG-09.

###### Test Flows
- [ ] **Auth:** Sign up → login → protected page access → logout → redirect to `/login`
- [ ] **Projects:** Create project → appears in LeftDivision → select → view details
- [ ] **Issues:** Create issue in modal → appears in RightDivision list → click → issue detail loads
- [ ] **Comments:** Open issue → post comment → comment appears → post again → appends
- [ ] **Status change:** Change issue status → badge updates → refresh → status persists
- [ ] **Filters:** Apply status filter in LeftDivision → list narrows → clear filter → list restores
- [ ] **Search:** Type query → results dropdown → click result → correct page
- [ ] **Activity:** Perform several actions → activity page shows events in order
- [ ] **Role gates:** Member login → edit/delete buttons absent → lead login → buttons present
- [ ] **Error paths:** Kill API → error toasts appear; restart API → normal operation resumes

###### Acceptance Criteria
- [ ] All 10 flows above pass without console errors in happy path
- [ ] Each found bug logged as a follow-up issue before this ticket is closed
- [ ] No hardcoded API URLs remain anywhere in `src/`
