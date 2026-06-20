# Calendar API

**Base path:** `/api/v1/calendar/`  
**Authentication:** By default, these GET endpoints are open to all authenticated users or public (depending on default Django REST Framework configuration). No specific roles are required to query these calendar views.

---

## Table of Contents

| # | Endpoint | Method | Description |
|---|----------|--------|-------------|
| 1 | [`ig-mentor/<ig_id>/sessions/`](#1-ig-mentor-session-calendar) | `GET` | Calendar of mentorship sessions for a specific Interest Group |
| 2 | [`campus-mentor/<campus_id>/sessions/`](#2-campus-mentor-session-calendar) | `GET` | Calendar of mentorship sessions for a specific Campus |
| 3 | [`company/<company_org_id>/sessions/`](#3-company-session-calendar) | `GET` | Calendar of mentorship sessions for a specific Company Organization |
| 4 | [`events/`](#4-global-event-calendar) | `GET` | Global platform events calendar |
| 5 | [`ig/<ig_id>/events/`](#5-ig-event-calendar) | `GET` | Events calendar scoped to a specific Interest Group |
| 6 | [`campus/<campus_id>/events/`](#6-campus-event-calendar) | `GET` | Events calendar scoped to a specific Campus |
| 7 | [`company/<company_id>/events/`](#7-company-event-calendar) | `GET` | Events calendar scoped to a specific Company |

---

## 1. IG Mentor Session Calendar

**`GET /api/v1/calendar/ig-mentor/<ig_id>/sessions/`**

Returns mentorship sessions for a specific Interest Group, grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `ig_id` *(string, required)* — UUID of the Interest Group.

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by session status (`SCHEDULED`, `COMPLETED`, `CANCELLED`).

### Request Body
None

### Response Example (Success - `200` OK)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "upcoming": [
      {
        "id": "a0e28d84-c5a4-4df8-9271-945763914aab",
        "title": "Python Basics Mentorship",
        "description": "Introduction to Python syntax and data types.",
        "mode": "ONLINE",
        "starts_at": "2026-07-20T10:00:00Z",
        "ends_at": "2026-07-20T12:00:00Z",
        "status": "SCHEDULED",
        "meeting_link": "https://meet.google.com/abc-defg-hij",
        "venue": null,
        "mentor_name": "Gokul Shaji",
        "mentee_count": 15
      }
    ],
    "ongoing": [],
    "completed": [
      {
        "id": "e4f8d2b2-65df-4d6b-80a1-9c8842188ab1",
        "title": "Python Setup Workshop",
        "description": "Configuring Python virtual environments.",
        "mode": "ONLINE",
        "starts_at": "2026-06-10T14:00:00Z",
        "ends_at": "2026-06-10T15:30:00Z",
        "status": "COMPLETED",
        "meeting_link": "https://meet.google.com/xyz-pqrs-tuv",
        "venue": null,
        "mentor_name": "Gokul Shaji",
        "mentee_count": 22
      }
    ]
  }
}
```

### Error Responses
- **400 Bad Request** (When Interest Group is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Interest Group not found"
      ]
    },
    "response": {}
  }
  ```

---

## 2. Campus Mentor Session Calendar

**`GET /api/v1/calendar/campus-mentor/<campus_id>/sessions/`**

Returns mentorship sessions for a specific Campus (College Organization), grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `campus_id` *(string, required)* — UUID of the campus (Organization with `org_type="College"`).

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by session status (`SCHEDULED`, `COMPLETED`, `CANCELLED`).

### Request Body
None

### Response Example (Success - `200` OK)
Same data shape as the [IG Mentor Session Calendar](#1-ig-mentor-session-calendar).

### Error Responses
- **400 Bad Request** (When Campus is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Campus not found"
      ]
    },
    "response": {}
  }
  ```

---

## 3. Company Session Calendar

**`GET /api/v1/calendar/company/<company_org_id>/sessions/`**

Returns mentorship sessions for a specific Company Organization, grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `company_org_id` *(string, required)* — UUID of the company organization (Organization with `org_type="Company"`).

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by session status (`SCHEDULED`, `COMPLETED`, `CANCELLED`).

### Request Body
None

### Response Example (Success - `200` OK)
Same data shape as the [IG Mentor Session Calendar](#1-ig-mentor-session-calendar).

### Error Responses
- **400 Bad Request** (When Company organization is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Company org not found"
      ]
    },
    "response": {}
  }
  ```

---

## 4. Global Event Calendar

**`GET /api/v1/calendar/events/`**

Returns global platform events, grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `scope` *(string, optional)* — Filter by scope (`ig`, `campus`, `global`, `company`).
- `status` *(string, optional)* — Filter by status (`ongoing`, `upcoming`, `completed`).

### Request Body
None

### Response Example (Success - `200` OK)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "upcoming": [
      {
        "id": "e7b0c8b0-8e12-4c9c-b26a-93f8e5e31a24",
        "title": "Global Hackathon 2026",
        "slug": "global-hackathon-2026",
        "status": "published",
        "start": "2026-08-01T09:00:00Z",
        "end": "2026-08-03T18:00:00Z",
        "venue_type": "ONLINE",
        "organiser_name": "muLearn",
        "category_name": "Hackathon",
        "is_featured": true
      }
    ],
    "ongoing": [],
    "completed": []
  }
}
```

---

## 5. IG Event Calendar

**`GET /api/v1/calendar/ig/<ig_id>/events/`**

Returns events scoped to a specific Interest Group (either matching the target Interest Group scope or organized by the Interest Group), grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `ig_id` *(string, required)* — UUID of the Interest Group.

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by status (`ongoing`, `upcoming`, `completed`).

### Request Body
None

### Response Example (Success - `200` OK)
Same data shape as the [Global Event Calendar](#4-global-event-calendar).

### Error Responses
- **400 Bad Request** (When Interest Group is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Interest Group not found"
      ]
    },
    "response": {}
  }
  ```

---

## 6. Campus Event Calendar

**`GET /api/v1/calendar/campus/<campus_id>/events/`**

Returns events scoped to a specific Campus (either matching the target Campus scope or organized by the Campus), grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `campus_id` *(string, required)* — UUID of the campus (Organization with `org_type="College"`).

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by status (`ongoing`, `upcoming`, `completed`).

### Request Body
None

### Response Example (Success - `200` OK)
Same data shape as the [Global Event Calendar](#4-global-event-calendar).

### Error Responses
- **400 Bad Request** (When Campus is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Campus not found"
      ]
    },
    "response": {}
  }
  ```

---

## 7. Company Event Calendar

**`GET /api/v1/calendar/company/<company_id>/events/`**

Returns events scoped to a specific Company (either matching the target Company scope or organized by the Company), grouped into `upcoming`, `ongoing`, and `completed` buckets.

### Path Parameters
- `company_id` *(string, required)* — UUID of the company (Organization with `org_type="Company"`).

### Query Parameters
- `month` *(string, optional)* — Filter by month (Format: `YYYY-MM`).
- `status` *(string, optional)* — Filter by status (`ongoing`, `upcoming`, `completed`).

### Request Body
None

### Response Example (Success - `200` OK)
Same data shape as the [Global Event Calendar](#4-global-event-calendar).

### Error Responses
- **400 Bad Request** (When Company is not found):
  ```json
  {
    "hasError": true,
    "statusCode": 400,
    "message": {
      "general": [
        "Company not found"
      ]
    },
    "response": {}
  }
  ```
