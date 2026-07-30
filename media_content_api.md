# Media Content API — Comprehensive Documentation

> Base URL prefix: `/api/v1/dashboard/media-content/`

This module exposes CRUD endpoints for four content types (three previously managed via CMS, plus one native addition), all stored in a single `media_content` database table with a `content_type` discriminator.

---

## Table of Contents

- [Authentication & Roles](#authentication--roles)
- [Common Response Envelope](#common-response-envelope)
- [Common Query Parameters](#common-query-parameters-list-endpoints)
- [Office Hours](#-office-hours)
- [Salt Mango Tree](#-salt-mango-tree)
- [Inspiration Station Radio](#-inspiration-station-radio)
- [Grab Your Superpowers](#-grab-your-superpowers)
- [Error Reference](#error-reference)
- [Enum Reference](#enum-reference)

---

## Authentication & Roles

| Operation | Auth Required | Role Required |
|-----------|--------------|---------------|
| `GET` list / detail | ❌ Public | None |
| `POST` create | ✅ Bearer JWT | `admin` |
| `PATCH` update | ✅ Bearer JWT | `admin` |
| `DELETE` soft-delete | ✅ Bearer JWT | `admin` |

**Header for write operations:**
```
Authorization: Bearer <jwt_token>
```

> **Note:** DELETE is a **soft delete** — `deleted_at` is set and the record is excluded from all future list/detail queries. Data is **not** removed from the database.

---

## Common Response Envelope

**Success:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Success message"] },
  "response": {}
}
```

**Paginated list:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [],
    "pagination": {
      "count": 42,
      "totalPages": 5,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

**Error:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Error message"] },
  "response": {}
}
```

---

## Common Query Parameters (List Endpoints)

| Parameter | Type | Description |
|-----------|------|-------------|
| `pageIndex` | integer | Page number (default: `1`) |
| `perPage` | integer | Items per page (default: `10`) |
| `search` | string | Case-insensitive text search across indexed fields |
| `sortBy` | string | Field to sort by. Prefix with `-` for descending. e.g. `-date` |
| `status` | `upcoming` / `ongoing` / `completed` | Filter by computed status based on date |
| `zone` | string | `north` / `central` / `south` — SMT & Inspiration Station only |

---

## 📅 Office Hours

**Content type discriminator:** `office_hours`
**Date input format:** `DD/MM/YYYY` | **Date output format:** `YYYY-MM-DD`

### Field Reference

| Field | Type | Required | Max Length | Notes |
|-------|------|----------|------------|-------|
| `title` | string | ✅ | 300 | Session title |
| `date` | string | ✅ | — | Must be `DD/MM/YYYY` |
| `performer` | string | ❌ | 200 | Speaker / host name |
| `designation` | string | ❌ | 200 | e.g. "Senior Developer" |
| `description` | string | ❌ | unlimited | Session description |
| `link` | URL | ❌ | 500 | Meeting or streaming link |
| `interest_groups` | string[] | ❌ | — | Array of IG slugs |
| `poster_thumbnail` | string | ❌ | 512 | Image URL |

---

### `GET /api/v1/dashboard/media-content/office-hours/`

List all active Office Hours sessions. **Public.**

**Search fields:** `title`, `performer`, `description`

**Example:**
```
GET /api/v1/dashboard/media-content/office-hours/?status=upcoming&pageIndex=1&perPage=5
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "title": "Intro to REST APIs with Django",
        "performer": "Alice Thomas",
        "designation": "Senior Developer",
        "description": "A hands-on session covering DRF best practices.",
        "date": "2025-08-15",
        "link": "https://meet.google.com/xyz-abc",
        "interest_groups": ["web-development", "ai"],
        "poster_thumbnail": "https://cdn.example.com/poster1.jpg",
        "status": "upcoming",
        "created_at": "2025-06-27T06:30:00.000000Z",
        "updated_at": "2025-06-27T06:30:00.000000Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

### `POST /api/v1/dashboard/media-content/office-hours/`

Create an Office Hours session. **Admin only.**

**Full request body:**
```json
{
  "title": "Introduction to Web3",
  "date": "15/09/2025",
  "performer": "Priya Nair",
  "designation": "Blockchain Developer",
  "description": "Explore decentralised applications and smart contracts.",
  "link": "https://meet.google.com/web3-session",
  "interest_groups": ["blockchain", "web-development"],
  "poster_thumbnail": "https://cdn.example.com/posters/web3.jpg"
}
```

**Minimal request (required fields only):**
```json
{
  "title": "Quick Dev Talk",
  "date": "20/09/2025"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Office Hours session created successfully."] },
  "response": {
    "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
    "title": "Introduction to Web3",
    "performer": "Priya Nair",
    "designation": "Blockchain Developer",
    "description": "Explore decentralised applications and smart contracts.",
    "date": "2025-09-15",
    "link": "https://meet.google.com/web3-session",
    "interest_groups": ["blockchain", "web-development"],
    "poster_thumbnail": "https://cdn.example.com/posters/web3.jpg",
    "status": "upcoming",
    "created_at": "2025-06-27T06:45:00.000000Z",
    "updated_at": "2025-06-27T06:45:00.000000Z"
  }
}
```

**400 — Missing required fields:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "title": ["This field is required."],
    "date": ["This field is required."]
  },
  "response": {}
}
```

**400 — Wrong date format:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "date": ["Invalid date format. Expected DD/MM/YYYY (e.g. 27/06/2025)."]
  },
  "response": {}
}
```

**400 — Non-admin:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["You do not have the required role to access this page."]
  },
  "response": {}
}
```

---

### `GET /api/v1/dashboard/media-content/office-hours/{record_id}/`

Retrieve a single session. **Public.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Office Hours session retrieved."] },
  "response": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "title": "Intro to REST APIs with Django",
    "performer": "Alice Thomas",
    "designation": "Senior Developer",
    "description": "A hands-on session covering DRF best practices.",
    "date": "2025-08-15",
    "link": "https://meet.google.com/xyz-abc",
    "interest_groups": ["web-development", "ai"],
    "poster_thumbnail": "https://cdn.example.com/poster1.jpg",
    "status": "upcoming",
    "created_at": "2025-06-27T06:30:00.000000Z",
    "updated_at": "2025-06-27T06:30:00.000000Z"
  }
}
```

**400 — Not found or soft-deleted:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Office Hours session not found."] },
  "response": {}
}
```

---

### `PATCH /api/v1/dashboard/media-content/office-hours/{record_id}/`

Partially update a session. **Admin only.** All fields optional.

**Request (update link + interest groups):**
```json
{
  "link": "https://meet.google.com/new-link",
  "interest_groups": ["ai", "generative-ai"]
}
```

**Request (update date):**
```json
{
  "date": "30/09/2025"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Office Hours session updated."] },
  "response": {
    "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "title": "Intro to REST APIs with Django",
    "performer": "Alice Thomas",
    "designation": "Senior Developer",
    "description": "A hands-on session covering DRF best practices.",
    "date": "2025-09-30",
    "link": "https://meet.google.com/new-link",
    "interest_groups": ["ai", "generative-ai"],
    "poster_thumbnail": "https://cdn.example.com/poster1.jpg",
    "status": "upcoming",
    "created_at": "2025-06-27T06:30:00.000000Z",
    "updated_at": "2025-06-27T08:15:00.000000Z"
  }
}
```

---

### `DELETE /api/v1/dashboard/media-content/office-hours/{record_id}/`

Soft-delete a session. **Admin only.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Office Hours session deleted."] },
  "response": {}
}
```

**400 — Record not found:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Office Hours session not found."] },
  "response": {}
}
```

---

## 🥭 Salt Mango Tree

**Content type discriminator:** `salt_mango_tree`
**Date format:** `YYYY-MM-DD`

> **Note:** The CMS field `topic` is accepted in write requests and returned in responses. It maps internally to the `title` column in the database.

### Field Reference

| Field | Type | Required | Max Length | Notes |
|-------|------|----------|------------|-------|
| `topic` | string | ✅ | 300 | Episode topic |
| `campus` | string | ✅ | 200 | Campus name |
| `date` | string | ✅ | — | Must be `YYYY-MM-DD` |
| `zone` | enum | ❌ | — | `north` / `central` / `south` |
| `description` | string | ❌ | unlimited | Episode description |
| `link` | URL | ❌ | 500 | Streaming link |

---

### `GET /api/v1/dashboard/media-content/salt-mango-tree/`

List active SMT episodes. **Public.**

**Search fields:** `topic`, `campus`, `description`

**Example:**
```
GET /api/v1/dashboard/media-content/salt-mango-tree/?zone=north&status=upcoming
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
        "topic": "Building Sustainable Startups",
        "campus": "NIT Calicut",
        "zone": "north",
        "date": "2025-08-20",
        "description": "Student founders share their journey from idea to product.",
        "link": "https://youtube.com/live/smt-ep12",
        "status": "upcoming",
        "created_at": "2025-06-27T09:00:00.000000Z",
        "updated_at": "2025-06-27T09:00:00.000000Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

### `POST /api/v1/dashboard/media-content/salt-mango-tree/`

Create an SMT episode. **Admin only.**

**Full request body:**
```json
{
  "topic": "AI in Agriculture",
  "campus": "Kerala Agricultural University",
  "zone": "south",
  "date": "2025-10-05",
  "description": "How students are using AI to solve farming challenges.",
  "link": "https://youtube.com/live/smt-ep15"
}
```

**Minimal request:**
```json
{
  "topic": "Open Source Culture",
  "campus": "IIT Palakkad",
  "date": "2025-10-15"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Salt Mango Tree episode created successfully."] },
  "response": {
    "id": "e5f6a7b8-c9d0-1234-efab-345678901234",
    "topic": "AI in Agriculture",
    "campus": "Kerala Agricultural University",
    "zone": "south",
    "date": "2025-10-05",
    "description": "How students are using AI to solve farming challenges.",
    "link": "https://youtube.com/live/smt-ep15",
    "status": "upcoming",
    "created_at": "2025-06-27T09:15:00.000000Z",
    "updated_at": "2025-06-27T09:15:00.000000Z"
  }
}
```

**400 — Missing required fields:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "campus": ["This field is required."]
  },
  "response": {}
}
```

**400 — Invalid zone value:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "zone": ["\"east\" is not a valid choice."]
  },
  "response": {}
}
```

**400 — Wrong date format (e.g. DD/MM/YYYY sent instead of YYYY-MM-DD):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "date": ["Date has wrong format. Use one of these formats instead: YYYY-MM-DD."]
  },
  "response": {}
}
```

---

### `GET /api/v1/dashboard/media-content/salt-mango-tree/{record_id}/`

Retrieve a single SMT episode. **Public.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Salt Mango Tree episode retrieved."] },
  "response": {
    "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
    "topic": "Building Sustainable Startups",
    "campus": "NIT Calicut",
    "zone": "north",
    "date": "2025-08-20",
    "description": "Student founders share their journey from idea to product.",
    "link": "https://youtube.com/live/smt-ep12",
    "status": "upcoming",
    "created_at": "2025-06-27T09:00:00.000000Z",
    "updated_at": "2025-06-27T09:00:00.000000Z"
  }
}
```

**400 — Not found or soft-deleted:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Salt Mango Tree episode not found."] },
  "response": {}
}
```

---

### `PATCH /api/v1/dashboard/media-content/salt-mango-tree/{record_id}/`

Partially update an SMT episode. **Admin only.**

**Request (update zone and link):**
```json
{
  "zone": "central",
  "link": "https://youtube.com/live/smt-ep12-v2"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Salt Mango Tree episode updated."] },
  "response": {
    "id": "d4e5f6a7-b8c9-0123-defa-234567890123",
    "topic": "Building Sustainable Startups",
    "campus": "NIT Calicut",
    "zone": "central",
    "date": "2025-08-20",
    "description": "Student founders share their journey from idea to product.",
    "link": "https://youtube.com/live/smt-ep12-v2",
    "status": "upcoming",
    "created_at": "2025-06-27T09:00:00.000000Z",
    "updated_at": "2025-06-27T10:00:00.000000Z"
  }
}
```

---

### `DELETE /api/v1/dashboard/media-content/salt-mango-tree/{record_id}/`

Soft-delete an SMT episode. **Admin only.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Salt Mango Tree episode deleted."] },
  "response": {}
}
```

---

## 📻 Inspiration Station Radio

**Content type discriminator:** `inspiration_station`
**Date format:** `YYYY-MM-DD`

> **Note:** Identical field structure to Salt Mango Tree. `topic` is used in request and response, mapped to `title` internally.

### Field Reference

| Field | Type | Required | Max Length | Notes |
|-------|------|----------|------------|-------|
| `topic` | string | ✅ | 300 | Episode topic |
| `campus` | string | ✅ | 200 | Campus name |
| `date` | string | ✅ | — | Must be `YYYY-MM-DD` |
| `zone` | enum | ❌ | — | `north` / `central` / `south` |
| `description` | string | ❌ | unlimited | Episode description |
| `link` | URL | ❌ | 500 | Streaming link |

---

### `GET /api/v1/dashboard/media-content/inspiration-station/`

List active Inspiration Station episodes. **Public.**

**Search fields:** `topic`, `campus`, `description`

**Example:**
```
GET /api/v1/dashboard/media-content/inspiration-station/?status=completed&perPage=3
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "f6a7b8c9-d0e1-2345-fabc-456789012345",
        "topic": "From Classroom to Startup",
        "campus": "IIM Kozhikode",
        "zone": "south",
        "date": "2025-04-10",
        "description": "A founder shares how an MBA project became a funded startup.",
        "link": null,
        "status": "completed",
        "created_at": "2025-04-01T05:00:00.000000Z",
        "updated_at": "2025-04-01T05:00:00.000000Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

### `POST /api/v1/dashboard/media-content/inspiration-station/`

Create an Inspiration Station episode. **Admin only.**

**Full request body:**
```json
{
  "topic": "Women in Tech: Breaking Barriers",
  "campus": "Model Engineering College",
  "zone": "central",
  "date": "2025-11-01",
  "description": "Female founders and engineers share their stories and advice.",
  "link": "https://youtube.com/live/is-ep20"
}
```

**Minimal request:**
```json
{
  "topic": "Building in Public",
  "campus": "CUSAT",
  "date": "2025-11-10"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Inspiration Station episode created successfully."] },
  "response": {
    "id": "b8c9d0e1-f2a3-4567-bcde-678901234567",
    "topic": "Women in Tech: Breaking Barriers",
    "campus": "Model Engineering College",
    "zone": "central",
    "date": "2025-11-01",
    "description": "Female founders and engineers share their stories and advice.",
    "link": "https://youtube.com/live/is-ep20",
    "status": "upcoming",
    "created_at": "2025-06-27T10:00:00.000000Z",
    "updated_at": "2025-06-27T10:00:00.000000Z"
  }
}
```

**400 — Invalid zone:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "zone": ["\"east\" is not a valid choice."]
  },
  "response": {}
}
```

---

### `GET /api/v1/dashboard/media-content/inspiration-station/{record_id}/`

Retrieve a single Inspiration Station episode. **Public.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Inspiration Station episode retrieved."] },
  "response": {
    "id": "b8c9d0e1-f2a3-4567-bcde-678901234567",
    "topic": "Women in Tech: Breaking Barriers",
    "campus": "Model Engineering College",
    "zone": "central",
    "date": "2025-11-01",
    "description": "Female founders and engineers share their stories and advice.",
    "link": "https://youtube.com/live/is-ep20",
    "status": "upcoming",
    "created_at": "2025-06-27T10:00:00.000000Z",
    "updated_at": "2025-06-27T10:00:00.000000Z"
  }
}
```

**400 — Not found or soft-deleted:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Inspiration Station episode not found."] },
  "response": {}
}
```

---

### `PATCH /api/v1/dashboard/media-content/inspiration-station/{record_id}/`

Partially update an episode. **Admin only.**

**Request:**
```json
{
  "topic": "Women in Tech: Breaking Barriers (Extended Edition)",
  "date": "2025-11-05"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Inspiration Station episode updated."] },
  "response": {
    "id": "b8c9d0e1-f2a3-4567-bcde-678901234567",
    "topic": "Women in Tech: Breaking Barriers (Extended Edition)",
    "campus": "Model Engineering College",
    "zone": "central",
    "date": "2025-11-05",
    "description": "Female founders and engineers share their stories and advice.",
    "link": "https://youtube.com/live/is-ep20",
    "status": "upcoming",
    "created_at": "2025-06-27T10:00:00.000000Z",
    "updated_at": "2025-06-27T11:00:00.000000Z"
  }
}
```

---

### `DELETE /api/v1/dashboard/media-content/inspiration-station/{record_id}/`

Soft-delete an episode. **Admin only.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Inspiration Station episode deleted."] },
  "response": {}
}
```

---

## 🦸 Grab Your Superpowers

**Content type discriminator:** `grab_your_superpowers`
**Date format:** `YYYY-MM-DD` | **Time format:** `HH:MM` (24h, e.g. `14:30`)

> **Note:** This is the only content type with a `time` field — a session-start time distinct from the `date`. It reuses `performer`/`designation` for speaker details (like Office Hours) and `campus` for the hosting college (like SMT/Inspiration Station), so no schema changes were needed beyond adding `time`.

### Field Reference

| Field | Type | Required | Max Length | Notes |
|-------|------|----------|------------|-------|
| `title` | string | ✅ | 300 | Session name |
| `date` | string | ✅ | — | Must be `YYYY-MM-DD` |
| `time` | string | ❌ | — | Must be `HH:MM` (or `HH:MM:SS`), 24h |
| `description` | string | ❌ | unlimited | Session description |
| `performer` | string | ❌ | 200 | Speaker name |
| `designation` | string | ❌ | 200 | Speaker's role/title, e.g. "Product Manager" |
| `campus` | string | ✅ | 200 | College conducting the session |
| `link` | URL | ❌ | 500 | Meet link |

---

### `GET /api/v1/dashboard/media-content/grab-your-superpowers/`

List active Grab Your Superpowers sessions. **Public.**

**Search fields:** `title`, `performer`, `campus`, `description`

**Example:**
```
GET /api/v1/dashboard/media-content/grab-your-superpowers/?status=upcoming&sortBy=date
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "12ab34cd-56ef-7890-1234-56789abcdef0",
        "title": "Unlock Your Product Sense",
        "date": "2025-09-12",
        "time": "15:00:00",
        "description": "A workshop on developing product intuition through real case studies.",
        "performer": "Anjali Menon",
        "designation": "Product Manager, Zoho",
        "campus": "College of Engineering Trivandrum",
        "link": "https://meet.google.com/gys-session1",
        "status": "upcoming",
        "created_at": "2025-06-27T11:00:00.000000Z",
        "updated_at": "2025-06-27T11:00:00.000000Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

### `POST /api/v1/dashboard/media-content/grab-your-superpowers/`

Create a Grab Your Superpowers session. **Admin only.**

**Full request body:**
```json
{
  "title": "Cracking the Case Interview",
  "date": "2025-10-02",
  "time": "17:30",
  "description": "Learn frameworks for structured problem solving.",
  "performer": "Rahul Krishnan",
  "designation": "Strategy Consultant",
  "campus": "IIM Kozhikode",
  "link": "https://meet.google.com/gys-session2"
}
```

**Minimal request (required fields only):**
```json
{
  "title": "Quick Superpower Talk",
  "date": "2025-10-10",
  "campus": "CUSAT"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Grab Your Superpowers session created successfully."] },
  "response": {
    "id": "23bc45de-67fa-8901-2345-6789abcdef01",
    "title": "Cracking the Case Interview",
    "date": "2025-10-02",
    "time": "17:30:00",
    "description": "Learn frameworks for structured problem solving.",
    "performer": "Rahul Krishnan",
    "designation": "Strategy Consultant",
    "campus": "IIM Kozhikode",
    "link": "https://meet.google.com/gys-session2",
    "status": "upcoming",
    "created_at": "2025-06-27T11:15:00.000000Z",
    "updated_at": "2025-06-27T11:15:00.000000Z"
  }
}
```

**400 — Missing required fields:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "campus": ["This field is required."]
  },
  "response": {}
}
```

**400 — Wrong time format:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "time": ["Time has wrong format. Use one of these formats instead: hh:mm."]
  },
  "response": {}
}
```

**400 — Wrong date format:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid data."],
    "date": ["Date has wrong format. Use one of these formats instead: YYYY-MM-DD."]
  },
  "response": {}
}
```

---

### `GET /api/v1/dashboard/media-content/grab-your-superpowers/{record_id}/`

Retrieve a single session. **Public.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Grab Your Superpowers session retrieved."] },
  "response": {
    "id": "12ab34cd-56ef-7890-1234-56789abcdef0",
    "title": "Unlock Your Product Sense",
    "date": "2025-09-12",
    "time": "15:00:00",
    "description": "A workshop on developing product intuition through real case studies.",
    "performer": "Anjali Menon",
    "designation": "Product Manager, Zoho",
    "campus": "College of Engineering Trivandrum",
    "link": "https://meet.google.com/gys-session1",
    "status": "upcoming",
    "created_at": "2025-06-27T11:00:00.000000Z",
    "updated_at": "2025-06-27T11:00:00.000000Z"
  }
}
```

**400 — Not found or soft-deleted:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Grab Your Superpowers session not found."] },
  "response": {}
}
```

---

### `PATCH /api/v1/dashboard/media-content/grab-your-superpowers/{record_id}/`

Partially update a session. **Admin only.** All fields optional.

**Request (reschedule time and update link):**
```json
{
  "time": "16:00",
  "link": "https://meet.google.com/gys-session1-v2"
}
```

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Grab Your Superpowers session updated."] },
  "response": {
    "id": "12ab34cd-56ef-7890-1234-56789abcdef0",
    "title": "Unlock Your Product Sense",
    "date": "2025-09-12",
    "time": "16:00:00",
    "description": "A workshop on developing product intuition through real case studies.",
    "performer": "Anjali Menon",
    "designation": "Product Manager, Zoho",
    "campus": "College of Engineering Trivandrum",
    "link": "https://meet.google.com/gys-session1-v2",
    "status": "upcoming",
    "created_at": "2025-06-27T11:00:00.000000Z",
    "updated_at": "2025-06-27T12:00:00.000000Z"
  }
}
```

---

### `DELETE /api/v1/dashboard/media-content/grab-your-superpowers/{record_id}/`

Soft-delete a session. **Admin only.**

**200 OK:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Grab Your Superpowers session deleted."] },
  "response": {}
}
```

**400 — Record not found:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Grab Your Superpowers session not found."] },
  "response": {}
}
```

---

## Bulk Import / Export

`grab_your_superpowers` is a supported value for the `content_type` column in CSV bulk import (`POST /media-content/bulk/import/`) and is a valid path segment for bulk export (`GET /media-content/bulk/export/grab_your_superpowers/`). CSV rows use the same field names as the write serializer above (`title`, `date`, `time`, `description`, `performer`, `designation`, `campus`, `link`).

---

## Error Reference

| Scenario | HTTP Status | `hasError` | Sample Message |
|----------|------------|------------|----------------|
| Success | `200` | `false` | Varies |
| Validation failure | `400` | `true` | Field-level errors inside `message` |
| Record not found | `400` | `true` | `"<Type> not found."` |
| Soft-deleted record accessed | `400` | `true` | `"<Type> not found."` |
| Wrong type ID on wrong endpoint | `400` | `true` | `"<Type> not found."` |
| Non-admin write attempt | `400` | `true` | `"You do not have the required role..."` |
| No / invalid JWT | `401` / `403` | `true` | `"Invalid token header"` |

---

## Enum Reference

### Zone (Salt Mango Tree & Inspiration Station)

| API Value | Label |
|-----------|-------|
| `north` | North |
| `central` | Central |
| `south` | South |

### Content Type Discriminator (read-only, set per endpoint)

| Value | Module |
|-------|--------|
| `office_hours` | Office Hours |
| `salt_mango_tree` | Salt Mango Tree |
| `inspiration_station` | Inspiration Station Radio |
| `grab_your_superpowers` | Grab Your Superpowers |

---

