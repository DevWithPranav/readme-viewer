# Mentor Dashboard API Documentation

> **Base prefix**: `/api/v1/dashboard/mentor/`  
> **Auth**: All endpoints require a valid JWT (`Authorization: Bearer <token>`), unless marked **[Public — no auth]**.  
> **Roles**: `ADMIN` = platform admin · `MENTOR` = verified mentor

---

## Table of Contents

1. [Onboarding](#1-onboarding)
2. [Admin Mentor Roster](#2-admin-mentor-roster)
3. [Overview & Stats](#3-overview--stats)
4. [Leaderboard](#4-leaderboard)
5. [Sessions](#5-sessions)
6. [Global Session Approval Queue](#6-global-session-approval-queue)
7. [Task Review Queue](#7-task-review-queue)
8. [Availability Slots](#8-availability-slots)
9. [Task Requests](#9-task-requests)
10. [Opportunities](#10-opportunities)
11. [Mentees & Activity Log](#11-mentees--activity-log)
12. [Karma Award](#12-karma-award)
13. [Session Reminder](#13-session-reminder)
14. [My IGs](#14-my-igs)
15. [IG Mentor Link Requests](#15-ig-mentor-link-requests)
16. [Admin Mentor Tier Update](#16-admin-mentor-tier-update)
17. [Bulk Attendance Update](#17-bulk-attendance-update)
18. [Session Clone](#18-session-clone)
19. [Availability Calendar](#19-availability-calendar)
20. [Public Endpoints](#20-public-endpoints)

---

## 1. Onboarding

### `GET /mentor/onboarding/`

Fetch the authenticated user's mentor application.

| | |
|---|---|
| **Roles** | Any authenticated user |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "mentor": {
      "id": "um-uuid-001",
      "user_id": "user-uuid-001",
      "full_name": "Arjun Nair",
      "email": "arjun@example.com",
      "muid": "arjun@mulearn",
      "about": "Passionate about open-source and teaching.",
      "expertise": ["Python", "Django", "REST APIs"],
      "reason": "I want to help junior devs grow.",
      "preferred_ig_ids": ["ig-uuid-001", "ig-uuid-002"],
      "mentor_tier": "IG_MENTOR",
      "is_verified": true,
      "verified_by": "admin-uuid",
      "verified_at": "2025-01-15T10:30:00Z",
      "verification_note": "Strong background verified.",
      "hours": 12,
      "created_at": "2025-01-01T08:00:00Z"
    }
  }
}
```

**Response — 400 (not applied)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "You have not applied to become a mentor yet."
}
```

---

### `POST /mentor/onboarding/`

Submit a mentor application.

| | |
|---|---|
| **Roles** | Any authenticated user |
| **Auth** | JWT required |

**Request Body**
```json
{
  "about": "Senior software engineer with 5 years in web development.",
  "expertise": ["JavaScript", "React", "Node.js"],
  "reason": "I want to mentor students on modern web development.",
  "preferred_ig_ids": ["ig-uuid-001", "ig-uuid-003"]
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor application submitted successfully. Awaiting admin review.",
  "response": {
    "mentor": {
      "id": "um-uuid-002",
      "user_id": "user-uuid-010",
      "full_name": "Priya Menon",
      "email": "priya@example.com",
      "muid": "priya@mulearn",
      "about": "Senior software engineer with 5 years in web development.",
      "expertise": ["JavaScript", "React", "Node.js"],
      "reason": "I want to mentor students on modern web development.",
      "preferred_ig_ids": ["ig-uuid-001", "ig-uuid-003"],
      "mentor_tier": null,
      "is_verified": false,
      "hours": 0,
      "created_at": "2026-05-24T10:00:00Z"
    }
  }
}
```

---

### `PATCH /mentor/onboarding/`

Update own mentor profile fields. If the mentor is already verified and sends `preferred_ig_ids`, pending IG link requests are automatically created and IG Leads are notified.

| | |
|---|---|
| **Roles** | Any authenticated user with a mentor application |
| **Auth** | JWT required |

**Request Body** *(all fields optional)*
```json
{
  "about": "Updated bio with new experience.",
  "expertise": ["Python", "FastAPI", "Kubernetes"],
  "reason": "Expanded focus to DevOps mentoring.",
  "preferred_ig_ids": ["ig-uuid-005"]
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor profile updated.",
  "response": {
    "mentor": { "...": "full mentor object as above" }
  }
}
```

---

## 2. Admin Mentor Roster

### `GET /mentor/list/`

Paginated list of all mentor applications.

| | |
|---|---|
| **Roles** | ADMIN |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `is_verified` | boolean string | `true` / `false` — filter by verification status |
| `search` | string | Search by name, email, or muid |
| `sort_by` | string | `full_name`, `created_at`, `mentor_tier` |
| `pageIndex` | int | Page number (default 1) |
| `perPage` | int | Page size (default 10) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": [
    {
      "id": "um-uuid-001",
      "user_id": "user-uuid-001",
      "full_name": "Arjun Nair",
      "email": "arjun@example.com",
      "muid": "arjun@mulearn",
      "mentor_tier": "IG_MENTOR",
      "is_verified": true,
      "hours": 12,
      "created_at": "2025-01-01T08:00:00Z"
    }
  ],
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

### `PATCH /mentor/<mentor_id>/verify/`

Approve or reject a pending mentor application.

| | |
|---|---|
| **Roles** | ADMIN |
| **Auth** | JWT required |

**Request Body**
```json
{
  "action": "approve",
  "note": "Background verified. Welcome aboard!",
  "mentor_tier": "IG_MENTOR"
}
```

> `action`: `"approve"` | `"reject"`  
> `mentor_tier` (approve only): `"IG_MENTOR"` | `"MENTOR"` (default: `"IG_MENTOR"`)  
> `note`: optional message shown to the mentor in a notification

**Response — Approve (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor application approved. Mentor role assigned.",
  "response": {
    "mentor": {
      "id": "um-uuid-001",
      "is_verified": true,
      "mentor_tier": "IG_MENTOR",
      "verified_at": "2026-05-24T11:00:00Z"
    }
  }
}
```

**Response — Reject (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor application rejected. User notified and may reapply."
}
```

---

## 3. Overview & Stats

### `GET /mentor/overview/`
### `GET /mentor/stats/` *(alias — same response)*

Single-call dashboard snapshot. Admins see platform-wide data; mentors see their own scoped data.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Scope session / opportunity counts to one IG |

**Response — 200 OK (Admin view)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Success",
  "response": {
    "overview": {
      "mentors": {
        "total": 120,
        "verified": 95,
        "unverified": 25,
        "pending_verification": 25
      },
      "sessions": {
        "counts": {
          "pending_approval": 3,
          "scheduled": 18,
          "completed": 74,
          "cancelled": 5,
          "rejected": 2,
          "total": 102
        },
        "upcoming": [
          {
            "id": "sess-uuid-001",
            "title": "Intro to Django REST Framework",
            "ig_name": "Web Dev",
            "starts_at": "2026-05-25T14:00:00Z",
            "status": "SCHEDULED"
          }
        ],
        "pending_global": [
          {
            "id": "sess-uuid-009",
            "title": "Open Source Contribution Tips",
            "status": "PENDING_APPROVAL",
            "created_by": "mentor-uuid-003"
          }
        ]
      },
      "task_requests": {
        "pending": 7,
        "approved": 34,
        "rejected": 4,
        "recent_pending": [
          {
            "id": "tr-uuid-001",
            "title": "Build a REST API task",
            "ig_name": "Web Dev",
            "status": "PENDING"
          }
        ]
      },
      "opportunities": {
        "total": 22,
        "published": 15,
        "draft": 5,
        "closed": 2,
        "by_ig": [
          { "ig_id": "ig-uuid-001", "ig_name": "Web Dev", "count": 8 },
          { "ig_id": "ig-uuid-002", "ig_name": "AI/ML", "count": 6 }
        ]
      },
      "mentees": {
        "total_unique": 210
      },
      "recent_activity": [
        {
          "id": "log-uuid-001",
          "action_type": "SESSION_CREATE",
          "actor": "Arjun Nair",
          "entity_name": "mentorship_session",
          "entity_id": "sess-uuid-001",
          "created_at": "2026-05-24T09:00:00Z"
        }
      ]
    }
  }
}
```

> **Note**: Session counts now include `"rejected"` status in addition to `no_show` previously documented.

**Response — Mentor (own) view**
```json
{
  "response": {
    "overview": {
      "mentors": {
        "is_verified": true,
        "mentor_tier": "IG_MENTOR",
        "hours": 12
      },
      "sessions": { "...": "scoped to sessions they participate in" },
      "task_requests": { "...": "scoped to their task requests" },
      "opportunities": { "...": "all platform opportunities" },
      "mentees": { "total_unique": 28 },
      "recent_activity": []
    }
  }
}
```

---

## 4. Leaderboard

### `GET /mentor/leaderboard/`

Ranked list of verified mentors.  
**Score formula** = `(sessions_completed × 3) + (mentees_attended × 2) + (hours × 1)`

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Scope to a specific IG |
| `pageIndex` | int | Page number (default 1) |
| `perPage` | int | Page size (default 10) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "rank": 1,
      "mentor_id": "user-uuid-001",
      "full_name": "Arjun Nair",
      "muid": "arjun@mulearn",
      "profile_pic": "https://cdn.example.com/pics/arjun.jpg",
      "mentor_tier": "IG_MENTOR",
      "sessions_completed": 20,
      "mentees_attended": 85,
      "hours": 30,
      "score": 260
    },
    {
      "rank": 2,
      "mentor_id": "user-uuid-007",
      "full_name": "Priya Menon",
      "muid": "priya@mulearn",
      "profile_pic": null,
      "mentor_tier": "IG_MENTOR",
      "sessions_completed": 15,
      "mentees_attended": 60,
      "hours": 25,
      "score": 190
    }
  ],
  "pagination": {
    "count": 95,
    "totalPages": 10,
    "isNext": true,
    "isPrev": false,
    "nextPage": 2
  }
}
```

---

## 5. Sessions

### `GET /mentor/sessions/`

Paginated session list. Admin sees all sessions; mentor sees only their own.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Filter by Interest Group |
| `status` | string | `SCHEDULED` `COMPLETED` `CANCELLED` `REJECTED` `PENDING_APPROVAL` |
| `is_global` | boolean string | `true` / `false` |
| `search` | string | Search by title or IG name |
| `sort_by` | string | `title`, `starts_at`, `status`, `created_at` |
| `pageIndex` | int | Page number |
| `perPage` | int | Page size |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "sess-uuid-001",
      "title": "Intro to Django REST Framework",
      "description": "A hands-on intro session.",
      "ig": "ig-uuid-001",
      "ig_name": "Web Dev",
      "is_global": false,
      "status": "SCHEDULED",
      "starts_at": "2026-05-25T14:00:00Z",
      "ends_at": "2026-05-25T16:00:00Z",
      "meet_link": "https://meet.google.com/abc-defg",
      "created_by": "user-uuid-001",
      "created_at": "2026-05-20T10:00:00Z"
    }
  ],
  "pagination": { "count": 18, "totalPages": 2, "isNext": true, "isPrev": false }
}
```

---

### `POST /mentor/sessions/`

Create a new mentorship session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Rules**
- Supplying `ig` → IG-scoped session (status = `SCHEDULED`) — Admin only, or IG_MENTOR linked to that IG
- Omitting `ig` → global session (status = `PENDING_APPROVAL`) — any verified mentor or admin
- Global Mentors (tier=`MENTOR`) cannot create IG-scoped sessions

**Request Body — IG-scoped session**
```json
{
  "title": "Python Debugging Workshop",
  "description": "Deep-dive into Python debugging techniques.",
  "ig": "ig-uuid-001",
  "starts_at": "2026-06-01T10:00:00Z",
  "ends_at": "2026-06-01T12:00:00Z",
  "meet_link": "https://meet.google.com/xyz-1234",
  "max_participants": 30
}
```

**Request Body — Global session**
```json
{
  "title": "Open Source Contribution for Beginners",
  "description": "Learn how to make your first PR.",
  "starts_at": "2026-06-05T15:00:00Z",
  "ends_at": "2026-06-05T17:00:00Z",
  "meet_link": "https://meet.google.com/global-sess"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Global session submitted for admin approval.",
  "response": {
    "session": {
      "id": "sess-uuid-099",
      "title": "Open Source Contribution for Beginners",
      "is_global": true,
      "status": "PENDING_APPROVAL",
      "starts_at": "2026-06-05T15:00:00Z",
      "participants": [
        {
          "user_id": "user-uuid-001",
          "full_name": "Arjun Nair",
          "participant_role": "MENTOR",
          "attendance_status": "INVITED"
        }
      ]
    }
  }
}
```

---

### `GET /mentor/sessions/<session_id>/`

Retrieve full details of a single session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "session": {
      "id": "sess-uuid-001",
      "title": "Intro to Django REST Framework",
      "description": "A hands-on intro session.",
      "ig": "ig-uuid-001",
      "ig_name": "Web Dev",
      "is_global": false,
      "status": "SCHEDULED",
      "starts_at": "2026-05-25T14:00:00Z",
      "ends_at": "2026-05-25T16:00:00Z",
      "meet_link": "https://meet.google.com/abc-defg",
      "max_participants": 30,
      "created_by": "user-uuid-001",
      "updated_by": "user-uuid-001",
      "approved_by": null,
      "approved_at": null,
      "participants": [
        {
          "user_id": "user-uuid-001",
          "full_name": "Arjun Nair",
          "participant_role": "MENTOR",
          "attendance_status": "INVITED"
        },
        {
          "user_id": "user-uuid-050",
          "full_name": "Riya Sharma",
          "participant_role": "MENTEE",
          "attendance_status": "ATTENDED"
        }
      ]
    }
  }
}
```

---

### `PATCH /mentor/sessions/<session_id>/`

Update session details. Mentors can only edit sessions they created.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Request Body** *(all fields optional)*
```json
{
  "title": "Updated: Intro to DRF + Viewsets",
  "description": "Now covering ViewSets and Routers.",
  "starts_at": "2026-05-26T14:00:00Z",
  "ends_at": "2026-05-26T16:00:00Z",
  "meet_link": "https://meet.google.com/new-link"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Session updated.",
  "response": { "session": { "...": "updated session object" } }
}
```

---

### `DELETE /mentor/sessions/<session_id>/`

Soft-delete a session (sets status to `CANCELLED`).

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Session cancelled successfully."
}
```

---

### `PATCH /mentor/sessions/<session_id>/status/`

Update only the status of a session. Uses strict transition rules.

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Allowed Transitions**

| From | To |
|---|---|
| `SCHEDULED` | `COMPLETED`, `CANCELLED` |
| `COMPLETED` | *(none)* |
| `CANCELLED` | *(none)* |
| `REJECTED` | *(none)* |
| `PENDING_APPROVAL` | Use `/sessions/<id>/approve/` instead |

**Request Body**
```json
{
  "status": "COMPLETED"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Session status updated to 'COMPLETED'."
}
```

---

### `GET /mentor/sessions/<session_id>/participants/`

List all participants in a session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "participants": [
      {
        "user_id": "user-uuid-001",
        "full_name": "Arjun Nair",
        "email": "arjun@example.com",
        "participant_role": "MENTOR",
        "attendance_status": "ATTENDED"
      },
      {
        "user_id": "user-uuid-050",
        "full_name": "Riya Sharma",
        "email": "riya@example.com",
        "participant_role": "MENTEE",
        "attendance_status": "ATTENDED"
      }
    ]
  }
}
```

---

### `POST /mentor/sessions/<session_id>/participants/`

Add a participant to a session. Enforces `max_participants` cap for `MENTEE` role.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Request Body**
```json
{
  "user": "user-uuid-050",
  "participant_role": "MENTEE",
  "attendance_status": "INVITED"
}
```

> `participant_role`: `MENTOR` | `CO_MENTOR` | `MENTEE`  
> `attendance_status`: `INVITED` | `ATTENDED` | `ABSENT`

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Participant added.",
  "response": {
    "participant": {
      "user_id": "user-uuid-050",
      "full_name": "Riya Sharma",
      "participant_role": "MENTEE",
      "attendance_status": "INVITED"
    }
  }
}
```

**Error — Session full (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Session capacity reached (30 mentees)."
}
```

---

### `DELETE /mentor/sessions/<session_id>/participants/<user_id>/`

Remove a participant from a session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Required | Description |
|---|---|---|---|
| `participant_role` | string | ✅ Yes | The role of the participant to remove (`MENTOR`, `CO_MENTOR`, `MENTEE`) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Participant removed."
}
```

---

## 6. Global Session Approval Queue

### `GET /mentor/sessions/pending/`

Paginated list of global sessions awaiting admin approval. Includes keyword-based IG suggestions per session.

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `search` | string | Filter by title or creator name |
| `sort_by` | string | `title`, `created_at` |
| `pageIndex` | int | Page number |
| `perPage` | int | Page size |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "sess-uuid-099",
      "title": "Open Source Contribution for Beginners",
      "description": "Learn how to make your first PR.",
      "is_global": true,
      "status": "PENDING_APPROVAL",
      "starts_at": "2026-06-05T15:00:00Z",
      "created_by": "Arjun Nair",
      "created_at": "2026-05-24T10:00:00Z",
      "suggested_igs": [
        { "ig_id": "ig-uuid-003", "ig_name": "Open Source" },
        { "ig_id": "ig-uuid-007", "ig_name": "Developer Tools" }
      ]
    }
  ],
  "pagination": { "count": 3, "totalPages": 1, "isNext": false, "isPrev": false }
}
```

---

### `PATCH /mentor/sessions/<session_id>/approve/`

Approve or reject a pending global session. Optionally convert it to an IG-scoped session on approval.

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Request Body — Approve**
```json
{
  "action": "approve",
  "remarks": "Great topic — scheduling it.",
  "ig_id": "ig-uuid-003"
}
```

> `ig_id` is optional. If provided on approve, the session is converted from global → IG-scoped (`is_global` becomes `false`).

**Request Body — Reject**
```json
{
  "action": "reject",
  "remarks": "Topic already covered recently. Please check the schedule."
}
```

**Response — Approve (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Global session approved and scheduled.",
  "response": {
    "session": {
      "id": "sess-uuid-099",
      "status": "SCHEDULED",
      "is_global": false,
      "ig": "ig-uuid-003",
      "approved_by": "admin-uuid-001",
      "approved_at": "2026-05-24T12:00:00Z"
    }
  }
}
```

**Response — Reject (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Global session rejected.",
  "response": {
    "session": { "status": "REJECTED", "...": "full session object" }
  }
}
```

---

## 7. Task Review Queue

### `GET /mentor/review-queue/`

Paginated list of `KarmaActivityLog` entries pending mentor review. Mentors only see tasks from their linked IGs.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `status` | string | `PENDING` (default), `APPROVED`, `REJECTED` |
| `ig_id` | UUID | Filter by Interest Group |
| `search` | string | Filter by user name, task title, or hashtag |
| `sort_by` | string | `created_at`, `karma` |
| `pageIndex` | int | Page number |
| `perPage` | int | Page size |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "kal-uuid-001",
      "user_id": "user-uuid-050",
      "user_name": "Riya Sharma",
      "task_id": "task-uuid-010",
      "task_title": "Build a REST API",
      "task_hashtag": "#restapi",
      "ig_name": "Web Dev",
      "karma": 500,
      "mentor_review_status": "PENDING",
      "submission_url": "https://github.com/riya/rest-api-project",
      "submitted_at": "2026-05-23T08:00:00Z"
    }
  ],
  "pagination": { "count": 7, "totalPages": 1, "isNext": false, "isPrev": false }
}
```

---

### `GET /mentor/review-queue/<kal_id>/`

Retrieve a single task submission entry. Mentors can only access entries from their own IGs.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "submission": {
      "id": "kal-uuid-001",
      "user_id": "user-uuid-050",
      "user_name": "Riya Sharma",
      "task_id": "task-uuid-010",
      "task_title": "Build a REST API",
      "task_hashtag": "#restapi",
      "ig_name": "Web Dev",
      "karma": 500,
      "mentor_review_status": "PENDING",
      "submission_url": "https://github.com/riya/rest-api-project",
      "submitted_at": "2026-05-23T08:00:00Z"
    }
  }
}
```

---

### `PATCH /mentor/review-queue/<kal_id>/`

Mentor approves or rejects a task submission. Karma is NOT credited here — admin finalises via the appraiser flow.

| | |
|---|---|
| **Roles** | MENTOR only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "status": "APPROVED",
  "feedback": "Great implementation! Clean code and well-documented."
}
```

> `status`: `"APPROVED"` | `"REJECTED"`

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task submission marked as APPROVED.",
  "response": {
    "status": "APPROVED"
  }
}
```

**Response — Already reviewed (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Already reviewed: status is 'APPROVED'."
}
```

---

## 8. Availability Slots

### `GET /mentor/availability/`

Paginated list of active availability slots. Mentors see only their own slots.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `mentor_user_id` | UUID | Admin only — filter by specific mentor |
| `ig_id` | UUID | Filter by Interest Group |
| `sort_by` | string | `weekday`, `start_time` |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "slot-uuid-001",
      "mentor_user_id": "user-uuid-001",
      "mentor_name": "Arjun Nair",
      "ig_id": "ig-uuid-001",
      "ig_name": "Web Dev",
      "weekday": 1,
      "start_time": "14:00:00",
      "end_time": "16:00:00",
      "timezone": "Asia/Kolkata",
      "is_active": true
    }
  ],
  "pagination": { "count": 5, "totalPages": 1, "isNext": false, "isPrev": false }
}
```

---

### `POST /mentor/availability/`

Create a new availability slot (sets `mentor_user` to the authenticated user automatically).

| | |
|---|---|
| **Roles** | MENTOR only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "ig": "ig-uuid-001",
  "weekday": 1,
  "start_time": "14:00:00",
  "end_time": "16:00:00",
  "timezone": "Asia/Kolkata"
}
```

> `weekday`: `0` = Monday … `6` = Sunday

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Availability slot created.",
  "response": {
    "slot": {
      "id": "slot-uuid-010",
      "mentor_user_id": "user-uuid-001",
      "weekday": 1,
      "start_time": "14:00:00",
      "end_time": "16:00:00",
      "timezone": "Asia/Kolkata",
      "is_active": true
    }
  }
}
```

---

### `PUT /mentor/availability/<slot_id>/`

Full replace of an existing slot. Mentor can only update their own slots.

| | |
|---|---|
| **Roles** | MENTOR only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "ig": "ig-uuid-001",
  "weekday": 3,
  "start_time": "10:00:00",
  "end_time": "12:00:00",
  "timezone": "UTC"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Availability slot updated.",
  "response": { "slot": { "...": "updated slot object" } }
}
```

---

### `DELETE /mentor/availability/<slot_id>/`

Soft-delete (deactivate) an availability slot.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

Mentors can only deactivate their own slots. Admins can deactivate any slot.

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Availability slot deactivated."
}
```

---

## 9. Task Requests

### `GET /mentor/task-requests/`

Paginated list of mentor task proposals. Mentors see only their own requests.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `status` | string | `PENDING`, `APPROVED`, `REJECTED` |
| `search` | string | Filter by title, hashtag, mentor name, or IG name |
| `sort_by` | string | `title`, `status`, `created_at` |
| `pageIndex` | int | Page number |
| `perPage` | int | Page size |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "tr-uuid-001",
      "mentor_id": "user-uuid-001",
      "mentor_name": "Arjun Nair",
      "ig_id": "ig-uuid-001",
      "ig_name": "Web Dev",
      "title": "Build a REST API with Django",
      "hashtag": "#djangorest",
      "description": "Students will build a fully functional REST API using Django REST Framework.",
      "karma": 500,
      "status": "PENDING",
      "admin_note": null,
      "reviewed_by": null,
      "reviewed_at": null,
      "created_task": null,
      "created_at": "2026-05-20T10:00:00Z"
    }
  ],
  "pagination": { "count": 7, "totalPages": 1, "isNext": false, "isPrev": false }
}
```

---

### `POST /mentor/task-requests/`

Submit a new task proposal for admin review.

| | |
|---|---|
| **Roles** | MENTOR only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "ig": "ig-uuid-001",
  "title": "Build a REST API with Django",
  "hashtag": "#djangorest",
  "description": "Students will build a fully functional REST API using Django REST Framework, covering serializers, viewsets, authentication, and pagination.",
  "karma": 500
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task proposal submitted. Awaiting admin review.",
  "response": {
    "task_request": {
      "id": "tr-uuid-010",
      "title": "Build a REST API with Django",
      "hashtag": "#djangorest",
      "karma": 500,
      "status": "PENDING",
      "created_at": "2026-05-24T11:00:00Z"
    }
  }
}
```

---

### `GET /mentor/task-requests/<task_request_id>/`

Retrieve a single task request.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

Mentors can only retrieve their own requests.

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "task_request": { "...": "full task_request object as above" }
  }
}
```

---

### `PATCH /mentor/task-requests/<task_request_id>/`

Admin reviews (approve or reject) a pending task request. On approval, a `TaskList` entry is automatically created.

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "status": "APPROVED",
  "admin_note": "Great task proposal! Added to the Web Dev IG task list."
}
```

> `status`: `"APPROVED"` | `"REJECTED"`

**Response — 200 OK (Approved)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task request approved.",
  "response": {
    "task_request": {
      "id": "tr-uuid-001",
      "status": "APPROVED",
      "admin_note": "Great task proposal! Added to the Web Dev IG task list.",
      "reviewed_by": "admin-uuid-001",
      "reviewed_at": "2026-05-24T14:00:00Z",
      "created_task": {
        "id": "task-uuid-999",
        "title": "Build a REST API with Django",
        "hashtag": "#djangorest",
        "karma": 500
      }
    }
  }
}
```

---

### `DELETE /mentor/task-requests/<task_request_id>/`

Mentor withdraws their own **PENDING** task request before admin review. Only `PENDING` requests can be withdrawn.

| | |
|---|---|
| **Roles** | MENTOR only |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Task request withdrawn successfully."
}
```

**Error — Not pending (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Cannot withdraw a task request with status 'APPROVED'. Only PENDING requests can be withdrawn."
}
```

---

## 10. Opportunities

### `GET /mentor/opportunities/`

Paginated list of IG opportunities.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Filter by Interest Group |
| `type` | string | Opportunity type (e.g. `INTERNSHIP`, `HACKATHON`, `JOB`) |
| `status` | string | `DRAFT`, `PUBLISHED`, `CLOSED`, `ARCHIVED` |
| `search` | string | Filter by title or IG name |
| `sort_by` | string | `title`, `status`, `starts_at`, `created_at` |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "opp-uuid-001",
      "ig_id": "ig-uuid-001",
      "ig_name": "Web Dev",
      "title": "Frontend Internship — TechCorp",
      "description": "3-month paid internship for React developers.",
      "type": "INTERNSHIP",
      "status": "PUBLISHED",
      "application_url": "https://techcorp.com/apply",
      "starts_at": "2026-07-01T00:00:00Z",
      "ends_at": "2026-09-30T00:00:00Z",
      "created_by": "user-uuid-001",
      "created_at": "2026-05-10T08:00:00Z"
    }
  ],
  "pagination": { "count": 15, "totalPages": 2, "isNext": true, "isPrev": false }
}
```

---

### `POST /mentor/opportunities/`

Create a new IG opportunity.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Request Body**
```json
{
  "ig": "ig-uuid-001",
  "title": "Frontend Internship — TechCorp",
  "description": "3-month paid internship for React developers.",
  "type": "INTERNSHIP",
  "status": "PUBLISHED",
  "application_url": "https://techcorp.com/apply",
  "starts_at": "2026-07-01T00:00:00Z",
  "ends_at": "2026-09-30T00:00:00Z"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Opportunity created.",
  "response": { "opportunity": { "...": "full opportunity object" } }
}
```

---

### `GET /mentor/opportunities/<opportunity_id>/`

Retrieve a single opportunity.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "opportunity": { "...": "full opportunity object as above" }
  }
}
```

---

### `PATCH /mentor/opportunities/<opportunity_id>/`

Partially update an opportunity.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Request Body** *(all fields optional)*
```json
{
  "status": "CLOSED",
  "description": "Applications are now closed. Thanks for applying!"
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Opportunity updated.",
  "response": { "opportunity": { "...": "updated opportunity object" } }
}
```

---

### `DELETE /mentor/opportunities/<opportunity_id>/`

Archive (soft-delete) an opportunity (sets status to `ARCHIVED`).

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Opportunity archived."
}
```

---

## 11. Mentees & Activity Log

### `GET /mentor/mentees/`

Paginated list of distinct mentees across sessions.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Scope to a specific IG |
| `mentor_user_id` | UUID | Admin only — scope to a specific mentor |
| `sort_by` | string | `full_name`, `total_sessions` |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "user_id": "user-uuid-050",
      "user__full_name": "Riya Sharma",
      "user__muid": "riya@mulearn",
      "user__email": "riya@example.com",
      "total_sessions": 8
    },
    {
      "user_id": "user-uuid-075",
      "user__full_name": "Kiran R",
      "user__muid": "kiran@mulearn",
      "user__email": "kiran@example.com",
      "total_sessions": 3
    }
  ],
  "pagination": { "count": 210, "totalPages": 21, "isNext": true, "isPrev": false }
}
```

---

### `GET /mentor/mentees/<user_id>/`

Full profile of a single mentee: user info, shared sessions, karma earned, and task review stats.  
Mentors only see mentees from sessions they were a MENTOR/CO_MENTOR in.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "mentee": {
      "user_id": "user-uuid-050",
      "full_name": "Riya Sharma",
      "email": "riya@example.com",
      "muid": "riya@mulearn",
      "total_sessions": 5,
      "completed_sessions": 4,
      "total_karma_earned": 1500,
      "tasks_reviewed": 3,
      "tasks_approved": 2,
      "tasks_rejected": 1,
      "sessions": [
        {
          "session_id": "sess-uuid-001",
          "title": "Intro to DRF",
          "ig_name": "Web Dev",
          "status": "COMPLETED",
          "starts_at": "2026-05-25T14:00:00Z",
          "ends_at": "2026-05-25T16:00:00Z"
        }
      ]
    }
  }
}
```

**Error — No shared sessions (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Mentee has no sessions with you."
}
```

---

### `GET /mentor/activity-log/`

Paginated system action log. Admins see all entries; mentors see only their own.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Filter by IG |
| `action_type` | string | `SESSION_CREATE` `SESSION_UPDATE` `SESSION_STATUS` `KARMA_AWARD` `TASK_REVIEW` `OPPORTUNITY_POST` `IG_CONTENT_UPDATE` |
| `sort_by` | string | `created_at`, `action_type` |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "log-uuid-001",
      "action_type": "SESSION_CREATE",
      "actor_user_id": "user-uuid-001",
      "actor_name": "Arjun Nair",
      "subject_user_id": null,
      "subject_name": null,
      "ig_id": "ig-uuid-001",
      "ig_name": "Web Dev",
      "entity_name": "mentorship_session",
      "entity_id": "sess-uuid-001",
      "old_data": null,
      "new_data": { "title": "Intro to DRF", "is_global": false },
      "remarks": null,
      "created_at": "2026-05-20T10:00:00Z"
    }
  ],
  "pagination": { "count": 88, "totalPages": 9, "isNext": true, "isPrev": false }
}
```

---

## 12. Karma Award

### `GET /mentor/sessions/<session_id>/karma-award/`

List karma awards for a session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "awards": [
      {
        "id": "award-uuid-001",
        "session_id": "sess-uuid-001",
        "mentor_id": "user-uuid-001",
        "mentor_name": "Arjun Nair",
        "karma": 200,
        "note": "Excellent mentoring session on DRF.",
        "awarded_by": "admin-uuid-001",
        "awarded_at": "2026-05-26T17:00:00Z"
      }
    ]
  }
}
```

---

### `POST /mentor/sessions/<session_id>/karma-award/`

Admin awards karma to a mentor for a **COMPLETED** session. One award per mentor per session.  
Also increments the mentor's `hours` by 1 and updates their wallet karma.

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "mentor_id": "user-uuid-001",
  "karma": 200,
  "note": "Excellent mentoring session on DRF."
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "200 karma awarded to mentor.",
  "response": {
    "award": {
      "id": "award-uuid-001",
      "session_id": "sess-uuid-001",
      "mentor_id": "user-uuid-001",
      "mentor_name": "Arjun Nair",
      "karma": 200,
      "note": "Excellent mentoring session on DRF.",
      "awarded_by": "admin-uuid-001",
      "awarded_at": "2026-05-26T17:00:00Z"
    }
  }
}
```

**Error — Session not completed (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Karma can only be awarded for COMPLETED sessions."
}
```

**Error — Not a MENTOR participant (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "User is not a MENTOR participant in this session."
}
```

**Error — Already awarded (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Karma already awarded to this mentor for this session."
}
```

---

## 13. Session Reminder

### `POST /mentor/sessions/<session_id>/remind/`

Send reminder notifications to all `INVITED` or `ATTENDED` participants. Only works for sessions in `SCHEDULED` or `PENDING_APPROVAL` status.  
A **12-hour cooldown** is enforced between consecutive reminders for the same session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Reminder sent to 24 participant(s)."
}
```

**Error — Wrong status (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Reminders can only be sent for SCHEDULED or PENDING_APPROVAL sessions."
}
```

**Error — Cooldown active (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Cooldown active. Please wait 5 hours before sending another reminder."
}
```

---

## 14. My IGs

### `GET /mentor/my-igs/`

Returns all IGs the authenticated mentor is actively linked to as a mentor.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "igs": [
      {
        "ig_id": "ig-uuid-001",
        "ig_name": "Web Dev",
        "ig_code": "WEB"
      },
      {
        "ig_id": "ig-uuid-002",
        "ig_name": "AI/ML",
        "ig_code": "AIML"
      }
    ]
  }
}
```

---

## 15. IG Mentor Link Requests

### `GET /mentor/ig-requests/?ig_id=<ig_id>`

List pending mentor-to-IG link requests for a given IG. Accessible by Admin or the IG Lead of that IG.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR (IG Lead only) |
| **Auth** | JWT required |

**Query Parameters**

| Param | Type | Required | Description |
|---|---|---|---|
| `ig_id` | UUID | ✅ Yes | The Interest Group to inspect |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "requests": [
      {
        "id": "uil-uuid-001",
        "user_id": "user-uuid-010",
        "full_name": "Priya Menon",
        "email": "priya@example.com",
        "muid": "priya@mulearn",
        "ig_id": "ig-uuid-001",
        "ig_name": "Web Dev",
        "requested_at": "2026-05-22T09:00:00Z"
      }
    ]
  }
}
```

---

### `PATCH /mentor/ig-requests/<request_id>/`

IG Lead (or Admin) approves or rejects a pending mentor IG link request.

- **Approve** → `UserIgLink.is_active = True`, mentor is notified ✅
- **Reject** → `UserIgLink` row is deleted, mentor is notified ❌

| | |
|---|---|
| **Roles** | ADMIN, MENTOR (IG Lead only) |
| **Auth** | JWT required |

**Request Body — Approve**
```json
{
  "action": "approve"
}
```

**Request Body — Reject**
```json
{
  "action": "reject",
  "note": "We currently have enough mentors in this IG."
}
```

**Response — Approve (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor approved for Web Dev."
}
```

**Response — Reject (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor request for Web Dev rejected."
}
```

---

## 16. Admin Mentor Tier Update

### `PATCH /mentor/list/<mentor_id>/tier/`

Change the `mentor_tier` of an already-verified mentor. Sends a notification to the mentor.  
Cannot be used on unverified mentors (approve the application first).

| | |
|---|---|
| **Roles** | ADMIN only |
| **Auth** | JWT required |

**Request Body**
```json
{
  "mentor_tier": "MENTOR"
}
```

> `mentor_tier`: `"IG_MENTOR"` | `"MENTOR"`

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor tier updated from 'IG_MENTOR' to 'MENTOR'.",
  "response": {
    "mentor": { "...": "full mentor object" }
  }
}
```

**Error — Not verified (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Cannot change the tier of an unverified mentor. Approve the application first."
}
```

**Response — No change (200 OK)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Mentor is already at tier 'MENTOR'. No change made."
}
```

---

## 17. Bulk Attendance Update

### `PATCH /mentor/sessions/<session_id>/attendance/`

Bulk-update `attendance_status` for multiple participants in a single request.  
Admin can update any session; Mentors can only update sessions they created.  
Every `user_id` in the body must already be a participant in the session.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Request Body**
```json
{
  "participants": [
    { "user_id": "user-uuid-050", "attendance_status": "ATTENDED" },
    { "user_id": "user-uuid-075", "attendance_status": "ABSENT" },
    { "user_id": "user-uuid-088", "attendance_status": "NO_SHOW" }
  ]
}
```

> `attendance_status`: `INVITED` | `ATTENDED` | `ABSENT` | `NO_SHOW`

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Attendance updated for 3 participant(s)."
}
```

**Error — Non-participants found (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "The following users are not participants in this session: user-uuid-999"
}
```

---

## 18. Session Clone

### `POST /mentor/sessions/<session_id>/clone/`

Deep-copies a session with the following rules:
- **title** → `"Copy of <original title>"`
- **status** → `PENDING_APPROVAL` (if global) or `SCHEDULED` (if IG-scoped)
- **starts_at / ends_at** → cleared (`null`) — caller must `PATCH` to set new times
- All other fields (description, mode, ig, max_participants, meeting_link, venue, is_global) are preserved
- Creator is automatically added as `MENTOR` participant
- Original participants are **NOT** copied

Mentors can only clone sessions they created.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Session cloned successfully. Update starts_at and ends_at before publishing.",
  "response": {
    "session": {
      "id": "sess-uuid-200",
      "title": "Copy of Intro to Django REST Framework",
      "is_global": false,
      "status": "SCHEDULED",
      "starts_at": null,
      "ends_at": null,
      "ig": "ig-uuid-001",
      "participants": [
        {
          "user_id": "user-uuid-001",
          "full_name": "Arjun Nair",
          "participant_role": "MENTOR",
          "attendance_status": "INVITED"
        }
      ]
    }
  }
}
```

**Error — Not creator (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "You can only clone sessions you created."
}
```

---

## 19. Availability Calendar

### `GET /mentor/availability/calendar/`

Returns all active availability slots formatted for calendar widget consumption.  
Mentors receive only their own slots; Admins receive all mentor slots.

| | |
|---|---|
| **Roles** | ADMIN, MENTOR |
| **Auth** | JWT required |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "slot-uuid-001",
      "mentor_user_id": "user-uuid-001",
      "mentor_name": "Arjun Nair",
      "ig_id": "ig-uuid-001",
      "ig_name": "Web Dev",
      "weekday": 1,
      "start_time": "14:00:00",
      "end_time": "16:00:00",
      "timezone": "Asia/Kolkata",
      "is_active": true
    }
  ]
}
```

> **Note**: Unlike `GET /availability/`, this endpoint returns all slots in a flat list without pagination, optimised for rendering in calendar UIs.

---

## 20. Public Endpoints

All endpoints in this section require **no authentication**.

---

### `GET /mentor/<muid>/public/`

Public read-only mentor profile card. Only returns verified mentors.

| | |
|---|---|
| **Roles** | Public — no auth |
| **Auth** | None |

**Path Parameters**

| Param | Description |
|---|---|
| `muid` | The mentor's muLearn user ID (e.g. `arjun@mulearn`) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "full_name": "Arjun Nair",
    "muid": "arjun@mulearn",
    "profile_pic": "https://cdn.example.com/pics/arjun.jpg",
    "about": "Passionate about open-source and teaching.",
    "expertise": ["Python", "Django", "REST APIs"],
    "mentor_tier": "IG_MENTOR",
    "hours": 12
  }
}
```

**Error — Not found (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Verified mentor profile not found."
}
```

---

### `GET /mentor/<muid>/public/sessions/`

Paginated list of **completed** sessions where the mentor was `MENTOR` or `CO_MENTOR`.

| | |
|---|---|
| **Roles** | Public — no auth |
| **Auth** | None |

**Query Parameters**

| Param | Type | Description |
|---|---|---|
| `ig_id` | UUID | Filter by Interest Group |
| `mode` | string | Filter by session mode (`ONLINE`, `OFFLINE`, `HYBRID`) |
| `sort_by` | string | `starts_at`, `title` |
| `pageIndex` | int | Page number |
| `perPage` | int | Page size |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": [
    {
      "id": "sess-uuid-001",
      "title": "Intro to Django REST Framework",
      "ig_name": "Web Dev",
      "status": "COMPLETED",
      "starts_at": "2026-05-25T14:00:00Z",
      "ends_at": "2026-05-25T16:00:00Z"
    }
  ],
  "pagination": { "count": 20, "totalPages": 2, "isNext": true, "isPrev": false }
}
```

---

### `GET /mentor/availability/public/?mentor_muid=<muid>`

Public availability slots for a verified mentor. Useful for mentee scheduling flows.

| | |
|---|---|
| **Roles** | Public — no auth |
| **Auth** | None |

**Query Parameters**

| Param | Type | Required | Description |
|---|---|---|---|
| `mentor_muid` | string | ✅ Yes | The mentor's muLearn user ID |
| `ig_id` | UUID | No | Filter slots by Interest Group |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "response": {
    "mentor": {
      "full_name": "Arjun Nair",
      "muid": "arjun@mulearn"
    },
    "availability": [
      {
        "id": "slot-uuid-001",
        "ig_id": "ig-uuid-001",
        "ig_name": "Web Dev",
        "weekday": 1,
        "start_time": "14:00:00",
        "end_time": "16:00:00",
        "timezone": "Asia/Kolkata",
        "is_active": true
      }
    ]
  }
}
```

**Error — Missing param (400)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "'mentor_muid' query parameter is required."
}
```

---

## Common Error Responses

| Scenario | Status | Example message |
|---|---|---|
| Not authenticated | `401` | `"Authentication credentials were not provided."` |
| Wrong role | `403` | `"Permission denied."` |
| Resource not found | `400` | `"Session not found."` |
| Validation error | `400` | `{ "title": ["This field is required."] }` |
| Duplicate action | `400` | `"Karma already awarded to this mentor for this session."` |
| Cooldown active | `400` | `"Cooldown active. Please wait 5 hours before sending another reminder."` |

---

## Pagination Object

All list endpoints return a `pagination` object:

```json
{
  "pagination": {
    "count": 95,
    "totalPages": 10,
    "isNext": true,
    "isPrev": false,
    "nextPage": 2
  }
}
```

Common pagination query params: `pageIndex` (default `1`), `perPage` (default `10`), `search`, `sort_by`.
