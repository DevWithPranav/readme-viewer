# 🎓 Intern Dashboard — Structured Requirements

> **POV:** Intern (authenticated user with the `Intern` role)
> **Backend context:** Django REST Framework · `mulearnbackend`

---

## Roles & Personas

| Role | Description |
|------|-------------|
| `Intern` | A registered user assigned to an intern team via `UserRoleLink` |
| `Intern Lead / Admin` | Manages intern teams, tasks, and leave approvals |

---

## Module 1 — Overview Page

### Purpose
Landing page for the intern after login. Provides a quick snapshot of their activity, points, and streaks.

### 1.1 Intern Profile Card

| Field | Source | Notes |
|-------|--------|-------|
| Full Name | `User.full_name` | |
| Profile Picture | `User.profile_pic` (filesystem) | Fallback to avatar initials |
| Intern Team | `UserRoleLink → Role.title` | e.g., Backend, Frontend |
| Intern ID / muid | `User.muid` | |
| Email | `User.email` | |
| Join Date | `User.created_at` | Displayed as "Interning since DD MMM YYYY" |
| Status | `UserRoleLink.is_active` | Active / Inactive badge |

### 1.2 Points & Karma Summary

| Widget | Description | Source |
|--------|-------------|--------|
| Total Karma Points | Cumulative karma earned | `Wallet.karma` |
| This Week's Karma | Karma earned in the current ISO week | Aggregate `KarmaActivityLog` |
| Karma Rank | Position on leaderboard | Computed |

### 1.3 Streak Trackers

| Streak | Description | Logic |
|--------|-------------|-------|
| Daily Timesheet Streak | Consecutive days the intern submitted a timesheet | Count consecutive days with a `InternDailyTimesheet` submission |
| Weekly Review Streak | Consecutive weeks the intern submitted a weekly review | Count consecutive ISO weeks with a `InternWeeklyReview` submission |

- Streaks reset to `0` if a submission is missed on that day/week.
- Display: flame icon + count + "day streak" / "week streak" label.

### 1.4 Activity Log Feed

A chronological list of recent actions performed by the intern.

| Field | Description |
|-------|-------------|
| Action Type | `TIMESHEET_SUBMITTED`, `WEEKLY_REVIEW_SUBMITTED`, `TASK_STATUS_UPDATED`, `LEAVE_REQUESTED`, `LEAVE_APPROVED` |
| Description | Human-readable summary (e.g., "Submitted daily timesheet for 15 May 2026") |
| Timestamp | ISO timestamp, displayed as relative time ("2 hours ago") |

**Business Rules:**
- Show the latest 20 log entries.
- Paginate beyond 20.

---

## Module 2 — Daily Timesheet Form

### Purpose
An intern submits a structured daily activity log once per calendar day.

### 2.1 Form Fields

#### Read-Only Fields (auto-populated, not editable by the intern)

| Field | Type | Description |
|-------|------|-------------|
| `date` | `Date` | Calendar date of the timesheet (today's date) |
| `submitted_at` | `DateTimeField` | Timestamp when the form was submitted |
| `updated_at` | `DateTimeField` | Last update timestamp |
| `updated_by` | `FK → User` | ID/name of whoever last edited this record |
| `intern_team` | `CharField` | Derived from the intern's active `UserRoleLink → Role.title` (e.g., Backend, Frontend) |
| `task_category` | `CharField` | Category under the intern's team. Pre-defined per team (see §2.2) |
| `task_title` | `CharField (max 150)` | Title of the main task worked on |
| `task_details` | `TextField` | Detailed description of the work done |
| `status` | `CharField` | Current task status (see §2.3) |

> **Note:** The above fields are set by the system or the intern lead at task creation time. The intern **cannot edit** these fields after submission — they are read-only in the UI.

#### Editable Fields (intern can fill / update)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `remark` | `TextField` | No | Intern's personal note / observation for the day |
| `end_of_day_note` | `TextField` | No | A summary of what was done by end of day |
| `edit_reason` | `CharField (max 300)` | Yes (if editing after submission) | Mandatory reason whenever the intern updates a previously submitted form |

### 2.2 Task Categories per Team

| Team | Available Categories |
|------|---------------------|
| Backend | Backend API, Auth API, Bot, Database, DevOps, Documentation |
| Frontend | UI Components, API Integration, Bug Fix, Performance, Accessibility, Documentation |
| Design | Wireframes, Prototyping, Branding, Research |
| (Extensible) | Admin-configurable |

### 2.3 Task Status Options

| Value | Label |
|-------|-------|
| `NOT_STARTED` | Not Yet Started |
| `IN_PROGRESS` | In Progress |
| `COMPLETED` | Completed |
| `WAITING_FOR_REVIEW` | Waiting for Review |

### 2.4 Business Rules & Constraints

| # | Rule |
|---|------|
| BR-1 | An intern can submit **exactly one** timesheet per calendar day. A second attempt returns a `409 Conflict` with the message "Timesheet already submitted for today." |
| BR-2 | After submission, only `remark` and `end_of_day_note` are editable; all other fields become read-only. |
| BR-3 | If the intern edits an already-submitted form, `edit_reason` becomes **mandatory**. The edit is logged in the activity log. |
| BR-4 | Timesheets cannot be submitted for future dates. |
| BR-5 | Timesheets for past dates (within 24 hours) can be submitted with `edit_reason` noting the delay. |
| BR-6 | Submission contributes +1 to the intern's daily streak. |

### 2.5 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/intern/timesheet/today/` | Get today's timesheet (if exists) |
| `POST` | `/api/v1/intern/timesheet/` | Submit today's timesheet |
| `PATCH` | `/api/v1/intern/timesheet/{id}/` | Edit remark / end-of-day note (with reason) |
| `GET` | `/api/v1/intern/timesheet/history/` | Paginated list of past timesheets |

---

## Module 3 — Weekly Review Form

### Purpose
An intern reflects on their week — listing assigned tasks, noting incomplete ones with reasons, and writing a broader review of their weekly activity.

### 3.1 Form Fields

#### Auto-Populated / Read-Only Fields

| Field | Type | Description |
|-------|------|-------------|
| `week_number` | `PositiveIntegerField` | ISO week number |
| `week_start_date` | `DateField` | Monday of the review week |
| `week_end_date` | `DateField` | Sunday of the review week |
| `submitted_at` | `DateTimeField` | When the form was submitted |
| `updated_at` | `DateTimeField` | Last updated timestamp |
| `updated_by` | `FK → User` | User who last modified |
| `intern_team` | `CharField` | Derived from `UserRoleLink` |
| `assigned_tasks` | `JSONField / ManyToMany` | List of tasks assigned to the intern for this week (pre-filled by admin) |

#### Intern-Filled Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `task_remarks` | `JSONField` | Conditional | Per-task remark — **required for every task that is NOT completed by the deadline** (keyed by `task_id`) |
| `weekly_review` | `TextField (max 2000)` | Yes | Intern's narrative: what they worked on, what they learnt, blockers faced |
| `suggestions` | `TextField (max 1000)` | No | Optional improvement suggestions for the team or process |

### 3.2 Task Completion Remark Logic

```
For each task in assigned_tasks:
  if task.status != COMPLETED and task.deadline <= week_end_date:
      → task_remarks[task_id] is REQUIRED
  else:
      → task_remarks[task_id] is OPTIONAL
```

### 3.3 Business Rules & Constraints

| # | Rule |
|---|------|
| BR-1 | An intern can submit **exactly one** weekly review per ISO week. Duplicate attempt returns `409 Conflict`. |
| BR-2 | The form becomes available on **Friday** of the current week and must be submitted by **Sunday 23:59**. |
| BR-3 | Overdue submissions (Monday onward) are flagged as `LATE` and still accepted; the streak is broken. |
| BR-4 | `task_remarks` is mandatory for all incomplete tasks (enforced server-side). |
| BR-5 | Submission contributes +1 to the intern's weekly streak. |

### 3.4 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/intern/weekly-review/current/` | Get current week's review (if exists) |
| `POST` | `/api/v1/intern/weekly-review/` | Submit weekly review |
| `PATCH` | `/api/v1/intern/weekly-review/{id}/` | Update before deadline |
| `GET` | `/api/v1/intern/weekly-review/history/` | Paginated past reviews |

---

## Module 4 — Intern Leaderboard

### Purpose
Ranks all active interns to drive healthy competition and surface high performers.

### 4.1 Leaderboard Card Fields

| Field | Description |
|-------|-------------|
| `rank` | Numeric rank (1st, 2nd, 3rd…) with medal icons for top 3 |
| `full_name` | `User.full_name` |
| `profile_picture` | `User.profile_pic` (with fallback avatar) |
| `intern_team` | `UserRoleLink → Role.title` |
| `total_score` | Computed leaderboard score (see §4.2) |
| `karma_points` | `Wallet.karma` |
| `daily_streak` | Current daily timesheet streak |
| `weekly_streak` | Current weekly review streak |

### 4.2 Ranking Algorithm

Score is a weighted composite:

| Component | Weight | Source |
|-----------|--------|--------|
| Total Karma Points | 40% | `Wallet.karma` |
| Daily Timesheet Streak | 20% | Consecutive daily submissions |
| Weekly Review Streak | 20% | Consecutive weekly submissions |
| Number of Completed Tasks | 10% | `InternTask.status == COMPLETED` |
| Task Complexity Score | 10% | Sum of `InternTask.complexity_weight` for completed tasks |

> **Complexity weights:** Low = 1, Medium = 2, High = 3, Critical = 5

**Tie-breaking:** Earlier join date (`User.created_at`) ranks higher.

### 4.3 Filters & Display

| Filter | Options |
|--------|---------|
| Team Filter | All, Backend, Frontend, Design, etc. |
| Time Period | All Time, This Week, This Month |

- Top 3 get podium styling (gold / silver / bronze).
- The logged-in intern's row is always highlighted regardless of rank.

### 4.4 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/intern/leaderboard/` | Ranked list with optional `?team=&period=` filters |
| `GET` | `/api/v1/intern/leaderboard/me/` | Logged-in intern's rank and score |

---

## Module 5 — Intern Management *(Admin / Lead POV)*

### Purpose
Admin-facing module to manage intern enrollment, task allocation, and health-check stats.

### 5.1 Summary Statistics Panel

| Stat | Description |
|------|-------------|
| Total Interns | All users with the Intern role |
| Active Interns | `UserRoleLink.is_active == True` |
| Inactive Interns | `UserRoleLink.is_active == False` |
| Interns per Team | Breakdown by `Role.title` |
| Tasks by Status | Count of tasks per `InternTask.status` |
| Average Karma | Mean `Wallet.karma` across all active interns |

### 5.2 Intern CRUD

#### Add Intern
- Search existing `User` by email / muid
- Assign to a team (`Role`)
- Set start date
- Auto-create `UserRoleLink` with `is_active = True`

#### View Intern
- Full profile (name, email, team, join date)
- Karma total, daily & weekly streaks
- Last timesheet submission date
- Last weekly review date

#### Edit Intern
- Change assigned team
- Toggle `is_active`

#### Remove Intern
- Soft-deactivate: set `UserRoleLink.is_active = False`
- Hard-delete: remove `UserRoleLink` (requires superadmin)

### 5.3 Intern Task CRUD

#### Create Task

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | `CharField (max 150)` | Yes | Task title |
| `description` | `TextField` | Yes | Full task details |
| `assigned_to` | `FK → User` | Yes | Intern the task is assigned to |
| `team` | `CharField` | Yes | Backend / Frontend / etc. |
| `category` | `CharField` | Yes | Task category (see §2.2) |
| `status` | `CharField` | Auto | Default: `NOT_STARTED` |
| `complexity` | `CharField` | Yes | Low / Medium / High / Critical |
| `deadline` | `DateField` | Yes | Expected completion date |
| `week_number` | `PositiveIntegerField` | Auto | ISO week of deadline |

#### Read / List Tasks
- Filter by: intern, team, status, week, complexity
- Sort by: deadline, created_at, status

#### Update Task
- Admin can update all fields
- Status transitions follow: `NOT_STARTED → IN_PROGRESS → COMPLETED / WAITING_FOR_REVIEW`

#### Delete Task
- Soft delete (mark as `archived`)

### 5.4 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/intern/management/stats/` | Summary statistics |
| `GET` | `/api/v1/intern/management/interns/` | List all interns |
| `POST` | `/api/v1/intern/management/interns/` | Add intern |
| `GET` | `/api/v1/intern/management/interns/{id}/` | Intern detail |
| `PATCH` | `/api/v1/intern/management/interns/{id}/` | Update intern |
| `DELETE` | `/api/v1/intern/management/interns/{id}/` | Deactivate intern |
| `GET` | `/api/v1/intern/management/tasks/` | List tasks |
| `POST` | `/api/v1/intern/management/tasks/` | Create task |
| `GET` | `/api/v1/intern/management/tasks/{id}/` | Task detail |
| `PATCH` | `/api/v1/intern/management/tasks/{id}/` | Update task |
| `DELETE` | `/api/v1/intern/management/tasks/{id}/` | Archive task |

---

## Module 6 — Intern Leave Management System

### Purpose
Allows interns to request leaves and leads/admins to approve or reject them.

### 6.1 Leave Request Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `leave_type` | `CharField` | Yes | Sick Leave, Casual Leave, Emergency Leave, WFH |
| `start_date` | `DateField` | Yes | First day of leave |
| `end_date` | `DateField` | Yes | Last day of leave (inclusive) |
| `duration_days` | `PositiveIntegerField` | Auto | Computed: `(end_date - start_date).days + 1` |
| `reason` | `TextField (max 500)` | Yes | Reason for the leave |
| `supporting_document` | `FileField` | No | Optional (e.g., medical certificate) |
| `status` | `CharField` | Auto | `PENDING`, `APPROVED`, `REJECTED`, `CANCELLED` |
| `applied_at` | `DateTimeField` | Auto | Submission timestamp |
| `reviewed_by` | `FK → User` | Auto | Lead/admin who acted on the request |
| `reviewed_at` | `DateTimeField` | Auto | When the review happened |
| `review_note` | `CharField (max 300)` | No | Optional note from reviewer |

### 6.2 Leave Types

| Type | Max Days / Period |
|------|-----------------|
| Sick Leave | 2 days / month |
| Casual Leave | 1 day / month |
| Emergency Leave | No cap (admin discretion) |
| Work From Home | 2 days / week |

### 6.3 Leave Status Flow

```
PENDING → APPROVED
        → REJECTED
PENDING → CANCELLED (by intern, before review)
```

### 6.4 Intern-Side Views

| View | Description |
|------|-------------|
| Apply Leave | Form to submit a new leave request |
| My Leaves | List of all past and pending requests with status badges |
| Cancel Leave | Can cancel a `PENDING` request |
| Leave Balance | Remaining leave quota per type for the current month |

### 6.5 Admin / Lead Views

| View | Description |
|------|-------------|
| Pending Requests | All unreviewed leave requests across teams |
| All Requests | Filterable by intern, team, status, date range |
| Approve / Reject | Action with optional `review_note` |

### 6.6 Business Rules & Constraints

| # | Rule |
|---|------|
| BR-1 | Leave requests must be applied **at least 1 working day in advance** (except Emergency Leave). |
| BR-2 | An intern cannot have two overlapping leave requests. |
| BR-3 | Approved leaves do **not** break the timesheet / weekly review streak for those days. |
| BR-4 | Leave on days with an already-submitted timesheet is flagged as a warning to the admin. |
| BR-5 | `duration_days` is auto-calculated excluding weekends. |
| BR-6 | Monthly leave balance is reset on the 1st of every month. |

### 6.7 API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/v1/intern/leave/` | Intern's leave history |
| `POST` | `/api/v1/intern/leave/` | Apply for leave |
| `PATCH` | `/api/v1/intern/leave/{id}/cancel/` | Cancel pending leave |
| `GET` | `/api/v1/intern/leave/balance/` | Remaining leave quota |
| `GET` | `/api/v1/intern/leave/admin/` | Admin: all leave requests |
| `PATCH` | `/api/v1/intern/leave/admin/{id}/review/` | Admin: approve / reject |


