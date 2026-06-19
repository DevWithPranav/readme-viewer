# Task Public List API Documentation

## Endpoint Details
**URL:** `/api/dashboard/task/list/`  
**Method:** `GET`  
**Authentication:** Optional Bearer Token (Authenticated learners get personalized tasks; unauthenticated users only see global and company tasks).

## Request Parameters (Query String)

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `pageIndex` | Integer | No | The page number to fetch (default is usually 1). |
| `perPage` | Integer | No | The number of items per page. |
| `search` | String | No | Search query for hashtag, title, description, karma, etc. |
| `sortBy` | String | No | Field name to sort by (e.g., `karma`, `created_at`). |
| `sortOrder` | String | No | Sort direction: `ASC` or `DESC`. |
| `ig_id` | String (UUID) | No | Filter tasks specifically by a given Interest Group ID. |

---

## Example Request

```http
GET /api/dashboard/task/list/?pageIndex=1&perPage=10&sortBy=created_at&sortOrder=DESC
Authorization: Bearer <your_jwt_token_here> (Optional)
```

---

## Example JSON Response

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
        "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
        "hashtag": "#learn-react",
        "title": "Build a React Component",
        "description": "Create a reusable React functional component with hooks.",
        "karma": 100,
        "channel": "frontend-dev",
        "discord_id": "987654321098765432",
        "type": "Learning Task",
        "variable_karma": false,
        "level": "L1",
        "ig": "Frontend",
        "event": null,
        "event_id": null
      },
      {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "hashtag": "#company-internship",
        "title": "Submit Resume",
        "description": "Submit your resume for the summer internship program.",
        "karma": 50,
        "channel": null,
        "discord_id": null,
        "type": "Career Task",
        "variable_karma": true,
        "level": null,
        "ig": null,
        "event": "Summer Hiring 2026",
        "event_id": "abcdef12-3456-7890-abcd-ef1234567890"
      }
    ],
    "pagination": {
      "count": 42,
      "totalPages": 5,
      "isNext": true,
      "isPrev": false,
      "nextPage": 2,
      "prevPage": null
    }
  }
}
```

## Response Field Definitions

### Root Object
- `hasError` (boolean): Indicates if the request failed.
- `statusCode` (integer): HTTP equivalent application status code (e.g., 200).
- `message` (object): Contains a `general` array of strings for success/error messages.
- `response` (object): Contains `data` and `pagination` details.

### `data` Array Elements
- `id` (string/uuid): Unique identifier of the task.
- `hashtag` (string): The unique hashtag associated with the task.
- `title` (string): Display title of the task.
- `description` (string): Detailed description of what the task entails.
- `karma` (integer): Amount of karma points awarded upon completion.
- `channel` (string | null): The Discord channel name where the task resides.
- `discord_id` (string | null): The Discord channel ID.
- `type` (string): The category/type of the task.
- `variable_karma` (boolean): Flag indicating if the karma is variable or fixed.
- `level` (string | null): The difficulty level assigned to the task.
- `ig` (string | null): The Interest Group this task is assigned to.
- `event` (string | null): Event name associated with this task.
- `event_id` (string/uuid | null): Foreign key ID to the event table.

### `pagination` Object
- `count` (integer): Total number of items matching the query.
- `totalPages` (integer): Total number of available pages.
- `isNext` (boolean): `true` if there is a subsequent page.
- `isPrev` (boolean): `true` if there is a preceding page.
- `nextPage` (integer | null): The page number of the next page.
- `prevPage` (integer | null): The page number of the previous page.
