# Company Mentor — API Reference

> **Base URL prefix** assumed throughout this document:
> - Company endpoints → `/api/v1/dashboard/company/`
> - Mentor endpoints → `/api/v1/dashboard/mentor/`
> - Calendar endpoints → `/api/v1/calendar/`
> - Leaderboard endpoints → `/api/v1/leaderboard/`
>
> All authenticated endpoints require `Authorization: Bearer <JWT>` header.

---

## Table of Contents

1. [Company Mentor — Nomination](#1-company-mentor--nomination)
2. [Admin — Mentor Approval](#2-admin--mentor-approval)
3. [Company Management (Extended)](#3-company-management-extended)
   - [Profile](#31-company-profile)
   - [Jobs](#32-company-jobs)
   - [Applications](#33-job-applications)
   - [MuLearner Directory](#34-mulearner-directory)
   - [Gig Analytics](#35-gig-analytics)
   - [Tasks](#36-task-management)
4. [Mentor Sessions (Extended)](#4-mentor-sessions-extended)
   - [Create Session](#41-create-session)
   - [Available Sessions](#42-available-sessions-for-learners)
5. [Calendar](#5-calendar)
   - [Company Session Calendar](#51-company-session-calendar)
   - [IG Mentor Session Calendar](#52-ig-mentor-session-calendar)
   - [Campus Mentor Session Calendar](#53-campus-mentor-session-calendar)
6. [Leaderboard](#6-leaderboard)
   - [IG Mentor Leaderboard](#61-ig-mentor-leaderboard)
   - [Campus Mentor Leaderboard](#62-campus-mentor-leaderboard)

---

## 1. Company Mentor — Nomination

> **Who can nominate?** Only the **verified company creator** (the user who registered the company).
> The nominated user must already be a member of the company's Organisation record (`UserOrganizationLink`).
> After nomination the record enters `PENDING` state until an admin approves via the existing mentor verify endpoint.

---

### `POST /dashboard/company/mentor/nominate/`

Nominate a platform user (identified by their `muid`) as a Company Mentor for your company.

**Auth:** JWT · Company Creator role required

**Request body**

```json
{
  "muid": "john-doe@mulearn",
  "reason": "Senior developer, deeply involved in company projects."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `muid` | string | ✅ | MuID of the user to nominate (e.g. `john-doe@mulearn`) |
| `reason` | string | ❌ | Optional reason / recommendation note |

**Validation rules**

- `muid` must resolve to an existing platform user
- The resolved user must have a `UserOrganizationLink` entry for this company's org
- No active (non-rejected) `COMPANY_MENTOR` nomination must already exist for that user + company

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "User nominated as Company Mentor. Pending admin approval.",
  "response": {
    "id": "a1b2c3d4-...",
    "user_id": "u-uuid-here",
    "user_name": "John Doe",
    "user_email": "john@example.com",
    "org_name": "Acme Corp",
    "mentor_tier": "COMPANY_MENTOR",
    "status": "PENDING",
    "reason": "Senior developer, deeply involved in company projects.",
    "verification_note": null,
    "verified_at": null
  }
}
```

**Error responses**

| HTTP | Scenario |
|---|---|
| `403` | Caller is not a verified company creator |
| `400` | `muid` not found on platform |
| `400` | User is not a member of this company's organisation |
| `400` | User already has an active nomination for this company |

---

### `GET /dashboard/company/mentor/list/`

List all Company Mentor nominations for the authenticated company.

**Auth:** JWT · Company Creator role required

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": [
    {
      "id": "a1b2c3d4-...",
      "user_id": "u-uuid-here",
      "user_name": "John Doe",
      "user_email": "john@example.com",
      "org_name": "Acme Corp",
      "mentor_tier": "COMPANY_MENTOR",
      "status": "PENDING",
      "reason": "Senior developer.",
      "verification_note": null,
      "verified_at": null
    },
    {
      "id": "e5f6g7h8-...",
      "user_id": "u-uuid-2",
      "user_name": "Jane Smith",
      "user_email": "jane@example.com",
      "org_name": "Acme Corp",
      "mentor_tier": "COMPANY_MENTOR",
      "status": "APPROVED",
      "reason": "Team lead.",
      "verification_note": "Verified after background check.",
      "verified_at": "2026-06-02T10:00:00Z"
    }
  ]
}
```

---

## 2. Admin — Mentor Approval

> **Existing endpoint extended.** The `PATCH /dashboard/mentor/verify/<mentor_id>/` endpoint already handles all mentor tiers. When an admin approves a `COMPANY_MENTOR`:
>
> 1. `UserMentor.status` → `APPROVED`
> 2. `UserRoleLink` (Mentor role) is created/updated for the user
> 3. **NEW**: A `UserOrganizationLink` is auto-created, linking the mentor user to the company's Organisation

### `PATCH /dashboard/mentor/verify/<mentor_id>/`

**Auth:** JWT · Admin role required

**Request body**

```json
{
  "status": "APPROVED",
  "verification_note": "Verified and approved as company mentor for Acme Corp."
}
```

| Field | Type | Values |
|---|---|---|
| `status` | string | `APPROVED` or `REJECTED` |
| `verification_note` | string | Optional note (required if rejecting) |

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Mentor status updated successfully."
}
```

**Side-effects on approval (COMPANY_MENTOR)**

| Side-effect | Table | Result |
|---|---|---|
| Mentor role granted | `user_role_link` | Role `Mentor` linked to user |
| Org link created | `user_organization_link` | `user → company_org`, `verified = true` |

---

## 3. Company Management (Extended)

> All endpoints in this section are now accessible to **both** the company creator **and** approved `COMPANY_MENTOR` users for that company.
> The underlying guard logic resolves the company automatically from the JWT token.

---

### 3.1 Company Profile

#### `GET /dashboard/company/profile/`

Retrieve the company profile.

**Auth:** JWT · Company Creator or approved Company Mentor

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "id": "cmp-uuid",
    "name": "Acme Corp",
    "slug": "acme-corp",
    "status": "verified",
    "logo": "https://cdn.example.com/logo.png",
    "description": "We build products.",
    "short_pitch": "The best product company.",
    "industry_sector": "Technology",
    "website_link": "https://acme.com",
    "email": "hr@acme.com",
    "location": "Bangalore",
    "company_size": "51-200",
    "linkedin_url": "https://linkedin.com/company/acme",
    "founded_year": 2015,
    "remote_policy": "Hybrid",
    "culture_text": "We value innovation.",
    "tech_stack": ["Django", "React", "PostgreSQL"],
    "perks": ["Health insurance", "Remote-friendly"],
    "testimonials": [],
    "gallery": []
  }
}
```

#### `PATCH /dashboard/company/profile/`

Update the company profile.

**Auth:** JWT · Company Creator or approved Company Mentor

**Request body** *(all fields optional)*

```json
{
  "description": "Updated description.",
  "short_pitch": "Redefining product development.",
  "remote_policy": "Remote-first",
  "tech_stack": ["Django", "React", "PostgreSQL", "Redis"]
}
```

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Company profile updated successfully."
}
```

---

### 3.2 Company Jobs

#### `POST /dashboard/company/jobs/`

Post a new job or gig.

**Auth:** JWT · Company Creator or approved Company Mentor

**Request body**

```json
{
  "title": "Backend Engineer",
  "description": "Build and maintain APIs.",
  "job_type": "Job",
  "location": "Bangalore",
  "mode": "Remote",
  "experience_min": 1,
  "experience_max": 3,
  "salary_min": 600000,
  "salary_max": 900000,
  "skills_required": ["Python", "Django"],
  "status": "Active"
}
```

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Job posted successfully.",
  "response": {
    "id": "job-uuid",
    "title": "Backend Engineer",
    "job_type": "Job",
    "location": "Bangalore",
    "status": "Active"
  }
}
```

---

#### `GET /dashboard/company/jobs/`

List all jobs for the authenticated company.

**Auth:** JWT · Company Creator or approved Company Mentor

**Query params**

| Param | Type | Description |
|---|---|---|
| `search` | string | Search by title / location / job_type |
| `sort_by` | string | `title` or `created_at` |
| `page` | int | Page number |
| `per_page` | int | Page size |

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "job-uuid",
        "title": "Backend Engineer",
        "job_type": "Job",
        "location": "Bangalore",
        "status": "Active",
        "created_at": "2026-06-01T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "next": null,
      "previous": null
    }
  }
}
```

#### `GET /dashboard/company/jobs/<job_id>/`

Retrieve details of a specific job.

**Auth:** JWT · Company Creator or approved Company Mentor

#### `PATCH /dashboard/company/jobs/<job_id>/`

Update a specific job.

**Auth:** JWT · Company Creator or approved Company Mentor

#### `DELETE /dashboard/company/jobs/<job_id>/`

Soft-delete a specific job.

**Auth:** JWT · Company Creator or approved Company Mentor

---

### 3.3 Job Applications

#### `GET /dashboard/company/jobs/<job_id>/applications/`

List all applicants for a specific job.

**Auth:** JWT · Company Creator or approved Company Mentor

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "app-uuid",
        "user_id": "u-uuid",
        "user_name": "Priya K",
        "user_email": "priya@example.com",
        "status": "Pending",
        "applied_at": "2026-06-02T08:30:00Z",
        "resume_url": "https://cdn.example.com/resume.pdf"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

#### `PATCH /dashboard/company/applications/<app_id>/status/`

Update the status of a job application.

**Auth:** JWT · Company Creator or approved Company Mentor

**Request body**

```json
{
  "status": "Shortlisted"
}
```

| Allowed `status` values |
|---|
| `Pending`, `In-Review`, `Shortlisted`, `Interview`, `Selected`, `Rejected` |

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Application status updated successfully.",
  "response": {
    "id": "app-uuid",
    "status": "Shortlisted"
  }
}
```

---

### 3.4 MuLearner Directory

#### `GET /dashboard/company/mulearners/`

Browse the directory of MuLearners. Supports rich filtering.

**Auth:** JWT · Company Creator or approved Company Mentor

**Query params**

| Param | Type | Description |
|---|---|---|
| `min_karma` | int | Minimum karma score |
| `max_karma` | int | Maximum karma score |
| `level` | int | Level order number |
| `college` | string | College name (partial match) |
| `department` | string | Department (partial match) |
| `graduation_year` | string | Graduation year |
| `ig` | string | Interest Group name (partial match) |
| `skill` | string | Skill ID |
| `achievement` | string | Achievement ID |
| `task` | string | Task ID |
| `search` | string | Search full_name / muid / email |

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "u-uuid",
        "full_name": "Arjun Nair",
        "muid": "arjun-nair@mulearn",
        "karma": 4200,
        "level": "Level 4",
        "college": "CUSAT",
        "igs": ["Python", "Web Dev"]
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

### 3.5 Gig Analytics

#### `GET /dashboard/company/analytics/gigs/`

Retrieve analytics for company gigs.

**Auth:** JWT · Company Creator or approved Company Mentor

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "total_gigs_posted": 12,
    "active_gigs": 5,
    "closed_gigs": 7,
    "average_hourly_rate": 850.00,
    "application_funnel": {
      "Total": 148,
      "Pending": 40,
      "In-Review": 30,
      "Shortlisted": 22,
      "Interview": 18,
      "Selected": 8,
      "Rejected": 30
    },
    "conversion_rate": "5.41%"
  }
}
```

---

### 3.6 Task Management

> Company Mentors can submit tasks on behalf of their company. Tasks start in `pending` state and go active only after admin approval.

#### `GET /dashboard/company/tasks/`

List all tasks submitted by the company.

**Auth:** JWT · Company Creator or approved Company Mentor

**Query params**

| Param | Description |
|---|---|
| `approval_status` | Filter: `pending` / `approved` / `rejected` |
| `search` | Search by hashtag, title, IG, type |

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "task-uuid",
        "hashtag": "#acme-onboarding-2026",
        "title": "Acme Onboarding Challenge",
        "karma": 200,
        "approval_status": "pending",
        "ig": "Python",
        "type": "Learning",
        "created_at": "2026-06-01T00:00:00Z"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

#### `POST /dashboard/company/tasks/`

Submit a new task for admin approval.

**Auth:** JWT · Company Creator or approved Company Mentor

**Request body**

```json
{
  "hashtag": "#acme-onboarding-2026",
  "title": "Acme Onboarding Challenge",
  "description": "Complete the onboarding modules provided by Acme Corp.",
  "karma": 200,
  "channel": "channel-uuid",
  "type": "type-uuid",
  "ig": "ig-uuid",
  "skill_ids": ["skill-uuid-1", "skill-uuid-2"]
}
```

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Task submitted for approval."
}
```

#### `GET /dashboard/company/tasks/<task_id>/`

Retrieve details of a specific company task.

**Auth:** JWT · Company Creator or approved Company Mentor

#### `PUT /dashboard/company/tasks/<task_id>/`

Edit a task (resets it to `pending` for re-approval).

**Auth:** JWT · Company Creator or approved Company Mentor

#### `DELETE /dashboard/company/tasks/<task_id>/`

Delete a task. Only `pending` tasks can be deleted.

**Auth:** JWT · Company Creator or approved Company Mentor

---

## 4. Mentor Sessions (Extended)

### 4.1 Create Session

#### `POST /dashboard/mentor/session/create/`

Create a new mentorship session. The endpoint **auto-detects** the mentor type:

- **Company Mentor** → `session_type = company_session`, `entity_id = company org UUID` (auto-resolved, no need to pass)
- **IG Mentor** → `session_type = ig_session`, `entity_id = ig` (caller must pass `ig` field)

**Auth:** JWT · Mentor role required

**Request body — IG Mentor**

```json
{
  "ig": "ig-uuid",
  "title": "Python Basics for Beginners",
  "description": "Covering core Python concepts.",
  "mode": "Online",
  "starts_at": "2026-06-10T10:00:00Z",
  "ends_at": "2026-06-10T11:30:00Z",
  "meeting_link": "https://meet.google.com/abc-defg-hij",
  "max_participants": 30
}
```

**Request body — Company Mentor**

```json
{
  "title": "Acme Developer Mentoring — Sprint 1",
  "description": "Weekly sync for onboarding batch.",
  "mode": "Online",
  "starts_at": "2026-06-11T15:00:00Z",
  "ends_at": "2026-06-11T16:00:00Z",
  "meeting_link": "https://meet.google.com/xyz-1234-abc",
  "max_participants": 20
}
```

> **Note:** Company Mentors do **not** pass `ig`. `entity_id` and `session_type` are automatically set to the company's Organisation.

**Success response `200`**

```json
{
  "statusCode": 200,
  "message": "Session created successfully and is pending approval.",
  "response": {
    "entity_id": "org-uuid-of-company",
    "session_type": "company_session",
    "title": "Acme Developer Mentoring — Sprint 1",
    "mode": "Online",
    "starts_at": "2026-06-11T15:00:00Z",
    "ends_at": "2026-06-11T16:00:00Z",
    "meeting_link": "https://meet.google.com/xyz-1234-abc",
    "max_participants": 20
  }
}
```

---

### 4.2 Available Sessions (for Learners)

#### `GET /dashboard/mentor/session/available/`

List all scheduled sessions available to the authenticated user.

Now returns **both**:
- IG sessions for Interest Groups the user belongs to
- **Company sessions** for company orgs the user is linked to

**Auth:** JWT (any authenticated user)

**Query params**

| Param | Description |
|---|---|
| `search` | Search by title or description |
| `sort_by` | `starts_at` or `created_at` |

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "sess-uuid-1",
        "entity_id": "ig-uuid",
        "entity_name": "Python",
        "session_type": "ig_session",
        "title": "Python Basics for Beginners",
        "mode": "Online",
        "starts_at": "2026-06-10T10:00:00Z",
        "ends_at": "2026-06-10T11:30:00Z",
        "status": "SCHEDULED",
        "created_by_name": "Mentor One",
        "max_participants": 30
      },
      {
        "id": "sess-uuid-2",
        "entity_id": "org-uuid-of-acme",
        "entity_name": "Acme Corp",
        "session_type": "company_session",
        "title": "Acme Developer Mentoring — Sprint 1",
        "mode": "Online",
        "starts_at": "2026-06-11T15:00:00Z",
        "ends_at": "2026-06-11T16:00:00Z",
        "status": "SCHEDULED",
        "created_by_name": "Jane Smith",
        "max_participants": 20
      }
    ],
    "pagination": { "count": 2, "next": null, "previous": null }
  }
}
```

---

## 5. Calendar

> All calendar endpoints return sessions grouped into `upcoming`, `ongoing`, and `completed` buckets.
> Use the `month` query param (format `YYYY-MM`) to filter by month.

---

### 5.1 Company Session Calendar

#### `GET /calendar/company/<company_org_id>/sessions/`

Calendar view of all mentorship sessions for a specific company org.

**Auth:** None (public)

**Path param:** `company_org_id` — UUID of the company's `Organisation` record

**Query params**

| Param | Type | Description |
|---|---|---|
| `month` | string | `YYYY-MM` (e.g. `2026-06`) |
| `status` | string | `SCHEDULED`, `COMPLETED`, or `CANCELLED` |

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "upcoming": [
      {
        "id": "sess-uuid",
        "title": "Acme Developer Mentoring — Sprint 1",
        "session_type": "company_session",
        "mode": "Online",
        "starts_at": "2026-06-11T15:00:00Z",
        "ends_at": "2026-06-11T16:00:00Z",
        "status": "SCHEDULED",
        "participants": []
      }
    ],
    "ongoing": [],
    "completed": [
      {
        "id": "sess-uuid-old",
        "title": "Acme Kick-off Session",
        "session_type": "company_session",
        "mode": "Offline",
        "starts_at": "2026-06-01T10:00:00Z",
        "ends_at": "2026-06-01T11:00:00Z",
        "status": "COMPLETED",
        "participants": [
          { "user_id": "u-uuid", "user_name": "Arjun Nair" }
        ]
      }
    ]
  }
}
```

---

### 5.2 IG Mentor Session Calendar

#### `GET /calendar/ig-mentor/<ig_id>/sessions/`

Calendar view of mentorship sessions for a specific Interest Group.

**Auth:** None (public)

**Query params:** `month` (YYYY-MM), `status`

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": {
    "upcoming": [
      {
        "id": "sess-uuid",
        "title": "Python Basics for Beginners",
        "session_type": "ig_session",
        "mode": "Online",
        "starts_at": "2026-06-10T10:00:00Z",
        "ends_at": "2026-06-10T11:30:00Z",
        "status": "SCHEDULED",
        "participants": []
      }
    ],
    "ongoing": [],
    "completed": []
  }
}
```

---

### 5.3 Campus Mentor Session Calendar

#### `GET /calendar/campus-mentor/<campus_id>/sessions/`

Calendar view of mentorship sessions for a specific campus.

**Auth:** None (public)

**Query params:** `month` (YYYY-MM), `status`

**Success response `200`** *(same shape as IG calendar above)*

---

## 6. Leaderboard

### 6.1 IG Mentor Leaderboard

#### `GET /leaderboard/ig-mentor/<ig_id>/`

Ranked list of IG Mentors for a specific Interest Group.

Ranked by:
1. Number of **completed sessions** in that IG (descending)
2. Total **karma** as tiebreaker (descending)

**Auth:** None (public)

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": [
    {
      "rank": 1,
      "mentor_id": "m-uuid",
      "user_id": "u-uuid",
      "full_name": "Rahul Menon",
      "muid": "rahul-menon@mulearn",
      "total_karma": 8500,
      "completed_sessions": 14,
      "ig_name": "Python"
    },
    {
      "rank": 2,
      "mentor_id": "m-uuid-2",
      "user_id": "u-uuid-2",
      "full_name": "Sneha Das",
      "muid": "sneha-das@mulearn",
      "total_karma": 7200,
      "completed_sessions": 10,
      "ig_name": "Python"
    }
  ]
}
```

---

### 6.2 Campus Mentor Leaderboard

#### `GET /leaderboard/campus-mentor/<campus_id>/`

Ranked list of Campus Mentors for a specific campus (college org).

Ranked by:
1. Number of **completed campus sessions** (descending)
2. Total **karma** as tiebreaker (descending)

**Auth:** None (public)

**Success response `200`**

```json
{
  "statusCode": 200,
  "response": [
    {
      "rank": 1,
      "mentor_id": "m-uuid",
      "user_id": "u-uuid",
      "full_name": "Divya Krishnan",
      "muid": "divya-krishnan@mulearn",
      "total_karma": 6400,
      "completed_sessions": 9,
      "campus_name": "CUSAT"
    },
    {
      "rank": 2,
      "mentor_id": "m-uuid-2",
      "user_id": "u-uuid-2",
      "full_name": "Arun Pillai",
      "muid": "arun-pillai@mulearn",
      "total_karma": 5100,
      "completed_sessions": 6,
      "campus_name": "CUSAT"
    }
  ]
}
```

---

## DB Migration Requirement

> [!IMPORTANT]
> Before deploying, run the following SQL on the database to add `company_session` to the `session_type` ENUM column (since the model is `managed = False`, Django migrations won't auto-generate this):

```sql
ALTER TABLE mentorship_session
  MODIFY COLUMN session_type
    ENUM('ig_session', 'campus_session', 'company_session')
    NOT NULL DEFAULT 'ig_session';
```

File: `migrations/alter_session_type_add_company.sql`

---

## Access Control Summary

| Role | Nomination | Profile | Jobs | Tasks | Sessions | Analytics |
|---|---|---|---|---|---|---|
| Company Creator | ✅ | ✅ | ✅ | ✅ | (as Mentor) | ✅ |
| Approved Company Mentor | ❌ (nominate) / ✅ (list) | ✅ | ✅ | ✅ | ✅ (create) | ✅ |
| IG Mentor | — | — | — | ✅ (own) | ✅ (IG only) | — |
| Campus Mentor | — | — | — | — | ✅ (campus) | — |
| Admin | ✅ (approve/reject) | — | — | ✅ (approve) | ✅ (verify) | — |
| Any Authenticated User | — | — | apply | — | join/view | — |
| Public (no auth) | — | ✅ (public) | ✅ (browse) | — | ✅ (calendar) | ✅ (leaderboard) |
