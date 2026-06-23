# Student Session Requests API Documentation

This document outlines the API endpoints for the student-initiated mentorship session requests flow. These endpoints allow students to request sessions from mentors mapped to their Campus, Company, or Interest Group (IG), and allow mentors to review, approve, or reject these requests.

---

## 1. Submit a Session Request

Allows any authenticated user (e.g., student, learner) to request a mentorship session from the mentor(s) assigned to their entity.

**Endpoint:**
`POST /api/v1/dashboard/mentor/session/student/request/`

**Required Roles & Permissions:**
- **Role:** Any Authenticated User.
- **Permission:** The user must be a **verified member** of the target entity (`entity_id` + `session_type`).

### Request Payload (JSON)

| Field | Type | Required | Description |
|---|---|---|---|
| `session_type` | String | Yes | Type of session. Must be one of: `campus_session`, `company_session`, `ig_session`. |
| `entity_id` | String (UUID) | Yes | The ID of the Campus, Company, or IG. |
| `title` | String | Yes | The title of the session. |
| `description` | String | No | Brief description of what help is needed. |
| `mode` | String | Yes | `ONLINE`, `OFFLINE`, or `HYBRID`. |
| `starts_at` | Datetime | Yes | Session start time (must be in the future). |
| `ends_at` | Datetime | Yes | Session end time (must be strictly after `starts_at`). |
| `meeting_link` | String | Conditional | Required if mode is `HYBRID`. Must be omitted for `OFFLINE`. |
| `venue` | String | Conditional | Required if mode is `HYBRID`. Must be omitted for `ONLINE`. |
| `max_participants` | Integer | No | Maximum number of allowed participants. |

> [!IMPORTANT]
> **`session_type` valid values:**
> | Value | Meaning |
> |---|---|
> | `campus_session` | Campus / college-scoped session |
> | `company_session` | Company-scoped session |
> | `ig_session` | Interest Group session |

> [!NOTE]
> **Mode / venue / meeting-link rules:**
> - `ONLINE` → `meeting_link` is allowed; `venue` must **not** be provided.
> - `OFFLINE` → `venue` is allowed; `meeting_link` must **not** be provided.
> - `HYBRID` → **both** `venue` and `meeting_link` are **required**.

#### Example Request
```json
{
    "session_type": "campus_session",
    "entity_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "title": "Need help with React Hooks",
    "description": "I am struggling with useEffect and custom hooks.",
    "mode": "ONLINE",
    "starts_at": "2026-06-25T10:00:00Z",
    "ends_at": "2026-06-25T11:00:00Z",
    "meeting_link": "https://meet.google.com/abc-defg-hij",
    "max_participants": 5
}
```

### Responses

**Success (200 OK)**
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": [
            "Session request submitted successfully. A mentor will review your request shortly."
        ]
    },
    "response": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "session_type": "campus_session",
        "entity_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "title": "Need help with React Hooks",
        "description": "I am struggling with useEffect and custom hooks.",
        "mode": "ONLINE",
        "starts_at": "2026-06-25T10:00:00Z",
        "ends_at": "2026-06-25T11:00:00Z",
        "meeting_link": "https://meet.google.com/abc-defg-hij",
        "venue": null,
        "max_participants": 5
    }
}
```

**Failure (400 Bad Request) — Validation Errors**
```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "entity_id": ["You are not a verified member of this campus."],
        "starts_at": ["Session start time must be in the future."]
    },
    "response": {}
}
```

**Failure (400 Bad Request) — Invalid `session_type`**
```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "session_type": ["\"CAMPUS\" is not a valid choice."]
    },
    "response": {}
}
```

---

## 2. List Student's Own Requests

Returns a paginated list of session requests initiated by the logged-in student.

**Endpoint:**
`GET /api/v1/dashboard/mentor/session/student/my-requests/`

**Required Roles & Permissions:**
- **Role:** Any Authenticated User.

### Query Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `status` | String | No | Filter by status: `REQUESTED`, `PENDING_APPROVAL`, `SCHEDULED`, `COMPLETED`, `CANCELLED`, `REJECTED`. |
| `search` | String | No | Search across `title` and `description`. |
| `pageIndex` | Integer | No | Page number for pagination. |
| `perPage` | Integer | No | Number of items per page. |

### Responses

**Success (200 OK)**
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
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "session_type": "campus_session",
                "entity_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "entity_name": "NSS College of Engineering",
                "title": "Need help with React Hooks",
                "description": "I am struggling with useEffect and custom hooks.",
                "mode": "ONLINE",
                "starts_at": "2026-06-25T10:00:00Z",
                "ends_at": "2026-06-25T11:00:00Z",
                "meeting_link": "https://meet.google.com/abc-defg-hij",
                "venue": null,
                "max_participants": 5,
                "status": "REQUESTED",
                "requested_by_id": "user-uuid",
                "requested_by_name": "John Doe",
                "requested_by_muid": "john@mulearn",
                "created_at": "2026-06-22T12:00:00Z"
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

## 3. List Incoming Requests for Mentors

Returns all pending `REQUESTED` sessions that fall within the logged-in mentor's designated scope (their assigned IGs, Campus, or Company). Mentors will only see requests they are authorized to act upon.

**Endpoint:**
`GET /api/v1/dashboard/mentor/session/student-requests/`

**Required Roles & Permissions:**
- **Role:** Must have the `Mentor` role.
- **Scope:** Automatically filtered by mentor tier:

| Mentor Tier | Visible Requests |
|---|---|
| `IG_MENTOR` | `ig_session` requests where `entity_id` matches one of their IGs |
| `CAMPUS_MENTOR` | `campus_session` requests where `entity_id` matches their org |
| `COMPANY_MENTOR` | `company_session` requests where `entity_id` matches their org |
| `MENTOR` (global) | All `REQUESTED` sessions |

### Query Parameters

| Parameter | Type | Required | Description |
|---|---|---|---|
| `search` | String | No | Search by `title` or the requesting student's name. |
| `pageIndex` | Integer | No | Page number for pagination. |
| `perPage` | Integer | No | Number of items per page. |

### Responses

**Success (200 OK)**
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
                "id": "123e4567-e89b-12d3-a456-426614174000",
                "session_type": "campus_session",
                "entity_id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
                "entity_name": "NSS College of Engineering",
                "title": "Need help with React Hooks",
                "description": "I am struggling with useEffect and custom hooks.",
                "mode": "ONLINE",
                "starts_at": "2026-06-25T10:00:00Z",
                "ends_at": "2026-06-25T11:00:00Z",
                "meeting_link": "https://meet.google.com/abc-defg-hij",
                "venue": null,
                "max_participants": 5,
                "status": "REQUESTED",
                "requested_by_id": "user-uuid",
                "requested_by_name": "John Doe",
                "requested_by_muid": "john@mulearn",
                "created_at": "2026-06-22T12:00:00Z"
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

## 4. Verify / Process a Student Request

Allows an eligible mentor to approve or reject a student's session request. If approved, the mentor can optionally override the time, mode, and logistics to suit their availability. Ownership of the session (`created_by`) is transferred to the mentor upon approval, and the session moves to `PENDING_APPROVAL` (awaiting admin scheduling).

**Endpoint:**
`PATCH /api/v1/dashboard/mentor/session/student-requests/<str:session_id>/verify/`

**Required Roles & Permissions:**
- **Role:** Must have the `Mentor` role.
- **Permission:** The mentor's scope *must* cover the session's `entity_id` and `session_type`.

### Request Payload (JSON)

| Field | Type | Required | Description |
|---|---|---|---|
| `status` | String | Yes | Must be `APPROVED` or `REJECTED`. |
| `starts_at` | Datetime | No | (Approval only) Override the proposed start time. Must be in the future. |
| `ends_at` | Datetime | No | (Approval only) Override the proposed end time. |
| `mode` | String | No | (Approval only) Override mode: `ONLINE`, `OFFLINE`, or `HYBRID`. |
| `meeting_link` | String | No | (Approval only) Override the meeting link. |
| `venue` | String | No | (Approval only) Override the physical venue. |

> [!NOTE]
> Override fields are **only applied when `status` is `APPROVED`**. For rejections, only `status` is required — all other fields are ignored.
> If override fields are omitted on approval, the session retains the student's originally proposed values.

#### Example Request (Approval with overrides)
```json
{
    "status": "APPROVED",
    "starts_at": "2026-06-26T14:00:00Z",
    "ends_at": "2026-06-26T15:00:00Z",
    "meeting_link": "https://zoom.us/j/123456"
}
```

#### Example Request (Rejection)
```json
{
    "status": "REJECTED"
}
```

### Responses

**Success (200 OK) — Approved**
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": [
            "Session request approved. It is now pending admin scheduling. You can manage it from your sessions dashboard."
        ]
    },
    "response": {}
}
```

**Success (200 OK) — Rejected**
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": [
            "Session request rejected."
        ]
    },
    "response": {}
}
```

**Failure (404 Not Found) — Session not in REQUESTED state**
```json
{
    "hasError": true,
    "statusCode": 404,
    "message": {
        "general": ["Session request not found or is not in REQUESTED status."]
    },
    "response": {}
}
```

**Failure (403 Forbidden) — Mentor Scope Mismatch**
```json
{
    "hasError": true,
    "statusCode": 403,
    "message": {
        "general": ["You do not have permission to act on this session request."]
    },
    "response": {}
}
```

**Failure (400 Bad Request) — Invalid Overrides**
```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "starts_at": ["Session start time must be in the future."]
    },
    "response": {}
}
```

---

## Session Status Flow

```
REQUESTED → (mentor approves) → PENDING_APPROVAL → (admin schedules) → SCHEDULED
REQUESTED → (mentor rejects) → REJECTED
```
