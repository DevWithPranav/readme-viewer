# Company Dashboard — API Testing Guide

> **Base URL**: `http://localhost:8000/api/v1/dashboard/company/`  
> **Server**: `py manage.py runserver` must be running  
> **Auth**: All endpoints require `Authorization: Bearer <jwt_token>`

---

## 🔑 Getting a JWT Token

Before testing any endpoint, obtain a token from the auth flow and set it as a header:

```
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

You need **two different tokens** for full testing:
- **Company token** — user with `Company` role and an active company record
- **Learner token** — any regular user (Student / Mulearner)

---

## 📋 Prerequisites

| What | How to get it |
|---|---|
| `{JOB_ID}` | A valid `CompanyJob` ID (`status='Active'`, `is_deleted=False`) |
| `{APP_ID}` | Returned from the Apply endpoint |
| Active company | Company record with `status='active'` linked to the company user |
| DB table | Run `python alter-scripts/alter-1.60.py` to create `company_job_applications` |

---

---

## 1. Learner Discovery API

### `GET /api/v1/dashboard/company/learners/`

Lists learners with `is_public=True` profiles, filterable and paginated.

**Auth required**: Company JWT

---

#### Test 1.1 — Basic list (no filters)

```http
GET /api/v1/dashboard/company/learners/
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Learners fetched successfully."] },
  "response": {
    "learners": [
      {
        "id": "uuid",
        "muid": "john-doe@mulearn",
        "full_name": "John Doe",
        "gender": "Male",
        "district": "Ernakulam",
        "karma": 4200,
        "level": { "id": "uuid", "name": "Pathfinder", "level_order": 5 },
        "interest_groups": [{ "id": "uuid", "name": "AI/ML" }],
        "interested_in_work": true,
        "interested_in_gig_work": false
      }
    ],
    "pagination": {
      "count": 120,
      "totalPages": 12,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

#### Test 1.2 — Filter by karma range

```http
GET /api/v1/dashboard/company/learners/?karma_min=1000&karma_max=5000
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected**: Only learners with `wallet.karma` between 1000–5000.

---

#### Test 1.3 — Filter by Interest Group

```http
GET /api/v1/dashboard/company/learners/?ig_ids={IG_UUID_1},{IG_UUID_2}
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected**: Only learners who are members of either IG.

---

#### Test 1.4 — Filter by Achievement / Skill Badge

```http
GET /api/v1/dashboard/company/learners/?achievement_ids={ACHIEVEMENT_UUID}
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected**: Only learners who hold that badge.

---

#### Test 1.5 — Work-intent filter

```http
GET /api/v1/dashboard/company/learners/?interested_in_work=true
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 1.6 — Minimum level filter

```http
GET /api/v1/dashboard/company/learners/?level_order_min=3
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 1.7 — Search by name

```http
GET /api/v1/dashboard/company/learners/?search=john
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected**: Matches `full_name`, `muid`, `district__name`.

---

#### Test 1.8 — Sort by karma descending (default) / ascending

```http
GET /api/v1/dashboard/company/learners/?sortBy=-karma
GET /api/v1/dashboard/company/learners/?sortBy=karma
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 1.9 — Pagination

```http
GET /api/v1/dashboard/company/learners/?pageIndex=2&perPage=5
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 1.10 — Error: Non-company token

```http
GET /api/v1/dashboard/company/learners/
Authorization: Bearer {LEARNER_TOKEN}
```

**Expected `403`:**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": {
    "general": ["Company role required to access learner discovery."],
    "error_code": "COMPANY_ROLE_REQUIRED"
  }
}
```

---

#### Test 1.11 — Error: Invalid karma_min

```http
GET /api/v1/dashboard/company/learners/?karma_min=abc
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `400`** with `error_code: INVALID_FILTER_VALUE`.

---

#### Test 1.12 — Error: No token

```http
GET /api/v1/dashboard/company/learners/
```

**Expected `401`** from JWT middleware.

---
---

## 2. Job Application Tracking APIs

### Status Flow Reference

```
applied → shortlisted → accepted  ✓ (terminal)
        ↘            ↘ rejected   ✓ (terminal)
          → rejected
```

---

### 2.1 Learner — Apply to a Job

#### `POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/`

**Auth required**: Learner JWT (non-company)

---

#### Test 2.1.1 — Successful application (no cover note)

```http
POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/
Authorization: Bearer {LEARNER_TOKEN}
Content-Type: application/json

{}
```

**Expected `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Application submitted successfully."] },
  "response": {
    "application_id": "uuid",
    "job_id":         "{JOB_ID}",
    "job_title":      "Backend Developer",
    "status":         "applied",
    "applied_at":     "2026-05-07T17:00:00+00:00"
  }
}
```

> **Save the `application_id`** — you'll need it for status update tests.

---

#### Test 2.1.2 — Successful application (with cover note)

```http
POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/
Authorization: Bearer {LEARNER_TOKEN}
Content-Type: application/json

{
  "cover_note": "I am very interested in this role. I have 2 years of experience in Django."
}
```

**Expected `200`** (same shape, cover_note stored).

---

#### Test 2.1.3 — Error: Duplicate application

```http
POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/
Authorization: Bearer {LEARNER_TOKEN}
Content-Type: application/json

{}
```

*(Same learner, same job — second time)*

**Expected `400`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["You have already applied to this job."],
    "error_code": "DUPLICATE_APPLICATION"
  }
}
```

---

#### Test 2.1.4 — Error: Applying to a non-existent / deleted job

```http
POST /api/v1/dashboard/company/jobs/non-existent-id/apply/
Authorization: Bearer {LEARNER_TOKEN}
Content-Type: application/json

{}
```

**Expected `404`** with `error_code: JOB_NOT_FOUND`.

---

#### Test 2.1.5 — Error: Company user tries to apply

```http
POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{}
```

**Expected `403`:**
```json
{
  "hasError": true,
  "statusCode": 403,
  "message": {
    "general": ["Company users cannot apply to job postings."],
    "error_code": "COMPANY_CANNOT_APPLY"
  }
}
```

---

#### Test 2.1.6 — Error: cover_note too long (> 1000 chars)

```http
POST /api/v1/dashboard/company/jobs/{JOB_ID}/apply/
Authorization: Bearer {LEARNER_TOKEN}
Content-Type: application/json

{
  "cover_note": "A... (1001+ characters)"
}
```

**Expected `400`** with `error_code: VALIDATION_ERROR`.

---

### 2.2 Learner — View Own Applications

#### `GET /api/v1/dashboard/company/applications/`

**Auth required**: Learner JWT

---

#### Test 2.2.1 — List own applications

```http
GET /api/v1/dashboard/company/applications/
Authorization: Bearer {LEARNER_TOKEN}
```

**Expected `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Applications fetched successfully."] },
  "response": {
    "applications": [
      {
        "id":           "uuid",
        "status":       "applied",
        "cover_note":   "I am very interested...",
        "job_title":    "Backend Developer",
        "job_type":     "Full-Time",
        "company_name": "Acme Corp",
        "company_id":   "uuid",
        "created_at":   "2026-05-07T17:00:00Z",
        "updated_at":   "2026-05-07T17:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### Test 2.2.2 — Search by job title

```http
GET /api/v1/dashboard/company/applications/?search=Backend
Authorization: Bearer {LEARNER_TOKEN}
```

---

#### Test 2.2.3 — Sort by applied date

```http
GET /api/v1/dashboard/company/applications/?sortBy=-appliedAt
Authorization: Bearer {LEARNER_TOKEN}
```

---

#### Test 2.2.4 — Error: Company user accessing this endpoint

```http
GET /api/v1/dashboard/company/applications/
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `403`** with `error_code: COMPANY_ROLE_NOT_ALLOWED`.

---

### 2.3 Company — List Applicants for a Job

#### `GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/`

**Auth required**: Company JWT (must own the job)

---

#### Test 2.3.1 — List all applicants (no filter)

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Applicants fetched successfully."] },
  "response": {
    "job_id":    "{JOB_ID}",
    "job_title": "Backend Developer",
    "applicants": [
      {
        "id":            "application-uuid",
        "status":        "applied",
        "cover_note":    "I am very interested...",
        "applicant_id":  "user-uuid",
        "full_name":     "John Doe",
        "muid":          "john-doe@mulearn",
        "district":      "Ernakulam",
        "karma":         4200,
        "level":         { "id": "uuid", "name": "Pathfinder", "level_order": 5 },
        "reviewed_by_id": null,
        "reviewed_at":   null,
        "created_at":    "2026-05-07T17:00:00Z",
        "updated_at":    "2026-05-07T17:00:00Z"
      }
    ],
    "pagination": { "count": 1, "totalPages": 1, "isNext": false, "isPrev": false, "nextPage": null }
  }
}
```

---

#### Test 2.3.2 — Filter by status: `applied`

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?status=applied
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 2.3.3 — Filter by status: `shortlisted`

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?status=shortlisted
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 2.3.4 — Filter by status: `accepted` / `rejected`

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?status=accepted
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?status=rejected
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 2.3.5 — Sort by karma

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?sortBy=-karma
Authorization: Bearer {COMPANY_TOKEN}
```

---

#### Test 2.3.6 — Error: Invalid status filter

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/?status=pending
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `400`** with `error_code: INVALID_STATUS_FILTER`.

---

#### Test 2.3.7 — Error: Learner trying to access this endpoint

```http
GET /api/v1/dashboard/company/jobs/{JOB_ID}/applications/
Authorization: Bearer {LEARNER_TOKEN}
```

**Expected `403`** — learner is not a company user.

---

#### Test 2.3.8 — Error: Company accessing another company's job

```http
GET /api/v1/dashboard/company/jobs/{OTHER_COMPANY_JOB_ID}/applications/
Authorization: Bearer {COMPANY_TOKEN}
```

**Expected `403`** with `error_code: UNAUTHORIZED`.

---

### 2.4 Company — Update Application Status

#### `PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/`

**Auth required**: Company JWT (must own the job)

---

#### Test 2.4.1 — Shortlist an applicant (`applied → shortlisted`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "shortlisted"
}
```

**Expected `200`:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Application shortlisted successfully."] },
  "response": {
    "application_id": "{APP_ID}",
    "applicant_id":   "user-uuid",
    "new_status":     "shortlisted",
    "reviewed_by":    "company-user-uuid",
    "reviewed_at":    "2026-05-07T17:05:00+00:00"
  }
}
```

---

#### Test 2.4.2 — Accept a shortlisted applicant (`shortlisted → accepted`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "accepted"
}
```

**Expected `200`** with `new_status: accepted`.

---

#### Test 2.4.3 — Reject a shortlisted applicant (`shortlisted → rejected`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "rejected"
}
```

**Expected `200`** with `new_status: rejected`.

---

#### Test 2.4.4 — Reject directly (`applied → rejected`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "rejected"
}
```

*(On an application that is still in `applied` state)*

**Expected `200`** with `new_status: rejected`.

---

#### Test 2.4.5 — Error: Back-step transition (`shortlisted → applied`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "applied"
}
```

**Expected `400`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid status transition."],
    "error_code": "INVALID_TRANSITION",
    "errors": {
      "non_field_errors": [
        "Cannot transition from 'shortlisted' to 'applied'. Allowed: ['accepted', 'rejected']."
      ]
    }
  }
}
```

---

#### Test 2.4.6 — Error: Updating a terminal status (`accepted → rejected`)

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{
  "status": "rejected"
}
```

*(On an already-accepted application)*

**Expected `400`:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid status transition."],
    "error_code": "INVALID_TRANSITION",
    "errors": {
      "non_field_errors": ["Status 'accepted' is terminal — no further transitions allowed."]
    }
  }
}
```

---

#### Test 2.4.7 — Error: Application not found

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/non-existent-id/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{ "status": "shortlisted" }
```

**Expected `404`** with `error_code: APPLICATION_NOT_FOUND`.

---

#### Test 2.4.8 — Error: Missing `status` field

```http
PATCH /api/v1/dashboard/company/jobs/{JOB_ID}/applications/{APP_ID}/
Authorization: Bearer {COMPANY_TOKEN}
Content-Type: application/json

{}
```

**Expected `400`** with validation errors (status is required).

---

---

## 3. Complete Testing Sequence

Follow this order to test the full lifecycle end-to-end:

```
Step 1  → Learner Discovery
          GET company/learners/  (company token)

Step 2  → Learner applies
          POST company/jobs/{JOB_ID}/apply/  (learner token)
          → Save application_id

Step 3  → Learner views own applications
          GET company/applications/  (learner token)

Step 4  → Company views applicants
          GET company/jobs/{JOB_ID}/applications/  (company token)

Step 5  → Company shortlists
          PATCH company/jobs/{JOB_ID}/applications/{APP_ID}/
          { "status": "shortlisted" }

Step 6  → Company accepts
          PATCH company/jobs/{JOB_ID}/applications/{APP_ID}/
          { "status": "accepted" }

Step 7  → Try invalid transition (expect 400)
          PATCH company/jobs/{JOB_ID}/applications/{APP_ID}/
          { "status": "rejected" }   ← already accepted, terminal
```

---

## 4. Error Code Reference

| Error Code | HTTP | Trigger |
|---|---|---|
| `USER_NOT_FOUND` | 401 | JWT missing / invalid / expired |
| `COMPANY_ROLE_REQUIRED` | 403 | Non-company user hits company-only endpoint |
| `NO_ACTIVE_COMPANY` | 403 | Company user has no active company record |
| `COMPANY_CANNOT_APPLY` | 403 | Company user tries to apply to a job |
| `COMPANY_ROLE_NOT_ALLOWED` | 403 | Company user hits learner-only endpoint |
| `UNAUTHORIZED` | 403 | Company accessing another company's job/application |
| `JOB_NOT_FOUND` | 404 | Job does not exist, is deleted, or is not Active |
| `APPLICATION_NOT_FOUND` | 404 | Application ID not found for this job |
| `DUPLICATE_APPLICATION` | 400 | Learner already applied to this job |
| `INVALID_FILTER_VALUE` | 400 | Non-integer karma_min/karma_max |
| `INVALID_STATUS_FILTER` | 400 | ?status= value not in allowed choices |
| `INVALID_TRANSITION` | 400 | FSM violation in status update |
| `VALIDATION_ERROR` | 400 | cover_note too long or bad serializer input |
| `SERVER_ERROR` | 500 | Unexpected internal error |

---

## 5. Query Parameter Reference

### Learner Discovery (`company/learners/`)

| Param | Type | Example |
|---|---|---|
| `karma_min` | int | `?karma_min=500` |
| `karma_max` | int | `?karma_max=10000` |
| `ig_ids` | comma UUIDs | `?ig_ids=uuid1,uuid2` |
| `achievement_ids` | comma UUIDs | `?achievement_ids=uuid1` |
| `level_order_min` | int | `?level_order_min=3` |
| `interested_in_work` | bool | `?interested_in_work=true` |
| `interested_in_gig_work` | bool | `?interested_in_gig_work=true` |
| `search` | string | `?search=john` |
| `sortBy` | string | `?sortBy=-karma` or `?sortBy=name` |
| `pageIndex` | int | `?pageIndex=2` |
| `perPage` | int | `?perPage=5` |

### Applicant List (`company/jobs/{id}/applications/`)

| Param | Type | Example |
|---|---|---|
| `status` | string | `?status=applied` |
| `search` | string | `?search=john` |
| `sortBy` | string | `?sortBy=-karma` or `?sortBy=-appliedAt` |
| `pageIndex` | int | `?pageIndex=1` |
| `perPage` | int | `?perPage=10` |
