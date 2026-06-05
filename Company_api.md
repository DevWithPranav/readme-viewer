# Company Dashboard — API Reference

> **Base URL prefix:** `/api/v1/dashboard/company/`  
> **Authentication:** All private endpoints require a JWT Bearer token in the `Authorization` header.  
> **Format:** All request and response bodies are `application/json`.

---

## Legend

| Symbol | Meaning |
|--------|---------|
|  **Private** | Requires `Authorization: Bearer <JWT>` header |
|  **Public** | No authentication required |
|  **Admin** | Requires `ADMIN` role |
|  **Company** | Requires verified `COMPANY` role (creator or approved Company Mentor) |
|  **Any Auth** | Any authenticated user |

---

## Table of Contents

1. [Company Registration & Profile](#1-company-registration--profile)
2. [Company Admin Endpoints](#2-company-admin-endpoints)
3. [Public Company Endpoints](#3-public-company-endpoints)
4. [Company Jobs & Gigs](#4-company-jobs--gigs)
5. [Job Applications](#5-job-applications)
6. [Company Tasks](#6-company-tasks)
7. [Company Mentor Nomination](#7-company-mentor-nomination)
8. [MuLearner Directory](#8-mulearner-directory)
9. [Analytics](#9-analytics)

---

## 1. Company Registration & Profile

### 1.1 Register a New Company

| Field | Value |
|-------|-------|
| **Method** | `POST` |
| **Endpoint** | `/api/v1/dashboard/company/register/` |
| **Auth** |  Private |
| **Role** |  Any authenticated user |
| **Usage** | Submit a new company registration request. Status starts as `pending` until admin verifies. Only one registration per user. |

**Request Body:**

```json
{
  "name": "TechCorp India",
  "logo": "https://cdn.example.com/logos/techcorp.png",
  "description": "A leading fintech startup building innovative payment solutions.",
  "short_pitch": "We simplify payments for Bharat.",
  "industry_sector": "Fintech",
  "website_link": "https://techcorp.in",
  "email": "hello@techcorp.in",
  "location": "Kochi, Kerala",
  "district_id": "uuid-of-district",
  "state_id": "uuid-of-state",
  "country_id": "uuid-of-country",
  "legal_name": "TechCorp India Pvt Ltd",
  "registration_number": "CIN123456",
  "tax_id": "GSTIN98765",
  "company_size": "11-50",
  "linkedin_url": "https://linkedin.com/company/techcorp-india",
  "founded_year": 2021,
  "remote_policy": "Hybrid",
  "culture_text": "We value curiosity and ownership.",
  "tech_stack": ["Django", "React", "PostgreSQL"],
  "perks": ["Health Insurance", "Remote Fridays", "Learning Budget"],
  "testimonials": [],
  "gallery": []
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Company registration submitted successfully.",
  "response": {
    "name": "TechCorp India",
    "logo": "https://cdn.example.com/logos/techcorp.png",
    "description": "A leading fintech startup building innovative payment solutions.",
    "short_pitch": "We simplify payments for Bharat.",
    "industry_sector": "Fintech",
    "website_link": "https://techcorp.in",
    "email": "hello@techcorp.in",
    "location": "Kochi, Kerala",
    "district_id": "uuid-of-district",
    "state_id": "uuid-of-state",
    "country_id": "uuid-of-country",
    "legal_name": "TechCorp India Pvt Ltd",
    "registration_number": "CIN123456",
    "tax_id": "GSTIN98765",
    "company_size": "11-50",
    "linkedin_url": "https://linkedin.com/company/techcorp-india",
    "founded_year": 2021,
    "remote_policy": "Hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Django", "React", "PostgreSQL"],
    "perks": ["Health Insurance", "Remote Fridays", "Learning Budget"],
    "testimonials": [],
    "gallery": []
  }
}
```

**Error Response `400 Bad Request` (duplicate registration):**

```json
{
  "statusCode": 400,
  "message": "A company request already exists for your account."
}
```

---

### 1.2 Update / Resubmit Company Registration

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/register/` |
| **Auth** |  Private |
| **Role** |  Any authenticated user (must have a pending/rejected registration) |
| **Usage** | Update a pending or rejected company registration. If previously rejected, this automatically resubmits it (status resets to `pending`, rejection reason cleared). |

> [!IMPORTANT]
> Cannot be used if company is already `verified`. Use the `/profile/` endpoint instead.

**Request Body (all fields optional):**

```json
{
  "name": "TechCorp India Updated",
  "email": "contact@techcorp.in",
  "company_size": "51-200",
  "description": "Updated description for TechCorp."
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Company registration updated and resubmitted successfully.",
  "response": {
    "name": "TechCorp India Updated",
    "email": "contact@techcorp.in",
    "company_size": "51-200",
    "description": "Updated description for TechCorp."
  }
}
```

**Error Response `404`:**

```json
{
  "statusCode": 404,
  "message": "No company registration request found for your account."
}
```

---

### 1.3 Check Company Registration Status

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/status/` |
| **Auth** |  Private |
| **Role** |  Any authenticated user |
| **Usage** | Poll this endpoint to know if your company registration is `pending`, `verified`, or `rejected`. Includes rejection reason if applicable. |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "status": "pending",
    "rejection_reason": null,
    "company_id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
    "name": "TechCorp India",
    "slug": "techcorp-india"
  }
}
```

**Rejected example:**

```json
{
  "statusCode": 200,
  "response": {
    "status": "rejected",
    "rejection_reason": "Incomplete documentation. Please provide a valid registration number.",
    "company_id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
    "name": "TechCorp India",
    "slug": "techcorp-india"
  }
}
```

---

### 1.4 Get Company Profile (Authenticated)

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/profile/` |
| **Auth** |  Private |
| **Role** |  Verified Company creator **or** approved Company Mentor |
| **Usage** | Retrieve the full company profile for the currently authenticated user's company. |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
    "name": "TechCorp India",
    "slug": "techcorp-india",
    "logo": "https://cdn.example.com/logos/techcorp.png",
    "description": "A leading fintech startup.",
    "short_pitch": "We simplify payments for Bharat.",
    "industry_sector": "Fintech",
    "website_link": "https://techcorp.in",
    "email": "hello@techcorp.in",
    "location": "Kochi, Kerala",
    "district_name": "Ernakulam",
    "legal_name": "TechCorp India Pvt Ltd",
    "registration_number": "CIN123456",
    "tax_id": "GSTIN98765",
    "company_size": "11-50",
    "linkedin_url": "https://linkedin.com/company/techcorp-india",
    "founded_year": 2021,
    "remote_policy": "Hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Django", "React", "PostgreSQL"],
    "perks": ["Health Insurance", "Remote Fridays"],
    "testimonials": [],
    "gallery": [],
    "status": "verified",
    "rejection_reason": null,
    "company_user_id": "user-uuid",
    "company_user_name": "Jane Doe",
    "company_user_email": "jane@techcorp.in",
    "created_at": "2025-01-10T09:30:00Z",
    "updated_at": "2025-01-15T12:00:00Z"
  }
}
```

---

### 1.5 Update Company Profile (Authenticated)

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/profile/` |
| **Auth** |  Private |
| **Role** |  Verified Company creator **or** approved Company Mentor |
| **Usage** | Update any fields of a verified company's profile. All fields are optional (partial update). |

**Request Body (any subset of fields):**

```json
{
  "short_pitch": "Empowering India's next billion with seamless payments.",
  "culture_text": "Innovation, Ownership, and Impact are our core values.",
  "tech_stack": ["Django", "React", "PostgreSQL", "Redis"],
  "perks": ["Health Insurance", "Remote Fridays", "ESOPs"]
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Company profile updated successfully.",
  "response": {
    "short_pitch": "Empowering India's next billion with seamless payments.",
    "culture_text": "Innovation, Ownership, and Impact are our core values.",
    "tech_stack": ["Django", "React", "PostgreSQL", "Redis"],
    "perks": ["Health Insurance", "Remote Fridays", "ESOPs"]
  }
}
```

---

## 2. Company Admin Endpoints

> All endpoints in this section require `ADMIN` role.

### 2.1 Admin Summary Dashboard

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/summary/` |
| **Auth** |  Private |
| **Role** |  Admin only |
| **Usage** | High-level statistics for the admin dashboard — total companies, verification breakdown, job counts, and company task counts. |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "total_companies": 42,
    "verified_companies": 30,
    "pending_companies": 8,
    "rejected_companies": 4,
    "total_jobs": 175,
    "total_company_tasks": 63
  }
}
```

---

### 2.2 List All Companies

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/list/` |
| **Auth** |  Private |
| **Role** |  Admin only |
| **Usage** | Paginated list of all companies with optional filters. Supports search and sort. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter by status: `pending`, `verified`, `rejected` |
| `industry_sector` | string | No | Filter by industry sector |
| `company_size` | string | No | Filter by company size (e.g. `11-50`) |
| `district` | string | No | Filter by district name |
| `state` | string | No | Filter by state name |
| `country` | string | No | Filter by country name |
| `search` | string | No | Search across name, slug, email, industry_sector |
| `sort_by` | string | No | Sort field: `name`, `status`, `created_at` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number (default: 1) |
| `per_page` | int | No | Items per page (default: 10) |

**Example Request:** `GET /api/v1/dashboard/company/list/?status=pending&page=1&per_page=5`

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
        "name": "TechCorp India",
        "slug": "techcorp-india",
        "status": "pending",
        "email": "hello@techcorp.in",
        "company_user_id": "user-uuid-1",
        "company_user_name": "Jane Doe",
        "industry_sector": "Fintech",
        "company_size": "11-50",
        "location": "Kochi, Kerala",
        "district_name": "Ernakulam",
        "state_name": "Kerala",
        "country_name": "India",
        "verification_requested_at": "2025-01-10T09:30:00Z",
        "verified_at": null
      }
    ],
    "pagination": {
      "count": 8,
      "total_pages": 2,
      "current_page": 1,
      "per_page": 5,
      "next": "/api/v1/dashboard/company/list/?status=pending&page=2",
      "previous": null
    }
  }
}
```

---

### 2.3 Get Company Details by ID (Admin)

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/{company_id}/` |
| **Auth** |  Private |
| **Role** |  Admin only |
| **Usage** | Get full details of any company by its UUID. Used by admin to review registration details before verifying. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `company_id` | UUID string | Company's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
    "name": "TechCorp India",
    "slug": "techcorp-india",
    "logo": "https://cdn.example.com/logos/techcorp.png",
    "description": "A leading fintech startup.",
    "short_pitch": "We simplify payments for Bharat.",
    "industry_sector": "Fintech",
    "website_link": "https://techcorp.in",
    "email": "hello@techcorp.in",
    "location": "Kochi, Kerala",
    "legal_name": "TechCorp India Pvt Ltd",
    "registration_number": "CIN123456",
    "tax_id": "GSTIN98765",
    "company_size": "11-50",
    "linkedin_url": "https://linkedin.com/company/techcorp-india",
    "founded_year": 2021,
    "remote_policy": "Hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Django", "React", "PostgreSQL"],
    "perks": ["Health Insurance", "Remote Fridays"],
    "testimonials": [],
    "gallery": [],
    "status": "pending",
    "rejection_reason": null,
    "company_user_id": "user-uuid-1",
    "company_user_name": "Jane Doe",
    "company_user_email": "jane@techcorp.in",
    "district_name": "Ernakulam",
    "verification_requested_at": "2025-01-10T09:30:00Z",
    "verified_at": null,
    "verified_by": null,
    "created_at": "2025-01-10T09:30:00Z",
    "updated_at": "2025-01-15T12:00:00Z"
  }
}
```

**Error Response `404`:**

```json
{
  "statusCode": 404,
  "message": "Company not found."
}
```

---

### 2.4 Verify or Reject a Company

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/verify/{company_id}/` |
| **Auth** |  Private |
| **Role** |  Admin only |
| **Usage** | Approve or reject a pending company registration. On approval, an `Organization` record is created and the `COMPANY` role is granted to the creator. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `company_id` | UUID string | Company's unique ID |

**Request Body — Verify:**

```json
{
  "status": "verified"
}
```

**Request Body — Reject:**

```json
{
  "status": "rejected",
  "rejection_reason": "Incomplete documentation. Please provide a valid GST certificate."
}
```

> [!IMPORTANT]
> `rejection_reason` is **required** when `status` is `rejected`.

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Company status updated to verified successfully."
}
```

**Error — Already Verified:**

```json
{
  "statusCode": 400,
  "message": "Company is already verified."
}
```

---

## 3. Public Company Endpoints

> No authentication required.

### 3.1 View Public Company Profile

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/profile/public/{slug}/` |
| **Auth** |  Public |
| **Role** | Anyone |
| **Usage** | View the public-facing profile of a verified company. Only companies with `status=verified` are accessible. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `slug` | string | Company's URL slug (e.g. `techcorp-india`) |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "id": "c3d1e2f4-1234-5678-abcd-ef0123456789",
    "name": "TechCorp India",
    "slug": "techcorp-india",
    "logo": "https://cdn.example.com/logos/techcorp.png",
    "description": "A leading fintech startup building innovative payment solutions.",
    "short_pitch": "We simplify payments for Bharat.",
    "industry_sector": "Fintech",
    "website_link": "https://techcorp.in",
    "email": "hello@techcorp.in",
    "location": "Kochi, Kerala",
    "district_name": "Ernakulam",
    "state_name": "Kerala",
    "country_name": "India",
    "company_size": "11-50",
    "linkedin_url": "https://linkedin.com/company/techcorp-india",
    "founded_year": 2021,
    "remote_policy": "Hybrid",
    "culture_text": "We value curiosity and ownership.",
    "tech_stack": ["Django", "React", "PostgreSQL"],
    "perks": ["Health Insurance", "Remote Fridays", "Learning Budget"],
    "testimonials": [],
    "gallery": []
  }
}
```

**Error `404`:**

```json
{
  "statusCode": 404,
  "message": "Company not found."
}
```

---

### 3.2 View Public Jobs for a Specific Company

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/profile/public/{slug}/jobs/` |
| **Auth** |  Public |
| **Role** | Anyone |
| **Usage** | List all active, non-deleted jobs posted by a specific verified company. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `slug` | string | Company slug |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by title, location, job_type |
| `sort_by` | string | No | `title` or `created_at` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "job-uuid-1",
        "company_name": "TechCorp India",
        "company_logo": "https://cdn.example.com/logos/techcorp.png",
        "title": "Backend Engineer",
        "experience": "1-3 years",
        "job_description": "Build and maintain APIs using Django REST Framework.",
        "location": "Kochi / Remote",
        "salary_range": "6-10 LPA",
        "job_type": "Full-Time",
        "status": "Active",
        "duration_value": null,
        "duration_unit": null,
        "hourly_rate": null,
        "deliverables": null,
        "stipend": null,
        "certificate_provided": false,
        "rules": [
          { "id": "rule-uuid-1", "rule_type": "min_karma", "rule_value": "500" }
        ],
        "created_at": "2025-02-01T08:00:00Z"
      }
    ],
    "pagination": {
      "count": 3,
      "total_pages": 1,
      "current_page": 1,
      "per_page": 10,
      "next": null,
      "previous": null
    }
  }
}
```

---

### 3.3 List All Active Jobs (Public)

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/all/` |
| **Auth** |  Private (JWT required, but no specific role check) |
| **Role** |  Any authenticated user |
| **Usage** | Browse all active jobs across all verified companies. Supports full-text search and pagination. |

> [!NOTE]
> While declared under `PublicJobAPI`, this endpoint uses `CustomizePermission` so a valid JWT token is still required.

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by title, location, job_type, company name |
| `sort_by` | string | No | `title` or `created_at` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "job-uuid-1",
        "company_name": "TechCorp India",
        "company_logo": "https://cdn.example.com/logos/techcorp.png",
        "title": "Backend Engineer",
        "experience": "1-3 years",
        "job_description": "Build and maintain APIs using Django REST Framework.",
        "location": "Kochi / Remote",
        "salary_range": "6-10 LPA",
        "job_type": "Full-Time",
        "status": "Active",
        "duration_value": null,
        "duration_unit": null,
        "hourly_rate": null,
        "deliverables": null,
        "stipend": null,
        "certificate_provided": false,
        "rules": [],
        "created_at": "2025-02-01T08:00:00Z"
      },
      {
        "id": "job-uuid-2",
        "company_name": "DevHub Solutions",
        "company_logo": null,
        "title": "React Developer Gig",
        "experience": "Fresher",
        "job_description": "Build a dashboard UI in React.",
        "location": "Remote",
        "salary_range": null,
        "job_type": "Gig",
        "status": "Active",
        "duration_value": 2,
        "duration_unit": "Weeks",
        "hourly_rate": 500.00,
        "deliverables": "Fully functional dashboard with unit tests.",
        "stipend": null,
        "certificate_provided": true,
        "rules": [],
        "created_at": "2025-02-05T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 175,
      "total_pages": 18,
      "current_page": 1,
      "per_page": 10,
      "next": "/api/v1/dashboard/company/jobs/all/?page=2",
      "previous": null
    }
  }
}
```

---

## 4. Company Jobs & Gigs

> All endpoints require a verified company (creator or approved Company Mentor).

### 4.1 List Company's Own Jobs

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | Retrieve all non-deleted jobs posted by your company (includes all statuses — Active, Closed, Draft). |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by title, location, job_type |
| `sort_by` | string | No | `title` or `created_at` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "job-uuid-1",
        "company_name": "TechCorp India",
        "company_logo": "https://cdn.example.com/logos/techcorp.png",
        "title": "Backend Engineer",
        "experience": "1-3 years",
        "job_description": "Build and maintain APIs.",
        "location": "Kochi / Remote",
        "salary_range": "6-10 LPA",
        "job_type": "Full-Time",
        "status": "Active",
        "duration_value": null,
        "duration_unit": null,
        "hourly_rate": null,
        "deliverables": null,
        "stipend": null,
        "certificate_provided": false,
        "rules": [
          { "id": "rule-uuid-1", "rule_type": "min_karma", "rule_value": "500" }
        ],
        "created_at": "2025-02-01T08:00:00Z"
      }
    ],
    "pagination": {
      "count": 12,
      "total_pages": 2,
      "current_page": 1,
      "per_page": 10,
      "next": null,
      "previous": null
    }
  }
}
```

---

### 4.2 Post a New Job / Gig

| Field | Value |
|-------|-------|
| **Method** | `POST` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | Create a new job or gig listing under the authenticated user's verified company. |

**Request Body — Full-Time Job:**

```json
{
  "title": "Backend Engineer",
  "experience": "1-3 years",
  "job_description": "Build and maintain REST APIs using Django REST Framework. Collaborate with frontend team.",
  "location": "Kochi / Remote",
  "salary_range": "6-10 LPA",
  "job_type": "Full-Time",
  "status": "Active",
  "certificate_provided": false,
  "rules": [
    { "rule_type": "min_karma", "rule_value": "500" },
    { "rule_type": "min_level", "rule_value": "3" }
  ]
}
```

**Request Body — Gig:**

```json
{
  "title": "React Dashboard UI",
  "experience": "Fresher",
  "job_description": "Design and build a responsive admin dashboard using React and Tailwind.",
  "location": "Remote",
  "job_type": "Gig",
  "status": "Active",
  "duration_value": 2,
  "duration_unit": "Weeks",
  "hourly_rate": 500.00,
  "deliverables": "Fully functional dashboard with unit tests and Storybook documentation.",
  "stipend": 5000.00,
  "certificate_provided": true,
  "rules": []
}
```

**Available `rule_type` values:**

| rule_type | Description |
|-----------|-------------|
| `min_karma` | Minimum karma score required to apply |
| `max_karma` | Maximum karma score allowed |
| `min_level` | Minimum platform level required |
| `max_level` | Maximum platform level allowed |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Job posted successfully.",
  "response": {
    "id": "job-uuid-3",
    "title": "Backend Engineer",
    "experience": "1-3 years",
    "job_description": "Build and maintain REST APIs using Django REST Framework.",
    "location": "Kochi / Remote",
    "salary_range": "6-10 LPA",
    "job_type": "Full-Time",
    "status": "Active",
    "duration_value": null,
    "duration_unit": null,
    "hourly_rate": null,
    "deliverables": null,
    "stipend": null,
    "certificate_provided": false,
    "rules": [
      { "id": "rule-uuid-1", "rule_type": "min_karma", "rule_value": "500" },
      { "id": "rule-uuid-2", "rule_type": "min_level", "rule_value": "3" }
    ]
  }
}
```

---

### 4.3 Get Job Details

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/{job_id}/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor (own company jobs only) |
| **Usage** | Fetch details of a specific job by ID. Only accessible for your own company's jobs. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | UUID string | Job's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "id": "job-uuid-1",
    "company_name": "TechCorp India",
    "company_logo": "https://cdn.example.com/logos/techcorp.png",
    "title": "Backend Engineer",
    "experience": "1-3 years",
    "job_description": "Build and maintain REST APIs using Django REST Framework.",
    "location": "Kochi / Remote",
    "salary_range": "6-10 LPA",
    "job_type": "Full-Time",
    "status": "Active",
    "duration_value": null,
    "duration_unit": null,
    "hourly_rate": null,
    "deliverables": null,
    "stipend": null,
    "certificate_provided": false,
    "rules": [
      { "id": "rule-uuid-1", "rule_type": "min_karma", "rule_value": "500" }
    ],
    "created_at": "2025-02-01T08:00:00Z"
  }
}
```

---

### 4.4 Update a Job

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/{job_id}/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | Partially update a job. If `rules` is provided, **all existing rules are replaced** with the new set. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | UUID string | Job's unique ID |

**Request Body (all fields optional):**

```json
{
  "status": "Closed",
  "salary_range": "8-12 LPA",
  "rules": [
    { "rule_type": "min_karma", "rule_value": "1000" }
  ]
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Job updated successfully.",
  "response": {
    "title": "Backend Engineer",
    "experience": "1-3 years",
    "job_description": "Build and maintain REST APIs.",
    "location": "Kochi / Remote",
    "salary_range": "8-12 LPA",
    "job_type": "Full-Time",
    "status": "Closed",
    "duration_value": null,
    "duration_unit": null,
    "hourly_rate": null,
    "deliverables": null,
    "stipend": null,
    "certificate_provided": false,
    "rules": [
      { "id": "rule-uuid-new", "rule_type": "min_karma", "rule_value": "1000" }
    ]
  }
}
```

---

### 4.5 Delete a Job (Soft Delete)

| Field | Value |
|-------|-------|
| **Method** | `DELETE` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/{job_id}/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | Soft-deletes a job (`is_deleted=True`). The job is no longer visible to applicants or in public listings. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | UUID string | Job's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Job deleted successfully."
}
```

**Error `404`:**

```json
{
  "statusCode": 404,
  "message": "Job not found or access denied."
}
```

---

## 5. Job Applications

### 5.1 Apply to a Job

| Field | Value |
|-------|-------|
| **Method** | `POST` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/{job_id}/apply/` |
| **Auth** |  Private |
| **Role** |  Any authenticated user |
| **Usage** | Apply to an active job. The system validates karma and level rules before allowing the application. Duplicate applications are rejected. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | UUID string | ID of the active job to apply to |

**Request Body:**

```json
{
  "resume_link": "https://drive.google.com/file/d/abc123/view",
  "cover_letter": "I am excited to apply for the Backend Engineer role at TechCorp India. With 2 years of Django experience..."
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Application submitted successfully."
}
```

**Error — Duplicate Application:**

```json
{
  "statusCode": 400,
  "message": {
    "non_field_errors": ["You have already applied for this job."]
  }
}
```

**Error — Karma Rule Violated:**

```json
{
  "statusCode": 400,
  "message": {
    "non_field_errors": ["Insufficient Karma. Minimum 500 required."]
  }
}
```

---

### 5.2 List Applications for a Job (Company View)

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/jobs/{job_id}/applications/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | View all applications received for a specific job posted by your company. Includes applicant details and current status. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `job_id` | UUID string | Job's unique ID |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by applicant name or status |
| `sort_by` | string | No | `applied_at` or `status` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "app-uuid-1",
        "job": "job-uuid-1",
        "applicant_name": "Arjun Menon",
        "applicant_email": "arjun@example.com",
        "resume_link": "https://drive.google.com/file/d/abc123/view",
        "cover_letter": "I am excited to apply...",
        "status": "Pending",
        "rejection_reason": null,
        "applied_at": "2025-02-10T14:30:00Z"
      },
      {
        "id": "app-uuid-2",
        "job": "job-uuid-1",
        "applicant_name": "Priya Nair",
        "applicant_email": "priya@example.com",
        "resume_link": "https://drive.google.com/file/d/xyz789/view",
        "cover_letter": "I bring 3 years of Django experience...",
        "status": "Shortlisted",
        "rejection_reason": null,
        "applied_at": "2025-02-11T09:00:00Z"
      }
    ],
    "pagination": {
      "count": 25,
      "total_pages": 3,
      "current_page": 1,
      "per_page": 10,
      "next": "/api/v1/dashboard/company/jobs/job-uuid-1/applications/?page=2",
      "previous": null
    }
  }
}
```

---

### 5.3 Update Application Status (Company)

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/applications/{app_id}/status/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | Move an application through the hiring funnel. Optionally provide a rejection reason when rejecting. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | UUID string | Application's unique ID |

**Available `status` values:**

`Pending` → `In-Review` → `Shortlisted` → `Interview` → `Selected` / `Rejected`

**Request Body:**

```json
{
  "status": "Shortlisted"
}
```

**Request Body (Rejection):**

```json
{
  "status": "Rejected",
  "rejection_reason": "Profile does not match the required experience level."
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Application status updated successfully.",
  "response": {
    "id": "app-uuid-1",
    "job": "job-uuid-1",
    "applicant_name": "Arjun Menon",
    "applicant_email": "arjun@example.com",
    "resume_link": "https://drive.google.com/file/d/abc123/view",
    "cover_letter": "I am excited to apply...",
    "status": "Shortlisted",
    "rejection_reason": null,
    "applied_at": "2025-02-10T14:30:00Z"
  }
}
```

---

### 5.4 List My Applications (User View)

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/applications/me/` |
| **Auth** |  Private |
| **Role** |  Any authenticated user |
| **Usage** | Retrieve all job applications submitted by the currently authenticated user. Includes full job details for each application. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `search` | string | No | Search by job title, company name, or status |
| `sort_by` | string | No | `applied_at` or `status` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "app-uuid-1",
        "job": {
          "id": "job-uuid-1",
          "company_name": "TechCorp India",
          "company_logo": "https://cdn.example.com/logos/techcorp.png",
          "title": "Backend Engineer",
          "experience": "1-3 years",
          "job_description": "Build and maintain REST APIs.",
          "location": "Kochi / Remote",
          "salary_range": "6-10 LPA",
          "job_type": "Full-Time",
          "status": "Active",
          "rules": [],
          "created_at": "2025-02-01T08:00:00Z"
        },
        "resume_link": "https://drive.google.com/file/d/abc123/view",
        "cover_letter": "I am excited to apply...",
        "status": "Shortlisted",
        "rejection_reason": null,
        "applied_at": "2025-02-10T14:30:00Z"
      }
    ],
    "pagination": {
      "count": 3,
      "total_pages": 1,
      "current_page": 1,
      "per_page": 10,
      "next": null,
      "previous": null
    }
  }
}
```

---

### 5.5 Withdraw an Application

| Field | Value |
|-------|-------|
| **Method** | `DELETE` |
| **Endpoint** | `/api/v1/dashboard/company/applications/{app_id}/withdraw/` |
| **Auth** |  Private |
| **Role** |  The applicant (own applications only) |
| **Usage** | Permanently withdraw (hard delete) your own job application. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | UUID string | Application's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Application withdrawn successfully."
}
```

**Error `404`:**

```json
{
  "statusCode": 404,
  "message": "Application not found or you do not have permission to withdraw it."
}
```

---

### 5.6 Resubmit a Rejected Application

| Field | Value |
|-------|-------|
| **Method** | `PATCH` |
| **Endpoint** | `/api/v1/dashboard/company/applications/{app_id}/resubmit/` |
| **Auth** |  Private |
| **Role** |  The applicant (own rejected applications only) |
| **Usage** | Resubmit an application that was previously rejected. Updates resume/cover letter and resets status to `Pending`. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `app_id` | UUID string | Application's unique ID |

**Request Body (all fields optional):**

```json
{
  "resume_link": "https://drive.google.com/file/d/new-resume/view",
  "cover_letter": "I have addressed the previous concerns. I now have 2+ years of Django experience..."
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Application resubmitted successfully."
}
```

**Error — Not Rejected:**

```json
{
  "statusCode": 400,
  "message": "Only rejected applications can be resubmitted."
}
```

---

## 6. Company Tasks

> Tasks submitted by companies go through an admin approval flow. Status is always `pending` (and `active=false`) until an admin approves.

### 6.1 List Company Tasks

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/tasks/` |
| **Auth** |  Private |
| **Role** |  Company creator or approved Company Mentor |
| **Usage** | List all tasks submitted by the authenticated company user. Supports filtering by approval status, search, and sort. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `approval_status` | string | No | Filter: `pending`, `approved`, `rejected` |
| `search` | string | No | Search by hashtag, title, description, karma, IG name, type |
| `sort_by` | string | No | `hashtag`, `title`, `karma`, `ig`, `type`, `approval_status`, `created_at`, `updated_at` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "task-uuid-1",
        "hashtag": "#techcorp-backend-challenge",
        "discord_link": null,
        "title": "Build a REST API",
        "description": "Create a fully documented REST API using Django REST Framework.",
        "karma": 150,
        "channel": null,
        "type": "Task",
        "active": false,
        "variable_karma": false,
        "usage_count": 0,
        "level": null,
        "org": null,
        "ig": null,
        "event": null,
        "bonus_karma": 0,
        "bonus_time": null,
        "approval_status": "pending",
        "rejection_reason": null,
        "reviewed_at": null,
        "requested_by_name": "Jane Doe",
        "requested_at": "2025-03-01T10:00:00Z",
        "skills": [
          { "id": "skill-uuid-1", "name": "Django", "code": "DJANGO" }
        ],
        "created_at": "2025-03-01T10:00:00Z",
        "updated_at": "2025-03-01T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 5,
      "total_pages": 1,
      "current_page": 1,
      "per_page": 10,
      "next": null,
      "previous": null
    }
  }
}
```

---

### 6.2 Create a Company Task

| Field | Value |
|-------|-------|
| **Method** | `POST` |
| **Endpoint** | `/api/v1/dashboard/company/tasks/` |
| **Auth** |  Private |
| **Role** |  Verified Company creator or approved Company Mentor |
| **Usage** | Submit a new task for admin approval. Task is created with `approval_status=pending` and `active=false`. An optional list of skill IDs can be linked. |

**Request Body:**

```json
{
  "hashtag": "#techcorp-api-challenge",
  "title": "Build a REST API with Django",
  "karma": 150,
  "usage_count": 0,
  "description": "Create a fully documented REST API using Django REST Framework with JWT authentication.",
  "type": "task-type-uuid",
  "level": "level-uuid",
  "created_by": "user-uuid",
  "updated_by": "user-uuid",
  "skill_ids": ["skill-uuid-1", "skill-uuid-2"]
}
```

> [!NOTE]
> `skill_ids` is an optional array of Skill UUIDs. Only active skills will be linked. Can be sent as a JSON array or a JSON string.

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Task submitted for approval."
}
```

**Error — Not Verified Company:**

```json
{
  "statusCode": 403,
  "message": "You must have a verified company profile to submit tasks."
}
```

**Error — Duplicate Hashtag:**

```json
{
  "statusCode": 400,
  "message": {
    "hashtag": ["A task with this hashtag already exists."]
  }
}
```

---

### 6.3 Get Task Details

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/tasks/{task_id}/` |
| **Auth** |  Private |
| **Role** |  Verified Company (task must belong to the authenticated user) |
| **Usage** | Retrieve full details of a specific task submitted by the authenticated user. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `task_id` | UUID string | Task's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "id": "task-uuid-1",
    "hashtag": "#techcorp-api-challenge",
    "discord_link": null,
    "title": "Build a REST API with Django",
    "description": "Create a fully documented REST API using Django REST Framework.",
    "karma": 150,
    "channel": null,
    "type": "Task",
    "active": false,
    "variable_karma": false,
    "usage_count": 0,
    "level": null,
    "org": null,
    "ig": null,
    "event": null,
    "bonus_karma": 0,
    "bonus_time": null,
    "approval_status": "pending",
    "rejection_reason": null,
    "reviewed_at": null,
    "requested_by_name": "Jane Doe",
    "requested_at": "2025-03-01T10:00:00Z",
    "skills": [
      { "id": "skill-uuid-1", "name": "Django", "code": "DJANGO" },
      { "id": "skill-uuid-2", "name": "REST API", "code": "REST" }
    ],
    "created_at": "2025-03-01T10:00:00Z",
    "updated_at": "2025-03-01T10:00:00Z"
  }
}
```

---

### 6.4 Update a Task (Full Resubmit)

| Field | Value |
|-------|-------|
| **Method** | `PUT` |
| **Endpoint** | `/api/v1/dashboard/company/tasks/{task_id}/` |
| **Auth** |  Private |
| **Role** |  Verified Company (task must belong to the authenticated user) |
| **Usage** | Update a task. Regardless of previous status (`approved`, `rejected`, or `pending`), the task reverts to `approval_status=pending` and `active=false` after editing. If `skill_ids` is provided, existing skill links are fully replaced. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `task_id` | UUID string | Task's unique ID |

**Request Body (partial update supported):**

```json
{
  "title": "Build a REST API with Django and JWT",
  "karma": 200,
  "description": "Updated: Create a fully documented REST API with JWT authentication and rate limiting.",
  "skill_ids": ["skill-uuid-1", "skill-uuid-3"]
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Task updated and re-submitted for approval."
}
```

---

### 6.5 Delete a Task

| Field | Value |
|-------|-------|
| **Method** | `DELETE` |
| **Endpoint** | `/api/v1/dashboard/company/tasks/{task_id}/` |
| **Auth** |  Private |
| **Role** |  Verified Company (task must belong to the authenticated user) |
| **Usage** | Hard-delete a task. Only tasks with `approval_status=pending` can be deleted. |

**Path Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `task_id` | UUID string | Task's unique ID |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "Task deleted successfully."
}
```

**Error — Non-Pending Task:**

```json
{
  "statusCode": 400,
  "message": "Cannot delete a task with status 'approved'. Only pending tasks can be deleted."
}
```

---

## 7. Company Mentor Nomination

> Only the verified company **creator** (not a Company Mentor) can nominate mentors.

### 7.1 Nominate a Company Mentor

| Field | Value |
|-------|-------|
| **Method** | `POST` |
| **Endpoint** | `/api/v1/dashboard/company/mentor/nominate/` |
| **Auth** |  Private |
| **Role** |  `COMPANY` role (verified company creator only) |
| **Usage** | Nominate a platform user (by their `muid`) as a Company Mentor. The user must be an existing member of the company's Organization. The nomination is `PENDING` until an admin approves. |

**Pre-conditions:**
- You must have a verified company
- The nominated user must exist on the platform
- The user must already be linked to the company's Organization (`UserOrganizationLink`)
- No existing active (non-rejected) nomination for the same user

**Request Body:**

```json
{
  "muid": "arjun-menon@mulearn",
  "reason": "Arjun has been an active contributor and has strong technical mentoring skills."
}
```

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "message": "User nominated as Company Mentor. Pending admin approval.",
  "response": {
    "id": "mentor-uuid-1",
    "user_id": "user-uuid-of-arjun",
    "user_name": "Arjun Menon",
    "user_email": "arjun@example.com",
    "org_name": "TechCorp India",
    "mentor_tier": "Company Mentor",
    "status": "Pending",
    "reason": "Arjun has been an active contributor and has strong technical mentoring skills.",
    "verification_note": null,
    "verified_at": null
  }
}
```

**Error — User Not in Organization:**

```json
{
  "statusCode": 400,
  "message": {
    "muid": ["User 'arjun-menon@mulearn' is not a member of this company's organisation."]
  }
}
```

**Error — Duplicate Nomination:**

```json
{
  "statusCode": 400,
  "message": "This user already has a Pending Company Mentor nomination for your company."
}
```

---

### 7.2 List Company Mentor Nominations

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/mentor/list/` |
| **Auth** |  Private |
| **Role** |  `COMPANY` role (verified company creator only) |
| **Usage** | List all Company Mentor nominations for your company — including pending, approved, and rejected nominations. |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": [
    {
      "id": "mentor-uuid-1",
      "user_id": "user-uuid-of-arjun",
      "user_name": "Arjun Menon",
      "user_email": "arjun@example.com",
      "org_name": "TechCorp India",
      "mentor_tier": "Company Mentor",
      "status": "Approved",
      "reason": "Arjun has strong technical mentoring skills.",
      "verification_note": "Approved by admin on review.",
      "verified_at": "2025-02-20T11:00:00Z"
    },
    {
      "id": "mentor-uuid-2",
      "user_id": "user-uuid-of-priya",
      "user_name": "Priya Nair",
      "user_email": "priya@example.com",
      "org_name": "TechCorp India",
      "mentor_tier": "Company Mentor",
      "status": "Pending",
      "reason": "Excellent communicator and domain expert.",
      "verification_note": null,
      "verified_at": null
    }
  ]
}
```

---

## 8. MuLearner Directory

### 8.1 Browse MuLearner Directory

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/mulearners/` |
| **Auth** |  Private |
| **Role** |  Verified Company creator or approved Company Mentor |
| **Usage** | Browse public MuLearner profiles. Companies can filter candidates by karma, level, college, department, graduation year, interest group, skill, achievement, or task completion. Only users with `is_public=True` are returned. |

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `min_karma` | integer | No | Minimum karma score |
| `max_karma` | integer | No | Maximum karma score |
| `level` | integer | No | Platform level (level_order value) |
| `college` | string | No | College name (partial match) |
| `department` | string | No | Department name (partial match) |
| `graduation_year` | string | No | Graduation year (e.g. `2025`) |
| `ig` | string | No | Interest Group name (partial match) |
| `skill` | string | No | Skill UUID (exact match) |
| `achievement` | string | No | Achievement UUID |
| `task` | string | No | Task UUID (users who completed this task) |
| `search` | string | No | Search by full name, muid, or email |
| `sort_by` | string | No | `full_name`, `created_at`, `karma` |
| `sort_order` | string | No | `asc` or `desc` |
| `page` | int | No | Page number |
| `per_page` | int | No | Items per page |

**Example Request:**
`GET /api/v1/dashboard/company/mulearners/?min_karma=500&level=3&college=CUSAT&graduation_year=2025`

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "data": [
      {
        "id": "user-uuid-1",
        "full_name": "Arjun Menon",
        "muid": "arjun-menon@mulearn",
        "email": "arjun@example.com",
        "karma": 1250,
        "level": 4,
        "college": "Cochin University of Science and Technology",
        "department": "Computer Science",
        "graduation_year": "2025"
      },
      {
        "id": "user-uuid-2",
        "full_name": "Sneha Krishnan",
        "muid": "sneha-krishnan@mulearn",
        "email": "sneha@example.com",
        "karma": 880,
        "level": 3,
        "college": "TKM College of Engineering",
        "department": "Information Technology",
        "graduation_year": "2025"
      }
    ],
    "pagination": {
      "count": 48,
      "total_pages": 5,
      "current_page": 1,
      "per_page": 10,
      "next": "/api/v1/dashboard/company/mulearners/?min_karma=500&level=3&page=2",
      "previous": null
    }
  }
}
```

---

## 9. Analytics

### 9.1 Company Gig Analytics

| Field | Value |
|-------|-------|
| **Method** | `GET` |
| **Endpoint** | `/api/v1/dashboard/company/analytics/gigs/` |
| **Auth** |  Private |
| **Role** |  Verified Company creator or approved Company Mentor |
| **Usage** | Retrieve analytics for all gig-type jobs posted by your company — including counts, average hourly rate, full application funnel, and conversion rate. |

**Success Response `200 OK`:**

```json
{
  "statusCode": 200,
  "response": {
    "total_gigs_posted": 12,
    "active_gigs": 7,
    "closed_gigs": 5,
    "average_hourly_rate": 425.50,
    "application_funnel": {
      "Total": 134,
      "Pending": 45,
      "In-Review": 30,
      "Shortlisted": 25,
      "Interview": 18,
      "Selected": 10,
      "Rejected": 6
    },
    "conversion_rate": "7.46%"
  }
}
```

**Error `404` (No verified company):**

```json
{
  "statusCode": 404,
  "message": "Company profile not found or access denied."
}
```

---

## Quick Reference Table

| # | Method | Endpoint | Auth | Role | Public |
|---|--------|----------|------|------|--------|
| 1.1 | `POST` | `/register/` | JWT | Any Auth |  |
| 1.2 | `PATCH` | `/register/` | JWT | Any Auth |  |
| 1.3 | `GET` | `/status/` | JWT | Any Auth |  |
| 1.4 | `GET` | `/profile/` | JWT | Company/Mentor |  |
| 1.5 | `PATCH` | `/profile/` | JWT | Company/Mentor |  |
| 2.1 | `GET` | `/summary/` | JWT | Admin |  |
| 2.2 | `GET` | `/list/` | JWT | Admin |  |
| 2.3 | `GET` | `/{company_id}/` | JWT | Admin |  |
| 2.4 | `PATCH` | `/verify/{company_id}/` | JWT | Admin |  |
| 3.1 | `GET` | `/profile/public/{slug}/` | None | Anyone |  |
| 3.2 | `GET` | `/profile/public/{slug}/jobs/` | None | Anyone |  |
| 3.3 | `GET` | `/jobs/all/` | JWT | Any Auth |  |
| 4.1 | `GET` | `/jobs/` | JWT | Company/Mentor |  |
| 4.2 | `POST` | `/jobs/` | JWT | Company/Mentor |  |
| 4.3 | `GET` | `/jobs/{job_id}/` | JWT | Company/Mentor |  |
| 4.4 | `PATCH` | `/jobs/{job_id}/` | JWT | Company/Mentor |  |
| 4.5 | `DELETE` | `/jobs/{job_id}/` | JWT | Company/Mentor |  |
| 5.1 | `POST` | `/jobs/{job_id}/apply/` | JWT | Any Auth |  |
| 5.2 | `GET` | `/jobs/{job_id}/applications/` | JWT | Company/Mentor |  |
| 5.3 | `PATCH` | `/applications/{app_id}/status/` | JWT | Company/Mentor |  |
| 5.4 | `GET` | `/applications/me/` | JWT | Any Auth |  |
| 5.5 | `DELETE` | `/applications/{app_id}/withdraw/` | JWT | Applicant |  |
| 5.6 | `PATCH` | `/applications/{app_id}/resubmit/` | JWT | Applicant |  |
| 6.1 | `GET` | `/tasks/` | JWT | Company/Mentor |  |
| 6.2 | `POST` | `/tasks/` | JWT | Company/Mentor |  |
| 6.3 | `GET` | `/tasks/{task_id}/` | JWT | Company/Mentor |  |
| 6.4 | `PUT` | `/tasks/{task_id}/` | JWT | Company/Mentor |  |
| 6.5 | `DELETE` | `/tasks/{task_id}/` | JWT | Company/Mentor |  |
| 7.1 | `POST` | `/mentor/nominate/` | JWT | Company (creator) |  |
| 7.2 | `GET` | `/mentor/list/` | JWT | Company (creator) |  |
| 8.1 | `GET` | `/mulearners/` | JWT | Company/Mentor |  |
| 9.1 | `GET` | `/analytics/gigs/` | JWT | Company/Mentor |  |

---

> **Note on Error Responses:** All error responses follow the format:
> ```json
> { "statusCode": 4xx, "message": "Error description" }
> ```
> Authentication failures return `401 Unauthorized`. Role/permission failures return `403 Forbidden`.
