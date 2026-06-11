# API Documentation: Admin Assign User Role

## Endpoint Overview
`POST /api/v1/dashboard/roles/user-role/`

This endpoint is used by Admins from the dashboard profile panel to assign a specific role to a user. Depending on the role assigned (e.g., General, Intern, or Mentor), the endpoint automatically creates all the required underlying database side-effects in an atomic transaction.

### Headers Required
- `Authorization: Bearer <token>`

---

## 1. Assigning a General Role
Use this for roles that do not require any specialized sub-tiers or scoping (e.g., Campus Lead, Admin).

**Request Body:**
```json
{
  "user_id": "<uuid>",
  "role_id": "<role-uuid>"
}
```

**Response:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Role Added Successfully"]
  },
  "response": {
    "message": "Role Added Successfully"
  }
}
```

---

## 2. Assigning the Intern Role
When assigning the `Intern` role, the system automatically registers the user into the Intern system by creating or reactivating a `UserInternGuildLink`.

**Request Body:**
```json
{
  "user_id": "<uuid>",
  "role_id": "<intern-role-uuid>",
  "guild": "Backend Guild"
}
```
*Note: `guild` is strictly required when `role_id` corresponds to the Intern role.*

**Response:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Role Added Successfully"]
  },
  "response": {
    "message": "Role Added Successfully",
    "intern_guild_created": true 
  }
}
```
*(If `intern_guild_created` is `false`, it means the intern already existed and their profile was reactivated/updated.)*

---

## 3. Assigning the Mentor Role
When assigning the `Mentor` role, the system automatically creates the `UserMentor` profile and attaches the user to the correct organizations or interest groups based on their tier.

### A. General Platform Mentor
```json
{
  "user_id": "<uuid>",
  "role_id": "<mentor-role-uuid>",
  "mentor_tier": "MENTOR"
}
```

### B. Interest Group (IG) Mentor
Requires a list of IG UUIDs. The system automatically creates active `UserIgLink` assignments.
```json
{
  "user_id": "<uuid>",
  "role_id": "<mentor-role-uuid>",
  "mentor_tier": "IG_MENTOR",
  "ig_ids": ["<ig-uuid-1>", "<ig-uuid-2>"]
}
```

### C. Campus or Company Mentor
Requires the specific College or Company UUID. The system automatically creates a verified `UserOrganizationLink`.
```json
{
  "user_id": "<uuid>",
  "role_id": "<mentor-role-uuid>",
  "mentor_tier": "CAMPUS_MENTOR",  // or "COMPANY_MENTOR"
  "org_id": "<college-or-company-uuid>"
}
```

**Response:**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Role Added Successfully"]
  },
  "response": {
    "message": "Role Added Successfully",
    "mentor_profile_created": true
  }
}
```
*(If `mentor_profile_created` is `false`, the user was already a mentor, and the system simply approved their existing mentor application and added any missing links).*

---

## Error Handling

If required tier-specific fields are missing, the endpoint will return a 400 Bad Request with a descriptive error. For example:

**Missing Guild for Intern:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "guild": ["guild is required when assigning the Intern role."]
  }
}
```

**Missing Org for Campus Mentor:**
```json
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "org_id": ["org_id is required for CAMPUS_MENTOR."]
  }
}
```
