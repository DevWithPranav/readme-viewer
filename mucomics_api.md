# MuComics API Documentation

> **Base URL:** `/api/v1/muComics/`
>
> **Authentication:** All endpoints require a valid JWT token in the `Authorization` header unless otherwise noted.
>
> **Standard Pagination Query Parameters** (applies to all list endpoints):
> | Parameter   | Type    | Default | Description                           |
> |-------------|---------|---------|---------------------------------------|
> | `page`      | integer | 1       | Page number (1-indexed)               |
> | `per_page`  | integer | 10      | Items per page                        |
> | `search`    | string  | —       | Search term (field-specific per API)  |
> | `sort`      | string  | —       | Sort field name (prefix `-` for DESC) |

---

## Table of Contents

1. [Comics](#1-comics)
   - [1.1 List Comics](#11-list-comics)
   - [1.2 Create Comic](#12-create-comic)
   - [1.3 Get Comic Detail](#13-get-comic-detail)
   - [1.4 Update Comic](#14-update-comic)
   - [1.5 Delete Comic](#15-delete-comic)
   - [1.6 Publish Comic](#16-publish-comic)
   - [1.7 Archive Comic](#17-archive-comic)
2. [Genres (Admin)](#2-genres-admin)
   - [2.1 List Genres](#21-list-genres)
   - [2.2 Create Genre](#22-create-genre)
   - [2.3 Get Genre Detail](#23-get-genre-detail)
   - [2.4 Update Genre](#24-update-genre)
   - [2.5 Deactivate Genre (Soft Delete)](#25-deactivate-genre-soft-delete)
   - [2.6 Reinstate Genre](#26-reinstate-genre)
3. [Comic–Genre Links](#3-comicgenre-links)
   - [3.1 Assign Genre to Comic](#31-assign-genre-to-comic)
   - [3.2 Remove Genre from Comic](#32-remove-genre-from-comic)
4. [Contributors](#4-contributors)
   - [4.1 List Contributors](#41-list-contributors)
   - [4.2 Add Contributor](#42-add-contributor)
   - [4.3 Update Contributor Role](#43-update-contributor-role)
   - [4.4 Remove Contributor](#44-remove-contributor)
5. [Chapters](#5-chapters)
   - [5.1 List Chapters](#51-list-chapters)
   - [5.2 Create Chapter](#52-create-chapter)
   - [5.3 Get Chapter Detail](#53-get-chapter-detail)
   - [5.4 Update Chapter](#54-update-chapter)
   - [5.5 Delete Chapter](#55-delete-chapter)
   - [5.6 Publish Chapter](#56-publish-chapter)
   - [5.7 Archive Chapter](#57-archive-chapter)
6. [Pages](#6-pages)
   - [6.1 List Pages](#61-list-pages)
   - [6.2 Add Page](#62-add-page)
   - [6.3 Update Page](#63-update-page)
   - [6.4 Delete Page](#64-delete-page)
   - [6.5 Reorder Pages](#65-reorder-pages)
7. [Uploads](#7-uploads)
   - [7.1 Upload File](#71-upload-file)
   - [7.2 Register Uploaded Images as Pages](#72-register-uploaded-images-as-pages)
8. [Comments](#8-comments)
   - [8.1 List Comic Comments](#81-list-comic-comments)
   - [8.2 Create Comic Comment](#82-create-comic-comment)
   - [8.3 List Chapter Comments](#83-list-chapter-comments)
   - [8.4 Create Chapter Comment](#84-create-chapter-comment)
   - [8.5 Edit Comment](#85-edit-comment)
   - [8.6 Delete Comment (User)](#86-delete-comment-user)
   - [8.7 Admin — List All Comments](#87-admin--list-all-comments)
   - [8.8 Admin — Delete Comment](#88-admin--delete-comment)
9. [Reader](#9-reader)
   - [9.1 Like Comic](#91-like-comic)
   - [9.2 Unlike Comic](#92-unlike-comic)
   - [9.3 Bookmark Comic](#93-bookmark-comic)
   - [9.4 Remove Bookmark](#94-remove-bookmark)
   - [9.5 Get Interaction Status](#95-get-interaction-status)
   - [9.6 Get Reading Progress](#96-get-reading-progress)
   - [9.7 Save / Update Reading Progress](#97-save--update-reading-progress)
   - [9.8 Reader Dashboard](#98-reader-dashboard)
   - [9.9 My Bookmarks](#99-my-bookmarks)
   - [9.10 My Reading Progress](#910-my-reading-progress)

---

## 1. Comics

### 1.1 List Comics

- **API Name:** List Comics
- **Endpoint:** `GET /api/v1/muComics/comics/`
- **Description:** List all active (non-deleted) comics. Supports search by title, status filter, multi-genre filter, and sort ordering. Returns paginated results.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter | Type   | Required | Description                                                         |
  |-----------|--------|----------|---------------------------------------------------------------------|
  | `status`  | string | No       | Filter by status. Allowed: `draft`, `published`, `archived`         |
  | `genre`   | string | No       | Filter by genre slug(s), comma-separated. e.g. `action,horror`      |
  | `search`  | string | No       | Search comics by `title`                                            |
  | `sort`    | string | No       | Sort field (see below)                                              |
  | `page`    | int    | No       | Page number                                                         |
  | `per_page`| int    | No       | Items per page                                                      |

- **Supported Ordering Fields:** `created_at`, `title`
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comics/?status=published&genre=action,romance&search=hero&sort=-created_at&page=1&per_page=10
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": []
    },
    "response": {
      "data": [
        {
          "id": "uuid-string",
          "title": "My Comic",
          "slug": "my-comic",
          "cover_image_key": "comics/covers/uuid.jpg",
          "status": "published",
          "like_count": 5,
          "comment_count": 12,
          "bookmark_count": 3,
          "published_at": "2026-07-15T10:00:00Z",
          "created_by": {
            "id": "user-uuid",
            "full_name": "John Doe",
            "muid": "john@mulearn"
          },
          "created_at": "2026-07-10T08:30:00Z",
          "genres": [
            { "id": "genre-uuid", "name": "Action", "slug": "action" }
          ]
        }
      ],
      "pagination": {
        "count": 1,
        "totalPages": 1,
        "isNext": false,
        "isPrev": false,
        "nextPage": null,
        "prevPage": null
      }
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                              |
  |-------------------|----------|------------------------------------------|
  | `id`              | string   | Comic UUID                               |
  | `title`           | string   | Comic title                              |
  | `slug`            | string   | URL-safe slug                            |
  | `cover_image_key` | string   | Storage key for cover image              |
  | `status`          | string   | `draft` / `published` / `archived`       |
  | `like_count`      | integer  | Number of likes                          |
  | `comment_count`   | integer  | Number of comments                       |
  | `bookmark_count`  | integer  | Number of bookmarks                      |
  | `published_at`    | datetime | When the comic was published (nullable)  |
  | `created_by`      | object   | Creator user (`id`, `full_name`, `muid`) |
  | `created_at`      | datetime | Creation timestamp                       |
  | `genres`          | array    | Active genres (`id`, `name`, `slug`)     |

- **Error Responses:**

  | Status | Condition                            | Message                                            |
  |--------|--------------------------------------|----------------------------------------------------|
  | 400    | Invalid `status` parameter           | `Invalid status. Allowed: draft, published, archived.` |

---

### 1.2 Create Comic

- **API Name:** Create Comic
- **Endpoint:** `POST /api/v1/muComics/comics/`
- **Description:** Create a new comic. Any authenticated user may create a comic. The comic is created in `draft` status. A unique slug is auto-generated from the title.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field             | Type   | Required | Description                                      |
  |-------------------|--------|----------|--------------------------------------------------|
  | `title`           | string | Yes      | Comic title (max 150 chars, must not be blank)   |
  | `description`     | string | No       | Comic description (nullable)                     |
  | `cover_image_key` | string | No       | Storage key for cover image (max 255 chars)      |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "title": "Adventures of MuBot",
    "description": "A thrilling adventure of a robot in the Mulearn universe.",
    "cover_image_key": "comics/covers/mubot-cover.jpg"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot\" created successfully."]
    },
    "response": {
      "id": "generated-uuid",
      "title": "Adventures of MuBot",
      "slug": "adventures-of-mubot",
      "description": "A thrilling adventure of a robot in the Mulearn universe.",
      "cover_image_key": "comics/covers/mubot-cover.jpg",
      "status": "draft",
      "like_count": 0,
      "comment_count": 0,
      "bookmark_count": 0,
      "published_at": null,
      "genres": [],
      "contributors": [],
      "created_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "created_at": "2026-07-19T08:00:00Z",
      "updated_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Comic Detail (see [1.3](#13-get-comic-detail)).
- **Error Responses:**

  | Status | Condition                  | Message                       |
  |--------|----------------------------|-------------------------------|
  | 400    | Validation errors          | Serializer error details      |
  | 400    | Title is blank             | `Title must not be blank.`    |

---

### 1.3 Get Comic Detail

- **API Name:** Get Comic Detail
- **Endpoint:** `GET /api/v1/muComics/comics/<comic_id>/`
- **Description:** Retrieve full detail for a single active comic, including contributors and genres.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comics/abc-123-def/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot\" retrieved successfully."]
    },
    "response": {
      "id": "abc-123-def",
      "title": "Adventures of MuBot",
      "slug": "adventures-of-mubot",
      "description": "A thrilling adventure...",
      "cover_image_key": "comics/covers/mubot-cover.jpg",
      "status": "published",
      "like_count": 42,
      "comment_count": 10,
      "bookmark_count": 7,
      "published_at": "2026-07-15T10:00:00Z",
      "genres": [
        { "id": "genre-uuid", "name": "Action", "slug": "action" }
      ],
      "contributors": [
        {
          "id": "link-uuid",
          "user": { "id": "user-uuid", "full_name": "Jane Artist", "muid": "jane@mulearn" },
          "contributor_type": "artist",
          "created_at": "2026-07-12T09:00:00Z"
        }
      ],
      "created_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "created_at": "2026-07-10T08:30:00Z",
      "updated_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "updated_at": "2026-07-15T10:00:00Z"
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                                                     |
  |-------------------|----------|-----------------------------------------------------------------|
  | `id`              | string   | Comic UUID                                                      |
  | `title`           | string   | Comic title                                                     |
  | `slug`            | string   | URL-safe slug                                                   |
  | `description`     | string   | Full description (nullable)                                     |
  | `cover_image_key` | string   | Storage key for cover image (nullable)                          |
  | `status`          | string   | `draft` / `published` / `archived`                              |
  | `like_count`      | integer  | Number of likes                                                 |
  | `comment_count`   | integer  | Number of comments                                              |
  | `bookmark_count`  | integer  | Number of bookmarks                                             |
  | `published_at`    | datetime | When published (nullable)                                       |
  | `genres`          | array    | Active genres: `[{id, name, slug}]`                             |
  | `contributors`    | array    | Contributors: `[{id, user{id,full_name,muid}, contributor_type, created_at}]` |
  | `created_by`      | object   | Creator: `{id, full_name, muid}`                                |
  | `created_at`      | datetime | Creation timestamp                                              |
  | `updated_by`      | object   | Last updater: `{id, full_name, muid}`                           |
  | `updated_at`      | datetime | Last update timestamp                                           |

- **Error Responses:**

  | Status | Condition        | Message             |
  |--------|------------------|---------------------|
  | 400    | Comic not found  | `Comic not found.`  |

---

### 1.4 Update Comic

- **API Name:** Update Comic (Partial)
- **Endpoint:** `PATCH /api/v1/muComics/comics/<comic_id>/`
- **Description:** Partially update a comic's title, description, or cover image. Only the creator or an assigned editor contributor may edit. Archived comics cannot be edited. Slug is regenerated if title changes.
- **Permissions / Access Control:** Comic creator OR assigned editor contributor.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** (all fields optional — partial update)

  | Field             | Type   | Required | Description                                  |
  |-------------------|--------|----------|----------------------------------------------|
  | `title`           | string | No       | New title (max 150 chars, must not be blank)  |
  | `description`     | string | No       | New description (nullable)                    |
  | `cover_image_key` | string | No       | New cover image key (max 255 chars, nullable) |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/comics/abc-123-def/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "title": "Adventures of MuBot v2"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot v2\" updated successfully."]
    },
    "response": { "...full comic detail object..." }
  }
  ```
- **Response Fields:** Same as Comic Detail (see [1.3](#13-get-comic-detail)).
- **Error Responses:**

  | Status | Condition                       | Message                                                    |
  |--------|---------------------------------|------------------------------------------------------------|
  | 400    | Comic not found                 | `Comic not found.`                                         |
  | 400    | Comic is archived               | `Archived comics cannot be edited.`                        |
  | 400    | Validation errors               | Serializer error details                                   |
  | 403    | Not creator or editor           | `You do not have permission to edit this comic.`           |

---

### 1.5 Delete Comic

- **API Name:** Delete Comic (Soft Delete)
- **Endpoint:** `DELETE /api/v1/muComics/comics/<comic_id>/`
- **Description:** Soft-delete a comic (sets `deleted_at`). The comic is hidden from all list/detail responses after deletion. Only the original creator may delete.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comics/abc-123-def/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot\" deleted successfully."]
    },
    "response": {
      "id": "abc-123-def"
    }
  }
  ```
- **Response Fields:**

  | Field | Type   | Description          |
  |-------|--------|----------------------|
  | `id`  | string | UUID of deleted comic|

- **Error Responses:**

  | Status | Condition                   | Message                                             |
  |--------|-----------------------------|-----------------------------------------------------|
  | 400    | Comic not found             | `Comic not found.`                                  |
  | 403    | Not the creator             | `Only the comic creator can delete this comic.`     |

---

### 1.6 Publish Comic

- **API Name:** Publish Comic
- **Endpoint:** `POST /api/v1/muComics/comics/<comic_id>/publish/`
- **Description:** Transition a draft comic to `published` status. Only the creator may publish. Requires `title` and `description` to be present. Archived comics cannot be re-published.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/abc-123-def/publish/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot\" published successfully."]
    },
    "response": {
      "id": "abc-123-def",
      "status": "published"
    }
  }
  ```
- **Response Fields:**

  | Field    | Type   | Description                      |
  |----------|--------|----------------------------------|
  | `id`     | string | Comic UUID                       |
  | `status` | string | New status (`published`)         |

- **Error Responses:**

  | Status | Condition                              | Message                                                                  |
  |--------|----------------------------------------|--------------------------------------------------------------------------|
  | 400    | Comic not found                        | `Comic not found.`                                                       |
  | 400    | Already published                      | `Comic is already published.`                                            |
  | 400    | Comic is archived                      | `Archived comics cannot be published. Unarchive it first.`               |
  | 400    | Missing required fields                | `Cannot publish: missing required fields: title, description.`           |
  | 403    | Not the creator                        | `Only the comic creator can publish this comic.`                         |

---

### 1.7 Archive Comic

- **API Name:** Archive Comic
- **Endpoint:** `POST /api/v1/muComics/comics/<comic_id>/archive/`
- **Description:** Transition a draft or published comic to `archived` status. Only the creator may archive. Already-archived comics are rejected.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/abc-123-def/archive/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comic \"Adventures of MuBot\" archived successfully."]
    },
    "response": {
      "id": "abc-123-def",
      "status": "archived"
    }
  }
  ```
- **Response Fields:**

  | Field    | Type   | Description                    |
  |----------|--------|--------------------------------|
  | `id`     | string | Comic UUID                     |
  | `status` | string | New status (`archived`)        |

- **Error Responses:**

  | Status | Condition                   | Message                                             |
  |--------|-----------------------------|-----------------------------------------------------|
  | 400    | Comic not found             | `Comic not found.`                                  |
  | 400    | Already archived            | `Comic is already archived.`                        |
  | 403    | Not the creator             | `Only the comic creator can archive this comic.`    |

---

## 2. Genres (Admin)

### 2.1 List Genres

- **API Name:** List Genres
- **Endpoint:** `GET /api/v1/muComics/comics/genres/`
- **Description:** List genres. Non-admins see only active genres. Admins can filter with `?is_active=true|false` and see all genres. Supports search by name and sorting.
- **Permissions / Access Control:** Authenticated user (any role). Admin sees all; non-admin sees active only.
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter    | Type   | Required | Description                                                       |
  |--------------|--------|----------|-------------------------------------------------------------------|
  | `is_active`  | string | No       | Admin only. Filter: `true` or `false`                              |
  | `search`     | string | No       | Search genres by `name`                                            |
  | `sort`       | string | No       | Sort field (see below)                                             |
  | `page`       | int    | No       | Page number                                                        |
  | `per_page`   | int    | No       | Items per page                                                     |

- **Supported Ordering Fields:** `name`, `created_at`
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comics/genres/?is_active=true&search=action&sort=name
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "genre-uuid",
          "name": "Action",
          "slug": "action",
          "is_active": true,
          "created_at": "2026-07-01T00:00:00Z",
          "updated_at": "2026-07-01T00:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field        | Type     | Description                 |
  |--------------|----------|-----------------------------|
  | `id`         | string   | Genre UUID                  |
  | `name`       | string   | Genre display name          |
  | `slug`       | string   | URL-safe slug               |
  | `is_active`  | boolean  | Active/inactive status      |
  | `created_at` | datetime | Creation timestamp          |
  | `updated_at` | datetime | Last update timestamp       |

- **Error Responses:**

  | Status | Condition                               | Message                                             |
  |--------|-----------------------------------------|-----------------------------------------------------|
  | 400    | Invalid `is_active` value               | `Invalid value for is_active. Use true or false.`   |

---

### 2.2 Create Genre

- **API Name:** Create Genre
- **Endpoint:** `POST /api/v1/muComics/comics/genres/`
- **Description:** Create a new genre. `is_active` defaults to `true`. Slug is auto-generated from name.
- **Permissions / Access Control:** **Admin role required.**
- **Path Parameters:** None
- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field  | Type   | Required | Description                              |
  |--------|--------|----------|------------------------------------------|
  | `name` | string | Yes      | Genre name (max 75 chars, must not be blank) |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/genres/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "name": "Horror"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Horror\" created successfully."]
    },
    "response": {
      "id": "generated-uuid",
      "name": "Horror",
      "slug": "horror",
      "is_active": true,
      "created_at": "2026-07-19T08:00:00Z",
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Genre object (see [2.1](#21-list-genres)).
- **Error Responses:**

  | Status | Condition              | Message                        |
  |--------|------------------------|--------------------------------|
  | 400    | Validation errors      | Serializer error details       |
  | 400    | Name is blank          | `Name must not be blank.`      |
  | 403    | Not admin              | Unauthorized                   |

---

### 2.3 Get Genre Detail

- **API Name:** Get Genre Detail
- **Endpoint:** `GET /api/v1/muComics/comics/genres/<genre_id>/`
- **Description:** Retrieve a genre. Non-admins only see active genres (inactive returns 404). Admins can retrieve any genre regardless of `is_active` status.
- **Permissions / Access Control:** Authenticated user (any role). Non-admins see active only.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `genre_id` | string | UUID of the genre    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comics/genres/genre-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Horror\" retrieved successfully."]
    },
    "response": {
      "id": "genre-uuid-123",
      "name": "Horror",
      "slug": "horror",
      "is_active": true,
      "created_at": "2026-07-01T00:00:00Z",
      "updated_at": "2026-07-01T00:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Genre object (see [2.1](#21-list-genres)).
- **Error Responses:**

  | Status | Condition        | Message              |
  |--------|------------------|----------------------|
  | 400    | Genre not found  | `Genre not found.`   |

---

### 2.4 Update Genre

- **API Name:** Update Genre
- **Endpoint:** `PATCH /api/v1/muComics/comics/genres/<genre_id>/`
- **Description:** Update a genre's name. Slug is regenerated automatically from the new name. Can edit both active and inactive genres.
- **Permissions / Access Control:** **Admin role required.**
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `genre_id` | string | UUID of the genre    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field  | Type   | Required | Description                       |
  |--------|--------|----------|-----------------------------------|
  | `name` | string | No       | New genre name (max 75 chars)     |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/comics/genres/genre-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "name": "Sci-Fi Horror"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Sci-Fi Horror\" updated successfully."]
    },
    "response": {
      "id": "genre-uuid-123",
      "name": "Sci-Fi Horror",
      "slug": "sci-fi-horror",
      "is_active": true,
      "created_at": "2026-07-01T00:00:00Z",
      "updated_at": "2026-07-19T08:30:00Z"
    }
  }
  ```
- **Response Fields:** Same as Genre object (see [2.1](#21-list-genres)).
- **Error Responses:**

  | Status | Condition              | Message                   |
  |--------|------------------------|---------------------------|
  | 400    | Genre not found        | `Genre not found.`        |
  | 400    | Validation errors      | Serializer error details  |
  | 403    | Not admin              | Unauthorized              |

---

### 2.5 Deactivate Genre (Soft Delete)

- **API Name:** Deactivate Genre
- **Endpoint:** `DELETE /api/v1/muComics/comics/genres/<genre_id>/`
- **Description:** Soft-deactivate a genre by setting `is_active = false`. The genre row and all `comic_genre_link` rows are preserved; the genre simply stops appearing for non-admin users.
- **Permissions / Access Control:** **Admin role required.**
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `genre_id` | string | UUID of the genre    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comics/genres/genre-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Horror\" deactivated successfully."]
    },
    "response": {
      "id": "genre-uuid-123",
      "is_active": false
    }
  }
  ```
- **Response Fields:**

  | Field       | Type    | Description                        |
  |-------------|---------|------------------------------------|
  | `id`        | string  | Genre UUID                         |
  | `is_active` | boolean | Always `false` after deactivation  |

- **Error Responses:**

  | Status | Condition               | Message                                       |
  |--------|-------------------------|-----------------------------------------------|
  | 400    | Genre not found         | `Genre not found.`                             |
  | 400    | Already inactive        | `Genre "Horror" is already inactive.`          |
  | 403    | Not admin               | Unauthorized                                   |

---

### 2.6 Reinstate Genre

- **API Name:** Reinstate Genre
- **Endpoint:** `POST /api/v1/muComics/comics/genres/<genre_id>/reinstate/`
- **Description:** Reactivate a deactivated genre by setting `is_active = true`.
- **Permissions / Access Control:** **Admin role required.**
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `genre_id` | string | UUID of the genre    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/genres/genre-uuid-123/reinstate/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Horror\" reinstated successfully."]
    },
    "response": {
      "id": "genre-uuid-123",
      "is_active": true
    }
  }
  ```
- **Response Fields:**

  | Field       | Type    | Description                       |
  |-------------|---------|-----------------------------------|
  | `id`        | string  | Genre UUID                        |
  | `is_active` | boolean | Always `true` after reinstatement |

- **Error Responses:**

  | Status | Condition              | Message                                   |
  |--------|------------------------|-------------------------------------------|
  | 400    | Genre not found        | `Genre not found.`                         |
  | 400    | Already active         | `Genre "Horror" is already active.`        |
  | 403    | Not admin              | Unauthorized                               |

---

## 3. Comic–Genre Links

### 3.1 Assign Genre to Comic

- **API Name:** Assign Genre to Comic
- **Endpoint:** `POST /api/v1/muComics/comics/<comic_id>/genres/`
- **Description:** Assign a genre to a comic. The genre must exist and be active. Duplicate assignments are rejected.
- **Permissions / Access Control:** Comic creator, assigned editor, or Admin.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field      | Type   | Required | Description               |
  |------------|--------|----------|---------------------------|
  | `genre_id` | string | Yes      | UUID of the genre to assign |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/abc-123-def/genres/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "genre_id": "genre-uuid-123"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Action\" assigned to comic \"Adventures of MuBot\" successfully."]
    },
    "response": {
      "link_id": "link-uuid",
      "genre": {
        "id": "genre-uuid-123",
        "name": "Action",
        "slug": "action"
      }
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description                                |
  |-----------|--------|--------------------------------------------|
  | `link_id` | string | UUID of the created comic-genre link       |
  | `genre`   | object | Genre details: `{id, name, slug}`          |

- **Error Responses:**

  | Status | Condition                          | Message                                                         |
  |--------|------------------------------------|-----------------------------------------------------------------|
  | 400    | Comic not found                    | `Comic not found.`                                              |
  | 400    | Genre not found                    | `Genre with id 'xxx' not found.`                                |
  | 400    | Genre is inactive                  | `Cannot assign an inactive genre.`                              |
  | 400    | Genre already assigned             | `This genre is already assigned to the comic.` / `This genre is already assigned to this comic.` |
  | 403    | Insufficient permissions           | `Only the comic creator, assigned editor, or an Admin can manage genres for this comic.` |

---

### 3.2 Remove Genre from Comic

- **API Name:** Remove Genre from Comic
- **Endpoint:** `DELETE /api/v1/muComics/comics/<comic_id>/genres/<link_id>/`
- **Description:** Remove a genre assignment from a comic by deleting the comic-genre link record.
- **Permissions / Access Control:** Comic creator, assigned editor, or Admin.
- **Path Parameters:**

  | Parameter  | Type   | Description                    |
  |------------|--------|--------------------------------|
  | `comic_id` | string | UUID of the comic              |
  | `link_id`  | string | UUID of the comic-genre link   |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comics/abc-123-def/genres/link-uuid-456/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Genre \"Action\" removed from comic \"Adventures of MuBot\" successfully."]
    },
    "response": {
      "link_id": "link-uuid-456"
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description                          |
  |-----------|--------|--------------------------------------|
  | `link_id` | string | UUID of the removed link             |

- **Error Responses:**

  | Status | Condition                        | Message                                                         |
  |--------|----------------------------------|-----------------------------------------------------------------|
  | 400    | Comic not found                  | `Comic <comic_id> not found.`                                  |
  | 400    | Link not found                   | `Genre link not found for this comic.`                          |
  | 403    | Insufficient permissions         | `Only the comic creator, assigned editor, or an Admin can manage genres for this comic.` |

---

## 4. Contributors

### 4.1 List Contributors

- **API Name:** List Contributors
- **Endpoint:** `GET /api/v1/muComics/comics/<comic_id>/contributors/`
- **Description:** List all contributors for a comic. Supports pagination, search, optional role filter, and sorting.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:**

  | Parameter | Type   | Required | Description                                                                       |
  |-----------|--------|----------|-----------------------------------------------------------------------------------|
  | `role`    | string | No       | Filter by contributor role: `writer`, `artist`, `colorist`, `editor`               |
  | `search`  | string | No       | Search by user's `full_name` or `contributor_type`                                 |
  | `sort`    | string | No       | Sort field (see below)                                                             |
  | `page`    | int    | No       | Page number                                                                        |
  | `per_page`| int    | No       | Items per page                                                                     |

- **Supported Ordering Fields:** `role` (maps to `contributor_type`)
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comics/abc-123-def/contributors/?role=artist&sort=role
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "contributor_id": "link-uuid",
          "user_id": "user-uuid",
          "role": "artist",
          "name": "Jane Artist",
          "comic_name": "Adventures of MuBot"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field            | Type   | Description                   |
  |------------------|--------|-------------------------------|
  | `contributor_id` | string | Contributor link UUID         |
  | `user_id`        | string | User UUID                     |
  | `role`           | string | Contributor role              |
  | `name`           | string | User's full name              |
  | `comic_name`     | string | Parent comic's title          |

- **Error Responses:**

  | Status | Condition       | Message            |
  |--------|-----------------|--------------------|
  | 400    | Comic not found | `Comic not found.` |

---

### 4.2 Add Contributor

- **API Name:** Add Contributor
- **Endpoint:** `POST /api/v1/muComics/comics/<comic_id>/contributors/`
- **Description:** Add a contributor to a comic by their muid and role. The `CREATOR` role cannot be assigned via this endpoint. Duplicate (same user + same role) assignments are rejected.
- **Permissions / Access Control:** Comic creator or Admin.
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field  | Type   | Required | Description                                                             |
  |--------|--------|----------|-------------------------------------------------------------------------|
  | `muid` | string | Yes      | Muid of the user to add                                                 |
  | `role` | string | Yes      | Contributor role: `writer`, `artist`, `colorist`, or `editor`            |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comics/abc-123-def/contributors/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "muid": "jane@mulearn",
    "role": "artist"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["artist jane@mulearn added successfully."]
    }
  }
  ```
- **Response Fields:** Message confirmation only.
- **Error Responses:**

  | Status | Condition                       | Message                                                       |
  |--------|---------------------------------|---------------------------------------------------------------|
  | 400    | Comic not found                 | `Comic not found.`                                            |
  | 400    | User not found                  | `No Active user with muid "xxx" not found.`                   |
  | 400    | Duplicate role                  | `Contributor with this role already exists.`                   |
  | 400    | Validation errors               | Serializer error details                                      |
  | 403    | Not creator or admin            | `Only the comic creator or an admin can add contributors.`    |

---

### 4.3 Update Contributor Role

- **API Name:** Update Contributor Role
- **Endpoint:** `PATCH /api/v1/muComics/comics/<comic_id>/contributors/<contributor_id>/`
- **Description:** Update a contributor's role. The `CREATOR` role cannot be assigned.
- **Permissions / Access Control:** Comic creator or Admin.
- **Path Parameters:**

  | Parameter        | Type   | Description                   |
  |------------------|--------|-------------------------------|
  | `comic_id`       | string | UUID of the comic             |
  | `contributor_id` | string | UUID of the contributor link   |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field  | Type   | Required | Description                                                   |
  |--------|--------|----------|---------------------------------------------------------------|
  | `role` | string | Yes      | New role: `writer`, `artist`, `colorist`, or `editor`          |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/comics/abc-123-def/contributors/link-uuid-789/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "role": "writer"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["writer jane@mulearn updated."]
    }
  }
  ```
- **Response Fields:** Message confirmation only.
- **Error Responses:**

  | Status | Condition                       | Message                                                          |
  |--------|---------------------------------|------------------------------------------------------------------|
  | 400    | Comic not found                 | `Comic not found.`                                               |
  | 400    | Contributor not found           | `Contributor not found.`                                          |
  | 400    | Duplicate role conflict         | `Contributor with this role already exists.`                      |
  | 400    | Validation errors               | Serializer error details                                         |
  | 403    | Not creator or admin            | `Only the comic creator or an admin can update contributors.`    |

---

### 4.4 Remove Contributor

- **API Name:** Remove Contributor
- **Endpoint:** `DELETE /api/v1/muComics/comics/<comic_id>/contributors/<contributor_id>/`
- **Description:** Remove a contributor from a comic. Permanently deletes the contributor link.
- **Permissions / Access Control:** Comic creator or Admin.
- **Path Parameters:**

  | Parameter        | Type   | Description                   |
  |------------------|--------|-------------------------------|
  | `comic_id`       | string | UUID of the comic             |
  | `contributor_id` | string | UUID of the contributor link   |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comics/abc-123-def/contributors/link-uuid-789/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["artist jane@mulearn removed."]
    }
  }
  ```
- **Response Fields:** Message confirmation only.
- **Error Responses:**

  | Status | Condition                       | Message                                                          |
  |--------|---------------------------------|------------------------------------------------------------------|
  | 400    | Comic not found                 | `Comic not found.`                                               |
  | 400    | Contributor not found           | `Contributor not found.`                                          |
  | 403    | Not creator or admin            | `Only the comic creator or an admin can remove contributors.`    |

---

## 5. Chapters

### 5.1 List Chapters

- **API Name:** List Chapters
- **Endpoint:** `GET /api/v1/muComics/chapters/`
- **Description:** List all active (non-deleted) chapters for a specific comic. Requires `comic_id` as a query parameter. Supports status filter, search by title, and sorting.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter  | Type   | Required | Description                                                     |
  |------------|--------|----------|-----------------------------------------------------------------|
  | `comic_id` | string | **Yes**  | UUID of the parent comic                                         |
  | `status`   | string | No       | Filter by status: `draft`, `published`, `archived`               |
  | `search`   | string | No       | Search chapters by `title`                                       |
  | `sort`     | string | No       | Sort field (see below)                                           |
  | `page`     | int    | No       | Page number                                                      |
  | `per_page` | int    | No       | Items per page                                                   |

- **Supported Ordering Fields:** `chapter_number`, `created_at`
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/chapters/?comic_id=abc-123-def&status=published&sort=chapter_number
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "chapter-uuid",
          "comic": "abc-123-def",
          "title": "Chapter 1: The Beginning",
          "slug": "chapter-1-the-beginning",
          "chapter_number": 1,
          "cover_image_key": "http://domain.com/muback-media/comics/chapters/cover.jpg",
          "status": "published",
          "published_at": "2026-07-15T10:00:00Z",
          "created_at": "2026-07-10T08:30:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                            |
  |-------------------|----------|----------------------------------------|
  | `id`              | string   | Chapter UUID                           |
  | `comic`           | string   | Parent comic UUID                      |
  | `title`           | string   | Chapter title                          |
  | `slug`            | string   | URL-safe slug                          |
  | `chapter_number`  | number   | Chapter number (decimal, e.g. 1.5)     |
  | `cover_image_key` | string   | Resolved URL for cover image (nullable)|
  | `status`          | string   | `draft` / `published` / `archived`     |
  | `published_at`    | datetime | Publication timestamp (nullable)       |
  | `created_at`      | datetime | Creation timestamp                     |

- **Error Responses:**

  | Status | Condition                          | Message                                                  |
  |--------|------------------------------------|----------------------------------------------------------|
  | 400    | Missing `comic_id`                 | `comic_id query parameter is required.`                  |
  | 400    | Invalid `status`                   | `Invalid status. Allowed: draft, published, archived.`   |
  | 404    | Comic not found                    | `Comic not found.`                                       |

---

### 5.2 Create Chapter

- **API Name:** Create Chapter
- **Endpoint:** `POST /api/v1/muComics/chapters/`
- **Description:** Create a new chapter inside a comic. Created in `draft` status. Slug is auto-generated from title. Chapter number must be unique within the comic.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:** None
- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field             | Type   | Required | Description                                                |
  |-------------------|--------|----------|------------------------------------------------------------|
  | `comic`           | string | Yes      | UUID of the parent comic                                    |
  | `title`           | string | Yes      | Chapter title (max 150 chars, must not be blank)            |
  | `chapter_number`  | number | Yes      | Chapter number (must be ≥ 0, unique per comic)              |
  | `description`     | string | No       | Chapter description (nullable)                              |
  | `cover_image_key` | string | No       | Storage key for chapter cover image (max 255 chars, nullable)|

- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "comic": "abc-123-def",
    "title": "Chapter 1: The Beginning",
    "chapter_number": 1,
    "description": "Our hero embarks on their journey."
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1: The Beginning\" created successfully."]
    },
    "response": {
      "id": "chapter-uuid",
      "comic": "abc-123-def",
      "title": "Chapter 1: The Beginning",
      "slug": "chapter-1-the-beginning",
      "description": "Our hero embarks on their journey.",
      "chapter_number": 1,
      "cover_image_key": null,
      "status": "draft",
      "published_at": null,
      "pages": [],
      "created_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "created_at": "2026-07-19T08:00:00Z",
      "updated_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Chapter Detail (see [5.3](#53-get-chapter-detail)).
- **Error Responses:**

  | Status | Condition                                | Message                                                          |
  |--------|------------------------------------------|------------------------------------------------------------------|
  | 400    | Missing `comic`                          | `comic is required.`                                             |
  | 400    | Validation errors                        | Serializer error details                                         |
  | 400    | Duplicate chapter number                 | `A chapter with this number already exists for this comic.`      |
  | 400    | Negative chapter number                  | `Chapter number must not be negative.`                           |
  | 403    | Not creator or editor                    | `You do not have permission to add a chapter to this comic.`     |
  | 404    | Comic not found                          | `Comic not found.`                                               |

---

### 5.3 Get Chapter Detail

- **API Name:** Get Chapter Detail
- **Endpoint:** `GET /api/v1/muComics/chapters/<chapter_id>/`
- **Description:** Retrieve full detail of a chapter including its active (non-deleted) pages ordered by page number.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/chapters/chapter-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1: The Beginning\" retrieved successfully."]
    },
    "response": {
      "id": "chapter-uuid-123",
      "comic": "abc-123-def",
      "title": "Chapter 1: The Beginning",
      "slug": "chapter-1-the-beginning",
      "description": "Our hero embarks on their journey.",
      "chapter_number": 1,
      "cover_image_key": "http://domain.com/muback-media/comics/chapters/cover.jpg",
      "status": "published",
      "published_at": "2026-07-15T10:00:00Z",
      "pages": [
        {
          "id": "page-uuid",
          "chapter": "chapter-uuid-123",
          "page_number": 1,
          "image_key": "http://domain.com/muback-media/comics/chapters/page1.jpg",
          "created_at": "2026-07-12T09:00:00Z"
        }
      ],
      "created_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "created_at": "2026-07-10T08:30:00Z",
      "updated_by": { "id": "user-uuid", "full_name": "John Doe", "muid": "john@mulearn" },
      "updated_at": "2026-07-15T10:00:00Z"
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                                           |
  |-------------------|----------|-------------------------------------------------------|
  | `id`              | string   | Chapter UUID                                          |
  | `comic`           | string   | Parent comic UUID                                     |
  | `title`           | string   | Chapter title                                         |
  | `slug`            | string   | URL-safe slug                                         |
  | `description`     | string   | Chapter description (nullable)                        |
  | `chapter_number`  | number   | Chapter number                                        |
  | `cover_image_key` | string   | Resolved cover image URL (nullable)                   |
  | `status`          | string   | `draft` / `published` / `archived`                    |
  | `published_at`    | datetime | Publication timestamp (nullable)                      |
  | `pages`           | array    | Active pages: `[{id, chapter, page_number, image_key, created_at}]` |
  | `created_by`      | object   | Creator: `{id, full_name, muid}`                      |
  | `created_at`      | datetime | Creation timestamp                                    |
  | `updated_by`      | object   | Last updater: `{id, full_name, muid}`                 |
  | `updated_at`      | datetime | Last update timestamp                                 |

- **Error Responses:**

  | Status | Condition          | Message               |
  |--------|--------------------|-----------------------|
  | 404    | Chapter not found  | `Chapter not found.`  |

---

### 5.4 Update Chapter

- **API Name:** Update Chapter (Partial)
- **Endpoint:** `PATCH /api/v1/muComics/chapters/<chapter_id>/`
- **Description:** Update chapter details (title, description, chapter_number, cover_image_key). The parent comic cannot be changed. Archived chapters cannot be edited. Slug is regenerated if title changes.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** (all fields optional — partial update)

  | Field             | Type   | Required | Description                                       |
  |-------------------|--------|----------|---------------------------------------------------|
  | `title`           | string | No       | New title (max 150 chars)                          |
  | `description`     | string | No       | New description (nullable)                         |
  | `chapter_number`  | number | No       | New chapter number (≥ 0, unique per comic)         |
  | `cover_image_key` | string | No       | New cover image key (max 255 chars, nullable)      |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/chapters/chapter-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "title": "Chapter 1: A New Beginning"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1: A New Beginning\" updated successfully."]
    },
    "response": { "...full chapter detail object..." }
  }
  ```
- **Response Fields:** Same as Chapter Detail (see [5.3](#53-get-chapter-detail)).
- **Error Responses:**

  | Status | Condition                            | Message                                                      |
  |--------|--------------------------------------|--------------------------------------------------------------|
  | 400    | Validation errors                    | Serializer error details                                     |
  | 400    | Chapter is archived                  | `Archived chapters cannot be edited.`                        |
  | 400    | Attempted to change parent comic     | `Chapter parent comic cannot be modified.`                   |
  | 400    | Duplicate chapter number             | `A chapter with this number already exists for this comic.`  |
  | 403    | Not creator or editor                | `You do not have permission to edit this chapter.`           |
  | 404    | Chapter not found                    | `Chapter not found.`                                         |

---

### 5.5 Delete Chapter

- **API Name:** Delete Chapter (Soft Delete)
- **Endpoint:** `DELETE /api/v1/muComics/chapters/<chapter_id>/`
- **Description:** Soft-delete a chapter and all its linked pages (sets `deleted_at` on both). Only the comic creator can delete chapters.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/chapters/chapter-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1: The Beginning\" deleted successfully."]
    },
    "response": {
      "id": "chapter-uuid-123"
    }
  }
  ```
- **Response Fields:**

  | Field | Type   | Description            |
  |-------|--------|------------------------|
  | `id`  | string | UUID of deleted chapter |

- **Error Responses:**

  | Status | Condition             | Message                                                 |
  |--------|-----------------------|---------------------------------------------------------|
  | 403    | Not the comic creator | `Only the comic creator can delete this chapter.`       |
  | 404    | Chapter not found     | `Chapter not found.`                                    |

---

### 5.6 Publish Chapter

- **API Name:** Publish Chapter
- **Endpoint:** `POST /api/v1/muComics/chapters/<chapter_id>/publish/`
- **Description:** Transition a draft chapter to `published` status. Only the comic creator can publish. Archived chapters cannot be published.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/chapter-uuid-123/publish/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1\" published successfully."]
    },
    "response": {
      "id": "chapter-uuid-123",
      "status": "published"
    }
  }
  ```
- **Response Fields:**

  | Field    | Type   | Description                    |
  |----------|--------|--------------------------------|
  | `id`     | string | Chapter UUID                   |
  | `status` | string | New status (`published`)       |

- **Error Responses:**

  | Status | Condition                     | Message                                                  |
  |--------|-------------------------------|----------------------------------------------------------|
  | 400    | Already published             | `Chapter is already published.`                          |
  | 400    | Chapter is archived           | `Archived chapters cannot be published.`                 |
  | 403    | Not the comic creator         | `Only the comic creator can publish this chapter.`       |
  | 404    | Chapter not found             | `Chapter not found.`                                     |

---

### 5.7 Archive Chapter

- **API Name:** Archive Chapter
- **Endpoint:** `POST /api/v1/muComics/chapters/<chapter_id>/archive/`
- **Description:** Transition a draft or published chapter to `archived` status. Only the comic creator can archive. Already-archived chapters are rejected.
- **Permissions / Access Control:** Comic creator only.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/chapter-uuid-123/archive/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Chapter \"Chapter 1\" archived successfully."]
    },
    "response": {
      "id": "chapter-uuid-123",
      "status": "archived"
    }
  }
  ```
- **Response Fields:**

  | Field    | Type   | Description                  |
  |----------|--------|------------------------------|
  | `id`     | string | Chapter UUID                 |
  | `status` | string | New status (`archived`)      |

- **Error Responses:**

  | Status | Condition                | Message                                                  |
  |--------|--------------------------|----------------------------------------------------------|
  | 400    | Already archived         | `Chapter is already archived.`                           |
  | 403    | Not the comic creator    | `Only the comic creator can archive this chapter.`       |
  | 404    | Chapter not found        | `Chapter not found.`                                     |

---

## 6. Pages

### 6.1 List Pages

- **API Name:** List Chapter Pages
- **Endpoint:** `GET /api/v1/muComics/chapters/<chapter_id>/pages/`
- **Description:** List all active (non-deleted) pages of a chapter. Supports pagination and sorting.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:**

  | Parameter  | Type | Required | Description        |
  |------------|------|----------|--------------------|
  | `sort`     | string | No     | Sort field          |
  | `page`     | int  | No       | Page number         |
  | `per_page` | int  | No       | Items per page      |

- **Supported Ordering Fields:** `page_number`, `created_at`
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/chapters/chapter-uuid-123/pages/?sort=page_number
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "page-uuid",
          "chapter": "chapter-uuid-123",
          "page_number": 1,
          "image_key": "http://domain.com/muback-media/comics/chapters/uuid.jpg",
          "created_at": "2026-07-12T09:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field         | Type     | Description                         |
  |---------------|----------|-------------------------------------|
  | `id`          | string   | Page UUID                           |
  | `chapter`     | string   | Parent chapter UUID                 |
  | `page_number` | integer  | Page ordering number                |
  | `image_key`   | string   | Resolved URL for the page image     |
  | `created_at`  | datetime | Creation timestamp                  |

- **Error Responses:**

  | Status | Condition          | Message               |
  |--------|--------------------|-----------------------|
  | 404    | Chapter not found  | `Chapter not found.`  |

---

### 6.2 Add Page

- **API Name:** Add Page to Chapter
- **Endpoint:** `POST /api/v1/muComics/chapters/<chapter_id>/pages/`
- **Description:** Add a single page to a chapter. The page number must be unique within the chapter. Archived chapters cannot have pages added.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field         | Type    | Required | Description                                       |
  |---------------|---------|----------|---------------------------------------------------|
  | `page_number` | integer | Yes      | Page number (must be ≥ 1, unique within chapter)   |
  | `image_key`   | string  | Yes      | Storage key for the page image (must not be blank) |

- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/chapter-uuid-123/pages/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "page_number": 1,
    "image_key": "comics/chapters/uuid-page1.jpg"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Page added successfully."]
    },
    "response": {
      "id": "generated-page-uuid",
      "chapter": "chapter-uuid-123",
      "page_number": 1,
      "image_key": "http://domain.com/muback-media/comics/chapters/uuid-page1.jpg",
      "created_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Page object (see [6.1](#61-list-pages)).
- **Error Responses:**

  | Status | Condition                              | Message                                                          |
  |--------|----------------------------------------|------------------------------------------------------------------|
  | 400    | Validation errors                      | Serializer error details                                         |
  | 400    | Duplicate page number                  | `A page with this number already exists in this chapter.`        |
  | 400    | Page number < 1                        | `Page number must be a positive integer starting from 1.`        |
  | 400    | Image key blank                        | `Image key must not be blank.`                                   |
  | 400    | Chapter is archived                    | `Archived chapters cannot be edited.`                            |
  | 403    | Not creator or editor                  | `You do not have permission to add a page to this chapter.`      |
  | 404    | Chapter not found                      | `Chapter not found.`                                             |

---

### 6.3 Update Page

- **API Name:** Update Page
- **Endpoint:** `PATCH /api/v1/muComics/chapters/pages/<page_id>/`
- **Description:** Update page information (e.g. `page_number` or `image_key`). Archived chapters cannot have pages edited.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter | Type   | Description        |
  |-----------|--------|--------------------|
  | `page_id` | string | UUID of the page   |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** (all fields optional — partial update)

  | Field         | Type    | Required | Description                                     |
  |---------------|---------|----------|-------------------------------------------------|
  | `page_number` | integer | No       | New page number (≥ 1, unique within chapter)     |
  | `image_key`   | string  | No       | New image key (must not be blank)                |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/chapters/pages/page-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "page_number": 3
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Page updated successfully."]
    },
    "response": {
      "id": "page-uuid-123",
      "chapter": "chapter-uuid",
      "page_number": 3,
      "image_key": "http://domain.com/muback-media/comics/chapters/page.jpg",
      "created_at": "2026-07-12T09:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as Page object (see [6.1](#61-list-pages)).
- **Error Responses:**

  | Status | Condition                         | Message                                                      |
  |--------|-----------------------------------|--------------------------------------------------------------|
  | 400    | Validation errors                 | Serializer error details                                     |
  | 400    | Duplicate page number             | `A page with this number already exists in this chapter.`    |
  | 400    | Chapter is archived               | `Archived chapters cannot be edited.`                        |
  | 403    | Not creator or editor             | `You do not have permission to update this page.`            |
  | 404    | Page not found                    | `Page not found.`                                            |

---

### 6.4 Delete Page

- **API Name:** Delete Page (Soft Delete)
- **Endpoint:** `DELETE /api/v1/muComics/chapters/pages/<page_id>/`
- **Description:** Soft-delete a page (sets `deleted_at`). Archived chapters cannot have pages deleted.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter | Type   | Description        |
  |-----------|--------|--------------------|
  | `page_id` | string | UUID of the page   |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/chapters/pages/page-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Page deleted successfully."]
    },
    "response": {
      "id": "page-uuid-123"
    }
  }
  ```
- **Response Fields:**

  | Field | Type   | Description          |
  |-------|--------|----------------------|
  | `id`  | string | UUID of deleted page |

- **Error Responses:**

  | Status | Condition                | Message                                                  |
  |--------|--------------------------|----------------------------------------------------------|
  | 400    | Chapter is archived      | `Archived chapters cannot be edited.`                    |
  | 403    | Not creator or editor    | `You do not have permission to delete this page.`        |
  | 404    | Page not found           | `Page not found.`                                        |

---

### 6.5 Reorder Pages

- **API Name:** Reorder Pages (Bulk)
- **Endpoint:** `POST /api/v1/muComics/chapters/<chapter_id>/pages/reorder/`
- **Description:** Bulk reorder page numbers within a chapter. **All active pages** in the chapter must be included in the request. Page numbers must be positive, unique, and without duplicates. Archived chapters cannot be reordered.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field         | Type  | Required | Description                                         |
  |---------------|-------|----------|-----------------------------------------------------|
  | `page_orders` | array | Yes      | Array of `{ "id": "<page_uuid>", "page_number": N }`|

- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/chapter-uuid-123/pages/reorder/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "page_orders": [
      { "id": "page-uuid-1", "page_number": 2 },
      { "id": "page-uuid-2", "page_number": 1 },
      { "id": "page-uuid-3", "page_number": 3 }
    ]
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Pages reordered successfully."]
    },
    "response": {
      "success": true
    }
  }
  ```
- **Response Fields:**

  | Field     | Type    | Description            |
  |-----------|---------|------------------------|
  | `success` | boolean | Always `true`          |

- **Error Responses:**

  | Status | Condition                                       | Message                                                                    |
  |--------|-------------------------------------------------|----------------------------------------------------------------------------|
  | 400    | Empty or non-list `page_orders`                 | `page_orders must be a non-empty list of page ordering objects.`           |
  | 400    | Missing `id` or `page_number` in item           | `Each page order item must be an object containing both "id" and "page_number".` |
  | 400    | Duplicate page IDs                              | `Duplicate page IDs in request.`                                           |
  | 400    | Duplicate page numbers                          | `Duplicate page numbers in request.`                                       |
  | 400    | Page numbers < 1                                | `Page numbers must be positive integers starting from 1.`                  |
  | 400    | Not all pages included                          | `All active pages in the chapter must be included in the reorder request.` |
  | 400    | Chapter is archived                             | `Archived chapters cannot be edited.`                                      |
  | 403    | Not creator or editor                           | `You do not have permission to reorder pages.`                             |
  | 404    | Chapter not found                               | `Chapter not found.`                                                       |
  | 404    | Page ID not in chapter                          | `Page ID <id> does not exist in this chapter.`                             |

---

## 7. Uploads

### 7.1 Upload File

- **API Name:** Upload File
- **Endpoint:** `POST /api/v1/muComics/chapters/upload-url/`
- **Description:** Upload a media file (image or PDF) to local storage. Returns the public URL and storage key. Images must be under 5 MB; PDFs must be under 20 MB. Supported image formats: JPG, JPEG, PNG, WebP, GIF. Images are verified for integrity using PIL.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description                   |
  |-----------------|----------|-------------------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>`          |
  | `Content-Type`  | Yes      | `multipart/form-data`         |

- **Request Body:** (`multipart/form-data`)

  | Field  | Type | Required | Description                                                        |
  |--------|------|----------|--------------------------------------------------------------------|
  | `file` | File | Yes      | The file to upload (image or PDF)                                   |

- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/upload-url/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: multipart/form-data

  file: <binary file data>
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["File uploaded successfully."]
    },
    "response": {
      "upload_url": "http://domain.com/muback-media/comics/chapters/uuid.jpg",
      "image_key": "comics/chapters/uuid.jpg"
    }
  }
  ```
- **Response Fields:**

  | Field        | Type   | Description                                             |
  |--------------|--------|---------------------------------------------------------|
  | `upload_url` | string | Full public URL to the uploaded file                    |
  | `image_key`  | string | Storage key (use this when registering pages)           |

- **Error Responses:**

  | Status | Condition                          | Message                                                    |
  |--------|------------------------------------|------------------------------------------------------------|
  | 400    | No file provided                   | `No file provided.`                                        |
  | 400    | Unsupported file type              | `Unsupported file type. Only images and PDFs are allowed.` |
  | 400    | Image exceeds 5 MB                 | `Image file must be under 5 MB.`                           |
  | 400    | PDF exceeds 20 MB                  | `PDF file must be under 20 MB.`                            |
  | 400    | Corrupted/invalid image            | `Invalid or corrupted image file.`                         |

---

### 7.2 Register Uploaded Images as Pages

- **API Name:** Register Uploaded Images as Pages
- **Endpoint:** `POST /api/v1/muComics/chapters/<chapter_id>/pages/register/`
- **Description:** Register multiple previously uploaded image keys as sequential pages in a chapter. Page numbers are auto-assigned starting after the current maximum page number. Archived chapters cannot have pages registered.
- **Permissions / Access Control:** Comic creator or assigned editor.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field        | Type     | Required | Description                                                |
  |--------------|----------|----------|------------------------------------------------------------|
  | `image_keys` | string[] | Yes      | Non-empty array of storage keys (from upload API)          |

- **Sample Request:**
  ```
  POST /api/v1/muComics/chapters/chapter-uuid-123/pages/register/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "image_keys": [
      "comics/chapters/uuid-page1.jpg",
      "comics/chapters/uuid-page2.jpg",
      "comics/chapters/uuid-page3.jpg"
    ]
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Registered 3 pages successfully."]
    },
    "response": [
      {
        "id": "page-uuid-1",
        "chapter": "chapter-uuid-123",
        "page_number": 1,
        "image_key": "http://domain.com/muback-media/comics/chapters/uuid-page1.jpg",
        "created_at": "2026-07-19T08:00:00Z"
      },
      {
        "id": "page-uuid-2",
        "chapter": "chapter-uuid-123",
        "page_number": 2,
        "image_key": "http://domain.com/muback-media/comics/chapters/uuid-page2.jpg",
        "created_at": "2026-07-19T08:00:00Z"
      },
      {
        "id": "page-uuid-3",
        "chapter": "chapter-uuid-123",
        "page_number": 3,
        "image_key": "http://domain.com/muback-media/comics/chapters/uuid-page3.jpg",
        "created_at": "2026-07-19T08:00:00Z"
      }
    ]
  }
  ```
- **Response Fields:** Array of Page objects (see [6.1](#61-list-pages)).
- **Error Responses:**

  | Status | Condition                             | Message                                                         |
  |--------|---------------------------------------|-----------------------------------------------------------------|
  | 400    | Empty or non-list `image_keys`        | `image_keys must be a non-empty list of string paths.`          |
  | 400    | Non-string or empty key in array      | `Each image key must be a non-empty string.`                    |
  | 400    | Chapter is archived                   | `Archived chapters cannot be edited.`                           |
  | 403    | Not creator or editor                 | `You do not have permission to register pages in this chapter.` |
  | 404    | Chapter not found                     | `Chapter not found.`                                            |

---

## 8. Comments

### 8.1 List Comic Comments

- **API Name:** List Comic Comments
- **Endpoint:** `GET /api/v1/muComics/comments/comic/<comic_id>/list/`
- **Description:** List top-level comments on a comic with nested replies. Soft-deleted top-level comments are shown if they have active replies (message replaced with `[deleted]`). Replies are only shown if active.
- **Permissions / Access Control:** Public (no authentication required; `is_owner` field will be `false` for unauthenticated users).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:**

  | Parameter  | Type   | Required | Description                  |
  |------------|--------|----------|------------------------------|
  | `search`   | string | No       | Search comments by `message` |
  | `sort`     | string | No       | Sort field (see below)       |
  | `page`     | int    | No       | Page number                  |
  | `per_page` | int    | No       | Items per page               |

- **Supported Ordering Fields:** `created_at`
- **Headers:**

  | Header          | Required | Description                       |
  |-----------------|----------|-----------------------------------|
  | `Authorization` | No       | `Bearer <JWT token>` (optional)   |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comments/comic/abc-123-def/list/?sort=-created_at&page=1
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "comment-uuid",
          "comic_id": "abc-123-def",
          "chapter_id": null,
          "parent_id": null,
          "user": {
            "id": "user-uuid",
            "full_name": "John Doe",
            "muid": "john@mulearn",
            "profile_pic": "https://..."
          },
          "message": "Great comic!",
          "is_edited": false,
          "is_deleted": false,
          "is_owner": true,
          "reply_count": 2,
          "replies": [
            {
              "id": "reply-uuid",
              "parent_id": "comment-uuid",
              "user": { "id": "...", "full_name": "...", "muid": "...", "profile_pic": "..." },
              "message": "Thanks!",
              "is_edited": false,
              "is_deleted": false,
              "is_owner": false,
              "created_at": "2026-07-15T12:00:00Z",
              "updated_at": "2026-07-15T12:00:00Z"
            }
          ],
          "created_at": "2026-07-15T10:00:00Z",
          "updated_at": "2026-07-15T10:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field         | Type     | Description                                                     |
  |---------------|----------|-----------------------------------------------------------------|
  | `id`          | string   | Comment UUID                                                    |
  | `comic_id`    | string   | Parent comic UUID                                               |
  | `chapter_id`  | string   | Chapter UUID (null for comic-level comments)                    |
  | `parent_id`   | string   | Parent comment UUID (null for top-level)                        |
  | `user`        | object   | Author: `{id, full_name, muid, profile_pic}`                    |
  | `message`     | string   | Comment text (or `[deleted]` if soft-deleted)                   |
  | `is_edited`   | boolean  | Whether the comment was edited after creation                   |
  | `is_deleted`  | boolean  | Whether the comment is soft-deleted                             |
  | `is_owner`    | boolean  | Whether the authenticated user is the author                    |
  | `reply_count` | integer  | Number of active replies                                        |
  | `replies`     | array    | Nested active replies (same shape minus `reply_count`/`replies`)|
  | `created_at`  | datetime | Creation timestamp                                              |
  | `updated_at`  | datetime | Last update timestamp                                           |

- **Error Responses:**

  | Status | Condition       | Message             |
  |--------|-----------------|---------------------|
  | 404    | Comic not found | `Comic not found`   |

---

### 8.2 Create Comic Comment

- **API Name:** Create Comic Comment
- **Endpoint:** `POST /api/v1/muComics/comments/comic/<comic_id>/create/`
- **Description:** Create a top-level comment or reply on a comic. Replies can only target top-level comments (no nested threading). Replies to deleted comments are rejected. The comic's `comment_count` is atomically incremented.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field       | Type   | Required | Description                                                |
  |-------------|--------|----------|------------------------------------------------------------|
  | `message`   | string | Yes      | Comment text (1–2000 chars, must not be empty/whitespace)  |
  | `parent_id` | string | No       | UUID of top-level comment to reply to (null for top-level) |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comments/comic/abc-123-def/create/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "message": "Amazing artwork!",
    "parent_id": null
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comment created successfully"]
    },
    "response": {
      "id": "new-comment-uuid",
      "comic_id": "abc-123-def",
      "chapter_id": null,
      "parent_id": null,
      "user": { "id": "...", "full_name": "...", "muid": "...", "profile_pic": "..." },
      "message": "Amazing artwork!",
      "is_edited": false,
      "created_at": "2026-07-19T08:00:00Z",
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:**

  | Field        | Type     | Description                                |
  |--------------|----------|--------------------------------------------|
  | `id`         | string   | New comment UUID                           |
  | `comic_id`   | string   | Parent comic UUID                          |
  | `chapter_id` | string   | Always `null` for comic comments           |
  | `parent_id`  | string   | Parent comment UUID or `null`              |
  | `user`       | object   | Author: `{id, full_name, muid, profile_pic}` |
  | `message`    | string   | Comment text                               |
  | `is_edited`  | boolean  | Always `false` on creation                 |
  | `created_at` | datetime | Creation timestamp                         |
  | `updated_at` | datetime | Same as `created_at` on creation           |

- **Error Responses:**

  | Status | Condition                                      | Message                                                     |
  |--------|------------------------------------------------|-------------------------------------------------------------|
  | 400    | Validation errors                              | Serializer error details                                    |
  | 400    | Empty/whitespace message                       | `Message cannot be empty or whitespace only.`               |
  | 400    | Parent comment not found                       | `Parent comment not found`                                  |
  | 400    | Reply to non-top-level comment                 | `Replies can only be made to top-level comments`            |
  | 400    | Parent belongs to different comic              | `Parent comment does not belong to this comic`              |
  | 400    | Reply to deleted comment                       | `Cannot reply to a deleted comment`                         |
  | 400    | Parent is a chapter comment                    | `Replies to chapter comments must use the chapter endpoint` |
  | 404    | Comic not found                                | `Comic not found`                                           |

---

### 8.3 List Chapter Comments

- **API Name:** List Chapter Comments
- **Endpoint:** `GET /api/v1/muComics/comments/chapter/<chapter_id>/list/`
- **Description:** List top-level comments on a chapter with nested replies. Same soft-delete visibility rules as comic comments.
- **Permissions / Access Control:** Public (no authentication required).
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:**

  | Parameter  | Type   | Required | Description                  |
  |------------|--------|----------|------------------------------|
  | `search`   | string | No       | Search comments by `message` |
  | `sort`     | string | No       | Sort field                   |
  | `page`     | int    | No       | Page number                  |
  | `per_page` | int    | No       | Items per page               |

- **Supported Ordering Fields:** `created_at`
- **Headers:**

  | Header          | Required | Description                       |
  |-----------------|----------|-----------------------------------|
  | `Authorization` | No       | `Bearer <JWT token>` (optional)   |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comments/chapter/chapter-uuid-123/list/
  ```
- **Success Response:** `200 OK` — Same shape as [8.1 List Comic Comments](#81-list-comic-comments).
- **Response Fields:** Same as [8.1](#81-list-comic-comments).
- **Error Responses:**

  | Status | Condition          | Message               |
  |--------|--------------------|-----------------------|
  | 404    | Chapter not found  | `Chapter not found`   |

---

### 8.4 Create Chapter Comment

- **API Name:** Create Chapter Comment
- **Endpoint:** `POST /api/v1/muComics/comments/chapter/<chapter_id>/create/`
- **Description:** Create a top-level comment or reply on a chapter. Same validation rules as comic comment creation. The parent comic's `comment_count` is atomically incremented.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `chapter_id` | string | UUID of the chapter    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field       | Type   | Required | Description                                                |
  |-------------|--------|----------|------------------------------------------------------------|
  | `message`   | string | Yes      | Comment text (1–2000 chars, must not be empty/whitespace)  |
  | `parent_id` | string | No       | UUID of top-level comment to reply to (null for top-level) |

- **Sample Request:**
  ```
  POST /api/v1/muComics/comments/chapter/chapter-uuid-123/create/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "message": "This chapter is fire!",
    "parent_id": null
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comment created successfully"]
    },
    "response": {
      "id": "new-comment-uuid",
      "comic_id": "parent-comic-uuid",
      "chapter_id": "chapter-uuid-123",
      "parent_id": null,
      "user": { "id": "...", "full_name": "...", "muid": "...", "profile_pic": "..." },
      "message": "This chapter is fire!",
      "is_edited": false,
      "created_at": "2026-07-19T08:00:00Z",
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
- **Response Fields:** Same as [8.2 Create Comic Comment](#82-create-comic-comment), with `chapter_id` populated.
- **Error Responses:**

  | Status | Condition                                  | Message                                                      |
  |--------|--------------------------------------------|--------------------------------------------------------------|
  | 400    | Validation errors                          | Serializer error details                                     |
  | 400    | Empty/whitespace message                   | `Message cannot be empty or whitespace only.`                |
  | 400    | Parent comment not found                   | `Parent comment not found`                                   |
  | 400    | Reply to non-top-level                     | `Replies can only be made to top-level comments`             |
  | 400    | Parent belongs to different chapter        | `Parent comment does not belong to this chapter`             |
  | 400    | Reply to deleted comment                   | `Cannot reply to a deleted comment`                          |
  | 404    | Chapter not found                          | `Chapter not found`                                          |

---

### 8.5 Edit Comment

- **API Name:** Edit Comment
- **Endpoint:** `PATCH /api/v1/muComics/comments/<comment_id>/`
- **Description:** Edit the message of your own comment. Deleted comments cannot be edited. The `is_edited` flag becomes `true` after the first edit (determined by `updated_at > created_at`).
- **Permissions / Access Control:** Comment owner only.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `comment_id` | string | UUID of the comment    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field     | Type   | Required | Description                                              |
  |-----------|--------|----------|----------------------------------------------------------|
  | `message` | string | Yes      | New message text (1–2000 chars, must not be empty/whitespace) |

- **Sample Request:**
  ```
  PATCH /api/v1/muComics/comments/comment-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "message": "Updated: Great comic!"
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comment updated successfully"]
    },
    "response": {
      "id": "comment-uuid-123",
      "message": "Updated: Great comic!",
      "is_edited": true,
      "updated_at": "2026-07-19T09:00:00Z"
    }
  }
  ```
- **Response Fields:**

  | Field        | Type     | Description                        |
  |--------------|----------|------------------------------------|
  | `id`         | string   | Comment UUID                       |
  | `message`    | string   | Updated message text               |
  | `is_edited`  | boolean  | `true` after editing               |
  | `updated_at` | datetime | Update timestamp                   |

- **Error Responses:**

  | Status | Condition                    | Message                                  |
  |--------|------------------------------|------------------------------------------|
  | 400    | Comment is deleted           | `Cannot edit a deleted comment`           |
  | 400    | Validation errors            | Serializer error details                 |
  | 403    | Not the comment owner        | `You can only edit your own comments`    |
  | 404    | Comment not found            | `Comment not found`                      |

---

### 8.6 Delete Comment (User)

- **API Name:** Delete Comment (User Soft Delete)
- **Endpoint:** `DELETE /api/v1/muComics/comments/<comment_id>/`
- **Description:** Soft-delete your own comment (sets `deleted_at`). The parent comic's `comment_count` is atomically decremented.
- **Permissions / Access Control:** Comment owner only.
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `comment_id` | string | UUID of the comment    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comments/comment-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comment deleted successfully"]
    }
  }
  ```
- **Response Fields:** Message confirmation only.
- **Error Responses:**

  | Status | Condition                    | Message                                     |
  |--------|------------------------------|---------------------------------------------|
  | 400    | Already deleted              | `Comment has already been deleted`           |
  | 403    | Not the comment owner        | `You can only delete your own comments`     |
  | 404    | Comment not found            | `Comment not found`                         |

---

### 8.7 Admin — List All Comments

- **API Name:** Admin List All Comments
- **Endpoint:** `GET /api/v1/muComics/comments/admin/`
- **Description:** List all comments for moderation in a flat list (no nesting). Supports filtering by comic, chapter, user, and deletion status. Returns additional metadata like comic/chapter titles and who deleted a comment.
- **Permissions / Access Control:** **Comic Admin role required.**
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter         | Type   | Required | Description                                                          |
  |-------------------|--------|----------|----------------------------------------------------------------------|
  | `comic_id`        | string | No       | Filter by comic UUID                                                  |
  | `chapter_id`      | string | No       | Filter by chapter UUID                                                |
  | `user_id`         | string | No       | Filter by user UUID                                                   |
  | `include_deleted` | string | No       | `true` to include deleted comments (default: `false`)                 |
  | `deleted_only`    | string | No       | `true` to show only deleted comments                                  |
  | `search`          | string | No       | Search comments by `message`                                          |
  | `sort`            | string | No       | Sort field                                                            |
  | `page`            | int    | No       | Page number                                                           |
  | `per_page`        | int    | No       | Items per page                                                        |

- **Supported Ordering Fields:** `created_at`
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/comments/admin/?comic_id=abc-123&deleted_only=true&sort=-created_at
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "comment-uuid",
          "comic_id": "abc-123-def",
          "comic_title": "Adventures of MuBot",
          "chapter_id": null,
          "chapter_title": null,
          "parent_id": null,
          "user": { "id": "...", "full_name": "...", "muid": "...", "profile_pic": "..." },
          "message": "Offensive comment text",
          "is_edited": false,
          "is_deleted": true,
          "deleted_at": "2026-07-18T14:00:00Z",
          "deleted_by_user": { "id": "admin-uuid", "full_name": "Admin User", "muid": "admin@mulearn" },
          "reply_count": 0,
          "created_at": "2026-07-17T10:00:00Z",
          "updated_at": "2026-07-18T14:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                                                    |
  |-------------------|----------|----------------------------------------------------------------|
  | `id`              | string   | Comment UUID                                                   |
  | `comic_id`        | string   | Parent comic UUID                                              |
  | `comic_title`     | string   | Parent comic title                                             |
  | `chapter_id`      | string   | Chapter UUID (nullable)                                        |
  | `chapter_title`   | string   | Chapter title (nullable)                                       |
  | `parent_id`       | string   | Parent comment UUID (nullable)                                 |
  | `user`            | object   | Author: `{id, full_name, muid, profile_pic}`                   |
  | `message`         | string   | Comment text (not redacted in admin view)                      |
  | `is_edited`       | boolean  | Whether the comment was edited                                 |
  | `is_deleted`      | boolean  | Whether the comment is soft-deleted                            |
  | `deleted_at`      | datetime | Deletion timestamp (nullable)                                  |
  | `deleted_by_user` | object   | User who deleted: `{id, full_name, muid}` (nullable)          |
  | `reply_count`     | integer  | Number of active replies                                       |
  | `created_at`      | datetime | Creation timestamp                                             |
  | `updated_at`      | datetime | Last update timestamp                                          |

- **Error Responses:**

  | Status | Condition   | Message        |
  |--------|-------------|----------------|
  | 403    | Not admin   | Unauthorized   |

---

### 8.8 Admin — Delete Comment

- **API Name:** Admin Delete Comment
- **Endpoint:** `DELETE /api/v1/muComics/comments/admin/<comment_id>/`
- **Description:** Moderator soft-delete of any comment (sets `deleted_at`). The parent comic's `comment_count` is atomically decremented.
- **Permissions / Access Control:** **Comic Admin role required.**
- **Path Parameters:**

  | Parameter    | Type   | Description            |
  |--------------|--------|------------------------|
  | `comment_id` | string | UUID of the comment    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/comments/admin/comment-uuid-123/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Comment removed by moderator"]
    }
  }
  ```
- **Response Fields:** Message confirmation only.
- **Error Responses:**

  | Status | Condition              | Message                             |
  |--------|------------------------|-------------------------------------|
  | 400    | Already deleted        | `Comment has already been deleted`  |
  | 403    | Not admin              | Unauthorized                        |
  | 404    | Comment not found      | `Comment not found`                 |

---

## 9. Reader

### 9.1 Like Comic

- **API Name:** Like Comic
- **Endpoint:** `POST /api/v1/muComics/reader/comics/<comic_id>/likes/`
- **Description:** Like a published comic. The comic's `like_count` is atomically incremented. Each user can only like a comic once — duplicate likes return a 409 conflict.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/reader/comics/abc-123-def/likes/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "message": "Comic liked successfully"
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description          |
  |-----------|--------|----------------------|
  | `message` | string | Success confirmation |

- **Error Responses:**

  | Status | Condition                          | Message                                    |
  |--------|------------------------------------|--------------------------------------------|
  | 400    | Comic not found / not published    | `Comic not found or not available.`        |
  | 400    | Validation failed                  | Serializer error details                   |
  | 409    | Already liked                      | `Comic already liked.`                     |

---

### 9.2 Unlike Comic

- **API Name:** Unlike Comic
- **Endpoint:** `DELETE /api/v1/muComics/reader/comics/<comic_id>/likes/`
- **Description:** Remove a like from a comic. The comic's `like_count` is atomically decremented (minimum 0).
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/reader/comics/abc-123-def/likes/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "message": "Comic unliked successfully"
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description          |
  |-----------|--------|----------------------|
  | `message` | string | Success confirmation |

- **Error Responses:**

  | Status | Condition          | Message                    |
  |--------|--------------------|----------------------------|
  | 404    | Like not found     | `Comic like not found.`    |

---

### 9.3 Bookmark Comic

- **API Name:** Bookmark Comic
- **Endpoint:** `POST /api/v1/muComics/reader/comics/<comic_id>/bookmarks/`
- **Description:** Bookmark a published comic. The comic's `bookmark_count` is atomically incremented. Each user can only bookmark a comic once — duplicate bookmarks return a 409 conflict.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  POST /api/v1/muComics/reader/comics/abc-123-def/bookmarks/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "message": "Comic bookmarked successfully"
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description          |
  |-----------|--------|----------------------|
  | `message` | string | Success confirmation |

- **Error Responses:**

  | Status | Condition                          | Message                                    |
  |--------|------------------------------------|--------------------------------------------|
  | 400    | Comic not found / not published    | `Comic not found or not available.`        |
  | 400    | Validation failed                  | Serializer error details                   |
  | 409    | Already bookmarked                 | `Comic already bookmarked.`                |

---

### 9.4 Remove Bookmark

- **API Name:** Remove Bookmark
- **Endpoint:** `DELETE /api/v1/muComics/reader/comics/<comic_id>/bookmarks/`
- **Description:** Remove a bookmark from a comic. The comic's `bookmark_count` is atomically decremented (minimum 0).
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  DELETE /api/v1/muComics/reader/comics/abc-123-def/bookmarks/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "message": "Comic bookmark removed successfully"
    }
  }
  ```
- **Response Fields:**

  | Field     | Type   | Description          |
  |-----------|--------|----------------------|
  | `message` | string | Success confirmation |

- **Error Responses:**

  | Status | Condition             | Message                          |
  |--------|------------------------|----------------------------------|
  | 404    | Bookmark not found     | `Comic bookmark not found.`     |

---

### 9.5 Get Interaction Status

- **API Name:** Get Interaction Status
- **Endpoint:** `GET /api/v1/muComics/reader/comics/<comic_id>/interaction-status/`
- **Description:** Check whether the authenticated user has liked and/or bookmarked a specific comic.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/reader/comics/abc-123-def/interaction-status/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "is_liked": true,
      "is_bookmarked": false
    }
  }
  ```
- **Response Fields:**

  | Field           | Type    | Description                               |
  |-----------------|---------|-------------------------------------------|
  | `is_liked`      | boolean | `true` if the user has liked the comic    |
  | `is_bookmarked` | boolean | `true` if the user has bookmarked the comic |

- **Error Responses:** None (always returns 200 with boolean flags).

---

### 9.6 Get Reading Progress

- **API Name:** Get Reading Progress
- **Endpoint:** `GET /api/v1/muComics/reader/comics/<comic_id>/progress/`
- **Description:** Retrieve the authenticated user's reading progress for a specific comic. Returns `null` response data if no progress has been saved yet.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/reader/comics/abc-123-def/progress/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "id": "progress-uuid",
      "comic_id": "abc-123-def",
      "last_chapter_id": "chapter-uuid-123",
      "last_page_number": 5,
      "updated_at": "2026-07-19T08:00:00Z"
    }
  }
  ```
  **When no progress exists:**
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": null
  }
  ```
- **Response Fields:**

  | Field              | Type     | Description                                           |
  |--------------------|----------|-------------------------------------------------------|
  | `id`               | string   | Reading progress UUID                                 |
  | `comic_id`         | string   | Comic UUID                                            |
  | `last_chapter_id`  | string   | UUID of the last chapter read (nullable)              |
  | `last_page_number` | integer  | Last page number read within the chapter (nullable)   |
  | `updated_at`       | datetime | When progress was last updated                        |

- **Error Responses:** None (returns `null` response if no progress exists).

---

### 9.7 Save / Update Reading Progress

- **API Name:** Save / Update Reading Progress
- **Endpoint:** `PUT /api/v1/muComics/reader/comics/<comic_id>/progress/`
- **Description:** Save or update reading progress for a comic. Uses upsert semantics — creates a new progress record if none exists, otherwise updates the existing one. The comic must be published and active. If a `last_chapter_id` is provided, the chapter must belong to the comic and be published. If `last_page_number` is provided, the page must exist in the specified chapter.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:**

  | Parameter  | Type   | Description          |
  |------------|--------|----------------------|
  | `comic_id` | string | UUID of the comic    |

- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:**

  | Field              | Type    | Required | Description                                                           |
  |--------------------|---------|----------|-----------------------------------------------------------------------|
  | `last_chapter_id`  | string  | No       | UUID of the last chapter read (nullable)                              |
  | `last_page_number` | integer | No       | Last page number read (nullable; requires `last_chapter_id` if set)   |

- **Sample Request:**
  ```
  PUT /api/v1/muComics/reader/comics/abc-123-def/progress/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  Content-Type: application/json

  {
    "last_chapter_id": "chapter-uuid-123",
    "last_page_number": 12
  }
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": ["Reading progress saved successfully"]
    },
    "response": {
      "id": "progress-uuid",
      "comic_id": "abc-123-def",
      "last_chapter_id": "chapter-uuid-123",
      "last_page_number": 12,
      "updated_at": "2026-07-19T09:00:00Z"
    }
  }
  ```
- **Response Fields:**

  | Field              | Type     | Description                                           |
  |--------------------|----------|-------------------------------------------------------|
  | `id`               | string   | Reading progress UUID                                 |
  | `comic_id`         | string   | Comic UUID                                            |
  | `last_chapter_id`  | string   | UUID of the last chapter read (nullable)              |
  | `last_page_number` | integer  | Last page number read (nullable)                      |
  | `updated_at`       | datetime | When progress was saved/updated                       |

- **Error Responses:**

  | Status | Condition                                      | Message                                                                    |
  |--------|------------------------------------------------|----------------------------------------------------------------------------|
  | 400    | Comic not found / not published                | `Comic not found or not available.`                                        |
  | 400    | Chapter not found / not published / wrong comic| `Chapter not found, not published, or does not belong to this comic.`      |
  | 400    | Page not found in chapter                      | `Page not found in this chapter.`                                          |
  | 400    | Page number without chapter                    | `Cannot provide page number without chapter.`                              |
  | 400    | Validation failed                              | Serializer error details                                                   |

---

### 9.8 Reader Dashboard

- **API Name:** Reader Dashboard
- **Endpoint:** `GET /api/v1/muComics/reader/me/`
- **Description:** Retrieve the authenticated user's reader dashboard overview, including user info and aggregate stats (total likes, total bookmarks, in-progress comics count).
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:** None
- **Supported Ordering Fields:** N/A
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/reader/me/
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "user": {
        "id": "user-uuid",
        "full_name": "John Doe",
        "muid": "john@mulearn"
      },
      "stats": {
        "total_likes": 15,
        "total_bookmarks": 8,
        "in_progress": 3
      }
    }
  }
  ```
- **Response Fields:**

  | Field                   | Type    | Description                             |
  |-------------------------|---------|-----------------------------------------|
  | `user`                  | object  | User info: `{id, full_name, muid}`      |
  | `stats.total_likes`     | integer | Total comics liked by the user          |
  | `stats.total_bookmarks` | integer | Total comics bookmarked by the user     |
  | `stats.in_progress`     | integer | Total comics with reading progress      |

- **Error Responses:**

  | Status | Condition        | Message            |
  |--------|------------------|--------------------|  
  | 404    | User not found   | `User not found`   |

---

### 9.9 My Bookmarks

- **API Name:** My Bookmarks
- **Endpoint:** `GET /api/v1/muComics/reader/me/bookmarks/`
- **Description:** List all comics bookmarked by the authenticated user. Only includes bookmarks for active (non-deleted) comics. Supports pagination and search by comic title or slug.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter  | Type   | Required | Description                                 |
  |------------|--------|----------|---------------------------------------------|
  | `search`   | string | No       | Search by comic `title` or `slug`           |
  | `page`     | int    | No       | Page number                                 |
  | `per_page` | int    | No       | Items per page                              |

- **Supported Ordering Fields:** Default ordered by `created_at` descending (most recently bookmarked first).
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/reader/me/bookmarks/?search=mubot&page=1
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "bookmark-link-uuid",
          "comic_id": "abc-123-def",
          "title": "Adventures of MuBot",
          "slug": "adventures-of-mubot",
          "cover_image_key": "comics/covers/mubot-cover.jpg",
          "bookmarked_at": "2026-07-15T10:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field             | Type     | Description                      |
  |-------------------|----------|----------------------------------|
  | `id`              | string   | Bookmark link UUID               |
  | `comic_id`        | string   | Comic UUID                       |
  | `title`           | string   | Comic title                      |
  | `slug`            | string   | Comic slug                       |
  | `cover_image_key` | string   | Comic cover image key (nullable) |
  | `bookmarked_at`   | datetime | When the comic was bookmarked    |

- **Error Responses:** None (returns empty data array if no bookmarks).

---

### 9.10 My Reading Progress

- **API Name:** My Reading Progress
- **Endpoint:** `GET /api/v1/muComics/reader/me/progress/`
- **Description:** List all reading progress entries for the authenticated user. Only includes progress for active (non-deleted) comics. Supports pagination and search by comic title or slug. Ordered by most recently updated first.
- **Permissions / Access Control:** Authenticated user (any role).
- **Path Parameters:** None
- **Query Parameters:**

  | Parameter  | Type   | Required | Description                                 |
  |------------|--------|----------|---------------------------------------------|
  | `search`   | string | No       | Search by comic `title` or `slug`           |
  | `page`     | int    | No       | Page number                                 |
  | `per_page` | int    | No       | Items per page                              |

- **Supported Ordering Fields:** Default ordered by `updated_at` descending (most recently read first).
- **Headers:**

  | Header          | Required | Description          |
  |-----------------|----------|----------------------|
  | `Authorization` | Yes      | `Bearer <JWT token>` |

- **Request Body:** None
- **Sample Request:**
  ```
  GET /api/v1/muComics/reader/me/progress/?page=1
  Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
  ```
- **Success Response:** `200 OK`
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "response": {
      "data": [
        {
          "id": "progress-uuid",
          "comic_id": "abc-123-def",
          "title": "Adventures of MuBot",
          "slug": "adventures-of-mubot",
          "cover_image_key": "comics/covers/mubot-cover.jpg",
          "last_chapter_id": "chapter-uuid-123",
          "last_page_number": 12,
          "last_read_at": "2026-07-19T08:00:00Z"
        }
      ],
      "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
    }
  }
  ```
- **Response Fields:**

  | Field              | Type     | Description                                            |
  |--------------------|----------|--------------------------------------------------------|
  | `id`               | string   | Progress record UUID                                   |
  | `comic_id`         | string   | Comic UUID                                             |
  | `title`            | string   | Comic title                                            |
  | `slug`             | string   | Comic slug                                             |
  | `cover_image_key`  | string   | Comic cover image key (nullable)                       |
  | `last_chapter_id`  | string   | UUID of the last chapter read (nullable)               |
  | `last_page_number` | integer  | Last page number read (nullable)                       |
  | `last_read_at`     | datetime | When the progress was last updated                     |

- **Error Responses:** None (returns empty data array if no progress).

---

## Appendix

### A. Status Workflow

```
┌─────────┐   publish   ┌───────────┐
│  DRAFT  │ ──────────→ │ PUBLISHED │
└─────────┘             └───────────┘
     │                        │
     │       archive          │  archive
     └──────────┐   ┌────────┘
                ▼   ▼
           ┌──────────┐
           │ ARCHIVED  │
           └──────────┘
```

- **Draft → Published:** Requires title & description to be filled.
- **Draft → Archived:** Allowed.
- **Published → Archived:** Allowed.
- **Archived → Published:** ❌ Not allowed.
- **Archived → Draft:** ❌ Not allowed (no unarchive endpoint for comics; genres have reinstate).

### B. Contributor Roles

| Role       | Value      | Assignable via API | Description                |
|------------|------------|-------------------|----------------------------|
| Creator    | `creator`  | ❌ No              | Set via `comic.created_by` |
| Writer     | `writer`   | ✅ Yes             | Script/story writer        |
| Artist     | `artist`   | ✅ Yes             | Visual artist              |
| Colorist   | `colorist` | ✅ Yes             | Color specialist           |
| Editor     | `editor`   | ✅ Yes             | Can edit comic & chapters  |

### C. Soft Delete Pattern

All entities (Comics, Chapters, Pages, Comments) use **soft deletion**:
- `deleted_at` is set to the current timestamp.
- `deleted_by` references the user who performed the deletion.
- Soft-deleted records are excluded from all list and detail queries.
- For Comments: soft-deleted top-level comments with active replies remain visible with `message = "[deleted]"`.

### D. Pagination Response Shape

All paginated endpoints return:
```json
{
  "data": [ ... ],
  "pagination": {
    "count": 50,
    "totalPages": 5,
    "isNext": true,
    "isPrev": false,
    "nextPage": 2,
    "prevPage": null
  }
}
```
