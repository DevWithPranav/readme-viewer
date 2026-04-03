# Dashboard / Events API (Full)
Base path: `/api/v1/dashboard/events/`

## Authentication
- Header: `Authorization: Bearer <JWT>`
- JWT must include `id` (user id) and `roles` (array of role strings).
- Admin-only endpoints require the `roles` array to include `RoleType.ADMIN.value` which is `"Admins"`.

## Common Response Envelope
All endpoints return the same top-level JSON keys.

### Success (non-paginated)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["<success message>"] },
  "response": { }
}
```

### Success (paginated)
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": ["<success message>"] },
  "response": {
    "data": [ /* items */ ],
    "pagination": {
      "count": 0,
      "totalPages": 0,
      "isNext": false,
      "isPrev": false,
      "nextPage": null
    }
  }
}
```

### Failure
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["<error message or validation object>"] },
  "response": { }
}
```

## Common Pagination Query Params (used by list endpoints)
These query params are supported by `CommonUtils.get_paginated_queryset`:
- `pageIndex` (int, default `1`)
- `perPage` (int, default `10`)
- `search` (string, substring match on endpoint-specific fields)
- `sortBy` (string, optional, can be prefixed with `-` for descending)

## Event Status / Enums
### `Event.status` values
`draft`, `pending_campus_approval`, `pending_approval`, `pending_mentor_approval`, `published`, `ongoing`, `completed`, `cancelled`

### `Event.scope` values
`global`, `campus`, `ig`, `campus_ig`, `company`

### `Event.venue_type` values
`physical`, `online`, `hybrid`

### `Event.organiser_type` values
`global_ig`, `campus_ig`, `campus`, `company`, `admin`

## Shared Payload Schemas

### `EventVenue` (nested)
```json
{
  "type": "physical|online|hybrid",
  "address": "string|null",
  "city": "string|null",
  "maps_url": "string|null",
  "online_link": "string|null",
  "platform": "string|null"
}
```

### `OrganizerInfo` (nested)
```json
{
  "type": "global_ig|campus_ig|campus|company|admin",
  "ig": { "id": "uuid", "name": "string", "icon": "string" } | null,
  "campus": { "id": "uuid", "title": "string", "org_type": "College|School" } | null,
  "company": { "id": "uuid", "title": "string", "org_type": "College|School" } | null,
  "campus_ig_id": "uuid|null"
}
```

### `EventListItem` (used by list/feed endpoints)
```json
{
  "id": "uuid",
  "title": "string",
  "slug": "string",
  "cover_image": "string|null",
  "status": "draft|pending_campus_approval|pending_approval|pending_mentor_approval|published|ongoing|completed|cancelled",
  "scope": "global|campus|ig|campus_ig|company",
  "start_datetime": "ISO-8601 datetime string",
  "end_datetime": "ISO-8601 datetime string",
  "venue": { /* EventVenue */ },
  "organizer": { /* OrganizerInfo */ },
  "is_featured": true,
  "is_collaboration": true,
  "interest_count": 0,
  "min_karma": 0,
  "tags": [/* JSON */] ,
  "user_limit": 0,
  "category_name": "string|null",
  "viewer_interest_status": "interested|none|null"
}
```

### `LinkedTask` (nested inside `EventDetail`)
```json
{
  "id": "uuid",
  "title": "string",
  "description": "string",
  "hashtag": "string|null",
  "karma": 0,
  "bonus_time": 0,
  "bonus_karma": 0,
  "active": true,
  "ig": { "id": "uuid", "name": "string", "icon": "string" }
}
```

### `EventCoOwner` (nested)
```json
{
  "id": "uuid",
  "entity_id": "uuid",
  "user": { "id": "uuid", "full_name": "string", "muid": "string", "profile_pic": "string|null" },
  "added_by": { "id": "uuid", "full_name": "string", "muid": "string", "profile_pic": "string|null" },
  "added_at": "ISO-8601 datetime string"
}
```

### `EventCollaborator` (nested)
```json
{
  "id": "uuid",
  "entity_type": "collab_ig|collab_campus|collab_campus_ig|collab_company",
  "entity_id": "uuid",
  "entity_detail": {
    "id": "uuid",
    "name": "string",
    "icon": "string",
    "code": "string"
  } | { "id": "uuid", "title": "string", "org_type": "College|School" } | { "campus_ig_id": "uuid" } | null,
  "role_label": "string|null",
  "invite_status": "pending|accepted|rejected|null",
  "rejection_reason": "string|null",
  "responded_at": "ISO-8601 datetime string|null",
  "created_at": "ISO-8601 datetime string"
}
```

### `EventLog` (used for `edit_history` in manage detail)
```json
{
  "id": "uuid",
  "edited_by": { "id": "uuid", "full_name": "string", "muid": "string", "profile_pic": "string|null" },
  "changed_fields": { } | [ ],
  "edited_at": "ISO-8601 datetime string"
}
```

### `EventDetail` (full event object in responses)
```json
{
  "id": "uuid",
  "title": "string",
  "slug": "string",
  "description": "string|null",
  "cover_image": "string|null",
  "banner_image": "string|null",
  "category_name": "string|null",
  "status": "draft|pending_campus_approval|pending_approval|pending_mentor_approval|published|ongoing|completed|cancelled",
  "scope": "global|campus|ig|campus_ig|company",
  "scope_org": { "id": "uuid", "title": "string", "org_type": "College|School" } | null,
  "scope_ig": { "id": "uuid", "name": "string", "icon": "string" } | null,
  "scope_ci_id": "uuid|null",
  "organizer": { /* OrganizerInfo */ },
  "venue": { /* EventVenue */ },
  "start_datetime": "ISO-8601 datetime string",
  "end_datetime": "ISO-8601 datetime string",
  "registration_url": "string|null",
  "registration_deadline": "ISO-8601 datetime string|null",
  "min_karma": 0,
  "is_featured": true,
  "is_collaboration": true,
  "interest_count": 0,
  "tags": [/* JSON */] | null,
  "user_limit": 0,
  "linked_tasks": [ /* LinkedTask */ ],
  "co_owners": [ /* EventCoOwner */ ],
  "collaborators": [ /* EventCollaborator */ ],
  "viewer_interest_status": "interested|none|null",
  "viewer_can_access_registration": true|false,
  "viewer_access_blocked_reason": "string|null",
  "created_by": { "id": "uuid", "full_name": "string", "muid": "string", "profile_pic": "string|null" },
  "updated_by": { "id": "uuid", "full_name": "string", "muid": "string", "profile_pic": "string|null" },
  "created_at": "ISO-8601 datetime string",
  "updated_at": "ISO-8601 datetime string",
  "edit_history": [ /* EventLog */ ]  /* only in manage GET */
}
```

### `EventWriteRequest` (request body for create/update)
This matches `EventWriteSerializer` (fields accepted by the API).
```json
{
  "title": "string",
  "description": "string|null",
  "cover_image": "string|null",
  "banner_image": "string|null",
  "category": "uuid|null",
  "start_datetime": "ISO-8601 datetime string",
  "end_datetime": "ISO-8601 datetime string",
  "registration_url": "string|null",
  "registration_deadline": "ISO-8601 datetime string|null",
  "min_karma": 0,
  "venue_type": "physical|online|hybrid",
  "venue_address": "string|null",
  "venue_city": "string|null",
  "venue_maps_url": "string|null",
  "venue_online_link": "string|null",
  "venue_platform": "string|null",
  "scope": "global|campus|ig|campus_ig|company",
  "scope_org": "uuid|null",
  "scope_ig": "uuid|null",
  "scope_ci_id": "uuid|null",
  "organiser_type": "global_ig|campus_ig|campus|company|admin",
  "organiser_ig": "uuid|null",
  "organiser_org": "uuid|null",
  "organiser_ci_id": "uuid|null",
  "is_collaboration": true,
  "is_featured": true,
  "tags": [/* JSON */] | null,
  "user_limit": 0
}
```

Notes:
- Server generates `id`, `slug`, `created_by`, `updated_by`.
- Validation: `end_datetime` must be greater than `start_datetime` (when both are provided).

## Public Events (no role requirement)

### GET `/api/v1/dashboard/events/`
Description: Public event feed (visible by scope rules). Default returns `published` + `ongoing` only.
Permissions: Requires valid JWT header; visibility is based on the viewer.
Request query params:
- `pageIndex`, `perPage`, `search`, `sortBy`
- `event_type` (string, matches `organiser_type`)
- `ig_id` (uuid, matches `scope_ig_id`)
- `campus_id` (uuid, matches `scope_org_id`)
- `cluster` (string, matches organiser IG category)
- `is_featured` (boolean as string: `"true"` / `"false"`)
- `start_date` (date string, compares `start_datetime__date__gte`)
- `end_date` (date string, compares `end_datetime__date__lte`)
- `tags` (string; matched against JSON field)
- `eligible_only` (when `"true"` and authenticated: filters by `min_karma` <= viewer karma)
Request body: none
Response body:
- Success envelope with paginated `response.data` as `EventListItem[]`

### GET `/api/v1/dashboard/events/featured/`
Description: Featured events (top 20), only `published` and `is_featured=true`.
Permissions: JWT not required by view logic, but endpoint uses no `CustomizePermission`.
Request body: none
Response body:
- Success envelope where `response` is an array of `EventListItem`
Example (shape):
```json
{ "response": [ /* EventListItem */ ] }
```

### GET `/api/v1/dashboard/events/<str:event_id>/`
Description: Full event detail. Draft/pending are visible only to organiser/admin.
Permissions: Requires valid JWT header; otherwise you get “Event not found” for non-public statuses.
Request body: none
Response body:
- Success envelope with `response` as `EventDetail`

### POST `/api/v1/dashboard/events/<str:event_id>/interest/`
Description: Express “I’m Going”.
Request body: none
Response body:
- Success envelope with:
```json
{
  "event_id": "uuid",
  "user_id": "uuid",
  "status": "interested"
}
```

### DELETE `/api/v1/dashboard/events/<str:event_id>/interest/`
Description: Remove expressed interest.
Request body: none
Response body:
- Success envelope with `response: {}` and message `Your interest has been removed.`

## Scoped Feeds (published + ongoing only)

### GET `/api/v1/dashboard/events/ig/<str:ig_id>/`
Description: Events organised/scoped to an IG.
Request body: none
Request query params: `pageIndex`, `perPage`, `search`, `sortBy`
Response: paginated `EventListItem[]`

### GET `/api/v1/dashboard/events/ig/cluster/<str:cluster>/`
Description: Events from IGs in a given cluster (`InterestGroup.category`).
Request body: none
Request query params: `pageIndex`, `perPage`, `search`, `sortBy`
Response: paginated `EventListItem[]`

### GET `/api/v1/dashboard/events/campus/<str:campus_id>/`
Description: Events scoped to or organised by a campus. Optional `?cluster=` filter.
Request query params:
- `cluster` (optional)
- `pageIndex`, `perPage`, `search`, `sortBy`
Request body: none
Response: paginated `EventListItem[]`

### GET `/api/v1/dashboard/events/campus-ig/<str:campus_ig_id>/`
Description: Events created by a specific campus IG chapter.
Request body: none
Request query params: `pageIndex`, `perPage`, `search`, `sortBy`
Response: paginated `EventListItem[]`

### GET `/api/v1/dashboard/events/company/<str:company_id>/`
Description: Events organised by a company partner.
Request body: none
Request query params: `pageIndex`, `perPage`, `search`, `sortBy`
Response: paginated `EventListItem[]`

## Manage Events (organiser/co-owner or Admin)

### GET `/api/v1/dashboard/events/manage/`
Description: List events the caller can manage.
Permissions:
- Admin users can manage all events excluding `draft`.
- Non-admin users see events they created or where they are a co-owner.
Request query params:
- optional `status` (matches `Event.status`)
- `pageIndex`, `perPage`, `search`, `sortBy`
Request body: none
Response body:
- paginated `EventListItem[]`

### POST `/api/v1/dashboard/events/manage/`
Description: Create a new event.
Permissions: Caller must have at least one “event creation” role (`Admin`, `Campus Lead`, `IG Lead`, etc.).
Request body: `EventWriteRequest`
Response body:
- Success envelope with `response` as `EventDetail`

### GET `/api/v1/dashboard/events/manage/<str:event_id>/`
Description: Full event detail + edit history.
Request body: none
Response body:
- Success envelope with `response` as `EventDetail` plus `edit_history: EventLog[]`

### PUT `/api/v1/dashboard/events/manage/<str:event_id>/`
Description: Full update of an event (partial=False in serializer).
Request body: `EventWriteRequest`
Response body:
- Success envelope with `response` as `EventDetail`

### PATCH `/api/v1/dashboard/events/manage/<str:event_id>/`
Description: Partial update of an event (partial=True in serializer).
Request body: `EventWriteRequest` (send only fields you want to update)
Response body:
- Success envelope with `response` as `EventDetail`

### DELETE `/api/v1/dashboard/events/manage/<str:event_id>/`
Description: Soft cancel (sets `status=cancelled`, sets `deleted_at`).
Request body: none
Response body:
```json
{
  "id": "uuid",
  "status": "cancelled"
}
```

### POST `/api/v1/dashboard/events/manage/<str:event_id>/publish/`
Description: Publish draft into approval pipeline. It transitions the existing event status based on `organiser_type`.
Request body: none
Response body:
```json
{ "id": "uuid", "status": "published|pending_campus_approval|pending_approval|pending_mentor_approval" }
```

### GET `/api/v1/dashboard/events/manage/<str:event_id>/co-owners/`
Description: Retrieve co-owners for an event.
Permissions: organiser/co-owner or Admin.
Request body: none
Response body:
- Success envelope where `response` is an array of `EventCoOwner`:
```json
{
  "response": [
    { /* EventCoOwner */ }
  ]
}
```

### POST `/api/v1/dashboard/events/manage/<str:event_id>/co-owners/`
Description: Add a co-owner.
Permissions: organiser/co-owner or Admin.
Request body:
```json
{ "user_id": "uuid" }
```
Response body:
```json
{ /* EventCoOwner */ }
```

### DELETE `/api/v1/dashboard/events/manage/<str:event_id>/co-owners/<str:co_owner_id>/`
Description: Remove a co-owner row (by `EventConnection` id).
Permissions: organiser/co-owner or Admin.
Request body: none
Response body: `{}` (success envelope with message `Co-owner removed.`)

### GET `/api/v1/dashboard/events/manage/<str:event_id>/collaborators/`
Description: Retrieve collaborator invites for an event (incl. pending/rejected for manage views).
Permissions: organiser/co-owner or Admin.
Request body: none
Response body: `response: EventCollaborator[]`

### POST `/api/v1/dashboard/events/manage/<str:event_id>/collaborators/`
Description: Invite a collaborator entity (IG, campus, campus_ig, or company).
Permissions: organiser/co-owner or Admin.
Request body:
```json
{
  "entity_type": "collab_ig|collab_campus|collab_campus_ig|collab_company",
  "entity_id": "uuid",
  "role_label": "string"
}
```
Response body:
```json
{ /* EventCollaborator */ }
```

### POST `/api/v1/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/accept/`
Description: Accept a collaborator invite (only authorised lead can accept).
Permissions: lead for the invited entity.
Request body: none
Response body:
```json
{ /* EventCollaborator */ }
```

### POST `/api/v1/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/reject/`
Description: Reject a collaborator invite (only authorised lead can reject).
Request body:
```json
{ "reason": "string" }
```
Response body:
```json
{ /* EventCollaborator */ }
```

### DELETE `/api/v1/dashboard/events/manage/<str:event_id>/collaborators/<str:collaborator_id>/`
Description: Remove a collaborator invite/row.
Permissions: organiser/co-owner or Admin.
Request body: none
Response body: `{}` (success envelope with message `Collaborator removed.`)

## Admin Events (Admins only)

### GET `/api/v1/dashboard/events/admin/`
Description: Admin listing of all events excluding `draft` (all other statuses incl. cancelled/completed).
Permissions: Admin only.
Request query params:
- optional `status`
- optional `organiser_type`
- optional `created_by`
- optional `scope`
- optional `is_featured` (boolean as string: `"true"` / `"false"`)
- `pageIndex`, `perPage`, `search`, `sortBy`
Request body: none
Response body:
- paginated `EventListItem[]`

### POST `/api/v1/dashboard/events/admin/<str:event_id>/approve/`
Description: Approve a pending event and move it forward in the approval pipeline.
Permissions: Admin only.
Request body: none
Response body:
```json
{ "id": "uuid", "status": "published|pending_approval" }
```

### POST `/api/v1/dashboard/events/admin/<str:event_id>/reject/`
Description: Reject a pending event and return it to `draft`.
Permissions: Admin only.
Request body:
```json
{ "reason": "string" }
```
Response body:
```json
{ "id": "uuid", "status": "draft", "reason": "string" }
```

### PATCH `/api/v1/dashboard/events/admin/<str:event_id>/feature/`
Description: Toggle or set `is_featured`.
Permissions: Admin only.
Request body (optional key):
```json
{ "is_featured": true }
```
If `is_featured` is omitted, the API toggles the current value.
Response body:
```json
{ "id": "uuid", "is_featured": true|false }
```

## Events Meta (form options)

### GET `/api/v1/dashboard/events/meta/organizer-options/`
Description: Returns organiser contexts the caller is authorised to create events as.
Request body: none
Response body:
```json
{
  "global_ig": [ { "id": "uuid", "name": "string", "icon": "string", "code": "string" } ],
  "campus_ig": [ { "id": "uuid", "name": "string", "icon": "string", "code": "string" } ],
  "campus": [ { "id": "uuid", "title": "string", "org_type": "College|School" } ],
  "company": [ { "id": "uuid", "title": "string" } ],
  "admin": true
}
```

### GET `/api/v1/dashboard/events/meta/collaboration-targets/`
Description: Search entities that can be invited as collaborators.
Request query params:
- `search` (string, optional)
- `type` (string, optional; `ig|campus|campus_ig|company`)
Request body: none
Response body:
```json
{
  "ig": [ { "id": "uuid", "name": "string", "icon": "string", "code": "string" } ],
  "campus": [ { "id": "uuid", "title": "string", "org_type": "College|School" } ],
  "company": [ { "id": "uuid", "title": "string", "org_type": "College|School" } ],
  "campus_ig": [ { "id": "uuid", "name": "string", "icon": "string", "code": "string" } ]
}
```

