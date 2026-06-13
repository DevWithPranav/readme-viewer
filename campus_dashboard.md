# Campus Dashboard API Documentation

This document serves as the backend API reference for the Campus Dashboard module, derived entirely from actual source code implementations.

## Overview

### Base URL
`/api/v1/dashboard/campus`

### Authentication Model
- **Anonymous Endpoints**: `CollegeApi` and `CampusDetailsPublicAPI` are unauthenticated and publicly accessible.
- **JWT Requirements**: All non-public endpoints require a valid JWT Bearer token enforced via `CustomizePermission`.
- **Campus Role Requirements**: Most management APIs (e.g., Showcase, Execom, Transfers) require the `@campus_staff_required` or explicit `@role_required` decorators (`Campus Lead` or `Lead Enabler`).
- **Campus Scope Enforcement**: Standardized isolated endpoints use `get_campus_context(request)` to fetch tenant data based on the caller's JWT link. Legacy endpoints accept an `<str:org_id>` URL parameter.

---

## Dashboard Summary

### CampusDashboardSummaryAPIView
#### Route
```http
GET /api/v1/dashboard/campus/home-summary/
```
#### Description
Retrieves top-level aggregate summary metrics for the caller's campus.
#### Authentication
`Required: JWT`
#### Path / Query Parameters
None
#### Success Response
Returns integers for `total_karma`, `total_members`, `active_members`, `rank`, `active_ig_count`, and `karma_last_30_days`.

---

## Campus Information

### CampusDetailsAPI
#### Route
```http
GET /api/v1/dashboard/campus/campus-details/
```
#### Description
Retrieves comprehensive details about the authenticated user's campus.
#### Authentication
`Required: JWT` | `Role: Campus Staff`
#### Success Response
Returns `CampusDetailsSerializer` payload containing zone, college name, code, rank, leads, and social links.

### CampusDetailsPublicAPI
#### Route
```http
GET /api/v1/dashboard/campus/<str:org_id>/
```
#### Description
Publicly accessible college showcase details.
#### Authentication
`No Authentication Required`
#### Path Parameters
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `org_id` | UUID | Yes | Target Organization ID |

---

## Showcase Management

### CampusShowcaseAPI
#### Route
```http
GET /api/v1/dashboard/campus/showcase/
PATCH /api/v1/dashboard/campus/showcase/
```
#### Description
Fetches or updates the public showcase page configuration for the user's campus.
#### Authentication
**GET**: `Required: JWT` | `Role: Campus Staff`
**PATCH**: `Required: JWT` | `Role: Campus Lead, Lead Enabler`
#### Request Body (PATCH)
| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `about` | string | No | College description |
| `hero_image` | string | No | Image URL |
| `highlights` | list | No | List of highlight strings |
| `gallery` | list | No | List of image URLs |
| `testimonials` | list | No | List of JSON objects |

---

## Student Analytics

### CampusStudentDetailsAPI
#### Route
```http
GET /api/v1/dashboard/campus/student-details/
```
#### Description
Retrieves a paginated roster of students in the campus.
#### Authentication
`Required: JWT` | `Role: Campus Staff`
#### Query Parameters
| Name | Type | Required | Description |
| ---- | ---- | -------- | ----------- |
| `pageIndex` | int | No | Pagination offset |
| `perPage` | int | No | Items per page |
| `search` | str | No | MUID or name search |
| `is_alumni` | str | No | "true" or "false" |

### CampusStudentDetailsCSVAPI
#### Route
```http
GET /api/v1/dashboard/campus/student-details/csv/
```
#### Description
Exports the student roster as a CSV file.

### CampusStudentActivityAPI
#### Route
```http
GET /api/v1/dashboard/campus/students/<str:muid>/activity/
```
#### Description
Fetches individual karma activity logs for a specific student.

---

## Leaderboard & Karma

### CampusStudentLeaderboardAPI
#### Route
```http
GET /api/v1/dashboard/campus/<str:org_id>/leaderboard/
```
#### Description
Retrieves paginated student leaderboard for a specific campus.
#### Authentication
`Required: JWT`

### CampusKarmaByClusterAPI
#### Route
```http
GET /api/v1/dashboard/campus/<str:org_id>/karma-by-cluster/
```
#### Description
Aggregates total karma distributed across IG categories.

### CampusStudentInEachLevelAPI
#### Route
```http
GET /api/v1/dashboard/campus/student-level/
GET /api/v1/dashboard/campus/student-level/<str:org_id>/
```
#### Description
Fetches student distribution across gamification levels.

### WeeklyKarmaAPI
#### Route
```http
GET /api/v1/dashboard/campus/weekly-karma/
GET /api/v1/dashboard/campus/weekly-karma/<str:org_id>/
```
#### Description
Retrieves short-term campus karma accumulation trends.

---

## Events

### CampusEventsAPI
#### Route
```http
GET /api/v1/dashboard/campus/events/
```
#### Description
Fetches all learning initiatives and events tied to the campus.

### CampusEventDistributionAPI
#### Route
```http
GET /api/v1/dashboard/campus/events/distribution/
```
#### Description
Aggregates campus participation metrics across event categories.

---

## Execom Management

### CampusExecomAPI
#### Route
```http
GET /api/v1/dashboard/campus/execom/
POST /api/v1/dashboard/campus/execom/
DELETE /api/v1/dashboard/campus/execom/<str:member_id>/
```
#### Description
Lists, assigns, or revokes Executive Committee roles within the campus.

### CampusExecomRoleAPI
#### Route
```http
GET /api/v1/dashboard/campus/execom/roles/
POST /api/v1/dashboard/campus/execom/roles/
```
#### Description
Manages definitions and availability of Execom roles.

---

## IG Management

### CampusIGsAPI
#### Route
```http
GET /api/v1/dashboard/campus/igs/
```
#### Description
Lists all Interest Groups active in the campus.

### CampusIGMembersAPI
#### Route
```http
GET /api/v1/dashboard/campus/igs/<str:ig_id>/members/
```
#### Description
Fetches the student roster belonging to a specific IG.

### CampusIGChapterAPI
#### Route
```http
GET /api/v1/dashboard/campus/ig-chapters/
POST /api/v1/dashboard/campus/ig-chapters/
PATCH /api/v1/dashboard/campus/ig-chapters/<str:chapter_id>/
DELETE /api/v1/dashboard/campus/ig-chapters/<str:chapter_id>/
```
#### Description
Full CRUD operations for managing Interest Group Chapters within a campus.

---

## Learning Circles

### CampusLearningCirclesAPI
#### Route
```http
GET /api/v1/dashboard/campus/learning-circles/
```
#### Description
Retrieves all active Learning Circles running within the campus.

### CampusLCMembersAPI
#### Route
```http
GET /api/v1/dashboard/campus/learning-circles/<str:circle_id>/members/
```
#### Description
Retrieves the member roster for a specific Learning Circle.

---

## Mentorship

### CampusMentorSessionCreateAPI
#### Route
```http
POST /api/v1/dashboard/campus/sessions/create/
```
#### Description
Schedules a new mentorship session within the campus.

### CampusSessionListAPI
#### Route
```http
GET /api/v1/dashboard/campus/sessions/list/
```
#### Description
Retrieves historical and upcoming mentorship sessions.

### AssignCampusMentorAPI
#### Route
```http
POST /api/v1/dashboard/campus/assign-mentor/
```
#### Description
Links a user as a mentor to a specific campus initiative.

---

## Social Links

### CampusSocialLinkAPI
#### Route
```http
PUT /api/v1/dashboard/campus/social-links/
DELETE /api/v1/dashboard/campus/social-links/<str:link_id>/
```
#### Description
Creates, updates, or deletes social media platforms mapped to the college showcase.

---

## Role Transfers

### ChangeStudentTypeAPI
#### Route
```http
PATCH /api/v1/dashboard/campus/change-student-type/<str:member_id>/
```
#### Description
Updates the graduation or enrollment status (alumni tracking) for a specific user.

### TransferLeadRoleAPI
#### Route
```http
POST /api/v1/dashboard/campus/transfer-lead-role/
```
#### Description
Securely transitions the Campus Lead role to a new user.

### TransferEnablerRoleAPI
#### Route
```http
POST /api/v1/dashboard/campus/transfer-enabler-role/
```
#### Description
Transitions the Lead Enabler role to a new user.

### TransferIGRoleAPI
#### Route
```http
POST /api/v1/dashboard/campus/transfer-ig-role/
```
#### Description
Transitions an Interest Group Lead role to a new user within the campus hierarchy.
