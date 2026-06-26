# MuComics — Revised Technical Architecture

**Revision context:** This supersedes the original architecture report. The single biggest change is that MuComics does **not** stand up its own database, user system, or authentication. It runs on **µLearn's existing database**, reuses µLearn's identity, roles, and email infrastructure, and owns only its own comic-domain tables. Every section below is written to align with the current µLearn schema and conventions.

---

## 1. Project Overview

MuComics is a comic/manga publishing and collaboration platform inside the µLearn ecosystem. It supports comic creation and publishing, chapter management, image storage and delivery, reader engagement (likes, comments, bookmarks, reading progress), and team collaboration between writers and artists.

What changed in scope: MuComics is now framed as a **module on top of µLearn**, not a standalone product. Users, authentication, roles, password reset, OTP, and email are all provided by µLearn. MuComics adds only the comic domain.

---

## 2. µLearn Integration Model (new — read first)

This section governs the rest of the document.

### 2.1 What MuComics reuses vs. owns

| Concern | Owner | Notes |
|---|---|---|
| Users / identity | **µLearn** (`user` table) | MuComics never creates or migrates users. References `user(id)`. |
| Login / signup / refresh | **µLearn** | MuComics validates µLearn-issued JWTs; it has no login of its own. |
| Password reset / OTP | **µLearn** (`forgot_password`, `otp_verification`) | Do not rebuild. |
| Email / SMTP | **µLearn** | Reuse µLearn's mail/notification path; no parallel SMTP stack. |
| Roles / RBAC | **µLearn** (`role`, `user_role_link`) | MuComics roles are rows in `role`, granted via `user_role_link`. |
| Karma / wallet | **µLearn** (`wallet`, `karma_activity_log`) | Optional integration (see 9.2). |
| Comics, chapters, pages, likes, comments, bookmarks, progress | **MuComics** | The only tables MuComics creates. |

### 2.2 Database

MuComics tables live in the **same MySQL database** as µLearn. This is a deliberate, stated constraint (shared DB), so MuComics tables can declare real foreign keys to `user(id)` and enforce integrity at the database level.

> Correction from the original report: the stack is **MySQL**, not PostgreSQL, and there is no separate Supabase/RDS instance. Drop the PostgreSQL assumption everywhere.

### 2.3 Conventions MuComics must follow

Mirroring the existing µLearn schema so MuComics tables are indistinguishable from native ones:

- **Primary keys:** `VARCHAR(36)` UUID, application-generated (`default uuid4`).
- **Audit columns on every entity table:** `created_by`, `created_at`, `updated_by`, `updated_at`, where `created_by`/`updated_by` are `VARCHAR(36)` FKs to `user(id)`.
- **Soft delete** where content is user-visible: `deleted_at DATETIME NULL`, `deleted_by VARCHAR(36) NULL` (FK to `user`). µLearn users are themselves soft-deleted, so hard cascades rarely fire in practice.
- **Junction tables** named `<a>_<b>_link` with a surrogate `id` PK plus `created_by`/`created_at` (matching `user_role_link`, `user_ig_link`).
- **Constraint naming:** `fk_<table>_ref_<column>`.
- **Table names:** snake_case, singular. MuComics tables are prefixed `comic`/`chapter` so they group cleanly in the shared schema.

### 2.4 Deployment shape — two options

- **Option A (recommended): MuComics as a Django app inside the µLearn backend.** Reuses µLearn's ORM models, auth middleware, role checks, and DB connection directly. FK integrity to `user`/`role` is guaranteed. Lowest friction.
- **Option B: separate MuComics service against the shared MySQL.** It declares `user`, `role`, `user_role_link` as **unmanaged** read-only models (`managed = False`) and owns/migrates only `comic*` tables. Use this only if the team wants MuComics deployed independently. Tradeoff: shared-DB coupling — mitigate by never writing to µLearn tables.

Either way, MuComics writes **only** its own tables and treats `user`, `role`, `wallet`, etc. as read-only.

---

## 3. System Architecture

```
User Browser
    │
Next.js Frontend
    │  (REST, JWT issued by µLearn)
MuComics API  ──────────────┐
    │                       │ validates JWT (shared µLearn secret/JWKS)
    │                       │ reads roles from user_role_link
    ▼                       ▼
Shared µLearn MySQL    µLearn auth / email / notification
 ├─ user, role, … (read-only to MuComics)
 └─ comic, chapter, chapter_page, … (owned by MuComics)
    │
AWS S3 (cover images, chapter pages, banners)  ← served via CDN
```

Layers: Next.js frontend; MuComics API (Django + DRF, as app or service); shared µLearn MySQL; S3 + CDN for media; µLearn for identity/auth/email.

---

## 4. Tech Stack

**Frontend** (largely unchanged): Next.js, React, TypeScript, Tailwind + shadcn/ui, a typed fetch layer, Zustand for local UI state, TanStack Query for server state (preferred over hand-rolled Axios caching).

**Backend:** Python, Django, Django REST Framework. **MySQL** (shared µLearn DB). Auth = JWT validation against µLearn, not a local user store.

**Storage:** AWS S3 + CDN. **Store object keys, not full URLs** (see §7).

---

## 5. Data Model (MuComics-owned tables)

Normalization and integrity fixes from the review are folded in: genres are a many-to-many (not a string column), every interaction has a uniqueness guard, statuses/roles are constrained values, and counts are denormalized for read performance.

```sql
-- Comics ------------------------------------------------------------
CREATE TABLE comic
(
    id              VARCHAR(36) PRIMARY KEY NOT NULL,
    title           VARCHAR(150)            NOT NULL,
    slug            VARCHAR(180) UNIQUE KEY NOT NULL,
    description     TEXT,
    cover_image_key VARCHAR(255),                 -- S3 key, not URL
    status          VARCHAR(20) DEFAULT 'draft' NOT NULL,  -- draft|published|archived
    like_count      INT DEFAULT 0           NOT NULL,      -- denormalized
    comment_count   INT DEFAULT 0           NOT NULL,
    bookmark_count  INT DEFAULT 0           NOT NULL,
    published_at    DATETIME,
    deleted_at      DATETIME,
    deleted_by      VARCHAR(36),
    updated_by      VARCHAR(36)             NOT NULL,
    updated_at      DATETIME                NOT NULL,
    created_by      VARCHAR(36)             NOT NULL,       -- owning creator
    created_at      DATETIME                NOT NULL,
    CONSTRAINT fk_comic_ref_deleted_by FOREIGN KEY (deleted_by) REFERENCES user (id) ON DELETE SET NULL,
    CONSTRAINT fk_comic_ref_updated_by FOREIGN KEY (updated_by) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_comic_status_created (status, created_at),
    INDEX idx_comic_created_by (created_by)
);

-- Genres (M:N) ------------------------------------------------------
CREATE TABLE genre
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    name       VARCHAR(75) UNIQUE KEY  NOT NULL,
    slug       VARCHAR(90) UNIQUE KEY  NOT NULL,
    updated_by VARCHAR(36)             NOT NULL,
    updated_at DATETIME                NOT NULL,
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT fk_genre_ref_updated_by FOREIGN KEY (updated_by) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_genre_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE
);

CREATE TABLE comic_genre_link
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id   VARCHAR(36)             NOT NULL,
    genre_id   VARCHAR(36)             NOT NULL,
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT uq_comic_genre UNIQUE (comic_id, genre_id),
    CONSTRAINT fk_comic_genre_link_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_genre_link_ref_genre_id FOREIGN KEY (genre_id) REFERENCES genre (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_genre_link_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE
);

-- Contributors (writer/artist/colorist/editor) ----------------------
CREATE TABLE comic_contributor_link
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id   VARCHAR(36)             NOT NULL,
    user_id    VARCHAR(36)             NOT NULL,
    role       VARCHAR(20)             NOT NULL,  -- writer|artist|colorist|editor
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT uq_comic_contributor UNIQUE (comic_id, user_id, role),
    CONSTRAINT fk_comic_contributor_link_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_contributor_link_ref_user_id FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_contributor_link_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE
);

-- Chapters ----------------------------------------------------------
CREATE TABLE chapter
(
    id             VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id       VARCHAR(36)             NOT NULL,
    chapter_number INT                     NOT NULL,
    title          VARCHAR(150),
    status         VARCHAR(20) DEFAULT 'draft' NOT NULL,
    release_date   DATETIME,
    deleted_at     DATETIME,
    deleted_by     VARCHAR(36),
    updated_by     VARCHAR(36)             NOT NULL,
    updated_at     DATETIME                NOT NULL,
    created_by     VARCHAR(36)             NOT NULL,
    created_at     DATETIME                NOT NULL,
    CONSTRAINT uq_chapter_number UNIQUE (comic_id, chapter_number),
    CONSTRAINT fk_chapter_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_chapter_ref_deleted_by FOREIGN KEY (deleted_by) REFERENCES user (id) ON DELETE SET NULL,
    CONSTRAINT fk_chapter_ref_updated_by FOREIGN KEY (updated_by) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_chapter_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_chapter_comic (comic_id)
);

-- Chapter pages -----------------------------------------------------
CREATE TABLE chapter_page
(
    id          VARCHAR(36) PRIMARY KEY NOT NULL,
    chapter_id  VARCHAR(36)             NOT NULL,
    image_key   VARCHAR(255)            NOT NULL,  -- S3 key
    page_number INT                     NOT NULL,
    updated_by  VARCHAR(36)             NOT NULL,
    updated_at  DATETIME                NOT NULL,
    created_by  VARCHAR(36)             NOT NULL,
    created_at  DATETIME                NOT NULL,
    CONSTRAINT uq_chapter_page UNIQUE (chapter_id, page_number),
    CONSTRAINT fk_chapter_page_ref_chapter_id FOREIGN KEY (chapter_id) REFERENCES chapter (id) ON DELETE CASCADE,
    CONSTRAINT fk_chapter_page_ref_updated_by FOREIGN KEY (updated_by) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_chapter_page_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_chapter_page_chapter (chapter_id)
);

-- Comments (threaded, per comic or per chapter) ---------------------
CREATE TABLE comic_comment
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id   VARCHAR(36)             NOT NULL,
    chapter_id VARCHAR(36),                       -- null = comic-level comment
    parent_id  VARCHAR(36),                       -- null = top-level
    user_id    VARCHAR(36)             NOT NULL,
    message    TEXT                    NOT NULL,
    deleted_at DATETIME,
    deleted_by VARCHAR(36),
    updated_by VARCHAR(36)             NOT NULL,
    updated_at DATETIME                NOT NULL,
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT fk_comic_comment_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_comment_ref_chapter_id FOREIGN KEY (chapter_id) REFERENCES chapter (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_comment_ref_parent_id FOREIGN KEY (parent_id) REFERENCES comic_comment (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_comment_ref_user_id FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_comment_ref_updated_by FOREIGN KEY (updated_by) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_comment_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_comic_comment_comic (comic_id, created_at)
);

-- Likes & bookmarks (idempotent via UNIQUE) ------------------------
CREATE TABLE comic_like_link
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id   VARCHAR(36)             NOT NULL,
    user_id    VARCHAR(36)             NOT NULL,
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT uq_comic_like UNIQUE (comic_id, user_id),
    CONSTRAINT fk_comic_like_link_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_like_link_ref_user_id FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_like_link_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_comic_like_user (user_id)
);

CREATE TABLE comic_bookmark_link
(
    id         VARCHAR(36) PRIMARY KEY NOT NULL,
    comic_id   VARCHAR(36)             NOT NULL,
    user_id    VARCHAR(36)             NOT NULL,
    created_by VARCHAR(36)             NOT NULL,
    created_at DATETIME                NOT NULL,
    CONSTRAINT uq_comic_bookmark UNIQUE (comic_id, user_id),
    CONSTRAINT fk_comic_bookmark_link_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_bookmark_link_ref_user_id FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_bookmark_link_ref_created_by FOREIGN KEY (created_by) REFERENCES user (id) ON DELETE CASCADE,
    INDEX idx_comic_bookmark_user (user_id)
);

-- Reading progress (resume where you left off) ----------------------
CREATE TABLE comic_reading_progress
(
    id               VARCHAR(36) PRIMARY KEY NOT NULL,
    user_id          VARCHAR(36)             NOT NULL,
    comic_id         VARCHAR(36)             NOT NULL,
    last_chapter_id  VARCHAR(36),
    last_page_number INT,
    updated_at       DATETIME                NOT NULL,
    created_at       DATETIME                NOT NULL,
    CONSTRAINT uq_reading_progress UNIQUE (user_id, comic_id),
    CONSTRAINT fk_comic_reading_progress_ref_user_id FOREIGN KEY (user_id) REFERENCES user (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_reading_progress_ref_comic_id FOREIGN KEY (comic_id) REFERENCES comic (id) ON DELETE CASCADE,
    CONSTRAINT fk_comic_reading_progress_ref_last_chapter_id FOREIGN KEY (last_chapter_id) REFERENCES chapter (id) ON DELETE SET NULL
);
```

Notes:

- No MuComics `user` table — all user references point at µLearn's `user(id)` (`VARCHAR(36)`).
- `created_by`/`updated_by` use `ON DELETE CASCADE` to match µLearn's convention; content ownership and soft-delete columns mean a hard user delete is rare. If the team prefers content to survive creator deletion, switch `comic.created_by` to `ON DELETE SET NULL` — flag for the µLearn team.
- `like_count`/`comment_count`/`bookmark_count` are denormalized and maintained by the app (or DB triggers) so list/detail reads never `COUNT(*)` live.
- Full-text search on `comic.title`/`description` should use a MySQL `FULLTEXT` index rather than `LIKE '%q%'`.

---

## 6. API Design

Base path: `/api/mucomics/v1/`. REST corrections from the review are applied: plural noun collections, HTTP methods instead of verb URLs, idempotent like/bookmark, pagination on every collection, and standard status codes (`201`+`Location`, `204`, `409` on duplicate).

**Auth is delegated to µLearn — there are no auth endpoints here.** MuComics validates the µLearn JWT and reads roles. Register, login, refresh, logout, verify-email, and password-reset are all µLearn's.

```
Identity (thin, MuComics-scoped)
  GET    /api/mucomics/v1/me                       -- µLearn identity + my comic stats
  GET    /api/mucomics/v1/me/bookmarks
  GET    /api/mucomics/v1/me/progress

Comics
  GET    /api/mucomics/v1/comics?search=&genre=&sort=-created_at&cursor=&limit=
  POST   /api/mucomics/v1/comics                   -- role: Comic Creator
  GET    /api/mucomics/v1/comics/{id}
  PATCH  /api/mucomics/v1/comics/{id}
  DELETE /api/mucomics/v1/comics/{id}              -- soft delete

Chapters
  GET    /api/mucomics/v1/comics/{comic_id}/chapters?cursor=&limit=
  POST   /api/mucomics/v1/comics/{comic_id}/chapters
  GET    /api/mucomics/v1/chapters/{id}
  PATCH  /api/mucomics/v1/chapters/{id}
  DELETE /api/mucomics/v1/chapters/{id}

Pages
  GET    /api/mucomics/v1/chapters/{id}/pages
  POST   /api/mucomics/v1/chapters/{id}/pages/upload-url   -- presigned S3 URL
  POST   /api/mucomics/v1/chapters/{id}/pages             -- register uploaded page(s)
  PATCH  /api/mucomics/v1/chapters/{id}/pages             -- reorder
  DELETE /api/mucomics/v1/pages/{id}

Interactions
  POST   /api/mucomics/v1/comics/{id}/likes        -- idempotent (409 on repeat or no-op)
  DELETE /api/mucomics/v1/comics/{id}/likes
  POST   /api/mucomics/v1/comics/{id}/bookmarks
  DELETE /api/mucomics/v1/comics/{id}/bookmarks
  GET    /api/mucomics/v1/comics/{id}/comments?cursor=&limit=
  POST   /api/mucomics/v1/comics/{id}/comments
  PATCH  /api/mucomics/v1/comments/{id}
  DELETE /api/mucomics/v1/comments/{id}

Reading progress
  PUT    /api/mucomics/v1/comics/{id}/progress     -- idempotent upsert

Admin / moderation (role: Comic Admin)
  PATCH  /api/mucomics/v1/admin/comics/{id}/status -- approve / archive
  DELETE /api/mucomics/v1/admin/comments/{id}
```

Cross-cutting standards: cursor pagination returning `{ data, meta: { next_cursor, has_more } }`; sorting via `?sort=-field`; a single error shape (RFC 9457 `application/problem+json` preferred); immutable published pages served with `Cache-Control: immutable` + `ETag`.

---

## 7. Storage

S3 holds cover images, chapter pages, profile/promo banners. Two rules:

- **Store the S3 object key in the DB** (`cover_image_key`, `image_key`), never the full URL. Build the public/CDN URL at the app layer so a bucket/CDN change doesn't require rewriting rows.
- **Direct-to-S3 uploads via presigned URLs.** The client asks MuComics for a presigned URL, uploads the file straight to S3, then registers metadata (`page_number`, `chapter_id`) through the API. Large binaries never transit the app servers.

Suggested key layout: `mucomics/comics/{comic_id}/chapters/{chapter_id}/{page_number}.webp`, plus `covers/{comic_id}.webp` and `banners/...`.

---

## 8. Authentication & Authorization

- **Authentication:** MuComics trusts the JWT issued by µLearn. It validates the signature with the shared secret / JWKS, extracts the user id, and loads the corresponding `user` row (read-only). No local credential store, no `password` column, no login screen.
- **Authorization:** MuComics roles are registered as rows in µLearn's `role` table — e.g. `Comic Creator`, `Comic Admin` — and granted to users through `user_role_link` (respecting its `verified` flag). A plain authenticated µLearn user is a **reader** by default and needs no role.
- **No SMTP/OTP/reset stack.** `forgot_password` and `otp_verification` already exist in µLearn; MuComics reuses µLearn's flows. Comic-specific notifications (e.g. "new chapter published") should go through µLearn's existing email/notification path rather than a parallel one.

> Confirm with the µLearn team: the exact JWT verification method (shared HMAC secret vs. public key/JWKS) and the canonical claim that carries the user id (`id` vs `muid`).

---

## 9. Optional µLearn-Native Integrations

These are not required for MVP but are low-cost ways to make MuComics feel native:

1. **Interest groups:** comics could be tagged to `interest_group` so they surface in existing IG views (e.g. a "Comics/Art" IG).
2. **Karma:** publishing a chapter or hitting engagement milestones could award karma through `karma_activity_log` + `wallet`, consistent with how µLearn rewards other activity. This ties into the existing `task_list`/karma machinery, so scope it deliberately rather than inventing a parallel points system.

---

## 10. Security

HTTPS everywhere; JWT validation and expiry handled per µLearn; role checks on every write/admin route; input validation and output encoding (XSS); rate limiting on writes and comment creation; least-privilege S3 (private bucket, presigned reads/writes, no public ACLs); and parameterized queries / ORM only. CSRF protection applies to any cookie-based session surface; pure bearer-token APIs rely on JWT instead.

---

## 11. Deployment

- **Backend:** MuComics (Django app inside the µLearn backend, or a separate service) connecting to the **shared µLearn MySQL**. Must run with network access to that database and to S3.
- **Frontend:** Vercel or Netlify (Next.js).
- **Database:** the existing µLearn MySQL — no new database is provisioned. MuComics migrations create only `comic*` / `chapter*` / `genre` tables.
- **Storage:** AWS S3 + CDN.

Migrations must be reviewed by whoever owns the µLearn schema, since MuComics tables share the database with production µLearn data.

---

## 12. Summary of Changes From the Original Report

- **Database engine corrected:** MySQL (shared µLearn DB), not PostgreSQL/Supabase/RDS.
- **No MuComics user table:** identity comes from µLearn's `user`; all user FKs are `VARCHAR(36)`.
- **Auth removed from scope:** login, register, refresh, password reset, OTP, and SMTP are all µLearn's. MuComics only validates JWTs.
- **RBAC mapped to µLearn:** reader/creator/admin become `role` rows granted via `user_role_link`, not a `role` column on a user table.
- **µLearn conventions adopted:** UUID PKs, `created_by`/`updated_by`/`created_at`/`updated_at` audit columns, soft-delete columns, `_link` junctions, `fk_<table>_ref_<col>` naming.
- **Schema correctness:** genres as M:N; unique constraints on likes/bookmarks/chapters/pages; denormalized counts; reading-progress and contributor tables added; S3 keys instead of URLs.
- **REST corrections:** versioned base path, noun collections, idempotent interactions, pagination, presigned uploads, standard status codes and error format.

---

### Open items to confirm with the µLearn team
1. JWT verification mechanism and the user-id claim.
2. Deployment shape — app inside the monolith (Option A) vs. separate service (Option B).
3. `created_by` delete behaviour on `comic` (CASCADE to match convention vs. SET NULL to preserve content).
4. Whether comic notifications route through µLearn's existing email/notification system.
5. Karma/IG integration scope (if any) for MVP.
