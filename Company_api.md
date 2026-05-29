# Company Dashboard API Documentation

This document provides exhaustive documentation for the Company module in the dashboard. All endpoints are prefixed with `/api/v1/dashboard/company/` unless otherwise noted.

**Authentication:** All requests require a valid JWT Bearer token in the `Authorization` header (`Authorization: Bearer <token>`).

---

## 1. Company Registration & Profile Management

### 1.1 Register Company
- **Method:** `POST`
- **Endpoint:** `/register/`
- **Required Role:** Any authenticated user
- **Usage Scenario:** A user wants to register their organization as a Company on the platform.
- **Description:** Submits a company registration request. The `district_id`, `state_id`, and `country_id` fields require valid UUIDs from their respective tables.

**Request JSON:**
```json
{
  "name": "Acme Corp",
  "logo": "https://example.com/logo.png",
  "description": "A leading provider of everything.",
  "short_pitch": "We build great things.",
  "industry_sector": "IT",
  "email": "contact@acme.com",
  "location": "New York",
  "district_id": "90db1744-8848-43d9-952c-bd9f2b87a912",
  "state_id": "1198be19-215c-4d3f-bfe7-b768a35606e9",
  "country_id": "4b27ff54-722a-43dc-aecc-63ecf6dcdb21",
  "legal_name": "Acme Corporation Ltd.",
  "registration_number": "CIN12345678",
  "tax_id": "GSTIN987654",
  "company_size": "50-200",
  "linkedin_url": "https://linkedin.com/company/acmecorp",
  "website_link": "https://acme.com",
  "founded_year": 2015,
  "remote_policy": "Hybrid",
  "culture_text": "We value innovation and speed.",
  "tech_stack": "React, Python, AWS",
  "perks": "Health Insurance, Free Lunches",
  "testimonials": "[{\"name\": \"John\", \"quote\": \"Great place!\"}]",
  "gallery": "[\"https://link1.png\", \"https://link2.png\"]"
}
```

**Response JSON (Success - 200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Company registration submitted successfully."] },
  "response": {
    "name": "Acme Corp",
    "slug": "acme-corp",
    "status": "pending",
    "email": "contact@acme.com",
    "industry_sector": "IT",
    "location": "New York",
    "company_size": "50-200"
  }
}
```

### 1.2 Check Registration Status
- **Method:** `GET`
- **Endpoint:** `/status/`
- **Required Role:** Any authenticated user
- **Usage Scenario:** To check whether the user's company registration is pending, verified, or rejected.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "status": "pending",
    "rejection_reason": null,
    "company_id": "89000a6e-4f05-4f76-8806-039c095797d1",
    "name": "Acme Corp",
    "slug": "acme-corp"
  }
}
```

### 1.3 Get My Company Profile
- **Method:** `GET`
- **Endpoint:** `/profile/`
- **Required Role:** `Company`
- **Usage Scenario:** Fetching the company's own profile data for display on the dashboard settings page.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "id": "89000a6e-4f05-4f76-8806-039c095797d1",
    "company_user_id": "a9188a45-6a52-4416-834f-9e797ebcf043",
    "company_user_name": "John Doe",
    "company_user_email": "johndoe@example.com",
    "name": "Acme Corp",
    "slug": "acme-corp",
    "status": "verified",
    "logo": "https://example.com/logo.png",
    "description": "A leading provider of everything.",
    "short_pitch": "We build great things.",
    "industry_sector": "IT",
    "website_link": "https://acme.com",
    "email": "contact@acme.com",
    "location": "New York",
    "district_name": "Ernakulam",
    "district": "90db1744-8848-43d9-952c-bd9f2b87a912",
    "state": "1198be19-215c-4d3f-bfe7-b768a35606e9",
    "country": "4b27ff54-722a-43dc-aecc-63ecf6dcdb21",
    "legal_name": "Acme Corporation Ltd.",
    "registration_number": "CIN12345678",
    "tax_id": "GSTIN987654",
    "company_size": "50-200",
    "linkedin_url": "https://linkedin.com/company/acmecorp",
    "founded_year": 2015,
    "remote_policy": "Hybrid",
    "culture_text": "We value innovation and speed.",
    "tech_stack": "React, Python, AWS",
    "perks": "Health Insurance, Free Lunches",
    "testimonials": "[{\"name\": \"John\", \"quote\": \"Great place!\"}]",
    "gallery": "[\"https://link1.png\", \"https://link2.png\"]",
    "rejection_reason": null,
    "verification_requested_at": "2026-05-29T10:00:00Z",
    "verified_at": "2026-05-29T11:00:00Z",
    "created_at": "2026-05-29T10:00:00Z",
    "updated_at": "2026-05-29T12:00:00Z"
  }
}
```

### 1.4 Update My Company Profile
- **Method:** `PATCH`
- **Endpoint:** `/profile/`
- **Required Role:** `Company`
- **Usage Scenario:** Updating specific company details without sending the full object.

**Request JSON:**
```json
{
  "description": "This is our newly updated company description.",
  "website_link": "https://acme-new.com",
  "perks": "Health Insurance, Remote Work, Unlimited PTO",
  "company_size": "201-500"
}
```

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Company profile updated successfully."] },
  "response": {
    "name": "Acme Corp",
    "description": "This is our newly updated company description.",
    "website_link": "https://acme-new.com",
    "perks": "Health Insurance, Remote Work, Unlimited PTO",
    "company_size": "201-500",
    "district_id": "90db1744-8848-43d9-952c-bd9f2b87a912"
  }
}
```

---

## 2. Admin Operations

### 2.1 List All Companies
- **Method:** `GET`
- **Endpoint:** `/list/`
- **Required Role:** `Admin`
- **Usage Scenario:** The admin panel view showing all companies.
- **Query Params:** `?pageIndex=1&perPage=10&search=Acme&status=pending&industry_sector=IT&district=Ernakulam&state=Kerala&country=India`

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "89000a6e-4f05-4f76-8806-039c095797d1",
        "name": "Acme Corp",
        "slug": "acme-corp",
        "status": "pending",
        "email": "contact@acme.com",
        "company_user_id": "a9188a45-6a52-4416-834f-9e797ebcf043",
        "company_user_name": "John Doe",
        "industry_sector": "IT",
        "company_size": "50-200",
        "location": "New York",
        "district_name": "Ernakulam",
        "state_name": "Kerala",
        "country_name": "India",
        "verification_requested_at": "2026-05-29T10:00:00Z",
        "verified_at": null
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

### 2.2 Get Specific Company Details (Admin)
- **Method:** `GET`
- **Endpoint:** `/<str:company_id>/`
- **Required Role:** `Admin`
- **Usage Scenario:** Admin viewing the full details of a specific company.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "id": "89000a6e-4f05-4f76-8806-039c095797d1",
    "name": "Acme Corp",
    "status": "pending",
    "company_user_name": "John Doe",
    "company_user_email": "johndoe@example.com",
    "district_name": "Ernakulam"
    // ... Returns the full model fields
  }
}
```

### 2.3 Verify or Reject Company
- **Method:** `PATCH`
- **Endpoint:** `/verify/<str:company_id>/`
- **Required Role:** `Admin`
- **Usage Scenario:** Approving or rejecting a pending company registration.

**Request JSON (To Verify):**
```json
{
  "status": "verified"
}
```
**Request JSON (To Reject):**
```json
{
  "status": "rejected",
  "rejection_reason": "Documentation provided does not match company legal name."
}
```

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Company status updated to verified successfully."] },
  "response": {}
}
```

---

## 3. Job Management

### 3.1 Create a Job
- **Method:** `POST`
- **Endpoint:** `/jobs/`
- **Required Role:** `Company`
- **Usage Scenario:** Creating a new job posting with optional rule requirements (e.g., minimum karma, specific level).

**Request JSON:**
```json
{
  "title": "Frontend Developer",
  "experience": "2-3 Years",
  "job_description": "We are looking for an experienced React developer.",
  "location": "Remote",
  "salary_range": "60,000 - 80,000",
  "job_type": "Full-Time",
  "status": "Active",
  "duration_value": 0,
  "duration_unit": "months",
  "hourly_rate": 0,
  "deliverables": "Write scalable code, code reviews",
  "stipend": 0,
  "certificate_provided": false,
  "rules": [
    { "rule_type": "min_karma", "rule_value": "1000" },
    { "rule_type": "min_level", "rule_value": "3" }
  ]
}
```

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Job posted successfully."] },
  "response": {
    "id": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
    "title": "Frontend Developer",
    "experience": "2-3 Years",
    "job_description": "We are looking for an experienced React developer.",
    "location": "Remote",
    "salary_range": "60,000 - 80,000",
    "job_type": "Full-Time",
    "status": "Active",
    "duration_value": 0,
    "duration_unit": "months",
    "hourly_rate": 0,
    "deliverables": "Write scalable code, code reviews",
    "stipend": 0,
    "certificate_provided": false,
    "rules": [
      { "id": 1, "rule_type": "min_karma", "rule_value": "1000" },
      { "id": 2, "rule_type": "min_level", "rule_value": "3" }
    ]
  }
}
```

### 3.2 List Company Jobs
- **Method:** `GET`
- **Endpoint:** `/jobs/`
- **Required Role:** `Company`
- **Usage Scenario:** Viewing a list of jobs posted by the logged-in company.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
        "company_name": "Acme Corp",
        "company_logo": "https://example.com/logo.png",
        "title": "Frontend Developer",
        "experience": "2-3 Years",
        "job_description": "We are looking for an experienced React developer.",
        "location": "Remote",
        "salary_range": "60,000 - 80,000",
        "job_type": "Full-Time",
        "status": "Active",
        "duration_value": 0,
        "duration_unit": "months",
        "hourly_rate": 0,
        "deliverables": "Write scalable code, code reviews",
        "stipend": 0,
        "certificate_provided": false,
        "created_at": "2026-05-29T10:00:00Z",
        "rules": [
          { "id": 1, "rule_type": "min_karma", "rule_value": "1000" }
        ]
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

### 3.3 Get Specific Job
- **Method:** `GET`
- **Endpoint:** `/jobs/<str:job_id>/`
- **Required Role:** `Company`
- **Usage Scenario:** Fetching all data for a single job posting.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "id": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
    "company_name": "Acme Corp",
    "company_logo": "https://example.com/logo.png",
    "title": "Frontend Developer",
    "rules": []
    // ... all other job fields
  }
}
```

### 3.4 Update a Job
- **Method:** `PATCH`
- **Endpoint:** `/jobs/<str:job_id>/`
- **Required Role:** `Company`
- **Usage Scenario:** Updating an existing job post. Passing the `rules` array will completely overwrite all existing rules for the job.

**Request JSON:**
```json
{
  "title": "Senior Frontend Developer",
  "salary_range": "80,000 - 100,000",
  "rules": [
    { "rule_type": "min_level", "rule_value": "4" }
  ]
}
```

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Job updated successfully."] },
  "response": {
    "title": "Senior Frontend Developer",
    "salary_range": "80,000 - 100,000",
    "rules": [
      { "id": 3, "rule_type": "min_level", "rule_value": "4" }
    ]
  }
}
```

### 3.5 Delete a Job
- **Method:** `DELETE`
- **Endpoint:** `/jobs/<str:job_id>/`
- **Required Role:** `Company`
- **Usage Scenario:** Soft-deleting a job so it no longer accepts applicants or shows on the public board.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Job deleted successfully."] },
  "response": {}
}
```

### 3.6 List All Public Jobs
- **Method:** `GET`
- **Endpoint:** `/jobs/all/`
- **Required Role:** Any authenticated user
- **Usage Scenario:** A user browsing the public job board to see all active jobs.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
        "company_name": "Acme Corp",
        "title": "Frontend Developer"
      }
    ],
    "pagination": { ... }
  }
}
```

---

## 4. Job Applications

### 4.1 Apply for a Job
- **Method:** `POST`
- **Endpoint:** `/jobs/<str:job_id>/apply/`
- **Required Role:** Any authenticated user
- **Usage Scenario:** A MuLearner applying for a job. Returns an error if the user doesn't meet the job's requirements (e.g. Karma or Level rules).

**Request JSON:**
```json
{
  "resume_link": "https://drive.google.com/file/d/.../view",
  "cover_letter": "I would love to work here because..."
}
```

**Response JSON (Success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Successfully applied for the job."] },
  "response": {
    "id": "2b3cd6a9-8255-46f3-a3bc-22345d2f62b1",
    "job": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
    "resume_link": "https://drive.google.com/file/d/.../view",
    "cover_letter": "I would love to work here because...",
    "status": "pending"
  }
}
```

**Response JSON (Failure - Doesn't meet rules):**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "non_field_errors": ["Insufficient Karma. Minimum 1000 required."]
  },
  "response": {}
}
```

### 4.2 List Applications for a Job
- **Method:** `GET`
- **Endpoint:** `/jobs/<str:job_id>/applications/`
- **Required Role:** `Company`
- **Usage Scenario:** HR viewing all candidates who applied for a specific job posting.

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "2b3cd6a9-8255-46f3-a3bc-22345d2f62b1",
        "job": "19b5b5c7-2c93-4a11-b0e2-6323f46f48a1",
        "applicant_name": "Jane Doe",
        "applicant_email": "jane@example.com",
        "resume_link": "https://drive.google.com/file/d/.../view",
        "cover_letter": "I would love to work here because...",
        "status": "pending",
        "rejection_reason": null,
        "applied_at": "2026-05-29T10:00:00Z"
      }
    ],
    "pagination": { ... }
  }
}
```

### 4.3 Update Application Status
- **Method:** `PATCH`
- **Endpoint:** `/applications/<str:app_id>/status/`
- **Required Role:** `Company`
- **Usage Scenario:** Updating an applicant's status to `reviewed`, `shortlisted`, `rejected`, or `hired`.

**Request JSON (Approve/Shortlist):**
```json
{
  "status": "shortlisted"
}
```

**Request JSON (Reject):**
```json
{
  "status": "rejected",
  "rejection_reason": "Not enough experience."
}
```

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["Application status updated successfully."] },
  "response": {
    "id": "2b3cd6a9-8255-46f3-a3bc-22345d2f62b1",
    "status": "shortlisted",
    "rejection_reason": null
  }
}
```

---

## 5. Directory & Analytics

### 5.1 MuLearner Directory
- **Method:** `GET`
- **Endpoint:** `/mulearners/`
- **Required Role:** `Company`
- **Usage Scenario:** Sourcing talent from the platform. Lists verified users with privacy filtering applied (mobile numbers are excluded).

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": [
      {
        "id": "673f8e5b-43ac-4b09-b68a-9f5b61decf72",
        "full_name": "Jane Doe",
        "email": "jane@example.com",
        "karma": 1200,
        "level": 4,
        "college": "Tech University",
        "district": "Ernakulam"
      }
    ],
    "pagination": { ... }
  }
}
```

### 5.2 Gig Analytics (Funnel)
- **Method:** `GET`
- **Endpoint:** `/analytics/gigs/`
- **Required Role:** `Company`
- **Usage Scenario:** Viewing the conversion funnel of the company's active jobs (Views vs Applications vs Hires).

**Response JSON:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": {
    "data": {
      "total_jobs_posted": 5,
      "total_views": 1500,
      "total_applications": 120,
      "total_shortlisted": 15,
      "total_hired": 2,
      "conversion_rate_views_to_applications": "8.00%",
      "conversion_rate_applications_to_hires": "1.67%"
    }
  }
}
```
