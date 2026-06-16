# Intern Dashboard & Intern Management — API Reference

> **Base prefix:** `/api/v1/dashboard/intern/` (Intern dashboard)  
> `/api/v1/dashboard/manage-interns/` (Admin management)  
> All endpoints require a valid **JWT Bearer token** in the `Authorization` header.

---

## Table of Contents

- [Enums & Constants](#enums--constants)
- [Pagination Query Params](#pagination-query-params)
- [Part 1 — Intern Dashboard (Role: Intern)](#part-1--intern-dashboard-role-intern)
  - [Overview](#1-overview)
  - [Tasks](#2-tasks)
  - [Timesheets](#3-timesheets)
  - [Weekly Reviews](#4-weekly-reviews)
  - [Leave](#5-leave)
  - [Leaderboard](#6-leaderboard)
  - [Guilds](#7-guilds)
- [Part 2 — Intern Management (Role: Admin)](#part-2--intern-management-role-admin)
  - [Interns CRUD](#1-interns-crud)
  - [Task Management](#2-task-management)
  - [Timesheet & Review Administration](#3-timesheet--review-administration)
  - [Leave Administration](#4-leave-administration)

---

## Enums & Constants

### `InternGuildStatus`
| Value | Meaning |
|---|---|
| `ACTIVE` | Intern is actively working |
| `AT_RISK` | Intern has missed submissions |
| `ON_LEAVE` | Intern is on approved leave |
| `INACTIVE` | Intern has been deactivated |

### `InternGuild`
| Value |
|---|
| `Frontend Guild` |
| `Backend Guild` |
| `Design Guild` |
| `Mobile Guild` |

### `InternTaskStatus`
| Value | Editable by Intern |
|---|---|
| `NOT_STARTED` | ❌ |
| `IN_PROGRESS` | ✅ |
| `COMPLETED` | ✅ (requires `output_link`) |
| `WAITING_FOR_REVIEW` | ✅ |
| `ON_HOLD` | ✅ |
| `OVERDUE` | ❌ (set by system) |

### `InternTaskComplexity`
| Value | Weight |
|---|---|
| `LOW` | 1 |
| `MEDIUM` | 2 |
| `HIGH` | 3 |
| `CRITICAL` | 5 |

### `InternTaskCategory`
| Guild | Sub-categories |
|---|---|
| `BACKEND` | `Backend API`, `Auth API`, `Bot`, `Database`, `DevOps`, `Documentation` |
| `FRONTEND` | `UI Components`, `API Integration`, `Bug Fix`, `Performance`, `Accessibility`, `Documentation` |
| `DESIGN` | `Wireframes`, `Prototyping`, `Branding`, `Research` |

### `InternSubmissionStatus` (Timesheets & Weekly Reviews)
| Value |
|---|
| `PENDING` |
| `APPROVED` |
| `REJECTED` |

### `InternLeaveType`
| Value |
|---|
| `SICK` |
| `CASUAL` |
| `EMERGENCY` |
| `WFH` |

### `InternLeaveStatus`
| Value |
|---|
| `PENDING` |
| `APPROVED` |
| `REJECTED` |
| `CANCELLED` |

### Leaderboard Score Formula

**Intern overview score:**
```
score = (total_intern_karma × 1)
      + (daily_streak × 50)
      + (weekly_streak × 200)
      + (completed_tasks × 30)
      + (complexity_score × 20)
```

**Leaderboard score (authoritative):**
```
score = (approved_timesheets × 25)
      + (approved_weekly_reviews × 50)
      + sum(task.karma_awarded × complexity_weight for verified tasks)
```

---

## Pagination Query Params

All list endpoints support:

| Param | Type | Default | Description |
|---|---|---|---|
| `page` | int | `1` | Page number |
| `page_size` | int | `10` | Items per page |
| `search` | string | — | Full-text search on searchable fields |
| `sort_by` | string | — | Field to sort by (see per-endpoint docs) |
| `order` | `asc`\|`desc` | `desc` | Sort direction |

**Pagination response shape:**
```json
{
  "pagination": {
    "count": 42,
    "totalPages": 5,
    "isNext": true,
    "isPrev": false,
    "nextPage": 2
  }
}
```

---

## Part 1 — Intern Dashboard (Role: Intern)

### 1. Overview

#### `GET /api/v1/dashboard/intern/overview/status/`
**Description:** Retrieve the intern's overview stats (karma, streaks, tasks, score).  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "guild": "Backend Guild",
    "status": "ACTIVE",
    "join_date": "2025-06-01",
    "total_intern_karma": 120,
    "daily_streak": 5,
    "weekly_streak": 2,
    "completed_tasks": 3,
    "complexity_score": 7,
    "score": 870,
    "total_interns": 14
  }
}
```

**Constraints:**
- User must have a `UserInternGuildLink` record. Returns 400 with `"Not an intern."` otherwise.
- Karma only counts logs tagged with `#intern-*` hashtag.

---

#### `GET /api/v1/dashboard/intern/overview/activity/`
**Description:** Paginated list of the intern's karma activity log (intern tasks only).  
**Required Role:** `Intern`  
**Sortable by:** `created_at`  
**Searchable:** `task__title`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "uuid-of-log",
        "task_title": "Build Authentication API",
        "karma": 50,
        "created_at": "2025-06-10T08:30:00Z"
      }
    ],
    "pagination": { "count": 8, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `GET /api/v1/dashboard/intern/overview/leaderboard/top/`
**Description:** Top 3 interns on the leaderboard (uses authoritative leaderboard formula).  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": [
    {
      "user_id": "uuid-1",
      "full_name": "Alice Johnson",
      "guild": "Frontend Guild",
      "score": 3200,
      "rank": 1
    },
    {
      "user_id": "uuid-2",
      "full_name": "Bob Kumar",
      "guild": "Backend Guild",
      "score": 2750,
      "rank": 2
    },
    {
      "user_id": "uuid-3",
      "full_name": "Clara Thomas",
      "guild": "Design Guild",
      "score": 2100,
      "rank": 3
    }
  ]
}
```

**Constraints:**
- Only interns with status `ACTIVE`, `AT_RISK`, or `ON_LEAVE` appear.

---

### 2. Tasks

#### `GET /api/v1/dashboard/intern/tasks/categories/`
**Description:** List all task categories and their sub-categories.  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "BACKEND": ["Backend API", "Auth API", "Bot", "Database", "DevOps", "Documentation"],
    "FRONTEND": ["UI Components", "API Integration", "Bug Fix", "Performance", "Accessibility", "Documentation"],
    "DESIGN": ["Wireframes", "Prototyping", "Branding", "Research"]
  }
}
```

---

#### `GET /api/v1/dashboard/intern/tasks/mine/`
**Description:** Paginated list of all tasks assigned to the current intern.  
**Required Role:** `Intern`  
**Searchable:** `title`, `status`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "task-uuid",
        "title": "Build Auth API endpoint",
        "description": "Create JWT login/logout API",
        "category": "Auth API",
        "complexity": "MEDIUM",
        "assigned_to": "user-uuid",
        "assigned_to_name": "Alice Johnson",
        "status": "IN_PROGRESS",
        "karma_awarded": 0,
        "output_link": null,
        "is_verified": false,
        "verified_by": null,
        "created_by": "admin-uuid",
        "created_by_name": "Admin Name",
        "created_at": "2025-06-01T09:00:00Z",
        "updated_at": "2025-06-05T11:00:00Z"
      }
    ],
    "pagination": { "count": 5, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `GET /api/v1/dashboard/intern/tasks/<task_id>/detail/`
**Description:** Retrieve a single task detail.  
**Required Role:** `Intern`

**Path Param:** `task_id` — UUID of the task

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "id": "task-uuid",
    "title": "Build Auth API endpoint",
    "description": "Create JWT login/logout API",
    "category": "Auth API",
    "complexity": "MEDIUM",
    "assigned_to": "user-uuid",
    "assigned_to_name": "Alice Johnson",
    "status": "IN_PROGRESS",
    "karma_awarded": 0,
    "output_link": null,
    "is_verified": false,
    "verified_by": null,
    "created_by": "admin-uuid",
    "created_by_name": "Admin Name",
    "created_at": "2025-06-01T09:00:00Z",
    "updated_at": "2025-06-05T11:00:00Z"
  }
}
```

**Failure `400`** — Task not found or not assigned to the requesting intern.

---

#### `PATCH /api/v1/dashboard/intern/tasks/<task_id>/`
**Alias:** `PATCH /api/v1/dashboard/intern/tasks/<task_id>/submit/` *(backward compat)*  
**Description:** Directly update a task's status and/or output_link.  
**Required Role:** `Intern`

**Path Param:** `task_id` — UUID of the task

**Request Body:**
```json
{
  "status": "COMPLETED",
  "output_link": "https://github.com/user/repo/pull/42"
}
```

| Field | Type | Required | Valid Values |
|---|---|---|---|
| `status` | string | Optional (one of both must be present) | `IN_PROGRESS`, `COMPLETED`, `ON_HOLD`, `WAITING_FOR_REVIEW` |
| `output_link` | string | Required when `status=COMPLETED` | Any URL string |

**Constraints:**
- At least one of `status` or `output_link` must be provided.
- Cannot modify a task that is already `is_verified = true`.
- Status `COMPLETED` requires that `output_link` is set (either in request or already on the task).
- Interns **cannot** set `NOT_STARTED` or `OVERDUE` (system-only statuses).
- All changes are logged to `SystemActionLog` with `INTERN_TASK_UPDATE` action type.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Task status updated successfully.",
  "response": {}
}
```

**Failure `400`:**
```json
{
  "statusCode": 6001,
  "message": "output_link is required when marking a task as COMPLETED.",
  "response": {}
}
```

---

### 3. Timesheets

#### `GET /api/v1/dashboard/intern/timesheets/prefill/`
**Description:** Retrieve tasks for the current ISO week to pre-populate the daily timesheet form.  
**Required Role:** `Intern`

**Response `200 OK` (active):**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "tasks": [
      {
        "task_id": "task-uuid",
        "title": "Build Auth API endpoint",
        "category": "Auth API",
        "deadline": "2025-06-13",
        "status": "IN_PROGRESS",
        "complexity": "MEDIUM",
        "output_link": null,
        "is_overdue": false
      }
    ],
    "on_leave": false
  }
}
```

**Response `200 OK` (on leave):**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "tasks": [],
    "on_leave": true
  }
}
```

**Constraints:**
- Excludes archived tasks and tasks that are `COMPLETED` + `is_verified`.
- Returns `400` if intern status is `INACTIVE`.

---

#### `POST /api/v1/dashboard/intern/timesheets/`
**Description:** Submit a new daily timesheet entry.  
**Required Role:** `Intern`

**Request Body:**
```json
{
  "entry_date": "2025-06-10",
  "description": "Worked on JWT auth endpoints and wrote unit tests.",
  "hours": 6.5,
  "blockers": "Encountered issues with token expiry handling.",
  "end_of_day_note": "Will continue testing tomorrow.",
  "edit_reason": null,
  "task": [
    {
      "task_id": "task-uuid-1",
      "status": "COMPLETED",
      "remark": "PR raised and merged."
    },
    {
      "task_id": "task-uuid-2",
      "status": "IN_PROGRESS",
      "remark": ""
    }
  ]
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `entry_date` | date (`YYYY-MM-DD`) | ✅ | Cannot be a future date. Backdates > 1 day require `edit_reason`. |
| `description` | string | ✅ | Summary of work done |
| `hours` | float | ✅ | Must be > 0 |
| `blockers` | string | ❌ | Optional |
| `end_of_day_note` | string | ❌ | Optional |
| `edit_reason` | string | Conditional | Required for late submissions (> yesterday) |
| `task` | array | ❌ | Task status updates to snapshot in this timesheet |
| `task[].task_id` | string (UUID) | ✅ if task array present | Must belong to the intern |
| `task[].status` | string | ✅ | Any `InternTaskStatus` value |
| `task[].remark` | string | ❌ | Optional note for this task update |

**Constraints:**
- Intern must be `ACTIVE` or `AT_RISK` (not `INACTIVE` or `ON_LEAVE`).
- `task_id` must be assigned to the requesting intern.
- Cannot update a `is_verified = true` task.
- Marking `status = COMPLETED` in a timesheet requires the task to already have `output_link` set (use the task submit endpoint first).
- Duplicate `task_id` in the same submission is rejected.
- Created with `status = PENDING` — requires admin approval.
- A `409 Conflict` is returned if a timesheet for that `entry_date` already exists.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Timesheet submitted successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/intern/timesheets/`
**Description:** Paginated list of the intern's timesheets.  
**Required Role:** `Intern`  
**Searchable:** `entry_date`, `status`  
**Sortable by:** `entry_date`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "timesheet-uuid",
        "entry_date": "2025-06-10",
        "task": [
          {
            "task_id": "task-uuid-1",
            "title": "Build Auth API endpoint",
            "status": "COMPLETED",
            "remark": "PR raised and merged."
          }
        ],
        "description": "Worked on JWT auth endpoints.",
        "hours": 6.5,
        "blockers": "None",
        "end_of_day_note": "Will continue tomorrow.",
        "edit_reason": null,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-10T18:00:00Z"
      }
    ],
    "pagination": { "count": 12, "totalPages": 2, "isNext": true, "isPrev": false, "nextPage": 2 }
  }
}
```

---

#### `GET /api/v1/dashboard/intern/timesheets/<timesheet_id>/`
**Description:** Retrieve a specific timesheet by ID.  
**Required Role:** `Intern`

**Path Param:** `timesheet_id` — UUID  
**Response:** Same shape as single item in list above.

---

#### `PATCH /api/v1/dashboard/intern/timesheets/<timesheet_id>/`
**Description:** Edit the `end_of_day_note` on a pending timesheet.  
**Required Role:** `Intern`

**Request Body:**
```json
{
  "end_of_day_note": "Updated note: completed the testing phase.",
  "edit_reason": "Forgot to include final test results."
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `edit_reason` | string | ✅ | Mandatory for all modifications |
| `end_of_day_note` | string | ✅ | Only editable field |

**Constraints:**
- Only works on timesheets with `status = PENDING`.
- `edit_reason` is mandatory.
- Changes are logged to `SystemActionLog` with `INTERN_TIMESHEET_EDIT` action type.

---

#### `GET /api/v1/dashboard/intern/timesheets/today/`
**Description:** Retrieve today's timesheet entry.  
**Required Role:** `Intern`

**Response:** Same shape as single timesheet. Returns `400` if none submitted today.

---

#### `GET /api/v1/dashboard/intern/timesheets/history/`
**Description:** Paginated timesheet history (alias of the list endpoint).  
**Required Role:** `Intern`  
Same response shape as the list endpoint.

---

#### `GET /api/v1/dashboard/intern/timesheets/summary/`
**Description:** Retrieve current timesheet streak stats.  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "current_streak": 7,
    "longest_streak": 14
  }
}
```

---

### 4. Weekly Reviews

#### `GET /api/v1/dashboard/intern/reviews/prefill/`
**Description:** Retrieve this week's task snapshot to pre-populate the weekly review form.  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "iso_year": 2025,
    "iso_week": 24,
    "week_start": "2025-06-09",
    "week_end": "2025-06-15",
    "tasks": [
      {
        "task_id": "task-uuid-1",
        "title": "Build Auth API endpoint",
        "category": "Auth API",
        "complexity": "MEDIUM",
        "deadline": "2025-06-13",
        "status": "COMPLETED",
        "output_link": "https://github.com/user/repo/pull/42"
      }
    ]
  }
}
```

---

#### `POST /api/v1/dashboard/intern/reviews/`
**Description:** Submit a new weekly review.  
**Required Role:** `Intern`

**Request Body:**
```json
{
  "is_on_leave": false,
  "hours_committed": 40,
  "weekly_review": "Completed the JWT auth module and started on role-based access control.",
  "blockers": "Waiting for design finalization on the dashboard page.",
  "leave_days": 0,
  "suggestions": "Would love a dedicated code review session.",
  "team": "Backend Guild",
  "week_start_date": "2025-06-09",
  "rating": 4,
  "next_week_plan": "Finish RBAC and write integration tests.",
  "challenges_faced": "Debugging token refresh logic took longer than expected.",
  "learnings": "Learned about JWT expiry best practices."
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `is_on_leave` | boolean | ❌ | Defaults to `false` |
| `hours_committed` | float | Conditional | Required when `is_on_leave = false`, must be > 0 |
| `weekly_review` | string | ❌ | Auto-generated from `learnings`, `challenges_faced`, `next_week_plan` if omitted |
| `blockers` | string | ❌ | Optional |
| `leave_days` | int | ❌ | Optional |
| `suggestions` | string | ❌ | Optional |
| `team` | string | ❌ | Auto-populated from guild link if omitted |
| `week_start_date` | date | ❌ | Defaults to current week |
| `rating` | int (1-5) | ❌ | Stored in `task_remarks.rating` |
| `next_week_plan` | string | ❌ | Stored in `task_remarks.next_week_plan` |
| `challenges_faced` | string | ❌ | Stored in `task_remarks.challenges_faced` |
| `learnings` | string | ❌ | Stored in `task_remarks.learnings` |

**Constraints:**
- Intern must be `ACTIVE` or `AT_RISK`.
- One review per ISO week — `409` if already submitted.
- `tasks_completed` is auto-populated from `InternTask` records with status `COMPLETED` or `WAITING_FOR_REVIEW` for the current week.
- A review submitted after Sunday of that week is marked `is_late = true`.
- Created with `status = PENDING`.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Weekly review submitted successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/intern/reviews/`
**Description:** Paginated list of all weekly reviews.  
**Required Role:** `Intern`  
**Searchable:** `iso_year`, `iso_week`, `status`  
**Sortable by:** `iso_year`, `iso_week`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "review-uuid",
        "iso_year": 2025,
        "iso_week": 24,
        "week_start_date": "2025-06-09",
        "week_end_date": "2025-06-15",
        "team": "Backend Guild",
        "is_on_leave": false,
        "tasks_assigned": {},
        "tasks_completed": [
          {
            "task_id": "task-uuid-1",
            "title": "Build Auth API endpoint",
            "category": "Auth API",
            "complexity": "MEDIUM",
            "deadline": "2025-06-13",
            "final_status": "COMPLETED",
            "output_link": "https://github.com/user/repo/pull/42"
          }
        ],
        "weekly_review": "Completed JWT module.",
        "task_remarks": {
          "rating": 4,
          "next_week_plan": "Finish RBAC.",
          "challenges_faced": "Token refresh logic.",
          "learnings": "JWT best practices."
        },
        "hours_committed": 40,
        "blockers": "Waiting for design.",
        "leave_days": 0,
        "suggestions": "Code review sessions.",
        "is_late": false,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-15T18:00:00Z"
      }
    ],
    "pagination": { "count": 5, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `GET /api/v1/dashboard/intern/reviews/<review_id>/`
**Description:** Retrieve a single weekly review by ID.  
**Required Role:** `Intern`  
**Response:** Same as single item in list above.

---

#### `PATCH /api/v1/dashboard/intern/reviews/<review_id>/`
**Description:** Edit a pending weekly review (current week only).  
**Required Role:** `Intern`

**Request Body:** Any subset of the POST fields (partial update).

**Constraints:**
- Review must be `status = PENDING`.
- Review must be for the **current ISO week** — past weeks cannot be edited.

---

#### `GET /api/v1/dashboard/intern/reviews/current/`
**Description:** Retrieve the current week's submitted review.  
**Required Role:** `Intern`  
Returns `400` with `"No review submitted for the current week."` if none exists.

---

#### `GET /api/v1/dashboard/intern/reviews/history/`
**Description:** Paginated review history (alias of list endpoint).  
**Required Role:** `Intern`

---

### 5. Leave

#### `POST /api/v1/dashboard/intern/leave/`
**Description:** Submit a new leave request.  
**Required Role:** `Intern`

**Request Body:**
```json
{
  "leave_type": "SICK",
  "start_date": "2025-06-18",
  "end_date": "2025-06-19",
  "reason": "Fever and doctor's appointment.",
  "duration_days": 2
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `leave_type` | string | ✅ | `SICK`, `CASUAL`, `EMERGENCY`, `WFH` |
| `start_date` | date | ✅ | `YYYY-MM-DD` |
| `end_date` | date | ✅ | Must be ≥ `start_date` |
| `reason` | string | ✅ | Reason for leave |
| `duration_days` | int | ❌ | Auto-calculated as `(end - start).days + 1` if not provided |

**Constraints:**
- `start_date` must be ≤ `end_date`.
- No overlapping `PENDING` or `APPROVED` leaves allowed for the same date range.
- Created with `status = PENDING`.
- Action is logged to `SystemActionLog` with `INTERN_LEAVE_REQUEST` action type.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Leave request submitted successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/intern/leave/`
**Description:** Paginated list of all leave requests.  
**Required Role:** `Intern`  
**Searchable:** `leave_type`, `status`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "leave-uuid",
        "leave_type": "SICK",
        "start_date": "2025-06-18",
        "end_date": "2025-06-19",
        "reason": "Fever and doctor's appointment.",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-17T09:00:00Z"
      }
    ],
    "pagination": { "count": 3, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `GET /api/v1/dashboard/intern/leave/<leave_id>/`
**Description:** Retrieve a specific leave request.  
**Required Role:** `Intern`  
**Response:** Same as single item in list above.

---

#### `PATCH /api/v1/dashboard/intern/leave/<leave_id>/`
**Description:** Cancel a pending leave request.  
**Required Role:** `Intern`

**Request Body:**
```json
{
  "action": "cancel"
}
```

**Constraints:**
- Leave must be in `PENDING` status.
- Only `"cancel"` is a valid action from the intern side.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Leave request cancelled.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/intern/leave/history/`
**Description:** Paginated leave history (alias of list endpoint).  
**Required Role:** `Intern`

---

#### `GET /api/v1/dashboard/intern/leave/balance/`
**Description:** Retrieve total approved leave days used per leave type.  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "SICK": { "used": 2 },
    "CASUAL": { "used": 1 },
    "WFH": { "used": 5 },
    "EMERGENCY": { "used": 0 }
  }
}
```

---

### 6. Leaderboard

#### `GET /api/v1/dashboard/intern/leaderboard/`
**Description:** Full paginated intern leaderboard with ranks.  
**Required Role:** `Intern` or `Admin`

**Query Params:** `page`, `page_size`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "user_id": "uuid-1",
        "full_name": "Alice Johnson",
        "guild": "Frontend Guild",
        "score": 3200,
        "rank": 1
      },
      {
        "user_id": "uuid-2",
        "full_name": "Bob Kumar",
        "guild": "Backend Guild",
        "score": 2750,
        "rank": 2
      }
    ],
    "pagination": {
      "count": 14,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

**Constraints:**
- Only `ACTIVE`, `AT_RISK`, and `ON_LEAVE` interns are included.
- Score = `(approved_timesheets × 25) + (approved_weekly_reviews × 50) + sum(verified_task.karma_awarded × complexity_weight)`.

---

#### `GET /api/v1/dashboard/intern/leaderboard/me/`
**Description:** Retrieve the current intern's rank and score.  
**Required Role:** `Intern`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "rank": 3,
    "score": 2100
  }
}
```

**Failure `400`:**
```json
{
  "statusCode": 6001,
  "message": "Not found in leaderboard.",
  "response": {}
}
```

---

### 7. Guilds

#### `GET /api/v1/dashboard/intern/guilds/`
**Description:** Retrieve all available guild values.  
**Required Role:** `Intern` or `Admin`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": ["Frontend Guild", "Backend Guild", "Design Guild", "Mobile Guild"]
}
```

---

---

## Part 2 — Intern Management (Role: Admin)

> **Base prefix:** `/api/v1/dashboard/manage-interns/`

---

### 1. Interns CRUD

#### `GET /api/v1/dashboard/manage-interns/interns/`
**Description:** Paginated list of all interns.  
**Required Role:** `Admin`  
**Searchable:** `user__full_name`, `guild`, `status`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "guild-link-uuid",
        "user": "user-uuid",
        "full_name": "Alice Johnson",
        "muid": "AliceJohnson@mulearn",
        "guild": "Backend Guild",
        "status": "ACTIVE",
        "created_at": "2025-06-01T09:00:00Z"
      }
    ],
    "pagination": { "count": 14, "totalPages": 2, "isNext": true, "isPrev": false, "nextPage": 2 }
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/interns/<intern_id>/`
**Description:** Retrieve a specific intern by their guild-link UUID.  
**Required Role:** `Admin`  
**Path Param:** `intern_id` — UUID of the `UserInternGuildLink` record.

**Response:** Same as a single item in the list above.

---

#### `POST /api/v1/dashboard/manage-interns/interns/`
**Description:** Onboard a new intern.  
**Required Role:** `Admin`

**Request Body (by muid):**
```json
{
  "mu_id": "AliceJohnson@mulearn",
  "guild": "Backend Guild",
  "status": "ACTIVE"
}
```

**Request Body (by user_id):**
```json
{
  "user_id": "user-uuid",
  "guild": "Backend Guild",
  "status": "ACTIVE"
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `mu_id` | string | Conditional | MuLearn ID. Required if `user_id` not provided. |
| `user_id` | string (UUID) | Conditional | Required if `mu_id` not provided. |
| `guild` | string | ✅ | One of the `InternGuild` values |
| `status` | string | ❌ | Defaults to `ACTIVE` |

**Constraints:**
- Either `mu_id` or `user_id` must be provided.
- User must exist in the system.
- User must not already be an intern.
- Automatically assigns the `Intern` role to the user via `UserRoleLink`.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Intern onboarded successfully.",
  "response": {}
}
```

---

#### `PATCH /api/v1/dashboard/manage-interns/interns/<intern_id>/`
**Description:** Update intern details (partial update — guild, status).  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "guild": "Frontend Guild",
  "status": "AT_RISK"
}
```

| Field | Type | Notes |
|---|---|---|
| `guild` | string | Any valid `InternGuild` value |
| `status` | string | Any valid `InternGuildStatus` value |

**Constraints:**
- Guild reassignments are logged to `SystemActionLog` with `INTERN_GUILD_REASSIGN` action type.
- Changing status to `INACTIVE` removes the `Intern` role from the user.
- Changing status from `INACTIVE` to any other re-assigns the `Intern` role.
- Transitioning out of `ON_LEAVE` clears `previous_status`.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Intern details updated successfully.",
  "response": {}
}
```

---

#### `DELETE /api/v1/dashboard/manage-interns/interns/<intern_id>/`
**Description:** Deactivate (soft-delete) an intern.  
**Required Role:** `Admin`

**Constraints:**
- Sets `status = INACTIVE` (no hard delete).
- Removes the `Intern` role from `UserRoleLink` in an atomic transaction.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Intern deactivated successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/manage-interns/status/`
**Description:** Retrieve intern count statistics by status.  
**Required Role:** `Admin`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "ACTIVE": 10,
    "AT_RISK": 2,
    "ON_LEAVE": 1,
    "INACTIVE": 3,
    "total_interns": 16
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/interns/export/`
**Description:** Export all interns as a CSV file.  
**Required Role:** `Admin`

**Response:** `Content-Type: text/csv` attachment (`interns.csv`)

```
Full Name,MuID,Guild,Status,Created At
Alice Johnson,AliceJohnson@mulearn,Backend Guild,ACTIVE,2025-06-01 09:00:00
Bob Kumar,BobKumar@mulearn,Frontend Guild,AT_RISK,2025-05-15 10:30:00
```

---

### 2. Task Management

#### `GET /api/v1/dashboard/manage-interns/tasks/`
**Description:** Paginated list of all intern tasks.  
**Required Role:** `Admin`  
**Searchable:** `title`, `status`, `category`, `assigned_to__full_name`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "task-uuid",
        "title": "Build Auth API endpoint",
        "description": "Create JWT login/logout API",
        "category": "Auth API",
        "complexity": "MEDIUM",
        "assigned_to": "user-uuid",
        "status": "IN_PROGRESS",
        "team": "Backend Guild",
        "deadline": "2025-06-13",
        "iso_week": 24,
        "created_at": "2025-06-01T09:00:00Z"
      }
    ],
    "pagination": { "count": 22, "totalPages": 3, "isNext": true, "isPrev": false, "nextPage": 2 }
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/tasks/<task_id>/`
**Description:** Retrieve a specific task.  
**Required Role:** `Admin`  
**Response:** Same as a single item in list above.

---

#### `POST /api/v1/dashboard/manage-interns/tasks/`
**Description:** Create a new intern task.  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "title": "Build Auth API endpoint",
  "description": "Create JWT login and logout REST API with refresh token support.",
  "category": "Auth API",
  "complexity": "MEDIUM",
  "assigned_to": "AliceJohnson@mulearn",
  "deadline": "2025-06-20",
  "team": "Backend Guild",
  "iso_week": 25
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `title` | string | ✅ | Task name |
| `description` | string | ✅ | Detailed task description |
| `category` | string | ✅ | Any valid `InternTaskCategory` sub-category |
| `complexity` | string | ✅ | `LOW`, `MEDIUM`, `HIGH`, `CRITICAL` |
| `assigned_to` | string | ✅ | Accepts `muid` (e.g. `AliceJohnson@mulearn`) or a UUID |
| `deadline` | date | ✅ | `YYYY-MM-DD` |
| `team` | string | ❌ | Guild for this task (write-only alias of `team` field) |
| `iso_week` | int | ❌ | Auto-derived from `deadline` if omitted |

**Constraints:**
- `assigned_to` resolves by `muid` first, then by UUID.
- `iso_week` is auto-calculated from `deadline.isocalendar()[1]` if not provided.
- Initial status defaults to `NOT_STARTED`.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Task created successfully.",
  "response": {}
}
```

---

#### `PATCH /api/v1/dashboard/manage-interns/tasks/<task_id>/`
**Description:** Partially update an intern task.  
**Required Role:** `Admin`

**Request Body (any subset of create fields):**
```json
{
  "complexity": "HIGH",
  "deadline": "2025-06-25",
  "status": "ON_HOLD"
}
```

**Constraints:**
- Cannot edit tasks that are `is_verified = true`.
- Changes to `title`, `description`, `category`, `complexity`, `assigned_to_id`, `status` are logged to `SystemActionLog`.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Task updated successfully.",
  "response": {}
}
```

---

#### `DELETE /api/v1/dashboard/manage-interns/tasks/<task_id>/`
**Description:** Hard-delete an intern task.  
**Required Role:** `Admin`

**Constraints:**
- Cannot delete verified tasks (`is_verified = true`). Revoke verification first.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Task deleted successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/manage-interns/tasks/by-intern/<muid>/`
**Description:** Retrieve all tasks assigned to a specific intern, identified by their muid.  
**Required Role:** `Admin`  
**Path Param:** `muid` — MuLearn ID of the intern  
**Searchable:** `title`, `status`, `category`  
**Sortable by:** `created_at`, `status`, `deadline`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [ /* same shape as task list */ ],
    "pagination": { "count": 5, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `POST /api/v1/dashboard/manage-interns/tasks/<task_id>/verify/`
**Description:** Verify a completed task and award karma to the intern.  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "karma_awarded": 50
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `karma_awarded` | int | ✅ | Must be a non-negative integer |

**Constraints:**
- Task must not already be `is_verified = true`.
- If `karma_awarded > 0` and a `TaskList` with hashtag `#intern-task-verified` exists, a `KarmaActivityLog` is created and the intern's `Wallet` balance is incremented atomically.
- Action is logged to `SystemActionLog` with `INTERN_TASK_UPDATE` action type.
- All DB operations are wrapped in an atomic transaction.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Task verified successfully.",
  "response": {}
}
```

**Failure `400`:**
```json
{
  "statusCode": 6001,
  "message": "Task is already verified.",
  "response": {}
}
```

---

### 3. Timesheet & Review Administration

#### `GET /api/v1/dashboard/manage-interns/reviews/timesheets/`
**Description:** List all intern timesheets (filterable by status).  
**Required Role:** `Admin`  
**Query Param:** `status` — optional filter (`PENDING`, `APPROVED`, `REJECTED`)  
**Searchable:** `user__full_name`, `status`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "timesheet-uuid",
        "user_id": "user-uuid",
        "user_name": "Alice Johnson",
        "muid": "AliceJohnson@mulearn",
        "entry_date": "2025-06-10",
        "task": [
          {
            "task_id": "task-uuid-1",
            "title": "Build Auth API endpoint",
            "status": "COMPLETED",
            "remark": "PR merged."
          }
        ],
        "description": "Worked on JWT auth endpoints.",
        "hours": 6.5,
        "blockers": "None",
        "end_of_day_note": "Will continue tomorrow.",
        "edit_reason": null,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-10T18:00:00Z"
      }
    ],
    "pagination": { "count": 30, "totalPages": 3, "isNext": true, "isPrev": false, "nextPage": 2 }
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/reviews/timesheets/<timesheet_id>/review/`
**Description:** Retrieve a single timesheet for review.  
**Required Role:** `Admin`  
**Response:** Same shape as single item above.

---

#### `PATCH /api/v1/dashboard/manage-interns/reviews/timesheets/<timesheet_id>/review/`
**Description:** Approve or reject a timesheet.  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "action": "approve",
  "review_note": "Good detailed log."
}
```

| Field | Type | Required | Notes |
|---|---|---|---|
| `action` | string | ✅ | `"approve"` or `"reject"` |
| `review_note` | string | ❌ | Optional feedback. Stored on record. |

**Constraints / Side effects on Approval:**
- Timesheet must be `status = PENDING`.
- On **approve**:
  - Updates `UserStreak` (`streak_type = intern_timesheet`) with consecutive-day logic (skips weekends: Fri→Mon counts as consecutive).
  - Awards karma via `KarmaActivityLog` based on streak multiplier:
    - streak ≥ 30: `2.0×` multiplier
    - streak ≥ 14: `1.5×` multiplier
    - streak ≥ 7: `1.2×` multiplier
    - otherwise: `1.0×`
  - Awards milestone streak bonuses at streaks 7, 14, 30, 60, 90 (separate karma logs).
  - Updates intern's `Wallet` balance atomically.
  - If intern status is `AT_RISK` → restored to `ACTIVE`.
- All wrapped in atomic transaction.

**Streak Milestones:**
| Streak Days | Bonus Karma | Hashtag |
|---|---|---|
| 7 | 7 | `#intern-streak-7` |
| 14 | 14 | `#intern-streak-14` |
| 30 | 30 | `#intern-streak-30` |
| 60 | 60 | `#intern-streak-60` |
| 90 | 90 | `#intern-streak-90` |

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Timesheet approved successfully.",
  "response": {}
}
```

---

#### `GET /api/v1/dashboard/manage-interns/reviews/`
**Description:** List all weekly reviews (filterable by status).  
**Required Role:** `Admin`  
**Query Param:** `status` — optional filter (`PENDING`, `APPROVED`, `REJECTED`)  
**Searchable:** `user__full_name`, `status`, `team`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "review-uuid",
        "user_id": "user-uuid",
        "user_name": "Alice Johnson",
        "muid": "AliceJohnson@mulearn",
        "iso_year": 2025,
        "iso_week": 24,
        "week_start_date": "2025-06-09",
        "week_end_date": "2025-06-15",
        "team": "Backend Guild",
        "is_on_leave": false,
        "tasks_assigned": {},
        "tasks_completed": [ /* same shape as intern-side review list */ ],
        "weekly_review": "Completed JWT module.",
        "task_remarks": {
          "rating": 4,
          "next_week_plan": "Finish RBAC.",
          "challenges_faced": "Token refresh logic.",
          "learnings": "JWT best practices."
        },
        "hours_committed": 40,
        "blockers": "Waiting for design.",
        "leave_days": 0,
        "suggestions": "Code review sessions.",
        "is_late": false,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-15T18:00:00Z"
      }
    ],
    "pagination": { "count": 12, "totalPages": 2, "isNext": true, "isPrev": false, "nextPage": 2 }
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/reviews/reviews/<review_id>/review/`
**Description:** Retrieve a single weekly review for review.  
**Required Role:** `Admin`  
**Response:** Same shape as single item above.

---

#### `PATCH /api/v1/dashboard/manage-interns/reviews/reviews/<review_id>/review/`
**Description:** Approve or reject a weekly review.  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "action": "approve",
  "review_note": "Excellent weekly summary."
}
```

| Field | Type | Required |
|---|---|---|
| `action` | string | ✅ (`"approve"` or `"reject"`) |
| `review_note` | string | ❌ |

**Constraints / Side effects on Approval:**
- Review must be `status = PENDING`.
- On **approve**:
  - Updates `UserStreak` (`streak_type = intern_weekly_review`).
  - Late reviews (`is_late = true`) reset `current_streak` to 0.
  - Awards karma via `KarmaActivityLog` using `#intern-weekly-review` hashtag.
  - Updates intern's `Wallet` balance atomically.
- All wrapped in atomic transaction.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Weekly review approved successfully.",
  "response": {}
}
```

---

### 4. Leave Administration

#### `GET /api/v1/dashboard/manage-interns/leave/`
**Description:** List all intern leave requests.  
**Required Role:** `Admin`  
**Searchable:** `user__full_name`, `leave_type`, `status`  
**Sortable by:** `created_at`, `status`

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Successful",
  "response": {
    "data": [
      {
        "id": "leave-uuid",
        "user_id": "user-uuid",
        "user_name": "Alice Johnson",
        "muid": "AliceJohnson@mulearn",
        "leave_type": "SICK",
        "start_date": "2025-06-18",
        "end_date": "2025-06-19",
        "reason": "Fever.",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2025-06-17T09:00:00Z"
      }
    ],
    "pagination": { "count": 6, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### `GET /api/v1/dashboard/manage-interns/leave/<leave_id>/`
**Description:** Retrieve a single leave request.  
**Required Role:** `Admin`  
**Response:** Same as single item above.

---

#### `PATCH /api/v1/dashboard/manage-interns/leave/<leave_id>/review/`
**Description:** Approve or reject an intern leave request.  
**Required Role:** `Admin`

**Request Body:**
```json
{
  "action": "approve",
  "review_note": "Approved. Get well soon."
}
```

| Field | Type | Required |
|---|---|---|
| `action` | string | ✅ (`"approve"` or `"reject"`) |
| `review_note` | string | ❌ |

**Constraints / Side effects on Approval:**
- Leave must be `status = PENDING`.
- On **approve**:
  - If today falls within `start_date`–`end_date`, the intern's `UserInternGuildLink.status` is set to `ON_LEAVE` and `previous_status` is preserved.
- Action is logged to `SystemActionLog` with `INTERN_LEAVE_REVIEW` action type.
- All DB changes wrapped in atomic transaction.

**Response `200 OK`:**
```json
{
  "statusCode": 6000,
  "message": "Leave request approved successfully.",
  "response": {}
}
```

---

## Standard Error Responses

| HTTP Status | `statusCode` | When |
|---|---|---|
| `200` | `6000` | Success |
| `400` | `6001` | Business logic failure, validation error |
| `401` | `6003` | Missing or invalid JWT token |
| `403` | `6004` | Role not authorized |
| `409` | `6001` | Duplicate entry (e.g., timesheet/review for same date/week) |
| `500` | `6002` | Unexpected server error |

**Standard failure body:**
```json
{
  "statusCode": 6001,
  "message": "Error description here.",
  "response": {}
}
```
