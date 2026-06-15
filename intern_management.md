# Intern Management API Reference

**Base URL:** `/api/v1/dashboard/manage-interns/`  
**Authentication:** All endpoints require a valid JWT token via the `Authorization: Bearer <token>` header.  
**Role Required:** `ADMIN` — all endpoints in this module are restricted to administrators only.

---

## Table of Contents

1. [Interns Management](#1-interns-management)
2. [Task Management](#2-task-management)
3. [Leave Review](#3-leave-review)
4. [Timesheet Review](#4-timesheet-review)
5. [Weekly Review](#5-weekly-review)
6. [Common Patterns](#6-common-patterns)

---

## 1. Interns Management

Base path: `/api/v1/dashboard/manage-interns/`

These endpoints let admins onboard, list, update, and offboard interns. Onboarding automatically assigns the `INTERN` role to the user; deactivation removes it.

---

### 1.1 `GET /manage-interns/interns/`

Retrieve a paginated list of all interns (all statuses).

**Request**
```http
GET /api/v1/dashboard/manage-interns/interns/?page=1&perPage=10
Authorization: Bearer <admin-token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `page` | int | Page number (default: `1`) |
| `perPage` | int | Items per page (default: `10`) |
| `search` | string | Search by `full_name`, `guild`, or `status` |
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
        "id": "guild-link-uuid-0001",
        "user": "user-uuid-intern-001",
        "full_name": "Arun Dev",
        "muid": "arundev@mulearn",
        "guild": "BACKEND",
        "status": "ACTIVE",
        "created_at": "2024-05-01T10:00:00Z"
      },
      {
        "id": "guild-link-uuid-0002",
        "user": "user-uuid-intern-002",
        "full_name": "Meera Pillai",
        "muid": "meerapillai@mulearn",
        "guild": "DESIGN",
        "status": "AT_RISK",
        "created_at": "2024-04-15T09:30:00Z"
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

### 1.2 `GET /manage-interns/interns/<intern_id>/`

Retrieve a single intern's guild link record by its ID (`guild-link-uuid`, not `user-uuid`).

**Request**
```http
GET /api/v1/dashboard/manage-interns/interns/guild-link-uuid-0001/
Authorization: Bearer <admin-token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "id": "guild-link-uuid-0001",
    "user": "user-uuid-intern-001",
    "full_name": "Arun Dev",
    "muid": "arundev@mulearn",
    "guild": "BACKEND",
    "status": "ACTIVE",
    "created_at": "2024-05-01T10:00:00Z"
  }
}
```

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Intern not found." }
```

---

### 1.3 `POST /manage-interns/interns/`

Onboard a new intern. Automatically assigns the `INTERN` role to the user. The user can be identified by either their `mu_id` or `user_id`.

**Request**
```http
POST /api/v1/dashboard/manage-interns/interns/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**By MuID:**
```json
{
  "mu_id": "arundev@mulearn",
  "guild": "BACKEND",
  "status": "ACTIVE"
}
```

**By user_id (UUID):**
```json
{
  "user_id": "user-uuid-intern-001",
  "guild": "DESIGN"
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `mu_id` | string | ✅ (or `user_id`) | User's MuID (e.g. `arundev@mulearn`) |
| `user_id` | UUID | ✅ (or `mu_id`) | User's UUID |
| `guild` | string | ✅ | Intern's assigned guild |
| `status` | string | ❌ | Defaults to `ACTIVE` |

**Guild values:** `BACKEND`, `FRONTEND`, `DESIGN`, `DEVOPS`, `DATA`, `QA`

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Intern onboarded successfully." }
```

**Error `400`** — user already an intern
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "user": ["This user is already onboarded as an intern."] }
}
```

**Error `400`** — invalid mu_id
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "mu_id": ["User with this mu_id not found."] }
}
```

---

### 1.4 `PATCH /manage-interns/interns/<intern_id>/`

Update an intern's guild or status. Changing status to `INACTIVE` removes the `INTERN` role; restoring to an active status re-adds it. Guild reassignments are logged to `SystemActionLog`.

**Request**
```http
PATCH /api/v1/dashboard/manage-interns/interns/guild-link-uuid-0001/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**Change guild:**
```json
{
  "guild": "FRONTEND"
}
```

**Change status:**
```json
{
  "status": "AT_RISK"
}
```

**Update both:**
```json
{
  "guild": "DATA",
  "status": "ACTIVE"
}
```

**Valid status values:** `ACTIVE`, `AT_RISK`, `ON_LEAVE`, `INACTIVE`

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Intern details updated successfully." }
```

---

### 1.5 `DELETE /manage-interns/interns/<intern_id>/`

Deactivate (soft-delete) an intern. Sets their status to `INACTIVE` and removes their `INTERN` role from the platform. This action is **reversible** via PATCH.

**Request**
```http
DELETE /api/v1/dashboard/manage-interns/interns/guild-link-uuid-0001/
Authorization: Bearer <admin-token>
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Intern deactivated successfully." }
```

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Intern not found." }
```

> ⚠️ **Note:** This is a soft-delete. The `UserInternGuildLink` record is not removed from the database; status is set to `INACTIVE` and the `INTERN` role is revoked.

---

### 1.6 `GET /manage-interns/status/`

Retrieve a count of interns grouped by status. Useful for the admin dashboard overview card.

**Request**
```http
GET /api/v1/dashboard/manage-interns/status/
Authorization: Bearer <admin-token>
```

**Response `200 OK`**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successful",
  "response": {
    "ACTIVE": 9,
    "AT_RISK": 2,
    "ON_LEAVE": 1,
    "INACTIVE": 3
  }
}
```

---

### 1.7 `GET /manage-interns/interns/export/`

Export the full intern list as a CSV file download.

**Request**
```http
GET /api/v1/dashboard/manage-interns/interns/export/
Authorization: Bearer <admin-token>
```

**Response `200 OK`**

Returns a `text/csv` file attachment (`Content-Disposition: attachment; filename="interns.csv"`).

**CSV columns:**
```
Full Name, MuID, Guild, Status, Created At
```

**Example CSV content:**
```csv
Full Name,MuID,Guild,Status,Created At
Arun Dev,arundev@mulearn,BACKEND,ACTIVE,2024-05-01 10:00:00
Meera Pillai,meerapillai@mulearn,DESIGN,AT_RISK,2024-04-15 09:30:00
```

---

## 2. Task Management

Base path: `/api/v1/dashboard/manage-interns/tasks/`

Admins create, update, verify and delete tasks that are assigned to interns.

---

### 2.1 `GET /tasks/`

Retrieve a paginated list of all intern tasks.

**Request**
```http
GET /api/v1/dashboard/manage-interns/tasks/?page=1&perPage=10
Authorization: Bearer <admin-token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search by `title`, `status`, `category`, or `assigned_to__full_name` |
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
        "description": "Create CRUD endpoints for intern leave requests with validation",
        "category": "DEVELOPMENT",
        "complexity": "HIGH",
        "assigned_to": "user-uuid-intern-001",
        "status": "IN_PROGRESS",
        "karma_awarded": 0,
        "output_link": null,
        "is_verified": false,
        "verified_by": null,
        "team": "BACKEND",
        "deadline": "2024-06-15",
        "iso_week": 24,
        "created_at": "2024-05-28T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 18,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

### 2.2 `GET /tasks/<task_id>/`

Retrieve a single task by ID.

**Request**
```http
GET /api/v1/dashboard/manage-interns/tasks/task-uuid-0099/
Authorization: Bearer <admin-token>
```

**Response `200 OK`** — same shape as a single item in the list above.

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Task not found." }
```

---

### 2.3 `POST /tasks/`

Create and assign a new task to an intern.

**Request**
```http
POST /api/v1/dashboard/manage-interns/tasks/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**By intern's MuID:**
```json
{
  "title": "Build leave management API",
  "description": "Create CRUD endpoints for intern leave requests with validation",
  "category": "DEVELOPMENT",
  "complexity": "HIGH",
  "assigned_to": "arundev@mulearn",
  "team": "BACKEND",
  "deadline": "2024-06-15"
}
```

**By intern's user UUID:**
```json
{
  "title": "Design intern dashboard UI",
  "description": "Create Figma mockups for the intern overview page",
  "category": "DESIGN",
  "complexity": "MEDIUM",
  "assigned_to": "user-uuid-intern-002",
  "team": "DESIGN",
  "deadline": "2024-06-20",
  "iso_week": 25
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | ✅ | Task title |
| `description` | string | ✅ | Detailed task description |
| `category` | string | ✅ | Work category (e.g. `DEVELOPMENT`, `DESIGN`, `TESTING`) |
| `complexity` | string | ✅ | `LOW`, `MEDIUM`, `HIGH`, or `CRITICAL` |
| `assigned_to` | string/UUID | ✅ | Intern's MuID **or** user UUID |
| `team` | string | ❌ | Guild/team name (alias: `guild`) |
| `deadline` | date (`YYYY-MM-DD`) | ❌ | Task deadline; also used to auto-calculate `iso_week` |
| `iso_week` | int | ❌ | ISO week number; auto-calculated from `deadline` if omitted |
| `status` | string | ❌ | Defaults to `TODO` |

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Task created successfully." }
```

---

### 2.4 `PATCH /tasks/<task_id>/`

Update any field on a task. Changes are logged to `SystemActionLog`.

**Request**
```http
PATCH /api/v1/dashboard/manage-interns/tasks/task-uuid-0099/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

```json
{
  "complexity": "CRITICAL",
  "deadline": "2024-06-10",
  "status": "IN_PROGRESS"
}
```

**Reassign to a different intern:**
```json
{
  "assigned_to": "rahulkrishnan@mulearn"
}
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Task updated successfully." }
```

---

### 2.5 `POST /tasks/<task_id>/verify/`

Verify a task completed by an intern and optionally award custom karma. A task cannot be modified by an intern once it has been verified.

**Request**
```http
POST /api/v1/dashboard/manage-interns/tasks/task-uuid-0099/verify/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

```json
{
  "karma_awarded": 150
}
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Task verified successfully." }
```

**Error `400`** — Already verified
```json
{ "hasError": true, "statusCode": 400, "message": "Task is already verified." }
```

---

### 2.6 `DELETE /tasks/<task_id>/`

Permanently delete a task.

**Request**
```http
DELETE /api/v1/dashboard/manage-interns/tasks/task-uuid-0099/
Authorization: Bearer <admin-token>
```

**Response `200 OK`**
```json
{ "hasError": false, "statusCode": 200, "message": "Task deleted successfully." }
```

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Task not found." }
```

---

## 3. Leave Review

Base path: `/api/v1/dashboard/manage-interns/leave/`

Admins review pending leave requests submitted by interns. Approving a leave that falls within today's date automatically sets the intern's guild status to `ON_LEAVE`.

---

### 3.1 `GET /leave/`

Retrieve a paginated list of all intern leave requests across all interns.

**Request**
```http
GET /api/v1/dashboard/manage-interns/leave/?page=1&perPage=10
Authorization: Bearer <admin-token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `search` | string | Search by `user__full_name`, `leave_type`, or `status` |
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
        "id": "leave-uuid-0001",
        "user": "user-uuid-intern-001",
        "user_name": "Arun Dev",
        "leave_type": "SICK",
        "start_date": "2024-06-10",
        "end_date": "2024-06-11",
        "reason": "High fever, doctor advised rest",
        "status": "PENDING",
        "review_note": null,
        "created_at": "2024-06-09T08:00:00Z"
      },
      {
        "id": "leave-uuid-0002",
        "user": "user-uuid-intern-002",
        "user_name": "Meera Pillai",
        "leave_type": "WFH",
        "start_date": "2024-06-12",
        "end_date": "2024-06-12",
        "reason": "Power outage in area",
        "status": "APPROVED",
        "review_note": "Approved.",
        "created_at": "2024-06-11T07:00:00Z"
      }
    ],
    "pagination": {
      "count": 7,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

### 3.2 `PATCH /leave/<leave_id>/review/`

Approve or reject a **pending** leave request.

- **Approve:** Sets leave status to `APPROVED`. If today falls within the leave date range, the intern's guild status is changed to `ON_LEAVE`.
- **Reject:** Sets leave status to `REJECTED` with the provided review note.

All actions are logged to `SystemActionLog`.

**Request**
```http
PATCH /api/v1/dashboard/manage-interns/leave/leave-uuid-0001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**Approve:**
```json
{
  "action": "approve",
  "review_note": "Approved. Feel better soon!"
}
```

**Reject:**
```json
{
  "action": "reject",
  "review_note": "Insufficient leave balance for this month."
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | ✅ | Must be `"approve"` or `"reject"` |
| `review_note` | string | ❌ | Admin note/comment for the intern |

**Response `200 OK`** — approve
```json
{ "hasError": false, "statusCode": 200, "message": "Leave request approved successfully." }
```

**Response `200 OK`** — reject
```json
{ "hasError": false, "statusCode": 200, "message": "Leave request rejected successfully." }
```

**Error `400`** — invalid action
```json
{ "hasError": true, "statusCode": 400, "message": "Invalid action. Must be 'approve' or 'reject'." }
```

**Error `400`** — leave not found or not pending
```json
{ "hasError": true, "statusCode": 400, "message": "Pending leave request not found." }
```

---

## 4. Timesheet Review

Base path: `/api/v1/dashboard/manage-interns/reviews/`

Admins review and approve/reject daily timesheet submissions. On approval, karma is calculated and awarded with streak bonuses applied. Streak milestones trigger additional karma rewards.

---

### 4.1 `GET /reviews/timesheets/`

Retrieve a paginated list of all intern timesheet submissions (filterable by status).

**Request**
```http
GET /api/v1/dashboard/manage-interns/reviews/timesheets/?status=PENDING&page=1&perPage=10
Authorization: Bearer <admin-token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by `PENDING`, `APPROVED`, or `REJECTED` |
| `search` | string | Search by `user__full_name`, `status`, or `category` |
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
        "id": "ts-uuid-0001",
        "user_id": "user-uuid-intern-001",
        "user_name": "Arun Dev",
        "muid": "arundev@mulearn",
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
      "count": 28,
      "totalPages": 3,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

### 4.2 `GET /reviews/timesheets/<timesheet_id>/review/`

Retrieve a single timesheet submission for review.

**Request**
```http
GET /api/v1/dashboard/manage-interns/reviews/timesheets/ts-uuid-0001/review/
Authorization: Bearer <admin-token>
```

**Response `200 OK`** — same shape as a single item in the list above.

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Timesheet not found." }
```

---

### 4.3 `PATCH /reviews/timesheets/<timesheet_id>/review/`

Approve or reject a **pending** timesheet submission.

**On approval, the system:**
1. Calculates base karma from `InternHashtag.DAILY_LOG_KARMA`
2. Applies a streak multiplier based on consecutive submission days
3. Creates a `KarmaActivityLog` entry and updates the intern's `Wallet`
4. Awards bonus karma on streak milestones (7, 14, 30, 60, 90 days)
5. If the intern was `AT_RISK`, promotes them back to `ACTIVE`

**Streak Multipliers:**

| Streak Length | Karma Multiplier |
|--------------|-----------------|
| < 7 days | 1.0× (base) |
| ≥ 7 days | 1.2× |
| ≥ 14 days | 1.5× |
| ≥ 30 days | 2.0× |

**Streak Milestones (bonus karma):** 7, 14, 30, 60, 90 consecutive days.

**Request**
```http
PATCH /api/v1/dashboard/manage-interns/reviews/timesheets/ts-uuid-0001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**Approve:**
```json
{
  "action": "approve"
}
```

**Reject:**
```json
{
  "action": "reject",
  "review_note": "Description is too vague. Please provide more detail about your work."
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | ✅ | `"approve"` or `"reject"` |
| `review_note` | string | ❌ | Feedback note visible to the intern |

**Response `200 OK`** — approve
```json
{ "hasError": false, "statusCode": 200, "message": "Timesheet approved successfully." }
```

**Response `200 OK`** — reject
```json
{ "hasError": false, "statusCode": 200, "message": "Timesheet rejected successfully." }
```

**Error `400`** — not in PENDING state or not found
```json
{ "hasError": true, "statusCode": 400, "message": "Pending timesheet not found." }
```

**Error `400`** — invalid action
```json
{ "hasError": true, "statusCode": 400, "message": "Invalid action. Must be 'approve' or 'reject'." }
```

---

## 5. Weekly Review

Base path: `/api/v1/dashboard/manage-interns/reviews/`

Admins review and approve/reject weekly reviews. On approval, karma is awarded and the intern's weekly review streak is updated.

---

### 5.1 `GET /reviews/`

Retrieve a paginated list of all intern weekly review submissions (filterable by status).

**Request**
```http
GET /api/v1/dashboard/manage-interns/reviews/?status=PENDING&page=1&perPage=10
Authorization: Bearer <admin-token>
```

**Query Parameters**

| Parameter | Type | Description |
|-----------|------|-------------|
| `status` | string | Filter by `PENDING`, `APPROVED`, or `REJECTED` |
| `search` | string | Search by `user__full_name`, `status`, or `team` |
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
        "id": "rev-uuid-0001",
        "user_id": "user-uuid-intern-001",
        "user_name": "Arun Dev",
        "muid": "arundev@mulearn",
        "iso_year": 2024,
        "iso_week": 23,
        "week_start_date": "2024-06-03",
        "week_end_date": "2024-06-09",
        "team": "BACKEND",
        "is_on_leave": false,
        "tasks_assigned": {
          "task-uuid-0099": "COMPLETED"
        },
        "tasks_completed": "Leave API complete, tests written",
        "weekly_review": "Learnings: Better ORM usage. Challenges: Date timezone handling.",
        "task_remarks": {
          "rating": 4,
          "next_week_plan": "Start timesheet module",
          "challenges_faced": "Date timezone handling",
          "learnings": "Better ORM usage"
        },
        "hours_committed": 35,
        "blockers": "None",
        "leave_days": 0,
        "suggestions": null,
        "is_late": false,
        "status": "PENDING",
        "review_note": null,
        "created_at": "2024-06-07T17:00:00Z"
      }
    ],
    "pagination": {
      "count": 11,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

### 5.2 `GET /reviews/reviews/<review_id>/review/`

Retrieve a single weekly review for admin inspection.

**Request**
```http
GET /api/v1/dashboard/manage-interns/reviews/reviews/rev-uuid-0001/review/
Authorization: Bearer <admin-token>
```

**Response `200 OK`** — same shape as a single item in the list above.

**Error `400`**
```json
{ "hasError": true, "statusCode": 400, "message": "Weekly review not found." }
```

---

### 5.3 `PATCH /reviews/reviews/<review_id>/review/`

Approve or reject a **pending** weekly review.

**On approval, the system:**
1. Awards `InternHashtag.WEEKLY_REVIEW_KARMA` to the intern
2. Creates a `KarmaActivityLog` entry and updates their `Wallet`
3. Updates the weekly review streak:
   - Late submissions reset the streak to `0`
   - Consecutive weeks increment the streak by `1`
   - Missed weeks reset the streak to `1`

**Request**
```http
PATCH /api/v1/dashboard/manage-interns/reviews/reviews/rev-uuid-0001/review/
Authorization: Bearer <admin-token>
Content-Type: application/json
```

**Approve:**
```json
{
  "action": "approve",
  "review_note": "Great week! Keep up the consistency."
}
```

**Reject:**
```json
{
  "action": "reject",
  "review_note": "Please provide more detail on your learnings. Resubmit by EOD Friday."
}
```

**Fields**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `action` | string | ✅ | `"approve"` or `"reject"` |
| `review_note` | string | ❌ | Admin feedback visible to the intern |

**Response `200 OK`** — approve
```json
{ "hasError": false, "statusCode": 200, "message": "Weekly review approved successfully." }
```

**Response `200 OK`** — reject
```json
{ "hasError": false, "statusCode": 200, "message": "Weekly review rejected successfully." }
```

**Error `400`** — not pending or not found
```json
{ "hasError": true, "statusCode": 400, "message": "Pending weekly review not found." }
```

---

## 6. Common Patterns

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
| `search` | Full-text search string |
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

### Intern Status Values

| Status | Description |
|--------|-------------|
| `ACTIVE` | Healthy, participating intern |
| `AT_RISK` | Intern with missed submissions; auto-managed by daily cron |
| `ON_LEAVE` | Currently on approved leave |
| `INACTIVE` | Intern is offboarded/deactivated |
