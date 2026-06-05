# Mentorship Session Recurrence & Admin Approval API Documentation

This document describes the API endpoints, request/response models, validation rules, edge cases, and test details implemented for managing recurring mentorship sessions and their moderation workflows.

---

## 1. Create Mentorship Session

Creates a mentorship session. If recurrence fields are set, it generates the parent session and automatically spawns linked child sessions.

* **URL:** `/api/v1/dashboard/mentor/session/create/`
* **Method:** `POST`
* **Headers:** 
  * `Authorization: Bearer <token>`
* **Role Required:** `MENTOR`

### Request Body (Recurring Session Example)
```json
{
  "entity_id": "235ccbd6-07d9-445b-9236-078d5d2903b2",
  "session_type": "ig_session",
  "title": "Weekly Python Mentorship Session",
  "description": "Weekly deep-dive into advanced Python constructs.",
  "mode": "ONLINE",
  "starts_at": "2026-07-08T10:00:00Z",
  "ends_at": "2026-07-08T11:00:00Z",
  "meeting_link": "https://meet.google.com/abc-defg-hij",
  "venue": null,
  "max_participants": 20,
  "is_recurring": true,
  "recurrence_type": "WEEKLY",
  "recurrence_interval": 1,
  "recurrence_end_date": "2026-08-08"
}
```

### Response Body (200 OK)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": [
      "Session created successfully and is pending approval."
    ]
  },
  "response": {
    "id": "e2a39bcf-140b-47eb-9ee0-d5a28c31ab42",
    "entity_id": "235ccbd6-07d9-445b-9236-078d5d2903b2",
    "session_type": "ig_session",
    "title": "Weekly Python Mentorship Session",
    "description": "Weekly deep-dive into advanced Python constructs.",
    "mode": "ONLINE",
    "starts_at": "2026-07-08T10:00:00Z",
    "ends_at": "2026-07-08T11:00:00Z",
    "meeting_link": "https://meet.google.com/abc-defg-hij",
    "venue": null,
    "max_participants": 20,
    "is_recurring": true,
    "recurrence_type": "WEEKLY",
    "recurrence_interval": 1,
    "recurrence_end_date": "2026-08-08",
    "child_session_ids": [
      "5c4a5c5c-7d9a-4c28-91c6-2c93d9e29a01",
      "bfde8c12-32b0-40e1-a08b-498c0b93ca2d",
      "78a05c31-de8a-493a-867c-d67b2cb89e81"
    ]
  }
}
```

### Recurrence Field Validation Rules:
1. **If `is_recurring` is `True`:**
   * `recurrence_type`, `recurrence_interval`, and `recurrence_end_date` are strictly required. If any of these fields are missing or invalid, the API returns structured, field-specific validation errors:
     ```json
     {
       "hasError": true,
       "statusCode": 400,
       "message": {
         "recurrence_type": ["recurrence_type is required when is_recurring is true."],
         "recurrence_interval": ["recurrence_interval must be a positive integer when is_recurring is true."],
         "recurrence_end_date": ["recurrence_end_date is required when is_recurring is true."]
       }
     }
     ```
   * `recurrence_end_date` must be chronologically after the session's `starts_at` date.
2. **If `is_recurring` is `False` (or omitted):**
   * Even if the client sends dummy parameters for `recurrence_type`, `recurrence_interval`, or `recurrence_end_date`, they are automatically intercepted and forced to `null` before DB insertion to ensure database cleanliness.

---

## 2. List Sessions (Admin View)

Returns a list of all mentorship sessions. It contains updated recurrence fields allowing the client to easily distinguish single sessions, recurring parent sessions, and recurring child sessions.

* **URL:** `/api/v1/dashboard/mentor/session/admin/list/?status=PENDING_APPROVAL`
* **Method:** `GET`
* **Query Parameters:**
  * `status` (optional, e.g., `PENDING_APPROVAL`, `SCHEDULED`, `REJECTED`)
* **Headers:**
  * `Authorization: Bearer <token>`
* **Role Required:** `ADMIN`

### Response Body Example (200 OK)
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
        "id": "parent-session-uuid",
        "entity_id": "235ccbd6-07d9-445b-9236-078d5d2903b2",
        "entity_name": "Cloud And Devops",
        "session_type": "ig_session",
        "title": "Weekly Web Dev Standup",
        "mode": "ONLINE",
        "starts_at": "2026-07-08T10:00:00Z",
        "ends_at": "2026-07-08T11:00:00Z",
        "status": "PENDING_APPROVAL",
        "created_by_id": "mentor-uuid",
        "created_by_name": "Nandakishore P",
        "created_at": "2026-06-05T10:19:26Z",
        "max_participants": 20,
        "is_recurring": true,
        "parent_session_id": null,
        "recurrence_type": "WEEKLY",
        "recurrence_interval": 1,
        "recurrence_end_date": "2026-07-22"
      },
      {
        "id": "child-session-uuid-1",
        "entity_id": "235ccbd6-07d9-445b-9236-078d5d2903b2",
        "entity_name": "Cloud And Devops",
        "session_type": "ig_session",
        "title": "Weekly Web Dev Standup",
        "mode": "ONLINE",
        "starts_at": "2026-07-15T10:00:00Z",
        "ends_at": "2026-07-15T11:00:00Z",
        "status": "PENDING_APPROVAL",
        "created_by_id": "mentor-uuid",
        "created_by_name": "Nandakishore P",
        "created_at": "2026-06-05T10:19:26Z",
        "max_participants": 20,
        "is_recurring": true,
        "parent_session_id": "parent-session-uuid",
        "recurrence_type": "WEEKLY",
        "recurrence_interval": 1,
        "recurrence_end_date": "2026-07-22"
      }
    ],
    "pagination": {
      "count": 2,
      "totalPages": 1,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

### Client Classification Rules:
* **Single / Non-Recurring Session:** `"is_recurring": false` and `"parent_session_id": null`
* **Parent Recurring Session:** `"is_recurring": true` and `"parent_session_id": null`
* **Child Recurring Session:** `"is_recurring": true` and `"parent_session_id": "<parent_uuid>"`

---

## 3. Approve / Reject Mentorship Session

Allows an admin to approve or reject a session. If a session is part of a recurring chain, the admin can choose whether the decision applies only to the single target session, or cascades to the entire chain of pending sessions.

* **URL:** `/api/v1/dashboard/mentor/session/admin/verify/<session_id>/`
* **Method:** `PATCH`
* **Headers:**
  * `Authorization: Bearer <token>`
* **Role Required:** `ADMIN`

### Request Body Options:

#### Option A: Approve/Reject only the selected session (Single update)
```json
{
  "status": "SCHEDULED", // or "REJECTED"
  "apply_to_series": false
}
```
* **Response (200 OK):**
  ```json
  {
    "hasError": false,
    "statusCode": 200,
    "message": {
      "general": [
        "Session status updated to SCHEDULED successfully."
      ]
    },
    "response": {}
  }
  ```

#### Option B: Approve/Reject targeted session and all linked pending child sessions (Series-wide update)
```json
{
  "status": "SCHEDULED", // or "REJECTED"
  "apply_to_series": true
}
```
* **Response (200 OK):**
  * *If the session is recurring:*
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": [
          "Session and its pending series chain updated to SCHEDULED successfully."
        ]
      },
      "response": {}
    }
    ```
  * *If the session is non-recurring (automatically degrades to single session message):*
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": [
          "Session status updated to SCHEDULED successfully."
        ]
      },
      "response": {}
    }
    ```

---

## 4. Key Performance Optimizations

Updating a large series of recurring events can be resource-intensive due to the Django signals.
* **Single Save + Bulk Update:** Instead of looping through all child sessions and executing `.save()` for each record, the API runs `.save()` **only** on the targeted primary session (triggering model validate & the scope cache invalidation signal once).
* **Batch SQL:** All remaining pending child sessions are updated using Django's `.update()` in a single transaction. This prevents N database hits and N sequential Redis network connection timeouts, lowering update latency for a series of 50 sessions from **30+ seconds** to **~100ms**.

---

## 5. Summary of Automated Recurrence Tests

The recurrence logic is validated by the test suite located in `api/dashboard/mentor/tests/test_session_recurrence.py`:

1. **`test_create_weekly_series`**
   * Validates that setting `is_recurring=True` with a `WEEKLY` frequency successfully generates the correct number of child sessions.
   * Asserts that child sessions have `parent_session_id` pointing to the parent session, are marked `is_recurring=True`, and carry forward metadata like frequency.
2. **`test_recurrence_hard_cap`**
   * Verifies that the maximum number of child sessions generated does not exceed the hard limit of `50`, even if the `recurrence_end_date` is configured years in the future.
3. **`test_monthly_edge_case`**
   * Tests date calculation bounds (e.g. creating a monthly session starting on January 31st). Asserts that calculations correctly resolve month-end offsets (generating the next session on February 28th).
