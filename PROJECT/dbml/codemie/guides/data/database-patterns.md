# Database Patterns Guide

**Source**: design.md
**Stack**: PostgreSQL + SQLAlchemy + Alembic | **Package Manager**: uv
**Rule**: One database per service — no shared tables, no cross-service DB-level FKs.

---

## Per-Service Database Ownership

| Service              | Tables                                                         |
| -------------------- | -------------------------------------------------------------- |
| user-service         | `users`, `password_reset_tokens`                               |
| content-service      | `post`, `comment`, `post_likes`, `comment_likes`, `post_views` |
| feed-service         | `tags`, `post_tags`, `tag_weight`, `feed_post_seen`            |
| notification-service | `notifications`                                                |

Cross-service IDs are **logical references** (plain int columns), never DB-level FKs.

---

## 1. User Service Schema

```dbml
Table users {
  id           int       [pk, increment]
  email        varchar   [unique, not null]
  uname        varchar   [unique, not null]
  password     varchar   [not null]       // bcrypt hash
  avatar       varchar
  bio          text
  role         varchar   [not null, default: 'user']  // admin | mod | user
  created_at   timestamp
}
```

---

## 2. Content Service Schema

```dbml
Table post {
  id             int       [pk, increment]
  user_id        int       [not null]              // logical ref to users.id
  author_uname   varchar   [not null]              // denormalized from User Service
  author_avatar  varchar                           // denormalized from User Service
  title          varchar   [not null]
  content        text
  image_link     varchar
  like_count     int       [not null, default: 0]  // denormalized from post_likes
  comment_count  int       [not null, default: 0]  // denormalized from comment
  tag_names      varchar   // space-separated, for FTS trigger input
  search_vector  tsvector  // GIN index; maintained by DB trigger
  created_at     timestamp
  updated_at     timestamp
}

Table comment {
  id             int       [pk, increment]
  post_id        int       [not null, ref: > post.id]
  parent_id      int       [ref: > comment.id]     // null = top-level
  user_id        int       [not null]
  author_uname   varchar   [not null]              // denormalized
  author_avatar  varchar                           // denormalized
  content        text      [not null]
  like_count     int       [not null, default: 0]  // denormalized
  created_at     timestamp
  updated_at     timestamp
}

Table post_likes {
  post_id      int  [not null, ref: > post.id]
  user_id      int  [not null]
  created_at   timestamp
  indexes { (post_id, user_id) [pk] }
}

Table comment_likes {
  comment_id   int  [not null, ref: > comment.id]
  user_id      int  [not null]
  created_at   timestamp
  indexes { (comment_id, user_id) [pk] }
}

Table post_views {
  post_id    int  [not null, ref: > post.id]
  user_id    int  [not null]
  viewed_at  timestamp
  indexes { (post_id, user_id) [pk] }
}
```

---

## 3. Feed Service Schema

```dbml
Table tags {
  id          int      [pk, increment]
  name        varchar  [unique, not null]
  post_count  int      [not null, default: 0]  // denormalized; updated via content.events
  created_at  timestamp
}

Table post_tags {
  post_id  int  [not null]              // logical ref to content-service post.id
  tag_id   int  [not null, ref: > tags.id]
  indexes {
    (post_id, tag_id) [pk]
    (tag_id, post_id)                   // reverse index for feed queries
  }
}

Table tag_weight {
  user_id     int    [not null]         // logical ref to users.id
  tag_id      int    [not null, ref: > tags.id]
  weight      float  [not null, default: 0]  // like: +1, comment: +5
  updated_at  timestamp
  indexes {
    (user_id, tag_id) [pk]
    (user_id, weight) [name: 'idx_user_top_tags']  // fast top-K for feed
  }
}

Table feed_post_seen {
  user_id  int  [not null]
  post_id  int  [not null]             // logical ref to content-service post.id
  seen_at  timestamp
  indexes { (user_id, post_id) [pk] }
}
```

---

## 4. Notification Service Schema

```dbml
Table notifications {
  id           int      [pk, increment]
  user_id      int      [not null]          // recipient; logical ref to users.id
  actor_id     int      [not null]          // who triggered it
  actor_uname  varchar  [not null]          // denormalized from event payload
  type         varchar  [not null]          // reply|mention|like|comment|post_deleted|comment_deleted
  post_id      int                          // logical ref to content-service post.id
  post_title   varchar                      // denormalized from event payload
  comment_id   int                          // set for reply/mention; null for post-level like
  is_read      boolean  [not null, default: false]
  created_at   timestamp

  indexes {
    (user_id, created_at)   // Activity tab pagination
    (user_id, is_read)      // unread count badge
  }
}
```

---

## Denormalization Strategy

Fields are copied at **write time** to avoid cross-service reads at query time.

| Denormalized Field | Stored In | Source | When Populated |
|--------------------|-----------|--------|----------------|
| `author_uname`, `author_avatar` | `post`, `comment` | `X-User-Name`, `X-User-Avatar` headers | On create |
| `post.like_count` | `post` | count of `post_likes` rows | Atomic increment/decrement |
| `post.comment_count` | `post` | count of `comment` rows | Atomic increment/decrement |
| `comment.like_count` | `comment` | count of `comment_likes` rows | Atomic increment/decrement |
| `tags.post_count` | `tags` | count of `post_tags` rows | Via `content.events` consumer |
| `actor_uname`, `post_title` | `notifications` | Redis event payload | On event consume |

**Staleness trade-off**: User profile changes (uname/avatar) will leave stale copies in content/notification tables. Acceptable for MVP. Future option: emit `user.profile_updated` event.

---

## Full-Text Search (PostgreSQL FTS)

Applied in content-service `post` table.

```sql
-- DB trigger maintains search_vector on INSERT/UPDATE of post
NEW.search_vector :=
    setweight(to_tsvector('english', NEW.title),     'A') ||
    setweight(to_tsvector('english', NEW.tag_names), 'A') ||
    setweight(to_tsvector('english', NEW.content),   'B');

-- GIN index for fast FTS
CREATE INDEX idx_post_fts ON post USING GIN (search_vector);

-- Query pattern (GET /posts/search?q=rust+async)
SELECT *, ts_rank(search_vector, query) AS rank
FROM post, to_tsquery('english', :q) AS query
WHERE search_vector @@ query
ORDER BY rank DESC, created_at DESC;
```

- Title + tag matches (weight A) rank higher than content matches (weight B)
- Returns 400 if `?q` is empty

---

## Composite Primary Keys

Used for junction tables to enforce one-per-user uniqueness at the DB level.

| Table | Composite PK | Enforces |
|-------|-------------|---------|
| `post_likes` | `(post_id, user_id)` | One like per user per post |
| `comment_likes` | `(comment_id, user_id)` | One like per user per comment |
| `post_views` | `(post_id, user_id)` | Track unique views |
| `post_tags` | `(post_id, tag_id)` | No duplicate tag on a post |
| `tag_weight` | `(user_id, tag_id)` | One weight row per user-tag pair |
| `feed_post_seen` | `(user_id, post_id)` | No re-showing served posts |

---

## Migrations

Each service has its own Alembic directory. Never share migration history across services.

```bash
# Inside a service directory
uv run alembic revision --autogenerate -m "add post table"
uv run alembic upgrade head
```

---

## Key Rules

| ✅ DO | ❌ DON'T |
|-------|---------|
| Store user context from `X-User-*` headers at write time | Query user-service at read time for username/avatar |
| Use composite PKs for junction tables | Use separate auto-increment PK + unique constraint on junctions |
| Use DB trigger for `search_vector` maintenance | Update `search_vector` in application code |
| Use atomic `UPDATE post SET like_count = like_count + 1` | Read count then write (race condition) |
| Migrations per service, never shared | Run Alembic with a shared DB URL across services |
