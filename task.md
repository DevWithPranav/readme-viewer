# Task API Reference — All Variants & Usage

> A comprehensive reference of all **task-related APIs** across the µLearn backend, showing how task management is implemented in different contexts: **Admin/Management**, **Mentors**, **Companies**, **Events**, and **Interns**.

---

## Table of Contents

- [1. Admin / Management Tasks](#1-admin--management-tasks)
  - [1.1 Task CRUD](#11-task-crud)
  - [1.2 Task Type CRUD](#12-task-type-crud)
  - [1.3 Task Dropdowns (Form Helpers)](#13-task-dropdowns-form-helpers)
  - [1.4 Task Import / Export / Templates](#14-task-import--export--templates)
  - [1.5 Admin Task Approval Workflow](#15-admin-task-approval-workflow)
- [2. Mentor Tasks](#2-mentor-tasks)
  - [2.1 Mentor Task CRUD](#21-mentor-task-crud)
  - [2.2 Mentor Task Dropdowns](#22-mentor-task-dropdowns)
- [3. Company Tasks](#3-company-tasks)
  - [3.1 Company Task CRUD](#31-company-task-crud)
- [4. Event Tasks](#4-event-tasks)
  - [4.1 Event Task CRUD](#41-event-task-crud)
  - [4.2 Event Task Metadata](#42-event-task-metadata)
- [5. Intern Tasks](#5-intern-tasks)
  - [5.1 Intern Task (Self-View)](#51-intern-task-self-view)
  - [5.2 Manage Intern Tasks (Admin)](#52-manage-intern-tasks-admin)
- [6. Cross-Cutting Patterns](#6-cross-cutting-patterns)

---

## 1. Admin / Management Tasks

**Base URL:** `api/v1/dashboard/task/`
**View File:** `api/dashboard/task/dash_task_view.py`
**Roles Required:** `Admin`, `Fellow`, `Associate` (unless noted)

### 1.1 Task CRUD

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `task/` | `TaskListAPI` | List all active tasks (paginated, searchable, sortable) | Admin / Fellow / Associate |
| `POST` | `task/` | `TaskListAPI` | Create a new task (optionally attach `skill_ids`) | Admin / Fellow / Associate |
| `GET` | `task/list/` | `TaskPublicListAPI` | Public task list (active tasks only, supports `ig_id` filter) | Any authenticated |
| `GET` | `task/<task_id>/` | `TaskAPI` | Retrieve a single task by ID | Admin / Fellow / Associate |
| `PUT` | `task/<task_id>/` | `TaskAPI` | Update a task (partial update, optionally update `skill_ids`) | Admin / Fellow / Associate |
| `DELETE` | `task/<task_id>/` | `TaskAPI` | Hard-delete a task | Admin / Fellow / Associate |

### 1.2 Task Type CRUD

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `task/list-task-type/` | `TaskTypeCrudAPI` | List all task types (paginated) | Admin / Company / Mentor |
| `POST` | `task/list-task-type/` | `TaskTypeCrudAPI` | Create a new task type | Admin |
| `PUT` | `task/task-type/<task_type_id>/` | `TaskTypeCrudAPI` | Update a task type | Admin |
| `DELETE` | `task/task-type/<task_type_id>/` | `TaskTypeCrudAPI` | Delete a task type | Admin |

### 1.3 Task Dropdowns (Form Helpers)

These endpoints provide dropdown data for the task creation/edit forms.

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `task/channel/` | `ChannelDropdownAPI` | List all channels | Admin / Fellow / Associate / Company / Mentor |
| `GET` | `task/ig/` | `IGDropdownAPI` | List all interest groups | Admin / Fellow / Associate / Company / Mentor |
| `GET` | `task/organization/` | `OrganizationDropdownAPI` | List all organizations | Admin / Fellow / Associate / Company / Mentor |
| `GET` | `task/level/` | `LevelDropdownAPI` | List all levels | Admin / Fellow / Associate / Company / Mentor |
| `GET` | `task/task-types/` | `TaskTypesDropDownAPI` | List all task types (dropdown) | Admin / Fellow / Associate / Company / Mentor |
| `GET` | `task/events/` | `EventDropDownApi` | List all events (enum) | Admin |

### 1.4 Task Import / Export / Templates

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `task/csv/` | `TaskListCSV` | Download all active tasks as CSV | Admin / Fellow / Associate |
| `POST` | `task/import/` | `ImportTaskListCSV` | Bulk import tasks from Excel (`.xlsx`) | Admin / Fellow / Associate |
| `GET` | `task/base-template/` | `TaskBaseTemplateAPI` | Download the task import Excel template | Any authenticated |

### 1.5 Admin Task Approval Workflow

Used by admins to review tasks submitted by **Companies** and **Mentors**.

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `task/pending/` | `AdminTaskApprovalAPI` | List tasks awaiting admin review (filterable by `approval_status`, `source`=company/mentor, `company_name`, `mentor_name`) | Admin |
| `PATCH` | `task/<task_id>/review/` | `AdminTaskApprovalAPI` | Approve or reject a pending task. Body: `{"action": "approve"\|"reject", "reason": "..."}` | Admin |

---

## 2. Mentor Tasks

**Base URL:** `api/v1/dashboard/mentor/tasks/`
**View File:** `api/dashboard/mentor/task_views.py`
**Roles Required:** `Mentor`

### 2.1 Mentor Task CRUD

Mentors submit tasks for their Interest Groups. Tasks start as `pending` and require **admin approval** before going live.

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `mentor/tasks/` | `MentorTaskListCreateAPI` | List all tasks submitted by the authenticated mentor (filterable by `approval_status`) | Mentor |
| `POST` | `mentor/tasks/` | `MentorTaskListCreateAPI` | Create a new task for an IG the mentor belongs to. Starts as `pending`, `active=False`. Accepts optional `skill_ids`. | Mentor |
| `GET` | `mentor/tasks/<task_id>/` | `MentorTaskDetailAPI` | Retrieve detail of a specific task submitted by the mentor | Mentor |
| `PUT` | `mentor/tasks/<task_id>/` | `MentorTaskDetailAPI` | Update a task — **reverts** to `pending` / `active=False` (re-triggers admin approval). Accepts optional `skill_ids`. | Mentor |
| `DELETE` | `mentor/tasks/<task_id>/` | `MentorTaskDetailAPI` | Delete a task — **only allowed when `approval_status == pending`** | Mentor |

### 2.2 Mentor Task Dropdowns

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `mentor/tasks/ig-dropdown/` | `MentorIGDropdownAPI` | List only the IGs the authenticated mentor is assigned to (scoped dropdown) | Mentor |

> **Key Difference from Admin:** Mentors can only create tasks for IGs they're assigned to. The IG dropdown is scoped to their active assignments.

---

## 3. Company Tasks

**Base URL:** `api/v1/dashboard/company/tasks/`
**View File:** `api/dashboard/company/task_views.py`
**Roles Required:** Verified company creator or approved Company Mentor

### 3.1 Company Task CRUD

Companies submit tasks that require **admin approval** before going live — same workflow as mentor tasks.

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `company/tasks/` | `CompanyTaskListCreateAPI` | List all tasks submitted by the authenticated company user (filterable by `approval_status`) | Company (verified) |
| `POST` | `company/tasks/` | `CompanyTaskListCreateAPI` | Create a new task. Starts as `pending`, `active=False`. Accepts optional `skill_ids`. | Company (verified) |
| `GET` | `company/tasks/<task_id>/` | `CompanyTaskDetailAPI` | Retrieve detail of a specific task | Company (verified) |
| `PUT` | `company/tasks/<task_id>/` | `CompanyTaskDetailAPI` | Full update — **reverts** to `pending` / `active=False` (re-triggers admin approval). Accepts optional `skill_ids`. | Company (verified) |
| `PATCH` | `company/tasks/<task_id>/` | `CompanyTaskDetailAPI` | Partial update — also **reverts** to `pending` / `active=False` | Company (verified) |
| `DELETE` | `company/tasks/<task_id>/` | `CompanyTaskDetailAPI` | **Soft-delete** (marks `active=False`) | Company (verified) |

> **Key Differences from Admin/Mentor:**
> - Company tasks support both `PUT` (full update) and `PATCH` (partial update), unlike mentor tasks which only have `PUT`.
> - Delete is a **soft-delete** (`active=False`), unlike admin (hard-delete) and mentor (hard-delete, pending only).

---

## 4. Event Tasks

**Base URL:** `api/v1/dashboard/events/manage/<event_id>/tasks/`
**View File:** `api/dashboard/events/task_views.py`
**Roles Required:** Event organiser, co-owner, or Admin

### 4.1 Event Task CRUD

Tasks can be linked to events. These tasks also require **admin approval**.

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `events/manage/<event_id>/tasks/` | `EventTaskListCreateAPI` | List all tasks linked to an event (paginated) | Event organiser / co-owner / Admin |
| `POST` | `events/manage/<event_id>/tasks/` | `EventTaskListCreateAPI` | Create a new task linked to the event. Starts as `pending`, `active=False`. | Event organiser / co-owner / Admin |
| `GET` | `events/manage/<event_id>/tasks/<task_id>/` | `EventTaskDetailAPI` | Retrieve a specific event task | Event organiser / co-owner / Admin |
| `PATCH` | `events/manage/<event_id>/tasks/<task_id>/` | `EventTaskDetailAPI` | Partial update of an event task | Event organiser / co-owner / Admin |
| `DELETE` | `events/manage/<event_id>/tasks/<task_id>/` | `EventTaskDetailAPI` | **Hard-delete** the event task | Event organiser / co-owner / Admin |

### 4.2 Event Task Metadata

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `events/manage/<event_id>/tasks/meta/` | `EventTaskMetaAPI` | Dropdown data for task forms (task types, IGs, levels, channels, organizations) | Event organiser / co-owner / Admin |

> **Key Differences:**
> - Event tasks are **scoped to an event** (`event_fk` foreign key).
> - No `PUT` support — only `PATCH` for partial updates.
> - No approval status reverts on update (unlike mentor/company tasks).

---

## 5. Intern Tasks

A completely separate task model (`InternTask`) from the main `TaskList`. Used for internship task management.

### 5.1 Intern Task (Self-View)

**Base URL:** `api/v1/dashboard/intern/tasks/`
**View File:** `api/dashboard/intern/tasks/tasks_views.py`
**Roles Required:** `Intern`

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `intern/tasks/mine/` | `InternTaskMineAPI` | List all tasks assigned to the current intern (paginated) | Intern |
| `PATCH` | `intern/tasks/<task_id>/` | `InternTaskMineAPI` | Update task status (e.g., `in_progress` → `completed`). Logs to `SystemActionLog`. | Intern |

### 5.2 Manage Intern Tasks (Admin)

**Base URL:** `api/v1/dashboard/manage-interns/tasks/`
**View File:** `api/dashboard/manage_interns/tasks/tasks_views.py`
**Roles Required:** `Admin`

| Method | Endpoint | View Class | Description | Role |
|--------|----------|------------|-------------|------|
| `GET` | `manage-interns/tasks/` | `ManageInternTaskAPI` | List all intern tasks (paginated, searchable by title, status, category, assignee) | Admin |
| `GET` | `manage-interns/tasks/<task_id>/` | `ManageInternTaskAPI` | Retrieve a specific intern task by ID | Admin |
| `POST` | `manage-interns/tasks/` | `ManageInternTaskAPI` | Create an intern task and assign it to an intern | Admin |
| `PATCH` | `manage-interns/tasks/<task_id>/` | `ManageInternTaskAPI` | Update an intern task (partial update). Logs changes to `SystemActionLog`. | Admin |
| `DELETE` | `manage-interns/tasks/<task_id>/` | `ManageInternTaskAPI` | Hard-delete an intern task | Admin |

---

## 6. Cross-Cutting Patterns

### Approval Workflow Comparison

| Feature | Admin Task | Mentor Task | Company Task | Event Task | Intern Task |
|---------|-----------|-------------|--------------|------------|-------------|
| **Model** | `TaskList` | `TaskList` | `TaskList` | `TaskList` | `InternTask` |
| **Requires Approval** | No (directly active) | ✅ Yes | ✅ Yes | ✅ Yes | No (admin-created) |
| **approval_status field** | N/A | `pending` → `approved` / `rejected` | `pending` → `approved` / `rejected` | `pending` → `approved` / `rejected` | N/A |
| **Reverts on Edit** | No | ✅ Yes (back to `pending`) | ✅ Yes (back to `pending`) | No | N/A |
| **Supports `skill_ids`** | ✅ Yes | ✅ Yes | ✅ Yes | No | No |
| **Delete Type** | Hard-delete | Hard-delete (pending only) | Soft-delete (`active=False`) | Hard-delete | Hard-delete |
| **Scoped To** | Global | Mentor's IGs | Company | Event | Intern assignment |

### HTTP Methods Used Per Module

| Module | `GET` | `POST` | `PUT` | `PATCH` | `DELETE` |
|--------|-------|--------|-------|---------|----------|
| **Admin Task CRUD** | ✅ List + Detail | ✅ Create | ✅ Update | — | ✅ Delete |
| **Admin Task Approval** | ✅ List pending | — | — | ✅ Approve/Reject | — |
| **Admin Task Types** | ✅ List | ✅ Create | ✅ Update | — | ✅ Delete |
| **Mentor Tasks** | ✅ List + Detail | ✅ Create | ✅ Update | — | ✅ Delete |
| **Company Tasks** | ✅ List + Detail | ✅ Create | ✅ Full Update | ✅ Partial Update | ✅ Soft-delete |
| **Event Tasks** | ✅ List + Detail | ✅ Create | — | ✅ Partial Update | ✅ Delete |
| **Intern Tasks (self)** | ✅ List (mine) | — | — | ✅ Status update | — |
| **Manage Intern Tasks** | ✅ List + Detail | ✅ Create | — | ✅ Update | ✅ Delete |

### Shared Dropdown Endpoints

Task form dropdowns are reused across modules:

| Dropdown | Admin endpoint | Mentor endpoint | Company endpoint | Event endpoint |
|----------|---------------|-----------------|------------------|----------------|
| **Channels** | `task/channel/` | uses admin's | uses admin's | `events/manage/<id>/tasks/meta/` |
| **Interest Groups** | `task/ig/` | `mentor/tasks/ig-dropdown/` *(scoped)* | uses admin's | `events/manage/<id>/tasks/meta/` |
| **Organizations** | `task/organization/` | uses admin's | uses admin's | `events/manage/<id>/tasks/meta/` |
| **Levels** | `task/level/` | uses admin's | uses admin's | `events/manage/<id>/tasks/meta/` |
| **Task Types** | `task/task-types/` | uses admin's | uses admin's | `events/manage/<id>/tasks/meta/` |
| **Events** | `task/events/` | — | — | — |

### Task Report APIs (Supplementary)

**Base URL:** `api/v1/dashboard/task-report/`

| Method | Endpoint | View Class | Description |
|--------|----------|------------|-------------|
| `GET` | `task-report/` | `TaskReportInfoView` | List task reports |
| `GET` | `task-report/<report_id>/` | `TaskReportInfoView` | Get specific report |
| `GET` | `task-report/group-by-reporter/` | `TaskReportReporterGroupingView` | Group reports by reporter |

---

> **Last updated:** 2026-06-13
