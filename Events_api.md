# Complete Dashboard Events API Reference with Mock Responses

This document provides a line-by-line API reference for **every** endpoint defined in the Events module of the muLearn backend, including exact JSON mock responses for each.

**Base Path:** `/api/dashboard/events/`

---

## Standard Response Envelopes

All responses follow one of two standard envelopes depending on whether they are paginated or non-paginated.

### Non-Paginated Success Response Envelope
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Action message here."]
  },
  "response": {
    // API specific data here
  }
}
```

### Paginated Success Response Envelope
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "data": [
      // List of resources
    ],
    "pagination": {
      "count": 100,
      "totalPages": 10,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2
    }
  }
}
```

---

## Table of Contents
1. [Meta & Form Helpers](#1-meta--form-helpers)
2. [Scoped Feeds](#2-scoped-feeds)
3. [Admin Actions](#3-admin-actions)
4. [Mentor & Campus Approvals](#4-mentor--campus-approvals)
5. [Event Ownership & Details Management](#5-event-ownership--details-management)
6. [Co-Owner Connections](#6-co-owner-connections)
7. [Collaborator Connections](#7-collaborator-connections)
8. [Event Task Management](#8-event-task-management)
9. [Event Analytics](#9-event-analytics)
10. [User Dashboard & Invites](#10-user-dashboard--invites)
11. [Public & Guest Access Endpoints](#11-public--guest-access-endpoints)
12. [Event type & scope values](#12-Event-type-&-scope-values)

---

## 1. Meta & Form Helpers

### 1.1 Retrieve Event Categories
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/meta/categories/`
*   **Use/Description:** Retrieve valid event categories.
*   **Role who can access:** Public (No Auth).
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event categories retrieved."]
      },
      "response": [
        {
          "id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
          "name": "Workshop",
          "description": "Interactive hands-on learning session"
        },
        {
          "id": "f8e7d6c5-b4a3-2c1d-0e9f-8a7b6c5d4e3f",
          "name": "Hackathon",
          "description": "Competitive coding event"
        }
      ]
    }
    ```

### 1.2 Retrieve Organizer Options
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/meta/organizer-options/`
*   **Use/Description:** Retrieve list of organizations and IGs the caller can create events as.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Organiser options retrieved."]
      },
      "response": {
        "can_create_as_ig": [
          {
            "id": "e2f5b61c-843c-4a37-bc22-cf8a56213192",
            "name": "Python Interest Group",
            "icon": "python-icon",
            "code": "PYTHON"
          }
        ],
        "can_create_as_campus_ig": [],
        "can_create_as_campus": [
          {
            "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
            "title": "Government Engineering College",
            "org_type": "College"
          }
        ],
        "can_create_as_company": [
          {
            "id": "company-uuid-1",
            "title": "ACME Technologies"
          }
        ],
        "can_create_as_admin": false,
        "campus_context": {
          "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
          "title": "Government Engineering College"
        }
      }
    }
    ```

### 1.3 Retrieve Collaboration Targets (Search)
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/meta/collaboration-targets/`
*   **Use/Description:** Live autocomplete targets search for IGs, campuses, or companies to invite.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:**
    *   `search` (string, optional): Search term query.
    *   `type` (string, optional): Target filter. Options: `ig`, `campus`, `campus_ig`, `company`.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaboration targets retrieved."]
      },
      "response": {
        "ig": [
          {
            "id": "e2f5b61c-843c-4a37-bc22-cf8a56213192",
            "name": "Python Interest Group",
            "icon": "python-icon",
            "code": "PYTHON"
          }
        ],
        "campus": [
          {
            "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
            "title": "Government Engineering College",
            "org_type": "College"
          }
        ],
        "company": [],
        "campus_ig": []
      }
    }
    ```

### 1.4 Retrieve Event Types and Scopes
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/meta/event-type-scope/`
*   **Use/Description:** Retrieve the list of valid event types and event scopes formatted (first letter capitalized, underscores replaced by spaces).
*   **Role who can access:** Public (No Auth).
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event types and scopes retrieved."]
      },
      "response": {
        "event_type": [
          "Hackathon",
          "Workshop",
          "Webinar",
          "Seminar",
          "Bootcamp",
          "Meetup",
          "Conference",
          "Competition",
          "Ideathon",
          "Cultural event",
          "Sports event",
          "Community event",
          "Expo",
          "Networking event",
          "Tech talk",
          "Others"
        ],
        "event_scope": [
          "Maker",
          "Coder",
          "Manager",
          "Creative"
        ]
      }
    }
    ```

---

## 2. Scoped Feeds

### 2.1 Cluster Scope Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/ig/cluster/<str:cluster>/`
*   **Use/Description:** Fetch live cluster-specific interest group events.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** Standard pagination (`page`, `search`, `sortBy`).
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
            "title": "UI/UX Design Sprint",
            "slug": "uiux-design-sprint",
            "cover_image": "http://example.com/media/covers/uiux.png",
            "status": "published",
            "scope": "global",
            "event_scope": "creative",
            "event_type": "others",
            "start_datetime": "2026-07-20T09:00:00Z",
            "end_datetime": "2026-07-20T17:00:00Z",
            "venue": {
              "venue_type": "online",
              "venue_address": null,
              "venue_city": null,
              "venue_maps_url": null,
              "venue_online_link": "http://zoom.us/...",
              "venue_platform": "Zoom"
            },
            "organizer": {
              "organiser_type": "global_ig",
              "organiser_ig": {
                "id": "creative-ig-uuid",
                "name": "Design Interest Group",
                "icon": "design-icon"
              },
              "organiser_campus": null,
              "organiser_company": null,
              "organiser_ci_id": null
            },
            "is_featured": false,
            "is_collaboration": false,
            "interest_count": 35,
            "min_karma": 0,
            "tags": ["design", "uiux"],
            "user_limit": 100,
            "category_id": "cat-uuid-1",
            "category_name": "Sprint",
            "viewer_interest_status": "none"
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

### 2.2 Interest Group Scope Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/ig/<str:ig_id>/`
*   **Use/Description:** Fetch live events organised by or scoped to a specific IG.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** Standard pagination (`page`, `search`, `sortBy`).
*   **Request Body:** None.
*   **Response (200 OK):** Identical structure to Scoped Cluster list (2.1), filtered by Interest Group ID.

### 2.3 Campus Scope Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/campus/<str:campus_id>/`
*   **Use/Description:** Fetch live events scoped to or organised by a specific campus college.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:**
    *   `cluster` (string, optional): Cluster filter.
    *   Standard pagination (`page`, `search`, `sortBy`).
*   **Request Body:** None.
*   **Response (200 OK):** Identical structure to Scoped Cluster list (2.1), filtered by Campus ID.

### 2.4 Campus IG Scope Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/campus-ig/<str:campus_ig_id>/`
*   **Use/Description:** Fetch live events created by a specific campus IG chapter.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** Standard pagination (`page`, `search`, `sortBy`).
*   **Request Body:** None.
*   **Response (200 OK):** Identical structure to Scoped Cluster list (2.1), filtered by Campus Interest Group ID.

### 2.5 Partner Company Scope Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/company/<str:company_id>/`
*   **Use/Description:** Fetch live events organised by a partner company.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** Standard pagination (`page`, `search`, `sortBy`).
*   **Request Body:** None.
*   **Response (200 OK):** Identical structure to Scoped Cluster list (2.1), filtered by Partner Company ID.

---

## 3. Admin Actions

### 3.1 Retrieve Global Admin Event List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/admin/`
*   **Use/Description:** List all events in the system across all scopes and states.
*   **Role who can access:** `Admins`.
*   **Query Parameters:** `status`, `organiser_type`, `created_by`, `scope`, `is_featured`, `search`, `sortBy`.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
            "title": "Internal Python Workshop",
            "slug": "internal-python-workshop",
            "cover_image": null,
            "status": "draft",
            "scope": "campus",
            "event_scope": "coder",
            "event_type": "others",
            "start_datetime": "2026-07-20T09:00:00Z",
            "end_datetime": "2026-07-20T17:00:00Z",
            "venue": {
              "venue_type": "offline",
              "venue_address": "Tech Room 2",
              "venue_city": "Kochi",
              "venue_maps_url": null,
              "venue_online_link": null,
              "venue_platform": null
            },
            "organizer": {
              "organiser_type": "campus",
              "organiser_ig": null,
              "organiser_campus": {
                "id": "campus-uuid-1",
                "title": "CET Trivandrum"
              },
              "organiser_company": null,
              "organiser_ci_id": null
            },
            "is_featured": false,
            "is_collaboration": false,
            "interest_count": 0,
            "min_karma": 0,
            "tags": [],
            "user_limit": 50,
            "category_id": "cat-uuid-1",
            "category_name": "Workshop",
            "viewer_interest_status": "none"
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

### 3.2 Direct Override Approve Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/admin/<str:event_id>/approve/`
*   **Use/Description:** Administrative override to approve a pending event.
*   **Role who can access:** `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event approved: pending_approval → published."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "status": "published"
      }
    }
    ```

### 3.3 Direct Override Reject Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/admin/<str:event_id>/reject/`
*   **Use/Description:** Administrative override to reject a pending event.
*   **Role who can access:** `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "reason": "Event contact details are missing."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event rejected (was: pending_approval)."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "status": "rejected",
        "reason": "Event contact details are missing."
      }
    }
    ```

### 3.4 Toggle Event Featured Status
*   **HTTP Method:** `PATCH`
*   **Endpoint:** `/api/dashboard/events/admin/<str:event_id>/feature/`
*   **Use/Description:** Toggle or explicitly configure event featured status.
*   **Role who can access:** `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON, optional):**
    ```json
    {
      "is_featured": true
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event has been featured."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "is_featured": true
      }
    }
    ```

---

## 4. Mentor & Campus Approvals

### 4.1 Mentor Approve Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/mentor/<str:event_id>/approve/`
*   **Use/Description:** Mentor approval on a pending event.
*   **Role who can access:** Verified `Mentor`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event approved successfully."]
      },
      "response": {}
    }
    ```

### 4.2 Mentor Reject Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/mentor/<str:event_id>/reject/`
*   **Use/Description:** Mentor rejection on a pending event.
*   **Role who can access:** Verified `Mentor`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "reason": "Missing proper contact details and links."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event rejected successfully."]
      },
      "response": {}
    }
    ```

### 4.3 Campus Lead Approve Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/campus/<str:event_id>/approve/`
*   **Use/Description:** Campus Lead approvals on college pending events.
*   **Role who can access:** Verified Campus Lead or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event approved successfully."]
      },
      "response": {}
    }
    ```

### 4.4 Campus Lead Reject Event
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/campus/<str:event_id>/reject/`
*   **Use/Description:** Campus Lead rejections on college pending events.
*   **Role who can access:** Verified Campus Lead or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "reason": "Clashing venue schedule with semester exams."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event rejected successfully."]
      },
      "response": {}
    }
    ```

---

## 5. Event Ownership & Details Management

### 5.1 Retrieve Manageable Events List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/`
*   **Use/Description:** List manageable events.
*   **Role who can access:** Authenticated users with creation or management roles.
*   **Query Parameters:** `status` (optional), Standard pagination.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
            "title": "Flutter Development Seminar",
            "slug": "flutter-development-seminar",
            "cover_image": null,
            "status": "draft",
            "scope": "campus",
            "event_scope": "coder",
            "event_type": "others",
            "start_datetime": "2026-07-20T09:00:00Z",
            "end_datetime": "2026-07-20T17:00:00Z",
            "venue": {
              "venue_type": "offline",
              "venue_address": "Tech Room 2",
              "venue_city": "Kochi",
              "venue_maps_url": null,
              "venue_online_link": null,
              "venue_platform": null
            },
            "organizer": {
              "organiser_type": "campus",
              "organiser_ig": null,
              "organiser_campus": {
                "id": "campus-uuid-1",
                "title": "CET Trivandrum"
              },
              "organiser_company": null,
              "organiser_ci_id": null
            },
            "is_featured": false,
            "is_collaboration": false,
            "interest_count": 0,
            "min_karma": 0,
            "tags": [],
            "user_limit": 50,
            "category_id": "cat-uuid-1",
            "category_name": "Seminar",
            "viewer_interest_status": "none"
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

### 5.2 Create Event Draft
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/`
*   **Use/Description:** Create new event.
*   **Role who can access:** Authenticated users with creation roles.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "title": "React JS Workshop",
      "description": "Learn frontend React core details.",
      "category": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
      "start_datetime": "2026-08-01T10:00:00Z",
      "end_datetime": "2026-08-01T17:00:00Z",
      "venue_type": "offline",
      "venue_address": "Lab 3, CSE Dept",
      "venue_city": "Kochi",
      "scope": "campus",
      "organiser_type": "campus",
      "organiser_org": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
      "scope_org": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
      "event_scope": "coder",
      "min_karma": 200,
      "tags": "[\"react\", \"frontend\"]"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event created successfully."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "title": "React JS Workshop",
        "slug": "react-js-workshop",
        "description": "Learn frontend React core details.",
        "cover_image": null,
        "banner_image": null,
        "category_id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
        "category_name": "Workshop",
        "status": "draft",
        "scope": "campus",
        "event_scope": "coder",
        "event_type": "others",
        "scope_org": {
          "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
          "title": "Government Engineering College",
          "org_type": "College"
        },
        "scope_ig": null,
        "scope_ci_id": null,
        "organizer": {
          "organiser_type": "campus",
          "organiser_campus": {
            "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
            "title": "Government Engineering College",
            "org_type": "College"
          },
          "organiser_ig": null,
          "organiser_company": null,
          "organiser_ci_id": null
        },
        "venue": {
          "venue_type": "offline",
          "venue_address": "Lab 3, CSE Dept",
          "venue_city": "Kochi",
          "venue_maps_url": null,
          "venue_online_link": null,
          "venue_platform": null
        },
        "start_datetime": "2026-08-01T10:00:00Z",
        "end_datetime": "2026-08-01T17:00:00Z",
        "registration_url": null,
        "registration_deadline": null,
        "min_karma": 200,
        "is_featured": false,
        "is_collaboration": false,
        "interest_count": 0,
        "tags": ["react", "frontend"],
        "user_limit": 0,
        "linked_tasks": [],
        "co_owners": [],
        "collaborators": [],
        "viewer_interest_status": "none",
        "viewer_can_access_registration": true,
        "viewer_access_blocked_reason": null,
        "created_by": {
          "id": "user-uuid-1",
          "full_name": "Gokul Shaji",
          "muid": "gokul@mulearn"
        },
        "updated_by": {
          "id": "user-uuid-1",
          "full_name": "Gokul Shaji",
          "muid": "gokul@mulearn"
        },
        "created_at": "2026-06-20T12:00:00Z",
        "updated_at": "2026-06-20T12:00:00Z"
      }
    }
    ```

### 5.3 Submit Event Draft for Review (Publish)
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/publish/`
*   **Use/Description:** Submit the draft event to the approval queue.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event submitted: status is now \"pending_mentor_approval\"."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "status": "pending_mentor_approval"
      }
    }
    ```

### 5.4 Retrieve Managed Event Details
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/`
*   **Use/Description:** Retrieve managed event detail page data along with audit histories.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event detail retrieved."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "title": "React JS Workshop",
        "slug": "react-js-workshop",
        "description": "Learn frontend React core details.",
        "cover_image": null,
        "banner_image": null,
        "category_id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
        "category_name": "Workshop",
        "status": "draft",
        "scope": "campus",
        "event_scope": "coder",
        "event_type": "others",
        "scope_org": {
          "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
          "title": "Government Engineering College"
        },
        "scope_ig": null,
        "scope_ci_id": null,
        "organizer": {
          "organiser_type": "campus",
          "organiser_campus": {
            "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
            "title": "Government Engineering College"
          }
        },
        "venue": {
          "venue_type": "offline",
          "venue_address": "Lab 3, CSE Dept",
          "venue_city": "Kochi"
        },
        "start_datetime": "2026-08-01T10:00:00Z",
        "end_datetime": "2026-08-01T17:00:00Z",
        "registration_url": null,
        "registration_deadline": null,
        "min_karma": 200,
        "is_featured": false,
        "is_collaboration": false,
        "interest_count": 0,
        "tags": ["react", "frontend"],
        "user_limit": 0,
        "linked_tasks": [],
        "co_owners": [],
        "collaborators": [],
        "viewer_interest_status": "none",
        "viewer_can_access_registration": true,
        "viewer_access_blocked_reason": null,
        "created_by": { "id": "u-1", "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
        "updated_by": { "id": "u-1", "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
        "created_at": "2026-06-20T12:00:00Z",
        "updated_at": "2026-06-20T12:00:00Z",
        "edit_history": [
          {
            "id": "log-uuid",
            "action": "event_created",
            "edited_by": { "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
            "summary": "Event created",
            "changes": {},
            "details": {},
            "edited_at": "2026-06-20T12:00:00Z"
          }
        ]
      }
    }
    ```

### 5.5 Update Managed Event Details (Full)
*   **HTTP Method:** `PUT`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/`
*   **Use/Description:** Overwrite event details.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):** Full Event Write fields.
*   **Response (200 OK):** Identical response dictionary format to managed detail retrieve (5.4) without `edit_history`.

### 5.6 Update Managed Event Details (Partial)
*   **HTTP Method:** `PATCH`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/`
*   **Use/Description:** Partial updates to event details.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):** Partial Event fields.
*   **Response (200 OK):** Identical response dictionary format to managed detail retrieve (5.4) without `edit_history`.

### 5.7 Cancel Event (Soft Delete)
*   **HTTP Method:** `DELETE`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/`
*   **Use/Description:** Cancel/Soft-delete the event.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event has been cancelled."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "status": "cancelled"
      }
    }
    ```

---

## 6. Co-Owner Connections

### 6.1 List Event Co-Owners
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/co-owners/`
*   **Use/Description:** Retrieve co-owners list.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Co-owners retrieved."]
      },
      "response": [
        {
          "id": "connection-uuid-1",
          "entity_id": "coowner-user-uuid",
          "user": {
            "id": "coowner-user-uuid",
            "full_name": "Alan Turing",
            "muid": "alan@mulearn",
            "profile_pic": null
          },
          "added_by": {
            "id": "creator-user-uuid",
            "full_name": "Gokul Shaji",
            "muid": "gokul@mulearn",
            "profile_pic": null
          },
          "added_at": "2026-06-20T12:00:00Z"
        }
      ]
    }
    ```

### 6.2 Add Event Co-Owner
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/co-owners/`
*   **Use/Description:** Add a co-owner to the event.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "user_id": "coowner-user-uuid"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Co-owner added."]
      },
      "response": {
        "id": "connection-uuid-1",
        "entity_id": "coowner-user-uuid",
        "added_at": "2026-06-20T12:30:00Z"
      }
    }
    ```

### 6.3 Remove Event Co-Owner
*   **HTTP Method:** `DELETE`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/co-owners/<str:co_owner_id>/`
*   **Use/Description:** Remove co-owner right links from a user.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Co-owner removed."]
      },
      "response": {}
    }
    ```

---

## 7. Collaborator Connections

### 7.1 List Invited Collaborators
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/collaborators/`
*   **Use/Description:** List collaborator invite settings and states.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaborators retrieved."]
      },
      "response": [
        {
          "id": "collab-uuid-1",
          "entity_type": "collab_campus",
          "entity_id": "campus-org-uuid",
          "entity_detail": {
            "id": "campus-org-uuid",
            "title": "College of Engineering Trivandrum",
            "org_type": "College"
          },
          "role_label": "Venue Partner",
          "invite_status": "pending",
          "rejection_reason": null,
          "responded_at": null,
          "created_at": "2026-06-20T12:00:00Z"
        }
      ]
    }
    ```

### 7.2 Invite Collaborator
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/collaborators/`
*   **Use/Description:** Invite a collaborator.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "entity_type": "collab_campus",
      "entity_id": "campus-org-uuid",
      "role_label": "Venue Partner"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaborator invited."]
      },
      "response": {
        "id": "collab-uuid-1",
        "entity_type": "collab_campus",
        "entity_id": "campus-org-uuid",
        "invite_status": "pending",
        "role_label": "Venue Partner"
      }
    }
    ```

### 7.3 Accept Collaboration Invite
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/accept/`
*   **Use/Description:** Accept the sent invitation.
*   **Role who can access:** Contact leads for the invited target entity or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaboration accepted."]
      },
      "response": {
        "id": "collab-uuid-1",
        "invite_status": "accepted",
        "responded_at": "2026-06-20T12:45:00Z"
      }
    }
    ```

### 7.4 Reject Collaboration Invite
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/reject/`
*   **Use/Description:** Reject the invitation.
*   **Role who can access:** Contact leads for the invited target entity.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "reason": "Exams scheduled during those dates."
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaboration invite rejected."]
      },
      "response": {
        "id": "collab-uuid-1",
        "invite_status": "rejected",
        "rejection_reason": "Exams scheduled during those dates."
      }
    }
    ```

### 7.5 Remove Invited Collaborator
*   **HTTP Method:** `DELETE`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/`
*   **Use/Description:** Remove a collaborator invite/connection.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Collaborator removed."]
      },
      "response": {}
    }
    ```

---

## 8. Event Task Management

### 8.1 Retrieve Event Tasks List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/`
*   **Use/Description:** Retrieve tasks linked to the event.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** `search` (optional), `sortBy` (optional), Standard pagination.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "task-uuid-1",
            "hashtag": "#mureacttodo",
            "title": "Build React Todo App",
            "description": "Complete assignment and submit code repository.",
            "karma": 100,
            "approval_status": "approved",
            "active": true,
            "type": { "id": "type-uuid", "title": "Assignment" },
            "ig": { "id": "ig-uuid", "name": "Webdev IG" },
            "level": { "id": "level-uuid", "name": "Level 1" },
            "channel": "#general",
            "org": null,
            "variable_karma": false,
            "usage_count": 1,
            "bonus_time": null,
            "bonus_karma": null,
            "created_by": { "id": "u-1", "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
            "created_at": "2026-06-20T12:00:00Z",
            "updated_at": "2026-06-20T12:00:00Z"
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

### 8.2 Create Event Task
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/`
*   **Use/Description:** Create and link a new task.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):**
    ```json
    {
      "hashtag": "#mureacttodo",
      "title": "Build React Todo App",
      "description": "Complete assignment and submit code repository.",
      "karma": 100,
      "type": "type-uuid-1",
      "ig": "ig-uuid-1",
      "level": "level-uuid-1"
    }
    ```
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Task created and linked to event. Pending admin approval."]
      },
      "response": {
        "id": "task-uuid-1",
        "hashtag": "#mureacttodo",
        "title": "Build React Todo App",
        "description": "Complete assignment and submit code repository.",
        "karma": 100,
        "approval_status": "pending",
        "active": false,
        "type": { "id": "type-uuid-1", "title": "Assignment" },
        "ig": { "id": "ig-uuid-1", "name": "Webdev IG" },
        "level": { "id": "level-uuid-1", "name": "Level 1" },
        "channel": null,
        "org": null,
        "variable_karma": false,
        "usage_count": 1,
        "bonus_time": null,
        "bonus_karma": null,
        "created_by": { "id": "u-1", "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
        "created_at": "2026-06-20T12:00:00Z",
        "updated_at": "2026-06-20T12:00:00Z"
      }
    }
    ```

### 8.3 Get Event Task Detail
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/<str:task_id>/`
*   **Use/Description:** Retrieve specific task details.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Task detail retrieved."]
      },
      "response": {
        "id": "task-uuid-1",
        "hashtag": "#mureacttodo",
        "title": "Build React Todo App",
        "description": "Complete assignment and submit code repository.",
        "karma": 100,
        "approval_status": "approved",
        "active": true,
        "type": { "id": "type-uuid-1", "title": "Assignment" },
        "ig": { "id": "ig-uuid-1", "name": "Webdev IG" },
        "level": { "id": "level-uuid-1", "name": "Level 1" },
        "channel": "#general",
        "org": null,
        "variable_karma": false,
        "usage_count": 1,
        "bonus_time": null,
        "bonus_karma": null,
        "created_by": { "id": "u-1", "full_name": "Gokul Shaji", "muid": "gokul@mulearn" },
        "created_at": "2026-06-20T12:00:00Z",
        "updated_at": "2026-06-20T12:00:00Z"
      }
    }
    ```

### 8.5 Update Event Task (Partial)
*   **HTTP Method:** `PATCH`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/<str:task_id>/`
*   **Use/Description:** Partially update event task details.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body (JSON):** Partial task fields.
*   **Response (200 OK):** Identical structure to Task Detail (8.3).

### 8.5 Delete Event Task
*   **HTTP Method:** `DELETE`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/<str:task_id>/`
*   **Use/Description:** Delete a task.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Task deleted successfully."]
      },
      "response": {}
    }
    ```

### 8.6 Get Event Task Form Metadata
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/tasks/meta/`
*   **Use/Description:** Fetch dropdown options for task fields.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Task metadata retrieved."]
      },
      "response": {
        "task_types": [
          { "id": "type-uuid-1", "title": "Assignment" }
        ],
        "interest_groups": [
          { "id": "ig-uuid-1", "name": "Webdev IG" }
        ],
        "levels": [
          { "id": "level-uuid-1", "name": "Level 1" }
        ],
        "channels": [
          { "id": "channel-uuid-1", "name": "#webdev" }
        ],
        "organizations": []
      }
    }
    ```

---

## 9. Event Analytics

### 9.1 Retrieve Event Performance Stats
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/manage/<str:event_id>/analytics/`
*   **Use/Description:** Retrieve event signups trend, summary metrics, and task completions.
*   **Role who can access:** Event owner / co-owner or `Admins`.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event analytics retrieved."]
      },
      "response": {
        "summary": {
          "total_interests": 150,
          "total_tasks": 2,
          "approved_tasks": 2,
          "pending_tasks": 0,
          "total_task_completions": 85,
          "total_karma_awarded": 17000
        },
        "interest_trend": [
          {
            "date": "2026-06-19",
            "count": 150
          }
        ],
        "task_breakdown": [
          {
            "task_id": "task-uuid-1",
            "title": "Build React Todo App",
            "hashtag": "#mureacttodo",
            "karma": 100,
            "approval_status": "approved",
            "completions": 85,
            "total_karma_awarded": 8500
          }
        ]
      }
    }
    ```

---

## 10. User Dashboard & Invites

### 10.1 Retrieve User Collaboration Invites
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/my-invites/`
*   **Use/Description:** Retrieve collaboration invites directed to the user's campuses or IGs.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Pending invites retrieved."]
      },
      "response": [
        {
          "id": "collab-uuid-1",
          "entity_type": "collab_campus",
          "entity_id": "campus-org-uuid",
          "role_label": "Venue Partner",
          "invite_status": "pending",
          "event_id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
          "event_title": "React JS Workshop",
          "event_start_datetime": "2026-08-01T10:00:00Z",
          "event_cover_image": null
        }
      ]
    }
    ```

---

## 11. Public & Guest Access Endpoints

### 11.1 Retrieve Calendar Feed
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/calendar/`
*   **Use/Description:** Calendar feed overlapping requested range.
*   **Role who can access:** Public.
*   **Query Parameters:**
    *   `start_date` (string, required): `YYYY-MM-DD`
    *   `end_date` (string, required): `YYYY-MM-DD`
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": [
        {
          "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
          "title": "Flutter Development Seminar",
          "slug": "flutter-development-seminar",
          "status": "published",
          "start": "2026-07-20T09:00:00Z",
          "end": "2026-07-20T17:00:00Z",
          "venue_type": "offline",
          "organiser_name": "CET Trivandrum",
          "category_name": "Seminar",
          "is_featured": false
        }
      ]
    }
    ```

### 11.2 Retrieve Featured Events Feed (featured)
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/featured/`
*   **Use/Description:** Retrieve featured events.
*   **Role who can access:** Public.
*   **Query Parameters:** Standard query filters.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Featured events retrieved."]
      },
      "response": {
        "data": [
          {
            "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
            "title": "Hackathon 2026",
            "slug": "hackathon-2026",
            "cover_image": null,
            "status": "published",
            "scope": "global",
            "event_scope": "coder",
            "event_type": "others",
            "start_datetime": "2026-07-20T09:00:00Z",
            "end_datetime": "2026-07-22T18:00:00Z",
            "venue": {
              "venue_type": "offline",
              "venue_address": "Tech Hall",
              "venue_city": "Kochi",
              "venue_maps_url": null,
              "venue_online_link": null,
              "venue_platform": null
            },
            "organizer": {
              "organiser_type": "global_ig",
              "organiser_ig": {
                "id": "ig-uuid-1",
                "name": "Webdev IG",
                "icon": "web-icon"
              },
              "organiser_campus": null,
              "organiser_company": null,
              "organiser_ci_id": null
            },
            "is_featured": true,
            "is_collaboration": false,
            "interest_count": 85,
            "min_karma": 200,
            "tags": ["coding", "hackathon"],
            "user_limit": 200,
            "category_id": "cat-uuid-1",
            "category_name": "Hackathon",
            "viewer_interest_status": "none"
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

### 11.3 Retrieve Featured Events Feed (is-featured)
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/is-featured/`
*   **Use/Description:** Alias for featured events listing.
*   **Role who can access:** Public.
*   **Query Parameters:** Standard query filters.
*   **Request Body:** None.
*   **Response (200 OK):** Identical structure to Featured events (11.2).

### 11.4 Retrieve Public Task List
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/tasks/`
*   **Use/Description:** Public list of tasks linked to active events that the viewer is authorized to see.
*   **Role who can access:** Public.
*   **Query Parameters:** standard query filters.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "task-uuid-1",
            "title": "Build React Todo App",
            "hashtag": "#mureacttodo",
            "karma": 100,
            "description": "Complete assignment and submit code repository.",
            "type": "Assignment",
            "ig": "Webdev IG",
            "channel": "#general",
            "event": "React JS Workshop"
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

### 11.5 Express Event Interest (User Going)
*   **HTTP Method:** `POST`
*   **Endpoint:** `/api/dashboard/events/<str:event_id>/interest/`
*   **Use/Description:** Express interest. Checks `min_karma` limits.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["You're now marked as interested in this event."]
      },
      "response": {
        "event_id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "user_id": "user-uuid-1",
        "status": "interested"
      }
    }
    ```

### 11.6 Remove Expressed Event Interest
*   **HTTP Method:** `DELETE`
*   **Endpoint:** `/api/dashboard/events/<str:event_id>/interest/`
*   **Use/Description:** Remove expressed interest.
*   **Role who can access:** Authenticated users.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Your interest has been removed."]
      },
      "response": {}
    }
    ```

### 11.7 Retrieve Event Details (Public View)
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/<str:event_id>/`
*   **Use/Description:** Fetch event details page data. Non-live states (draft, pending) are restricted.
*   **Role who can access:** Public.
*   **Query Parameters:** None.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {
        "general": ["Event detail retrieved."]
      },
      "response": {
        "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
        "title": "React JS Workshop",
        "slug": "react-js-workshop",
        "description": "Learn frontend React core details.",
        "cover_image": null,
        "banner_image": null,
        "category_id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
        "category_name": "Workshop",
        "status": "published",
        "scope": "campus",
        "event_scope": "coder",
        "event_type": "others",
        "scope_org": {
          "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
          "title": "Government Engineering College"
        },
        "scope_ig": null,
        "scope_ci_id": null,
        "organizer": {
          "organiser_type": "campus",
          "organiser_campus": {
            "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
            "title": "Government Engineering College"
          }
        },
        "venue": {
          "venue_type": "offline",
          "venue_address": "Lab 3, CSE Dept",
          "venue_city": "Kochi"
        },
        "start_datetime": "2026-08-01T10:00:00Z",
        "end_datetime": "2026-08-01T17:00:00Z",
        "registration_url": null,
        "registration_deadline": null,
        "min_karma": 200,
        "is_featured": false,
        "is_collaboration": false,
        "interest_count": 50,
        "tags": ["react", "frontend"],
        "user_limit": 100,
        "linked_tasks": [
          {
            "id": "task-uuid-1",
            "title": "Build React Todo App",
            "description": "Complete assignment and submit code repository.",
            "hashtag": "#mureacttodo",
            "karma": 100,
            "bonus_time": null,
            "bonus_karma": null,
            "active": true,
            "ig": {
              "id": "ig-uuid-1",
              "name": "Webdev IG",
              "icon": null
            }
          }
        ],
        "co_owners": [],
        "collaborators": [],
        "viewer_interest_status": "none",
        "viewer_can_access_registration": true,
        "viewer_access_blocked_reason": null,
        "created_by": {
          "id": "u-1",
          "full_name": "Gokul Shaji",
          "muid": "gokul@mulearn",
          "profile_pic": null
        },
        "updated_by": {
          "id": "u-1",
          "full_name": "Gokul Shaji",
          "muid": "gokul@mulearn",
          "profile_pic": null
        },
        "created_at": "2026-06-20T12:00:00Z",
        "updated_at": "2026-06-20T12:00:00Z"
      }
    }
    ```

### 11.8 Retrieve Events List (Public View)
*   **HTTP Method:** `GET`
*   **Endpoint:** `/api/dashboard/events/`
*   **Use/Description:** Retrieve active list of events. Scope-aware filtering logic is applied dynamically if the caller is authenticated.
*   **Role who can access:** Public.
*   **Query Parameters:** `search`, `sortBy`, `event_type`, `ig_id`, `campus_id`, `cluster`, `start_date`, `end_date`, `tags`, `eligible_only`.
*   **Request Body:** None.
*   **Response (200 OK):**
    ```json
    {
      "hasError": false,
      "statusCode": 200,
      "message": {},
      "response": {
        "data": [
          {
            "id": "e3a1f07d-08b2-4d0f-a49f-b70c66fae911",
            "title": "React JS Workshop",
            "slug": "react-js-workshop",
            "cover_image": null,
            "status": "published",
            "scope": "campus",
            "event_scope": "coder",
            "event_type": "others",
            "start_datetime": "2026-08-01T10:00:00Z",
            "end_datetime": "2026-08-01T17:00:00Z",
            "venue": {
              "venue_type": "offline",
              "venue_address": "Lab 3, CSE Dept",
              "venue_city": "Kochi",
              "venue_maps_url": null,
              "venue_online_link": null,
              "venue_platform": null
            },
            "organizer": {
              "organiser_type": "campus",
              "organiser_ig": null,
              "organiser_campus": {
                "id": "8f7d6e5a-4b3c-2d1e-0f9a-8b7c6d5e4f3a",
                "title": "Government Engineering College",
                "org_type": "College"
              },
              "organiser_company": null,
              "organiser_ci_id": null
            },
            "is_featured": false,
            "is_collaboration": false,
            "interest_count": 50,
            "min_karma": 200,
            "tags": ["react", "frontend"],
            "user_limit": 100,
            "category_id": "a1b2c3d4-e5f6-7a8b-9c0d-e1f2a3b4c5d6",
            "category_name": "Workshop",
            "viewer_interest_status": "none"
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

## 12. Event type & scope values

### Event Types:
* Hackathon
* Workshop
* Webinar
* Seminar
* Bootcamp
* Meetup
* Conference
* Competition
* Ideathon
* Cultural event
* Sports event
* Community event
* Expo
* Networking event
* Tech talk
* Others

### Event Scopes:
* Maker
* Coder
* Manager
* Creative
