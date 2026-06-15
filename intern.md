# Intern Dashboard API Reference

**Base URL:** `/api/v1/dashboard/intern/`  
**Authentication:** All endpoints require a valid JWT token via the `Authorization: Bearer <token>` header.  
**Default Role Required:** `INTERN` (unless stated otherwise)

---

## Table of Contents

1. [Overview](#1-overview)
2. [Timesheets](#2-timesheets)
3. [Weekly Reviews](#3-weekly-reviews)
4. [Tasks](#4-tasks)
5. [Leave Management](#5-leave-management)
6. [Leaderboard](#6-leaderboard)
7. [Guilds](#7-guilds)
8. [Common Patterns](#8-common-patterns)

---

## 1. Overview

Base path: `/api/v1/dashboard/intern/overview/`

### 1.1 `GET /overview/status/`

Retrieve the authenticated intern's overview stats: guild, karma, streaks, completed tasks, complexity score, and weighted leaderboard score.

**Request**
```http
GET /api/v1/dashboard/intern/overview/status/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "guild": "DESIGN",
    "status": "ACTIVE",
    "total_intern_karma": 340,
    "daily_streak": 12,
    "weekly_streak": 5,
    "completed_tasks": 8,
    "complexity_score": 22,
    "score": 1248.5
  }
}
```

**Error `400 Bad Request`** — intern not found
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Not an intern."
}
```

---

### 1.2 `GET /overview/activity/`

Retrieve a paginated list of the intern's karma activity log (only `#intern-*` tagged tasks).

**Request**
```http
GET /api/v1/dashboard/intern/overview/activity/?page=1&perPage=10
Authorization: Bearer <token>
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | `1` | Page number |
| `perPage` | int | `10` | Items per page |
| `search` | string | — | Search by task title |
| `sortBy` | string | `created_at` | Sort field |
| `sortOrder` | string | `desc` | `asc` or `desc` |

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "task_title": "#intern-daily-timesheet",
        "karma": 10,
        "created_at": "2024-06-03T18:30:00Z"
      },
      {
        "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "task_title": "#intern-weekly-review",
        "karma": 25,
        "created_at": "2024-06-02T12:00:00Z"
      }
    ],
    "pagination": {
      "count": 24,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

### 1.3 `GET /overview/leaderboard/top/`

Retrieve the top 3 interns on the leaderboard (preview widget for the overview dashboard).

**Request**
```http
GET /api/v1/dashboard/intern/overview/leaderboard/top/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": [
    {
      "user_id": "a1b2c3d4-0000-0000-0000-000000000001",
      "full_name": "Arjun Nair",
      "guild": "BACKEND",
      "score": 2100.0,
      "rank": 1
    },
    {
      "user_id": "a1b2c3d4-0000-0000-0000-000000000002",
      "full_name": "Meera Pillai",
      "guild": "DESIGN",
      "score": 1850.0,
      "rank": 2
    },
    {
      "user_id": "a1b2c3d4-0000-0000-0000-000000000003",
      "full_name": "Rahul Krishnan",
      "guild": "FRONTEND",
      "score": 1620.5,
      "rank": 3
    }
  ]
}
```

---

## 2. Timesheets

Base path: `/api/v1/dashboard/intern/timesheets/`

### 2.1 `GET /timesheets/`

Retrieve a paginated list of all the authenticated intern's timesheet entries.

**Request**
```http
GET /api/v1/dashboard/intern/timesheets/?page=1&perPage=10
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "ts-uuid-0001",
        "entry_date": "2024-06-03",
        "task_id": null,
        "category": "DEVELOPMENT",
        "description": "Worked on the authentication module",
        "hours": "6.50",
        "blockers": "None",
        "task_status": null,
        "remark": null,
        "end_of_day_note": "Completed JWT integration",
        "edit_reason": null,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2024-06-03T18:45:00Z"
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

### 2.2 `GET /timesheets/<timesheet_id>/`

Retrieve a single timesheet entry by ID.

**Request**
```http
GET /api/v1/dashboard/intern/timesheets/ts-uuid-0001/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "id": "ts-uuid-0001",
    "entry_date": "2024-06-03",
    "task_id": "task-uuid-0099",
    "category": "DEVELOPMENT",
    "description": "Worked on the authentication module",
    "hours": "6.50",
    "blockers": "None",
    "task_status": "IN_PROGRESS",
    "remark": null,
    "end_of_day_note": "Completed JWT integration",
    "edit_reason": null,
    "status": "PENDING",
    "review_note": null,
    "created_at": "2024-06-03T18:45:00Z"
  }
}
```

**Error `400`** — not found
```json
{ "hasError": true, "statusCode": 400, "message": "Timesheet not found." }
```

---

### 2.3 `POST /timesheets/`

Submit a new daily timesheet entry. Only one entry per date is allowed.

**Request**
```http
POST /api/v1/dashboard/intern/timesheets/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "entry_date": "2024-06-04",
  "category": "DEVELOPMENT",
  "description": "Implemented leave management API endpoints",
  "hours": "7.00",
  "blockers": "None",
  "end_of_day_note": "All endpoints tested and working"
}
```

**With a linked task:**
```json
{
  "entry_date": "2024-06-04",
  "category": "DEVELOPMENT",
  "task": "task-uuid-0099",
  "task_status": "COMPLETED",
  "task_output_link": "https://github.com/my-repo/pr-123",
  "description": "Completed the assigned task on leave module",
  "hours": "5.50",
  "blockers": "None"
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `entry_date` | date (`YYYY-MM-DD`) | ✅ | Must not be a future date. Dates older than yesterday require `edit_reason`. |
| `category` | string | ✅ | Work category (e.g. `DEVELOPMENT`, `DESIGN`, `TESTING`) |
| `description` | string | ✅ | Description of work done (alias: `log_description`) |
| `hours` | decimal | ✅ | Hours worked, must be `> 0` (alias: `hours_worked`) |
| `blockers` | string | ❌ | Any blockers encountered |
| `task` | UUID | ❌ | Linked intern task ID |
| `task_status` | string | ❌ (required if `task` is set) | New status for the linked task |
| `task_output_link` | URL | ❌ (required if `task_status` is `COMPLETED`) | Output link proving completion |
| `end_of_day_note` | string | ❌ | End-of-day summary |
| `edit_reason` | string | ❌ (required for late submissions) | Reason for late entry |

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Timesheet submitted successfully." }
```

**Error `409 Conflict`** — duplicate entry for date
```json
{ "hasError": true, "statusCode": 409, "message": "Timesheet for this date already exists." }
```

---

### 2.4 `PATCH /timesheets/<timesheet_id>/`

Edit the `remark` or `end_of_day_note` of an existing timesheet. `edit_reason` is mandatory.

**Request**
```http
PATCH /api/v1/dashboard/intern/timesheets/ts-uuid-0001/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "remark": "Reviewed by senior dev — all good",
  "end_of_day_note": "Also fixed a bug in the auth flow",
  "edit_reason": "Forgot to add EOD note earlier"
}
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Timesheet updated successfully." }
```

**Error `400`** — no edit_reason provided
```json
{ "hasError": true, "statusCode": 400, "message": "edit_reason is mandatory for modifications." }
```

---

### 2.5 `GET /timesheets/today/`

Retrieve today's timesheet entry for the authenticated intern.

**Request**
```http
GET /api/v1/dashboard/intern/timesheets/today/
Authorization: Bearer <token>
```

**Response `200 OK`** — same shape as single timesheet response.

**Error `400`** — no entry today
```json
{ "hasError": true, "statusCode": 400, "message": "No timesheet submitted for today." }
```

---

### 2.6 `GET /timesheets/history/`

Retrieve paginated timesheet history (sorted by entry date descending).

**Request**
```http
GET /api/v1/dashboard/intern/timesheets/history/?page=1&perPage=20
Authorization: Bearer <token>
```

**Response** — same paginated shape as `GET /timesheets/`

---

### 2.7 `GET /timesheets/summary/`

Retrieve intern streak statistics for daily timesheet submissions.

**Request**
```http
GET /api/v1/dashboard/intern/timesheets/summary/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "current_streak": 12,
    "longest_streak": 18
  }
}
```

---

## 3. Weekly Reviews

Base path: `/api/v1/dashboard/intern/reviews/`

### 3.1 `GET /reviews/`

Retrieve all weekly reviews for the authenticated intern (paginated, latest first).

**Request**
```http
GET /api/v1/dashboard/intern/reviews/?page=1&perPage=10
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "rev-uuid-0001",
        "iso_year": 2024,
        "iso_week": 23,
        "week_start_date": "2024-06-03",
        "week_end_date": "2024-06-09",
        "team": "Backend",
        "is_on_leave": false,
        "tasks_assigned": {
          "task-uuid-0099": "COMPLETED"
        },
        "tasks_completed": "Leave API done, auth bug fixed",
        "weekly_review": "Learnings: Improved understanding of JWT. Challenges Faced: Complex edge cases in date logic.",
        "task_remarks": {
          "rating": 4,
          "next_week_plan": "Start timesheet module",
          "challenges_faced": "Complex edge cases in date logic",
          "learnings": "Improved understanding of JWT"
        },
        "hours_committed": 35,
        "blockers": "None this week",
        "leave_days": 0,
        "suggestions": null,
        "is_late": false,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2024-06-07T17:00:00Z"
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

### 3.2 `GET /reviews/<review_id>/`

Retrieve a single weekly review by ID.

**Request**
```http
GET /api/v1/dashboard/intern/reviews/rev-uuid-0001/
Authorization: Bearer <token>
```

**Response `200 OK`** — same shape as a single item in the list above.

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Weekly review not found." }
```

---

### 3.3 `POST /reviews/`

Submit a new weekly review. Only one review is allowed per ISO week.

**Request**
```http
POST /api/v1/dashboard/intern/reviews/
Authorization: Bearer <token>
Content-Type: application/json
```

**Standard submission (not on leave):**
```json
{
  "team": "Backend",
  "is_on_leave": false,
  "tasks_assigned": {
    "task-uuid-0099": "COMPLETED",
    "task-uuid-0100": "IN_PROGRESS"
  },
  "tasks_completed": "Leave API complete, tests written",
  "hours_committed": 35,
  "blockers": "None",
  "leave_days": 0,
  "suggestions": "Add more intern onboarding docs",
  "rating": 4,
  "learnings": "Deep dive into Django ORM aggregations",
  "challenges_faced": "Debugging timezone-aware date comparisons",
  "next_week_plan": "Start timesheet module"
}
```

**On-leave submission:**
```json
{
  "team": "Backend",
  "is_on_leave": true,
  "leave_days": 4,
  "hours_committed": 0,
  "blockers": null
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `team` | string | ✅ | The intern's team name |
| `is_on_leave` | boolean | ✅ | Whether the intern was on leave this week |
| `hours_committed` | int | ✅ (if not on leave) | Hours worked this week, must be `> 0` |
| `tasks_assigned` | object (JSON) | ❌ | Key-value pairs of Task IDs and their current statuses |
| `tasks_completed` | string | ❌ | Tasks completed during the week |
| `weekly_review` | string | ❌ | Free-form review text (auto-generated if omitted) |
| `blockers` | string | ❌ | Any blockers |
| `leave_days` | int | ❌ | Number of days on leave |
| `suggestions` | string | ❌ | Suggestions or feedback |
| `rating` | int | ❌ | Self-rating (stored in `task_remarks`) |
| `learnings` | string | ❌ | Key learnings (stored in `task_remarks`) |
| `challenges_faced` | string | ❌ | Challenges faced (stored in `task_remarks`) |
| `next_week_plan` | string | ❌ | Plan for next week (stored in `task_remarks`) |
| `week_start_date` | date | ❌ | Defaults to current week's Monday |

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Weekly review submitted successfully." }
```

**Error `409 Conflict`** — review already exists for this week
```json
{ "hasError": true, "statusCode": 409, "message": "Review for this week already exists." }
```

---

### 3.4 `PATCH /reviews/<review_id>/`

Edit a pending review, but **only for the current ISO week**.

**Request**
```http
PATCH /api/v1/dashboard/intern/reviews/rev-uuid-0001/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "blockers": "Encountered a DB migration conflict",
  "suggestions": "Need better local dev setup docs",
  "rating": 5
}
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Weekly review updated successfully." }
```

**Error `400`** — wrong week or not pending
```json
{ "hasError": true, "statusCode": 400, "message": "Cannot edit reviews for past weeks." }
```

---

### 3.5 `GET /reviews/current/`

Retrieve the current week's submitted review.

**Request**
```http
GET /api/v1/dashboard/intern/reviews/current/
Authorization: Bearer <token>
```

**Response `200 OK`** — same shape as a single review.

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "No review submitted for the current week." }
```

---

### 3.6 `GET /reviews/history/`

Retrieve paginated review history (sorted by year and week descending).

**Request**
```http
GET /api/v1/dashboard/intern/reviews/history/?page=1&perPage=10
Authorization: Bearer <token>
```

**Response** — same paginated shape as `GET /reviews/`

---

## 4. Tasks

Base path: `/api/v1/dashboard/intern/tasks/`

### 4.1 `GET /tasks/mine/`

Retrieve all tasks assigned to the authenticated intern.

**Request**
```http
GET /api/v1/dashboard/intern/tasks/mine/?page=1&perPage=10
Authorization: Bearer <token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search by `title`, `status`, or `category` |
| `sortBy` | string | `created_at` (default) or `status` |
| `sortOrder` | string | `asc` / `desc` |

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "task-uuid-0099",
        "title": "Build leave management API",
        "description": "Create CRUD endpoints for intern leave requests",
        "category": "DEVELOPMENT",
        "complexity": "HIGH",
        "assigned_to": "user-uuid-intern-001",
        "assigned_to_name": "Arun Dev",
        "status": "IN_PROGRESS",
        "karma_awarded": 0,
        "output_link": null,
        "is_verified": false,
        "verified_by": null,
        "created_by": "user-uuid-mentor-001",
        "created_by_name": "Priya Mentor",
        "created_at": "2024-05-28T10:00:00Z",
        "updated_at": "2024-06-03T14:30:00Z"
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

### 4.2 `PATCH /tasks/<task_id>/`

Update the status of a specific assigned task. Note that you cannot edit a task that has already been verified by an admin.

**Request**
```http
PATCH /api/v1/dashboard/intern/tasks/task-uuid-0099/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "status": "COMPLETED",
  "output_link": "https://github.com/my-repo/pr-123"
}
```

**Valid status values:** `TODO`, `IN_PROGRESS`, `COMPLETED`, `ON_HOLD`, `OVERDUE`

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Task status updated successfully." }
```

**Error `400`** — task not found
```json
{ "hasError": true, "statusCode": 400, "message": "Task not found." }
```

**Error `400`** — status missing
```json
{ "hasError": true, "statusCode": 400, "message": "Status is required." }
```

**Error `400`** — output_link missing when marking as COMPLETED
```json
{ "hasError": true, "statusCode": 400, "message": "output_link is required when completing a task." }
```

**Error `400`** — task is already verified
```json
{ "hasError": true, "statusCode": 400, "message": "Task is already verified and cannot be edited." }
```

---

## 5. Leave Management

Base path: `/api/v1/dashboard/intern/leave/`

### 5.1 `GET /leave/`

Retrieve a paginated list of all the intern's leave requests.

**Request**
```http
GET /api/v1/dashboard/intern/leave/?page=1&perPage=10
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "leave-uuid-0001",
        "leave_type": "SICK",
        "start_date": "2024-06-10",
        "end_date": "2024-06-11",
        "reason": "Fever and cold",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2024-06-09T08:00:00Z"
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

### 5.2 `GET /leave/<leave_id>/`

Retrieve a single leave request by ID.

**Request**
```http
GET /api/v1/dashboard/intern/leave/leave-uuid-0001/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "id": "leave-uuid-0001",
    "leave_type": "SICK",
    "start_date": "2024-06-10",
    "end_date": "2024-06-11",
    "reason": "Fever and cold",
    "status": "PENDING",
    "review_note": null,
    "created_at": "2024-06-09T08:00:00Z"
  }
}
```

---

### 5.3 `POST /leave/`

Submit a new leave request.

**Request**
```http
POST /api/v1/dashboard/intern/leave/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "leave_type": "SICK",
  "start_date": "2024-06-10",
  "end_date": "2024-06-11",
  "reason": "High fever, doctor advised rest"
}
```

**Leave types**

| Type | Period Limit | Description |
|------|-------------|-------------|
| `SICK` | 2 days / month | Sick leave |
| `CASUAL` | 1 day / month | Casual/personal leave |
| `WFH` | 2 days / week | Work from home |
| `EMERGENCY` | Unlimited | Emergency leave |

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `leave_type` | string | ✅ | One of `SICK`, `CASUAL`, `WFH`, `EMERGENCY` |
| `start_date` | date (`YYYY-MM-DD`) | ✅ | Leave start date |
| `end_date` | date (`YYYY-MM-DD`) | ✅ | Leave end date (must be ≥ `start_date`) |
| `reason` | string | ✅ | Reason for leave |
| `duration_days` | int | ❌ | Auto-calculated if omitted |

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Leave request submitted successfully." }
```

**Error `400`** — validation failure
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "start_date": ["Start date must be before or equal to end date."] }
}
```

---

### 5.4 `PATCH /leave/<leave_id>/` — Cancel Leave

Cancel a pending leave request.

**Request**
```http
PATCH /api/v1/dashboard/intern/leave/leave-uuid-0001/
Authorization: Bearer <token>
Content-Type: application/json
```

```json
{
  "action": "cancel"
}
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Leave request cancelled." }
```

**Error `400`** — not found or not in PENDING state
```json
{ "hasError": true, "statusCode": 400, "message": "Pending leave request not found." }
```

**Error `400`** — invalid action
```json
{ "hasError": true, "statusCode": 400, "message": "Invalid action." }
```

---

### 5.5 `GET /leave/history/`

Retrieve paginated leave history (same response as `GET /leave/`).

**Request**
```http
GET /api/v1/dashboard/intern/leave/history/?page=1&perPage=10
Authorization: Bearer <token>
```

---

### 5.6 `GET /leave/balance/`

Retrieve the intern's leave balance for each leave type.

**Request**
```http
GET /api/v1/dashboard/intern/leave/balance/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
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
      "used": 2,
      "remaining": 0,
      "period": "week"
    },
    "EMERGENCY": {
      "limit": null,
      "used": 1,
      "remaining": null,
      "period": "unlimited"
    }
  }
}
```

---

## 6. Leaderboard

Base path: `/api/v1/dashboard/intern/leaderboard/`

**Roles:** Both `INTERN` and `ADMIN` can access the full leaderboard. Only `INTERN` can access the `me/` endpoint.

### 6.1 `GET /leaderboard/`

Retrieve the full paginated intern leaderboard, sorted by score (descending).

**Request**
```http
GET /api/v1/dashboard/intern/leaderboard/?page=1&page_size=10
Authorization: Bearer <token>
```

**Query Parameters**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | int | `1` | Page number |
| `page_size` | int | `10` | Items per page |

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "data": [
      {
        "user_id": "a1b2c3d4-0000-0000-0000-000000000001",
        "full_name": "Arjun Nair",
        "guild": "BACKEND",
        "score": 2100.0,
        "rank": 1
      },
      {
        "user_id": "a1b2c3d4-0000-0000-0000-000000000002",
        "full_name": "Meera Pillai",
        "guild": "DESIGN",
        "score": 1850.0,
        "rank": 2
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

**Score Formula:**
```
score = (
    approved_daily_timesheets_count * 25 +
    approved_weekly_reviews_count * 50 +
    complexity_score * verified_intern_tasks_karma_sum
)
```

**Complexity score mapping:** `LOW=1`, `MEDIUM=2`, `HIGH=3`, `CRITICAL=5`

---

### 6.2 `GET /leaderboard/me/`

Retrieve the authenticated intern's own rank and score on the leaderboard.

**Request**
```http
GET /api/v1/dashboard/intern/leaderboard/me/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "rank": 4,
    "score": 1420.0
  }
}
```

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Not found in leaderboard." }
```

> Note: Only `ACTIVE` and `AT_RISK` interns appear in the leaderboard. `INACTIVE` interns will receive the not-found error.

---

## 7. Guilds

Base path: `/api/v1/dashboard/intern/`

### 7.1 `GET /guilds/`

Retrieve all available intern guild names.

**Roles:** `INTERN` or `ADMIN`

**Request**
```http
GET /api/v1/dashboard/intern/guilds/
Authorization: Bearer <token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": ["BACKEND", "FRONTEND", "DESIGN", "DEVOPS", "DATA", "QA"]
}
```

---

## 8. Common Patterns

### Authentication Header
```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### Pagination Query Parameters

All paginated list endpoints accept:

| Parameter | Description |
|-----------|-------------|
| `page` | Page number (default: `1`) |
| `perPage` | Items per page (default: `10`) |
| `search` | Search string |
| `sortBy` | Field to sort by |
| `sortOrder` | `asc` or `desc` (default: `desc`) |

### Standard Success Response
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": { ... }
}
```

### Standard Error Response
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Human readable error message."
}
```

### Leave & Review Statuses

| Status | Description |
|--------|-------------|
| `PENDING` | Submitted, awaiting mentor review |
| `APPROVED` | Approved by mentor |
| `REJECTED` | Rejected by mentor |
| `CANCELLED` | Cancelled by intern |

### Intern Status Values

| Status | Description |
|--------|-------------|
| `ACTIVE` | Active intern |
| `AT_RISK` | Intern flagged as at risk |
| `INACTIVE` | Inactive/offboarded intern |

### Timesheet / Review Submission Status

| Status | Description |
|--------|-------------|
| `PENDING` | Submitted, awaiting mentor review |
| `APPROVED` | Approved and karma awarded |
| `REJECTED` | Rejected by mentor |
