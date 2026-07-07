# Unified Dashboard Calendar API

**API Name**: Dashboard Calendar View
**Endpoint**: `GET /api/v1/dashboard/calendar/events/`
**Description**: This unified endpoint returns both Events and Mentorship Sessions pre-sorted into `upcoming`, `ongoing`, and `completed` buckets (max 100 items per bucket). 

**Permissions / Access Control**:
The API automatically detects the user's role from their JWT token and filters the calendar accordingly:
- **Global Events**: Visible to everyone (including unauthenticated users).
- **Company Events**: Visible to everyone (including unauthenticated users).
- **IG Events**: Only visible to users in that specific IG (unauthenticated users see all IG events).
- **Campus Events**: Only visible to users in that specific campus.
- **Mentor Sessions**: Only visible to the assigned mentor.

**Path Parameters**:
- None

**Query Parameters**:
You must provide either the `month` parameter OR the `start_date` & `end_date` parameters.
- `month` (string, optional): Name or abbreviation of the month (e.g., `June`, `jun`). Automatically defaults to the current year.
- `year` (integer, optional): Use alongside `month` if you need a specific year (e.g., `2026`). Defaults to current year if omitted.
- `start_date` (date, optional): Required if `month` is omitted. Format: `YYYY-MM-DD`. Max 93-day window.
- `end_date` (date, optional): Required if `month` is omitted. Format: `YYYY-MM-DD`. Max 93-day window.
- `status` (string, optional): Only return items for a specific bucket: `upcoming`, `ongoing`, or `completed`.

**Supported Ordering Fields**:
- None (Items are pre-sorted chronologically by start date/time automatically).

**Headers**:
- `Authorization`: Optional. Provide a valid Bearer Token for role-based viewing. If omitted, public data is returned.

**Request Body**:
- None (GET request)

**Sample Request**:
```http
GET /api/v1/dashboard/calendar/events/?month=June 
Authorization: Bearer <your_jwt_token>
```

**Success Response**:
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "events": {
      "upcoming": [
        {
          "id": "uuid",
          "title": "Hackathon 2026",
          "description": "Annual hackathon",
          "start_datetime": "2026-06-15T10:00:00Z",
          "end_datetime": "2026-06-16T18:00:00Z",
          "status": "Published",
          "category": "Event",
          "cover_image": "http://image-url"
        }
      ],
      "ongoing": [],
      "completed": []
    },
    "sessions": {
      "upcoming": [
        {
          "id": "uuid",
          "title": "React Native Mentorship",
          "description": "1 on 1 session",
          "starts_at": "2026-06-20T10:00:00Z",
          "ends_at": "2026-06-20T11:00:00Z",
          "status": "SCHEDULED",
          "meeting_link": "https://meet.google.com/..."
        }
      ],
      "ongoing": [],
      "completed": []
    }
  }
}
```

**Response Fields**:
- `events`: Object containing `upcoming`, `ongoing`, and `completed` arrays of event objects.
- `sessions`: Object containing `upcoming`, `ongoing`, and `completed` arrays of mentorship session objects. Will be empty if the user is not a Mentor.

**Error Responses**:
- **401 Unauthorized**:
```json
{
  "hasError": true,
  "statusCode": 401,
  "message": {
    "general": ["Invalid or expired token."]
  },
  "response": {}
}
```
- **400 Bad Request (Invalid Date Range)**:
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": ["Date range must not exceed 93 days."]
  },
  "response": {}
}
```
