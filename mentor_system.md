# Mentor System — Complete API Reference

**For frontend integration.** Covers every endpoint across the mentor domain:
mentor application/profile, verification, scope grants, admin assignment,
sessions, availability, participants, tasks, student session requests,
Company Mentor nomination, and Campus Mentor nomination.

**Base path:** `/api/v1/`
**Source:** `api/dashboard/mentor/`, `api/dashboard/company/` (mentor nomination only), `api/dashboard/campus/` (mentor nomination only)

---

## Table of Contents

1. [Conventions](#1-conventions)
2. [Mentor Registration & Profile](#2-mentor-registration--profile)
3. [Mentor Verification](#3-mentor-verification)
4. [Mentor Scope Grants](#4-mentor-scope-grants)
5. [Admin Bulk Assignment](#5-admin-bulk-assignment)
6. [Mentor Sessions](#6-mentor-sessions)
7. [Mentor Availability](#7-mentor-availability)
8. [Session Participants](#8-session-participants)
9. [Mentor Tasks](#9-mentor-tasks)
10. [Student Session Requests](#10-student-session-requests)
11. [Company Mentor Nomination](#11-company-mentor-nomination)
12. [Campus Mentor Nomination](#12-campus-mentor-nomination)
13. [Endpoint Summary Table](#13-endpoint-summary-table)
14. [Known Limitations](#14-known-limitations)

---

## 1. Conventions

### 1.1 Authentication

All endpoints require:

```http
Authorization: Bearer <jwt_access_token>
Content-Type: application/json
```

except those explicitly marked **Public** below (public profile/availability
lookups still require *a* valid JWT — "Public" here means no specific role is
required, not that the endpoint is unauthenticated).

### 1.2 Response envelope

Every response is wrapped identically. **`message.general` is always an
array**, even for a single message string:

**Success:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Human-readable success message"] },
  "response": {}
}
```

**Failure:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Error summary"],
    "field_name": ["Field-specific validation detail"]
  },
  "response": {}
}
```

### 1.3 Pagination

Applies to every endpoint marked **paginated** below.

**Query params:**

| Param | Default | Notes |
|---|---|---|
| `pageIndex` | `1` | 1-based page number |
| `perPage` | `10` | Items per page |
| `search` | — | Case-insensitive substring match over that endpoint's documented search fields |
| `sortBy` | — | One of that endpoint's documented sort fields; prefix `-` for descending |

**Response shape** (inside `response`):
```json
{
  "data": [ /* array of items */ ],
  "pagination": {
    "count": 42,
    "totalPages": 5,
    "isNext": true,
    "isPrev": false,
    "nextPage": 2
  }
}
```

### 1.4 Roles

| Role | Meaning |
|---|---|
| `Admin` | Platform admin |
| `Mentor` | Auto-granted the first time any `UserMentor` tier is approved for a user; required by most mentor-self-service endpoints |
| `Company` | Held only by the `company_user` who registered a verified `Company` |
| `CampusLead` / `LeadEnabler` | Campus staff roles |

### 1.5 Mentor tiers (`UserMentor.mentor_tier`)

| Tier | Meaning | Has `org`? |
|---|---|---|
| `MENTOR` | Platform-wide global mentor (admin-level visibility into student requests/events) | No |
| `IG_MENTOR` | Interest-Group-scoped mentor | No — IG scope lives in `UserIgLink` / `MentorScopeGrant`, one row per IG |
| `COMPANY_MENTOR` | Scoped to one Company `Organization` | Yes |
| `CAMPUS_MENTOR` | Scoped to one College `Organization` | Yes |

A user can hold **multiple** `UserMentor` rows simultaneously (one per tier)
— e.g. an `IG_MENTOR` later approved as `COMPANY_MENTOR` keeps both, and can
act in either capacity.

### 1.6 Mentor scope grants (`MentorScopeGrant`)

The enforced source of mentor authority: `(scope_type, scope_id, is_active)`
per mentor. Created automatically on every approval (`/mentor/verify/`,
`/mentor/admin/assign/`, IG-edit mentor assignment). `scope_id` is the org id
for `COMPANY_MENTOR`/`CAMPUS_MENTOR`, the IG id for `IG_MENTOR` (one grant
per IG), and `null` for the global `MENTOR` tier. This is what gates access
to company resources, campus dashboards, campus sessions, student
mentor-request visibility, and event creation/approval. IG-scoped session/
task/availability checks read `UserIgLink` directly (kept in sync with these
grants).

### 1.7 Session lifecycle (`MentorshipSession.status`)

| Status | Meaning |
|---|---|
| `REQUESTED` | Created by a student via §10.1; awaiting a mentor's decision |
| `PENDING_APPROVAL` | Legacy pre-moderation state |
| `SCHEDULED` | Live — visible to learners, joinable |
| `COMPLETED` | Finished |
| `CANCELLED` | Cancelled after being scheduled |
| `REJECTED` | Rejected |

Mentor-created sessions (§6.1) are **auto-`SCHEDULED`** on creation — no
separate approval step. Student-requested sessions (§10.1) start
`REQUESTED` and only become `SCHEDULED` once a mentor approves (§10.4).

---

## 2. Mentor Registration & Profile

### 2.1 Submit mentor application

`POST /mentor/register/`

Self-service application. **Always creates an `IG_MENTOR`-tier row** —
there's no way to self-apply for `COMPANY_MENTOR`/`CAMPUS_MENTOR`/`MENTOR`
tiers (those come from nomination or admin assignment, see §5, §11, §12).

**Roles:** any authenticated user (one application total per user — see
constraints)

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `about` | string | No | Max 1000 chars |
| `expertise` | string | No | Free text |
| `reason` | string | No | Max 1000 chars |
| `hours` | integer | No | Default `0` |
| `preferred_ig_ids` | array[uuid] | **Yes** | Non-empty; each id must be a valid `InterestGroup` |

```json
{
  "about": "Software engineer with 8 years of experience mentoring students.",
  "expertise": "Python, Django, system design, career guidance",
  "reason": "I want to give back to the muLearn community.",
  "hours": 5,
  "preferred_ig_ids": ["8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e"]
}
```

**Constraints:**
- If the caller already has a `UserMentor` row of any status, this returns
  `400` — one application total; use §2.2 to update or resubmit.
- `preferred_ig_ids` empty or containing an unknown IG id → `400`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentor registration submitted successfully."] },
  "response": {
    "about": "Software engineer with 8 years of experience mentoring students.",
    "expertise": "Python, Django, system design, career guidance",
    "reason": "I want to give back to the muLearn community.",
    "hours": 5,
    "preferred_ig_ids": ["8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e"]
  }
}
```

**Error — duplicate application (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["A mentor request already exists for your account."] },
  "response": {}
}
```

**Error — validation (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": [],
    "preferred_ig_ids": ["At least one preferred IG ID must be provided."]
  },
  "response": {}
}
```

---

### 2.2 Update / resubmit application

`PATCH /mentor/register/`

**Roles:** the application's owner

**Request body:** any subset of `about`, `expertise`, `reason`, `hours`,
`preferred_ig_ids` (partial update).

**Business rules:**
- `404` if no application exists for the caller.
- `400` if `status == APPROVED` — "already approved, use profile endpoint instead."
- If `status == REJECTED`: on success, resets `status → PENDING` and clears
  `verification_note`, so it re-enters the review queue.
- If still `PENDING`: plain field update, status unchanged.

**Success response — resubmission (`200`):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentor registration updated and resubmitted successfully."] },
  "response": {
    "about": "Updated bio text.",
    "expertise": "Python, Django, system design, career guidance",
    "reason": "I want to give back to the muLearn community.",
    "hours": 6,
    "preferred_ig_ids": ["8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e"]
  }
}
```

**Error — already approved (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Your mentor application is already approved. Please use the profile endpoint to update your details."] },
  "response": {}
}
```

**Error — no application (`404`, transport `400` per §1.2 caveat):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["No mentor registration request found for your account."] },
  "response": {}
}
```

---

### 2.3 Check own application status

`GET /mentor/status/`

**Roles:** any authenticated user

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "status": "PENDING",
    "organization": "Acme Corp",
    "verified_by": null,
    "verified_at": null
  }
}
```

`organization` resolves the caller's employer: `UserMentor.org` first,
falling back to their `UserOrganizationLink(org_type=Company)` — so it's
populated even for `IG_MENTOR` applicants who work at a company.

**Error — no application (`404`, see §1.2 transport caveat):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["No mentor request found for your account."] },
  "response": {}
}
```

> **Edge case:** if the caller holds multiple `UserMentor` rows (multi-tier
> mentor), this returns data for an *arbitrary* one (no ordering applied) —
> not fixed as of this document.

---

### 2.4 Get own mentor profile

`GET /mentor/profile/`

**Roles:** `Mentor`, and must have an `APPROVED` `UserMentor` row

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "id": "b6b1f6b0-6b1a-4c3e-9b0a-3e2f1a8c9d10",
    "user": "u-uuid-1234",
    "user_full_name": "Jane Doe",
    "user_email": "jane.doe@example.com",
    "about": "Software engineer with 8 years of experience mentoring students.",
    "expertise": "Python, Django, system design, career guidance",
    "reason": "I want to give back to the muLearn community.",
    "hours": 5,
    "mentor_tier": "IG_MENTOR",
    "status": "APPROVED",
    "preferred_ig_ids": ["8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e"],
    "org": null,
    "verified_by": "admin-uuid",
    "verified_at": "2026-06-01T09:12:00Z",
    "verification_note": null,
    "company": "Acme Corp",
    "updated_by": "admin-uuid",
    "updated_at": "2026-06-01T09:12:00Z",
    "created_by": "u-uuid-1234",
    "created_at": "2026-05-28T14:03:11Z"
  }
}
```
`company` is the employer-fallback field (same resolution as §2.3's
`organization`). All other fields are the raw `UserMentor` row (this
serializer exposes `fields = "__all__"`).

**Error — not approved / not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Mentor profile not found or not approved."] },
  "response": {}
}
```

---

### 2.5 Update own mentor profile

`PATCH /mentor/profile/`

The only way an already-approved mentor edits their own data — including
which Interest Groups they mentor, which **takes effect immediately, no
re-approval needed**.

**Roles:** `Mentor` (approved)

**Request body:** any subset of `about`, `expertise`, `reason`, `hours`,
`preferred_ig_ids`.

```json
{
  "hours": 8,
  "preferred_ig_ids": [
    "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
    "1c2d3e4f-5678-90ab-cdef-1234567890ab"
  ]
}
```

**Constraints:**
- If `preferred_ig_ids` is included, it **cannot be an empty array** —
  `400` "You must mentor at least one Interest Group."
- Each id must be a valid IG (invalid ids are silently dropped from the
  effective set, not rejected — the reconciler only intersects with real IGs).

**Business rule:** if `preferred_ig_ids` changes, both `UserIgLink` (the
permission table sessions/tasks/availability check) and the mentor's
`MentorScopeGrant(IG_MENTOR)` rows are reconciled to exactly match the new
list — additions activated/created, removals deactivated. Any
Company/Campus grant this mentor also holds is untouched.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentor profile updated successfully."] },
  "response": {
    "id": "b6b1f6b0-6b1a-4c3e-9b0a-3e2f1a8c9d10",
    "user_full_name": "Jane Doe",
    "user_email": "jane.doe@example.com",
    "about": "Software engineer with 8 years of experience mentoring students.",
    "expertise": "Python, Django, system design, career guidance",
    "hours": 8,
    "mentor_tier": "IG_MENTOR",
    "status": "APPROVED",
    "preferred_ig_ids": [
      "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
      "1c2d3e4f-5678-90ab-cdef-1234567890ab"
    ],
    "company": "Acme Corp"
  }
}
```

**Error — clearing IGs (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "preferred_ig_ids": ["You must mentor at least one Interest Group."] },
  "response": {}
}
```

---

### 2.6 List all mentor applications (Admin)

`GET /mentor/list/` — **paginated**

**Roles:** `Admin`

**Query params:** `status` (`PENDING`/`APPROVED`/`REJECTED`), `mentor_tier`,
plus §1.3 pagination params. Search fields: `user__full_name`,
`user__email`. Sort fields: `created_at`, `status`, `user_full_name`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "b6b1f6b0-6b1a-4c3e-9b0a-3e2f1a8c9d10",
        "user_id": "u-uuid-1234",
        "user_full_name": "Jane Doe",
        "user_email": "jane.doe@example.com",
        "muid": "jane-doe@mulearn",
        "about": "Software engineer...",
        "expertise": "Python, Django, system design",
        "verification_note": null,
        "verified_at": null,
        "mentor_tier": "IG_MENTOR",
        "status": "PENDING",
        "created_at": "2026-05-28T14:03:11Z",
        "updated_at": "2026-05-28T14:03:11Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

### 2.7 Get mentor detail by id (Admin)

`GET /mentor/detail/<mentor_id>/`

**Roles:** `Admin`

**Path params:** `mentor_id` — `UserMentor.id` (UUID)

**Success response — `200`:** same shape as §2.4's `response` (full
`UserMentor` fields + `company`), for any mentor id.

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Mentor not found."] },
  "response": {}
}
```

---

### 2.8 Public mentor profile

`GET /mentor/public/profile/<mentor_id>/`

**Roles:** any authenticated user (**Public** — no specific role required)

**Path params:** `mentor_id` — `UserMentor.id`

**Business rule:** only `status=APPROVED` mentors are returned.

**Success response — `200`:** same shape as §2.4.

**Error — not found/not approved (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Mentor profile not found or not approved."] },
  "response": {}
}
```

---

## 3. Mentor Verification

### 3.1 Approve or reject a mentor application

`PATCH /mentor/verify/<mentor_id>/`

**Roles:** `Admin`, **OR** the `Company.company_user` who owns the verified
company a `COMPANY_MENTOR` application is scoped to (checked by matching
`Company.name == mentor.org.title`). All other tiers (`MENTOR`/`IG_MENTOR`/
`CAMPUS_MENTOR`) — **Admin only**.

**Path params:** `mentor_id` — `UserMentor.id`

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `status` | enum | Yes | `APPROVED` or `REJECTED` |
| `verification_note` | string | Conditional | **Required** if `status=REJECTED` |

```json
{
  "status": "APPROVED"
}
```
or
```json
{
  "status": "REJECTED",
  "verification_note": "Insufficient mentoring experience described in application."
}
```

**Business rules on `APPROVED`:**
1. A `MentorScopeGrant` is created for the approved tier (`scope_id = org_id`
   for `COMPANY_MENTOR`/`CAMPUS_MENTOR`, `null` for global `MENTOR`; **not**
   created here for `IG_MENTOR` — that tier's grants are always per-IG, from
   step 3).
2. The global `Mentor` role is granted (idempotent).
3. If `preferred_ig_ids` is set (any tier — IG mentoring is orthogonal to
   Company/Campus/global scope): `UserIgLink` rows and per-IG
   `MentorScopeGrant(IG_MENTOR)` rows are reconciled to match exactly.
4. If `mentor_tier == COMPANY_MENTOR` and `org` is set:
   `UserOrganizationLink(user, org)` is created/re-verified — the mentor's
   actual employment record.

**Business rules on `REJECTED`:** `verification_note` stored; no grants
touched.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentor status updated to APPROVED successfully."] },
  "response": {}
}
```

**Error — unauthorized (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You are not authorized to verify this mentor application."] },
  "response": {}
}
```

**Error — already approved (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Mentor is already approved."] },
  "response": {}
}
```

**Error — reject without note (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["Verification note is required when rejecting."] },
  "response": {}
}
```

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Mentor request not found."] },
  "response": {}
}
```

---

## 4. Mentor Scope Grants

### 4.1 List a mentor's grants

`GET /mentor/<mentor_id>/grants/`

**Roles:** `Admin`, or the `Company.company_user` who owns the company the
mentor's `COMPANY_MENTOR` grant is scoped to.

**Path params:** `mentor_id` — `UserMentor.id`

**Success response — `200`** (not paginated, newest-first):
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    {
      "id": "grant-uuid-1",
      "scope_type": "COMPANY_MENTOR",
      "scope_id": "org-uuid-acme",
      "is_active": true,
      "granted_by_name": "Admin User",
      "granted_at": "2026-06-01T09:12:00Z",
      "revoked_by_name": null,
      "revoked_at": null
    },
    {
      "id": "grant-uuid-2",
      "scope_type": "IG_MENTOR",
      "scope_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
      "is_active": false,
      "granted_by_name": "Jane Doe",
      "granted_at": "2026-05-15T10:00:00Z",
      "revoked_by_name": "Admin User",
      "revoked_at": "2026-06-10T11:00:00Z"
    }
  ]
}
```

**Error — not found (`404`) / unauthorized (`403`):** same shapes as §3.1.

---

### 4.2 Revoke a single grant

`DELETE /mentor/<mentor_id>/grants/<grant_id>/`

**Roles:** same as §4.1

**Path params:** `mentor_id` — `UserMentor.id`; `grant_id` — `MentorScopeGrant.id`

**Business rule:** deactivates only that grant (`is_active=False`,
`revoked_by`, `revoked_at` stamped). Every other grant this mentor holds,
and their `UserOrganizationLink` employment record, are **untouched** —
revoking authority never mutates identity. If the revoked grant's
`scope_type == IG_MENTOR`, the matching `UserIgLink(assignment_type=MENTOR)`
row for that IG is also deactivated, so the mentor actually loses session/
task/availability access for that specific IG (not just the audit row).

This is a **single-IG surgical revoke** — distinct from §5.2, which revokes
an entire tier at once.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Grant revoked successfully."] },
  "response": {}
}
```

**Error — grant not found / already inactive (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Active grant not found."] },
  "response": {}
}
```

---

## 5. Admin Bulk Assignment

### 5.1 Bulk-assign mentors

`POST /mentor/admin/assign/`

Assigns one or more users as mentors of one tier, **immediately
`APPROVED`** — no pending stage, no verification-note flow.

**Roles:** `Admin`

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `user_muids` | array[string] | Yes, ≥1 | Every muid must resolve to a non-suspended user |
| `mentor_tier` | enum | Yes | `MENTOR` / `IG_MENTOR` / `COMPANY_MENTOR` / `CAMPUS_MENTOR` |
| `org_id` | uuid | Conditional | **Required** for `COMPANY_MENTOR`/`CAMPUS_MENTOR`; must be an `Organization` of the matching type |
| `ig_ids` | array[uuid] | Conditional | **Required, non-empty** for `IG_MENTOR`; each must be a valid IG |
| `about` | string | No | |
| `expertise` | string | No | |
| `hours` | integer | No | Default `0` |

**Example — Company Mentor:**
```json
{
  "user_muids": ["jane-doe@mulearn", "john-smith@mulearn"],
  "mentor_tier": "COMPANY_MENTOR",
  "org_id": "org-uuid-acme",
  "about": "Engineering leads at Acme.",
  "hours": 4
}
```

**Example — IG Mentor with multiple IGs:**
```json
{
  "user_muids": ["jane-doe@mulearn"],
  "mentor_tier": "IG_MENTOR",
  "ig_ids": ["8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e", "1c2d3e4f-5678-90ab-cdef-1234567890ab"]
}
```

**Business rules:**
- Each user's `UserMentor(user, mentor_tier, org)` is created (or
  re-approved if it already existed) with `status=APPROVED`.
- A `MentorScopeGrant` is created for the tier (skipped for `IG_MENTOR`,
  whose grants come from the `ig_ids` loop).
- Global `Mentor` role granted.
- For **any** tier, if `ig_ids` is supplied: `UserIgLink(MENTOR)` +
  per-IG `MentorScopeGrant(IG_MENTOR)` are created/reactivated for each —
  a Company/Campus mentor can also be given IG mentoring capability in the
  same call.
- For `COMPANY_MENTOR`/`CAMPUS_MENTOR`: `UserOrganizationLink(user, org)`
  created/re-verified.
- **Idempotent** per user+tier+org — safe to re-run with the same input.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentors assigned successfully."] },
  "response": {
    "assigned_user_muids": ["jane-doe@mulearn", "john-smith@mulearn"]
  }
}
```

**Error — validation (`400`, field-keyed):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": [],
    "user_muids": ["The following muids are invalid or belong to suspended users: bad-user@mulearn"],
    "org_id": ["org_id is required for COMPANY_MENTOR."]
  },
  "response": {}
}
```

---

### 5.2 Revoke mentor assignment

`DELETE /mentor/admin/assign/<user_muid>/`

**Roles:** `Admin`

**Path params:** `user_muid`

**Query params:** `mentor_tier` (optional) — restricts revocation to a
single tier; omit to revoke **all** approved tiers for the user.

**Business rules** (per revoked `UserMentor` row):
1. `status → REJECTED`.
2. If tier was `IG_MENTOR`: **all** `UserIgLink(MENTOR)` rows for the user
   are deactivated (not scoped to one IG — this removes IG mentoring
   entirely; use §4.2 to remove a single IG grant instead).
3. All active `MentorScopeGrant` rows for that `UserMentor` deactivated.
4. `UserOrganizationLink.verified` is **never touched** — revoking mentor
   authority never unverifies employment/enrollment.
5. If no `APPROVED` `UserMentor` row remains for the user (any tier), the
   global `Mentor` role is stripped.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Mentor assignment revoked successfully."] },
  "response": {}
}
```

**Error — user not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["No user found with muid 'bad-user@mulearn'."] },
  "response": {}
}
```

**Error — nothing to revoke (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["No approved mentor records found to revoke."] },
  "response": {}
}
```

---

## 6. Mentor Sessions

Three session types exist: `ig_session`, `campus_session`, `company_session`
— only `ig_session` is creatable through §6.1 (there is no
mentor-self-service create endpoint for campus/company sessions).

### 6.1 Create a session

`POST /mentor/session/create/`

**Roles:** `Mentor`, and caller must have an active `UserIgLink(MENTOR)`
for the target IG.

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `ig` | uuid | **Yes** | Becomes `entity_id`; not part of the response body's own fields |
| `title` | string | Yes | Max 150 chars |
| `description` | string | No | |
| `mode` | enum | Yes | `ONLINE` / `OFFLINE` / `HYBRID` |
| `starts_at` | datetime (ISO 8601) | Yes | |
| `ends_at` | datetime (ISO 8601) | Yes | Must be after `starts_at` |
| `meeting_link` | string | Conditional | Required for `HYBRID`; **forbidden** for `OFFLINE` |
| `venue` | string | Conditional | Required for `HYBRID`; **forbidden** for `ONLINE` |
| `max_participants` | integer | No | |
| `is_recurring` | boolean | No, default `false` | |
| `recurrence_type` | enum | Conditional | `DAILY`/`WEEKLY`/`MONTHLY`, required if `is_recurring` |
| `recurrence_interval` | integer | Conditional | ≥1, required if `is_recurring` |
| `recurrence_end_date` | date | Conditional | Required if `is_recurring`; must be after `starts_at`'s date |

**Example — one-off:**
```json
{
  "ig": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
  "title": "React Hooks Deep Dive",
  "description": "Covering useEffect, useMemo, and custom hooks.",
  "mode": "HYBRID",
  "starts_at": "2026-08-01T14:00:00Z",
  "ends_at": "2026-08-01T15:30:00Z",
  "meeting_link": "https://meet.example.com/abc-defg-hij",
  "venue": "Auditorium 2",
  "max_participants": 30
}
```

**Example — recurring:**
```json
{
  "ig": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
  "title": "Weekly Office Hours",
  "mode": "ONLINE",
  "starts_at": "2026-08-04T10:00:00Z",
  "ends_at": "2026-08-04T11:00:00Z",
  "meeting_link": "https://meet.example.com/office-hours",
  "is_recurring": true,
  "recurrence_type": "WEEKLY",
  "recurrence_interval": 1,
  "recurrence_end_date": "2026-09-29"
}
```

**Business rule:** the session is **auto-`SCHEDULED`** on creation
(`approved_by` = creator) — no separate admin approval for mentor-initiated
sessions. If recurring, up to 50 child sessions are generated (capped),
each a full copy with shifted dates, linked via `parent_session_id`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session created successfully."] },
  "response": {
    "id": "session-uuid-1",
    "entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
    "session_type": "ig_session",
    "title": "Weekly Office Hours",
    "description": null,
    "mode": "ONLINE",
    "starts_at": "2026-08-04T10:00:00Z",
    "ends_at": "2026-08-04T11:00:00Z",
    "meeting_link": "https://meet.example.com/office-hours",
    "venue": null,
    "max_participants": null,
    "is_recurring": true,
    "recurrence_type": "WEEKLY",
    "recurrence_interval": 1,
    "recurrence_end_date": "2026-09-29",
    "child_session_ids": [
      "session-uuid-2", "session-uuid-3", "session-uuid-4"
    ]
  }
}
```

**Error — missing IG (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Select an Interest Group to create a session."] },
  "response": {}
}
```

**Error — not assigned mentor for IG (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You are not assigned as a mentor for this Interest Group."] },
  "response": {}
}
```

**Error — duplicate session (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["A session with this exact title and start time already exists."] },
  "response": {}
}
```

**Error — mode/venue conflict (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "venue": ["Venue must not be provided for an online session."] },
  "response": {}
}
```

---

### 6.2 List own sessions

`GET /mentor/session/list/` — **paginated**

**Roles:** `Mentor`; scope: `created_by = caller` only.

**Query params:** `status`, plus §1.3 params. Search: `title`,
`description`. Sort: `created_at`, `starts_at`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "session-uuid-1",
        "entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
        "entity_name": "Web Development",
        "session_type": "ig_session",
        "title": "React Hooks Deep Dive",
        "description": "Covering useEffect, useMemo, and custom hooks.",
        "mode": "HYBRID",
        "starts_at": "2026-08-01T14:00:00Z",
        "ends_at": "2026-08-01T15:30:00Z",
        "status": "SCHEDULED",
        "created_by_id": "u-uuid-1234",
        "created_by_name": "Jane Doe",
        "created_at": "2026-07-15T09:00:00Z",
        "max_participants": 30,
        "meeting_link": "https://meet.example.com/abc-defg-hij",
        "venue": "Auditorium 2",
        "is_recurring": false,
        "parent_session_id": null,
        "recurrence_type": null,
        "recurrence_interval": null,
        "recurrence_end_date": null
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

### 6.3 Get session detail

`GET /mentor/session/list/<session_id>/`

**Roles:** `Mentor`; owned sessions only.

**Path params:** `session_id`

**Success response — `200`:** same object shape as one item from §6.2.

**Error — not found/not owned (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Session not found."] },
  "response": {}
}
```

---

### 6.4 Update a session

`PATCH /mentor/session/update/<session_id>/`

**Roles:** `Mentor`; owned sessions only.

**Path params:** `session_id`

**Request body:** any subset of `title`, `description`, `mode`, `starts_at`,
`ends_at`, `meeting_link`, `venue`, `max_participants`.

```json
{
  "starts_at": "2026-08-01T15:00:00Z",
  "ends_at": "2026-08-01T16:30:00Z",
  "max_participants": 40
}
```

**Constraint:** cannot edit a session whose `status` is `COMPLETED`,
`CANCELLED`, or `REJECTED`. Editing **does not** reset status (the API's own
success message says "Status reset to pending if previously scheduled" —
that text is stale/inaccurate; the status is left as-is).

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session updated successfully. Status reset to pending if previously scheduled."] },
  "response": {
    "id": "session-uuid-1",
    "title": "React Hooks Deep Dive",
    "starts_at": "2026-08-01T15:00:00Z",
    "ends_at": "2026-08-01T16:30:00Z",
    "max_participants": 40
  }
}
```

**Error — uneditable status (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Cannot edit a session that is COMPLETED."] },
  "response": {}
}
```

---

### 6.5 Delete a session

`DELETE /mentor/session/update/<session_id>/`

**Roles:** `Mentor`; owned sessions only.

**Path params:** `session_id`

**Business rule:** soft-delete (`is_deleted=True`). **No status guard** — an
in-progress or completed session can still be deleted.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session deleted successfully."] },
  "response": {}
}
```

---

### 6.6 List joinable sessions

`GET /mentor/session/available/` — **paginated**

**Roles:** any authenticated user (**no role restriction**)

Lists IG sessions for IGs the caller has *any* `UserIgLink` for (not
restricted to mentor-type links), plus Company sessions for companies the
caller is employed at. Only `status=SCHEDULED`, non-deleted sessions.

**Success response — `200`:** same paginated shape as §6.2.

---

### 6.7 Admin session list

`GET /mentor/session/admin/list/` — **paginated**

**Roles:** `Admin`

**Query params:** `status`, `ig_id` (filters to `entity_id=ig_id,
session_type=ig_session`), plus §1.3 params.

**Success response — `200`:** same paginated shape as §6.2 (across all
mentors' sessions, unrestricted).

---

### 6.8 Admin session verify

`PATCH /mentor/session/admin/verify/<session_id>/`

**Roles:** `Admin`

**Path params:** `session_id`

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `status` | enum | Yes | Target status: `SCHEDULED` / `REJECTED` / `CANCELLED` |
| `apply_to_series` | boolean | No, default `false` | Bulk-apply to sibling recurring sessions |

```json
{ "status": "SCHEDULED" }
```

**Allowed transitions only:**
- `PENDING_APPROVAL → SCHEDULED | REJECTED`
- `SCHEDULED → CANCELLED`

Any other transition → `400`.

**Business rule:** if `apply_to_series=true` and the session is part of a
recurring series, all sibling sessions still in the pre-update status are
bulk-updated to match.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session status updated to SCHEDULED."] },
  "response": {}
}
```

**Error — invalid transition (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid status transition for this session."] },
  "response": {}
}
```

---

## 7. Mentor Availability

### 7.1 List own availability slots

`GET /mentor/availability/` — **paginated**

**Roles:** `Mentor`

**Query params:** `ig_id`, `is_active` (string `'true'`), plus §1.3. Search:
`ig__name`. Sort: `weekday`, `start_time`, `created_at`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "slot-uuid-1",
        "mentor_user_id": "u-uuid-1234",
        "ig_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
        "ig_name": "Web Development",
        "weekday": 2,
        "start_time": "14:00:00",
        "end_time": "16:00:00",
        "timezone": "Asia/Kolkata",
        "is_active": true,
        "valid_from": "2026-08-01",
        "valid_to": "2026-12-31",
        "created_at": "2026-07-01T08:00:00Z",
        "updated_at": "2026-07-01T08:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

### 7.2 Get one availability slot

`GET /mentor/availability/<slot_id>/`

**Roles:** `Mentor`; own slots only.

**Success response — `200`:** single item shape from §7.1.

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Availability slot not found."] },
  "response": {}
}
```

---

### 7.3 Create an availability slot

`POST /mentor/availability/`

**Roles:** `Mentor`

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `ig` | uuid | No | If omitted, slot applies across **all** the mentor's IGs |
| `weekday` | integer | Yes | 1 (Mon) – 7 (Sun) |
| `start_time` | time (`HH:MM:SS`) | Yes | Must be before `end_time` |
| `end_time` | time | Yes | |
| `timezone` | string | No | Default `"Asia/Kolkata"` |
| `is_active` | boolean | No | |
| `valid_from` | date | No | Must be ≤ `valid_to` if both given |
| `valid_to` | date | No | |

```json
{
  "ig": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
  "weekday": 2,
  "start_time": "14:00:00",
  "end_time": "16:00:00",
  "timezone": "Asia/Kolkata",
  "valid_from": "2026-08-01",
  "valid_to": "2026-12-31"
}
```

**Constraint:** if `ig` is provided, caller must have an active
`UserIgLink(MENTOR)` for it (`403` otherwise). **If `ig` is omitted, this
check is skipped entirely** — by design, for a global slot.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Availability slot created successfully."] },
  "response": {
    "ig": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
    "weekday": 2,
    "start_time": "14:00:00",
    "end_time": "16:00:00",
    "timezone": "Asia/Kolkata",
    "is_active": true,
    "valid_from": "2026-08-01",
    "valid_to": "2026-12-31"
  }
}
```

**Error — not assigned mentor for IG (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You are not assigned as a mentor for this Interest Group."] },
  "response": {}
}
```

**Error — time range invalid (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["Start time must be before end time."] },
  "response": {}
}
```

---

### 7.4 Update an availability slot

`PATCH /mentor/availability/<slot_id>/`

**Roles:** `Mentor`; own slots only.

**Request body:** any subset of the §7.3 fields (partial).

> **⚠️ Known gap:** changing `ig` on an existing slot via this endpoint does
> **not** re-check IG mentor assignment — only the create path validates it.

**Success response — `200`:** same shape as §7.3's response, updated
values.

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Availability slot not found."] },
  "response": {}
}
```

---

### 7.5 Delete an availability slot

`DELETE /mentor/availability/<slot_id>/`

**Roles:** `Mentor`; own slots only. Hard delete.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Availability slot deleted successfully."] },
  "response": {}
}
```

---

### 7.6 Public availability lookup

`GET /mentor/public/availability/<mentor_id>/`

**Roles:** any authenticated user (**Public**)

**Path params:** `mentor_id` — `UserMentor.id`

**Business rule:** `mentor_id` must resolve to an `APPROVED` `UserMentor`;
returns their active slots, **unpaginated**.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    {
      "id": "slot-uuid-1",
      "mentor_user_id": "u-uuid-1234",
      "ig_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
      "ig_name": "Web Development",
      "weekday": 2,
      "start_time": "14:00:00",
      "end_time": "16:00:00",
      "timezone": "Asia/Kolkata",
      "is_active": true,
      "valid_from": "2026-08-01",
      "valid_to": "2026-12-31",
      "created_at": "2026-07-01T08:00:00Z",
      "updated_at": "2026-07-01T08:00:00Z"
    }
  ]
}
```

**Error — mentor not found/not approved (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Mentor not found or not approved."] },
  "response": {}
}
```

---

## 8. Session Participants

### 8.1 Join a session

`POST /mentor/session/participation/join/<session_id>/`

**Roles:** any authenticated user (**no membership check** — anyone can join
any joinable session)

**Path params:** `session_id`

**Request body:** none (empty body/`{}`)

**Constraints (`400` each):**
- Session must be `SCHEDULED`.
- Session must not be at `max_participants` capacity.
- Caller must not have already joined.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Successfully joined the session."] },
  "response": {
    "id": "link-uuid-1",
    "session_id": "session-uuid-1",
    "user_id": "u-uuid-5678",
    "user_full_name": "Alex Learner",
    "mu_id": "alex-learner@mulearn",
    "participant_role": "MENTEE",
    "attendance_status": "INVITED",
    "progress_note": null,
    "feedback": null,
    "contributed_minutes": null,
    "created_at": "2026-07-20T10:00:00Z",
    "session_title": "React Hooks Deep Dive",
    "session_starts_at": "2026-08-01T14:00:00Z",
    "session_ends_at": "2026-08-01T15:30:00Z",
    "session_mode": "HYBRID",
    "session_meeting_link": "https://meet.example.com/abc-defg-hij",
    "session_venue": "Auditorium 2",
    "session_status": "SCHEDULED",
    "session_entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
    "session_entity_name": "Web Development"
  }
}
```

**Error — full session (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["Session has reached its maximum participant limit."] },
  "response": {}
}
```

**Error — already joined (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["You have already joined this session."] },
  "response": {}
}
```

---

### 8.2 My participation history

`GET /mentor/session/participant/history/` — **paginated**

**Roles:** any authenticated user; self only.

**Query params:** §1.3 params. Search: `session__title`, `participant_role`.
Sort: `created_at`.

**Success response — `200`:** paginated array of items shaped like §8.1's
`response`.

---

### 8.3 Manually add a participant

`POST /mentor/session/participant/add/<session_id>/`

**Roles:** `Mentor`; caller must be the session's creator.

**Path params:** `session_id`

**Request body:**
```json
{ "muid": "alex-learner@mulearn" }
```

**Constraints (`400` unless noted):**
- `muid` must resolve to an existing, non-suspended user.
- Session must be `SCHEDULED`.
- Capacity check.
- Not already a participant.
- `403` if caller isn't the session's creator.

**Success response — `200`:** same shape as §8.1's `response`.

**Error — not session owner (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": [], "non_field_errors": ["You do not have permission to add participants to this session."] },
  "response": {}
}
```

---

### 8.4 List a session's participants

`GET /mentor/session/participant/list/<session_id>/` — **paginated**

**Roles:** `Mentor`; caller must be the session's creator.

**Path params:** `session_id`

**Query params:** §1.3. Search: `user__full_name`, `user__muid`. Sort:
`created_at`, `user_full_name`.

**Error — not authorized (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You don't have permission to view participants for this session."] },
  "response": {}
}
```

**Success response — `200`:** paginated array of §8.1's `response` shape.

---

### 8.5 Update a participant record

`PATCH /mentor/session/participant/update/<link_id>/`

**Roles:** `Mentor`; caller must be the parent session's creator.

**Path params:** `link_id` — `MentorshipSessionUserLink.id`

**Request body:** any subset of:

| Field | Type | Constraints |
|---|---|---|
| `attendance_status` | enum | `INVITED` / `ATTENDED` / `ABSENT` |
| `progress_note` | string | Max 500 chars |
| `contributed_minutes` | integer | Must be `> 0` if provided |

```json
{
  "attendance_status": "ATTENDED",
  "progress_note": "Actively participated, asked great questions.",
  "contributed_minutes": 90
}
```

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Participant record updated successfully."] },
  "response": {
    "attendance_status": "ATTENDED",
    "progress_note": "Actively participated, asked great questions.",
    "contributed_minutes": 90
  }
}
```

**Error — invalid minutes (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["Contributed minutes must be greater than zero."] },
  "response": {}
}
```

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Participant record not found."] },
  "response": {}
}
```

---

### 8.6 Leave session feedback

`PATCH /mentor/session/participant/feedback/<session_id>/`

**Roles:** any authenticated user who is a participant of the session.

**Path params:** `session_id`

**Request body:**
```json
{ "feedback": "Really helpful session, learned a lot about hooks!" }
```

**Constraints (`400`):**
- `feedback` non-empty.
- Caller's `attendance_status` must be `ATTENDED`.

**Success response — `200`:** same shape as §8.1's `response`, with
`feedback` populated.

**Error — not attended (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "non_field_errors": ["You can only leave feedback for sessions you have attended."] },
  "response": {}
}
```

**Error — not a participant (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["You are not a participant of this session."] },
  "response": {}
}
```

---

## 9. Mentor Tasks

Mentor-authored `TaskList` entries, always IG-scoped, always requiring
admin approval before going live.

### 9.1 IG dropdown

`GET /mentor/tasks/ig-dropdown/`

**Roles:** `Mentor`

**Success response — `200`** (unpaginated):
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    { "id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e", "name": "Web Development" }
  ]
}
```

---

### 9.2 List own tasks

`GET /mentor/tasks/` — **paginated**

**Roles:** `Mentor`; own submissions only.

**Query params:** `approval_status`, plus §1.3.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "task-uuid-1",
        "hashtag": "#react-hooks-101",
        "discord_link": null,
        "title": "Build a custom useDebounce hook",
        "description": "Implement and test a debounce hook in React.",
        "karma": 20,
        "channel": "Discord",
        "type": "Regular",
        "active": false,
        "variable_karma": false,
        "usage_count": 1,
        "level": "Level 1",
        "org": null,
        "ig": "Web Development",
        "event": null,
        "bonus_karma": null,
        "bonus_time": null,
        "approval_status": "pending",
        "rejection_reason": null,
        "reviewed_at": null,
        "requested_by_name": "Jane Doe",
        "requested_at": "2026-07-20T09:00:00Z",
        "skills": [
          { "id": "skill-uuid-1", "name": "React", "code": "REACT" }
        ],
        "created_at": "2026-07-20T09:00:00Z",
        "updated_at": "2026-07-20T09:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

### 9.3 Submit a task

`POST /mentor/tasks/`

**Roles:** `Mentor`

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `hashtag` | string | Yes | Must be globally unique |
| `title` | string | Yes | Max 75 chars |
| `karma` | integer | No | |
| `usage_count` | integer | No | |
| `description` | string | No | |
| `type` | uuid | Yes | `TaskType` id |
| `level` | uuid | No | `Level` id |
| `ig` | uuid | **Yes** | Caller must have an active `UserIgLink(MENTOR)` for it |
| `skill_ids` | array[uuid] | No | List of `Skill` ids; accepted as JSON string or array |

```json
{
  "hashtag": "#react-hooks-101",
  "title": "Build a custom useDebounce hook",
  "description": "Implement and test a debounce hook in React.",
  "karma": 20,
  "usage_count": 1,
  "type": "tasktype-uuid-regular",
  "level": "level-uuid-1",
  "ig": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
  "skill_ids": ["skill-uuid-1"]
}
```

**Business rule:** server forcibly sets `id`, `approval_status="pending"`,
`active=False`, `requested_by`, `requested_at` — client-submitted values for
these are ignored.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Task submitted for approval."] },
  "response": {}
}
```

**Error — duplicate hashtag (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "hashtag": ["A task with this hashtag already exists."] },
  "response": {}
}
```

**Error — not assigned mentor for IG (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "ig": ["You are not assigned as a mentor for this Interest Group."] },
  "response": {}
}
```

---

### 9.4 Get own task detail

`GET /mentor/tasks/<task_id>/`

**Roles:** `Mentor`; owned tasks only.

**Success response — `200`:** single item shape from §9.2.

**Error — not found (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Task not found."] },
  "response": {}
}
```

---

### 9.5 Update own task

`PUT /mentor/tasks/<task_id>/`

**Roles:** `Mentor`; owned tasks only.

**Request body:** same fields as §9.3 minus `created_by`, plus optional
`skill_ids` (replaces the full skill set if provided).

**Business rule:** unconditionally resets `approval_status="pending"`,
`active=False`, clears `rejection_reason`/`reviewed_by_admin`/`reviewed_at`
— **even if the task was already approved and live**, any edit pulls it
back into the review queue. Same `ig`-assignment and `hashtag`-uniqueness
checks as create.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Task updated and re-submitted for approval."] },
  "response": {}
}
```

---

### 9.6 Delete own task

`DELETE /mentor/tasks/<task_id>/`

**Roles:** `Mentor`; owned tasks only.

**Constraint:** only `approval_status="pending"` tasks can be deleted.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Task deleted successfully."] },
  "response": {}
}
```

**Error — non-pending task (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Cannot delete a task with status 'approved'. Only pending tasks can be deleted."] },
  "response": {}
}
```

---

## 10. Student Session Requests

Learners request a mentorship session; a mentor with authority over the
target IG approves or rejects it.

### 10.1 Submit a session request

`POST /mentor/session/student/request/`

**Roles:** any authenticated user

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `session_type` | enum | Yes | Must be `ig_session` — Company/Campus requests are rejected |
| `entity_id` | uuid | Yes | IG id; caller must have an active `UserIgLink` for it |
| `title` | string | Yes | |
| `description` | string | No | |
| `mode` | enum | Yes | `ONLINE` / `OFFLINE` / `HYBRID` |
| `starts_at` | datetime | Yes | Must be in the future, before `ends_at` |
| `ends_at` | datetime | Yes | |
| `meeting_link` | string | Conditional | Required for `HYBRID`, forbidden for `OFFLINE` |
| `venue` | string | Conditional | Required for `HYBRID`, forbidden for `ONLINE` |
| `max_participants` | integer | No | |

```json
{
  "session_type": "ig_session",
  "entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
  "title": "Help with async/await concepts",
  "description": "Struggling with promise chaining, would love a 1:1.",
  "mode": "ONLINE",
  "starts_at": "2026-08-05T16:00:00Z",
  "ends_at": "2026-08-05T16:30:00Z",
  "meeting_link": "https://meet.example.com/help-session"
}
```

**Business rule:** session created `status=REQUESTED`, `requested_by` =
caller; caller is auto-added as a `MENTEE`/`INVITED` participant.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session request submitted successfully. A mentor will review your request shortly."] },
  "response": {
    "id": "session-uuid-5",
    "session_type": "ig_session",
    "entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
    "title": "Help with async/await concepts",
    "description": "Struggling with promise chaining, would love a 1:1.",
    "mode": "ONLINE",
    "starts_at": "2026-08-05T16:00:00Z",
    "ends_at": "2026-08-05T16:30:00Z",
    "meeting_link": "https://meet.example.com/help-session",
    "venue": null,
    "max_participants": null
  }
}
```

**Error — not future (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "starts_at": ["Session start time must be in the future."] },
  "response": {}
}
```

**Error — not IG session type (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "session_type": ["Only Interest Group sessions can be requested. Company- and campus-scoped sessions are not supported."] },
  "response": {}
}
```

**Error — not a member (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "entity_id": ["You are not a member of this Interest Group."] },
  "response": {}
}
```

---

### 10.2 My submitted requests

`GET /mentor/session/student/my-requests/` — **paginated**

**Roles:** any authenticated user; self only.

**Query params:** `status`, plus §1.3.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "session-uuid-5",
        "session_type": "ig_session",
        "entity_id": "8f14e45f-ceea-4a4c-8535-7e5a6d3a1b1e",
        "entity_name": "Web Development",
        "title": "Help with async/await concepts",
        "description": "Struggling with promise chaining, would love a 1:1.",
        "mode": "ONLINE",
        "starts_at": "2026-08-05T16:00:00Z",
        "ends_at": "2026-08-05T16:30:00Z",
        "meeting_link": "https://meet.example.com/help-session",
        "venue": null,
        "max_participants": null,
        "status": "REQUESTED",
        "requested_by_id": "u-uuid-5678",
        "requested_by_name": "Alex Learner",
        "requested_by_muid": "alex-learner@mulearn",
        "created_at": "2026-07-22T08:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

### 10.3 List requests I can review

`GET /mentor/session/student-requests/` — **paginated**

**Roles:** `Mentor`

**Visibility rules:**
- No active mentor scope at all → sees nothing.
- Holds an active global `MENTOR`-tier scope → sees **all** `REQUESTED`
  sessions (admin-level visibility).
- Otherwise → sees only `ig_session` requests for IGs where they hold an
  active `UserIgLink(MENTOR)`.

**Query params:** §1.3. Search: `title`, `requested_by__full_name`. Sort:
`created_at`, `starts_at`.

**Success response — `200`:** paginated array of §10.2's item shape.

---

### 10.4 Approve or reject a request

`PATCH /mentor/session/student-requests/<session_id>/verify/`

**Roles:** `Mentor`, with the same scope rules as §10.3, evaluated per
request.

**Path params:** `session_id`

**Request body:**

| Field | Type | Required | Notes |
|---|---|---|---|
| `status` | enum | Yes | `APPROVED` / `REJECTED` |
| `starts_at` | datetime | No | Override, applied only if approving |
| `ends_at` | datetime | No | Override |
| `mode` | enum | No | Override |
| `meeting_link` | string | No | Override |
| `venue` | string | No | Override |

**Example — approve with time change:**
```json
{
  "status": "APPROVED",
  "starts_at": "2026-08-05T17:00:00Z",
  "ends_at": "2026-08-05T17:30:00Z"
}
```

**Example — reject:**
```json
{ "status": "REJECTED" }
```

**Validation on approve:** effective `starts_at` must be future, before
`ends_at`; mode/venue/meeting_link consistency.

**Business rule on approve:** session `status → SCHEDULED` directly (goes
live immediately, no separate admin step); `approved_by`/`approved_at` set;
**`created_by` is reassigned to the approving mentor** — the session then
appears in *their* `GET /mentor/session/list/`, not the original
requester's. `requested_by` stays unchanged (permanent audit trail).

**Business rule on reject:** `status → REJECTED`.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Session request approved. It is now pending admin scheduling."] },
  "response": {}
}
```
> Note: the success message text says "pending admin scheduling" but the
> session is actually set to `SCHEDULED` directly — this message is stale.

**Error — not authorized for this request (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You are not authorized to act on this session request."] },
  "response": {}
}
```

**Error — not found / not REQUESTED (`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Session request not found."] },
  "response": {}
}
```

---

## 11. Company Mentor Nomination

Distinct from §2–5 — this is how a Company's registrant nominates an
existing platform user as `COMPANY_MENTOR`. The nomination is created
`PENDING` and must still go through §3.1 to become `APPROVED` (the same
company owner, or an Admin, can do that).

### 11.1 Nominate a Company Mentor

`POST /company/mentor/nominate/`

**Roles:** `Company`, and caller must be the `company_user` of a
`status="verified"` `Company` (the company's own registrant — an already-
approved `COMPANY_MENTOR` cannot nominate another mentor through this
endpoint).

**Request body:**

| Field | Type | Required | Constraints |
|---|---|---|---|
| `muid` | string | Yes | Must resolve to an existing user who is already a member of the company's `Organization` (`UserOrganizationLink` exists, any state) |
| `reason` | string | No | |

```json
{
  "muid": "jane-doe@mulearn",
  "reason": "Senior engineer, great with junior mentees."
}
```

**Constraints (`400`):**
- `muid` doesn't resolve → error.
- Nominee not a member of the company's org → error.
- Nominee already has a non-`REJECTED` `COMPANY_MENTOR` nomination for this
  company → error, naming the existing status.

**Business rule:** creates `UserMentor(mentor_tier=COMPANY_MENTOR,
org=<resolved org>, status=PENDING)`. The nominated user is notified
(in-app notification) that their application is pending.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["User nominated as Company Mentor. Pending approval."] },
  "response": {
    "id": "mentor-uuid-9",
    "user_id": "u-uuid-1234",
    "user_name": "Jane Doe",
    "user_email": "jane.doe@example.com",
    "org_name": "Acme Corp",
    "mentor_tier": "COMPANY_MENTOR",
    "status": "PENDING"
  }
}
```

**Error — no verified company (`403`):**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": { "general": ["You must have a verified company profile to nominate mentors."] },
  "response": {}
}
```

**Error — duplicate nomination (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "muid": ["This user already has a PENDING Company Mentor nomination for your company."] },
  "response": {}
}
```

**Error — not a company member (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [], "muid": ["User 'jane-doe@mulearn' is not a member of this company's organisation."] },
  "response": {}
}
```

---

### 11.2 List company mentor nominations

`GET /company/mentor/list/`

**Roles:** `Company`; caller must be the verified company's registrant.

**Success response — `200`** (unpaginated array):
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    {
      "id": "mentor-uuid-9",
      "user_id": "u-uuid-1234",
      "user_name": "Jane Doe",
      "user_email": "jane.doe@example.com",
      "org_name": "Acme Corp",
      "mentor_tier": "COMPANY_MENTOR",
      "status": "PENDING"
    }
  ]
}
```

**Error — no verified company (`403`/`404`):**
```json
{
  "hasError": true,
  "statusCode": 404,
  "message": { "general": ["Verified company profile not found."] },
  "response": {}
}
```

---

## 12. Campus Mentor Nomination

### 12.1 Nominate a Campus Mentor

`POST /campus/assign-mentor/`

**Roles:** `CampusLead` or `LeadEnabler`

**Request body:**
```json
{ "muid": "student@mulearn" }
```

**Constraints (`400` each):**
- Caller must have a resolvable college org (own `UserOrganizationLink` to
  a College) — this becomes the nomination's scope.
- `muid` must resolve to an existing user.
- Nominee must be a **non-alumni** member of the same campus.
- Nominee must not already have a `CAMPUS_MENTOR` row for this org (any
  status).
- Nominee must not already hold **any** other `UserMentor` row of **any**
  tier — a user cannot be nominated for Campus Mentor while also holding
  another mentor tier (unlike Company/IG mentor stacking, which is additive
  elsewhere).

**Business rule:** creates `UserMentor(mentor_tier=CAMPUS_MENTOR,
org=<caller's college>, status=PENDING)`. Requires a subsequent
`PATCH /mentor/verify/<mentor_id>/` by an **Admin** — Campus Mentor
approvals are never delegated to a campus-side role, only §3.1's Company-
owner delegation exists.

**Success response — `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Student successfully nominated as a Campus Mentor"] },
  "response": {}
}
```

**Error — no organization (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["User have no organization"] },
  "response": {}
}
```

**Error — not a campus member (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Student is not a member of your campus"] },
  "response": {}
}
```

**Error — already a mentor of another tier (`400`):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Student is already a mentor with tier IG_MENTOR"] },
  "response": {}
}
```

---

## 13. Endpoint Summary Table

| # | Method | Path | Roles | Section |
|---|---|---|---|---|
| 1 | POST | `/mentor/register/` | Any user | 2.1 |
| 2 | PATCH | `/mentor/register/` | Owner | 2.2 |
| 3 | GET | `/mentor/status/` | Any user | 2.3 |
| 4 | GET | `/mentor/profile/` | Mentor | 2.4 |
| 5 | PATCH | `/mentor/profile/` | Mentor | 2.5 |
| 6 | GET | `/mentor/list/` | Admin | 2.6 |
| 7 | GET | `/mentor/detail/<id>/` | Admin | 2.7 |
| 8 | GET | `/mentor/public/profile/<id>/` | Any user | 2.8 |
| 9 | PATCH | `/mentor/verify/<id>/` | Admin / Company owner | 3.1 |
| 10 | GET | `/mentor/<id>/grants/` | Admin / Company owner | 4.1 |
| 11 | DELETE | `/mentor/<id>/grants/<grant_id>/` | Admin / Company owner | 4.2 |
| 12 | POST | `/mentor/admin/assign/` | Admin | 5.1 |
| 13 | DELETE | `/mentor/admin/assign/<muid>/` | Admin | 5.2 |
| 14 | POST | `/mentor/session/create/` | Mentor | 6.1 |
| 15 | GET | `/mentor/session/list/` | Mentor | 6.2 |
| 16 | GET | `/mentor/session/list/<id>/` | Mentor | 6.3 |
| 17 | PATCH | `/mentor/session/update/<id>/` | Mentor | 6.4 |
| 18 | DELETE | `/mentor/session/update/<id>/` | Mentor | 6.5 |
| 19 | GET | `/mentor/session/available/` | Any user | 6.6 |
| 20 | GET | `/mentor/session/admin/list/` | Admin | 6.7 |
| 21 | PATCH | `/mentor/session/admin/verify/<id>/` | Admin | 6.8 |
| 22 | GET, POST | `/mentor/availability/` | Mentor | 7.1, 7.3 |
| 23 | GET, PATCH, DELETE | `/mentor/availability/<id>/` | Mentor | 7.2, 7.4, 7.5 |
| 24 | GET | `/mentor/public/availability/<id>/` | Any user | 7.6 |
| 25 | POST | `/mentor/session/participation/join/<id>/` | Any user | 8.1 |
| 26 | GET | `/mentor/session/participant/history/` | Any user | 8.2 |
| 27 | POST | `/mentor/session/participant/add/<id>/` | Mentor | 8.3 |
| 28 | GET | `/mentor/session/participant/list/<id>/` | Mentor | 8.4 |
| 29 | PATCH | `/mentor/session/participant/update/<id>/` | Mentor | 8.5 |
| 30 | PATCH | `/mentor/session/participant/feedback/<id>/` | Participant | 8.6 |
| 31 | GET | `/mentor/tasks/ig-dropdown/` | Mentor | 9.1 |
| 32 | GET, POST | `/mentor/tasks/` | Mentor | 9.2, 9.3 |
| 33 | GET, PUT, DELETE | `/mentor/tasks/<id>/` | Mentor | 9.4, 9.5, 9.6 |
| 34 | POST | `/mentor/session/student/request/` | Any user | 10.1 |
| 35 | GET | `/mentor/session/student/my-requests/` | Any user | 10.2 |
| 36 | GET | `/mentor/session/student-requests/` | Mentor | 10.3 |
| 37 | PATCH | `/mentor/session/student-requests/<id>/verify/` | Mentor | 10.4 |
| 38 | POST | `/company/mentor/nominate/` | Company (owner) | 11.1 |
| 39 | GET | `/company/mentor/list/` | Company (owner) | 11.2 |
| 40 | POST | `/campus/assign-mentor/` | CampusLead / LeadEnabler | 12.1 |

---

## 14. Known Limitations

For frontend teams to design around — not contract violations, but current
behavior worth knowing:

1. **HTTP status vs body status mismatch** (see §1.2). Don't branch on the
   raw HTTP status code for 403/404 distinctions — read `response.statusCode`
   in the JSON body instead until this is fixed platform-wide.
2. `MentorStatusAPI` / `MentorProfileAPI` (§2.3, §2.4) return data for an
   arbitrary tier if the caller holds more than one `UserMentor` row.
3. `AvailableSessionListAPI` (§6.6) / `SessionJoinAPI` (§8.1) have no
   membership gating — any authenticated user can discover and join any
   scheduled session.
4. `PATCH /mentor/availability/<id>/` (§7.4) doesn't re-validate IG
   assignment if `ig` is changed.
5. No self-service "change employer" endpoint exists.
6. No unverified-organization → mentor-application linking exists yet — if
   an applicant's company isn't in the system, there's no path to apply and
   have it resolve once the org is verified.
7. No "Independent Mentor" / incomplete-profile flagging exists for
   IG-edit-assigned mentors with no employer on file.
8. `events/manage_views.py` `CAMPUS_IG` event creation/update checks only
   that the caller holds *a* `CAMPUS_MENTOR` grant somewhere, not that it's
   specifically for the campus the event targets.
9. Several success-message strings in §6.4, §6.8, §10.4 describe behavior
   (status resets, "pending admin scheduling") that doesn't match what the
   code actually does — the JSON *data* is correct; only the human-readable
   `message.general` text is stale. Don't parse message text for logic.
