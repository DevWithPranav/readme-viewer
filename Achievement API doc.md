# Achievement System — Comprehensive API Reference

> **Base Path:** `/api/v1/dashboard/achievement/`  
> **Auth:** All endpoints require a valid JWT in the `Authorization: Bearer <token>` header unless noted otherwise.

---

## Table of Contents

1. [System Overview](#1-system-overview)
2. [Data Models](#2-data-models)
3. [Rule Engine & Rule Types](#3-rule-engine--rule-types)
4. [User-Facing Endpoints](#4-user-facing-endpoints)
   - [GET `/list/`](#41-get-list)
   - [GET `/eligible/`](#42-get-eligible)
   - [POST `/claim/<achievement_id>/`](#43-post-claimachivement_id)
   - [GET `/list/user/<muid>/`](#44-get-listusermuid)
   - [GET `/progress/`](#45-get-progress)
5. [Admin — Achievement CRUD](#5-admin--achievement-crud)
   - [POST `/create/`](#51-post-create)
   - [PUT `/update/<achievement_id>/`](#52-put-updateachievement_id)
   - [DELETE `/delete/<achievement_id>/`](#53-delete-deleteachievement_id)
6. [Admin — Rule Management](#6-admin--rule-management)
   - [GET `/rules/`](#61-get-rules)
   - [POST `/rules/create/`](#62-post-rulescreate)
   - [GET `/rules/<rule_id>/`](#63-get-rulesrule_id)
   - [POST `/rules/<rule_id>/deactivate/`](#64-post-rulesrule_iddeactivate)
   - [POST `/rules/<rule_id>/activate/`](#65-post-rulesrule_idactivate)
7. [Admin — Simulation & Debugging](#7-admin--simulation--debugging)
   - [GET `/simulate/<muid>/`](#71-get-simulatemuid)
   - [GET `/debug/<muid>/<achievement_id>/`](#72-get-debugmuidachievement_id)
8. [Admin — Manual Operations](#8-admin--manual-operations)
   - [POST `/manual-issue/`](#81-post-manual-issue)
   - [POST `/revoke/`](#82-post-revoke)
9. [Admin — Audit & Logs](#9-admin--audit--logs)
   - [GET `/audit/<muid>/`](#91-get-auditmuid)
   - [GET `/issued-log/`](#92-get-issued-log)
10. [VC (Verifiable Credential) Endpoints](#10-vc-verifiable-credential-endpoints)
    - [POST `/issue-vc/`](#101-post-issue-vc)
11. [Bulk Operations](#11-bulk-operations)
    - [POST `/bulk-issue/`](#111-post-bulk-issue)
    - [GET `/bulk-issue/template/`](#112-get-bulk-issuetemplate)
    - [POST `/bulk-claim/`](#113-post-bulk-claim)
12. [Event System (Internal)](#12-event-system-internal)
13. [Background Tasks & Cron Jobs](#13-background-tasks--cron-jobs)
14. [Common Response Format](#14-common-response-format)
15. [Error Reference](#15-error-reference)

---

## 1. System Overview

The achievement system is built on a **claim-based model** with a **rule engine**:

```
User completes activity
       │
       ▼
Achievement Event emitted → Aggregates updated (IG karma, streaks, skill progress)
       │
       ▼
User visits /eligible/ → Rule Engine evaluates conditions
       │
       ▼
User calls /claim/<id>/ → Eligibility re-verified → Achievement issued → Audit logged
       │
       ▼
(Optional) VC issued via /issue-vc/ → Qseverse external call
```

**Key Principles:**
- Achievements are **never auto-issued**; users must explicitly claim them.
- All issuance operations are **idempotent** — safe to retry.
- Rules are **versioned and immutable** once created.
- Every issue/revoke action is captured in an **audit log**.

---

## 2. Data Models

### `Achievement`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | `UUID` (string) | PK | Unique identifier |
| `name` | `string` | max 75, unique | Display name |
| `description` | `string` | max 300 | Short description |
| `icon` | `string` | max 100, blank | Relative file path or full URL |
| `has_vc` | `boolean` | required | Whether a VC can be issued |
| `tags` | `JSON array` | required | Searchable tags |
| `type` | `string` | max 36 | Achievement category/type |
| `level_id` | `UUID` (FK → Level) | nullable | Optional associated level |
| `template_id` | `string` | nullable | Qseverse VC template ID |
| `created_by` | `UUID` (FK → User) | required | Creator user |
| `created_at` | `datetime` | auto | Creation timestamp |
| `updated_by` | `UUID` (FK → User) | required | Last editor |
| `updated_at` | `datetime` | auto | Last update timestamp |

### `AchievementRule`

| Field | Type | Constraints | Description |
|---|---|---|---|
| `id` | `UUID` | PK | Unique identifier |
| `achievement` | `UUID` (FK → Achievement) | required | Parent achievement |
| `version` | `integer` | unique per achievement | Auto-incremented on create |
| `rule_type` | `enum` | see Rule Types | Evaluator to use |
| `conditions` | `JSON object` | required | Rule parameters (varies by type) |
| `is_active` | `boolean` | default: `true` | Only one rule active per achievement |
| `created_by` | `UUID` (FK → User) | required | Creator |
| `created_at` | `datetime` | auto | |

> **Note:** Creating a new rule for an achievement **automatically deactivates** the previous active rule.

### `UserAchievementsLog`

| Field | Type | Description |
|---|---|---|
| `id` | `UUID` | PK |
| `user_id` | `UUID` (FK → User) | Owner |
| `achievement_id` | `UUID` (FK → Achievement) | Which achievement |
| `rule_version` | `integer` | Which rule version triggered it |
| `is_issued` | `boolean` | Whether VC has been processed |
| `vc_url` | `string` | URL to the issued VC |
| `created_at` | `datetime` | When achievement was claimed/issued |

> Unique constraint on `(user_id, achievement_id)`.

### `AchievementAuditLog`

| Field | Type | Description |
|---|---|---|
| `id` | `UUID` | PK |
| `user` | `UUID` (FK → User) | Affected user |
| `achievement` | `UUID` (FK → Achievement) | Affected achievement |
| `action` | `enum` | `issued`, `revoked`, `vc_issued`, `vc_failed` |
| `rule_version` | `integer` (nullable) | Which rule version at time of action |
| `metadata` | `JSON` (nullable) | Extra context (source, reason, vc_url, error) |
| `performed_by` | `UUID` (FK → User, nullable) | Admin who performed the action |
| `created_at` | `datetime` | Timestamp |

### Supporting Aggregation Models

| Model | Table | Purpose |
|---|---|---|
| `UserIgKarma` | `user_ig_karma` | Pre-aggregated karma per user per IG |
| `UserSkillProgress` | `user_skill_progress` | Pre-aggregated task count per skill |
| `UserDailyActivity` | `user_daily_activity` | Daily activity flags (task, karma, login) |
| `UserStreak` | `user_streak` | Current & longest streak per type |
| `AchievementEvent` | `achievement_event` | Immutable event log (append-only) |

---

## 3. Rule Engine & Rule Types

The rule engine is **stateless** and **deterministic** — same input always produces the same output.

### Rule Types

#### `ig_karma` — IG Karma Threshold

```json
{
  "rule_type": "ig_karma",
  "conditions": {
    "ig_id": "<uuid-of-interest-group>",
    "required_karma": 500
  }
}
```
Passes when the user's total karma for the given IG ≥ `required_karma`.

---

#### `skill` — Skill Task Count

```json
{
  "rule_type": "skill",
  "conditions": {
    "skill_id": "<uuid-of-skill>",
    "required_tasks": 10
  }
}
```
Passes when the user has completed ≥ `required_tasks` tasks tagged with `skill_id`.

---

#### `streak` — Daily Streak

```json
{
  "rule_type": "streak",
  "conditions": {
    "streak_type": "daily_task",
    "required_streak": 30
  }
}
```
`streak_type` options: `daily_task`, `daily_login`  
Passes when the user's **current** streak ≥ `required_streak`.

---

#### `milestone` — Total Milestone

```json
{
  "rule_type": "milestone",
  "conditions": {
    "milestone_type": "total_karma",
    "required_value": 1000
  }
}
```
`milestone_type` options: `total_karma`, `total_tasks`  
Passes when the aggregate value ≥ `required_value`.

---

#### `event` — Event Attendance

```json
{
  "rule_type": "event",
  "conditions": {
    "event_name": "MuLearn Summit 2024",
    "required_attendance": 1
  }
}
```
Passes when the user has ≥ `required_attendance` approved karma logs for tasks linked to `event_name`.

---

#### `task_completion` — Specific Task Completion

```json
{
  "rule_type": "task_completion",
  "conditions": {
    "task_hashtag": "#onboarding"
  }
}
```
Passes when the user has at least one approved `KarmaActivityLog` for a task with the given hashtag.

---

### Progress Object (returned by all evaluation endpoints)

```json
{
  "current": 350,
  "required": 500,
  "percentage": 70,
  "ig_id": "abc-123"
}
```

Field meanings vary by rule type:

| Rule Type | Extra Progress Fields |
|---|---|
| `ig_karma` | `ig_id` |
| `skill` | `skill_id` |
| `streak` | `streak_type` |
| `milestone` | `milestone_type` |
| `event` | `event_name` |
| `task_completion` | `task_hashtag` |

---

## 4. User-Facing Endpoints

### 4.1 `GET /list/`

Retrieve all achievements. Optionally annotate with whether a specific user holds each one.

**Query Parameters**

| Parameter | Type | Required | Description |
|---|---|---|---|
| `user_id` | `UUID` string | No | If provided, adds `has_achievement: true/false` per item |

**Request**
```http
GET /api/v1/dashboard/achievement/list/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "First Steps",
      "description": "Complete your first task on μLearn",
      "icon": "achievements/icons/first-steps.png",
      "icon_url": "https://api.mulearn.org/media/achievements/icons/first-steps.png",
      "has_vc": true,
      "tags": ["beginner", "onboarding"],
      "type": "milestone",
      "level_id": null,
      "template_id": "tmpl_abc123",
      "has_achievement": false,
      "created_by": "user-uuid",
      "updated_by": "user-uuid",
      "created_at": "2024-01-15T10:30:00Z",
      "updated_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

**Field Notes**
- `icon_url`: Fully resolved URL (built from `MEDIA_URL` or returned as-is if already `http://`).
- `has_achievement`: Only present when `?user_id=` param is supplied.

---

### 4.2 `GET /eligible/`

Returns achievements the **currently authenticated user** is eligible to claim (not yet claimed, rule conditions met).

**Request**
```http
GET /api/v1/dashboard/achievement/eligible/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
      "achievement_name": "First Steps",
      "eligible": true,
      "reason": "IG Karma: 550/500",
      "progress": {
        "current": 550,
        "required": 500,
        "percentage": 100,
        "ig_id": "ig-uuid-here"
      }
    }
  ]
}
```

> **Constraint:** Only returns achievements where the rule is `is_active = true` and the user has **not** already claimed the achievement.

---

### 4.3 `POST /claim/<achievement_id>/`

Claim a specific achievement. Eligibility is re-verified server-side at claim time.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `achievement_id` | `UUID` string | ID of the achievement to claim |

**Request**
```http
POST /api/v1/dashboard/achievement/claim/550e8400-e29b-41d4-a716-446655440000/
Authorization: Bearer <token>
```
_(No request body required)_

**Response — 200 OK (Success)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement claimed successfully!",
  "response": {
    "achievement_name": "First Steps",
    "vc_pending": true
  }
}
```

**Response — 400 Bad Request (Not Eligible)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Not eligible: IG Karma: 350/500",
  "response": {
    "progress": {
      "current": 350,
      "required": 500,
      "percentage": 70,
      "ig_id": "ig-uuid-here"
    }
  }
}
```

**Response — 400 Bad Request (Already Claimed)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Achievement already claimed",
  "response": {}
}
```

**Constraints**
- User must pass the active rule's eligibility check.
- If no active rule exists for the achievement, returns `"No active rule for this achievement"`.
- Issuance is idempotent; duplicate claims return an error instead of creating duplicate records.
- If `has_vc: true`, the `vc_pending` flag signals the frontend to offer VC issuance.

---

### 4.4 `GET /list/user/<muid>/`

Retrieve all achievements a specific user has claimed.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `muid` | `string` | The user's `muid` (e.g. `john@mulearn`) |

**Request**
```http
GET /api/v1/dashboard/achievement/list/user/john@mulearn/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "id": "log-uuid-here",
      "user_id": "user-uuid-here",
      "achievement": {
        "id": "achievement-uuid",
        "achievement_name": "First Steps",
        "description": "Complete your first task on μLearn",
        "icon": "achievements/icons/first-steps.png",
        "icon_url": "https://api.mulearn.org/media/achievements/icons/first-steps.png",
        "level_id": null,
        "tags": ["beginner", "onboarding"],
        "template_id": "tmpl_abc123"
      },
      "is_issued": true,
      "vc_url": "https://qseverse.com/vc/abc123"
    }
  ]
}
```

**Error — 400 (Invalid muid format)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Invalid format for muid"
}
```

---

### 4.5 `GET /progress/`

Get the authenticated user's progress towards **all** achievements (claimed or not), including percentage completion.

**Request**
```http
GET /api/v1/dashboard/achievement/progress/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
      "achievement_name": "First Steps",
      "eligible": true,
      "reason": "IG Karma: 550/500",
      "progress": {
        "current": 550,
        "required": 500,
        "percentage": 100,
        "ig_id": "ig-uuid-here"
      }
    },
    {
      "achievement_id": "660e8400-e29b-41d4-a716-446655440001",
      "achievement_name": "Streak Master",
      "eligible": false,
      "reason": "Already claimed",
      "progress": {
        "current": 30,
        "required": 30,
        "percentage": 100,
        "streak_type": "daily_task"
      }
    },
    {
      "achievement_id": "770e8400-e29b-41d4-a716-446655440002",
      "achievement_name": "Karma King",
      "eligible": false,
      "reason": "total_karma: 350/1000",
      "progress": {
        "current": 350,
        "required": 1000,
        "percentage": 35,
        "milestone_type": "total_karma"
      }
    }
  ]
}
```

> **Note:** This endpoint evaluates all active rules including ones the user has already claimed. Claimed achievements show `"reason": "Already claimed"` and `"eligible": false`.

---

## 5. Admin — Achievement CRUD

### 5.1 `POST /create/`

Create a new achievement. Accepts both `multipart/form-data` (for file uploads) and `application/json`.

**Request — JSON**
```http
POST /api/v1/dashboard/achievement/create/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "name": "First Steps",
  "description": "Complete your first task on μLearn",
  "tags": ["beginner", "onboarding"],
  "type": "milestone",
  "has_vc": true,
  "icon": "https://cdn.example.com/icons/first-steps.png",
  "level_id": null,
  "template_id": "tmpl_abc123"
}
```

**Request — Multipart (with icon file)**
```
POST /api/v1/dashboard/achievement/create/
Content-Type: multipart/form-data

name=First Steps
description=Complete your first task on μLearn
tags=["beginner","onboarding"]
type=milestone
has_vc=true
template_id=tmpl_abc123
icon=<binary file upload>
```

**Required Fields**

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Unique achievement name (max 75 chars) |
| `description` | `string` | Short description (max 300 chars) |
| `tags` | `JSON array` | Array of string tags |
| `type` | `string` | Achievement category |
| `has_vc` | `boolean` | `true`/`false` or `"true"`/`"false"` in form data |

**Optional Fields**

| Field | Type | Description |
|---|---|---|
| `icon` | `string` or file | URL or uploaded image file |
| `level_id` | `UUID` | FK to a `Level` object |
| `template_id` | `string` | Qseverse VC template ID |

**Icon File Constraints**
- Allowed types: `jpg`, `jpeg`, `png`, `gif`, `webp`, `svg`
- Max size: **5 MB**
- Saved to: `MEDIA_ROOT/achievements/icons/<uuid>.<ext>`

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement 'First Steps' created successfully!",
  "response": {}
}
```

**Errors**

| Condition | Message |
|---|---|
| Missing required fields | `"Missing required fields: name, description"` |
| Duplicate name | `"Name already exists"` |
| Invalid `level_id` | `"Invalid level_id"` |
| Invalid icon type | `"Invalid file type. Allowed: jpg, jpeg, png, gif, webp, svg"` |
| Icon too large | `"File size exceeds 5MB limit"` |

---

### 5.2 `PUT /update/<achievement_id>/`

Update an existing achievement (partial updates supported). Same `multipart` / JSON content types as create.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `achievement_id` | `UUID` | Achievement to update |

**Request**
```http
PUT /api/v1/dashboard/achievement/update/550e8400-e29b-41d4-a716-446655440000/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "description": "Updated description",
  "has_vc": false,
  "tags": ["advanced", "milestone"]
}
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement updated successfully",
  "response": {}
}
```

**Errors**

| Condition | Message |
|---|---|
| Achievement not found | `"Achievement not found"` |
| Invalid `level_id` | `"Invalid level_id"` |
| Validation errors | `{"field_name": ["error message"]}` |

---

### 5.3 `DELETE /delete/<achievement_id>/`

Permanently delete an achievement and all associated data.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `achievement_id` | `UUID` | Achievement to delete |

**Request**
```http
DELETE /api/v1/dashboard/achievement/delete/550e8400-e29b-41d4-a716-446655440000/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement deleted successfully",
  "response": {}
}
```

> ⚠️ **Warning:** This is a hard delete. All `UserAchievementsLog`, `AchievementRule`, and `AchievementAuditLog` records linked to this achievement will also be cascade-deleted.

---

## 6. Admin — Rule Management

### 6.1 `GET /rules/`

List all achievement rules across all achievements.

**Request**
```http
GET /api/v1/dashboard/achievement/rules/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "id": "rule-uuid-1",
      "achievement_id": "achievement-uuid",
      "achievement_name": "First Steps",
      "version": 2,
      "rule_type": "ig_karma",
      "conditions": {
        "ig_id": "ig-uuid",
        "required_karma": 500
      },
      "is_active": true,
      "created_at": "2024-03-01T12:00:00Z"
    },
    {
      "id": "rule-uuid-2",
      "achievement_id": "achievement-uuid",
      "achievement_name": "First Steps",
      "version": 1,
      "rule_type": "ig_karma",
      "conditions": {
        "ig_id": "ig-uuid",
        "required_karma": 300
      },
      "is_active": false,
      "created_at": "2024-01-15T10:30:00Z"
    }
  ]
}
```

---

### 6.2 `POST /rules/create/`

Create a new rule for an achievement. **Automatically deactivates** the current active rule for the same achievement and assigns the next version number.

**Request**
```http
POST /api/v1/dashboard/achievement/rules/create/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
  "rule_type": "ig_karma",
  "conditions": {
    "ig_id": "ig-uuid-here",
    "required_karma": 500
  }
}
```

**Required Fields**

| Field | Type | Description |
|---|---|---|
| `achievement_id` | `UUID` | Parent achievement |
| `rule_type` | `enum` | One of: `ig_karma`, `skill`, `streak`, `milestone`, `event`, `task_completion` |
| `conditions` | `JSON object` | Rule-type-specific parameters (see §3) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Rule v3 created successfully",
  "response": {
    "rule_id": "new-rule-uuid",
    "version": 3
  }
}
```

**Behaviour**
- Version is auto-calculated as `max(existing_versions) + 1`.
- All previously active rules for `achievement_id` are set `is_active = false`.
- New rule is created with `is_active = true`.

---

### 6.3 `GET /rules/<rule_id>/`

Get full details of a single rule.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `rule_id` | `UUID` | Rule identifier |

**Request**
```http
GET /api/v1/dashboard/achievement/rules/rule-uuid-1/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "id": "rule-uuid-1",
    "achievement_id": "achievement-uuid",
    "achievement_name": "First Steps",
    "version": 2,
    "rule_type": "ig_karma",
    "conditions": {
      "ig_id": "ig-uuid",
      "required_karma": 500
    },
    "is_active": true,
    "created_at": "2024-03-01T12:00:00Z"
  }
}
```

---

### 6.4 `POST /rules/<rule_id>/deactivate/`

Deactivate a specific rule. The achievement will then have **no active rule**, meaning no one can claim it until a new rule is created or this one is reactivated.

**Request**
```http
POST /api/v1/dashboard/achievement/rules/rule-uuid-1/deactivate/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Rule v2 deactivated",
  "response": {}
}
```

---

### 6.5 `POST /rules/<rule_id>/activate/`

Activate a previously deactivated rule.

> **Note:** This does **not** deactivate other rules. Use with care — multiple active rules per achievement could exist if you activate an old version while a new one is also active.

**Request**
```http
POST /api/v1/dashboard/achievement/rules/rule-uuid-2/activate/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Rule v1 activated",
  "response": {}
}
```

**Error (already active)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Rule v1 is already active",
  "response": {}
}
```

---

## 7. Admin — Simulation & Debugging

### 7.1 `GET /simulate/<muid>/`

Simulate rule evaluation for any user (by `muid`) without issuing anything. Shows all active rule results including not-yet-eligible ones.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `muid` | `string` | Target user's muid |

**Request**
```http
GET /api/v1/dashboard/achievement/simulate/john@mulearn/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "achievement_id": "achievement-uuid-1",
      "achievement_name": "First Steps",
      "eligible": true,
      "reason": "IG Karma: 600/500",
      "progress": {
        "current": 600,
        "required": 500,
        "percentage": 100,
        "ig_id": "ig-uuid"
      }
    },
    {
      "achievement_id": "achievement-uuid-2",
      "achievement_name": "Streak Master",
      "eligible": false,
      "reason": "Already claimed",
      "progress": {
        "current": 30,
        "required": 30,
        "percentage": 100,
        "streak_type": "daily_task"
      }
    },
    {
      "achievement_id": "achievement-uuid-3",
      "achievement_name": "Karma King",
      "eligible": false,
      "reason": "total_karma: 350/1000",
      "progress": {
        "current": 350,
        "required": 1000,
        "percentage": 35,
        "milestone_type": "total_karma"
      }
    }
  ]
}
```

---

### 7.2 `GET /debug/<muid>/<achievement_id>/`

Deep-debug a specific achievement for a user. Returns evaluation result plus raw user aggregate data for comparison.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `muid` | `string` | Target user's muid |
| `achievement_id` | `UUID` | Achievement to debug |

**Request**
```http
GET /api/v1/dashboard/achievement/debug/john@mulearn/550e8400-e29b-41d4-a716-446655440000/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "evaluation": {
      "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
      "achievement_name": "First Steps",
      "eligible": true,
      "reason": "IG Karma: 600/500",
      "progress": {
        "current": 600,
        "required": 500,
        "percentage": 100,
        "ig_id": "ig-uuid"
      }
    },
    "user_data": {
      "ig_karma": [
        {
          "ig_id": "ig-uuid",
          "total_karma": 600,
          "task_count": 24
        }
      ],
      "streaks": [
        {
          "streak_type": "daily_task",
          "current_streak": 15,
          "longest_streak": 30
        },
        {
          "streak_type": "daily_login",
          "current_streak": 10,
          "longest_streak": 25
        }
      ],
      "skill_progress": [
        {
          "skill_id": "skill-uuid",
          "completed_task_count": 8,
          "total_karma": 400
        }
      ]
    }
  }
}
```

**Error (no active rule)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "No active rule found for this achievement"
}
```

---

## 8. Admin — Manual Operations

### 8.1 `POST /manual-issue/`

Issue an achievement to a user **bypassing all rule checks**. Recorded with `source: "admin_manual"` in the audit log.

**Request**
```http
POST /api/v1/dashboard/achievement/manual-issue/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "muid": "john@mulearn",
  "achievement_id": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Required Fields**

| Field | Type | Description |
|---|---|---|
| `muid` | `string` | Target user's muid |
| `achievement_id` | `UUID` | Achievement to issue |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement 'First Steps' issued to user",
  "response": {}
}
```

**Response — 400 (Already Issued)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Achievement already issued or failed",
  "response": {}
}
```

**Audit Log Entry Created:**
```json
{
  "action": "issued",
  "rule_version": 0,
  "metadata": { "source": "admin_manual" },
  "performed_by": "<admin-user-id>"
}
```

---

### 8.2 `POST /revoke/`

Revoke a previously issued achievement. Deletes the `UserAchievementsLog` record and creates an audit entry.

**Request**
```http
POST /api/v1/dashboard/achievement/revoke/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "muid": "john@mulearn",
  "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
  "reason": "Duplicate issuance error"
}
```

**Fields**

| Field | Type | Required | Description |
|---|---|---|---|
| `muid` | `string` | Yes | Target user's muid |
| `achievement_id` | `UUID` | Yes | Achievement to revoke |
| `reason` | `string` | No | Reason for revocation (logged in audit) |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement revoked successfully",
  "response": {}
}
```

**Response — 400 (Not Found)**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Achievement not found for user",
  "response": {}
}
```

**Audit Log Entry Created:**
```json
{
  "action": "revoked",
  "metadata": { "reason": "Duplicate issuance error" },
  "performed_by": "<admin-user-id>"
}
```

> ⚠️ After revocation, the user can re-claim the achievement if they are still eligible (since the `UserAchievementsLog` record is deleted).

---

## 9. Admin — Audit & Logs

### 9.1 `GET /audit/<muid>/`

Retrieve the last 100 audit log entries for a user's achievements, ordered by most recent.

**Path Parameters**

| Parameter | Type | Description |
|---|---|---|
| `muid` | `string` | Target user's muid |

**Request**
```http
GET /api/v1/dashboard/achievement/audit/john@mulearn/
Authorization: Bearer <token>
```

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "id": "audit-log-uuid",
      "achievement_id": "achievement-uuid",
      "achievement_name": "First Steps",
      "action": "issued",
      "rule_version": 2,
      "metadata": {
        "source": "user_claim"
      },
      "performed_by": null,
      "created_at": "2024-03-15T14:22:00Z"
    },
    {
      "id": "audit-log-uuid-2",
      "achievement_id": "achievement-uuid",
      "achievement_name": "First Steps",
      "action": "vc_issued",
      "rule_version": null,
      "metadata": {
        "vc_url": "https://qseverse.com/vc/abc123"
      },
      "performed_by": null,
      "created_at": "2024-03-15T14:25:00Z"
    }
  ]
}
```

**Audit `action` Values**

| Action | Triggered By |
|---|---|
| `issued` | User claim or admin manual issue or bulk sync |
| `revoked` | Admin revoke |
| `vc_issued` | Successful VC generation via Qseverse |
| `vc_failed` | Failed VC generation attempt (with retry count) |

---

### 9.2 `GET /issued-log/`

Paginated list of all `UserAchievementsLog` records across all users. Supports search and sorting.

**Query Parameters**

| Parameter | Type | Description |
|---|---|---|
| `search` | `string` | Searches `muid`, `first_name`, `achievement name` |
| `sort_by` | `string` | Field to sort by (default: `created_at`) |
| `sort_order` | `string` | `asc` or `desc` |
| `page` | `integer` | Page number |
| `page_size` | `integer` | Items per page |

**Request**
```http
GET /api/v1/dashboard/achievement/issued-log/?search=john&page=1&page_size=20
Authorization: Bearer <token>
```

**Response — 200 OK (paginated)**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "data": [
      {
        "id": "log-uuid",
        "muid": "john@mulearn",
        "user_name": "John Doe",
        "achievement_name": "First Steps",
        "is_issued": true,
        "created_at": "2024-03-15T14:22:00Z",
        "issued_by": "Admin User"
      }
    ],
    "pagination": {
      "count": 150,
      "next": "http://...",
      "previous": null
    }
  }
}
```

---

## 10. VC (Verifiable Credential) Endpoints

### 10.1 `POST /issue-vc/`

Associate a Verifiable Credential URL with the user's achievement log. Typically called **by the frontend** after the user initiates VC issuance through Qseverse.

**Request**
```http
POST /api/v1/dashboard/achievement/issue-vc/
Authorization: Bearer <token>
Content-Type: application/json
```
```json
{
  "achievement_id": "550e8400-e29b-41d4-a716-446655440000",
  "vc_url": "https://qseverse.com/vc/abc123xyz"
}
```

**Required Fields**

| Field | Type | Description |
|---|---|---|
| `achievement_id` | `UUID` | Achievement to update |
| `vc_url` | `string` | URL of the issued VC |

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Achievement issued successfully",
  "response": {}
}
```

**Errors**

| Condition | Message |
|---|---|
| No `achievement_id` | `"Achievement ID is required"` |
| No `vc_url` | `"VC URL is required"` |
| No matching log | `"Achievement record not found"` |
| Already issued | `"This achievement has already been issued"` |

> **Note:** This endpoint is idempotent in the sense that calling it a second time returns an error if `vc_url` is already set.

---

## 11. Bulk Operations

### 11.1 `POST /bulk-issue/`

Upload an Excel file to issue achievements to multiple users at once.

**Request**
```http
POST /api/v1/dashboard/achievement/bulk-issue/
Authorization: Bearer <token>
Content-Type: multipart/form-data

excel_file=<.xlsx file>
```

**Excel File Format**

| Column | Description |
|---|---|
| `muid` | User's muid string |
| `achievement_id` | Achievement UUID |

Download the template from [`GET /bulk-issue/template/`](#112-get-bulk-issuetemplate).

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Bulk issue processing completed",
  "response": {
    "success_count": 45,
    "failed_count": 3,
    "failed_rows": [
      {
        "row": 5,
        "muid": "invalid@user",
        "reason": "User not found"
      },
      {
        "row": 12,
        "muid": "jane@mulearn",
        "reason": "Achievement already issued or failed"
      },
      {
        "row": 23,
        "muid": "bob@mulearn",
        "reason": "Achievement already issued or failed"
      }
    ]
  }
}
```

**Constraints**
- Rows with blank `muid` or `achievement_id` are silently skipped.
- Each row is processed independently; failures do not stop the batch.
- Uses the same `manual_issue_achievement` function as the single manual-issue endpoint.

---

### 11.2 `GET /bulk-issue/template/`

Download an empty Excel template with the required column headers.

**Request**
```http
GET /api/v1/dashboard/achievement/bulk-issue/template/
Authorization: Bearer <token>
```

**Response**
```
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=achievement_bulk_import_template.xlsx
```

The downloaded file contains one row of headers: `muid`, `achievement_id`.

---

### 11.3 `POST /bulk-claim/`

Trigger a background Celery job to auto-issue all eligible achievements for users who were active within a date range. Intended for scheduled/admin use.

**Authentication:** Requires **Backend API Key** (`BackendApiKeyPermission`) — **not** a JWT.

**Request**
```http
POST /api/v1/dashboard/achievement/bulk-claim/
X-API-Key: <backend-api-key>
Content-Type: application/json
```
```json
{
  "date_from": "2024-03-01",
  "date_to": "2024-03-31"
}
```

**Fields**

| Field | Type | Required | Default | Description |
|---|---|---|---|---|
| `date_from` | `ISO date string` | No | Yesterday | Start of active user range |
| `date_to` | `ISO date string` | No | Yesterday | End of active user range |

**Constraints**
- `date_from` must be ≤ `date_to`, otherwise returns 400.
- Dates must be valid ISO format (`YYYY-MM-DD`), otherwise returns 400.
- The job runs **asynchronously** via Celery. The API returns immediately.

**Response — 200 OK**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Bulk sync job scheduled successfully.",
  "response": {}
}
```

**What the background job does:**
1. Finds all unique users with approved `KarmaActivityLog` entries in the date range.
2. For each user, runs `RuleEvaluator.get_eligible_achievements()`.
3. Issues any eligible, unclaimed achievements.
4. Logs all issuances to the audit trail with `source: "bulk_sync"`.

---

## 12. Event System (Internal)

The event system is an internal mechanism — **not exposed as HTTP endpoints**. Other parts of the application call these functions to feed the achievement system.

### Event Types

| Constant | Value | Triggered When |
|---|---|---|
| `TASK_COMPLETED` | `task.completed` | User completes & appraiser approves a task |
| `KARMA_AWARDED` | `karma.awarded` | Karma is added to a user's wallet |
| `USER_LOGIN` | `user.login` | User logs in (once per day idempotency) |
| `DISCORD_ACTIVITY` | `discord.activity` | Discord activity recorded |
| `EVENT_ATTENDED` | `event.attended` | User attends an event |
| `LC_MEETING_ATTENDED` | `lc.meeting.attended` | User attends an LC meeting |
| `STREAK_MILESTONE` | `streak.milestone` | Streak cron emits this at milestones |

### Event Metadata Schemas

**`task.completed`**
```json
{
  "task_id": "uuid",
  "ig_id": "uuid or null",
  "karma": 50,
  "skill_ids": ["skill-uuid-1", "skill-uuid-2"]
}
```

**`karma.awarded`**
```json
{
  "karma": 100,
  "source": "task | referral | bonus",
  "ig_id": "uuid or null",
  "task_id": "uuid or null"
}
```

**`user.login`**
```json
{
  "date": "2024-03-15"
}
```

**`event.attended`**
```json
{
  "event_name": "MuLearn Summit 2024"
}
```

**`lc.meeting.attended`**
```json
{
  "circle_id": "uuid",
  "meeting_id": "uuid"
}
```

**`streak.milestone`**
```json
{
  "streak_type": "daily_task | daily_login",
  "streak_count": 30
}
```

### Idempotency

Each event function uses a deterministic `event_id` as an idempotency key:

| Event | Idempotency Key Pattern |
|---|---|
| `task.completed` | `task_{task_id}_{user_id}` |
| `user.login` | `login_{user_id}_{YYYY-MM-DD}` |
| `event.attended` | `event_{event_id}_{user_id}` |
| `lc.meeting.attended` | `lc_{meeting_id}_{user_id}` |
| `streak.milestone` | `streak_{user_id}_{streak_type}_{count}` |

Duplicate events with the same key are **silently dropped**.

---

## 13. Background Tasks & Cron Jobs

### Celery Tasks

| Task | Module | Type | Description |
|---|---|---|---|
| `process_achievement_event` | `mu_celery.achievement_tasks` | Async (retries ×3) | Updates aggregate tables from an event |
| `issue_vc_async` | `mu_celery.achievement_tasks` | Async (retries ×5) | Issues VC via Qseverse API |
| `bulk_check_and_issue_achievements` | `mu_celery.achievement_tasks` | Async (retries ×3) | Bulk-issue for date range |
| `evaluate_daily_streaks` | `mu_celery.achievement_cron` | Cron — 00:05 UTC daily | Updates all user streaks |
| `backfill_missing_aggregates` | `mu_celery.achievement_cron` | Cron — Sundays 02:00 UTC | Reconciles `UserIgKarma` with source data |
| `integrity_check` | `mu_celery.achievement_cron` | Cron — 03:00 UTC daily | Checks for missing audit logs & duplicates |

### Recommended Celery Beat Configuration

```python
from celery.schedules import crontab

CELERY_BEAT_SCHEDULE = {
    'evaluate-daily-streaks': {
        'task': 'mu_celery.achievement_cron.evaluate_daily_streaks',
        'schedule': crontab(hour=0, minute=5),  # 00:05 UTC daily
    },
    'backfill-aggregates': {
        'task': 'mu_celery.achievement_cron.backfill_missing_aggregates',
        'schedule': crontab(hour=2, minute=0, day_of_week=0),  # Sundays 02:00 UTC
    },
    'integrity-check': {
        'task': 'mu_celery.achievement_cron.integrity_check',
        'schedule': crontab(hour=3, minute=0),  # 03:00 UTC daily
    },
}
```

---

## 14. Common Response Format

All API responses follow this envelope:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Human-readable message or {}",
  "response": { ... }
}
```

### Paginated Response Format

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "data": [ ... ],
    "pagination": {
      "count": 150,
      "next": "http://...?page=2",
      "previous": null
    }
  }
}
```

---

## 15. Error Reference

| HTTP Code | `hasError` | When |
|---|---|---|
| `200` | `false` | Success |
| `400` | `true` | Validation error, not eligible, missing fields, already exists |
| `401` | `true` | Missing or invalid JWT |
| `404` | `true` | Resource not found (user, achievement, rule) |
| `500` | `true` | Unexpected server error |

### Common Error Messages

| Message | Cause |
|---|---|
| `"Invalid or missing token"` | JWT absent or expired |
| `"User Not Exists"` | User from JWT not found in DB |
| `"Achievement not found"` | Invalid `achievement_id` in path |
| `"Rule not found"` | Invalid `rule_id` in path |
| `"No active rule for this achievement"` | Achievement has no `is_active = true` rule |
| `"Achievement already claimed"` | `UserAchievementsLog` record exists |
| `"Not eligible: <reason>"` | Rule engine returned `eligible = false` |
| `"Missing required fields: <list>"` | Request body missing required keys |
| `"Name already exists"` | Duplicate achievement name on create |
| `"Invalid level_id"` | FK to `Level` not found |
| `"File size exceeds 5MB limit"` | Icon upload too large |
| `"Invalid date format. Use ISO format (YYYY-MM-DD)."` | Bad date string in bulk-claim |
| `"Invalid date range: date_from cannot be after date_to."` | Logical date range error |
