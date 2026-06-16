# Notification

Base path: `/api/v1/notification/`

This document outlines the API endpoints for direct and broadcast notifications within the MuLearn system.

---

## Overview: Notification Types

The system supports two main types of notifications:

### 1. Direct Notifications (Personal Alerts)
* **Scope:** Addressed to a single specific user. Only that user will receive it.
* **Lifetime:** Persistent. It remains in the user's notification list indefinitely until the user explicitly deletes it (either individually or by clearing all).
* **Examples & Triggers:**
  * **Learning Circle Join Request:** Sent to a circle lead when a member requests to join.
  * **Learning Circle Request Decision:** Sent to a member when their request to join is approved or rejected by the lead.
  * **Event Approval Decisions:** Sent to an event creator when their event is approved or rejected by a campus lead, mentor, or admin.
  * **Interest Group Status Changes:** Sent to the requester when an Interest Group request is approved or rejected.

### 2. Broadcast Notifications (Group / Audience Announcements)
* **Scope:** Targeted to specific cohorts of users based on their campus, Interest Groups, or event participation. The backend resolves these memberships dynamically at request-time.
* **Lifetime:** Temporary / Time-bound. They automatically disappear/expire after the `expires_at` timestamp. They cannot be individually deleted by normal users; only administrators can delete/manage them globally.
* **Target Scope Scenarios & Examples:**
  * `global`: Addressed to all users in the system (e.g. system-wide announcements). `target_id` is `null`.
  * `campus`: Addressed to users in a specific campus organization (e.g. alert when a new learning circle is formed). `target_id` is the campus `<org_id>`.
  * `interest_group`: Addressed to learners inside an Interest Group (e.g. new interest group events). `target_id` is the `<ig_id>`.
  * `campus_ig`: Addressed to users who are members of both the campus and the Interest Group in a specific Campus IG Chapter. `target_id` is the `<campus_ig_chapter_id>`.
  * `event_interest`: Addressed to users who have registered interest in a specific event (e.g. event cancellation alerts). `target_id` is the `<event_id>`.


---

## User Endpoints

### Endpoint: `list/`
- **Method:** `GET`
- **Brief:** Retrieval list of all notifications for the authenticated user.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Success"
    ]
  },
  "response": {
    "direct": [
      {
        "id": "<notification_id>",
        "title": "Member Request",
        "description": "John Doe has requested to join your learning circle 'Python Basics'",
        "button": "View",
        "url": "https://mulearn.org/dashboard/learningcircle/<circle_id>/",
        "created_at": "2026-06-15T22:30:00Z",
        "user": "<user_id>",
        "created_by": "<created_by_user_id>"
      }
    ],
    "broadcasts": [
      {
        "id": "<broadcast_id>",
        "title": "New Event Published",
        "description": "A new Interest Group event 'Hackathon 2026' is now live!",
        "url": "/events/<event_id>/",
        "target_type": "interest_group",
        "target_id": "<interest_group_id>",
        "created_by": "<created_by_user_id>",
        "created_at": "2026-06-15T22:31:00Z",
        "expires_at": "2026-06-22T22:31:00Z"
      }
    ]
  }
}
```

### Endpoint: `delete/id/<str:notification_id>/`
- **Method:** `DELETE`
- **Brief:** Resource-specific deletion endpoint for a direct notification.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Path params:**
  - `str:notification_id` (The UUID of the direct notification to delete)
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Notification deleted successfully"
    ]
  },
  "response": {}
}
```

### Endpoint: `delete/all/`
- **Method:** `DELETE`
- **Brief:** Collection deletion endpoint to delete all direct notifications for the authenticated user.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "All notification deleted successfully"
    ]
  },
  "response": {}
}
```

---

## Admin Broadcast CRUD Endpoints

> [!NOTE]
> All admin broadcast endpoints require the authenticated user to have the `Admins` role.

### Endpoint: `broadcast/list/all/`
- **Method:** `GET`
- **Brief:** List all broadcast notifications in the system with resolved target names.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Success"
    ]
  },
  "response": [
    {
      "id": "<broadcast_id>",
      "title": "New Event Published",
      "description": "A new Interest Group event 'Hackathon 2026' is now live!",
      "url": "/events/<event_id>/",
      "target_type": "interest_group",
      "target_id": "<interest_group_id>",
      "target_details": {
        "type": "interest_group",
        "name": "Web Development"
      },
      "created_by": "<created_by_user_id>",
      "created_by_name": "Admin User",
      "created_at": "2026-06-15T22:31:00Z",
      "expires_at": "2026-06-22T22:31:00Z"
    }
  ]
}
```

### Endpoint: `broadcast/create/`
- **Method:** `POST`
- **Brief:** Create a new global broadcast announcement. The API automatically sets `target_type` to `global` and `target_id` to `null`.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Request Body (JSON):**
```json
{
  "title": "Maintenance Window",
  "description": "The site will be down for scheduled maintenance tonight from 12 AM to 2 AM UTC.",
  "url": "/status/",
  "expires_at": "2026-06-17T02:00:00Z"
}
```
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Broadcast notification created successfully."
    ]
  },
  "response": {
    "id": "<broadcast_id>",
    "title": "Maintenance Window",
    "description": "The site will be down for scheduled maintenance tonight from 12 AM to 2 AM UTC.",
    "url": "/status/",
    "target_type": "global",
    "target_id": null,
    "target_details": {
      "type": "global",
      "name": "Global (All Users)"
    },
    "created_by": "<created_by_user_id>",
    "created_by_name": "Admin User",
    "created_at": "2026-06-16T10:40:00Z",
    "expires_at": "2026-06-17T02:00:00Z"
  }
}
```

### Endpoint: `broadcast/update/id/<str:broadcast_id>/`
- **Method:** `PATCH`
- **Brief:** Partially update an existing broadcast notification by its UUID.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Path params:**
  - `str:broadcast_id` (The UUID of the broadcast notification record to update)
- **Request Body (JSON - all fields optional):**
```json
{
  "title": "Urgent Maintenance Window",
  "expires_at": "2026-06-17T04:00:00Z"
}
```
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Broadcast notification updated successfully."
    ]
  },
  "response": {
    "id": "<broadcast_id>",
    "title": "Urgent Maintenance Window",
    "description": "The site will be down for scheduled maintenance tonight from 12 AM to 2 AM UTC.",
    "url": "/status/",
    "target_type": "global",
    "target_id": null,
    "target_details": {
      "type": "global",
      "name": "Global (All Users)"
    },
    "created_by": "<created_by_user_id>",
    "created_by_name": "Admin User",
    "created_at": "2026-06-16T10:40:00Z",
    "expires_at": "2026-06-17T04:00:00Z"
  }
}
```

### Endpoint: `broadcast/delete/id/<str:broadcast_id>/`
- **Method:** `DELETE`
- **Brief:** Delete a single broadcast notification.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Path params:**
  - `str:broadcast_id` (The UUID of the broadcast notification record to delete)
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Broadcast notification deleted successfully"
    ]
  },
  "response": {}
}
```

### Endpoint: `broadcast/delete/all/`
- **Method:** `DELETE`
- **Brief:** Bulk delete all broadcast notifications in the system.
- **Request Headers:**
  - `Authorization: Bearer <token>`
- **Request Body:** None
- **Response example (success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "All 5 broadcast notification(s) deleted successfully"
    ]
  },
  "response": {}
}
```
