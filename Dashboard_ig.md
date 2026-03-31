# 📘 Interest Group (IG) — API Documentation

> **Base URL:** `{HOST}/api/v1/dashboard/ig/`
>
> **Authentication:** JWT Bearer Token (via `Authorization` header) — required on all endpoints unless noted otherwise.

---

## Table of Contents

1. [Data Model Reference](#data-model-reference)
2. [Response Envelope](#response-envelope)
3. [Endpoints Overview](#endpoints-overview)
4. [1 — List All Interest Groups](#1--list-all-interest-groups)
5. [2 — Create Interest Group](#2--create-interest-group)
6. [3 — Update Interest Group](#3--update-interest-group)
7. [4 — Delete Interest Group](#4--delete-interest-group)
8. [5 — Get Single Interest Group](#5--get-single-interest-group)
9. [6 — Partial Update Interest Group (Patch)](#6--partial-update-interest-group-patch)
10. [7 — Export Interest Groups as CSV](#7--export-interest-groups-as-csv)
11. [8 — List Interest Groups (Public / Cached)](#8--list-interest-groups-public--cached)
12. [9 — List IG Creation Requests](#9--list-ig-creation-requests)
13. [10 — Submit IG Creation Request](#10--submit-ig-creation-request)
14. [11 — Update IG Request Status](#11--update-ig-request-status)
15. [Enums & Allowed Values](#enums--allowed-values)
16. [Error Codes](#error-codes)

---

## Data Model Reference

The `InterestGroup` model in the database contains the following fields:

| Field                  | Type              | Constraints                                           | Description                                          |
| ---------------------- | ----------------- | ----------------------------------------------------- | ---------------------------------------------------- |
| `id`                   | `string` (UUID)   | Primary key, auto-generated                           | Unique identifier for the IG                         |
| `name`                 | `string` (max 75) | Unique, required                                      | Display name of the interest group                   |
| `code`                 | `string` (max 10) | Unique, required                                      | Short code identifier (e.g., `WEB`, `AI`)            |
| `icon`                 | `string` (max 10) | Required                                              | Emoji or icon code                                   |
| `category`             | `string` (max 20) | One of: `maker`, `coder`, `creative`, `manager`, `others` | IG category classification                       |
| `status`               | `string` (max 20) | One of: `active`, `requested`, `cancelled`, `rejected` | Current lifecycle status                            |
| `about`                | `text`            | Optional                                              | Detailed description of the IG                       |
| `prerequisites`        | `text` (JSON)     | Optional                                              | JSON-encoded list of prerequisites                   |
| `career_opportunities` | `text` (JSON)     | Optional                                              | JSON-encoded list of career paths                    |
| `top_blogs`            | `text` (JSON)     | Optional                                              | JSON-encoded list of recommended blogs               |
| `people_to_follow`     | `text` (JSON)     | Optional                                              | JSON-encoded list of people to follow                |
| `leads`                | `text` (JSON)     | Optional                                              | JSON-encoded list of IG leads                        |
| `mentors`              | `text` (JSON)     | Optional                                              | JSON-encoded list of mentors                         |
| `resource`             | `text`            | Optional                                              | Learning resources / links                           |
| `thinktank`            | `text`            | Optional                                              | Think-tank notes                                     |
| `office_hours`         | `string` (max 200)| Optional                                              | Office hours information                             |
| `created_by`           | `string` (UUID)   | FK → User                                             | User who created this IG                             |
| `created_at`           | `datetime`        | Auto-set on creation                                  | Creation timestamp                                   |
| `updated_by`           | `string` (UUID)   | FK → User                                             | User who last updated this IG                        |
| `updated_at`           | `datetime`        | Auto-set on update                                    | Last update timestamp                                |

> [!NOTE]
> Fields marked as **(JSON)** are stored as JSON-serialized strings in the database. The API automatically deserializes them to arrays/objects in responses and accepts either JSON arrays/objects or pre-serialized strings in requests.

---

## Response Envelope

All API responses use a standard envelope format:

### Success Response

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    // ... endpoint-specific data
  }
}
```

### Failure Response

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Error description here"]
  },
  "response": {}
}
```

### Paginated Response

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [ /* array of items */ ],
    "pagination": {
      "count": 50,
      "next": "http://host/api/v1/dashboard/ig/?page=2",
      "previous": null,
      "isNext": true,
      "isPrev": false,
      "currentPage": 1
    }
  }
}
```

---

## Endpoints Overview

| #  | Method   | Endpoint                              | Auth  | Role Required       | Description                               |
| -- | -------- | ------------------------------------- | ----- | ------------------- | ----------------------------------------- |
| 1  | `GET`    | `/api/v1/dashboard/ig/`               | ✅ JWT | Any authenticated   | List all IGs (paginated, searchable)      |
| 2  | `POST`   | `/api/v1/dashboard/ig/`               | ✅ JWT | Admin               | Create a new interest group               |
| 3  | `PUT`    | `/api/v1/dashboard/ig/{id}/`          | ✅ JWT | Admin               | Full update of an interest group          |
| 4  | `DELETE` | `/api/v1/dashboard/ig/{id}/`          | ✅ JWT | Admin               | Delete an interest group                  |
| 5  | `GET`    | `/api/v1/dashboard/ig/get/{id}/`      | ✅ JWT | Admin               | Get a single IG by ID                     |
| 6  | `PATCH`  | `/api/v1/dashboard/ig/get/{id}/`      | ✅ JWT | Admin or IG Lead    | Partial update of IG fields               |
| 7  | `GET`    | `/api/v1/dashboard/ig/csv/`           | ✅ JWT | Admin               | Download all IGs as CSV                   |
| 8  | `GET`    | `/api/v1/dashboard/ig/list/`          | ❌ None| Public (no auth)    | List all IGs (cached 10 min)              |
| 9  | `GET`    | `/api/v1/dashboard/ig/request/`       | ✅ JWT | Admin or Company    | List IG creation requests                 |
| 10 | `POST`   | `/api/v1/dashboard/ig/request/`       | ✅ JWT | Admin or Company    | Submit IG creation request                |
| 11 | `PATCH`  | `/api/v1/dashboard/ig/request/{id}/`  | ✅ JWT | Admin               | Update IG request status                  |

---

## 1 — List All Interest Groups

Retrieves a paginated list of all interest groups with member counts.

| Property     | Value                            |
| ------------ | -------------------------------- |
| **Endpoint** | `GET /api/v1/dashboard/ig/`      |
| **Auth**     | JWT Bearer Token                 |
| **Role**     | Any authenticated user           |

### Query Parameters

| Parameter   | Type     | Required | Description                                                                                    |
| ----------- | -------- | -------- | ---------------------------------------------------------------------------------------------- |
| `page`      | `int`    | No       | Page number for pagination (default: 1)                                                        |
| `perPage`   | `int`    | No       | Number of results per page                                                                     |
| `search`    | `string` | No       | Search by `name`, `created_by.full_name`, or `updated_by.full_name`                            |
| `sortBy`    | `string` | No       | Sort field. Allowed: `name`, `members`, `status`, `updated_on`, `updated_by`, `created_on`, `created_by` |
| `orderBy`   | `string` | No       | Sort direction: `asc` or `desc`                                                                |

### Example Request

```http
GET /api/v1/dashboard/ig/?page=1&perPage=10&search=web&sortBy=name&orderBy=asc
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Example Response — `200 OK`

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
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "name": "Web Development",
        "resource": "https://resources.mulearn.org/web-dev",
        "about": "A community focused on modern web technologies.",
        "prerequisites": ["HTML", "CSS", "JavaScript basics"],
        "career_opportunities": ["Frontend Developer", "Full Stack Developer", "UI Engineer"],
        "top_blogs": ["https://css-tricks.com", "https://dev.to"],
        "people_to_follow": ["@dan_abramov", "@sarah_edo"],
        "leads": ["John Doe", "Jane Smith"],
        "mentors": ["Prof. Alan", "Dr. Grace"],
        "thinktank": "Weekly brainstorming sessions on Fridays",
        "office_hours": "Mon-Fri 10:00 AM - 12:00 PM",
        "icon": "🌐",
        "code": "WEB",
        "category": "coder",
        "status": "active",
        "members": 156,
        "updated_by": "Admin User",
        "updated_at": "2026-03-15T10:30:00Z",
        "created_by": "Admin User",
        "created_at": "2025-06-01T09:00:00Z"
      },
      {
        "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "name": "AI/ML",
        "resource": "https://resources.mulearn.org/ai-ml",
        "about": "Interest group for Artificial Intelligence and Machine Learning enthusiasts.",
        "prerequisites": ["Python", "Linear Algebra", "Statistics"],
        "career_opportunities": ["ML Engineer", "Data Scientist", "AI Researcher"],
        "top_blogs": ["https://towardsdatascience.com"],
        "people_to_follow": ["@AndrewYNg"],
        "leads": ["Alice Johnson"],
        "mentors": ["Dr. Hinton"],
        "thinktank": null,
        "office_hours": "Tue-Thu 2:00 PM - 4:00 PM",
        "icon": "🤖",
        "code": "AIML",
        "category": "coder",
        "status": "active",
        "members": 230,
        "updated_by": "Admin User",
        "updated_at": "2026-03-20T14:00:00Z",
        "created_by": "Admin User",
        "created_at": "2025-07-15T11:00:00Z"
      }
    ],
    "pagination": {
      "count": 25,
      "next": "http://localhost:8000/api/v1/dashboard/ig/?page=2&perPage=10",
      "previous": null,
      "isNext": true,
      "isPrev": false,
      "currentPage": 1
    }
  }
}
```

---

## 2 — Create Interest Group

Creates a new interest group and automatically creates associated roles (Member, Campus Lead, IG Lead).

| Property     | Value                            |
| ------------ | -------------------------------- |
| **Endpoint** | `POST /api/v1/dashboard/ig/`     |
| **Auth**     | JWT Bearer Token                 |
| **Role**     | `Admin` only                     |

### Request Body

| Field                  | Type              | Required | Description                                          |
| ---------------------- | ----------------- | -------- | ---------------------------------------------------- |
| `name`                 | `string`          | ✅ Yes   | Display name (max 75 chars, must be unique)          |
| `code`                 | `string`          | ✅ Yes   | Short code (max 10 chars, must be unique)            |
| `icon`                 | `string`          | ✅ Yes   | Emoji or icon code (max 10 chars)                    |
| `category`             | `string`          | ✅ Yes   | One of: `maker`, `coder`, `creative`, `manager`, `others` |
| `status`               | `string`          | No       | One of: `active`, `requested`, `cancelled`, `rejected` (default: `requested`) |
| `about`                | `string`          | No       | Description text                                     |
| `prerequisites`        | `array` or `string` | No     | List of prerequisites (array preferred)              |
| `career_opportunities` | `array` or `string` | No     | List of career paths                                 |
| `resource`             | `string`          | No       | Resource URL or description                          |
| `top_blogs`            | `array` or `string` | No     | List of blog URLs                                    |
| `people_to_follow`     | `array` or `string` | No     | List of people to follow                             |
| `leads`                | `array` or `string` | No     | List of IG leads                                     |
| `mentors`              | `array` or `string` | No     | List of mentors                                      |
| `thinktank`            | `string`          | No       | Think-tank description                               |
| `office_hours`         | `string`          | No       | Office hours info (max 200 chars)                    |

### Example Request

```http
POST /api/v1/dashboard/ig/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

```json
{
  "name": "Cybersecurity",
  "code": "CYSEC",
  "icon": "🔒",
  "category": "coder",
  "status": "active",
  "about": "Interest group focused on cybersecurity, ethical hacking, and digital forensics.",
  "prerequisites": [
    "Networking fundamentals",
    "Linux basics",
    "Programming in Python or C"
  ],
  "career_opportunities": [
    "Security Analyst",
    "Penetration Tester",
    "SOC Analyst",
    "Security Architect"
  ],
  "resource": "https://resources.mulearn.org/cybersecurity",
  "top_blogs": [
    "https://krebsonsecurity.com",
    "https://thehackernews.com"
  ],
  "people_to_follow": [
    "@troaborr",
    "@SwiftOnSecurity"
  ],
  "leads": [
    "Rahul Menon",
    "Priya Nair"
  ],
  "mentors": [
    "Prof. Suresh Kumar"
  ],
  "thinktank": "Monthly CTF practice sessions",
  "office_hours": "Wed & Fri 3:00 PM - 5:00 PM"
}
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "interestGroup": {
      "name": "Cybersecurity",
      "code": "CYSEC",
      "category": "coder",
      "status": "active",
      "icon": "🔒",
      "about": "Interest group focused on cybersecurity, ethical hacking, and digital forensics.",
      "prerequisites": "[\"Networking fundamentals\", \"Linux basics\", \"Programming in Python or C\"]",
      "career_opportunities": "[\"Security Analyst\", \"Penetration Tester\", \"SOC Analyst\", \"Security Architect\"]",
      "resource": "https://resources.mulearn.org/cybersecurity",
      "top_blogs": "[\"https://krebsonsecurity.com\", \"https://thehackernews.com\"]",
      "people_to_follow": "[\"@troaborr\", \"@SwiftOnSecurity\"]",
      "leads": "[\"Rahul Menon\", \"Priya Nair\"]",
      "mentors": "[\"Prof. Suresh Kumar\"]",
      "thinktank": "Monthly CTF practice sessions",
      "office_hours": "Wed & Fri 3:00 PM - 5:00 PM",
      "created_by": "d290f1ee-6c54-4b01-90e6-d701748f0851",
      "updated_by": "d290f1ee-6c54-4b01-90e6-d701748f0851"
    }
  }
}
```

### Example Error Response — `400 Bad Request`

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": {
      "name": ["interest group with this name already exists."],
      "code": ["interest group with this code already exists."]
    }
  },
  "response": {}
}
```

> [!IMPORTANT]
> Creating an IG also auto-creates **three associated Role records**:
> - `{name}` — IG Member role
> - `{code} CampusLead` — IG Campus Lead role
> - `{code} IGLead` — IG Lead role
>
> A Discord webhook notification is also sent upon successful creation.

---

## 3 — Update Interest Group

Performs a full update on an existing interest group. Also updates the associated roles if the name or code changes.

| Property     | Value                                 |
| ------------ | ------------------------------------- |
| **Endpoint** | `PUT /api/v1/dashboard/ig/{id}/`      |
| **Auth**     | JWT Bearer Token                      |
| **Role**     | `Admin` only                          |

### Path Parameters

| Parameter | Type     | Required | Description              |
| --------- | -------- | -------- | ------------------------ |
| `id`      | `string` | ✅ Yes   | UUID of the interest group |

### Request Body

Same fields as [Create Interest Group](#2--create-interest-group). This is a **partial update** (`partial=True`), so only changed fields need to be sent.

### Example Request

```http
PUT /api/v1/dashboard/ig/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

```json
{
  "name": "Web Development & Design",
  "about": "Updated description covering both web development and UI/UX design.",
  "category": "creative",
  "prerequisites": [
    "HTML & CSS",
    "JavaScript",
    "Figma basics"
  ],
  "office_hours": "Mon-Fri 9:00 AM - 1:00 PM"
}
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "interestGroup": {
      "name": "Web Development & Design",
      "code": "WEB",
      "category": "creative",
      "status": "active",
      "icon": "🌐",
      "about": "Updated description covering both web development and UI/UX design.",
      "prerequisites": "[\"HTML & CSS\", \"JavaScript\", \"Figma basics\"]",
      "career_opportunities": "[\"Frontend Developer\", \"Full Stack Developer\", \"UI Engineer\"]",
      "resource": "https://resources.mulearn.org/web-dev",
      "top_blogs": "[\"https://css-tricks.com\", \"https://dev.to\"]",
      "people_to_follow": "[\"@dan_abramov\", \"@sarah_edo\"]",
      "leads": "[\"John Doe\", \"Jane Smith\"]",
      "mentors": "[\"Prof. Alan\", \"Dr. Grace\"]",
      "thinktank": "Weekly brainstorming sessions on Fridays",
      "office_hours": "Mon-Fri 9:00 AM - 1:00 PM",
      "created_by": "d290f1ee-6c54-4b01-90e6-d701748f0851",
      "updated_by": "d290f1ee-6c54-4b01-90e6-d701748f0851"
    }
  }
}
```

> [!NOTE]
> If the `name` or `code` changes, the corresponding Role records (Member, Campus Lead, IG Lead) are automatically renamed. A Discord webhook is also triggered.

---

## 4 — Delete Interest Group

Deletes an interest group and all associated roles (Member, Campus Lead, IG Lead).

| Property     | Value                                    |
| ------------ | ---------------------------------------- |
| **Endpoint** | `DELETE /api/v1/dashboard/ig/{id}/`      |
| **Auth**     | JWT Bearer Token                         |
| **Role**     | `Admin` only                             |

### Path Parameters

| Parameter | Type     | Required | Description              |
| --------- | -------- | -------- | ------------------------ |
| `id`      | `string` | ✅ Yes   | UUID of the interest group |

### Example Request

```http
DELETE /api/v1/dashboard/ig/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["ig deleted successfully"]
  },
  "response": {}
}
```

> [!WARNING]
> This action is **irreversible**. It permanently deletes the IG record and all three associated Role records. A Discord webhook notification is sent upon deletion.

---

## 5 — Get Single Interest Group

Retrieves the details of a single interest group by its ID.

| Property     | Value                                     |
| ------------ | ----------------------------------------- |
| **Endpoint** | `GET /api/v1/dashboard/ig/get/{id}/`      |
| **Auth**     | JWT Bearer Token                          |
| **Role**     | `Admin` only                              |

### Path Parameters

| Parameter | Type     | Required | Description              |
| --------- | -------- | -------- | ------------------------ |
| `id`      | `string` | ✅ Yes   | UUID of the interest group |

### Example Request

```http
GET /api/v1/dashboard/ig/get/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "interestGroup": {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "name": "Web Development",
      "resource": "https://resources.mulearn.org/web-dev",
      "about": "A community focused on modern web technologies.",
      "prerequisites": ["HTML", "CSS", "JavaScript basics"],
      "career_opportunities": ["Frontend Developer", "Full Stack Developer", "UI Engineer"],
      "top_blogs": ["https://css-tricks.com", "https://dev.to"],
      "people_to_follow": ["@dan_abramov", "@sarah_edo"],
      "leads": ["John Doe", "Jane Smith"],
      "mentors": ["Prof. Alan", "Dr. Grace"],
      "thinktank": "Weekly brainstorming sessions on Fridays",
      "office_hours": "Mon-Fri 10:00 AM - 12:00 PM",
      "icon": "🌐",
      "code": "WEB",
      "category": "coder",
      "status": "active",
      "members": 156,
      "updated_by": "Admin User",
      "updated_at": "2026-03-15T10:30:00Z",
      "created_by": "Admin User",
      "created_at": "2025-06-01T09:00:00Z"
    }
  }
}
```

### Example Error Response — `400 Bad Request`

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Interest Group Does Not Exist"]
  },
  "response": {}
}
```

---

## 6 — Partial Update Interest Group (Patch)

Allows an Admin or the IG Lead to partially update editable fields of an interest group.

| Property     | Value                                      |
| ------------ | ------------------------------------------ |
| **Endpoint** | `PATCH /api/v1/dashboard/ig/get/{id}/`     |
| **Auth**     | JWT Bearer Token                           |
| **Role**     | `Admin` or `{code} IGLead`                 |

### Path Parameters

| Parameter | Type     | Required | Description              |
| --------- | -------- | -------- | ------------------------ |
| `id`      | `string` | ✅ Yes   | UUID of the interest group |

### Request Body

All fields are optional — send only the fields you want to update.

| Field                  | Type              | Description                              |
| ---------------------- | ----------------- | ---------------------------------------- |
| `name`                 | `string`          | Display name                             |
| `code`                 | `string`          | Short code                               |
| `icon`                 | `string`          | Icon/emoji                               |
| `category`             | `string`          | Category classification                  |
| `status`               | `string`          | Status value                             |
| `about`                | `string`          | Description                              |
| `prerequisites`        | `array` or `string` | Prerequisites list                     |
| `career_opportunities` | `array` or `string` | Career paths list                      |
| `resource`             | `string`          | Resource link                            |
| `top_blogs`            | `array` or `string` | Blog URLs list                         |
| `people_to_follow`     | `array` or `string` | People to follow list                  |
| `leads`                | `array` or `string` | Leads list                             |
| `mentors`              | `array` or `string` | Mentors list                           |
| `thinktank`            | `string`          | Think-tank info                          |
| `office_hours`         | `string`          | Office hours                             |

### Example Request

```http
PATCH /api/v1/dashboard/ig/get/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

```json
{
  "about": "Updated IG description with new focus areas.",
  "office_hours": "Mon-Sat 10:00 AM - 2:00 PM",
  "mentors": ["Dr. Grace Hopper", "Prof. Dijkstra"]
}
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "interestGroup": {
      "name": "Web Development",
      "code": "WEB",
      "category": "coder",
      "status": "active",
      "icon": "🌐",
      "about": "Updated IG description with new focus areas.",
      "prerequisites": "[\"HTML\", \"CSS\", \"JavaScript basics\"]",
      "career_opportunities": "[\"Frontend Developer\", \"Full Stack Developer\"]",
      "resource": "https://resources.mulearn.org/web-dev",
      "top_blogs": "[\"https://css-tricks.com\"]",
      "people_to_follow": "[\"@dan_abramov\"]",
      "leads": "[\"John Doe\"]",
      "mentors": "[\"Dr. Grace Hopper\", \"Prof. Dijkstra\"]",
      "thinktank": "Weekly brainstorming sessions on Fridays",
      "office_hours": "Mon-Sat 10:00 AM - 2:00 PM",
      "created_by": "d290f1ee-6c54-4b01-90e6-d701748f0851",
      "updated_by": "d290f1ee-6c54-4b01-90e6-d701748f0851"
    }
  }
}
```

### Example Error Response — `400 Bad Request` (Permission Denied)

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["You do not have permission to update this Interest Group"]
  },
  "response": {}
}
```

---

## 7 — Export Interest Groups as CSV

Downloads all interest groups in CSV format.

| Property     | Value                                |
| ------------ | ------------------------------------ |
| **Endpoint** | `GET /api/v1/dashboard/ig/csv/`      |
| **Auth**     | JWT Bearer Token                     |
| **Role**     | `Admin` only                         |

### Example Request

```http
GET /api/v1/dashboard/ig/csv/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Response

Returns a `text/csv` file download containing all IG records with the following columns:

```
id, name, resource, about, prerequisites, career_opportunities, top_blogs, people_to_follow, leads, mentors, thinktank, office_hours, icon, code, category, status, members, updated_by, updated_at, created_by, created_at
```

---

## 8 — List Interest Groups (Public / Cached)

Public endpoint that lists all interest groups with member count. **No authentication required.** Response is cached for 10 minutes.

| Property     | Value                                 |
| ------------ | ------------------------------------- |
| **Endpoint** | `GET /api/v1/dashboard/ig/list/`      |
| **Auth**     | None (public)                         |
| **Role**     | None                                  |
| **Cache**    | 10 minutes                            |

### Example Request

```http
GET /api/v1/dashboard/ig/list/
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "interestGroup": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "name": "Web Development",
        "resource": "https://resources.mulearn.org/web-dev",
        "about": "A community focused on modern web technologies.",
        "prerequisites": ["HTML", "CSS", "JavaScript basics"],
        "career_opportunities": ["Frontend Developer", "Full Stack Developer"],
        "top_blogs": ["https://css-tricks.com", "https://dev.to"],
        "people_to_follow": ["@dan_abramov"],
        "leads": ["John Doe"],
        "mentors": ["Prof. Alan"],
        "thinktank": "Weekly brainstorming sessions",
        "office_hours": "Mon-Fri 10:00 AM - 12:00 PM",
        "icon": "🌐",
        "code": "WEB",
        "category": "coder",
        "status": "active",
        "members": 156,
        "updated_by": "Admin User",
        "updated_at": "2026-03-15T10:30:00Z",
        "created_by": "Admin User",
        "created_at": "2025-06-01T09:00:00Z"
      }
    ]
  }
}
```

> [!TIP]
> Use this endpoint for public-facing IG listing pages. It does not require authentication and has built-in caching for performance.

---

## 9 — List IG Creation Requests

Retrieves paginated list of IG creation requests. Company users see only their own; admins can see all or filter by user.

| Property     | Value                                    |
| ------------ | ---------------------------------------- |
| **Endpoint** | `GET /api/v1/dashboard/ig/request/`      |
| **Auth**     | JWT Bearer Token                         |
| **Role**     | `Admin` or `Company`                     |

### Query Parameters

| Parameter   | Type     | Required | Description                                                                 |
| ----------- | -------- | -------- | --------------------------------------------------------------------------- |
| `user_id`   | `string` | No       | Filter by a specific user's requests (admin-only for other users' requests) |
| `status`    | `string` | No       | Filter by status: `active`, `requested`, `cancelled`, `rejected`            |
| `page`      | `int`    | No       | Page number for pagination                                                  |
| `perPage`   | `int`    | No       | Number of results per page                                                  |
| `search`    | `string` | No       | Search by `name`, `code`, or `category`                                     |
| `sortBy`    | `string` | No       | Sort field. Allowed: `name`, `status`, `ig_name`, `user_full_name`, `created_at`, `created_on`, `updated_on` |
| `orderBy`   | `string` | No       | Sort direction: `asc` or `desc`                                             |

### Example Request

```http
GET /api/v1/dashboard/ig/request/?status=requested&page=1&perPage=5
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
```

### Example Response — `200 OK`

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
        "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
        "name": "Blockchain",
        "resource": null,
        "about": "Exploring decentralized technologies and smart contracts.",
        "prerequisites": ["Cryptography basics", "Programming fundamentals"],
        "career_opportunities": ["Blockchain Developer", "Smart Contract Auditor"],
        "top_blogs": [],
        "people_to_follow": [],
        "leads": [],
        "mentors": [],
        "thinktank": null,
        "office_hours": null,
        "icon": "⛓️",
        "code": "BLKCH",
        "category": "coder",
        "status": "requested",
        "members": 0,
        "updated_by": "Company User",
        "updated_at": "2026-03-28T16:00:00Z",
        "created_by": "Company User",
        "created_at": "2026-03-28T16:00:00Z"
      }
    ],
    "pagination": {
      "count": 3,
      "next": null,
      "previous": null,
      "isNext": false,
      "isPrev": false,
      "currentPage": 1
    }
  }
}
```

### Example Error Response — `400 Bad Request`

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["You can only view your own IG requests"]
  },
  "response": {}
}
```

---

## 10 — Submit IG Creation Request

Allows a Company user or Admin to submit a new interest group creation request. The IG is created with `status: "requested"` and requires admin approval to become active.

| Property     | Value                                    |
| ------------ | ---------------------------------------- |
| **Endpoint** | `POST /api/v1/dashboard/ig/request/`     |
| **Auth**     | JWT Bearer Token                         |
| **Role**     | `Admin` or `Company`                     |

### Request Body

| Field                  | Type              | Required | Description                              |
| ---------------------- | ----------------- | -------- | ---------------------------------------- |
| `name`                 | `string`          | ✅ Yes   | Display name (max 75, unique)            |
| `code`                 | `string`          | ✅ Yes   | Short code (max 10, unique)              |
| `icon`                 | `string`          | ✅ Yes   | Emoji or icon code                       |
| `category`             | `string`          | ✅ Yes   | One of: `maker`, `coder`, `creative`, `manager`, `others` |
| `about`                | `string`          | No       | Description                              |
| `prerequisites`        | `array` or `string` | No     | Prerequisites list                       |
| `career_opportunities` | `array` or `string` | No     | Career paths list                        |
| `resource`             | `string`          | No       | Resource link                            |
| `top_blogs`            | `array` or `string` | No     | Blog URLs                                |
| `people_to_follow`     | `array` or `string` | No     | People to follow                         |
| `leads`                | `array` or `string` | No     | Leads list                               |
| `mentors`              | `array` or `string` | No     | Mentors list                             |
| `thinktank`            | `string`          | No       | Think-tank info                          |
| `office_hours`         | `string`          | No       | Office hours                             |

> [!NOTE]
> The `status` field is automatically set to `"requested"` — callers cannot override it. The `created_by` and `updated_by` are auto-populated from the JWT.

### Example Request

```http
POST /api/v1/dashboard/ig/request/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

```json
{
  "name": "Blockchain",
  "code": "BLKCH",
  "icon": "⛓️",
  "category": "coder",
  "about": "Exploring decentralized technologies and smart contracts.",
  "prerequisites": [
    "Cryptography basics",
    "Programming fundamentals"
  ],
  "career_opportunities": [
    "Blockchain Developer",
    "Smart Contract Auditor"
  ],
  "resource": "https://ethereum.org/developers",
  "top_blogs": [
    "https://blog.chain.link"
  ],
  "people_to_follow": [
    "@VitalikButerin"
  ],
  "leads": [],
  "mentors": [],
  "thinktank": "Bi-weekly smart contract review sessions",
  "office_hours": "Thursday 4:00 PM - 6:00 PM"
}
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Interest Group request submitted successfully. It will be reviewed by admins."]
  },
  "response": {
    "interestGroup": {
      "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
      "name": "Blockchain",
      "resource": "https://ethereum.org/developers",
      "about": "Exploring decentralized technologies and smart contracts.",
      "prerequisites": ["Cryptography basics", "Programming fundamentals"],
      "career_opportunities": ["Blockchain Developer", "Smart Contract Auditor"],
      "top_blogs": ["https://blog.chain.link"],
      "people_to_follow": ["@VitalikButerin"],
      "leads": [],
      "mentors": [],
      "thinktank": "Bi-weekly smart contract review sessions",
      "office_hours": "Thursday 4:00 PM - 6:00 PM",
      "icon": "⛓️",
      "code": "BLKCH",
      "category": "coder",
      "status": "requested",
      "members": 0,
      "updated_by": "Company User",
      "updated_at": "2026-03-31T12:00:00Z",
      "created_by": "Company User",
      "created_at": "2026-03-31T12:00:00Z"
    }
  }
}
```

---

## 11 — Update IG Request Status

Allows an Admin to approve, reject, or cancel an IG creation request by updating its status.

| Property     | Value                                            |
| ------------ | ------------------------------------------------ |
| **Endpoint** | `PATCH /api/v1/dashboard/ig/request/{id}/`       |
| **Auth**     | JWT Bearer Token                                 |
| **Role**     | `Admin` only                                     |

### Path Parameters

| Parameter | Type     | Required | Description              |
| --------- | -------- | -------- | ------------------------ |
| `id`      | `string` | ✅ Yes   | UUID of the interest group |

### Request Body

| Field    | Type     | Required | Description                                                    |
| -------- | -------- | -------- | -------------------------------------------------------------- |
| `status` | `string` | ✅ Yes   | New status. Must be one of: `active`, `requested`, `cancelled`, `rejected` |

### Example Request

```http
PATCH /api/v1/dashboard/ig/request/c3d4e5f6-a7b8-9012-cdef-123456789012/
Authorization: Bearer eyJhbGciOiJIUzI1NiIs...
Content-Type: application/json
```

```json
{
  "status": "active"
}
```

### Example Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Interest Group status updated to 'active'"]
  },
  "response": {
    "interestGroup": {
      "id": "c3d4e5f6-a7b8-9012-cdef-123456789012",
      "name": "Blockchain",
      "resource": "https://ethereum.org/developers",
      "about": "Exploring decentralized technologies and smart contracts.",
      "prerequisites": ["Cryptography basics", "Programming fundamentals"],
      "career_opportunities": ["Blockchain Developer", "Smart Contract Auditor"],
      "top_blogs": ["https://blog.chain.link"],
      "people_to_follow": ["@VitalikButerin"],
      "leads": [],
      "mentors": [],
      "thinktank": "Bi-weekly smart contract review sessions",
      "office_hours": "Thursday 4:00 PM - 6:00 PM",
      "icon": "⛓️",
      "code": "BLKCH",
      "category": "coder",
      "status": "active",
      "members": 0,
      "updated_by": "Admin User",
      "updated_at": "2026-03-31T14:00:00Z",
      "created_by": "Company User",
      "created_at": "2026-03-31T12:00:00Z"
    }
  }
}
```

### Example Error Responses

**IG Not Found — `400 Bad Request`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Interest Group not found"]
  },
  "response": {}
}
```

**Missing Status — `400 Bad Request`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Status field is required"]
  },
  "response": {}
}
```

**Invalid Status — `400 Bad Request`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid status. Must be one of: active, requested, cancelled, rejected"]
  },
  "response": {}
}
```

---

## Enums & Allowed Values

### Category

| Value      | Description                          |
| ---------- | ------------------------------------ |
| `maker`    | Hardware / IoT / Maker projects      |
| `coder`    | Software development & programming   |
| `creative` | Design, content creation, media      |
| `manager`  | Project management, leadership       |
| `others`   | Miscellaneous interest groups        |

### Status

| Value       | Description                                    |
| ----------- | ---------------------------------------------- |
| `active`    | Approved and currently active IG               |
| `requested` | Pending admin review (default for new requests)|
| `cancelled` | Cancelled by admin or requester                |
| `rejected`  | Rejected by admin                              |

### Roles

| Role                   | Description                         |
| ---------------------- | ----------------------------------- |
| `Admin`                | Full CRUD access on all endpoints   |
| `Company`              | Can submit & view own IG requests   |
| `{code} IGLead`        | Can partially update their own IG   |
| `{code} CampusLead`    | Campus lead role (created with IG)  |

---

## Error Codes

| HTTP Status | `statusCode` | Typical Cause                                          |
| ----------- | ------------ | ------------------------------------------------------ |
| `200`       | `200`        | Success                                                |
| `400`       | `400`        | Validation error, missing fields, invalid status, etc. |
| `403`       | `403`        | Insufficient permissions / unauthorized                |
| `404`       | —            | IG not found (returned as 400 with message)            |

> [!CAUTION]
> All mutating endpoints (`POST`, `PUT`, `PATCH`, `DELETE`) require the JWT `Authorization` header. Requests without it will receive a `403 Forbidden` response.

---

## Source Files Reference

| File | Purpose |
| ---- | ------- |
| [urls.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/ig/urls.py) | URL route definitions |
| [dash_ig_view.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/ig/dash_ig_view.py) | View classes (business logic) |
| [dash_ig_serializer.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/api/dashboard/ig/dash_ig_serializer.py) | Serializer definitions |
| [task.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/db/task.py) | InterestGroup model definition |
| [response.py](file:///c:/Users/prana/Desktop/internship/mulearnbackend/utils/response.py) | CustomResponse envelope |
