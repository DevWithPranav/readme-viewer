# 1. Company Mentor API Documentation

This document serves as the backend API reference for the Company Mentor functionalities, built into the Company Dashboard module.

## 1.1. Overview

### 1.1.1. Base URL
`/api/v1/dashboard/company`

### 1.1.2. Authentication Model
- **JWT Requirements**: All endpoints require a valid JWT Bearer token enforced via `CustomizePermission`.
- **Role Requirements**: Actions are restricted based on company association via the `UserOrganizationLink` table. Nominations typically require Company Admin access.

---

## 1.2. Mentor Management

### 1.2.1. CompanyMentorNominateAPI
#### Route
```http
POST /api/v1/dashboard/company/mentor/nominate/
```
#### Description
Allows a Company Administrator to nominate an employee as a `COMPANY_MENTOR`. Creates a pending `UserMentor` application that admins can verify.
#### Authentication
`Required: JWT` | `Role: COMPANY`

### 1.2.2. CompanyMentorListAPI
#### Route
```http
GET /api/v1/dashboard/company/mentor/list/
```
#### Description
Lists all mentors currently associated with the authenticated user's company.
#### Authentication
`Required: JWT` | `Role: COMPANY`

---

## 1.3. Sessions & Tasks

Company Mentors utilize the generalized mentor dashboard APIs for session and task management, operating under the `COMPANY_SESSION` type.

### 1.3.1. Session Scheduling Reference
- Company mentors use `POST /api/v1/dashboard/mentor/session/create/`.
- The backend automatically resolves the company org ID and assigns `session_type = COMPANY_SESSION`.

### 1.3.2. CompanyTaskListCreateAPI
#### Route
```http
GET /api/v1/dashboard/company/tasks/
POST /api/v1/dashboard/company/tasks/
```
#### Description
Lists tasks submitted by the company or creates a new company-specific task for administrative approval.
#### Authentication
`Required: JWT` | `Role: COMPANY`

### 1.3.3. CompanyTaskDetailAPI
#### Route
```http
GET /api/v1/dashboard/company/tasks/<str:task_id>/
PUT /api/v1/dashboard/company/tasks/<str:task_id>/
DELETE /api/v1/dashboard/company/tasks/<str:task_id>/
```
#### Description
Retrieves, updates, or deletes a specific pending task created by the company.
#### Authentication
`Required: JWT` | `Role: COMPANY`
