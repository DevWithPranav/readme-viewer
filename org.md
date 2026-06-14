# Management Module — API Reference

> **Base URL:** `https://<your-domain>/api/v1/`

This document is the complete API reference for the **Management** section of the µLearn dashboard. It covers every endpoint used by the following sub-modules:

| # | Section | URL Prefix |
|---|---------|------------|
| 1 | [Organization Affiliation](#1-organization-affiliation) | `dashboard/organisation/institutes/org/affiliation/` and `dashboard/affiliation/` |
| 2 | [Organization Transfer](#2-organization-transfer) | `dashboard/organisation/transfer/` |
| 3 | [Organization Department](#3-organization-department) | `dashboard/organisation/departments/` |
| 4 | [Organization Verification](#4-organization-verification) | `dashboard/organisation/verify/` |

---

## Authentication & Authorization

All endpoints in this module (unless explicitly marked **Public**) require:

| Header | Value | Description |
|--------|-------|-------------|
| `Authorization` | `Bearer <JWT_TOKEN>` | JSON Web Token obtained from the Auth module |

**Role-based access:** Most write operations require the **`Admins`** role. The specific role requirement is noted for each endpoint below.

### Standard Response Envelope

Every API response is wrapped in a consistent envelope:

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Success message"]
  },
  "response": { }
}
```

### Common Error Responses

| HTTP Status | `statusCode` | Scenario |
|-------------|-------------|----------|
| `400` | `400` | Validation error / bad request |
| `401` | `401` | Missing or invalid JWT token |
| `403` | `403` | Insufficient role permissions |

---

## 1. Organization Affiliation

The **Organization Affiliation** section manages university/organization affiliations (e.g., KTU, MG University, CUSAT). There are **two sets** of affiliation endpoints:

- **Set A** — Under `dashboard/organisation/` (inline with organization management)
- **Set B** — Under `dashboard/affiliation/` (dedicated affiliation CRUD module)

---

### Set A: Inline Affiliation APIs (under `dashboard/organisation/`)

---

#### 1.1 List Affiliations (Paginated — Dropdown)

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/organisation/institutes/org/affiliation/show/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Returns a paginated list of all organization affiliations with `value` (ID) and `label` (title) fields. Used primarily for populating dropdown selectors in organization forms. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number (default: 1) |
| `perPage` | integer | No | Items per page |
| `search` | string | No | Search by `id` or `title` |

**Sample Request:**

```http
GET /api/v1/dashboard/organisation/institutes/org/affiliation/show/?page=1&perPage=10&search=KTU
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "value": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "label": "KTU"
      },
      {
        "value": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "label": "MG University"
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

---

#### 1.2 List All Affiliations (Non-Paginated)

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/organisation/affiliation/list/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Returns a flat list of all affiliations with `id` and `title`. Lightweight endpoint for quick lookups. |

**Sample Request:**

```http
GET /api/v1/dashboard/organisation/affiliation/list/
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "KTU"
    },
    {
      "id": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
      "title": "MG University"
    }
  ]
}
```

---

#### 1.3 Create Affiliation (Inline)

| Property | Value |
|----------|-------|
| **Endpoint** | `POST /api/v1/dashboard/organisation/institutes/org/affiliation/create/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Creates a new organization affiliation record. The `title` must be unique across all affiliations. |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Name of the affiliation (must be unique) |

**Sample Request:**

```http
POST /api/v1/dashboard/organisation/institutes/org/affiliation/create/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Calicut University"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Calicut University added successfully"]
  },
  "response": {}
}
```

**Error Response — Duplicate Title (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": [
      { "title": ["Affiliation already exist"] }
    ]
  },
  "response": {}
}
```

---

#### 1.4 Update Affiliation (Inline)

| Property | Value |
|----------|-------|
| **Endpoint** | `PUT /api/v1/dashboard/organisation/institutes/org/affiliation/edit/{affiliation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Updates the title of an existing affiliation by its UUID. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `affiliation_id` | string (UUID) | Yes | The UUID of the affiliation to update |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | New title for the affiliation |

**Sample Request:**

```http
PUT /api/v1/dashboard/organisation/institutes/org/affiliation/edit/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Kerala Technological University"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["KTU Edited Successfully"]
  },
  "response": {}
}
```

**Error Response — Invalid Affiliation (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid affiliation"]
  },
  "response": {}
}
```

---

#### 1.5 Delete Affiliation (Inline)

| Property | Value |
|----------|-------|
| **Endpoint** | `DELETE /api/v1/dashboard/organisation/institutes/org/affiliation/delete/{affiliation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Permanently deletes an affiliation by its UUID. **Warning:** This may cascade-delete all organizations linked to this affiliation. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `affiliation_id` | string (UUID) | Yes | The UUID of the affiliation to delete |

**Sample Request:**

```http
DELETE /api/v1/dashboard/organisation/institutes/org/affiliation/delete/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["KTU Deleted Successfully"]
  },
  "response": {}
}
```

**Error Response — Invalid ID (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid affiliation id"]
  },
  "response": {}
}
```

---

### Set B: Dedicated Affiliation CRUD APIs (under `dashboard/affiliation/`)

These endpoints provide the same CRUD operations but with richer response payloads (including `created_by`, `updated_by`, `organization_count`).

---

#### 1.6 List Affiliations (with Metadata)

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/affiliation/` |
| **Authentication** | Required (JWT) |
| **Authorization** | Authenticated users |
| **Description** | Returns a paginated list of affiliations with full audit metadata, including the count of organizations linked to each affiliation. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number |
| `perPage` | integer | No | Items per page |
| `search` | string | No | Search by `title` |
| `sortBy` | string | No | Sort field: `title` |
| `orderBy` | string | No | `asc` or `desc` |

**Sample Request:**

```http
GET /api/v1/dashboard/affiliation/?page=1&perPage=10&sortBy=title&orderBy=asc
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "title": "KTU",
        "organization_count": 45,
        "created_by": "John Doe",
        "updated_by": "Jane Smith",
        "updated_at": "2025-06-10T12:30:00Z",
        "created_at": "2024-01-15T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

---

#### 1.7 Create Affiliation (Dedicated)

| Property | Value |
|----------|-------|
| **Endpoint** | `POST /api/v1/dashboard/affiliation/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Creates a new affiliation with auto-generated UUID. Returns the created affiliation data. |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Name of the affiliation (must be unique) |

**Sample Request:**

```http
POST /api/v1/dashboard/affiliation/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Anna University"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Anna University Affiliation created successfully"]
  },
  "response": {
    "title": "Anna University"
  }
}
```

---

#### 1.8 Update Affiliation (Dedicated)

| Property | Value |
|----------|-------|
| **Endpoint** | `PUT /api/v1/dashboard/affiliation/{affiliation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Updates an existing affiliation's title. Also updates the `updated_by` and `updated_at` fields. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `affiliation_id` | string (UUID) | Yes | The UUID of the affiliation |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | New title |

**Sample Request:**

```http
PUT /api/v1/dashboard/affiliation/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "APJ Abdul Kalam Technological University"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["KTU Edited Successfully"]
  },
  "response": {}
}
```

**Error Response — Invalid Affiliation (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Invalid affiliation id"]
  },
  "response": {}
}
```

---

#### 1.9 Delete Affiliation (Dedicated)

| Property | Value |
|----------|-------|
| **Endpoint** | `DELETE /api/v1/dashboard/affiliation/{affiliation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Permanently deletes an affiliation record. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `affiliation_id` | string (UUID) | Yes | The UUID of the affiliation to delete |

**Sample Request:**

```http
DELETE /api/v1/dashboard/affiliation/a1b2c3d4-e5f6-7890-abcd-ef1234567890/
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["KTU Deleted Successfully"]
  },
  "response": {}
}
```

---

### Affiliation Workflow

```
┌──────────────┐     ┌──────────────┐     ┌──────────────────────────┐
│  List / Show │────▶│  Create New  │────▶│  Assign to Organization  │
│  Affiliations│     │  Affiliation │     │  (via Org Create/Edit)   │
└──────────────┘     └──────────────┘     └──────────────────────────┘
                            │
                     ┌──────┴──────┐
                     ▼             ▼
              ┌──────────┐  ┌───────────┐
              │  Update  │  │  Delete   │
              │  Title   │  │  Record   │
              └──────────┘  └───────────┘
```

1. **List** existing affiliations to see what's available.
2. **Create** a new affiliation if the desired one doesn't exist.
3. **Assign** the affiliation to organizations when creating/editing colleges.
4. **Update** the title if the affiliation name changes.
5. **Delete** affiliations that are no longer needed.

---

## 2. Organization Transfer

The **Organization Transfer** section allows administrators to transfer all user links from one organization to another, effectively merging the source organization into the destination.

---

#### 2.1 Transfer Organization

| Property | Value |
|----------|-------|
| **Endpoint** | `POST /api/v1/dashboard/organisation/transfer/` |
| **Authentication** | Not explicitly required (no `authentication_classes` set on view) |
| **Authorization** | None (no `@role_required` decorator) |
| **Description** | Transfers all `UserOrganizationLink` records from the source organization to the destination organization, then **permanently deletes** the source organization. This is a destructive, irreversible operation. |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `from_id` | string | Yes | The `code` of the source organization (to be transferred **from** and deleted) |
| `to_id` | string | Yes | The `code` of the destination organization (to be transferred **to**) |

**Sample Request:**

```http
POST /api/v1/dashboard/organisation/transfer/
Content-Type: application/json

{
  "from_id": "MITS-KL",
  "to_id": "MITS-TVM"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": ["Organisations transferred successfully"]
}
```

**Error Response — Source Not Found (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [] },
  "response": ["From Organisations not present"]
}
```

**Error Response — Destination Not Found (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [] },
  "response": ["To Organisations not present"]
}
```

---

#### 2.2 Merge Organizations (Preview)

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/organisation/merge_organizations/{organisation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Previews the impact of merging a source organization into the destination (identified by `organisation_id`). Returns a summary of all related models and the number of records that would be affected. **Does not modify any data.** |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `organisation_id` | string (UUID) | Yes | The UUID of the **destination** organization |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source_org` | string | Yes | The `code` of the source organization to merge |

**Sample Request:**

```http
GET /api/v1/dashboard/organisation/merge_organizations/d4e5f6a7-b8c9-0123-def0-123456789abc/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "source_org": "MITS-KL"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "update_summary": [
      {
        "model": "userorganizationlink",
        "field_name": "org",
        "count": 150,
        "action": "update",
        "instance_ids": ["uuid-1", "uuid-2", "..."]
      },
      {
        "model": "college",
        "field_name": "org",
        "count": 1,
        "action": "delete",
        "instance_ids": ["uuid-3"]
      },
      {
        "model": "orgdiscordlink",
        "field_name": "org",
        "count": 1,
        "action": "delete",
        "instance_ids": ["uuid-4"]
      }
    ],
    "source_org": "MITS-KL"
  }
}
```

**Error Response — Organization Not Found (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["An organization with the given id doesn't exist"]
  },
  "response": {}
}
```

---

#### 2.3 Merge Organizations (Execute)

| Property | Value |
|----------|-------|
| **Endpoint** | `PATCH /api/v1/dashboard/organisation/merge_organizations/{organisation_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Executes the organization merger. All related records from the source organization are re-pointed to the destination organization. OneToOne conflicts are resolved by deleting the existing destination record. The source organization is then **permanently deleted**. This operation runs inside a database transaction. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `organisation_id` | string (UUID) | Yes | The UUID of the **destination** organization |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source_org` | string | Yes | The `code` of the source organization to merge (will be deleted) |

**Sample Request:**

```http
PATCH /api/v1/dashboard/organisation/merge_organizations/d4e5f6a7-b8c9-0123-def0-123456789abc/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "source_org": "MITS-KL"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Organizations merged successfully into Muthoot Institute of Technology and Science."]
  },
  "response": {}
}
```

**Error Response — Self-Merge Attempt (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": {
      "source_org": ["You can't merge an organization into itself."]
    }
  },
  "response": {}
}
```

---

### Transfer & Merge Workflow

```
                    ┌─────────────────────────────┐
                    │    Select Source & Target     │
                    │      Organizations           │
                    └──────────────┬───────────────┘
                                   │
                    ┌──────────────┴───────────────┐
                    │                              │
             ┌──────▼───────┐            ┌─────────▼────────┐
             │   Simple     │            │   Full Merger     │
             │  Transfer    │            │  (Preview + Exec) │
             └──────┬───────┘            └─────────┬────────┘
                    │                              │
                    │                    ┌─────────▼────────┐
                    │                    │  GET Preview      │
                    │                    │  (inspect impact) │
                    │                    └─────────┬────────┘
                    │                              │
                    │                    ┌─────────▼────────┐
                    │                    │  PATCH Execute    │
                    │                    │  (merge + delete) │
                    │                    └──────────────────┘
                    │
             ┌──────▼──────────────────────────────┐
             │  POST /transfer/                     │
             │  Moves all user links; deletes source│
             └─────────────────────────────────────┘
```

> **⚠️ Important:** Both the Transfer and Merge operations are **destructive** — the source organization is permanently deleted after the operation completes.

---

## 3. Organization Department

The **Organization Department** section manages academic departments (e.g., Computer Science, Mechanical Engineering) that users can be associated with through their organization link.

---

#### 3.1 List Departments

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/organisation/departments/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Returns a paginated list of all departments. Supports search and sorting by title. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | integer | No | Page number |
| `perPage` | integer | No | Items per page |
| `search` | string | No | Search by `title` |
| `sortBy` | string | No | Sort field: `title` |
| `orderBy` | string | No | `asc` or `desc` |

**Sample Request:**

```http
GET /api/v1/dashboard/organisation/departments/?page=1&perPage=10&search=Computer
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "f1e2d3c4-b5a6-7890-fedc-ba9876543210",
        "title": "Computer Science and Engineering"
      },
      {
        "id": "e2d3c4b5-a6f7-8901-edcb-a98765432101",
        "title": "Computer Applications"
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

---

#### 3.2 Create Department

| Property | Value |
|----------|-------|
| **Endpoint** | `POST /api/v1/dashboard/organisation/departments/create/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Creates a new department record. The `created_by` and `updated_by` are set from the JWT token. |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | Name of the department |

**Sample Request:**

```http
POST /api/v1/dashboard/organisation/departments/create/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Artificial Intelligence and Data Science"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Artificial Intelligence and Data Science created successfully"]
  },
  "response": {}
}
```

**Error Response — Validation Error (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": [] },
  "response": {
    "title": ["This field is required."]
  }
}
```

---

#### 3.3 Update Department

| Property | Value |
|----------|-------|
| **Endpoint** | `PUT /api/v1/dashboard/organisation/departments/edit/{department_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Updates the title of an existing department. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `department_id` | string (UUID) | Yes | The UUID of the department to update |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `title` | string | Yes | New title for the department |

**Sample Request:**

```http
PUT /api/v1/dashboard/organisation/departments/edit/f1e2d3c4-b5a6-7890-fedc-ba9876543210/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "title": "Computer Science & Engineering"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Computer Science and Engineering updated successfully"]
  },
  "response": {}
}
```

---

#### 3.4 Delete Department

| Property | Value |
|----------|-------|
| **Endpoint** | `DELETE /api/v1/dashboard/organisation/departments/delete/{department_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | `Admins` role |
| **Description** | Permanently deletes a department. **Warning:** This may affect `UserOrganizationLink` records that reference this department. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `department_id` | string (UUID) | Yes | The UUID of the department to delete |

**Sample Request:**

```http
DELETE /api/v1/dashboard/organisation/departments/delete/f1e2d3c4-b5a6-7890-fedc-ba9876543210/
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Computer Science and Engineering deleted successfully"]
  },
  "response": {}
}
```

---

### Department Workflow

```
┌────────────────────┐     ┌──────────────────┐
│   List Departments │────▶│ Create Department│
│   GET /departments/│     │ POST /create/    │
└────────────────────┘     └──────────────────┘
         │                         │
         │                  ┌──────┴──────────────────────────────┐
         │                  ▼                                     ▼
         │          ┌──────────────────┐             ┌────────────────────┐
         │          │ Update Department│             │  Delete Department │
         │          │ PUT /edit/{id}/  │             │  DELETE /delete/{id}│
         │          └──────────────────┘             └────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│   Used in UserOrganizationLink for user-org binding │
│   (Student selects department during registration)  │
└─────────────────────────────────────────────────────┘
```

---

## 4. Organization Verification

The **Organization Verification** section manages the verification workflow for organizations submitted by users that are not yet in the system. When a user registers with an organization that doesn't exist, an `UnverifiedOrganization` record is created and queued for admin review.

---

#### 4.1 List Unverified Organizations

| Property | Value |
|----------|-------|
| **Endpoint** | `GET /api/v1/dashboard/organisation/verify/list/` |
| **Authentication** | Required (JWT) |
| **Authorization** | Authenticated users (permission class on view) |
| **Description** | Returns all unverified organizations that are pending review (where `verified` is `null`). Results are ordered by most recently created first. |

**Sample Request:**

```http
GET /api/v1/dashboard/organisation/verify/list/
Authorization: Bearer <JWT_TOKEN>
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": [
    {
      "id": "c3d4e5f6-a7b8-9012-cdef-012345678901",
      "title": "Government Engineering College Thrissur",
      "org_type": "College",
      "graduation_year": 2026,
      "department": "Computer Science and Engineering",
      "created_by": "Arun Kumar",
      "created_at": "2025-06-10T14:22:00Z"
    },
    {
      "id": "d4e5f6a7-b8c9-0123-def0-123456789abc",
      "title": "TechCorp Solutions",
      "org_type": "Company",
      "graduation_year": null,
      "department": null,
      "created_by": "Priya Nair",
      "created_at": "2025-06-09T10:15:00Z"
    }
  ]
}
```

**Response Field Descriptions:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | string (UUID) | Unique ID of the unverified organization record |
| `title` | string | Name submitted by the user |
| `org_type` | string | One of: `College`, `Company`, `Community`, `School` |
| `graduation_year` | integer/null | Graduation year (only for colleges) |
| `department` | string/null | Department title (only for colleges) |
| `created_by` | string | Full name of the user who submitted the request |
| `created_at` | string (datetime) | Timestamp when the request was created |

---

#### 4.2 Verify Organization

| Property | Value |
|----------|-------|
| **Endpoint** | `POST /api/v1/dashboard/organisation/verify/{uorg_id}/` |
| **Authentication** | Required (JWT) |
| **Authorization** | Authenticated users (permission class on view) |
| **Description** | Verifies (approves or rejects) an unverified organization. When approved (`verified: true`), the system maps the unverified organization to an existing organization and automatically creates a `UserOrganizationLink` for the requesting user with their specified department and graduation year. |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uorg_id` | string (UUID) | Yes | The UUID of the unverified organization record |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `verified` | boolean | Yes | `true` to approve, `false` to reject |
| `org_id` | string (UUID) | Yes | The UUID of the actual `Organization` record to map to |

**Sample Request — Approve:**

```http
POST /api/v1/dashboard/organisation/verify/c3d4e5f6-a7b8-9012-cdef-012345678901/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "verified": true,
  "org_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Sample Response (200 OK):**

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Organization verified successfully"]
  },
  "response": {}
}
```

**Sample Request — Reject:**

```http
POST /api/v1/dashboard/organisation/verify/c3d4e5f6-a7b8-9012-cdef-012345678901/
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "verified": false,
  "org_id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890"
}
```

**Error Response — Organization Not Found (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Organization does not exist"]
  },
  "response": {}
}
```

**Error Response — Already Verified (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Organization already verified"]
  },
  "response": {}
}
```

**Error Response — User Already Linked (400):**

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Unable to assign organization to user"]
  },
  "response": {}
}
```

---

### Verification Workflow

```
┌──────────────────────────────────┐
│   User Registration / Profile    │
│   User submits an org that       │
│   doesn't exist in the system    │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│   UnverifiedOrganization record  │
│   created (verified = NULL)      │
└──────────────┬───────────────────┘
               │
               ▼
┌──────────────────────────────────┐
│   Admin views pending list       │
│   GET /verify/list/              │
└──────────────┬───────────────────┘
               │
        ┌──────┴──────┐
        ▼             ▼
┌──────────────┐ ┌──────────────┐
│   Approve    │ │   Reject     │
│ verified=true│ │ verified=false│
└──────┬───────┘ └──────────────┘
       │
       ▼
┌──────────────────────────────────────────┐
│   System auto-creates:                    │
│   1. Maps to actual Organization (org_id) │
│   2. Creates UserOrganizationLink         │
│      - user = requesting user             │
│      - org  = mapped organization         │
│      - department = from unverified record│
│      - graduation_year = from unverified  │
│      - verified = true                    │
└──────────────────────────────────────────┘
```

---

## Complete API Summary Table

| # | Method | Endpoint | Auth | Role | Section |
|---|--------|----------|------|------|---------|
| 1 | `GET` | `/dashboard/organisation/institutes/org/affiliation/show/` | JWT | Admin | Affiliation |
| 2 | `GET` | `/dashboard/organisation/affiliation/list/` | JWT | Admin | Affiliation |
| 3 | `POST` | `/dashboard/organisation/institutes/org/affiliation/create/` | JWT | Admin | Affiliation |
| 4 | `PUT` | `/dashboard/organisation/institutes/org/affiliation/edit/{id}/` | JWT | Admin | Affiliation |
| 5 | `DELETE` | `/dashboard/organisation/institutes/org/affiliation/delete/{id}/` | JWT | Admin | Affiliation |
| 6 | `GET` | `/dashboard/affiliation/` | JWT | Any | Affiliation |
| 7 | `POST` | `/dashboard/affiliation/` | JWT | Admin | Affiliation |
| 8 | `PUT` | `/dashboard/affiliation/{id}/` | JWT | Admin | Affiliation |
| 9 | `DELETE` | `/dashboard/affiliation/{id}/` | JWT | Admin | Affiliation |
| 10 | `POST` | `/dashboard/organisation/transfer/` | — | — | Transfer |
| 11 | `GET` | `/dashboard/organisation/merge_organizations/{id}/` | JWT | Admin | Transfer |
| 12 | `PATCH` | `/dashboard/organisation/merge_organizations/{id}/` | JWT | Admin | Transfer |
| 13 | `GET` | `/dashboard/organisation/departments/` | JWT | Admin | Department |
| 14 | `POST` | `/dashboard/organisation/departments/create/` | JWT | Admin | Department |
| 15 | `PUT` | `/dashboard/organisation/departments/edit/{id}/` | JWT | Admin | Department |
| 16 | `DELETE` | `/dashboard/organisation/departments/delete/{id}/` | JWT | Admin | Department |
| 17 | `GET` | `/dashboard/organisation/verify/list/` | JWT | Any | Verification |
| 18 | `POST` | `/dashboard/organisation/verify/{id}/` | JWT | Any | Verification |

> All endpoints are prefixed with `/api/v1/`

---

## Data Models Reference

### OrgAffiliation

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID (PK) | Auto-generated primary key |
| `title` | string(75) | Affiliation name |
| `created_by` | FK → User | User who created the record |
| `updated_by` | FK → User | User who last updated |
| `created_at` | datetime | Auto-set on creation |
| `updated_at` | datetime | Auto-set on update |

### Department

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID (PK) | Auto-generated primary key |
| `title` | string(100) | Department name |
| `created_by` | FK → User | User who created the record |
| `updated_by` | FK → User | User who last updated |
| `created_at` | datetime | Auto-set on creation |
| `updated_at` | datetime | Auto-set on update |

### UnverifiedOrganization

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID (PK) | Auto-generated primary key |
| `title` | string(100) | Submitted organization name |
| `org_type` | string(25) | `College`, `Company`, `Community`, `School` |
| `graduation_year` | integer (nullable) | Graduation year (colleges only) |
| `department` | FK → Department | Academic department (nullable) |
| `verified` | boolean (nullable) | `null` = pending, `true` = approved, `false` = rejected |
| `verified_by` | FK → User | Admin who performed verification |
| `verified_at` | datetime (nullable) | When verification occurred |
| `org` | FK → Organization | Mapped actual organization |
| `created_by` | FK → User | User who submitted the request |
| `created_at` | datetime | Submission timestamp |

### UserOrganizationLink

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID (PK) | Auto-generated primary key |
| `user` | FK → User | The linked user |
| `org` | FK → Organization | The linked organization |
| `department` | FK → Department (nullable) | Academic department |
| `graduation_year` | string(10) (nullable) | Year of graduation |
| `verified` | boolean | Whether the link is verified |
| `is_alumni` | boolean (nullable) | Alumni status flag |
| `created_by` | FK → User | Who created the link |
| `created_at` | datetime | Auto-set on creation |
