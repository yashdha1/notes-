 # Frontend Workflow — Issue Tracker

> Stack: React · Zustand · Tailwind CSS · React Router DOM · JavaScript

---

## Pages & Component Inventory

| ID  | Artifact      | Type        | Route                        |
| --- | ------------- | ----------- | ---------------------------- |
| P1  | Login         | Page        | `/login`                     |
| P2  | Signup        | Page        | `/signup`                    |
| P3  | Home          | Page        | `/home`                      |
| P4  | Activity      | Page        | `/activity`                  |
| P5  | Issue Detail  | Page        | `/:project/issues/:issue_id` |
| C1  | Navbar        | Component   | (global)                     |
| C2  | LeftDivision  | Component   | (inside /home)               |
| C3  | RightDivision | Component   | (inside /home)               |
| M1  | CreateProject | Modal/Popup | (triggered from C2)          |
| M2  | CreateIssue   | Modal/Popup | (triggered from C2 / C3)     |

---

## Zustand Stores Required

| Store           | Owns                                           |
| --------------- | ---------------------------------------------- |
| `authStore`     | user session, login/logout state               |
| `projectStore`  | project list, active project, CRUD             |
| `issueStore`    | issues per project, active issue, CRUD, status |
| `activityStore` | activity feed / notifications                  |
| `uiStore`       | modal open states, sidebar filters             |

---

## Dependency Graph

```
[S0] Project Bootstrap
      ├──> [S1a] Router Setup
      ├──> [S1b] Zustand Stores
      └──> [S1c] Tailwind Config + Base Styles
                    │
            [S2] Auth Pages (/login, /signup)
              uses: authStore, Router
                    │
            [S3] Protected Route Wrapper
              uses: authStore, Router
                    │
              ┌─────┴─────┐
         [S4a] Navbar   [S4b] Activity Page
          (C1)            (P4)
              │
         [S5] Home Page Shell (P3 layout)
              │
       ┌──────┴──────┐
  [S6a] LeftDivision  [S6b] RightDivision
    (C2)                 (C3)
       │                    │
  [S7a] CreateProject   [S7b] CreateIssue
    Popup (M1)            Popup (M2)
              │
         [S8] Issue Detail Page (P5)
              │
         [S9] Backend API Integration
```

---

## Topological Stages

### Stage 0 — Foundation (no dependencies)
- [ ] Scaffold project: `npm create vite@latest` with React template
- [ ] Install deps: `react-router-dom`, `zustand`, `tailwindcss`, `postcss`, `autoprefixer`
- [ ] Configure `tailwind.config.js` + `index.css`
- [ ] Set up base folder structure: `pages/`, `components/`, `stores/`, `api/`

---

### Stage 1 — Infrastructure (depends on S0)
**1a — Router Setup**
- [ ] Configure `BrowserRouter` in `main.jsx`
- [ ] Define all routes (`/login`, `/signup`, `/home`, `/activity`, `/:project/issues/:id`)
- [ ] Create `ProtectedRoute` wrapper shell (auth guard placeholder)

**1b — Zustand Stores**
- [ ] `authStore` — `{ user, token, login(), logout() }`
- [ ] `projectStore` — `{ projects[], activeProject, setActive(), addProject() }`
- [ ] `issueStore` — `{ issues[], activeIssue, addIssue(), updateStatus() }`
- [ ] `activityStore` — `{ feed[], fetchActivity() }`
- [ ] `uiStore` — `{ modals: { createProject, createIssue }, toggleModal() }`

**1c — Tailwind Base**
- [ ] Color palette, font, spacing tokens in `tailwind.config.js`
- [ ] Global reset + typography in `index.css`

---

### Stage 2 — Auth Pages (depends on S1a, S1b, S1c)
**Pages: `/login`, `/signup`**
- [ ] `LoginPage` — email + password form, submit calls `authStore.login()`
- [ ] `SignupPage` — name + email + password form, submit calls `authStore.signup()`
- [ ] Redirect to `/home` on success
- [ ] Basic form validation (client-side)
- [ ] Tailwind styled cards/inputs

---

### Stage 3 — Protected Route (depends on S2)
- [ ] `ProtectedRoute` reads `authStore.user`; redirects to `/login` if null
- [ ] Wrap all private routes (`/home`, `/activity`, `/:project/issues/:id`)

---

### Stage 4 — Navbar + Activity Page (depends on S3)
**4a — Navbar (C1)**
- [ ] Search bar (UI only first, wire later)
- [ ] Profile icon — links to profile / logout
- [ ] Activity bell icon — links to `/activity`
- [ ] Persistent across all protected pages via layout

**4b — Activity Page (P4)**
- [ ] Consumes `activityStore.feed`
- [ ] List of activity events (created issue, changed status, commented)
- [ ] Empty state

---

### Stage 5 — Home Page Shell (depends on S4a, S3)
**Page: `/home`**
- [ ] Two-column layout: `LeftDivision` | `RightDivision`
- [ ] Navbar at top via shared layout
- [ ] Responsive breakpoint for sidebar collapse

---

### Stage 6 — Home Divisions (depends on S5)
**6a — LeftDivision (C2)**
- [ ] Project list — reads `projectStore.projects`
- [ ] Active project highlight — `projectStore.setActive()`
- [ ] Filter controls (by status, assignee — UI shell)
- [ ] "Create Project" button → triggers `uiStore.toggleModal('createProject')`
- [ ] "Create Issue" button → triggers `uiStore.toggleModal('createIssue')`

**6b — RightDivision (C3)**
- [ ] Issues list for `projectStore.activeProject` — reads `issueStore.issues`
- [ ] Project header / details (name, description, stats)
- [ ] Issue row component: title · status badge · assignee · date
- [ ] Click issue row → navigate to `/:project/issues/:id`
- [ ] Empty state when no project selected

---

### Stage 7 — Modals / Popups (depends on S6a, S6b)
**7a — CreateProject Modal (M1)**
- [ ] Fields: project name, description, (optional) members
- [ ] Submit calls `projectStore.addProject()`
- [ ] Closes modal on success, auto-selects new project
- [ ] Controlled by `uiStore.modals.createProject`

**7b — CreateIssue Modal (M2)**
- [ ] Fields: title, description, status (open/in-progress/closed), assignee, project (pre-filled)
- [ ] Submit calls `issueStore.addIssue()`
- [ ] Closes modal, updates `RightDivision` list
- [ ] Controlled by `uiStore.modals.createIssue`

---

### Stage 8 — Issue Detail Page (depends on S7)
**Page: `/:project/issues/:issue_id`**
- [ ] Full issue details: title, description, status, assignee, created_at
- [ ] Status change control — calls `issueStore.updateStatus()`
- [ ] Comments section (list + add comment)
- [ ] Breadcrumb back to `/home`
- [ ] All data from `issueStore.activeIssue`

---

### Stage 9 — Backend Integration (depends on S0–S8)
- [ ] Set up `api/` layer: `axios` or `fetch` wrappers with base URL + auth header
- [ ] Wire `authStore` → `POST /auth/login`, `POST /auth/signup`
- [ ] Wire `projectStore` → `GET /projects`, `POST /projects`
- [ ] Wire `issueStore` → `GET /projects/:id/issues`, `POST /issues`, `PATCH /issues/:id`
- [ ] Wire `activityStore` → `GET /activity`
- [ ] Wire comments → `GET/POST /issues/:id/comments`
- [ ] Handle loading states, error states in all stores
- [ ] Persist auth token (`localStorage`) and rehydrate on app load

---

## Summary Build Order

```
S0 → S1(a,b,c) → S2 → S3 → S4(a,b) → S5 → S6(a,b) → S7(a,b) → S8 → S9
```

| Stage | Deliverable                  | Blocks     |
| ----- | ---------------------------- | ---------- |
| S0    | Project scaffold + tooling   | everything |
| S1    | Router + Stores + Tailwind   | S2         |
| S2    | Login + Signup pages         | S3         |
| S3    | Protected route              | S4, S5     |
| S4    | Navbar + Activity page       | S5         |
| S5    | Home layout shell            | S6         |
| S6    | LeftDivision + RightDivision | S7         |
| S7    | Create Project/Issue modals  | S8         |
| S8    | Issue Detail page            | S9         |
| S9    | Full backend wiring          | —          |




### TECH STACK

1. js           -> flexiblity and extensiblity. 
2. react     -> component based design and easy page making. 
3. zustand -> store management. for global variables management. 
4. tailwind -> for making the UI. 
5. NPM    -> package manager. 
6. vite       -> package runner. 




filter weighteage : 


dataes: 
pending, 
priority,
User Issues verses the team or the project issues:-> 

analytics -> of the project detials:-> 


issue_type ->  

epics in the project: 

epics can be choosing:-> when creating the issues:

cutome id creation logic :->





```dbml
![[Pasted image 20260423173225.png]]
```






dbml : 

![[Pasted image 20260423173316.png]] 

