# Google Authentication & JWT Token API Documentation

> **Base URL — Auth Server:** `http://localhost:8080/api/v1/auth`  
> **Base URL — Main Backend:** `http://localhost:8000/api/v1`  
> **Content-Type:** `application/json` (unless noted otherwise)  
> **All timestamps:** UTC unless otherwise specified

---

## Table of Contents

1. [Overview & Architecture](#1-overview--architecture)
2. [JWT Token Reference](#2-jwt-token-reference)
3. [Google Sign-In — Web (OAuth2 Redirect Flow)](#3-google-sign-in--web-oauth2-redirect-flow)
4. [Google Sign-In — Mobile (ID Token Flow)](#4-google-sign-in--mobile-id-token-flow)
5. [Google Sign-Up — New User Registration](#5-google-sign-up--new-user-registration)
6. [Refresh Access Token](#6-refresh-access-token)
7. [Logout](#7-logout)
8. [Error Reference](#8-error-reference)
9. [Security Notes](#9-security-notes)

---

## 1. Overview & Architecture

The authentication system is split across **two services**:

| Service | Port | Responsibility |
|---|---|---|
| **Auth Server** (`authserver`) | `8080` | Handles all OAuth flows, token issuance, OTP, and logout |
| **Main Backend** (`mulearnbackend`) | `8000` | Handles user registration, profile, resources |

### Google Auth Flow — High Level

```
Browser / Mobile App
      |
      |  1. GET /signin-with-google/?redirect_uri=...
      v
Auth Server ----> Returns Google OAuth2 authorization URL
      |
      |  2. User is redirected to Google login page
      v
Google Accounts
      |
      |  3. Google redirects back with ?code=...
      v
Auth Server /google/login/callback/
      |
      |-- Existing user --> Issues accessToken + refreshToken (Login complete)
      |
      +-- New user --> Issues tempToken (Sign-up required)
              |
              |  4. Frontend calls mulearnbackend /register/ with tempToken + profile data
              v
      mulearnbackend --> Verifies tempToken internally --> Creates user --> Issues JWT
```

---

## 2. JWT Token Reference

All JWTs are **HS256-signed** using the server's `SECRET_KEY`.

### 2.1 Access Token

Used to authenticate every protected API request. Pass it as a Bearer token.

```
Authorization: Bearer <accessToken>
```

#### Payload Structure

```json
{
  "id":        "550e8400-e29b-41d4-a716-446655440000",
  "muid":      "johndoe@mulearn",
  "roles":     ["Mulearner"],
  "expiry":    "2026-07-16 12:30:00+0000",
  "tokenType": "access",
  "iat":       1752652200
}
```

| Field | Type | Description |
|---|---|---|
| `id` | UUID string | User's database UUID |
| `muid` | string | Unique mulearn ID (e.g. `johndoe@mulearn`) |
| `roles` | string[] | User's assigned roles |
| `expiry` | string | ISO-format expiry timestamp (UTC) |
| `tokenType` | string | Always `"access"` |
| `iat` | int (Unix) | Issued-at timestamp — used for global logout checks |

#### Lifetime

| Property | Value |
|---|---|
| **Duration** | **15 minutes** |
| Algorithm | HS256 |
| Refresh strategy | Use the Refresh Token to get a new Access Token |

---

### 2.2 Refresh Token

Long-lived token used exclusively to obtain new Access Tokens via the `/get-access-token/` endpoint.

#### Payload Structure

```json
{
  "id":        "550e8400-e29b-41d4-a716-446655440000",
  "muid":      "johndoe@mulearn",
  "roles":     ["Mulearner"],
  "expiry":    "2026-07-23 12:00:00+0000",
  "tokenType": "refresh",
  "iat":       1752652200
}
```

| Field | Type | Description |
|---|---|---|
| `id` | UUID string | User's database UUID |
| `muid` | string | Unique mulearn ID |
| `roles` | string[] | User's roles at token issuance time |
| `expiry` | string | ISO-format expiry timestamp (UTC) |
| `tokenType` | string | Always `"refresh"` |
| `iat` | int (Unix) | Issued-at — compared against global logout timestamp in Redis |

#### Lifetime

| Property | Value |
|---|---|
| **Duration** | **7 days** |
| Algorithm | HS256 |
| Revocation | Global logout writes a timestamp to Redis; any token with `iat <= logout_time` is rejected |
| Redis TTL | 7 days (auto-expires with the token) |

---

### 2.3 Google Temporary Token (`tempToken`)

Short-lived, single-use token issued by the Auth Server for **new** Google users during sign-up.

#### Payload Structure

```json
{
  "email":     "user@gmail.com",
  "full_name": "John Doe",
  "tokenType": "google_signup",
  "jti":       "a1b2c3d4-uuid-here",
  "iat":       1752652200,
  "exp":       1752653100
}
```

| Field | Type | Description |
|---|---|---|
| `email` | string | Google-verified email (normalized: lowercased + stripped) |
| `full_name` | string | Google profile name |
| `tokenType` | string | Always `"google_signup"` |
| `jti` | UUID | Unique token ID for replay protection — consumed on first use |
| `iat` | int (Unix) | Issued-at |
| `exp` | int (Unix) | Expiry |

#### Lifetime

| Property | Value |
|---|---|
| **Duration** | **15 minutes** |
| Single-use | Yes — `jti` is stored in Django cache on first use; reuse returns 400 |
| Purpose | Transferred from frontend to mulearnbackend to prove Google ownership without DB write |

---

## 3. Google Sign-In — Web (OAuth2 Redirect Flow)

Two-step redirect-based flow for web browsers.

---

### Step 1 — Get Google Authorization URL

Obtain the URL to redirect the user to Google's login page.

**`GET /signin-with-google/`**
Auth Server

#### Query Parameters

| Parameter | Required | Description |
|---|---|---|
| `redirect_uri` | Yes | Frontend callback URL after Google authenticates the user |

#### Allowed `redirect_uri` Values

```
http://localhost:3000/callback/
https://dev.mulearn.org/callback/
https://mulearn-dashboard.vercel.app/callback/
https://app.mulearn.org/callback/
```

> The `redirect_uri` **must** match one of the above exactly. Any other value returns `400`.

#### Example Request

```http
GET /api/v1/auth/signin-with-google/?redirect_uri=http://localhost:3000/callback/
```

#### Success Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "redirect_url": "https://accounts.google.com/o/oauth2/auth?response_type=code&scope=email%20profile&client_id=<CLIENT_ID>&redirect_uri=http://localhost:3000/callback/"
  }
}
```

#### Error Responses

```json
// Missing redirect_uri
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["redirect_uri query parameter is required"] },
  "response": {}
}
```

```json
// Disallowed redirect_uri
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid redirect_uri"] },
  "response": {}
}
```

---

### Step 2 — Handle Google Callback

After Google redirects back to your frontend with `?code=...`, pass the code to the auth server.

**`GET /google/login/callback/`**
Auth Server

#### Query Parameters

| Parameter | Required | Description |
|---|---|---|
| `code` | Yes | Authorization code returned by Google |
| `redirect_uri` | Yes | Must exactly match the one used in Step 1 |

#### Example Request

```http
GET /api/v1/auth/google/login/callback/?code=4/0AX4XfWg...&redirect_uri=http://localhost:3000/callback/
```

---

#### Response A — Existing User (Login Success) — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Access Granted"]
  },
  "response": {
    "accessToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "isNewUser":    false
  }
}
```

| Field | Description |
|---|---|
| `accessToken` | JWT — valid for **15 minutes** |
| `refreshToken` | JWT — valid for **7 days** |
| `isNewUser` | `false` — user already exists and is now logged in |

---

#### Response B — New User (Sign-Up Required) — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["New user"]
  },
  "response": {
    "isNewUser":  true,
    "tempToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "email":      "user@gmail.com",
    "fullName":   "John Doe"
  }
}
```

| Field | Description |
|---|---|
| `isNewUser` | `true` — user does not exist, registration required |
| `tempToken` | Short-lived (15 min), single-use JWT — pass to `/register/` |
| `email` | For **UI pre-fill only** — email is enforced server-side from tempToken |
| `fullName` | For **UI pre-fill only** |

> **Next step:** Call the [Google Sign-Up endpoint](#5-google-sign-up--new-user-registration) with the `tempToken`.

---

#### Error Responses

```json
// Authorization code not provided
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Authorization code not provided"] },
  "response": {}
}
```

```json
// Google communication failure
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Failed to communicate with Google"] },
  "response": {}
}
```

```json
// No email returned from Google
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Could not retrieve email from Google"] },
  "response": {}
}
```

---

## 4. Google Sign-In — Mobile (ID Token Flow)

For Flutter / React Native apps that use the Google Sign-In SDK natively.
The SDK returns an **ID token** directly — no redirect flow needed.

**`POST /google-mobile/`**
Auth Server

#### Request Body

```json
{
  "id_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

> Accepts both `id_token` and `idToken` (camelCase alias).

| Field | Required | Description |
|---|---|---|
| `id_token` | Yes | ID token from Google Sign-In SDK |

#### Headers

```
Content-Type: application/json
```

---

#### Response A — Existing User (Login Success) — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Access Granted"]
  },
  "response": {
    "accessToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "isNewUser":    false,
    "user": {
      "muid":     "johndoe@mulearn",
      "email":    "user@gmail.com",
      "fullName": "John Doe"
    }
  }
}
```

---

#### Response B — New User (Sign-Up Required) — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["New user"]
  },
  "response": {
    "isNewUser":  true,
    "tempToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "email":      "user@gmail.com",
    "fullName":   "John Doe"
  }
}
```

---

#### Error Responses

```json
// Missing id_token
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["ID token is required"] },
  "response": {}
}
```

```json
// Invalid or expired ID token
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid or expired ID token"] },
  "response": {}
}
```

```json
// Email not verified with Google
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Email not verified with Google"] },
  "response": {}
}
```

```json
// Could not retrieve email from token
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Could not retrieve email from token"] },
  "response": {}
}
```

---

## 5. Google Sign-Up — New User Registration

Called by the **frontend** after receiving `isNewUser: true` and a `tempToken` from the Google callback/mobile endpoint.
This hits the **main backend** (`mulearnbackend`, port 8000).

**`POST /api/v1/register/`**
Main Backend

#### How it works internally

1. Frontend sends `tempToken` + user profile fields.
2. mulearnbackend calls Auth Server's internal `/google/verify-temp-token/` endpoint.
3. Auth Server verifies the JWT signature, checks the `jti` for single-use, and returns `{email, fullName}`.
4. mulearnbackend forces `email` from Google — the client **cannot override it**.
5. `full_name` can be edited by the user on the registration form.
6. No password is set — user logs in with Google exclusively going forward.
7. JWT tokens are issued via the internal token-verification endpoint.

#### Request Body

```json
{
  "tempToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "full_name": "John Doe",
    "dob":       "2000-05-14",
    "gender":    "Male",
    "district":  "3fa85f64-5717-4562-b3fc-2c963f66afa6"
  },
  "referral": {
    "muid": "referrer@mulearn"
  }
}
```

| Field | Required | Description |
|---|---|---|
| `tempToken` | Yes | Received from Google callback — 15 min TTL, single-use |
| `user.full_name` | Yes | Editable name — overrides Google's name if provided |
| `user.dob` | Yes | Date of birth in `YYYY-MM-DD` format |
| `user.gender` | Yes | `"Male"`, `"Female"`, or `"Other"` |
| `user.district` | Yes | UUID of the user's district |
| `user.email` | Ignored | **Forced from Google** — any client-supplied value is discarded |
| `user.password` | Ignored | Not set for Google users — `null` in the database |
| `referral.muid` | Optional | Referrer's mulearn ID |

#### Success Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "accessToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "data": {
      "id":        "550e8400-e29b-41d4-a716-446655440000",
      "muid":      "johndoe@mulearn",
      "email":     "user@gmail.com",
      "role":      null,
      "full_name": "John Doe"
    }
  }
}
```

| Field | Description |
|---|---|
| `accessToken` | JWT — valid for **15 minutes** |
| `refreshToken` | JWT — valid for **7 days** |
| `data.id` | User's UUID |
| `data.muid` | Auto-generated mulearn ID (e.g. `johndoe@mulearn`) |
| `data.email` | Google-verified and normalized email |
| `data.role` | Role title if assigned, `null` otherwise |

---

#### Error Responses

```json
// tempToken expired (more than 15 minutes old)
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Google session expired. Please sign in with Google again."] },
  "response": {}
}
```

```json
// tempToken already used (replay / double-submit)
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["This Google session was already used. Please sign in with Google again."] },
  "response": {}
}
```

```json
// Email already registered via password (classic signup path)
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["An account with this email already exists. Please sign in normally."] },
  "response": {}
}
```

```json
// Field validation error
{
  "hasError": true,
  "statusCode": 400,
  "message": {
    "general": [],
    "user": {
      "full_name": ["This field is required."]
    }
  },
  "response": {}
}
```

---

## 6. Refresh Access Token

Exchange a valid **Refresh Token** for a new **Access Token**.
The Refresh Token itself is returned unchanged in the response (sliding window not used).

**`POST /get-access-token/`**
Auth Server

#### Request Body

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

| Field | Required | Description |
|---|---|---|
| `refreshToken` | Yes | The user's current, valid refresh token |

#### Headers

```
Content-Type: application/json
```

---

#### Success Response — `200 OK`

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": []
  },
  "response": {
    "accessToken":  "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiry":       "2026-07-16 12:30:00+0000"
  }
}
```

| Field | Description |
|---|---|
| `accessToken` | New JWT — valid for **15 minutes** from now |
| `refreshToken` | The same refresh token that was submitted — unchanged |
| `expiry` | Expiry timestamp of the **new** access token (UTC string) |

---

#### Error Responses

```json
// Refresh token expired (older than 7 days)
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Refresh token has expired. Please log in again."] },
  "response": {}
}
```

```json
// Token invalidated by global logout
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Token has been invalidated. Please log in again."] },
  "response": {}
}
```

```json
// Wrong token type — access token submitted instead of refresh
{
  "hasError": true,
  "statusCode": 1003,
  "message": { "general": ["Invalid refresh token"] },
  "response": {}
}
```

```json
// Missing iat claim — old token format
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid token: missing issue time"] },
  "response": {}
}
```

```json
// User associated with the token no longer exists
{
  "hasError": true,
  "statusCode": 1004,
  "message": { "general": ["User invalid"] },
  "response": {}
}
```

```json
// Malformed or tampered token
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Signature verification failed"] },
  "response": {}
}
```

---

## 7. Logout

Invalidates **all sessions** for the authenticated user globally.  
Any refresh token issued **on or before** the logout timestamp will be permanently rejected when next used against `/get-access-token/`.

**`POST /logout/`**
Auth Server

> **How it works:**  
> The current Unix timestamp is written to Redis under the key `global_logout:<user_id>` with a **7-day TTL** (matching the maximum refresh token lifetime). Every call to `/get-access-token/` reads this key. If the incoming token's `iat <= logout_timestamp`, the request is rejected and the user must log in again.

#### Request Body

```json
{
  "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
}
```

| Field | Required | Description |
|---|---|---|
| `refreshToken` | Yes | The user's current refresh token — identifies the user and sets the invalidation boundary |

#### Headers

```
Content-Type: application/json
```

---

#### Success Response — `200 OK`

Returned in two cases:
- Logout succeeded — Redis key written successfully.
- The submitted token was already expired — treated as already logged out (idempotent).

```json
{
  "hasError": false,
  "statusCode": 200,
  "message": {
    "general": ["Logged out successfully"]
  },
  "response": {}
}
```

> **After receiving this response, the frontend must:**
> 1. Delete `accessToken` and `refreshToken` from local/secure storage.
> 2. Redirect the user to the login screen.

---

#### Error Responses

```json
// refreshToken field missing from request body
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["refreshToken is required"] },
  "response": {}
}
```

```json
// Access token submitted instead of refresh token
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["A refresh token is required to log out"] },
  "response": {}
}
```

```json
// Malformed or tampered token signature
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid token: Signature verification failed"] },
  "response": {}
}
```

```json
// Token payload does not contain a user ID
{
  "hasError": true,
  "statusCode": 400,
  "message": { "general": ["Invalid token payload"] },
  "response": {}
}
```

```json
// Redis is unavailable — logout cannot be persisted
// HTTP status: 503 Service Unavailable
{
  "hasError": true,
  "statusCode": 500,
  "message": { "general": ["Logout service temporarily unavailable. Please try again."] },
  "response": {}
}
```

---

## 8. Error Reference

### Standard Response Envelope

All endpoints return the same response envelope:

```json
{
  "hasError":   true,
  "statusCode": 400,
  "message": {
    "general": ["Human-readable error message"]
  },
  "response": {}
}
```

### Common Status Codes

| `statusCode` | HTTP Status | Meaning |
|---|---|---|
| `200` | `200 OK` | Success |
| `400` | `400 Bad Request` | Validation error or bad input |
| `1003` | `400 Bad Request` | Wrong token type (access token used where refresh expected) |
| `1004` | `400 Bad Request` | User not found for the given token |
| `500` | `503 Service Unavailable` | Redis or downstream service unavailable |

---

## 9. Security Notes

| ID | Concern | Implementation |
|---|---|---|
| B1 | Token integrity | All JWTs verified with `HS256` + `SECRET_KEY`; body `statusCode` checked in addition to HTTP status |
| B3 | Orphaned user recovery | If a Google sign-up failed mid-way leaving a null-password user, the system detects it and re-issues tokens rather than creating a duplicate |
| B4 | Auth service timeouts | mulearnbackend calls to authserver use `(3s connect, 10s read)` timeouts to prevent hangs |
| B5 | Email normalization | Google emails are always `.strip().lower()` before DB lookup to prevent case-mismatch duplicates |
| B6 | `tempToken` replay protection | Each `tempToken` contains a unique `jti`; marked as used in Django cache (15-min TTL) on first consume — reuse returns 400 |
| GL | Global logout | Current Unix timestamp written to Redis on logout; all tokens with `iat <= timestamp` are rejected by `/get-access-token/` |
| RD | Redis key auto-cleanup | Global logout key TTL is 7 days — matches the refresh token lifetime so the key cleans itself up |

---
