# 1. Mentor Dashboard API Documentation

This document serves as the backend API reference for the Mentor Dashboard module, derived entirely from actual source code implementations.

## 1.1. Overview

### 1.1.1. Base URL
`/api/v1/dashboard/mentor`

### 1.1.2. Authentication Model
- **Public Endpoints**: `/public/profile/<str:mentor_id>/` and `/public/availability/<str:mentor_id>/` are publicly accessible.
- **JWT Requirements**: All other endpoints require a valid JWT Bearer token enforced via `CustomizePermission`.
- **Role Requirements**: Management and creation APIs require the `@role_required([RoleType.MENTOR.value])` decorator. Approval routes require the `Admin` role.

---

## 1.2. Mentor Profile & Registration

### 1.2.1. MentorRegistrationAPI
#### Route
```http
POST /api/v1/dashboard/mentor/register/
PATCH /api/v1/dashboard/mentor/register/
```
#### Description
Registers a user as a mentor or updates/resubmits a rejected mentor application.
#### Authentication
`Required: JWT`
#### Request Body
| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `about` | string | Yes | Mentor bio |
| `expertise` | string | Yes | Area of expertise |
| `reason` | string | Yes | Why they want to be a mentor |
| `hours` | integer | Yes | Hours available per week |
| `preferred_ig_ids` | list | Yes | Array of Interest Group UUIDs |

### 1.2.2. MentorStatusAPI
#### Route
```http
GET /api/v1/dashboard/mentor/status/
```
#### Description
Checks the current approval status of the authenticated user's mentor application.

### 1.2.3. MentorProfileAPI
#### Route
```http
GET /api/v1/dashboard/mentor/profile/
PATCH /api/v1/dashboard/mentor/profile/
```
#### Description
Fetches or updates the approved mentor's profile information.

### 1.2.4. MentorActivityListAPI
#### Route
```http
GET /api/v1/dashboard/mentor/activity/
```
#### Description
Fetches a merged, paginated timeline of the mentor's recent activity (sessions created and tasks appraised).

### 1.2.5. MentorPublicProfileAPI
#### Route
```http
GET /api/v1/dashboard/mentor/public/profile/<str:mentor_id>/
```
#### Description
Publicly accessible mentor profile details.

---

## 1.3. Session Management

### 1.3.1. MentorSessionCreateAPI
#### Route
```http
POST /api/v1/dashboard/mentor/session/create/
```
#### Description
Schedules a new mentorship session. Automatically resolves to IG or Company based on mentor type and input.
#### Request Body
| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `ig` | string | Optional | IG UUID (if IG mentor) |
| `title` | string | Yes | Session title |
| `description` | string | Yes | Description |
| `mode` | string | Yes | ONLINE/OFFLINE |
| `starts_at` | datetime | Yes | Start time |
| `ends_at` | datetime | Yes | End time |
| `meeting_link` | string | Optional | Meeting URL |
| `venue` | string | Optional | Physical venue |
| `max_participants` | int | Optional | Participant limit |

### 1.3.2. MentorSessionListAPI
#### Route
```http
GET /api/v1/dashboard/mentor/session/list/
GET /api/v1/dashboard/mentor/session/list/<str:session_id>/
```
#### Description
Lists all sessions created by the logged-in mentor, or retrieves details of a specific session.

### 1.3.3. MentorSessionUpdateAPI
#### Route
```http
PATCH /api/v1/dashboard/mentor/session/update/<str:session_id>/
DELETE /api/v1/dashboard/mentor/session/update/<str:session_id>/
```
#### Description
Updates or deletes a mentorship session.

### 1.3.4. AvailableSessionListAPI
#### Route
```http
GET /api/v1/dashboard/mentor/session/available/
```
#### Description
Lists all scheduled sessions available to the current user based on their IG and Company affiliations.

---

## 1.4. Mentor Availability

### 1.4.1. MentorAvailabilitySlotAPI
#### Route
```http
GET /api/v1/dashboard/mentor/availability/
POST /api/v1/dashboard/mentor/availability/
PATCH /api/v1/dashboard/mentor/availability/<str:slot_id>/
DELETE /api/v1/dashboard/mentor/availability/<str:slot_id>/
```
#### Description
CRUD operations for a mentor's recurring or fixed availability slots.

### 1.4.2. MentorPublicAvailabilityAPI
#### Route
```http
GET /api/v1/dashboard/mentor/public/availability/<str:mentor_id>/
```
#### Description
Publicly lists an approved mentor's available slots.

---

## 1.5. Participant Management

### 1.5.1. SessionJoinAPI
#### Route
```http
POST /api/v1/dashboard/mentor/session/participation/join/<str:session_id>/
```
#### Description
Allows a MuLearn user to join a scheduled mentorship session.

### 1.5.2. UserSessionHistoryAPI
#### Route
```http
GET /api/v1/dashboard/mentor/session/participant/history/
```
#### Description
Lists all mentorship sessions the logged-in user has joined.

### 1.5.3. MentorParticipantListAPI
#### Route
```http
GET /api/v1/dashboard/mentor/session/participant/list/<str:session_id>/
```
#### Description
Mentor view to list all participants in a specific session.

### 1.5.4. MentorParticipantUpdateAPI
#### Route
```http
PATCH /api/v1/dashboard/mentor/session/participant/update/<str:link_id>/
```
#### Description
Mentor view to update a participant's attendance status and contributed minutes.

### 1.5.5. ParticipantFeedbackAPI
#### Route
```http
PATCH /api/v1/dashboard/mentor/session/participant/feedback/<str:session_id>/
```
#### Description
Allows an attended user to submit feedback for a session.

---

## 1.6. Mentor Tasks

### 1.6.1. MentorIGDropdownAPI
#### Route
```http
GET /api/v1/dashboard/mentor/tasks/ig-dropdown/
```
#### Description
Returns Interest Groups the authenticated mentor belongs to for task creation.

### 1.6.2. MentorTaskListCreateAPI
#### Route
```http
GET /api/v1/dashboard/mentor/tasks/
POST /api/v1/dashboard/mentor/tasks/
```
#### Description
List all tasks submitted by the mentor, or submit a new task for admin approval.

### 1.6.3. MentorTaskDetailAPI
#### Route
```http
GET /api/v1/dashboard/mentor/tasks/<str:task_id>/
PUT /api/v1/dashboard/mentor/tasks/<str:task_id>/
DELETE /api/v1/dashboard/mentor/tasks/<str:task_id>/
```
#### Description
Retrieve, update, or delete a specific pending task submitted by the mentor.

---

## 1.7. Admin Endpoints

### 1.7.1. MentorListAPI, MentorVerifyAPI, MentorDetailAPI
#### Routes
```http
GET /api/v1/dashboard/mentor/list/
PATCH /api/v1/dashboard/mentor/verify/<str:mentor_id>/
GET /api/v1/dashboard/mentor/detail/<str:mentor_id>/
```
#### Description
Admin APIs to list, review, approve, and reject mentor applications.

### 1.7.2. AdminSessionListAPI, AdminSessionVerifyAPI
#### Routes
```http
GET /api/v1/dashboard/mentor/session/admin/list/
PATCH /api/v1/dashboard/mentor/session/admin/verify/<str:session_id>/
```
#### Description
Admin APIs to list and verify (approve/reject) pending mentorship sessions.
