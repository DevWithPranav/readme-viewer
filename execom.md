# Postman Testing Guide: Dynamic Execom Roles

This document outlines how to manually test the new API functionalities using Postman. Make sure you have your local server running (`python manage.py runserver`).

## Prerequisites
- A valid JWT Access Token for a user who holds the **Campus Lead** role.
- Set `{{base_url}}` in your Postman environment to your local server url (e.g., `http://127.0.0.1:8000`).
- Set `{{jwt_token}}` to your Campus Lead's access token.

All endpoints require the following headers:
- `Content-Type`: `application/json`
- `Authorization`: `Bearer {{jwt_token}}`

---

## 1. Get All Assignable Roles (GET)

Test whether the system properly collates standard roles, active Interest Group roles for your campus, and any globally created custom roles.

**Method**: `GET`
**Endpoint**: `{{base_url}}/api/v1/dashboard/campus/execom/roles/`
*(Note: If your local URL structure differs, ensure you include the correct prefix. Usually it is `/api/v1/dashboard/campus/execom/roles/`)*

**Expected Response**:
```json
{
    "hasError": false,
    "statusCode": 200,
    "response": {
        "data": [
            "Campus Lead",
            "Enabler",
            "Lead Enabler",
            "Web Design Lead",  // (Example if the Web IG is active in your campus)
            "Custom Event Coordinator" // (Example of a global custom role)
        ]
    }
}
```

---

## 2. Create a Custom Role Explicitly (POST)

Test explicitly registering a new customized role into the global system (which is what happens if a Campus Lead uses the "Other" option to add a new role).

**Method**: `POST`
**Endpoint**: `{{base_url}}/api/v1/dashboard/campus/execom/roles/`

**Body (raw JSON)**:
```json
{
    "role_title": "Fun Manager"
}
```

**Expected Response**:
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": ["Role created successfully"]
    }
}
```
*Try hitting the `GET` endpoint again—you should now see `"Fun Manager"` in the list varying alphabetically.*

---

## 3. Test Security Blacklist (POST)
Test attempting to create a highly privileged system admin role to ensure our security check triggers properly.

**Method**: `POST`
**Endpoint**: `{{base_url}}/api/v1/dashboard/campus/execom/roles/`

**Body (raw JSON)**:
```json
{
    "role_title": "Admin"
}
```

**Expected Error Response (400 or similar failure)**:
```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "general": ["Cannot create highly privileged system role: Admin"]
    }
}
```

---

## 4. Assign a Role (POST execom)

Test the main assignment API using your newly created custom role.

**Method**: `POST`
**Endpoint**: `{{base_url}}/api/v1/dashboard/campus/execom/`

**Body (raw JSON)**:
```json
{
    "muid": "targetuser@mulearn",
    "role_title": "Fun Manager"
}
```

**Expected Response**:
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": ["Role assigned successfully"]
    }
}
```

---

## 5. Test Interest Group (IG) Validation

Test assigning an Interest Group role for an IG that you **know** is completely inactive or does not exist on your specific campus.

**Method**: `POST`
**Endpoint**: `{{base_url}}/api/v1/dashboard/campus/execom/`

**Body (raw JSON)**:
```json
{
    "muid": "targetuser@mulearn",
    "role_title": "FakeIGName Design Lead"
}
```

**Expected Error Response**:
```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "general": ["The Interest Group for this role is not active in your campus."]
    }
}
```
