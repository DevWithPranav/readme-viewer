# Task List — Public API

**Endpoint** `GET /api/v1/dashboard/task/list/`  
**Authentication** Required (JWT Bearer token)  
**Permissions** No role restriction — open to any authenticated user, subject to visibility rules below

---

## Overview

Returns a paginated list of active tasks visible to the caller. What a caller can see depends entirely on their campus, IG, and event memberships. Tasks are always filtered to `active = true`.

---

## Request

### Headers

| Header | Required | Description |
| :--- | :--- | :--- |
| `Authorization` | Required | `Bearer <JWT>` — JWT token of the authenticated user |

### Query Parameters

#### Pagination

| Parameter | Type | Default | Description |
| :--- | :--- | :--- | :--- |
| `pageIndex` | integer | `1` | Page number (1-indexed) |
| `perPage` | integer | `10` | Number of results per page |

#### Search

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `search` | string | Case-insensitive substring match across the fields listed in the [Searchable Fields](#searchable-fields) table |

#### Sort

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `sortBy` | string | Sort key from the [Sortable Fields](#sortable-fields) table. Prefix with `-` for descending order (e.g. `-karma`) |

#### Filters

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `ig_id` | UUID | Return only tasks belonging to a specific Interest Group |
| `event_id` | UUID | Return only tasks linked to a specific event. Returns an empty result set if the caller does not have access to that event |
| `is_event_task` | `true` | Return only tasks that are linked to any event the caller can access |
| `event_tasks_only` | `true` | Alias for `is_event_task=true` |
| `task_source` | enum | Filter tasks by who created them. See [Task Source Filter](#task-source-filter) for allowed values |

> **Note:** `event_id`, `is_event_task`, and `event_tasks_only` are mutually exclusive. If `event_id` is provided the other two are ignored.

---

## Searchable Fields

The `search` query parameter performs a case-insensitive `icontains` match against **any** of the following fields. A task is returned if any field matches.

| Field Key | What is searched |
| :--- | :--- |
| `hashtag` | Task hashtag |
| `title` | Task title |
| `description` | Task description |
| `karma` | Karma value (numeric cast to string) |
| `channel__name` | Associated channel name |
| `type__title` | Task type title |
| `active` | Active flag |
| `variable_karma` | Variable karma flag |
| `usage_count` | Submission usage count |
| `level__name` | Associated level name |
| `org__title` | Associated organisation name |
| `ig__name` | Associated Interest Group name |
| `event` | Legacy event label |
| `event_fk__title` | Title of the linked Event record |

---

## Sortable Fields

Pass the key in `sortBy`. Prefix with `-` for descending order.

| `sortBy` key | Sorts by |
| :--- | :--- |
| `hashtag` | Task hashtag |
| `title` | Task title |
| `description` | Description |
| `karma` | Karma points |
| `channels` | Channel name |
| `type` | Task type title |
| `active` | Active status |
| `variable_karma` | Variable karma flag |
| `usage_count` | Number of times submitted |
| `level` | Level name |
| `org` | Organisation name |
| `ig` | Interest Group name |
| `event` | Legacy event label |
| `event_title` | Linked Event title |
| `updated_at` | Last updated timestamp |
| `created_at` | Creation timestamp |

**Example:** Sort by karma descending → `?sortBy=-karma`

---

## Visibility Rules

Visibility is determined by the task's `org` and `ig` fields, and the caller's memberships.

### Task Scope Matrix

| Task Configuration | Who all is visible |
| :--- | :--- |
| **Global** — no `org`, no `ig` | All authenticated users |
| **Company** — `org` is a Company | All authenticated users |
| **Campus** — `org` is a College or School | Members of that campus/college/school only |
| **IG** — `ig` is set | Members of that Interest Group (IG) only |
| **Event task** — `event_fk` is set | Authenticated users with access to that event (following event scope rules) |

### Event Task Visibility

Event-linked tasks follow the event's own scope rules. An event task is visible if the event is:
- Published or Ongoing, and
- The event's scope permits the caller (Global events are open to all authenticated users; org/campus-scoped events follow the same membership rules as regular tasks)

---

## Task Source Filter

The `task_source` query parameter lets you narrow results to tasks created by a specific type of contributor. It applies **after** the standard visibility rules — meaning a user can only see tasks they are already eligible to see.

| `task_source` value | Tasks returned | Eligibility note |
| :--- | :--- | :--- |
| `company` | Tasks submitted by a verified company user | Company tasks are visible to all authenticated users |
| `ig_mentor` | Tasks submitted by an approved IG mentor | Task has `ig` set → only visible to members of that IG |
| `campus_mentor` | Tasks submitted by an approved campus mentor | Task has `ig` set → only visible to members of that IG |


> **Important:** `task_source` does **not** bypass visibility rules. For example, `?task_source=ig_mentor` will only return IG mentor tasks that belong to an IG the caller is a member of. A user not in that IG will receive an empty result set — not an error.

---

## Response

### HTTP 200 — Success

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Done",
  "response": {
    "data": [
      {
        "id": "<id>",
        "hashtag": "#skip-lvl3",
        "title": "SKpi",
        "description": null,
        "karma": 1600,
        "channel": "task-dropbox",
        "discord_id": "1409243188195102850",
        "type": "Collaboration",
        "variable_karma": false,
        "level": "lvl3",
        "ig": null,
        "event": null,
        "event_id": null
      }
    ],
    "pagination": {
      "count": 84,
      "totalPages": 9,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

### Response Fields — `data[]`

| Field | Type | Nullable | Description |
| :--- | :--- | :--- | :--- |
| `id` | UUID string | No | Unique task identifier (e.g. `"13e91e1d-4d5b-46a7-b39f-faf747d398a9"`) |
| `hashtag` | string | No | Unique hashtag for the task (e.g. `#skip-lvl3`) |
| `title` | string | No | Human-readable task name (e.g. `"SKpi"`) |
| `description` | string | Yes | Detailed task description |
| `karma` | integer | No | Base karma awarded on approval (e.g. `1600`) |
| `channel` | string | Yes | Submission channel name (e.g. `"task-dropbox"`) |
| `discord_id` | string | Yes | Discord channel ID associated with the channel |
| `type` | string | No | Task type label (e.g. `"Collaboration"`) |
| `variable_karma` | boolean | No | If `true`, actual karma awarded may vary from base karma |
| `level` | string | Yes | Associated level name (e.g. `"lvl3"`) |
| `ig` | string | Yes | Interest Group name the task belongs to. `null` for global/company tasks |
| `event` | string | Yes | Legacy event label stored as a plain string. Use `event_id` for structured event lookups |
| `event_id` | UUID string | Yes | ID of the linked Event record. `null` if the task is not event-linked |

### Response Fields — `pagination`

| Field | Type | Description |
| :--- | :--- | :--- |
| `count` | integer | Total number of matching tasks across all pages |
| `totalPages` | integer | Total number of pages given current `perPage` |
| `isNext` | boolean | Whether a next page exists |
| `isPrev` | boolean | Whether a previous page exists |
| `nextPage` | integer \| null | Next page number, or `null` if on the last page |

---

## Error Responses

This endpoint enforces authentication and returns an empty result set (not an error) when filters match nothing or when an inaccessible `event_id` is supplied.

| HTTP Status | Condition |
| :--- | :--- |
| `401` | Missing or invalid authentication token (JWT) |
| `200` with empty `data` | No tasks match the given filters/scope |
| `500` | Internal server error |

---

## Examples

### 1. Default task listing

```
GET /api/v1/dashboard/task/list/
```

Returns global and company tasks (paginated, 10 per page) visible to the authenticated user.

---

### 2. Paginated — page 3, 25 results per page

```
GET /api/v1/dashboard/task/list/?pageIndex=3&perPage=25
```

---

### 3. Search by keyword

```
GET /api/v1/dashboard/task/list/?search=python
```

Matches tasks where `title`, `hashtag`, `description`, `type`, `ig`, `channel`, or any other searchable field contains "python".

---

### 4. Filter by Interest Group

```
GET /api/v1/dashboard/task/list/?ig_id=<ig-uuid>
```

Only returns tasks belonging to that IG. The caller must be a member of the IG to see these tasks.

---

### 5. All tasks from a specific event

```
GET /api/v1/dashboard/task/list/?event_id=<event-uuid>
```

Returns tasks linked to that specific event. Returns empty if the caller does not have access to the event.

---

### 6. All event-linked tasks the caller can access

```
GET /api/v1/dashboard/task/list/?is_event_task=true
```

---

### 7. Filter + sort + search combined

```
GET /api/v1/dashboard/task/list/?ig_id=<ig-uuid>&search=react&sortBy=-karma&pageIndex=1&perPage=20
```

Returns IG-specific tasks matching "react", sorted by karma descending, 20 per page.

---

### 8. Tasks submitted by companies

```
GET /api/v1/dashboard/task/list/?task_source=company
```

Returns only tasks created by verified company users. Visible to all authenticated users.

---

### 9. Tasks submitted by IG mentors

```
GET /api/v1/dashboard/task/list/?task_source=ig_mentor
```

Returns only tasks created by approved IG mentors. The caller will only see tasks belonging to IGs they are a member of — tasks from IGs they don't belong to are silently excluded.

---

### 10. Tasks submitted by campus mentors

```
GET /api/v1/dashboard/task/list/?task_source=campus_mentor
```

Returns only tasks created by approved campus mentors. IG membership rules still apply.

---

### 11. Platform-created tasks only

```
GET /api/v1/dashboard/task/list/?task_source=platform
```

Returns only tasks created directly by platform admins (no `requested_by` set).

---

### 12. IG mentor tasks for a specific IG

```
GET /api/v1/dashboard/task/list/?task_source=ig_mentor&ig_id=<ig-uuid>
```

Combines both filters — returns IG mentor tasks scoped to a specific IG. Caller must be a member of that IG.

---

- The `discord_id` field is sourced from the task's linked **channel**, not the task itself.
- `org` is not exposed in the response payload — it is used internally to determine visibility only.
