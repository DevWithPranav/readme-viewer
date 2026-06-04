# Intern Dashboard API Documentation

> **Base URL:** `https://<host>/api/v1/dashboard/`  
> **Authentication:** All endpoints require a valid JWT Bearer token  
> **Content-Type:** `application/json` (unless noted otherwise)

---

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Role Permissions](#role-permissions)
- [Intern APIs (`/intern/`)](#intern-apis)
  - [Overview](#1-overview)
  - [Timesheets](#2-timesheets)
  - [Weekly Reviews](#3-weekly-reviews)
  - [Leaderboard](#4-leaderboard)
  - [Tasks (Mine)](#5-tasks-mine)
  - [Leave](#6-leave)
  - [Guilds](#7-guilds)
- [Manage Interns APIs (`/manage-interns/`)](#manage-interns-apis)
  - [Interns Management](#1-interns-management)
  - [Status Stats](#2-status-stats)
  - [Export Interns](#3-export-interns)
  - [Reviews & Timesheets (Admin)](#4-reviews--timesheets-admin)
  - [Tasks (Admin)](#5-tasks-admin)
  - [Leave (Admin)](#6-leave-admin)
- [Common Query Parameters](#common-query-parameters)
- [Common Response Format](#common-response-format)

---

## Overview

The Intern Dashboard is split into two groups:

| Group | Base Path | Audience |
|-------|-----------|----------|
| **Intern** | `/api/v1/dashboard/intern/` | Interns (role: `Intern`) |
| **Manage Interns** | `/api/v1/dashboard/manage-interns/` | Admins (role: `Admin`) |

---

## Authentication

All requests must include a JWT token in the `Authorization` header:

```
Authorization: Bearer <your-jwt-token>
```

---

## Role Permissions

| Role | Access |
|------|--------|
| `Intern` | Can access all `/intern/` endpoints and view leaderboard |
| `Admin` | Can access all `/manage-interns/` endpoints and view leaderboard |

---

## Intern APIs

### 1. Overview

#### `GET /intern/overview/status/`

Returns the authenticated intern's current status, karma, streaks, completed tasks, and leaderboard score.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/overview/status/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "guild": "CATALYST",
    "status": "ACTIVE",
    "total_intern_karma": 320,
    "daily_streak": 5,
    "weekly_streak": 3,
    "completed_tasks": 8,
    "complexity_score": 18,
    "score": 472.5
  }
}
```

---

#### `GET /intern/overview/activity/`

Returns a paginated list of the intern's karma activity logs (intern-specific tasks only).

**Role:** `Intern`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number (default: 1) |
| `page_size` | int | Results per page (default: 10) |
| `search` | string | Filter by task title |
| `sortBy` | string | Sort field (e.g., `created_at`) |

**Sample Request:**
```http
GET /api/v1/dashboard/intern/overview/activity/?page=1&page_size=5
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "a3f91c2e-0001-4b2d-9bbb-000000000001",
        "task_title": "Daily Standup Log",
        "karma": 10,
        "created_at": "2026-06-03T09:15:00Z"
      },
      {
        "id": "a3f91c2e-0001-4b2d-9bbb-000000000002",
        "task_title": "Weekly Review Submission",
        "karma": 25,
        "created_at": "2026-05-30T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 14,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /intern/overview/leaderboard/top/`

Returns the top 3 interns across all guilds by computed leaderboard score.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/overview/leaderboard/top/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": [
    {
      "user_id": "usr-uuid-001",
      "full_name": "Alice Johnson",
      "guild": "CATALYST",
      "score": 820.0,
      "rank": 1
    },
    {
      "user_id": "usr-uuid-002",
      "full_name": "Bob Smith",
      "guild": "NEXUS",
      "score": 710.5,
      "rank": 2
    },
    {
      "user_id": "usr-uuid-003",
      "full_name": "Carol White",
      "guild": "VANGUARD",
      "score": 680.0,
      "rank": 3
    }
  ]
}
```

---

### 2. Timesheets

#### `GET /intern/timesheets/`

Returns a paginated list of all timesheets submitted by the authenticated intern.

**Role:** `Intern`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `entry_date`, `status`, `category` |
| `sortBy` | string | `entry_date` or `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/intern/timesheets/?page=1&sortBy=entry_date
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "ts-uuid-001",
        "entry_date": "2026-06-03",
        "task_id": "task-uuid-001",
        "category": "Development",
        "description": "Worked on intern onboarding flow",
        "hours": "6.50",
        "blockers": "None",
        "task_status": "IN_PROGRESS",
        "remark": null,
        "end_of_day_note": "Good progress made today",
        "edit_reason": null,
        "status": "PENDING",
        "karma_awarded": null,
        "review_note": null,
        "created_at": "2026-06-03T18:00:00Z"
      }
    ],
    "pagination": {
      "count": 20,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /intern/timesheets/<timesheet_id>/`

Returns a single timesheet entry by ID (must belong to the authenticated intern).

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/timesheets/ts-uuid-001/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "id": "ts-uuid-001",
    "entry_date": "2026-06-03",
    "task_id": "task-uuid-001",
    "category": "Development",
    "description": "Worked on intern onboarding flow",
    "hours": "6.50",
    "blockers": "None",
    "task_status": "IN_PROGRESS",
    "remark": null,
    "end_of_day_note": "Good progress today",
    "edit_reason": null,
    "status": "PENDING",
    "karma_awarded": null,
    "review_note": null,
    "created_at": "2026-06-03T18:00:00Z"
  }
}
```

---

#### `POST /intern/timesheets/`

Submit a new daily timesheet entry. Only active interns can submit.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entry_date` | date | Yes | Date of the entry (not future) |
| `description` | string | Yes* | Description of work done |
| `log_description` | string | Yes* | Alias for `description` |
| `hours` | decimal | Yes* | Hours worked (> 0) |
| `hours_worked` | decimal | Yes* | Alias for `hours` |
| `category` | string | No | Category of work |
| `task` | uuid | No | Linked intern task ID |
| `task_status` | string | Conditional | Required if `task` is provided (`PENDING`, `IN_PROGRESS`, `COMPLETED`) |
| `blockers` | string | No | Any blockers faced |
| `remark` | string | No | Additional remarks |
| `end_of_day_note` | string | No | EOD note |
| `edit_reason` | string | Conditional | Required for late submissions (entry_date < yesterday) |

> *One of `description`/`log_description` and one of `hours`/`hours_worked` is required.

**Sample Request:**
```http
POST /api/v1/dashboard/intern/timesheets/
Authorization: Bearer <token>
Content-Type: application/json

{
  "entry_date": "2026-06-03",
  "description": "Implemented the leave request form UI and connected it to the backend API",
  "hours": 7.5,
  "category": "Development",
  "task": "task-uuid-001",
  "task_status": "IN_PROGRESS",
  "blockers": "None",
  "end_of_day_note": "Will continue the review flow tomorrow"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Timesheet submitted successfully.",
  "response": {}
}
```

**Error – Duplicate Submission (409):**
```json
{
  "hasError": true,
  "statusCode": 409,
  "message": "Timesheet for this date already exists.",
  "response": {}
}
```

---

#### `PATCH /intern/timesheets/<timesheet_id>/`

Update editable fields (`remark`, `end_of_day_note`) on a submitted timesheet. Requires an `edit_reason`.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `edit_reason` | string | Yes | Reason for editing |
| `remark` | string | No | Updated remark |
| `end_of_day_note` | string | No | Updated EOD note |

**Sample Request:**
```http
PATCH /api/v1/dashboard/intern/timesheets/ts-uuid-001/
Authorization: Bearer <token>
Content-Type: application/json

{
  "edit_reason": "Corrected the EOD note to better reflect progress",
  "end_of_day_note": "Completed 70% of the leave request integration"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Timesheet updated successfully.",
  "response": {}
}
```

---

#### `GET /intern/timesheets/today/`

Returns today's timesheet for the authenticated intern, if submitted.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/timesheets/today/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "id": "ts-uuid-002",
    "entry_date": "2026-06-04",
    "task_id": null,
    "category": "Research",
    "description": "Explored API documentation tooling",
    "hours": "4.00",
    "blockers": "None",
    "task_status": null,
    "remark": null,
    "end_of_day_note": "Finished today's research",
    "edit_reason": null,
    "status": "PENDING",
    "karma_awarded": null,
    "review_note": null,
    "created_at": "2026-06-04T17:00:00Z"
  }
}
```

---

#### `GET /intern/timesheets/history/`

Returns a paginated timesheet history for the authenticated intern (same as `GET /intern/timesheets/` – dedicated history route).

**Role:** `Intern`

**Query Parameters:** Same as `GET /intern/timesheets/`

---

#### `GET /intern/timesheets/summary/`

Returns the authenticated intern's current and longest timesheet streak.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/timesheets/summary/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "current_streak": 5,
    "longest_streak": 14
  }
}
```

---

### 3. Weekly Reviews

#### `GET /intern/reviews/`

Returns a paginated list of all weekly review submissions for the authenticated intern.

**Role:** `Intern`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `iso_year`, `iso_week`, `status` |
| `sortBy` | string | `iso_year`, `iso_week`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/intern/reviews/?page=1&sortBy=iso_week
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "rev-uuid-001",
        "iso_year": 2026,
        "iso_week": 23,
        "week_start_date": "2026-06-01",
        "week_end_date": "2026-06-07",
        "team": "CATALYST",
        "is_on_leave": false,
        "tasks_assigned": "Build intern leave API",
        "tasks_completed": "Leave request form UI",
        "weekly_review": "Good week overall",
        "task_remarks": {
          "rating": 4,
          "next_week_plan": "Complete review integration",
          "challenges_faced": "Understanding streak logic",
          "learnings": "Django transactions, streaks"
        },
        "hours_committed": 35,
        "blockers": "None",
        "leave_days": 0,
        "suggestions": "Better documentation",
        "is_late": false,
        "status": "PENDING",
        "karma_awarded": null,
        "review_note": null,
        "created_at": "2026-06-04T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 5,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### `GET /intern/reviews/<review_id>/`

Returns a single weekly review by ID (must belong to the authenticated intern).

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/reviews/rev-uuid-001/
Authorization: Bearer <token>
```

**Sample Response:** Same shape as a single item in the list above.

---

#### `POST /intern/reviews/`

Submit a new weekly review. Only active interns can submit.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `team` | string | No | Guild/team name |
| `is_on_leave` | bool | No | Whether intern was on leave this week (default: false) |
| `hours_committed` | int | Conditional | Required if not on leave (> 0) |
| `tasks_assigned` | string | No | Tasks assigned this week |
| `tasks_completed` | string | No | Tasks completed this week |
| `weekly_review` | string | No | Free-text review |
| `task_remarks` | object | No | Structured remarks (JSON) |
| `blockers` | string | No | Any blockers |
| `leave_days` | int | No | Number of leave days |
| `suggestions` | string | No | Suggestions for improvement |
| `week_start_date` | date | No | Defaults to current week's Monday |
| `rating` | int | No | Self-rating (stored in `task_remarks`) |
| `next_week_plan` | string | No | Plan for next week |
| `challenges_faced` | string | No | Challenges during the week |
| `learnings` | string | No | Key learnings |

**Sample Request:**
```http
POST /api/v1/dashboard/intern/reviews/
Authorization: Bearer <token>
Content-Type: application/json

{
  "team": "CATALYST",
  "is_on_leave": false,
  "hours_committed": 35,
  "tasks_assigned": "Build leave request API, write unit tests",
  "tasks_completed": "Leave request API complete",
  "rating": 4,
  "challenges_faced": "Understanding streak calculation logic",
  "learnings": "Django atomic transactions, custom permission decorators",
  "next_week_plan": "Complete timesheet review integration for admin panel",
  "blockers": "Waiting for design sign-off on leave UI",
  "suggestions": "Weekly sync calls would help"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Weekly review submitted successfully.",
  "response": {}
}
```

**Error – Duplicate Submission (409):**
```json
{
  "hasError": true,
  "statusCode": 409,
  "message": "Review for this week already exists.",
  "response": {}
}
```

---

#### `PATCH /intern/reviews/<review_id>/`

Update a pending weekly review (only edits for the current week are allowed).

**Role:** `Intern`

**Request Body:** Partial – any fields from the `POST` body.

**Sample Request:**
```http
PATCH /api/v1/dashboard/intern/reviews/rev-uuid-001/
Authorization: Bearer <token>
Content-Type: application/json

{
  "suggestions": "Add a Slack bot for timesheet reminders",
  "blockers": "None this week"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Weekly review updated successfully.",
  "response": {}
}
```

---

#### `GET /intern/reviews/current/`

Returns this week's review for the authenticated intern.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/reviews/current/
Authorization: Bearer <token>
```

**Sample Response:** Same shape as a single review object.

---

#### `GET /intern/reviews/history/`

Returns paginated review history (same as `GET /intern/reviews/` – dedicated history route).

**Role:** `Intern`

---

### 4. Leaderboard

#### `GET /intern/leaderboard/`

Returns the full intern leaderboard sorted by score, paginated.

**Role:** `Intern`, `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number (default: 1) |
| `page_size` | int | Results per page (default: 10) |

> **Score Formula:**  
> `score = (karma × KARMA_MULTIPLIER) + (daily_streak × DAILY_STREAK_MULTIPLIER) + (weekly_streak × WEEKLY_STREAK_MULTIPLIER) + (completed_tasks × COMPLETED_TASKS_MULTIPLIER) + (complexity_score × COMPLEXITY_SCORE_MULTIPLIER)`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/leaderboard/?page=1&page_size=10
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "user_id": "usr-uuid-001",
        "full_name": "Alice Johnson",
        "guild": "CATALYST",
        "score": 820.0,
        "rank": 1
      },
      {
        "user_id": "usr-uuid-002",
        "full_name": "Bob Smith",
        "guild": "NEXUS",
        "score": 710.5,
        "rank": 2
      }
    ],
    "pagination": {
      "count": 25,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /intern/leaderboard/me/`

Returns the authenticated intern's own rank and score on the leaderboard.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/leaderboard/me/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "rank": 5,
    "score": 472.5
  }
}
```

---

### 5. Tasks (Mine)

#### `GET /intern/tasks/mine/`

Returns a paginated list of intern tasks assigned to the authenticated intern.

**Role:** `Intern`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `title`, `status`, `category` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/intern/tasks/mine/?sortBy=status
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "task-uuid-001",
        "title": "Build Leave Request UI",
        "description": "Create the frontend form for submitting leave requests",
        "category": "Frontend",
        "complexity": "MEDIUM",
        "assigned_to": "usr-uuid-001",
        "assigned_to_name": "Alice Johnson",
        "status": "IN_PROGRESS",
        "created_by": "admin-uuid-001",
        "created_by_name": "Admin User",
        "created_at": "2026-05-28T09:00:00Z",
        "updated_at": "2026-06-03T14:00:00Z"
      }
    ],
    "pagination": {
      "count": 4,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### `PATCH /intern/tasks/<task_id>/`

Update the status of an assigned task.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | Yes | New status: `PENDING`, `IN_PROGRESS`, `COMPLETED` |

**Sample Request:**
```http
PATCH /api/v1/dashboard/intern/tasks/task-uuid-001/
Authorization: Bearer <token>
Content-Type: application/json

{
  "status": "COMPLETED"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task status updated successfully.",
  "response": {}
}
```

---

### 6. Leave

#### `GET /intern/leave/`

Returns a paginated list of all leave requests submitted by the authenticated intern.

**Role:** `Intern`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `leave_type`, `status` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/intern/leave/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "leave-uuid-001",
        "leave_type": "SICK",
        "start_date": "2026-06-05",
        "end_date": "2026-06-06",
        "reason": "Fever and cold",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2026-06-04T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 3,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### `GET /intern/leave/<leave_id>/`

Returns a single leave request by ID (must belong to the authenticated intern).

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/leave/leave-uuid-001/
Authorization: Bearer <token>
```

**Sample Response:** Same shape as a single item in the list above.

---

#### `POST /intern/leave/`

Submit a new leave request.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `leave_type` | string | Yes | `SICK`, `CASUAL`, `WFH`, `EMERGENCY` |
| `start_date` | date | Yes | Start date of leave |
| `end_date` | date | Yes | End date of leave (>= start_date) |
| `reason` | string | Yes | Reason for leave |
| `duration_days` | int | No | Auto-calculated if omitted |

**Sample Request:**
```http
POST /api/v1/dashboard/intern/leave/
Authorization: Bearer <token>
Content-Type: application/json

{
  "leave_type": "SICK",
  "start_date": "2026-06-05",
  "end_date": "2026-06-06",
  "reason": "Fever and cold, advised rest by doctor"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Leave request submitted successfully.",
  "response": {}
}
```

---

#### `PATCH /intern/leave/<leave_id>/`

Cancel a pending leave request.

**Role:** `Intern`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | Must be `"cancel"` |

**Sample Request:**
```http
PATCH /api/v1/dashboard/intern/leave/leave-uuid-001/
Authorization: Bearer <token>
Content-Type: application/json

{
  "action": "cancel"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Leave request cancelled.",
  "response": {}
}
```

---

#### `GET /intern/leave/history/`

Returns the full leave history for the authenticated intern (same as `GET /intern/leave/` – dedicated history route).

---

#### `GET /intern/leave/balance/`

Returns the leave balance for the authenticated intern for the current month/week.

**Role:** `Intern`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/leave/balance/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "SICK": {
      "limit": 2,
      "used": 1,
      "remaining": 1,
      "period": "month"
    },
    "CASUAL": {
      "limit": 1,
      "used": 0,
      "remaining": 1,
      "period": "month"
    },
    "WFH": {
      "limit": 2,
      "used": 0,
      "remaining": 2,
      "period": "week"
    },
    "EMERGENCY": {
      "limit": null,
      "used": 0,
      "remaining": null,
      "period": "unlimited"
    }
  }
}
```

---

### 7. Guilds

#### `GET /intern/guilds/`

Returns a list of all available intern guilds.

**Role:** `Intern`, `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/intern/guilds/
Authorization: Bearer <token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": ["CATALYST", "NEXUS", "VANGUARD", "PHOENIX", "AURORA"]
}
```

---

## Manage Interns APIs

> All endpoints in this group require **Admin** role.

### 1. Interns Management

#### `GET /manage-interns/interns/`

Returns a paginated list of all interns with their guild and status details.

**Role:** `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `full_name`, `guild`, `status` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/interns/?page=1&sortBy=status
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "guild-link-uuid-001",
        "user": "usr-uuid-001",
        "full_name": "Alice Johnson",
        "muid": "alice@mulearn",
        "guild": "CATALYST",
        "status": "ACTIVE",
        "created_at": "2026-05-01T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 30,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /manage-interns/interns/<intern_id>/`

Returns details of a specific intern by their guild-link ID.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/interns/guild-link-uuid-001/
Authorization: Bearer <admin-token>
```

**Sample Response:** Same shape as a single item in the list above.

---

#### `POST /manage-interns/interns/`

Onboard a new intern. Automatically assigns the `Intern` role to the user.

**Role:** `Admin`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mu_id` | string | Conditional | User's MuLearn ID (muid). Use this or `user_id` |
| `user_id` | uuid | Conditional | User's UUID. Use this or `mu_id` |
| `guild` | string | Yes | Guild to assign: `CATALYST`, `NEXUS`, `VANGUARD`, etc. |
| `status` | string | No | Default: `ACTIVE` |

**Sample Request:**
```http
POST /api/v1/dashboard/manage-interns/interns/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "mu_id": "alice@mulearn",
  "guild": "CATALYST"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Intern onboarded successfully.",
  "response": {}
}
```

---

#### `PATCH /manage-interns/interns/<intern_id>/`

Update an intern's guild or status.

**Role:** `Admin`

**Request Body (partial):**

| Field | Type | Description |
|-------|------|-------------|
| `guild` | string | New guild assignment |
| `status` | string | `ACTIVE`, `AT_RISK`, `ON_LEAVE`, `INACTIVE` |

**Sample Request:**
```http
PATCH /api/v1/dashboard/manage-interns/interns/guild-link-uuid-001/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "guild": "NEXUS",
  "status": "ACTIVE"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Intern details updated successfully.",
  "response": {}
}
```

---

#### `DELETE /manage-interns/interns/<intern_id>/`

Deactivates an intern (sets status to `INACTIVE`) and removes their `Intern` role.

**Role:** `Admin`

**Sample Request:**
```http
DELETE /api/v1/dashboard/manage-interns/interns/guild-link-uuid-001/
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Intern deactivated successfully.",
  "response": {}
}
```

---

### 2. Status Stats

#### `GET /manage-interns/status/`

Returns a breakdown of intern counts by status.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/status/
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "ACTIVE": 22,
    "AT_RISK": 3,
    "ON_LEAVE": 1,
    "INACTIVE": 4
  }
}
```

---

### 3. Export Interns

#### `GET /manage-interns/interns/export/`

Downloads all intern records as a CSV file.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/interns/export/
Authorization: Bearer <admin-token>
```

**Response:** `text/csv` file with `Content-Disposition: attachment; filename="interns.csv"`

**CSV Columns:**

| Column | Description |
|--------|-------------|
| `Full Name` | Intern's full name |
| `MuID` | Intern's mulearn ID |
| `Guild` | Assigned guild |
| `Status` | Current status |
| `Created At` | Onboarding date (YYYY-MM-DD HH:MM:SS) |

---

### 4. Reviews & Timesheets (Admin)

#### `GET /manage-interns/reviews/`

Returns a paginated list of all weekly review submissions across all interns.

**Role:** `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status: `PENDING`, `APPROVED`, `REJECTED` |
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `user__full_name`, `status`, `team` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/reviews/?status=PENDING&page=1
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "rev-uuid-001",
        "user_id": "usr-uuid-001",
        "user_name": "Alice Johnson",
        "muid": "alice@mulearn",
        "iso_year": 2026,
        "iso_week": 23,
        "week_start_date": "2026-06-01",
        "week_end_date": "2026-06-07",
        "team": "CATALYST",
        "is_on_leave": false,
        "tasks_assigned": "Build leave API",
        "tasks_completed": "Leave API complete",
        "weekly_review": "Great week",
        "task_remarks": { "rating": 4 },
        "hours_committed": 35,
        "blockers": "None",
        "leave_days": 0,
        "suggestions": null,
        "is_late": false,
        "status": "PENDING",
        "karma_awarded": null,
        "review_note": null,
        "created_at": "2026-06-04T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 8,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### `GET /manage-interns/reviews/reviews/<review_id>/review/`

Returns a specific weekly review for admin inspection.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/reviews/reviews/rev-uuid-001/review/
Authorization: Bearer <admin-token>
```

**Sample Response:** Same shape as single item in the list above.

---

#### `PATCH /manage-interns/reviews/reviews/<review_id>/review/`

Approve or reject a pending weekly review. Approving awards karma and updates the streak.

**Role:** `Admin`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `"approve"` or `"reject"` |
| `review_note` | string | No | Admin's note (mandatory for rejection best practice) |

**Sample Request – Approve:**
```http
PATCH /api/v1/dashboard/manage-interns/reviews/reviews/rev-uuid-001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "action": "approve"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Weekly review approved successfully.",
  "response": {}
}
```

**Sample Request – Reject:**
```http
PATCH /api/v1/dashboard/manage-interns/reviews/reviews/rev-uuid-001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "action": "reject",
  "review_note": "Review is incomplete. Please add tasks completed and hours committed."
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Weekly review rejected successfully.",
  "response": {}
}
```

---

#### `GET /manage-interns/reviews/timesheets/`

Returns a paginated list of all daily timesheet submissions across all interns.

**Role:** `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by status: `PENDING`, `APPROVED`, `REJECTED` |
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `user__full_name`, `status`, `category` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/reviews/timesheets/?status=PENDING
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "ts-uuid-001",
        "user_id": "usr-uuid-001",
        "user_name": "Alice Johnson",
        "muid": "alice@mulearn",
        "entry_date": "2026-06-03",
        "task_id": "task-uuid-001",
        "category": "Development",
        "description": "Worked on leave request form",
        "hours": "7.50",
        "blockers": "None",
        "task_status": "IN_PROGRESS",
        "remark": null,
        "end_of_day_note": "70% done",
        "edit_reason": null,
        "status": "PENDING",
        "karma_awarded": null,
        "review_note": null,
        "created_at": "2026-06-03T18:00:00Z"
      }
    ],
    "pagination": {
      "count": 15,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /manage-interns/reviews/timesheets/<timesheet_id>/review/`

Returns a specific timesheet for admin inspection.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/reviews/timesheets/ts-uuid-001/review/
Authorization: Bearer <admin-token>
```

**Sample Response:** Same shape as single timesheet item above.

---

#### `PATCH /manage-interns/reviews/timesheets/<timesheet_id>/review/`

Approve or reject a pending daily timesheet. Approving awards karma, updates streak, and may trigger milestone bonuses.

**Role:** `Admin`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `"approve"` or `"reject"` |
| `review_note` | string | No | Admin's feedback note |

**Karma Streak Multipliers (on approval):**

| Streak Days | Karma Multiplier |
|-------------|-----------------|
| < 7 | 1.0× |
| ≥ 7 | 1.2× |
| ≥ 14 | 1.5× |
| ≥ 30 | 2.0× |

**Milestone Streak Bonuses:** Additional karma is awarded at streaks of 7, 14, 30, 60, and 90 days.

**Sample Request:**
```http
PATCH /api/v1/dashboard/manage-interns/reviews/timesheets/ts-uuid-001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "action": "approve"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Timesheet approved successfully.",
  "response": {}
}
```

---

### 5. Tasks (Admin)

#### `GET /manage-interns/tasks/`

Returns a paginated list of all intern tasks.

**Role:** `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `title`, `status`, `category`, `assigned_to__full_name` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/tasks/?sortBy=created_at
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "task-uuid-001",
        "title": "Build Leave Request UI",
        "description": "Create the leave request form",
        "category": "Frontend",
        "complexity": "MEDIUM",
        "assigned_to": "usr-uuid-001",
        "status": "IN_PROGRESS",
        "team": "CATALYST",
        "deadline": "2026-06-10",
        "iso_week": 24,
        "created_at": "2026-05-28T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 12,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### `GET /manage-interns/tasks/<task_id>/`

Returns a specific task by ID.

**Role:** `Admin`

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/tasks/task-uuid-001/
Authorization: Bearer <admin-token>
```

**Sample Response:** Same shape as a single task item above.

---

#### `POST /manage-interns/tasks/`

Create a new intern task and assign it to an intern.

**Role:** `Admin`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Task title |
| `description` | string | No | Task description |
| `category` | string | No | Task category |
| `complexity` | string | No | `LOW`, `MEDIUM`, `HIGH`, `CRITICAL` |
| `assigned_to` | string | Yes | Intern's `muid` or `user_id` |
| `status` | string | No | Default: `PENDING` |
| `team` / `guild` | string | No | Guild name (alias: `guild`) |
| `deadline` | date | No | Task deadline |
| `iso_week` | int | No | Auto-calculated from deadline if omitted |

**Sample Request:**
```http
POST /api/v1/dashboard/manage-interns/tasks/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "title": "Build Leave Request UI",
  "description": "Create the frontend form for submitting leave requests with validation",
  "category": "Frontend",
  "complexity": "MEDIUM",
  "assigned_to": "alice@mulearn",
  "guild": "CATALYST",
  "deadline": "2026-06-10"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task created successfully.",
  "response": {}
}
```

---

#### `PATCH /manage-interns/tasks/<task_id>/`

Update an existing intern task.

**Role:** `Admin`

**Request Body (partial):** Any fields from `POST /manage-interns/tasks/`

**Sample Request:**
```http
PATCH /api/v1/dashboard/manage-interns/tasks/task-uuid-001/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "status": "COMPLETED",
  "complexity": "HIGH"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task updated successfully.",
  "response": {}
}
```

---

#### `DELETE /manage-interns/tasks/<task_id>/`

Permanently deletes an intern task.

**Role:** `Admin`

**Sample Request:**
```http
DELETE /api/v1/dashboard/manage-interns/tasks/task-uuid-001/
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task deleted successfully.",
  "response": {}
}
```

---

### 6. Leave (Admin)

#### `GET /manage-interns/leave/`

Returns a paginated list of all leave requests from all interns.

**Role:** `Admin`

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number |
| `page_size` | int | Results per page |
| `search` | string | Filter by `user__full_name`, `leave_type`, `status` |
| `sortBy` | string | `created_at`, `status` |

**Sample Request:**
```http
GET /api/v1/dashboard/manage-interns/leave/?sortBy=created_at
Authorization: Bearer <admin-token>
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "data": [
      {
        "id": "leave-uuid-001",
        "user": "usr-uuid-001",
        "user_name": "Alice Johnson",
        "leave_type": "SICK",
        "start_date": "2026-06-05",
        "end_date": "2026-06-06",
        "reason": "Fever and cold",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2026-06-04T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 5,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### `PATCH /manage-interns/leave/<leave_id>/review/`

Approve or reject a pending leave request. If approved and the leave period includes today, the intern's status is automatically set to `ON_LEAVE`.

**Role:** `Admin`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | Yes | `"approve"` or `"reject"` |
| `review_note` | string | No | Admin's feedback note |

**Sample Request – Approve:**
```http
PATCH /api/v1/dashboard/manage-interns/leave/leave-uuid-001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "action": "approve"
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Leave request approved successfully.",
  "response": {}
}
```

**Sample Request – Reject:**
```http
PATCH /api/v1/dashboard/manage-interns/leave/leave-uuid-001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "action": "reject",
  "review_note": "Leave quota exhausted for this month."
}
```

**Sample Response (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Leave request rejected successfully.",
  "response": {}
}
```

---

## Common Query Parameters

All paginated list endpoints support:

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | 1 | Page number |
| `page_size` | int | 10 | Items per page |
| `search` | string | — | Search term (fields vary per endpoint) |
| `sortBy` | string | — | Field to sort by |

---

## Common Response Format

All responses follow this wrapper shape:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": { ... }
}
```

**Error Response:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Error message here.",
  "response": {
    "field_name": ["Validation error detail"]
  }
}
```

---

## Enumerations

### Intern Status (`InternGuildStatus`)
| Value | Description |
|-------|-------------|
| `ACTIVE` | Intern is actively participating |
| `AT_RISK` | Intern has missed submissions |
| `ON_LEAVE` | Intern is on approved leave |
| `INACTIVE` | Intern has been deactivated |

### Submission Status (`InternSubmissionStatus`)
| Value | Description |
|-------|-------------|
| `PENDING` | Awaiting admin review |
| `APPROVED` | Approved by admin |
| `REJECTED` | Rejected by admin |

### Leave Type
| Value | Period Limit | Description |
|-------|-------------|-------------|
| `SICK` | 2 days/month | Medical leave |
| `CASUAL` | 1 day/month | Casual leave |
| `WFH` | 2 days/week | Work from home |
| `EMERGENCY` | Unlimited | Emergency leave |

### Task Status
| Value | Description |
|-------|-------------|
| `PENDING` | Not started |
| `IN_PROGRESS` | Actively being worked on |
| `COMPLETED` | Finished |

### Task Complexity & Score Weights
| Complexity | Score Weight |
|------------|-------------|
| `LOW` | 1 |
| `MEDIUM` | 2 |
| `HIGH` | 3 |
| `CRITICAL` | 5 |

---

*Generated from source code at `api/dashboard/intern/` and `api/dashboard/manage_interns/`*
