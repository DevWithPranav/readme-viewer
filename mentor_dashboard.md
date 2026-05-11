# Mentor Dashboard — API Reference

> **Base URL:** `{{host}}/api/v1/dashboard/mentor`  
> **Auth:** `Authorization: Bearer <JWT>` on all endpoints  
> **Content-Type:** `application/json`

---

## API Index

| # | Method | Endpoint | Auth | Description |
|---|---|---|---|---|
| 1 | `GET` | `/persona/ig-roles/` | JWT | List all IG-scoped mentor roles for the user |
| 2 | `POST` | `/persona/switch/` | JWT | Switch active persona to mentor for an IG |
| 3 | `POST` | `/persona/reset/` | JWT | Reset persona back to learner |
| 4 | `GET` | `/profile/` | MENTOR_PERSONA | Fetch own mentor profile |
| 5 | `PATCH` | `/profile/` | MENTOR_PERSONA | Update about / reason / expertise |
| 6 | `GET` | `/overview/` | MENTOR_PERSONA | Dashboard snapshot with stats |
| 7 | `GET` | `/availability/` | MENTOR_PERSONA | List availability slots |
| 8 | `POST` | `/availability/` | MENTOR_PERSONA | Create availability slots |
| 9 | `DELETE` | `/availability/<slot_id>/` | MENTOR_PERSONA | Soft-delete an availability slot |
| 10 | `GET` | `/sessions/` | MENTOR_PERSONA | List sessions (filterable) |
| 11 | `POST` | `/sessions/` | VERIFIED_MENTOR | Create a new session |
| 12 | `PATCH` | `/sessions/<session_id>/` | MENTOR_PERSONA | Update session status / attendance |
| 13 | `GET` | `/mentees/` | MENTOR_PERSONA | List mentees with karma and progress |
| 14 | `GET` | `/tasks/queue/` | VERIFIED_MENTOR | List pending mentee task submissions |
| 15 | `PATCH` | `/tasks/queue/<log_id>/` | VERIFIED_MENTOR | Approve or reject a mentee task |
| 16 | `GET` | `/tasks/requests/` | MENTOR_PERSONA | List own task creation requests |
| 17 | `POST` | `/tasks/requests/` | MENTOR_PERSONA | Submit a task creation request to admin |
| 18 | `GET` | `/admin/task-requests/` | ADMIN | View all mentor task creation requests |
| 19 | `PATCH` | `/admin/task-requests/<req_id>/` | ADMIN | Approve / reject a task creation request |
| 20 | `PATCH` | `/admin/tier/<mentor_profile_id>/` | ADMIN | Change mentor tier (auto-verifies on VERIFIED) |

---

## Auth Levels

| Label | Requirement |
|---|---|
| `JWT` | Any valid token |
| `MENTOR_PERSONA` | JWT + active mentor persona (`POST /persona/switch/` called first) |
| `VERIFIED_MENTOR` | JWT + active mentor persona + `is_verified = true` |
| `ADMIN` | JWT with Admin role |

---

## 1. Persona

### GET `/persona/ig-roles/`
**Auth:** JWT  
**Response:**
```json
{
  "response": {
    "ig_roles": [
      {
        "role_link_id": "rl-uuid",
        "ig_id": "ig-uuid",
        "ig_name": "Entrepreneurship",
        "role": "Mentor",
        "is_verified": false,
        "mentor_tier": "NORMAL"
      }
    ]
  }
}
```

---

### POST `/persona/switch/`
**Auth:** JWT  
**Request:**
```json
{ "active_role_link_id": "rl-uuid" }
```
**Response:**
```json
{
  "response": {
    "active_persona": "mentor",
    "active_role_link_id": "rl-uuid",
    "active_ig_id": "ig-uuid",
    "ig_name": "Entrepreneurship",
    "is_verified": false,
    "mentor_tier": "NORMAL",
    "profile_created": true,
    "last_persona_switched_at": "2026-05-11T10:00:00+05:30",
    "access": null
  }
}
```
> `profile_created: true` → first switch, `UserMentor` row auto-created. Prompt user to fill profile.

**Errors:**
```json
{ "message": { "general": ["No active IG-scoped mentor role found for this role link, or you do not own this role assignment."] } }
{ "message": { "general": ["The selected role link is not a Mentor role."] } }
```

---

### POST `/persona/reset/`
**Auth:** JWT | **Body:** none  
**Response:** `{ "response": { "active_persona": "learner" } }`

---

## 2. Profile

### GET `/profile/`
**Auth:** MENTOR_PERSONA  
**Response:**
```json
{
  "response": {
    "id": "um-uuid",
    "about": "I love helping learners.",
    "reason": "Giving back to community.",
    "expertise": "Python, Django, React",
    "volunteer_hours": 12,
    "mentor_tier": "VERIFIED",
    "is_verified": true,
    "verified_at": "2026-04-01T10:00:00+05:30",
    "verification_note": "Passed mentor interview."
  }
}
```

---

### PATCH `/profile/`
**Auth:** MENTOR_PERSONA  
**Request** (all optional):
```json
{
  "about": "Passionate about open source.",
  "reason": "To build the next generation.",
  "expertise": "Python, Django, REST APIs"
}
```
**Response:**
```json
{
  "response": {
    "id": "um-uuid",
    "about": "Passionate about open source.",
    "reason": "To build the next generation.",
    "expertise": "Python, Django, REST APIs",
    "volunteer_hours": 12,
    "mentor_tier": "NORMAL",
    "is_verified": false
  }
}
```
**Error:** `{ "general": ["No valid fields provided for update."] }`

---

## 3. Overview

### GET `/overview/`
**Auth:** MENTOR_PERSONA  
**Response:**
```json
{
  "response": {
    "user": {
      "full_name": "Pranav V",
      "muid": "pranav-v@mulearn",
      "profile_pic": "https://..."
    },
    "mentor_profile": {
      "about": "...", "expertise": "...", "reason": "...",
      "volunteer_hours": 12, "mentor_tier": "VERIFIED", "is_verified": true
    },
    "active_persona": {
      "active_persona": "mentor",
      "active_role_link_id": "rl-uuid",
      "active_ig_id": "ig-uuid",
      "ig_name": "Entrepreneurship",
      "is_verified": true
    },
    "stats": {
      "total_mentees": 5,
      "sessions_conducted": 12,
      "pending_task_approvals": 3,
      "volunteer_hours": 12
    }
  }
}
```

---

## 4. Availability

### GET `/availability/`
**Auth:** MENTOR_PERSONA  
**Response:**
```json
{
  "response": {
    "active_ig_id": "ig-uuid",
    "slots": [
      {
        "id": "slot-uuid",
        "ig_id": "ig-uuid",
        "ig_name": "Entrepreneurship",
        "weekday": 1,
        "start_time": "18:00",
        "end_time": "19:30",
        "timezone": "Asia/Kolkata",
        "is_active": true,
        "valid_from": null,
        "valid_to": null
      }
    ]
  }
}
```

---

### POST `/availability/`
**Auth:** MENTOR_PERSONA  
**Request:**
```json
{
  "slots": [
    { "weekday": 1, "start_time": "18:00", "end_time": "19:30", "timezone": "Asia/Kolkata" },
    { "weekday": 3, "start_time": "20:00", "end_time": "21:00", "timezone": "Asia/Kolkata" }
  ]
}
```
> `weekday`: 1=Mon … 7=Sun

**Response:**
```json
{
  "response": {
    "created_ids": ["slot-uuid-1", "slot-uuid-2"],
    "errors": []
  }
}
```

---

### DELETE `/availability/<slot_id>/`
**Auth:** MENTOR_PERSONA | **Body:** none  
**Response:** `{ "response": { "slot_id": "slot-uuid" } }`  
**Error:** `{ "general": ["Slot not found or access denied."] }`

---

## 5. Sessions

### GET `/sessions/`
**Auth:** MENTOR_PERSONA  
**Query params:**

| Param | Values |
|---|---|
| `status` | `SCHEDULED` `COMPLETED` `CANCELLED` `NO_SHOW` |
| `mode` | `ONLINE` `OFFLINE` `HYBRID` |
| `date_from` | `YYYY-MM-DD` |
| `date_to` | `YYYY-MM-DD` |
| `search` | string |
| `page` | int |
| `per_page` | int |

**Response:**
```json
{
  "response": {
    "data": [
      {
        "id": "sess-uuid",
        "ig_name": "Entrepreneurship",
        "title": "Intro to Lean Startup",
        "mode": "ONLINE",
        "starts_at": "2026-05-15T18:00:00+05:30",
        "ends_at": "2026-05-15T19:30:00+05:30",
        "status": "SCHEDULED",
        "meeting_link": "https://meet.google.com/abc",
        "participants": [
          { "user_id": "u1", "full_name": "Pranav V", "participant_role": "MENTOR", "attendance_status": "INVITED" },
          { "user_id": "u2", "full_name": "Akhil K", "participant_role": "MENTEE", "attendance_status": "INVITED" }
        ]
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

### POST `/sessions/`
**Auth:** VERIFIED_MENTOR  
**Request:**
```json
{
  "mentee_id": "user-uuid",
  "title": "Intro to Lean Startup",
  "description": "Covering MVP and validation loops.",
  "mode": "ONLINE",
  "starts_at": "2026-05-15T18:00:00+05:30",
  "ends_at": "2026-05-15T19:30:00+05:30",
  "meeting_link": "https://meet.google.com/abc"
}
```
> `ig_id` is auto-set from active persona. `description`, `mode`, `meeting_link` are optional.

**Response:** `{ "general": ["Session created successfully"] }`  
**Errors:**
```json
{ "general": ["Only verified mentors can create sessions"] }
{ "mentee_id": ["Mentee not found"] }
{ "ends_at": ["ends_at must be after starts_at"] }
```

---

### PATCH `/sessions/<session_id>/`
**Auth:** MENTOR_PERSONA (must be MENTOR participant)  
**Request:**
```json
{
  "status": "COMPLETED",
  "participants": [
    {
      "user_id": "u2",
      "attendance_status": "ATTENDED",
      "progress_note": "Completed MVP module.",
      "contributed_minutes": 90
    }
  ]
}
```
> `status` choices: `COMPLETED` `CANCELLED` `NO_SHOW`  
> `attendance_status` choices: `INVITED` `ATTENDED` `ABSENT`

**Response:** `{ "general": ["Session updated successfully"] }`  
**Errors:**
```json
{ "general": ["Session not found"] }
{ "general": ["You are not a mentor participant in this session"] }
```

---

## 6. Mentees

### GET `/mentees/`
**Auth:** MENTOR_PERSONA  
**Query params:** `ig_id`, `search`, `sort_by` (`full_name`/`karma`), `page`, `per_page`

**Response:**
```json
{
  "response": {
    "active_ig_id": "ig-uuid",
    "data": [
      {
        "user_id": "u2",
        "full_name": "Akhil Kumar",
        "muid": "akhil-kumar@mulearn",
        "profile_pic": "https://...",
        "karma": 1540,
        "level": "Catalyst",
        "ig_karma": 720,
        "ig_level": "Explorer",
        "session_count": 4,
        "last_session_at": "2026-05-08T18:00:00+05:30"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

## 7. Tasks

### GET `/tasks/queue/`
**Auth:** VERIFIED_MENTOR  
**Query params:** `status` (`PENDING`/`APPROVED`/`REJECTED`), `ig_id`, `search`, `page`, `per_page`

**Response:**
```json
{
  "response": {
    "active_ig_id": "ig-uuid",
    "data": [
      {
        "id": "kal-uuid",
        "mentee_id": "u2",
        "mentee_name": "Akhil Kumar",
        "task_id": "task-uuid",
        "task_title": "Build a REST API",
        "task_hashtag": "#django-rest-api",
        "task_karma": 300,
        "ig_name": "Entrepreneurship",
        "mentor_review_status": "PENDING",
        "mentor_review_feedback": null,
        "created_at": "2026-05-09T14:22:11+05:30"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

### PATCH `/tasks/queue/<log_id>/`
**Auth:** VERIFIED_MENTOR  
**Request:**
```json
{ "status": "APPROVED", "feedback": "Great work! Clean API design." }
```
> `status`: `APPROVED` or `REJECTED` (required) | `feedback`: max 500 chars (optional)

**Response:** `{ "general": ["Task approved successfully"] }`  
**Errors:**
```json
{ "general": ["Karma log entry not found"] }
{ "general": ["This task has already been actioned"] }
{ "general": ["status is required and must be 'APPROVED' or 'REJECTED'"] }
```

---

### GET `/tasks/requests/`
**Auth:** MENTOR_PERSONA  
**Query params:** `status` (`PENDING`/`APPROVED`/`REJECTED`), `page`, `per_page`

**Response:**
```json
{
  "response": {
    "data": [
      {
        "id": "req-uuid",
        "mentor_name": "Pranav V",
        "ig_name": "Entrepreneurship",
        "title": "Build a Pitch Deck",
        "hashtag": "#pitch-deck",
        "karma": 250,
        "description": "Create a 10-slide pitch deck.",
        "status": "PENDING",
        "admin_note": null,
        "reviewed_by_name": null,
        "reviewed_at": null,
        "created_task_hashtag": null,
        "created_at": "2026-05-11T10:00:00+05:30"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

### POST `/tasks/requests/`
**Auth:** MENTOR_PERSONA  
**Request:**
```json
{
  "title": "Build a Pitch Deck",
  "hashtag": "#pitch-deck",
  "karma": 250,
  "description": "Create a 10-slide pitch deck for a startup idea."
}
```
> `title`, `hashtag`, `karma` are required. `description` is optional.

**Response:**
```json
{
  "message": { "general": ["Task creation request submitted. Pending admin review."] },
  "response": { "id": "req-uuid", "status": "PENDING", "hashtag": "#pitch-deck", ... }
}
```
**Errors:**
```json
{ "message": { "title": ["This field is required."], "karma": ["This field is required."] } }
{ "general": ["A pending request for hashtag '#pitch-deck' already exists."] }
```

---

## 8. Admin Endpoints

> **Auth:** Admin role required on all endpoints below.

### GET `/admin/task-requests/`
**Query params:**

| Param | Values | Default |
|---|---|---|
| `status` | `PENDING` `APPROVED` `REJECTED` `ALL` | `PENDING` |
| `ig_id` | UUID | — |
| `search` | string | — |
| `page` / `per_page` | int | — |

**Response:**
```json
{
  "response": {
    "data": [
      {
        "id": "req-uuid",
        "mentor_name": "Pranav V",
        "ig_name": "Entrepreneurship",
        "title": "Build a Pitch Deck",
        "hashtag": "#pitch-deck",
        "karma": 250,
        "description": "...",
        "status": "PENDING",
        "admin_note": null,
        "reviewed_by_name": null,
        "reviewed_at": null,
        "created_task_hashtag": null,
        "created_at": "2026-05-11T10:00:00+05:30"
      }
    ],
    "pagination": { "count": 1, "next": null, "previous": null }
  }
}
```

---

### PATCH `/admin/task-requests/<req_id>/`
**Auth:** ADMIN  
**Request — Approve:**
```json
{
  "action": "APPROVE",
  "type_id": "task-type-uuid",
  "level_id": "level-uuid",
  "discord_link": "https://discord.com/channels/...",
  "admin_note": "Great task idea!"
}
```
> `type_id` is required for APPROVE. `level_id`, `discord_link`, `admin_note` are optional.

**Request — Reject:**
```json
{ "action": "REJECT", "admin_note": "Duplicate of existing task #lean-canvas." }
```

**Response (approve):**
```json
{
  "message": { "general": ["Task request approved — task created."] },
  "response": { "id": "req-uuid", "status": "APPROVED", "created_task_hashtag": "#pitch-deck", ... }
}
```
**Errors:**
```json
{ "general": ["Task request not found."] }
{ "general": ["This request has already been approved."] }
{ "general": ["type_id is required to approve and create the task."] }
{ "general": ["A task with hashtag '#pitch-deck' already exists."] }
```

---

### PATCH `/admin/tier/<mentor_profile_id>/`
**Auth:** ADMIN  
> `mentor_profile_id` = `UserMentor.id` (from `GET /profile/` response `id` field)

**Request:**
```json
{ "tier": "VERIFIED", "verification_note": "Passed mentor interview." }
```
> `tier`: `VERIFIED` or `NORMAL`. `verification_note` optional (used on VERIFIED).

**Response:**
```json
{
  "message": { "general": ["Mentor tier updated to VERIFIED."] },
  "response": {
    "mentor_profile_id": "um-uuid",
    "user_id": "u1",
    "full_name": "Pranav V",
    "mentor_tier": "VERIFIED",
    "is_verified": true,
    "verified_at": "2026-05-11T14:00:00+05:30",
    "verification_note": "Passed mentor interview."
  }
}
```
> **Auto side-effects on VERIFIED:** Sets `UserMentor.is_verified=true`, `verified_at`, `verified_by`, and all `UserRoleLink(role=Mentor).verified=true` for the user.  
> **Auto side-effects on NORMAL:** Clears `is_verified`, `verified_at`, `verified_by`, `verification_note`.

**Errors:**
```json
{ "general": ["Mentor profile not found."] }
{ "general": ["tier must be 'NORMAL' or 'VERIFIED'."] }
```

---

## How to Become a Mentor (Full Flow)

```
1. Admin:  POST /api/v1/dashboard/roles/user-role/
           { "user_id": "...", "role_id": "<Mentor-role-uuid>", "ig_id": "<IG-uuid>" }
           → UserRoleLink + UserMentor auto-created

2. User:   GET  /mentor/persona/ig-roles/          → get role_link_id
           POST /mentor/persona/switch/             → { "active_role_link_id": "..." }

3. User:   PATCH /mentor/profile/                  → fill about/expertise/reason

4. Admin:  PATCH /mentor/admin/tier/<profile_id>/  → { "tier": "VERIFIED" }
           → unlocks sessions + task queue

5. User:   POST /mentor/sessions/                  → create sessions (verified only)
           GET  /mentor/tasks/queue/               → review mentee task submissions
           POST /mentor/tasks/requests/            → request new task creation
```

---

## Common Error Responses

| Scenario | Status | Message |
|---|---|---|
| No / expired JWT | 401 | Unauthorized |
| No active mentor persona | 403 | `Active mentor persona required for this IG.` |
| Mentor not verified | 403 | `Verified mentor status required for this action.` |
| Non-admin on admin endpoint | 400 | `You do not have the required role to access this page.` |
