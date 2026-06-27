# 🛠️ Backend Fixes & Features — MuLearn Campus Dashboard
**Date:** 2026-06-27  
**Module:** `api/dashboard/campus/`  
**Base prefix:** `/api/v1/dashboard/campus/`

---

## 1. 📊 Filtering Campus Student CSV

**File:** [`campus_views.py` L293](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/campus_views.py#L293-L387)

### Endpoint
```
GET /api/v1/dashboard/campus/student-details/csv/
```

### Query Parameters (all optional)

| Param | Type | Description |
|---|---|---|
| `is_alumni` | `boolean` | Filter by alumni status (`true`/`false`) |
| `ig` | `UUID` | Filter by Interest Group ID |
| `category` | `string` | Filter by IG category |

### What Changed
Before this fix, the CSV export had no filter support. The endpoint now accepts the above query params and applies them to both the karma ranking queryset and the user queryset before serializing and returning the CSV file.

### Example Request
```
GET /api/v1/dashboard/campus/student-details/csv/?ig=<ig-uuid>&category=technology&is_alumni=false
```

### Response
Returns a downloadable CSV file: `Campus Student Details.csv`

---

## 2. 🗺️ Validating Google Maps URLs

**File:** [`serializers.py` L509](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/serializers.py#L509-L534)

### Endpoints Affected
```
POST  /api/v1/dashboard/campus/ig-chapters/
PATCH /api/v1/dashboard/campus/ig-chapters/<chapter_id>/
```

### What Changed
Added a `validate_icon_link` (URL field) in `CampusIGChapterCreateSerializer` and `CampusIGChapterUpdateSerializer`. Additionally, Google Maps URL validation was added to ensure submitted location/map URLs conform to accepted Google Maps patterns:

- `https://maps.google.*`  
- `https://goo.gl/maps/...`  
- `https://www.google.com/maps/...`

Any other URL format is rejected with a validation error.

### Validation Error Response
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "location_url": [
      "Enter a valid Google Maps URL (e.g. https://maps.google.com/... or https://goo.gl/maps/...)."
    ]
  },
  "response": {}
}
```

---

## 3. 👥 Fixing Manage Leadership Team Issues

**File:** [`events_views.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/events_views.py#L158-L388)

### Endpoints Affected
```
GET    /api/v1/dashboard/campus/execom/
POST   /api/v1/dashboard/campus/execom/
DELETE /api/v1/dashboard/campus/execom/<member_id>/
```

### What Changed
- `GET` — Fixed the blacklist of system roles so that all non-execom roles are properly excluded. Dynamic IG roles were missing from the listing because `IG_LEAD` was not in the blacklist.
- `POST` — Fixed role assignment: the new member's role is now validated against the campus's active IG chapters before being assigned.
- `DELETE` — Added guard to prevent the campus lead from removing their own Campus Lead role.

### GET Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "data": [
      {
        "id": "<user_role_link_uuid>",
        "user_id": "<user_uuid>",
        "full_name": "John Doe",
        "muid": "john-doe@mulearn",
        "profile_pic": "https://...",
        "role": "Design Lead"
      }
    ]
  }
}
```

### POST Request Body
```json
{
  "muid": "john-doe@mulearn",
  "role_title": "WebDev Tech Lead"
}
```

### POST Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Role assigned successfully",
  "response": {}
}
```

### DELETE Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Role removed successfully",
  "response": {}
}
```

---

## 4. 🔍 Fixing Execom Search Functionality

**File:** [`events_views.py` L483](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/events_views.py#L483-L573)  
**URL:** [`urls.py` L111](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/urls.py#L111-L115)

### Endpoint (NEW)
```
GET /api/v1/dashboard/campus/execom/search/?q=<query>
```

### Root Cause
The old search relied on an exact MUID lookup, which excluded users whose full names matched but didn't share the exact MUID. The new endpoint uses `__icontains` on both `full_name` and `muid` via a Django `Q` object.

### Query Parameters

| Param | Type | Required | Description |
|---|---|---|---|
| `q` | `string` | ✅ Yes | Case-insensitive partial match on `full_name` or `muid` |

### Example Request
```
GET /api/v1/dashboard/campus/execom/search/?q=john
```

### Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "data": [
      {
        "id": "<user_uuid>",
        "full_name": "John Doe",
        "muid": "john-doe@mulearn",
        "profile_pic": "https://..."
      }
    ]
  }
}
```

> **Note:** Returns up to 20 results. Only non-alumni active campus members are returned.

---

## 5. 🔄 Fixing IG Chapter Lead Transfer

**File:** [`campus_views.py` L620](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/campus_views.py#L620-L720)

### Endpoints
```
GET  /api/v1/dashboard/campus/transfer-ig-role/
POST /api/v1/dashboard/campus/transfer-ig-role/
```

### Root Cause
The `POST` handler was returning a success message even when the DB update failed because the `UserRoleLink.objects.filter(...).delete()` deleted the old role but then the serializer failed silently — not rolling back. Fixed by wrapping in `transaction.atomic()` to ensure atomic rollback on failure.

### GET Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "ig_list": ["AI", "WebDev", "Design"]
  }
}
```

### POST Request Body
```json
{
  "new_ig_muid": "jane-doe@mulearn",
  "ig_code": "WebDev"
}
```

### POST Response Body (Success)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Assigned new IG Campus Lead successfully",
  "response": {}
}
```

### POST Response Body (Failure — user not in campus)
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": "Can't find the user in your college",
  "response": {}
}
```

---

## 6. 🖼️ Adding Icon Link Support to IG Chapters

**Files:**  
- [`serializers.py` L509](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/serializers.py#L509-L550)  
- [`serializers.py` L553](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/serializers.py#L553-L593)  
- [`db/campus.py` L21](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/db/campus.py#L21)

### Endpoints Affected
```
POST  /api/v1/dashboard/campus/ig-chapters/
PATCH /api/v1/dashboard/campus/ig-chapters/<chapter_id>/
GET   /api/v1/dashboard/campus/ig-chapters/
```

### What Changed
`icon_link` field added to the `CampusIGChapter` model as `URLField(max_length=500, blank=True, null=True)`. It is now accepted in create and update serializers, and returned in the list serializer.

### POST Request Body
```json
{
  "ig": "<ig-uuid>",
  "description": "Our WebDev chapter",
  "icon_link": "https://example.com/icons/webdev.png",
  "lead": "jane-doe@mulearn"
}
```

### POST Response Body (Success)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "IG Chapter created successfully",
  "response": {}
}
```

### PATCH Request Body (partial update)
```json
{
  "icon_link": "https://example.com/icons/webdev-v2.png"
}
```

### GET Response Body (IG Chapter List)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": [
    {
      "id": "<chapter_uuid>",
      "ig_id": "<ig_uuid>",
      "ig_name": "Web Development",
      "ig_code": "WebDev",
      "ig_icon": "https://...",
      "lead_id": "<user_uuid>",
      "lead_name": "Jane Doe",
      "description": "Our WebDev chapter",
      "icon_link": "https://example.com/icons/webdev.png",
      "is_active": true,
      "campus_ig_member_count": 12
    }
  ]
}
```

---

## 7. 🔁 Synchronizing Interest Group Lists

**Files:**  
- [`campus_views.py` L620](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/campus_views.py#L620-L642) — `TransferIGRoleAPI.get()`  
- [`campus_views.py` L1011](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/campus_views.py#L1011-L1054) — `CampusIGChapterAPI.post()`

### Root Cause
`Transfer IG Lead Role` listed only active campus `CampusIGChapter` IG codes, while `Create IG Chapter` fetched all IGs from the system `InterestGroup` table. These were different datasets. Fixed by aligning the `GET /transfer-ig-role/` response to return only the active campus IGs — consistent with what's available for chapter creation.

### Endpoint (GET — now fixed & consistent)
```
GET /api/v1/dashboard/campus/transfer-ig-role/
```

### Response Body
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {},
  "response": {
    "ig_list": ["AI", "WebDev", "Design"]
  }
}
```
> These codes now reflect **only** the active `CampusIGChapter` entries for the authenticated campus, matching the list shown during chapter creation.

---

## 8. 👥 IG Chapter Join / Leave APIs

**File:** [`ig_views.py` L69](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/ig_views.py#L69-L206)

### Endpoints (NEW)
```
POST   /api/v1/dashboard/campus/ig-chapters/<chapter_id>/join/
DELETE /api/v1/dashboard/campus/ig-chapters/<chapter_id>/leave/
```

### POST — Join IG Chapter

**Request:** No body required (uses JWT for user identification)

**Response (Success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successfully joined the Campus IG Chapter",
  "response": {}
}
```

**Validation Rules:**
- Chapter must be active
- User must be a verified college member
- User must belong to the same campus as the chapter
- If already a member, returns error: `"You are already a member of this Interest Group"`

---

### DELETE — Leave IG Chapter

**Request:** No body required

**Response (Success):**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": "Successfully left the Campus IG Chapter",
  "response": {}
}
```

**Notes:**
- Performs a **soft-delete** — `UserIgLink.is_active` is set to `False` and `unassigned_at` is recorded
- History is preserved for analytics

---

## 📁 Files Modified Summary

| File | Changes |
|---|---|
| [`campus_views.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/campus_views.py) | CSV filters, transfer role fixes, execom fixes, IG sync |
| [`serializers.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/serializers.py) | `icon_link` support, Google Maps URL validation |
| [`ig_views.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/ig_views.py) | New Join/Leave chapter endpoints |
| [`events_views.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/events_views.py) | New execom user search endpoint, execom delete guard |
| [`urls.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/api/dashboard/campus/urls.py) | Registered `execom/search/`, `ig-chapters/<id>/join/`, `ig-chapters/<id>/leave/` |
| [`db/campus.py`](file:///c:/Users/prana/Desktop/company_v2/mulearnbackend/db/campus.py) | Added `icon_link` URLField to `CampusIGChapter` model |
