# User Stories, Use Cases, and Scenarios

> Story points use Fibonacci scale (1, 2, 3, 5, 8, 13).
> Calibration: 1 = trivial read, 2 = write + validation, 3 = write + side effect,
> 5 = multi-step/multi-table, 8 = multi-endpoint + async events, 13 = cross-service orchestration.

---

## Epic 1: Authentication and Account

**Total: 7 stories, 19 points**

### Stories


| ID   | Story                                                                                                                          | Points |
| ---- | ------------------------------------------------------------------------------------------------------------------------------ | ------ |
| AU-1 | As a visitor, I want to register with email, username, and password, so that I can create an account.                          | 3      |
| AU-2 | As a visitor, I want to log in with my email and password, so that I receive access and refresh tokens.                        | 2      |
| AU-3 | As a user, I want to log out, so that my refresh token is invalidated and no one can use my session.                           | 2      |
| AU-4 | As a user, I want to refresh my access token using my refresh token, so that I stay logged in without re-entering credentials. | 2      |
| AU-5 | As a user, I want to request a password reset OTP via email, so that I can recover my account.                                 | 3      |
| AU-6 | As a user, I want to confirm a password reset with the OTP and set a new password, so that I regain access.                    | 3      |
| AU-7 | As a user, I want to delete my account, so that all my data is removed.                                                        | 4      |


### Acceptance Criteria

**AU-1: Register**

- Email and username must be unique (409 if duplicate).
- Password is hashed before storage.
- Returns valid access + refresh tokens on success.
- Missing/invalid fields return 400.

**AU-2: Login**

- Returns 401 for wrong email or password.
- Returns access + refresh tokens on success.
- Passwords are compared via secure hash.

**AU-3: Logout**

- Refresh token is revoked server-side.
- Access token remains valid until expiry (stateless JWT).
- Returns 401 if no valid auth header.

**AU-4: Refresh**

- Returns 401 if refresh token is expired or revoked.
- Issues a new access token.
- Refresh token itself is NOT rotated (single-use rotation is optional).

**AU-5: Password Reset Request**

- Creates a `password_reset_tokens` row with expiry.
- Sends email with OTP/link.
- Returns 200 even if email doesn't exist (prevents enumeration).

**AU-6: Password Reset Confirm**

- Returns 400 if token is expired, already used, or invalid.
- Marks token as `used = true`.
- Updates password hash.

**AU-7: Delete Account**

- Removes user row.
- Optionally emits `user.deleted` event for cleanup in other services.
- Returns 401 if not authenticated.

### Scenarios


| Scenario                                 | Expected                              |
| ---------------------------------------- | ------------------------------------- |
| Register with duplicate email            | 409 Conflict                          |
| Register with duplicate username         | 409 Conflict                          |
| Register with missing password           | 400 Bad Request                       |
| Login with wrong password                | 401 Unauthorized                      |
| Login with non-existent email            | 401 Unauthorized                      |
| Refresh with revoked token               | 401 Unauthorized                      |
| Refresh with expired token               | 401 Unauthorized                      |
| Password reset for non-existent email    | 200 OK (no leak)                      |
| Password reset with expired OTP          | 400 Bad Request                       |
| Password reset with already-used OTP     | 400 Bad Request                       |
| Delete account when not logged in        | 401 Unauthorized                      |
| Concurrent registrations with same email | Only one succeeds (unique constraint) |


---

## Epic 2: Profile Management

**Total: 4 stories, 5 points**

### Stories


| ID   | Story                                                                                                                          | Points |
| ---- | ------------------------------------------------------------------------------------------------------------------------------ | ------ |
| PR-1 | As a user, I want to view my own profile, so that I can see all my account details.                                            | 1      |
| PR-2 | As a user, I want to view another user's public profile, so that I can learn about them.                                       | 1      |
| PR-3 | As a user, I want to update my username, avatar, or bio, so that I can personalize my profile.                                 | 2      |
| PR-4 | As a user, I want my updated username and avatar to appear on my future posts and comments, so that my identity is consistent. | 1      |


### Acceptance Criteria

**PR-1: View Own Profile**

- Returns all fields: id, email, uname, avatar, bio, role, created_at.
- Requires auth.

**PR-2: View Public Profile**

- Returns only public fields: uname, avatar, bio, role.
- Returns 404 if user_id doesn't exist.

**PR-3: Update Profile**

- All fields optional in request; only provided fields are updated.
- Username change must check uniqueness (409 if taken).
- Returns updated profile.

**PR-4: Denormalized Identity**

- Gateway injects current uname/avatar from JWT into headers.
- Future posts/comments use these headers (no stale data on new writes).
- Existing posts/comments retain old values (accepted staleness for MVP).

### Scenarios


| Scenario                             | Expected                       |
| ------------------------------------ | ------------------------------ |
| View profile when not logged in      | 401 Unauthorized               |
| View non-existent user's profile     | 404 Not Found                  |
| Update username to one already taken | 409 Conflict                   |
| Update with empty body               | 200 OK (no changes)            |
| Update avatar to invalid URL         | 400 Bad Request (if validated) |


---

## Epic 3: Post Management

**Total: 5 stories, 21 points**

### Stories


| ID   | Story                                                                                                                                       | Points |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| PO-1 | As a user, I want to create a post with a title, content, optional image, and tags, so that I can start a discussion.                       | 5      |
| PO-2 | As a user, I want to view a post's full details, so that I can read and engage with the discussion.                                         | 3      |
| PO-3 | As a user, I want to delete my own post, so that I can remove content I no longer want public.                                              | 3      |
| PO-4 | As a user, I want the feed listing to show posts with their static like and comment counts, so that I can gauge engagement before clicking. | 8      |


### Acceptance Criteria

**PO-1: Create Post**

- Title is required (400 if missing).
- tag_names[] is resolved by Feed Service via `post.created` event (auto-creates new tags).
- Emits `post.created` and `post.viewed` events.
- Returns created post with id.
- Denormalized `author_uname`, `author_avatar` from gateway headers.

**PO-2: View Post**

- Returns full post object including `liked_by_me` boolean.
- Inserts `post_views` row if first view by this user.
- Emits `post.viewed` event for feed "unseen" filtering.
- Returns 404 if post doesn't exist.

**PO-3: Edit Post**

- Only owner, mod, or admin can edit.
- 403 if not owner and role is `user`.
- Updates `updated_at` timestamp.

**PO-4: Delete Post**

- Only owner, mod, or admin can delete.
- Emits `post.deleted` event (Feed Service cleans up post_tags, decrements tag counts).
- Returns 404 if post doesn't exist.

**PO-5: Feed Listing**

- `like_count` and `comment_count` are served from denormalized columns (no joins).
- Pagination via cursor.

### Scenarios


| Scenario                           | Expected                                     |
| ---------------------------------- | -------------------------------------------- |
| Create post without title          | 400 Bad Request                              |
| Create post with non-existent tags | Tags auto-created by Feed Service            |
| Create post with > 10 tags         | 400 Bad Request (application limit)          |
| View non-existent post             | 404 Not Found                                |
| View same post twice               | post_views row not duplicated (composite PK) |
| Edit post as non-owner non-mod     | 403 Forbidden                                |
| Edit post as moderator             | 200 OK                                       |
| Delete post as owner               | 200 OK + post.deleted event                  |
| Delete already-deleted post        | 404 Not Found                                |
| Create post when not logged in     | 401 Unauthorized                             |


---

## Epic 4: Comment System

**Total: 6 stories, 31 points**

### Stories


| ID   | Story                                                                                                     | Points |
| ---- | --------------------------------------------------------------------------------------------------------- | ------ |
| CO-1 | As a user, I want to post a top-level comment on a post, so that I can share my thoughts.                 | 5      |
| CO-2 | As a user, I want to reply to a specific comment, so that I can have a threaded discussion.               | 8      |
| CO-3 | As a user, I want to view top-level comments on a post with pagination, so that I can browse discussions. | 3      |
| CO-4 | As a user, I want to click "load more replies" on a comment, so that I can see deeper threads on demand.  | 3      |
| CO-5 | As a user, I want to edit my own comment, so that I can correct what I wrote.                             | 2      |
| CO-6 | As a user, I want to delete my own comment, so that I can remove something I regret.                      | 2      |


### Acceptance Criteria

**CO-1: Top-Level Comment**

- Content is required (400 if empty).
- Increments `post.comment_count`.
- Emits `comment.created` event (tag_weight +5, notification for post author).
- WS broadcast to post room.
- Denormalized author fields from gateway headers.

**CO-2: Reply to Comment**

- parent_id is set to the comment being replied to.
- Increments `post.comment_count`.
- Content Service checks thread room presence:
  - Parent author in room: skip notification (they see it live).
  - Parent author not in room: include in `notify_user_ids[]`.
- WS broadcast to thread room.
- Emits `comment.created` event with `notify_user_ids[]`.
- Parses @mentions from content and adds mentioned users to `notify_user_ids[]`.

**CO-3: Get Top-Level Comments**

- Returns comments where `parent_id = null`, paginated by cursor.
- Each comment includes `reply_count` for "N replies" indicator.

**CO-4: Load More Replies**

- Returns comments where `parent_id = comment_id`, paginated.
- Supports multi-level nesting (replies to replies).

**CO-5: Edit Comment**

- Owner, mod, or admin only.
- Updates `updated_at`.

**CO-6: Delete Comment**

- Owner, mod, or admin only.
- If comment has children, content can be replaced with "[deleted]" instead of hard-deleting (preserves thread structure).

### Scenarios


| Scenario                                                  | Expected                                                           |
| --------------------------------------------------------- | ------------------------------------------------------------------ |
| Comment on non-existent post                              | 404 Not Found                                                      |
| Reply to non-existent comment                             | 404 Not Found                                                      |
| Comment with empty content                                | 400 Bad Request                                                    |
| Reply when parent author is in thread room                | No notification (live inline)                                      |
| Reply when parent author is NOT in thread room but online | Notification persisted + pushed live via ws/notifications          |
| Reply when parent author is offline                       | Notification persisted, visible in Activity tab on next visit      |
| Comment with @mention of existing user                    | Mention notification created                                       |
| Comment with @mention of non-existent user                | @mention ignored silently                                          |
| Edit comment as non-owner                                 | 403 Forbidden                                                      |
| Delete comment with nested replies                        | Content replaced with "[deleted]", children preserved              |
| Two users reply to same comment simultaneously            | Both succeed (separate rows, comment_count incremented atomically) |
| Comment on own post                                       | comment_count incremented, no self-notification                    |


---

## Epic 5: Likes

**Total: 4 stories, 12 points**

### Stories


| ID   | Story                                                                      | Points |
| ---- | -------------------------------------------------------------------------- | ------ |
| LK-1 | As a user, I want to like a post, so that I can show appreciation.         | 3      |
| LK-2 | As a user, I want to unlike a post, so that I can change my mind.          | 3      |
| LK-3 | As a user, I want to like a comment, so that I can upvote helpful replies. | 3      |
| LK-4 | As a user, I want to unlike a comment, so that I can change my mind.       | 3      |


### Acceptance Criteria

**LK-1: Like Post**

- Composite PK `(post_id, user_id)` prevents duplicate likes.
- Increments `post.like_count` atomically.
- Emits `post.liked` event (Feed Service: tag_weight +1).
- WS broadcast: `{ event: "like_update", like_count: N }` to post room.
- Returns new like_count.

**LK-2: Unlike Post**

- Returns 404 if user hasn't liked the post.
- Decrements `post.like_count` atomically.
- Emits `post.unliked` event.
- WS broadcast with updated like_count.

**LK-3: Like Comment**

- Composite PK `(comment_id, user_id)` prevents duplicates.
- Increments `comment.like_count` atomically.
- Returns new like_count.

**LK-4: Unlike Comment**

- Returns 404 if user hasn't liked the comment.
- Decrements `comment.like_count` atomically.

### Scenarios


| Scenario                                                   | Expected                                          |
| ---------------------------------------------------------- | ------------------------------------------------- |
| Like a post already liked                                  | 409 Conflict (composite PK violation)             |
| Unlike a post not previously liked                         | 404 Not Found                                     |
| Like a non-existent post                                   | 404 Not Found                                     |
| Like a comment already liked                               | 409 Conflict                                      |
| Unlike a comment not liked                                 | 404 Not Found                                     |
| Two users like same post simultaneously                    | Both succeed (separate PK rows, like_count += 2)  |
| Like own post                                              | Allowed (no restriction)                          |
| like_count goes negative (unlike without like due to race) | Prevented by checking row exists before decrement |
| 1000 concurrent likes on same post                         | Atomic increment handles correctly                |


---

## Epic 6: Tag and Feed

**Total: 6 stories, 34 points**

### Stories


| ID   | Story                                                                                                           | Points |
| ---- | --------------------------------------------------------------------------------------------------------------- | ------ |
| TF-1 | As a user, I want to see a personalized feed based on my interests, so that I discover relevant posts.          | 13     |
| TF-2 | As a user, I want tags to autocomplete when I create a post, so that I can use existing tags easily.            | 2      |
| TF-3 | As a mod/admin, I want to create a tag manually, so that I can pre-populate topic categories.                   | 2      |
| TF-4 | As a mod/admin, I want to delete a tag, so that I can remove irrelevant or spam tags.                           | 3      |
| TF-5 | As a user, I want to browse all posts under a specific tag, so that I can explore topics.                       | 5      |
| TF-6 | As a user, I want my like and comment actions to update my tag preferences, so that my feed improves over time. | 8      |


### Acceptance Criteria

**TF-1: Personalized Feed**

- Queries top-K tags from `tag_weight` for the user.
- Finds unseen post_ids from `post_tags` excluding `post_views`.
- Calls Content Service `/internal/posts/batch` for post details.
- Returns shuffled results with recency bias, paginated.
- Cold start (new user, no tag_weights): falls back to popular posts (by like_count).

**TF-2: Tag Autocomplete**

- `GET /tags?q=pyt&limit=10` returns matching tags sorted by `post_count DESC`.
- Prefix match on tag name.

**TF-3: Create Tag**

- Mod/admin only (403 for regular users).
- Tag name must be unique (409 if duplicate).
- Tags are also auto-created from `post.created` events.

**TF-4: Delete Tag**

- Mod/admin only.
- Cascades: removes all `post_tags` and `tag_weight` rows for that tag.
- Decrements nothing references the tag afterward.

**TF-5: Browse by Tag**

- Returns posts tagged with the given tag, paginated.
- Calls Content Service `/internal/posts/batch` for hydration.
- Returns 404 if tag doesn't exist.

**TF-6: Implicit Preference Learning**

- Like a post: +1 weight on each of that post's tags for the user.
- Comment on a post: +5 weight on each of that post's tags.
- Updated asynchronously via `content.events` consumer.
- UPSERT into `tag_weight` (create row if first interaction with that tag).

### Scenarios


| Scenario                                 | Expected                                                     |
| ---------------------------------------- | ------------------------------------------------------------ |
| New user with no interactions opens feed | Fallback to popular/recent posts                             |
| User with tag_weights opens feed         | Personalized by top-K tags                                   |
| Feed page 2 (cursor pagination)          | Returns next batch of unseen posts                           |
| All posts already viewed                 | Empty feed (or "You're all caught up")                       |
| Tag autocomplete with no matches         | Empty array                                                  |
| Create duplicate tag                     | 409 Conflict                                                 |
| Delete tag with 100 associated posts     | Cascades: 100 post_tags rows removed                         |
| Regular user tries to create tag         | 403 Forbidden                                                |
| Browse tag with zero posts               | Empty posts array                                            |
| User likes 500 posts with "python" tag   | Weight grows to 500 (cap/decay is application-level concern) |
| Post created with brand-new tag name     | Feed Service auto-creates the tag                            |
| Content Service down during feed build   | `/internal/posts/batch` fails, Feed Service returns 503      |


---

## Epic 7: Notifications

**Total: 5 stories, 18 points**

### Stories


| ID   | Story                                                                                                                | Points |
| ---- | -------------------------------------------------------------------------------------------------------------------- | ------ |
| NO-1 | As a user, I want to see my notifications in an Activity tab, so that I know when someone interacts with my content. | 3      |
| NO-2 | As a user, I want to see an unread notification count badge, so that I know at a glance if there's something new.    | 1      |
| NO-3 | As a user, I want to mark a notification as read, so that it stops appearing as new.                                 | 2      |
| NO-4 | As a user, I want to mark all notifications as read at once, so that I can clear the badge quickly.                  | 2      |
| NO-5 | As a user, I want to receive live notification pushes when I'm online, so that I see updates immediately.            | 8      |


### Acceptance Criteria

**NO-1: Activity Tab**

- Paginated, newest first.
- Each notification shows: actor_uname, type (reply/mention/like), post_title, timestamp.
- Clicking a notification navigates to the relevant post/comment.

**NO-2: Unread Count**

- Returns `{ count: N }` for unread notifications.
- Used for badge on the notification bell icon.

**NO-3: Mark Single as Read**

- Returns 404 if notification doesn't exist or belongs to another user.
- Sets `is_read = true`.

**NO-4: Mark All as Read**

- Bulk update: `SET is_read = true WHERE user_id = ? AND is_read = false`.

**NO-5: Live Notification Push**

- Notification Service always persists the row.
- If recipient has open `ws/notifications`, push payload immediately.
- If recipient is offline, row waits in DB for Activity tab.
- If recipient is in the relevant thread room (seeing reply live), notification is suppressed (not created).

### Scenarios


| Scenario                                               | Expected                                          |
| ------------------------------------------------------ | ------------------------------------------------- |
| User with zero notifications opens Activity tab        | Empty list                                        |
| User with 50 notifications, page 1 limit 20            | Returns first 20, next_cursor for page 2          |
| Mark notification that belongs to another user as read | 404 Not Found                                     |
| Mark already-read notification as read again           | 200 OK (idempotent, no error)                     |
| Mark all as read when no unread exist                  | 200 OK (no-op)                                    |
| User online, someone replies to their comment          | Live push via ws/notifications + row persisted    |
| User offline, someone replies to their comment         | Row persisted, visible on next visit              |
| User in thread room, someone replies in that thread    | No notification (suppressed, they see it inline)  |
| User NOT in thread room but online                     | Notification persisted + pushed live              |
| 10 people reply to user's post in 1 second             | 10 notifications created, 10 WS pushes if online  |
| Notification for @mention in comment                   | Type = "mention", includes post_id and comment_id |


---

## Epic 8: Real-Time / WebSocket

**Total: 4 stories, 34 points**

### Stories


| ID   | Story                                                                                                                     | Points |
| ---- | ------------------------------------------------------------------------------------------------------------------------- | ------ |
| RT-1 | As a user viewing a post, I want to see like count updates in real time, so that the page feels alive.                    | 8      |
| RT-2 | As a user viewing a post, I want new top-level comments to appear automatically, so that I don't have to refresh.         | 8      |
| RT-3 | As a user in a comment thread, I want to see new replies appear live, so that threaded conversations feel instant.        | 13     |
| RT-4 | As a user, I want a persistent notification WebSocket that stays open across page navigations, so that I get live alerts. | 5      |


### Acceptance Criteria

**RT-1: Live Like Count**

- Client joins `ws/post/{post_id}` when opening a post.
- On any like/unlike, Content Service broadcasts `{ event: "like_update", like_count: N }`.
- All connected clients in the room see updated count.

**RT-2: Live Top-Level Comments**

- Same post room `ws/post/{post_id}`.
- On new top-level comment, broadcasts `{ event: "new_comment", comment: {...} }`.
- Client appends comment to the comment list.

**RT-3: Live Thread Replies**

- Client joins `ws/post/{post_id}/thread/{comment_id}` when expanding a thread.
- On new reply within that thread, broadcasts `{ event: "new_reply", comment: {...} }`.
- Presence check: if parent author is in the room, suppress their notification.
- Anyone can join the thread room (not restricted to participants).

**RT-4: Notification WebSocket**

- Client opens `ws/notifications` on page load (global, not per-post).
- Receives `{ event: "notification", data: {...} }` and `{ event: "unread_count", count: N }`.
- Reconnects automatically on disconnect.
- Auth via token in query param or first message.

### Scenarios


| Scenario                                                  | Expected                                                 |
| --------------------------------------------------------- | -------------------------------------------------------- |
| 50 users viewing same post, one likes it                  | All 50 receive like_update via WS                        |
| User opens post, then navigates away                      | WS connection closed, removed from room                  |
| User opens post in two browser tabs                       | Both tabs join room, both receive events                 |
| WS connection drops mid-session                           | Client reconnects, re-joins room                         |
| User joins thread room then leaves                        | Removed from room, future replies generate notifications |
| JWT expires while WS is open                              | Connection should be closed, client re-authenticates     |
| Server restarts (Content Service)                         | All WS connections drop, clients reconnect               |
| 1000 concurrent users in one post room                    | Broadcast to all 1000 on each like/comment               |
| Notification WS receives event while tab is in background | Event queued in client, shown when tab becomes active    |


---

## Epic 9: Admin and Moderation

**Total: 5 stories, 17 points**

### Stories


| ID   | Story                                                                                                                                                         | Points |
| ---- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------ |
| AM-1 | As an admin, I want to list all users with pagination, so that I can manage the user base.                                                                    | 2      |
| AM-2 | As an admin, I want to promote or demote a user's role, so that I can assign moderators.                                                                      | 3      |
| AM-3 | As a moderator, I want to edit any post or comment, so that I can fix rule-violating content.                                                                 | 3      |
| AM-4 | As a moderator, I want to delete any post or comment, so that I can remove harmful content.                                                                   | 3      |
| AM-5 | As an admin, I want all my actions to be subject to the same feed and notification system, so that I have a normal user experience alongside my admin duties. | 5      |


### Acceptance Criteria

**AM-1: List Users**

- Admin only (403 for mod/user).
- Paginated with cursor.
- Returns: id, uname, email, role, created_at.

**AM-2: Promote/Demote**

- Admin only.
- Valid roles: admin, mod, user.
- Cannot demote yourself (prevents lockout).
- Returns updated user with new role.

**AM-3: Mod Edit Content**

- Mod or admin can edit any post (`PATCH /posts/{id}`) or comment (`PATCH /comments/{id}`).
- Permission check: `X-User-Role` in `[mod, admin]`.
- Updates `updated_at`.

**AM-4: Mod Delete Content**

- Mod or admin can delete any post or comment.
- Same permission check.
- Delete post emits `post.deleted` event.

**AM-5: Admin as User**

- Admin sees the same personalized feed based on tag_weight.
- Admin receives notifications like any user.
- Admin can like, comment, and create posts normally.

### Scenarios


| Scenario                                         | Expected                                     |
| ------------------------------------------------ | -------------------------------------------- |
| Regular user tries to list all users             | 403 Forbidden                                |
| Mod tries to list all users                      | 403 Forbidden (admin only)                   |
| Admin promotes user to mod                       | 200 OK, user.role = "mod"                    |
| Admin demotes themselves                         | 400 Bad Request (self-demotion blocked)      |
| Admin promotes user to invalid role "superadmin" | 400 Bad Request                              |
| Mod edits another user's post                    | 200 OK                                       |
| Mod edits another mod's post                     | 200 OK (mods can edit any content)           |
| User tries to delete someone else's post         | 403 Forbidden                                |
| Mod deletes post with many comments              | Post deleted, comments orphaned (or cascade) |
| Admin uses the feed                              | Personalized feed based on their tag_weights |


---

## Summary


| Epic                          | Stories | Points  |
| ----------------------------- | ------- | ------- |
| 1. Authentication and Account | 7       | 19      |
| 2. Profile Management         | 4       | 5       |
| 3. Post Management            | 5       | 21      |
| 4. Comment System             | 6       | 31      |
| 5. Likes                      | 4       | 12      |
| 6. Tag and Feed               | 6       | 34      |
| 7. Notifications              | 5       | 18      |
| 8. Real-Time / WebSocket      | 4       | 34      |
| 9. Admin and Moderation       | 5       | 17      |
| **Total**                     | **46**  | **191** |


### Suggested Sprint Priorities

1. **Sprint 1** (foundation): Epics 1 + 2 (Auth + Profile) = 24 pts
2. **Sprint 2** (core content): Epics 3 + 5 (Posts + Likes) = 33 pts
3. **Sprint 3** (discussions): Epic 4 (Comments) = 31 pts
4. **Sprint 4** (discovery): Epic 6 (Tags + Feed) = 34 pts
5. **Sprint 5** (engagement): Epics 7 + 8 (Notifications + Real-Time) = 52 pts
6. **Sprint 6** (governance): Epic 9 (Admin + Moderation) = 17 pts

