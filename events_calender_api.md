# `GET /events/calendar/` — Event Calendar API

**View:** [`EventCalendarAPI`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/events/public_views.py#L355-L419)  
**URL:** [`path('calendar/', ...)`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/events/urls.py#L62)  
**Serializer:** [`EventCalendarItemSerializer`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/events/serializers.py#L285-L303)

---

## Overview

Returns a lightweight list of events that **overlap** a requested date range, formatted specifically for calendar-widget display. Each item carries only the fields a calendar UI needs (title, slug, timing, venue type, organiser, category, and featured flag) — not the full event detail.

Visibility is **scope-aware** and **authentication-optional**:

| Caller | Events returned |
|---|---|
| **Unauthenticated** | `scope = global` only |
| **Authenticated** | `global` + any `campus`, `ig`, `campus_ig`, `company` event the user belongs to |

> [!IMPORTANT]
> The date range must not exceed **93 days**. Requests spanning more will be rejected.

---

## Authentication

| Property | Value |
|---|---|
| Required | **No** — but a valid JWT token broadens visibility to scoped events |
| Method | Bearer token in `Authorization` header (optional) |

---

## Request

### Method & URL

```
GET /events/calendar/
```

### Query Parameters

| Parameter | Type | Required | Format | Description |
|---|---|---|---|---|
| `start_date` | string | ✅ Yes | `YYYY-MM-DD` | Inclusive start of the calendar window |
| `end_date` | string | ✅ Yes | `YYYY-MM-DD` | Inclusive end of the calendar window |

> [!NOTE]
> Internally, `end_date` is made **end-of-day inclusive** by adding 1 day (i.e. events that **start before** midnight of `end_date + 1` and **end after** `start_date` are returned). This means an event that starts exactly on `end_date` will be included.

### Example Request

```http
GET /events/calendar/?start_date=2025-07-01&end_date=2025-07-31
Authorization: Bearer <optional-jwt-token>
```

---

## Response

### `200 OK` — Success

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Done",
  "response": [
    {
      "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
      "title": "Kerala Dev Summit 2025",
      "slug": "kerala-dev-summit-2025",
      "status": "published",
      "start": "2025-07-12T09:00:00Z",
      "end": "2025-07-12T17:00:00Z",
      "venue_type": "physical",
      "organiser_name": "TinkerHub",
      "category_name": "Technology",
      "is_featured": true
    },
    {
      "id": "6ba7b810-9dad-11d1-80b4-00c04fd430c8",
      "title": "Open Source Weekend",
      "slug": "open-source-weekend-jul-2025",
      "status": "ongoing",
      "start": "2025-07-19T00:00:00Z",
      "end": "2025-07-20T23:59:00Z",
      "venue_type": "online",
      "organiser_name": "muLearn",
      "category_name": null,
      "is_featured": false
    }
  ]
}
```

### Response Body — Top Level

| Field | Type | Description |
|---|---|---|
| `hasError` | boolean | `false` on success |
| `statusCode` | integer | HTTP-like status code |
| `message` | string | Human-readable status message |
| `response` | array of objects | List of calendar event items |

### Response Body — Each Event Object

| Field | Type | Nullable | Description |
|---|---|---|---|
| `id` | string (UUID) | No | Unique event identifier |
| `title` | string | No | Display title of the event |
| `slug` | string | No | URL-safe unique slug for deep-linking |
| `status` | string (enum) | No | Current lifecycle status — see [Status values](#status-enum) |
| `start` | string (ISO 8601 datetime) | No | Event start time (maps to `start_datetime`) |
| `end` | string (ISO 8601 datetime) | No | Event end time (maps to `end_datetime`) |
| `venue_type` | string (enum) | No | Format of the event — see [Venue Type values](#venue-type-enum) |
| `organiser_name` | string | No | Display name of the organiser — `"muLearn"` when no specific organiser is set |
| `category_name` | string | Yes | Human-readable event category; `null` when no category assigned |
| `is_featured` | boolean | No | Whether the event is currently featured on the platform |

---

## Enum Reference

### Status Enum

Only events with the following statuses are returned:

| Value | Description |
|---|---|
| `published` | Approved and live, upcoming |
| `ongoing` | Currently in progress |
| `completed` | Already ended (still visible in calendar range) |

### Venue Type Enum

| Value | Description |
|---|---|
| `physical` | In-person event at a physical location |
| `online` | Fully remote / virtual event |
| `hybrid` | Both in-person and online attendance options |

---

## Error Responses

### `400 Bad Request` — Missing parameters

Returned when either `start_date` or `end_date` is absent.

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Both start_date and end_date query parameters are required.",
  "response": []
}
```

### `400 Bad Request` — Invalid date format

Returned when a date string doesn't match `YYYY-MM-DD`.

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Invalid date format. Use YYYY-MM-DD.",
  "response": []
}
```

### `400 Bad Request` — Inverted date range

Returned when `start_date` is after `end_date`.

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "start_date must be before or equal to end_date.",
  "response": []
}
```

### `400 Bad Request` — Range too wide

Returned when the range exceeds 93 days.

```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Date range must not exceed 93 days.",
  "response": []
}
```

---

## Visibility / Scope Rules

The query is filtered by the caller's membership. The scope logic is implemented in [`_build_scope_filter()`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/events/public_views.py#L41-L85).

| Event Scope | Visible to |
|---|---|
| `global` | Everyone (authenticated & unauthenticated) |
| `campus` | Users with a verified `UserOrganizationLink` to `scope_org` |
| `ig` | Users with a `UserIgLink` to `scope_ig` |
| `campus_ig` | Users who belong to both `scope_org` (campus) **and** `scope_ig` |
| `company` | Users whose verified org matches the event's `scope_org` (company) |

---

## Ordering

Events are returned ordered by `start_datetime` ascending, with ties broken by `end_datetime` ascending.

---

## Notes

- **Soft-deleted events** are never returned (`deleted_at IS NULL` enforced via `get_live_events()`).
- The `organiser_name` field falls back to `"muLearn"` if the organiser IG or org reference is missing.
- All datetime values are returned in **UTC ISO 8601** format.
- The endpoint is intentionally **lightweight** — use `GET /events/<event_id>/` for full event details.
