# Task & Company Task API Documentation

This document provides complete details for all API endpoints related to **Tasks**, **Company Tasks**, **Mentor Tasks**, **Task Reports**, and **Launchpad Job Tasks** in the Mulearn backend.

---

## Base Path Configuration

The Mulearn backend prefixes all API versions. The routes documented below follow these base paths:
*   **Core / Admin / Dashboard Tasks:** `/api/v1/dashboard/task/`
*   **Company Tasks:** `/api/v1/dashboard/company/`
*   **Mentor Tasks:** `/api/v1/dashboard/mentor/`
*   **Task Reports:** `/api/v1/dashboard/task-report/`
*   **Launchpad Job Tasks:** `/api/v1/launchpad/`

Authentication is required for all endpoints via the HTTP `Authorization` header:
```http
Authorization: Bearer <access_token>
```

---

## Response Envelope Design Pattern

The application implements a custom response wrapper (`CustomResponse`). All responses follow this structured format:

### 1. Success Response (Standard)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Action completed successfully"
    ]
  },
  "response": {}
}
```

### 2. Success Response (Paginated)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [],
    "pagination": {
      "count": 12,
      "totalPages": 2,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2,
      "prevPage": null
    }
  }
}
```

### 3. Validation or Client Error Response (400 Bad Request)
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "hashtag": [
      "A task with this hashtag already exists."
    ],
    "karma": [
      "A valid integer is required."
    ]
  },
  "response": {}
}
```

---

## 1. Core / Admin Dashboard Task APIs

**Base Path:** `/api/v1/dashboard/task/`

### 1.1 List All Tasks (Admin View)
*   **Method:** `GET`
*   **Path:** `/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Retrieve a paginated list of all active tasks. Supports search, sorting, and pagination.
*   **Query Parameters:**
    *   `perPage` (int): Number of items per page.
    *   `pageIndex` (int): Page number.
    *   `search` (string): Search across `hashtag`, `title`, `description`, `karma`, `channel__name`, `type__title`, `level__name`, `org__title`, `ig__name`, `event`, `updated_by__full_name`, `created_by__full_name`.
    *   `sortBy` (string): Sort fields (e.g. `hashtag`, `title`, `-karma`, `created_at`).
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      {
        "id": "c6b8bf58-9be9-4171-ab51-e374ffc18683",
        "hashtag": "#introduction",
        "title": "Introduce Yourself",
        "description": "Post a short video or text introduction in the channel.",
        "karma": 100,
        "total_karma_gainers": 45,
        "channel": "intro-channel",
        "type": "General",
        "active": true,
        "variable_karma": false,
        "usage_count": 1,
        "level": "Level 1",
        "org": "Mulearn Foundation",
        "ig": "Public Relations",
        "event": "onboarding",
        "event_id": "evt-001",
        "updated_at": "2026-06-06T09:00:00Z",
        "updated_by": "John Doe",
        "created_by": "System Admin",
        "created_at": "2026-06-01T08:00:00Z",
        "bonus_time": "2026-06-10T23:59:59Z",
        "bonus_karma": 20,
        "skills": [
          {
            "id": "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef",
            "name": "Communication",
            "code": "COMM01"
          }
        ]
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 1.2 Create Task
*   **Method:** `POST`
*   **Path:** `/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Create a new task globally available on the platform.
*   **Request Body (JSON):**
```json
{
  "hashtag": "#build-calculator",
  "discord_link": "https://discord.com/channels/...",
  "title": "Build a Simple Calculator",
  "description": "Build a CLI calculator in Python supporting basic operations.",
  "karma": 150,
  "channel": "d3b07384-d113-48ef-a386-302a28dc7f01",
  "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
  "active": true,
  "variable_karma": false,
  "usage_count": 1,
  "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
  "org": "7e492211-1376-4dcd-a111-c918bfd045d9",
  "ig": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
  "event": "learning-challenges",
  "event_id": "evt-calc-101",
  "bonus_karma": 30,
  "bonus_time": "2026-06-20T18:30:00Z",
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef"
  ]
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task Created Successfully"
    ]
  },
  "response": {}
}
```

### 1.3 List Tasks (Public View)
*   **Method:** `GET`
*   **Path:** `/list/`
*   **Allowed Roles:** Any Authenticated User
*   **Usage:** Fetch list of active tasks with restricted fields (excluding creators, appraisers, counts). Supports query param `ig_id` to filter by Interest Group.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      {
        "id": "c6b8bf58-9be9-4171-ab51-e374ffc18683",
        "hashtag": "#introduction",
        "title": "Introduce Yourself",
        "description": "Post a short video or text introduction in the channel.",
        "karma": 100,
        "channel": "intro-channel",
        "discord_id": "987654321012345678",
        "type": "General",
        "variable_karma": false,
        "level": "Level 1",
        "ig": "Public Relations",
        "event": "onboarding",
        "event_id": "evt-001"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 1.4 Get Task Details
*   **Method:** `GET`
*   **Path:** `/<str:task_id>/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Get detail of a specific task.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "hashtag": "#build-calculator",
    "discord_link": "https://discord.com/channels/...",
    "title": "Build a Simple Calculator",
    "description": "Build a CLI calculator in Python supporting basic operations.",
    "karma": 150,
    "channel": "d3b07384-d113-48ef-a386-302a28dc7f01",
    "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
    "active": true,
    "variable_karma": false,
    "usage_count": 1,
    "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
    "org": "7e492211-1376-4dcd-a111-c918bfd045d9",
    "ig": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
    "event": "learning-challenges",
    "updated_by": "user-uuid-1111",
    "created_by": "user-uuid-0000",
    "bonus_karma": 30,
    "bonus_time": "2026-06-20T18:30:00Z",
    "event_id": "evt-calc-101"
  }
}
```

### 1.5 Update Task
*   **Method:** `PUT`
*   **Path:** `/<str:task_id>/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Edit task details. Supports partial updates via serializer config.
*   **Request Body (JSON):**
```json
{
  "title": "Build a Distributed CLI Calculator",
  "karma": 250,
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef",
    "ab1d82f3-1111-482f-87ab-8f9261a8ef3f"
  ]
}
```
*   **Response Body (200 OK):**
> [!NOTE]
> This endpoint returns the updated serializer model fields inside the `general_message` list of the message envelope instead of the standard `response` key.
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      {
        "hashtag": "#build-calculator",
        "discord_link": "https://discord.com/channels/...",
        "title": "Build a Distributed CLI Calculator",
        "description": "Build a CLI calculator in Python supporting basic operations.",
        "karma": 250,
        "channel": "d3b07384-d113-48ef-a386-302a28dc7f01",
        "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
        "active": true,
        "variable_karma": false,
        "usage_count": 1,
        "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
        "org": "7e492211-1376-4dcd-a111-c918bfd045d9",
        "ig": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
        "event": "learning-challenges",
        "updated_by": "user-uuid-current",
        "created_by": "user-uuid-0000",
        "bonus_karma": 30,
        "bonus_time": "2026-06-20T18:30:00Z",
        "event_id": "evt-calc-101"
      }
    ]
  },
  "response": {}
}
```

### 1.6 Delete Task
*   **Method:** `DELETE`
*   **Path:** `/<str:task_id>/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Hard delete a task from the database.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task deleted successfully"
    ]
  },
  "response": {}
}
```

### 1.7 Export Tasks as CSV
*   **Method:** `GET`
*   **Path:** `/csv/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Export all active tasks as a downloadable CSV.
*   **Request Body:** None
*   **Response Body (200 OK):** Returns raw CSV file output.

### 1.8 Import Tasks from CSV
*   **Method:** `POST`
*   **Path:** `/import/`
*   **Allowed Roles:** `Admin`, `Fellow`, `Associate`
*   **Usage:** Bulk import tasks via CSV/Excel upload.
*   **Request Body (multipart/form-data):**
    *   `task_list` (File): Excel/CSV sheet. Must match standard headers: `hashtag`, `title`, `description`, `karma`, `usage_count`, `variable_karma`, `level`, `channel`, `type`, `ig`, `org`, `event`.
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "Success": [
      {
        "hashtag": "#csv-task-1",
        "title": "CSV Task Title",
        "description": "CLI build task",
        "karma": 100,
        "usage_count": 1,
        "variable_karma": false,
        "level": "level-uuid",
        "channel": "channel-uuid",
        "type": "task-type-uuid",
        "ig": "ig-uuid",
        "org": "org-uuid",
        "event": "event-name"
      }
    ],
    "Failed": [
      {
        "hashtag": "#duplicate-hashtag",
        "title": "Failed Task",
        "level": "Level 1",
        "channel": "intro",
        "type": "General",
        "ig": "Public Relations",
        "org": "MU",
        "event": "onboarding",
        "error": "Duplicate hashtag in database: #duplicate-hashtag"
      }
    ]
  }
}
```

### 1.9 Download Task Template
*   **Method:** `GET`
*   **Path:** `/base-template/`
*   **Allowed Roles:** Any Authenticated User
*   **Usage:** Download an Excel sheet (`task_base_template.xlsx`) pre-filled with levels, channels, task types, interest groups, organizations, and events data options column-wise inside the "Data Definitions" worksheet.
*   **Request Body:** None
*   **Response Body (200 OK):** Binary file download of `task_base_template.xlsx`.

---

## 2. Task Dropdown / Reference APIs

These dropdowns are used in forms when creating or editing tasks to populate selections.

**Base Path:** `/api/v1/dashboard/task/`

| Endpoint | Method | Role Allowed | Response Schema Description |
|---|---|---|---|
| `/channel/` | `GET` | `Admin`, `Fellow`, `Associate` | List of `[ { "id": "uuid", "name": "channel-name" } ]` |
| `/ig/` | `GET` | `Admin`, `Fellow`, `Associate` | List of `[ { "id": "uuid", "name": "ig-name" } ]` |
| `/organization/` | `GET` | `Admin`, `Fellow`, `Associate` | List of `[ { "id": "uuid", "title": "org-title" } ]` |
| `/level/` | `GET` | `Admin`, `Fellow`, `Associate` | List of `[ { "id": "uuid", "name": "level-name" } ]` |
| `/task-types/` | `GET` | `Admin`, `Fellow`, `Associate` | List of `[ { "id": "uuid", "title": "tasktype-title" } ]` |
| `/events/` | `GET` | `Admin` | List of all configured events: `[ "event1", "event2", ... ]` |

---

## 3. Task Type CRUD APIs

These APIs manage task categories (e.g. "General", "Hackathon", "Project").

**Base Path:** `/api/v1/dashboard/task/`

### 3.1 List Task Types
*   **Method:** `GET`
*   **Path:** `/list-task-type/`
*   **Allowed Roles:** `Admin`
*   **Usage:** List task types with pagination and title search.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      {
        "id": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
        "title": "General",
        "updated_by": "System Admin",
        "updated_at": "2026-06-06T10:00:00Z",
        "created_by": "System Admin",
        "created_at": "2026-06-01T08:00:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 3.2 Create Task Type
*   **Method:** `POST`
*   **Path:** `/list-task-type/`
*   **Allowed Roles:** `Admin`
*   **Request Body (JSON):**
```json
{
  "title": "New Challenge Category"
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task type added successfully"
    ]
  },
  "response": {}
}
```

### 3.3 Update Task Type
*   **Method:** `PUT`
*   **Path:** `/task-type/<str:task_type_id>/`
*   **Allowed Roles:** `Admin`
*   **Request Body (JSON):**
```json
{
  "title": "Updated Challenge Category"
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Updated Challenge Category updated successfully"
    ]
  },
  "response": {}
}
```

### 3.4 Delete Task Type
*   **Method:** `DELETE`
*   **Path:** `/task-type/<str:task_type_id>/`
*   **Allowed Roles:** `Admin`
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Updated Challenge Category Deleted Successfully"
    ]
  },
  "response": {}
}
```

---

## 4. Admin Task Approval Workflow

Submitted tasks from companies or mentors are saved with a status of `pending` and remain inactive (`active=false`) until an administrator reviews them.

**Base Path:** `/api/v1/dashboard/task/`

### 4.1 List Pending Tasks Awaiting Review
*   **Method:** `GET`
*   **Path:** `/pending/`
*   **Allowed Roles:** `Admin`
*   **Usage:** Lists all submitted tasks that are awaiting approval.
*   **Query Parameters:**
    *   `approval_status` (string, defaults to `"pending"`): Review status (`pending`, `approved`, `rejected`).
    *   `source` / `role` (string): Filter by submitter role (`mentor`, `company`, or `admin` [tasks created directly by admin]).
    *   `company_name` (string): Filter tasks requested by users associated with specific company.
    *   `mentor_name` (string): Filter tasks requested by specific IG mentor.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Tasks fetched successfully."
    ]
  },
  "response": {
    "tasks": [
      {
        "id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
        "title": "Build a Rate Limiter",
        "hashtag": "#rate-limiter",
        "description": "Implement a token-bucket rate limiter with test cases.",
        "karma": 500,
        "approval_status": "pending",
        "ig": null,
        "type": {
          "id": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
          "title": "Implementation"
        },
        "company_name": "Acme Technologies",
        "requested_by": {
          "id": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
          "full_name": "Jane Developer"
        },
        "requested_at": "2026-06-05T14:30:00Z",
        "created_at": "2026-06-05T14:30:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 4.2 Approve or Reject Task
*   **Method:** `PATCH`
*   **Path:** `/<str:task_id>/review/`
*   **Allowed Roles:** `Admin`
*   **Usage:** Approve or reject a task awaiting review. Approved tasks become active (`active=true`) and live for learners. Rejected tasks remain inactive (`active=false`) with rejection explanation.
*   **Request Body (Approve - JSON):**
```json
{
  "action": "approve"
}
```
*   **Response Body (Approve - 200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task approved and is now live."
    ]
  },
  "response": {
    "task_id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
    "approval_status": "approved",
    "active": true,
    "rejection_reason": null,
    "reviewed_by": "user-uuid-admin",
    "reviewed_at": "2026-06-06T10:15:30Z"
  }
}
```
*   **Request Body (Reject - JSON):**
```json
{
  "action": "reject",
  "reason": "Please provide a more descriptive description and specify concrete learning outcomes."
}
```
*   **Response Body (Reject - 200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task rejected."
    ]
  },
  "response": {
    "task_id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
    "approval_status": "rejected",
    "active": false,
    "rejection_reason": "Please provide a more descriptive description and specify concrete learning outcomes.",
    "reviewed_by": "user-uuid-admin",
    "reviewed_at": "2026-06-06T10:18:22Z"
  }
}
```

---

## 5. Company Task Management

**Base Path:** `/api/v1/dashboard/company/`

To use these endpoints, the user must have a **verified** company profile (the user created it and it's marked as verified) OR be an approved `COMPANY_MENTOR` for the company.

### 5.1 List Company Tasks
*   **Method:** `GET`
*   **Path:** `tasks/`
*   **Access Allowed:** Verified company owner or approved company mentor.
*   **Usage:** Lists all tasks submitted by the authenticated company.
*   **Query Parameters:**
    *   `approval_status` (string, optional): Filter by review status (`pending`, `approved`, `rejected`).
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      {
        "id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
        "hashtag": "#rate-limiter",
        "discord_link": null,
        "title": "Build a Rate Limiter",
        "description": "Implement a token-bucket rate limiter with test cases.",
        "karma": 500,
        "channel": null,
        "type": "Implementation",
        "active": false,
        "variable_karma": false,
        "usage_count": 1,
        "level": "Level 4",
        "org": "Acme Technologies",
        "ig": null,
        "event": null,
        "bonus_karma": 0,
        "bonus_time": null,
        "approval_status": "pending",
        "rejection_reason": null,
        "reviewed_at": null,
        "requested_by_name": "Jane Developer",
        "requested_at": "2026-06-05T14:30:00Z",
        "skills": [
          {
            "id": "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef",
            "name": "Backend Development",
            "code": "BACKEND01"
          }
        ],
        "created_at": "2026-06-05T14:30:00Z",
        "updated_at": "2026-06-05T14:30:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 5.2 Create Company Task
*   **Method:** `POST`
*   **Path:** `tasks/`
*   **Access Allowed:** Verified company owner or approved company mentor.
*   **Usage:** Submits a new company task. Saved as `pending` and `active=false` until approved by an admin.
*   **Request Body (JSON):**
```json
{
  "hashtag": "#acme-widget-api",
  "title": "Build Acme Widget API",
  "karma": 300,
  "usage_count": 1,
  "description": "Implement standard CRUD API with widget validation.",
  "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
  "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef"
  ]
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task submitted for approval."
    ]
  },
  "response": {}
}
```

### 5.3 Get Company Task Details
*   **Method:** `GET`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Verified company owner or approved company mentor (only for tasks created by their company).
*   **Request Body:** None
*   **Response Body (200 OK):** Single task object (same schema as list elements in Section 5.1).

### 5.4 Update Company Task (Full PUT)
*   **Method:** `PUT`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Verified company owner or approved company mentor.
*   **Usage:** Fully updates a task.
*   **Warning:** Regardless of the current status, updating a task resets `approval_status` to `pending`, `active` to `false`, and clears any previous reviews/rejection reasons.
*   **Request Body (JSON):**
```json
{
  "hashtag": "#acme-widget-api-v2",
  "title": "Build Acme Widget API v2",
  "karma": 350,
  "usage_count": 1,
  "description": "Implement standard CRUD API with widget validation.",
  "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
  "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef"
  ]
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task updated and re-submitted for approval."
    ]
  },
  "response": {}
}
```

### 5.5 Update Company Task (Partial PATCH)
*   **Method:** `PATCH`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Verified company owner or approved company mentor.
*   **Usage:** Partially updates a task (e.g., just changing the title or description). Same state reset behavior applies (reverts to `pending`/`inactive`).
*   **Request Body (JSON):**
```json
{
  "title": "Build Acme Widget API (Refactored)",
  "karma": 400
}
```
*   **Response Body (200 OK):** Returns the fully updated company task in `response` property:
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task updated successfully and submitted for admin review."
    ]
  },
  "response": {
    "id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
    "hashtag": "#acme-widget-api",
    "discord_link": null,
    "title": "Build Acme Widget API (Refactored)",
    "description": "Implement standard CRUD API with widget validation.",
    "karma": 400,
    "channel": null,
    "type": "Implementation",
    "active": false,
    "variable_karma": false,
    "usage_count": 1,
    "level": "Level 4",
    "org": "Acme Technologies",
    "ig": null,
    "event": null,
    "bonus_karma": 0,
    "bonus_time": null,
    "approval_status": "pending",
    "rejection_reason": null,
    "reviewed_at": null,
    "requested_by_name": "Jane Developer",
    "requested_at": "2026-06-05T14:30:00Z",
    "skills": [],
    "created_at": "2026-06-05T14:30:00Z",
    "updated_at": "2026-06-06T10:45:00Z"
  }
}
```

### 5.6 Soft Delete Company Task
*   **Method:** `DELETE`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Verified company owner or approved company mentor.
*   **Usage:** Deletes task.
*   **Implementation Note:** In contrast to global tasks which are hard-deleted, company tasks are soft-deleted by setting `active=false`.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task deleted successfully."
    ]
  },
  "response": {
    "task_id": "e382d56a-1111-482f-87ab-8f9261a8ef1b",
    "deleted_at": "2026-06-06T10:50:00Z"
  }
}
```

---

## 6. Mentor Task Management

**Base Path:** `/api/v1/dashboard/mentor/`

To use these endpoints, the user must have the active role of `Mentor`.

### 6.1 Get Active Interest Groups Dropdown
*   **Method:** `GET`
*   **Path:** `tasks/ig-dropdown/`
*   **Access Allowed:** Active Mentor
*   **Usage:** Fetch only the Interest Groups the mentor is actively assigned to manage. Used to populate the IG selector when creating tasks.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": [
    {
      "id": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
      "name": "Public Relations"
    }
  ]
}
```

### 6.2 List Mentor-submitted Tasks
*   **Method:** `GET`
*   **Path:** `tasks/`
*   **Access Allowed:** Active Mentor
*   **Usage:** List all tasks requested/submitted by the authenticated mentor. Supports searching, sorting, and pagination.
*   **Query Parameters:**
    *   `approval_status` (string, optional): Filter by `pending`, `approved`, or `rejected`.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      {
        "id": "a1f8d2cc-0000-482f-87ab-8f9261a8ef1a",
        "hashtag": "#pr-campaign-design",
        "discord_link": null,
        "title": "Design a PR Campaign",
        "description": "Create a comprehensive public relations strategy presentation.",
        "karma": 300,
        "channel": null,
        "type": "General",
        "active": false,
        "variable_karma": false,
        "usage_count": 1,
        "level": "Level 3",
        "org": null,
        "ig": "Public Relations",
        "event": null,
        "bonus_karma": 0,
        "bonus_time": null,
        "approval_status": "pending",
        "rejection_reason": null,
        "reviewed_at": null,
        "requested_by_name": "Jane Mentor",
        "requested_at": "2026-06-06T10:00:00Z",
        "skills": [],
        "created_at": "2026-06-06T10:00:00Z",
        "updated_at": "2026-06-06T10:00:00Z"
      }
    ],
    "pagination": {
      "count": 1,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null,
      "prevPage": null
    }
  }
}
```

### 6.3 Create Mentor Task
*   **Method:** `POST`
*   **Path:** `tasks/`
*   **Access Allowed:** Active Mentor
*   **Usage:** Submit a new task for admin review.
*   **Warning:** The selector field `ig` (Interest Group) is strictly validated to ensure the mentor belongs to it.
*   **Request Body (JSON):**
```json
{
  "hashtag": "#pr-campaign-design",
  "title": "Design a PR Campaign",
  "karma": 300,
  "usage_count": 1,
  "description": "Create a comprehensive public relations strategy presentation.",
  "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
  "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
  "ig": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef"
  ]
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task submitted for approval."
    ]
  },
  "response": {}
}
```

### 6.4 Get Mentor Task Details
*   **Method:** `GET`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Active Mentor (restricted to tasks requested by the authenticated mentor)
*   **Request Body:** None
*   **Response Body (200 OK):** Single task object (same schema as list elements in Section 6.2).

### 6.5 Update Mentor Task
*   **Method:** `PUT`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Active Mentor
*   **Usage:** Fully updates a task. Reverts `approval_status` to `pending`, disables `active` status (`active=false`), and clears previous approval parameters.
*   **Request Body (JSON):**
```json
{
  "hashtag": "#pr-campaign-design-v2",
  "title": "Design a PR Campaign v2",
  "karma": 350,
  "usage_count": 1,
  "description": "Create a comprehensive public relations strategy presentation.",
  "type": "c4d081f9-03c0-4351-87ab-8f9261a8ef1b",
  "level": "4d1db261-0021-4f38-aef4-4f014e7a5522",
  "ig": "d0f0d111-9e7f-4409-b6fc-d6b8bf589be9",
  "skill_ids": [
    "e4b3e86c-7eb8-4221-88fc-8d1ef546c2ef"
  ]
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task updated and re-submitted for approval."
    ]
  },
  "response": {}
}
```

### 6.6 Hard Delete Mentor Task
*   **Method:** `DELETE`
*   **Path:** `tasks/<str:task_id>/`
*   **Access Allowed:** Active Mentor
*   **Usage:** Deletes task.
*   **Warning:** Only allowed if the task is still in a `pending` approval status.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Task deleted successfully."
    ]
  },
  "response": {}
}
```

---

## 7. Launchpad Task & Student Verification APIs

These APIs are specifically used by the launchpad career portal for task-based job listings.

**Base Path:** `/api/v1/launchpad/`

### 7.1 Admin Verify Launchpad Job Task
*   **Method:** `POST`
*   **Path:** `verify-task/`
*   **Allowed Roles:** `Admin`
*   **Usage:** Verify a task submitted as a requirement for a Launchpad job opening, mapping it to a hashtag.
*   **Request Body (JSON):**
```json
{
  "task_id": "9a12c4fe-1111-48fc-87ab-8f9261a8ef8a",
  "hashtag": "#lp-widget-task",
  "is_verified": true
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "task_id": "9a12c4fe-1111-48fc-87ab-8f9261a8ef8a",
    "task_description": "Task description details...",
    "hashtag": "#lp-widget-task",
    "is_verified": true,
    "message": "Task verified successfully."
  }
}
```

### 7.2 List Eligible Students by Task Completion
*   **Method:** `GET`
*   **Path:** `list-launchpad-students/<str:job_id>/`
*   **Allowed Roles:** `Recruiter`, `Company` (must own/manage the job)
*   **Usage:** Lists students who have successfully completed the task associated with the job (determined by matching the task hashtag).
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Job list fetched successfully"
    ]
  },
  "response": [
    {
      "id": "user-uuid-1122",
      "full_name": "Dev Student",
      "email": "student@example.com",
      "muid": "dev-student@mulearn",
      "profile_pic": "https://storage.googleapis.com/...",
      "karma": 1200,
      "level": "Level 4",
      "college_name": "Govt Engineering College",
      "interest_groups": [
        {
          "id": "ig-uuid-1",
          "name": "Backend Development"
        }
      ],
      "roles": [
        "Learner"
      ],
      "rank": 1,
      "karma_distribution": [
        {
          "task_type": "Implementation",
          "karma": 500
        }
      ],
      "application_id": "app-uuid-999",
      "application_status": "applied",
      "application_timeline": {
        "invited_at": "2026-06-05T10:00:00Z",
        "applied_at": "2026-06-06T09:00:00Z"
      },
      "candidate_links": {
        "resume_link": "https://drive.google.com/resume",
        "linkedin_link": "https://linkedin.com/in/...",
        "portfolio_link": "https://portfolio.dev",
        "cover_letter": "I'd love to join...",
        "other_link": null,
        "links_available": true
      }
    }
  ]
}
```

---

## 8. Task Report APIs

**Base Path:** `/api/v1/dashboard/task-report/`

> [!WARNING]
> **Technical Concern:** These URLs are defined in `api/dashboard/task_report/urls.py` but are **not** mounted in the sub-url configuration (`api/dashboard/urls.py` or the main `api/urls.py`). As a result, these endpoints are currently unreachable in production/dev. They are documented here to assist in future integration.

### 8.1 List All Task Reports
*   **Method:** `GET`
*   **Path:** `/`
*   **Allowed Roles:** `Admin`, `Fellow`
*   **Usage:** Lists all flagged task submissions/reports.
*   **Query Parameters:**
    *   `status` (string, optional): Filter by report status (`pending`, `resolved`, etc.).
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": [
    {
      "id": "report-uuid-001",
      "reporter": "Jane Reporter",
      "offender": "Bob Spammer",
      "message_id": "987654321012345678",
      "reason": "Submitted plagiarized link as proof.",
      "proof_link": "http://discord.com/channels/...",
      "status": "pending",
      "created_at": "2026-06-05 12:00:00",
      "updated_at": "2026-06-05 12:00:00"
    }
  ]
}
```

### 8.2 Update Task Report Status
*   **Method:** `PUT`
*   **Path:** `/<str:report_id>/`
*   **Allowed Roles:** `Admin`, `Fellow`
*   **Request Body (JSON):**
```json
{
  "status": "resolved"
}
```
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Report status updated successfully"
    ]
  },
  "response": {}
}
```

### 8.3 Task Report Grouping Analytics
*   **Method:** `GET`
*   **Path:** `/group-by-reporter/`
*   **Allowed Roles:** `Admin`, `Fellow`
*   **Usage:** Retrieve aggregated statistics of reporters submitting flags, and offenders accumulating reports.
*   **Request Body:** None
*   **Response Body (200 OK):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "reporters": [
      {
        "reporter__id": "user-uuid-1",
        "reporter__fullname": "Jane Reporter",
        "report_count": 5
      }
    ],
    "offenders": [
      {
        "offender__id": "user-uuid-2",
        "offender__fullname": "Bob Spammer",
        "report_count": 8
      }
    ]
  }
}
```

---

## 9. Senior Software Developer Notes & Concerns

During static analysis of the task, company task, and mentor task subsystems, several concerns and code inconsistencies were identified. Developers maintaining these endpoints should review the following details:

### 9.1 Unmounted Task Report Routes (Dead Endpoints)
The directory `api/dashboard/task_report/` is fully implemented containing serializers, views, and url patterns. However, `api/dashboard/urls.py` **omits** task report inclusion.
*   **Resolution Required:** Add the following line to `api/dashboard/urls.py` in the `urlpatterns` array:
    ```python
    path("task-report/", include("api.dashboard.task_report.urls")),
    ```

### 9.2 Inactive View (TaskReportTaskGroupingView)
In `api/dashboard/task_report/views.py`, there is a class called `TaskReportTaskGroupingView` (lines 50-80) which aggregates report counts by Discord message ID (`message_id`).
*   **Concern:** This class is completely absent from `api/dashboard/task_report/urls.py`. The routing only hooks up `TaskReportInfoView` and `TaskReportReporterGroupingView`. This analytics view is dead code unless wired.

### 9.3 Put Task Response Inconsistency
Most update APIs return the updated details inside the standard `response` field. However, `TaskAPI.put(...)` in `api/dashboard/task/dash_task_view.py` passes the serializer data to `general_message`:
```python
return CustomResponse(general_message=serializer.data).get_success_response()
```
This causes the client to receive the updated task object nested inside the envelope's `message.general[0]` list rather than inside the `response` dictionary, breaking standard envelope conventions.

### 9.4 Discrepancies in Task Deletion Logic
*   **Global Tasks & Mentor Tasks:** Hard-deleted from the database via `task.delete()`.
*   **Company Tasks:** Soft-deleted by mutating `task.active = False` in `CompanyTaskDetailAPI.delete`.
*   **Concern:** If a company soft-deletes a task (marking it inactive), it will still be stored in the DB but no longer display. However, they can never reuse that hashtag because of global uniqueness validation:
    ```python
    def validate_hashtag(self, value):
        qs = TaskList.objects.filter(hashtag=value)
        # ...
        if qs.exists():
            raise serializers.ValidationError("A task with this hashtag already exists.")
    ```
    This leaves the database cluttered with soft-deleted tasks that lock up valuable hashtags indefinitely. A hard delete or validation that checks `active=True` would resolve this limitation.
