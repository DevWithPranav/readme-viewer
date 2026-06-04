# 1. Campus Mentor API Documentation

This document serves as the backend API reference for the Campus Mentor functionalities, built into the Campus Dashboard module.

## 1.1. Overview

### 1.1.1. Base URL
`/api/v1/dashboard/campus`

### 1.1.2. Authentication Model
- **JWT Requirements**: All endpoints require a valid JWT Bearer token enforced via `CustomizePermission`.
- **Role Requirements**: Session creation requires the user to be an approved `CAMPUS_MENTOR` in the `UserMentor` table. Administrative assignment requires Campus Staff roles.

---

## 1.2. Mentorship

### 1.2.1. CampusMentorSessionCreateAPI
#### Route
```http
POST /api/v1/dashboard/campus/sessions/create/
```
#### Description
Schedules a new mentorship session within the campus. Resolves the `entity_id` to the assigned campus organization dynamically.
#### Authentication
`Required: JWT` | `Role: CAMPUS_MENTOR`
#### Request Body
| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `org_id` | string | Optional | Required only if mentor is assigned to multiple campuses |
| `title` | string | Yes | Session title |
| `description` | string | Yes | Description |
| `mode` | string | Yes | ONLINE/OFFLINE |
| `starts_at` | datetime | Yes | Start time |
| `ends_at` | datetime | Yes | End time |
| `meeting_link` | string | Optional | Meeting URL |
| `venue` | string | Optional | Physical venue |
| `max_participants` | int | Optional | Participant limit |

### 1.2.2. CampusSessionListAPI
#### Route
```http
GET /api/v1/dashboard/campus/sessions/list/
```
#### Description
Retrieves historical and upcoming mentorship sessions for the campus. 
**Visibility Rules**:
- Campus Mentors, Leads, and Enablers can see all sessions regardless of status.
- Regular students can only see `SCHEDULED` sessions.
- Users who are students at one campus and mentors at another receive mixed visibility depending on the session's organization.
#### Authentication
`Required: JWT`
#### Query Parameters
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `status` | string | No | Filter by session status (e.g., SCHEDULED, PENDING_APPROVAL) |

### 1.2.3. AssignCampusMentorAPI
#### Route
```http
POST /api/v1/dashboard/campus/assign-mentor/
```
#### Description
Links a user as a mentor to a specific campus initiative.
#### Authentication
`Required: JWT` | `Role: Campus Lead, Lead Enabler, Admin`
