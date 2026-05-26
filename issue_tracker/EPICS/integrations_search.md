---
epic: "Frontend–Backend Integration & Search"
status: in-progress
assignee: ""
created: 2026-04-27
---

# Integration & Search Tickets

---

#### INTG-01 · Authentication flow integration
**Size:** M | **Type:** Story

###### Description
Connect the login, signup, and logout UI flows to the backend auth endpoints. Persist the user session in client-side state so the token is available for subsequent requests. Add route protection so unauthenticated users are redirected to the login page when accessing any protected area of the app.

###### Acceptance Criteria
- [ ] User can sign up and is automatically logged in afterward
- [ ] User can log in; session state is updated and token is available for future requests
- [ ] Logout clears session state and redirects to login
- [ ] Accessing any protected route without a session redirects to login
- [ ] Validation errors from the server are shown on the form
- [ ] Session token is stored securely (not in a location accessible to injected scripts)

---

#### INTG-02 · Project data integration
**Size:** M | **Type:** Story

###### Description
Wire the project listing and project detail views to live backend data. The sidebar project list should load on page entry and reflect newly created projects immediately. Filters (e.g. by status or assignee) should query the backend rather than filter local data. The create-project flow should submit to the backend and update the UI on success.

###### Acceptance Criteria
- [ ] Project list loads from the backend on page entry
- [ ] Selecting a project loads its details and member list from the backend
- [ ] Filter controls narrow the list via backend queries
- [ ] Creating a project via the modal persists it to the backend and updates the list immediately
- [ ] Loading state is shown while data is fetching
- [ ] API errors are surfaced as UI feedback rather than silent failures

---

#### INTG-03 · Issue management integration
**Size:** M | **Type:** Story

###### Description
Connect the issue list and issue creation flow to backend data. When a project is selected, the issue list should load from the backend. Role-based controls (edit, delete) should be shown or hidden based on the current user's permissions. Creating a new issue should persist it and append it to the list without a full page reload.

###### Acceptance Criteria
- [ ] Issue list for a selected project loads from the backend
- [ ] Edit and delete controls are only visible to users with the appropriate role
- [ ] Creating an issue via the modal persists it and appends it to the list
- [ ] Clicking an issue navigates to the issue detail view
- [ ] Empty state is displayed when a project has no issues
- [ ] Loading state is shown while the issue list is fetching

---

#### INTG-04 · Issue detail & collaboration integration
**Size:** M | **Type:** Story

###### Description
Wire the issue detail page to load full issue data and its comments from the backend. Comment submission should post to the backend and append to the list without a reload. Status changes should be applied immediately in the UI and confirmed with the backend; if the update fails the UI should revert.

###### Acceptance Criteria
- [ ] Issue detail page loads all issue fields from the backend
- [ ] Comments load from the backend on page entry
- [ ] Submitting a comment posts it to the backend and appends it to the list
- [ ] Changing issue status updates the UI immediately (optimistic) and confirms with the server
- [ ] A failed status update reverts the UI and shows an error
- [ ] A 404 or deleted issue shows an appropriate not-found state

---

#### INTG-05 · Creation modal integrations
**Size:** M | **Type:** Story

###### Description
Wire the create-issue and create-epic modals to their respective backend endpoints. Both modals should use shared UI state for open/close control. On successful submission each modal should close, clear its form, and reflect the new item in the relevant list or dropdown immediately.

###### Acceptance Criteria
- [ ] Creating an issue via the modal persists to the backend and appears in the issue list
- [ ] Creating an epic via the modal persists to the backend and appears in the epic dropdown
- [ ] Both modals display field-level validation errors returned by the server
- [ ] Both modals close and reset their form on success
- [ ] Modal visibility is controlled through shared UI state (not local component state)
- [ ] Epic options in the create-issue modal come from already-loaded data where possible

---

#### INTG-06 · Activity feed integration
**Size:** S | **Type:** Task

###### Description
Wire the activity page to load events from the backend. Events should be displayed in reverse-chronological order. The feed should refresh periodically to pick up new activity without requiring a manual page reload.

###### Acceptance Criteria
- [ ] Activity page loads events from the backend on mount
- [ ] Events are displayed in reverse-chronological order with timestamps
- [ ] Feed auto-refreshes at a regular interval
- [ ] Polling stops when the component is unmounted
- [ ] A loading state is shown on initial load only
- [ ] Empty state is shown when there is no activity

--- 

####  SEARCH · Search feature making and Integrations
**Size:** M | **Type:** Story

###### Description
Implement the search experience end-to-end: wire the navbar search input to the backend search service and build out the full indexing pipeline that keeps search data in sync. User input should be debounced before querying. Results should be grouped by type (issues, projects) and clicking a result should navigate to the correct page. On the backend side, the search service must receive and index create/update/delete events from the project service so search results stay current.

###### Acceptance Criteria
- [ ] Typing in the search bar triggers a query to the search service after a short debounce
- [ ] Results are grouped by entity type in the dropdown
- [ ] Clicking a result navigates to the correct detail page
- [ ] No results shows an empty state message
- [ ] The search dropdown closes on Escape or an outside click
- [ ] The search service indexes issues and projects with relevant fields (title, description, status, assignee) 
- [ ] Failed indexing events are captured for retry rather than silently dropped.

--- 



1. issue shall be depended on some others issue. In the backend need to create. until then. 
2. EPIC nikaldo -> and add a field in the issues wala table only. 