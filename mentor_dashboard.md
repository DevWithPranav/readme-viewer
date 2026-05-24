# Mentor Dashboard — API Testing Guide

> Test in this exact order — each step depends on data created in the previous one.
> All endpoints are prefixed with: `POST /api/v1/dashboard/mentor/`
> Auth: `Authorization: Bearer <jwt_token>`

---

## Prerequisites

Run the alter script before any testing:
```bash
python alter-scripts/alter-1.61.py
```

You need two JWT tokens:
- **`ADMIN_TOKEN`** — a user with the Admin role
- **`MENTOR_TOKEN`** — a regular user who will apply as mentor

---

## Step 1 — Mentor Applies (Onboarding)

**Usage:** A user applies to become a mentor, optionally declaring preferred IGs.

**Endpoint:** `POST /api/v1/dashboard/mentor/onboarding/`  
**Auth:** Any logged-in user (`MENTOR_TOKEN`)

**Request:**
```json
{
  "about": "I am a full-stack developer with 5 years of experience in React and Django.",
  "expertise": "React, Django, REST APIs, System Design",
  "reason": "I want to give back to the community and help junior developers grow.",
  "preferred_ig_ids": [
    "ig-uuid-web-dev",
    "ig-uuid-backend"
  ]
}
```

**Response (201):**
```json
{
  "statusCode": 6000,
  "message": "Mentor application submitted successfully. Awaiting admin review.",
  "response": {
    "mentor": {
      "id": "mentor-row-uuid",
      "full_name": "Pranav Kumar",
      "email": "pranav@mulearn.org",
      "muid": "pranav-kumar@mulearn",
      "about": "I am a full-stack developer...",
      "expertise": "React, Django, REST APIs, System Design",
      "reason": "I want to give back...",
      "hours": 0,
      "mentor_tier": "NORMAL",
      "is_verified": false,
      "verified_by_name": null,
      "verified_at": null,
      "verification_note": null,
      "created_at": "2024-05-24T06:00:00Z"
    }
  }
}
```

**Error — Already applied:**
```json
{
  "statusCode": 6002,
  "message": "You have already applied to become a mentor."
}
```

---

## Step 2 — Admin Lists All Mentor Applications

**Usage:** Admin reviews the pending mentor application queue.

**Endpoint:** `GET /api/v1/dashboard/mentor/list/`  
**Auth:** `ADMIN_TOKEN`  
**Query params:** `?is_verified=false&perPage=10&pageIndex=1`

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "data": [
      {
        "id": "mentor-row-uuid",
        "full_name": "Pranav Kumar",
        "email": "pranav@mulearn.org",
        "muid": "pranav-kumar@mulearn",
        "about": "I am a full-stack developer...",
        "hours": 0,
        "mentor_tier": "NORMAL",
        "is_verified": false,
        "verified_by_name": null,
        "created_at": "2024-05-24T06:00:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false
    }
  }
}
```

---

## Step 3a — Admin Approves the Mentor

**Usage:** Admin approves the mentor application. Assigns the **Mentor role**, creates IG links, and notifies the user.

**Endpoint:** `PATCH /api/v1/dashboard/mentor/<mentor-row-uuid>/verify/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "action": "approve",
  "mentor_tier": "VERIFIED",
  "note": "Reviewed portfolio — excellent candidate."
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Mentor application approved. Mentor role assigned.",
  "response": {
    "mentor": {
      "id": "mentor-row-uuid",
      "full_name": "Pranav Kumar",
      "is_verified": true,
      "mentor_tier": "VERIFIED",
      "verified_by_name": "Admin User",
      "verified_at": "2024-05-24T06:05:00Z",
      "verification_note": "Reviewed portfolio — excellent candidate."
    }
  }
}
```

**Side effects (all happen automatically):**
- ✅ `UserRoleLink` created → mentor now has the **Mentor role** in their JWT
- ✅ `UserIgLink` rows created for `ig-uuid-web-dev` and `ig-uuid-backend` (`assignment_type=MENTOR`)
- ✅ Notification sent: _"🎉 Congratulations! Your mentor application has been approved."_
- ✅ `SystemActionLog` row created

**Error — Invalid action:**
```json
{
  "statusCode": 6002,
  "message": "'action' must be 'approve' or 'reject'."
}
```

---

## Step 3b — Admin Rejects the Mentor Application

**Usage:** Admin rejects the application. The `user_mentor` row is **deleted** so the user gets a clean slate to reapply with an improved profile.

**Endpoint:** `PATCH /api/v1/dashboard/mentor/<mentor-row-uuid>/verify/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "action": "reject",
  "note": "Profile incomplete. Please add more details about your expertise and share portfolio links."
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Mentor application rejected. User notified and may reapply."
}
```

**Side effects:**
- ❌ `user_mentor` row deleted → user's application is wiped (no `is_verified=false` ambiguity)
- ❌ No `UserRoleLink` created → user does NOT get the Mentor role
- ❌ No `UserIgLink` created
- 📢 Notification sent to user: _"Your mentor application was reviewed and not approved. Reason: Profile incomplete..."_

> **Why delete?** With only `is_verified` as the flag, `is_verified=false` can't distinguish  
> "pending review" from "rejected". Deleting the row means the user can reapply freely.

**Error — Mentor not found (already rejected/deleted):**
```json
{
  "statusCode": 6002,
  "message": "Mentor not found."
}
```

---

## Step 4 — Admin Creates an IG Session

**Usage:** Admin creates a session tied to a specific Interest Group.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "ig": "ig-uuid-web-dev",
  "title": "React Hooks Deep Dive",
  "description": "An advanced session covering useEffect, useMemo, and custom hooks.",
  "mode": "ONLINE",
  "starts_at": "2024-06-01T10:00:00Z",
  "ends_at": "2024-06-01T12:00:00Z",
  "meeting_link": "https://meet.google.com/abc-defg-hij"
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Session created successfully.",
  "response": {
    "session": {
      "id": "session-uuid-1",
      "title": "React Hooks Deep Dive",
      "status": "SCHEDULED",
      "is_global": false,
      "ig_id": "ig-uuid-web-dev",
      "ig_name": "Web Development",
      "mode": "ONLINE",
      "starts_at": "2024-06-01T10:00:00Z",
      "ends_at": "2024-06-01T12:00:00Z",
      "meeting_link": "https://meet.google.com/abc-defg-hij",
      "participants": [
        {
          "user_id": "admin-user-uuid",
          "full_name": "Admin User",
          "participant_role": "MENTOR",
          "attendance_status": "INVITED"
        }
      ]
    }
  }
}
```

---

## Step 5 — Add Mentor as Participant

**Usage:** Add the verified mentor as a MENTOR participant in the session.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/<session-uuid-1>/participants/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "user": "mentor-user-uuid",
  "participant_role": "MENTOR"
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Participant added.",
  "response": {
    "participant": {
      "id": "link-uuid",
      "user_id": "mentor-user-uuid",
      "full_name": "Pranav Kumar",
      "participant_role": "MENTOR",
      "attendance_status": "INVITED"
    }
  }
}
```

---

## Step 6 — Send Session Reminders (Feature 4)

**Usage:** Manually trigger reminder notifications to all participants before the session.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/<session-uuid-1>/remind/`  
**Auth:** `ADMIN_TOKEN` or `MENTOR_TOKEN`

**Request:** _(no body needed)_

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Reminder sent to 2 participant(s)."
}
```

> **Side effect:** `Notification` rows created for each INVITED/ATTENDED participant.

**Error — Session not scheduled:**
```json
{
  "statusCode": 6002,
  "message": "Reminders can only be sent for SCHEDULED or PENDING_APPROVAL sessions."
}
```

---

## Step 7 — Mark Session as Completed

**Usage:** Admin marks the session complete after it happens.

**Endpoint:** `PATCH /api/v1/dashboard/mentor/sessions/<session-uuid-1>/status/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "status": "COMPLETED"
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Session status updated to 'COMPLETED'."
}
```

---

## Step 8 — Admin Awards Karma to Mentor (Feature 1)

**Usage:** After a completed session, admin awards karma points to the mentor.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/<session-uuid-1>/karma-award/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "mentor_id": "mentor-user-uuid",
  "karma": 50,
  "note": "Excellent session delivery — mentees rated 4.8/5."
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "50 karma awarded to mentor.",
  "response": {
    "award": {
      "id": "award-uuid",
      "session_id": "session-uuid-1",
      "session_title": "React Hooks Deep Dive",
      "mentor_id": "mentor-user-uuid",
      "mentor_name": "Pranav Kumar",
      "karma": 50,
      "note": "Excellent session delivery — mentees rated 4.8/5.",
      "awarded_by_name": "Admin User",
      "awarded_at": "2024-06-01T13:00:00Z",
      "kal_id": null,
      "created_at": "2024-06-01T13:00:00Z"
    }
  }
}
```

**Side effects:**
- `Wallet.karma` for mentor incremented by 50
- `Notification` sent to mentor: _"You've been awarded 50 karma..."_
- `SystemActionLog` row created with `action_type=KARMA_AWARD`

**Error — Session not completed:**
```json
{
  "statusCode": 6002,
  "message": "Karma can only be awarded for COMPLETED sessions."
}
```

**Error — Double award attempt:**
```json
{
  "statusCode": 6002,
  "message": "Karma already awarded to this mentor for this session."
}
```

**Error — Not a mentor participant:**
```json
{
  "statusCode": 6002,
  "message": "User is not a MENTOR participant in this session."
}
```

---

## Step 9 — View Karma Awards for a Session

**Endpoint:** `GET /api/v1/dashboard/mentor/sessions/<session-uuid-1>/karma-award/`  
**Auth:** `ADMIN_TOKEN` or `MENTOR_TOKEN`

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "awards": [
      {
        "id": "award-uuid",
        "session_id": "session-uuid-1",
        "session_title": "React Hooks Deep Dive",
        "mentor_id": "mentor-user-uuid",
        "mentor_name": "Pranav Kumar",
        "karma": 50,
        "note": "Excellent session delivery...",
        "awarded_by_name": "Admin User",
        "awarded_at": "2024-06-01T13:00:00Z"
      }
    ]
  }
}
```

---

## Step 10 — Mentor Submits a Global Session (needs Admin Approval)

**Usage:** Mentor proposes a cross-IG platform-wide session. Goes to `PENDING_APPROVAL`.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/`  
**Auth:** `MENTOR_TOKEN`

**Request:**
```json
{
  "title": "System Design for Scale",
  "description": "Designing distributed systems — load balancing, caching, and database sharding.",
  "mode": "ONLINE",
  "starts_at": "2024-06-15T10:00:00Z",
  "ends_at": "2024-06-15T13:00:00Z",
  "meeting_link": "https://zoom.us/j/12345678"
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Global session submitted for admin approval.",
  "response": {
    "session": {
      "id": "session-uuid-2",
      "title": "System Design for Scale",
      "status": "PENDING_APPROVAL",
      "is_global": true,
      "ig_id": null,
      "ig_name": null
    }
  }
}
```

---

## Step 11 — Admin Views Pending Global Sessions (with IG Suggestions — Feature 6)

**Endpoint:** `GET /api/v1/dashboard/mentor/sessions/pending/`  
**Auth:** `ADMIN_TOKEN`

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "data": [
      {
        "id": "session-uuid-2",
        "title": "System Design for Scale",
        "status": "PENDING_APPROVAL",
        "is_global": true,
        "created_by_name": "Pranav Kumar",
        "created_at": "2024-06-10T08:00:00Z",
        "suggested_igs": [
          { "ig_id": "ig-uuid-backend", "ig_name": "Backend Development" },
          { "ig_id": "ig-uuid-cloud",   "ig_name": "Cloud & DevOps" }
        ]
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false }
  }
}
```

---

## Step 12 — Admin Approves Global Session (Optionally Attaches an IG)

**Endpoint:** `PATCH /api/v1/dashboard/mentor/sessions/<session-uuid-2>/approve/`  
**Auth:** `ADMIN_TOKEN`

**Request — approve and convert to IG-scoped:**
```json
{
  "action": "approve",
  "ig_id": "ig-uuid-backend",
  "remarks": "Great topic — attaching to Backend IG."
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Global session approved and scheduled.",
  "response": {
    "session": {
      "id": "session-uuid-2",
      "title": "System Design for Scale",
      "status": "SCHEDULED",
      "is_global": false,
      "ig_id": "ig-uuid-backend",
      "ig_name": "Backend Development",
      "approved_by_name": "Admin User",
      "approved_at": "2024-06-10T09:00:00Z"
    }
  }
}
```

**Request — reject:**
```json
{
  "action": "reject",
  "remarks": "Out of scope for current semester."
}
```

---

## Step 13 — Mentor Reviews a Task Submission (Feature 2)

**Usage:** Mentor sees pending task submissions from their IG's learners.

**Step 13a — List the review queue**

**Endpoint:** `GET /api/v1/dashboard/mentor/review-queue/?status=PENDING&ig_id=ig-uuid-web-dev`  
**Auth:** `MENTOR_TOKEN`

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "data": [
      {
        "id": "kal-uuid-1",
        "user_name": "Alice Learner",
        "user_muid": "alice@mulearn",
        "task_title": "Build a REST API",
        "task_hashtag": "#rest-api-build",
        "ig_name": "Web Development",
        "karma": 100,
        "mentor_review_status": "PENDING",
        "mentor_reviewed_at": null,
        "mentor_review_feedback": null,
        "created_at": "2024-05-20T10:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1 }
  }
}
```

**Step 13b — Mentor approves the submission**

**Endpoint:** `PATCH /api/v1/dashboard/mentor/review-queue/<kal-uuid-1>/`  
**Auth:** `MENTOR_TOKEN`

**Request:**
```json
{
  "status": "APPROVED",
  "feedback": "Good implementation. Clean code and proper error handling."
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Task submission marked as APPROVED.",
  "response": { "status": "APPROVED" }
}
```

**Request — reject:**
```json
{
  "status": "REJECTED",
  "feedback": "Missing authentication middleware. Please revise and resubmit."
}
```

**Error — Already reviewed:**
```json
{
  "statusCode": 6002,
  "message": "Already reviewed: status is 'APPROVED'."
}
```

> **Note:** Mentor approval does NOT credit karma. Admin must still `appraiser_approved=true` via the existing karma flow to actually credit the learner.

---

## Step 14 — Mentor Leaderboard (Feature 3)

**Usage:** See top mentors ranked by sessions, mentees, and hours.

**Endpoint:** `GET /api/v1/dashboard/mentor/leaderboard/`  
**Auth:** `ADMIN_TOKEN` or `MENTOR_TOKEN`  
**Query params:** `?ig_id=ig-uuid-web-dev&pageIndex=1&perPage=10` _(ig_id optional)_

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "data": [
      {
        "rank": 1,
        "mentor_id": "mentor-user-uuid",
        "full_name": "Pranav Kumar",
        "muid": "pranav-kumar@mulearn",
        "profile_pic": "https://example.com/profile.png",
        "mentor_tier": "VERIFIED",
        "sessions_completed": 5,
        "mentees_attended": 23,
        "hours": 12,
        "score": 73
      },
      {
        "rank": 2,
        "mentor_id": "other-mentor-uuid",
        "full_name": "Arjun Mentor",
        "muid": "arjun@mulearn",
        "profile_pic": null,
        "mentor_tier": "NORMAL",
        "sessions_completed": 3,
        "mentees_attended": 10,
        "hours": 8,
        "score": 37
      }
    ],
    "pagination": {
      "count": 2,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

> **Score formula:** `(sessions_completed × 3) + (mentees_attended × 2) + (hours × 1)`

---

## Step 15 — Dashboard Overview (single-call snapshot)

**Usage:** Load the mentor dashboard landing page with all counts in one call.

**Endpoint:** `GET /api/v1/dashboard/mentor/overview/`  
**Auth:** `ADMIN_TOKEN` or `MENTOR_TOKEN`  
**Query params:** `?ig_id=ig-uuid-web-dev` _(optional)_

**Response for Admin (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "overview": {
      "mentors": {
        "total": 42,
        "verified": 30,
        "unverified": 12,
        "pending_verification": 12
      },
      "sessions": {
        "counts": {
          "pending_approval": 3,
          "scheduled": 8,
          "completed": 15,
          "cancelled": 2,
          "no_show": 1,
          "total": 29
        },
        "upcoming": [
          {
            "id": "session-uuid-1",
            "title": "React Hooks Deep Dive",
            "starts_at": "2024-06-01T10:00:00Z",
            "status": "SCHEDULED",
            "ig_name": "Web Development"
          }
        ],
        "pending_global": [
          {
            "id": "session-uuid-3",
            "title": "Open Source Contribution Guide",
            "status": "PENDING_APPROVAL",
            "created_by_name": "Another Mentor"
          }
        ]
      },
      "task_requests": {
        "pending": 4,
        "approved": 11,
        "rejected": 2,
        "recent_pending": []
      },
      "opportunities": {
        "total": 18,
        "published": 10,
        "draft": 5,
        "closed": 3,
        "by_ig": [
          { "ig_id": "ig-uuid-web-dev", "ig_name": "Web Development", "count": 6 }
        ]
      },
      "mentees": { "total_unique": 67 },
      "recent_activity": [
        {
          "action_type": "KARMA_AWARD",
          "actor_name": "Admin User",
          "entity_name": "mentor_karma_award",
          "new_data": { "karma": 50, "session_id": "session-uuid-1" },
          "created_at": "2024-06-01T13:00:00Z"
        }
      ]
    }
  }
}
```

---

---

## Quick Reference — All Endpoints

| # | Method | Endpoint | Auth | Feature |
|---|---|---|---|---|
| 1 | `POST` | `/onboarding/` | Any | Apply as mentor |
| 2 | `GET` | `/onboarding/` | Any | View own mentor profile |
| 3 | `PATCH` | `/onboarding/` | Any | Update own profile + preferred IGs (verified → triggers IG Lead request) |
| 4 | `GET` | `/list/` | Admin | All mentor applications |
| 5 | `PATCH` | `/<pk>/verify/` | Admin | **Approve** mentor (`IG_MENTOR` or `MENTOR`) → assigns role, IG links, notification |
| 5b | `PATCH` | `/<pk>/verify/` | Admin | **Reject** mentor → deletes row, user can reapply, notified |
| 6 | `GET` | `/overview/` | Admin/Mentor | Dashboard snapshot |
| 7 | `GET` | `/leaderboard/` | Admin/Mentor | Mentor leaderboard |
| 8 | `GET` | `/sessions/pending/` | Admin | Global sessions queue |
| 9 | `PATCH` | `/sessions/<pk>/approve/` | Admin | Approve/reject global |
| 10 | `GET` | `/sessions/` | Admin/Mentor | List sessions |
| 11 | `POST` | `/sessions/` | Admin/Mentor/IG_MENTOR | Create session (IG or global, tier-gated) |
| 12 | `GET` | `/sessions/<pk>/` | Admin/Mentor | Session detail |
| 13 | `PATCH` | `/sessions/<pk>/` | Admin/Mentor | Edit session |
| 14 | `DELETE` | `/sessions/<pk>/` | Admin | Cancel session |
| 15 | `PATCH` | `/sessions/<pk>/status/` | Admin | Change session status |
| 16 | `GET` | `/sessions/<pk>/participants/` | Admin/Mentor | List participants |
| 17 | `POST` | `/sessions/<pk>/participants/` | Admin/Mentor | Add participant |
| 18 | `DELETE` | `/sessions/<pk>/participants/<upk>/` | Admin/Mentor | Remove participant |
| 19 | `GET` | `/sessions/<pk>/karma-award/` | Admin/Mentor | List karma awards |
| 20 | `POST` | `/sessions/<pk>/karma-award/` | Admin | Award karma to mentor |
| 21 | `POST` | `/sessions/<pk>/remind/` | Admin/Mentor | Send reminders |
| 22 | `GET` | `/review-queue/` | Admin/Mentor | Task review queue |
| 23 | `PATCH` | `/review-queue/<pk>/` | Mentor | Review a submission |
| 24 | `GET` | `/availability/` | Admin/Mentor | Availability slots |
| 25 | `POST` | `/availability/` | Mentor | Create slot |
| 26 | `PUT` | `/availability/<pk>/` | Mentor | Update slot |
| 27 | `DELETE` | `/availability/<pk>/` | Admin/Mentor | Deactivate slot |
| 28 | `GET` | `/task-requests/` | Admin/Mentor | Task proposals |
| 29 | `POST` | `/task-requests/` | Mentor | Submit task proposal |
| 30 | `PATCH` | `/task-requests/<pk>/` | Admin | Approve/reject proposal |
| 31 | `GET` | `/opportunities/` | Admin/Mentor | IG opportunities |
| 32 | `POST` | `/opportunities/` | Admin/Mentor | Create opportunity |
| 33 | `PATCH` | `/opportunities/<pk>/` | Admin/Mentor | Update opportunity |
| 34 | `DELETE` | `/opportunities/<pk>/` | Admin | Delete opportunity |
| 35 | `GET` | `/mentees/` | Admin/Mentor | Unique mentees list |
| 36 | `GET` | `/activity-log/` | Admin/Mentor | Audit log |
| 37 | `GET` | `/my-igs/` | Admin/Mentor | Mentor's active IG links |
| 38 | `GET` | `/ig-requests/?ig_id=<id>` | Admin/IG Lead | Pending mentor IG join requests |
| 39 | `PATCH` | `/ig-requests/<pk>/` | Admin/IG Lead | Approve or reject IG join request |
| 40 | `PATCH` | `PATCH /api/v1/dashboard/ig/get/<ig_pk>/` | Admin/IG Lead | Update IG fields + auto-create mentor profile/role/link |

---

## Step A — Run DB Migration (Tier Rename)

> [!IMPORTANT]
> Run this **before** testing any mentor features. It renames the tier enum in the DB.

```bash
cd alter-scripts
python alter-1.61.py
```

Expected output:
```
✓ Migrated mentor_tier values (NORMAL→IG_MENTOR, VERIFIED→MENTOR)
✓ Altered user_mentor.mentor_tier enum
✓ DB version set to 1.61
```

Verify:
```sql
SELECT mentor_tier, COUNT(*) FROM user_mentor GROUP BY mentor_tier;
-- Should only show IG_MENTOR or MENTOR, no NORMAL or VERIFIED
```

---

## Step B — Approve Mentor with `IG_MENTOR` Tier

**Usage:** Admin approves an applicant as an IG-scoped mentor, with specific IGs they'll manage.

**Endpoint:** `PATCH /api/v1/dashboard/mentor/<mentor_user_mentor_pk>/verify/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "action": "approve",
  "mentor_tier": "IG_MENTOR",
  "note": "Approved as Web Dev IG mentor"
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Mentor application approved. Mentor role assigned.",
  "response": {
    "mentor": {
      "mentor_tier": "IG_MENTOR",
      "is_verified": true
    }
  }
}
```

**Side-effects:**
- `UserRoleLink` created with `Mentor` role
- `UserIgLink(assignment_type=MENTOR, is_active=True)` created for each `preferred_ig_ids` UUID
- Notification sent to mentor

---

## Step C — Approve Mentor with `MENTOR` (Global) Tier

**Usage:** Admin approves as a global mentor (no specific IG).

**Endpoint:** `PATCH /api/v1/dashboard/mentor/<pk>/verify/`  
**Auth:** `ADMIN_TOKEN`

**Request:**
```json
{
  "action": "approve",
  "mentor_tier": "MENTOR",
  "note": "Approved as global mentor"
}
```

**Side-effects:**
- `UserRoleLink` created with `Mentor` role
- **No** `UserIgLink` created (global mentors are not IG-scoped)
- Notification sent

---

## Step D — IG_MENTOR Creates an IG Session

**Usage:** A verified `IG_MENTOR` creates a session for one of their linked IGs. Goes directly to `SCHEDULED`.

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/`  
**Auth:** `IG_MENTOR_TOKEN` (verified `IG_MENTOR`)

**Request:**
```json
{
  "title": "React Hooks Deep Dive",
  "description": "Advanced hooks patterns for production apps.",
  "ig": "ig-uuid-web-dev",
  "starts_at": "2024-07-15T10:00:00Z",
  "ends_at": "2024-07-15T11:30:00Z",
  "meeting_link": "https://meet.example.com/xyz"
}
```

**Response (201):**
```json
{
  "statusCode": 6000,
  "message": "Session created successfully.",
  "response": {
    "session": {
      "status": "SCHEDULED",
      "is_global": false,
      "ig": "ig-uuid-web-dev"
    }
  }
}
```

**Error — wrong IG (not linked):**
```json
{ "statusCode": 6001, "message": "You are not an IG Mentor for this interest group." }
```

---

## Step E — IG_MENTOR Creates a Global Session

**Usage:** An `IG_MENTOR` can also create global sessions (goes to PENDING_APPROVAL like the `MENTOR` tier).

**Endpoint:** `POST /api/v1/dashboard/mentor/sessions/`  
**Auth:** `IG_MENTOR_TOKEN`

**Request:**
```json
{
  "title": "Open Mentoring — Ask Me Anything",
  "description": "A platform-wide open session for all learners.",
  "starts_at": "2024-07-20T15:00:00Z",
  "ends_at": "2024-07-20T16:00:00Z"
}
```
> No `ig` field → treated as global.

**Response (201):**
```json
{
  "statusCode": 6000,
  "message": "Global session submitted for admin approval.",
  "response": {
    "session": {
      "status": "PENDING_APPROVAL",
      "is_global": true
    }
  }
}
```

**Error — MENTOR tier trying IG session:**
```json
{ "statusCode": 6001, "message": "Global Mentors cannot create IG-scoped sessions." }
```

---

## Step F — Verified Mentor Requests a New IG Link

**Usage:** A verified mentor adds new IG UUIDs to their profile. Each new IG triggers a pending `UserIgLink` and notifies the IG Lead.

**Endpoint:** `PATCH /api/v1/dashboard/mentor/onboarding/`  
**Auth:** `MENTOR_TOKEN` (must already be verified)

**Request:**
```json
{
  "preferred_ig_ids": ["ig-uuid-flutter", "ig-uuid-ai"]
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "message": "Mentor profile updated."
}
```

**What happens behind the scenes:**
- For each new IG UUID (not already in `UserIgLink`):
  - Creates `UserIgLink(assignment_type=MENTOR, is_active=False)` (pending)
  - Notifies all `{ig_code} IGLead` role holders for that IG
- Already-linked IGs are skipped silently

**Verify (DB check):**
```sql
SELECT * FROM user_ig_link WHERE user_id = '<mentor_user_id>' AND is_active = FALSE;
```

---

## Step G — IG Lead Views Pending Requests

**Endpoint:** `GET /api/v1/dashboard/mentor/ig-requests/?ig_id=<ig-uuid-flutter>`  
**Auth:** `IG_LEAD_TOKEN` (must have `{ig.code} IGLead` role)

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "requests": [
      {
        "id": "uil-uuid-pending",
        "user_id": "mentor-user-uuid",
        "full_name": "Pranav Kumar",
        "email": "pranav@mulearn.org",
        "muid": "pranav@mulearn",
        "ig_id": "ig-uuid-flutter",
        "ig_name": "Flutter Development",
        "requested_at": "2024-06-05T08:30:00Z"
      }
    ]
  }
}
```

**Error — not IG Lead for this IG:**
```json
{ "statusCode": 6001, "message": "You are not an IG Lead for this interest group." }
```

---

## Step H — IG Lead Approves a Request

**Endpoint:** `PATCH /api/v1/dashboard/mentor/ig-requests/uil-uuid-pending/`  
**Auth:** `IG_LEAD_TOKEN`

**Request:**
```json
{
  "action": "approve"
}
```

**Response (200):**
```json
{ "statusCode": 6000, "message": "Mentor approved for Flutter Development." }
```

**Side-effects:**
- `UserIgLink.is_active = True` (was `False`)
- Mentor gets notification: "✅ IG Mentor Request Approved — Flutter Development"
- Mentor can now create sessions for Flutter IG

---

## Step H2 — IG Lead Rejects a Request

**Request:**
```json
{
  "action": "reject",
  "note": "This IG already has enough mentors."
}
```

**Response (200):**
```json
{ "statusCode": 6000, "message": "Mentor request for Flutter Development rejected." }
```

**Side-effects:**
- `UserIgLink` row deleted (no pending tombstone)
- Mentor gets notification: "❌ IG Mentor Request Not Approved — Flutter Development. Reason: This IG already has enough mentors."

---

## Step I — IG Lead Directly Assigns a Mentor

**Usage:** IG Lead adds a user as a mentor by updating the IG's `mentors` list. System auto-creates all necessary rows.

**Endpoint:** `PATCH /api/v1/dashboard/ig/get/<ig_pk>/`  
**Auth:** `IG_LEAD_TOKEN` or `ADMIN_TOKEN`

**Request:**
```json
{
  "mentors": [
    { "muid": "target-user@mulearn" }
  ]
}
```

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "interestGroup": { ... }
  }
}
```

**Side-effects (per new mentor muid):**
- `UserMentor` created with `is_verified=True`, `mentor_tier=IG_MENTOR` (if not exists)
- `UserRoleLink` created with `Mentor` role (if not exists)
- `UserIgLink(assignment_type=MENTOR, is_active=True)` created (if not exists)
- Notification to newly assigned user: "🎓 You've been assigned as an IG Mentor — {ig_name}"

---

## Step J — Mentor Views Their IGs

**Endpoint:** `GET /api/v1/dashboard/mentor/my-igs/`  
**Auth:** `IG_MENTOR_TOKEN`

**Response (200):**
```json
{
  "statusCode": 6000,
  "response": {
    "igs": [
      {
        "ig_id": "ig-uuid-web-dev",
        "ig_name": "Web Development",
        "ig_code": "WD"
      },
      {
        "ig_id": "ig-uuid-flutter",
        "ig_name": "Flutter Development",
        "ig_code": "FLT"
      }
    ]
  }
}
```

