# Google Authentication API Documentation

**Base URL:** `http://127.0.0.1:8000/api/v1/auth/`

This document covers all Google Sign-In endpoints exposed by the authentication server. There are **two distinct integration flows**:

| Flow | Use Case | Endpoints |
|---|---|---|
| **Web OAuth2 (Redirect)** | Web browsers, web apps | `signin-with-google/` → `google/login/callback/` |
| **Mobile Token-based** | Flutter, React Native, Android, iOS | `google-mobile/` |

---

## Table of Contents

1. [Overview & Architecture](#1-overview--architecture)
2. [Web OAuth2 Flow](#2-web-oauth2-flow)
   - [Step 1 — Get Google Auth URL](#21-step-1--get-google-auth-url)
   - [Step 2 — OAuth2 Callback](#22-step-2--oauth2-callback)
3. [Mobile Token-Based Flow](#3-mobile-token-based-flow)
4. [Response Format](#4-response-format)
5. [JWT Token Details](#5-jwt-token-details)
6. [Error Reference](#6-error-reference)
7. [Login Attempt Logging](#7-login-attempt-logging)
8. [Integration Examples](#8-integration-examples)
9. [Environment Configuration](#9-environment-configuration)
10. [Security Considerations](#10-security-considerations)

---

## 1. Overview & Architecture

The authentication server supports two ways of authenticating a user via Google:

### Web Flow (OAuth2 Authorization Code)
```
Client (Browser)
    │
    ├─► GET /signin-with-google/          ← Get redirect URL
    │        └─► Response: { redirect_url: "https://accounts.google.com/o/oauth2/auth?..." }
    │
    ├─► Redirect user to Google (browser opens the URL)
    │        └─► User logs in & grants consent on Google's page
    │
    ├─► Google redirects back to:
    │        GET /google/login/callback/?code=AUTH_CODE
    │
    └─► Response: { accessToken, refreshToken }
```

### Mobile Flow (ID Token Verification)
```
Mobile App
    │
    ├─► User signs in via Google SDK (GoogleSignIn package)
    │        └─► SDK returns an id_token
    │
    ├─► POST /google-mobile/  { "id_token": "..." }
    │        └─► Server verifies token with Google tokeninfo endpoint
    │
    └─► Response: { accessToken, refreshToken, user: { muid, email, fullName } }
```

> **Important:** Both flows are **sign-in only** — they do not create new accounts. A user must already have a registered account with a matching email address.

---

## 2. Web OAuth2 Flow

### 2.1 Step 1 — Get Google Auth URL

Initiate the web-based Google OAuth2 flow by fetching the Google authorization URL.

```
GET /api/v1/auth/signin-with-google/
```

#### Request

| Parameter | Type | Required | Description |
|---|---|---|---|
| *(none)* | — | — | No body or query params needed |

**Headers:**
```
Content-Type: application/json
```

#### Success Response

**Status:** `200 OK`

```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": []
    },
    "response": {
        "redirect_url": "https://accounts.google.com/o/oauth2/auth?response_type=code&scope=email profile&client_id=<YOUR_CLIENT_ID>&redirect_uri=http://127.0.0.1:8000/api/v1/auth/google/login/callback/"
    }
}
```

#### How to Use

After receiving the `redirect_url`, redirect the user's browser to it:

```javascript
// Frontend JavaScript
const response = await fetch('/api/v1/auth/signin-with-google/');
const data = await response.json();
window.location.href = data.response.redirect_url;
```

The URL opens Google's consent screen and requests these **OAuth2 scopes**:
- `email` — user's email address
- `profile` — user's basic profile information

---

### 2.2 Step 2 — OAuth2 Callback

This endpoint is called **automatically by Google** after the user grants consent. Your frontend does not call this directly — the browser is redirected here with an authorization code.

```
GET /api/v1/auth/google/login/callback/?code=AUTH_CODE
```

#### Query Parameters (sent by Google)

| Parameter | Type | Description |
|---|---|---|
| `code` | `string` | Authorization code issued by Google. Short-lived, single-use. |

#### Internal Flow

1. Receives the `code` from Google's redirect.
2. Exchanges `code` for an **access token** via `POST https://accounts.google.com/o/oauth2/token`.
3. Uses the access token to fetch user info from `https://www.googleapis.com/oauth2/v3/userinfo`.
4. Looks up the user by email in the database.
5. Generates and returns JWT tokens.

#### Success Response

**Status:** `200 OK`

```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": ["Access Granted"]
    },
    "response": {
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
    }
}
```

#### Failure Responses

| Scenario | HTTP Status | `statusCode` | `message.general` |
|---|---|---|---|
| `code` not present in query | `400` | `400` | `"Authorization code not provided"` |
| Google token exchange fails | `400` | `400` | `"Failed to obtain access token from Google"` |
| Google API unreachable | `400` | `400` | `"Failed to communicate with Google"` |
| Email not in Google response | `400` | `400` | `"Could not retrieve email from Google"` |
| No account with this email | `400` | `400` | `"No account found with this email"` |

---

## 3. Mobile Token-Based Flow

For Flutter, React Native, Android, or iOS applications. The mobile app uses the **Google Sign-In SDK** to get an `id_token` and sends it directly to this endpoint.

```
POST /api/v1/auth/google-mobile/
```

#### Request

**Headers:**
```
Content-Type: application/json
```

**Body** (either key works):
```json
{
    "id_token": "eyJhbGciOiJSUzI1NiIsImtpZCI6Ij..."
}
```
or
```json
{
    "idToken": "eyJhbGciOiJSUzI1NiIsImtpZCI6Ij..."
}
```

| Field | Type | Required | Description |
|---|---|---|---|
| `id_token` | `string` | Yes* | ID token from Google Sign-In SDK |
| `idToken` | `string` | Yes* | Alias — camelCase variant also accepted |

> *One of `id_token` or `idToken` must be provided.

#### Internal Flow

1. Accepts `id_token` (or `idToken`) from the mobile client.
2. Sends the token to Google's **tokeninfo endpoint**:
   ```
   GET https://oauth2.googleapis.com/tokeninfo?id_token=<TOKEN>
   ```
3. Validates the HTTP response from Google (must be `200 OK`).
4. Checks that the token contains a verified email (`email_verified == "true"`).
5. Looks up the user by email in the database.
6. Returns JWT tokens plus basic user info.

#### Success Response

**Status:** `200 OK`

```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": ["Access Granted"]
    },
    "response": {
        "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
        "user": {
            "muid": "johndoe@mulearn",
            "email": "johndoe@gmail.com",
            "fullName": "John Doe"
        }
    }
}
```

> **Note:** The mobile endpoint returns extra `user` info (`muid`, `email`, `fullName`) in the response, unlike the web callback which only returns tokens.

#### Failure Responses

| Scenario | HTTP Status | `statusCode` | `message.general` |
|---|---|---|---|
| `id_token` not provided | `400` | `400` | `"ID token is required"` |
| Google returns non-200 | `400` | `400` | `"Invalid or expired ID token"` |
| Email not found in token | `400` | `400` | `"Could not retrieve email from token"` |
| Email not verified by Google | `400` | `400` | `"Email not verified with Google"` |
| Google API unreachable | `400` | `400` | `"Failed to verify token with Google"` |
| No account with this email | `400` | `400` | `"No account found with this email"` |

---

## 4. Response Format

All API responses follow this envelope structure:

```json
{
    "hasError": false | true,
    "statusCode": 200 | 400,
    "message": {
        "general": ["...message string..."]
    },
    "response": { ... }
}
```

| Field | Type | Description |
|---|---|---|
| `hasError` | `boolean` | `false` for success, `true` for errors |
| `statusCode` | `integer` | `200` for success, `400` for client errors |
| `message.general` | `string[]` | Human-readable message array |
| `response` | `object` | The actual payload data |

---

## 5. JWT Token Details

Both endpoints return two tokens on success:

### Access Token

- **Algorithm:** `HS256`
- **Expiry:** 3 hours (`10800` seconds) from issue time
- **Payload:**

```json
{
    "id": "uuid-of-the-user",
    "muid": "johndoe@mulearn",
    "roles": ["Admin", "Member"],
    "expiry": "2026-04-25 21:00:00+0000",
    "tokenType": "access"
}
```

### Refresh Token

- **Algorithm:** `HS256`
- **Expiry:** No hard expiry (long-lived)
- **Payload:**

```json
{
    "id": "uuid-of-the-user",
    "muid": "johndoe@mulearn",
    "roles": ["Admin", "Member"],
    "tokenType": "refresh"
}
```

### Refreshing the Access Token

Use the refresh token to get a new access token:

```
POST /api/v1/auth/get-access-token/
Content-Type: application/json

{
    "refreshToken": "<your_refresh_token>"
}
```

**Success Response:**
```json
{
    "hasError": false,
    "statusCode": 200,
    "message": { "general": [] },
    "response": {
        "accessToken": "eyJ...",
        "refreshToken": "eyJ...",
        "expiry": "2026-04-25 21:00:00+0000"
    }
}
```

---

## 6. Error Reference

### Complete Error Table — Google Endpoints

| Endpoint | Error Condition | HTTP | Response Message |
|---|---|---|---|
| `signin-with-google/` | *(no failure cases — just returns redirect URL)* | `200` | — |
| `google/login/callback/` | Missing `code` param | `400` | `"Authorization code not provided"` |
| `google/login/callback/` | Token exchange with Google failed | `400` | `"Failed to obtain access token from Google"` |
| `google/login/callback/` | Network error to Google API | `400` | `"Failed to communicate with Google"` |
| `google/login/callback/` | Email absent in Google userinfo | `400` | `"Could not retrieve email from Google"` |
| `google/login/callback/` | Email not registered in system | `400` | `"No account found with this email"` |
| `google-mobile/` | `id_token`/`idToken` missing | `400` | `"ID token is required"` |
| `google-mobile/` | Google tokeninfo returns non-200 | `400` | `"Invalid or expired ID token"` |
| `google-mobile/` | No email in tokeninfo response | `400` | `"Could not retrieve email from token"` |
| `google-mobile/` | `email_verified` != `"true"` | `400` | `"Email not verified with Google"` |
| `google-mobile/` | Network error to Google tokeninfo | `400` | `"Failed to verify token with Google"` |
| `google-mobile/` | Email not registered in system | `400` | `"No account found with this email"` |

---

## 7. Login Attempt Logging

Every Google sign-in attempt (success or failure) is logged in the `LoginAttemptsLog` table for security auditing. The log captures:

| Field | Description |
|---|---|
| `email_muid` | Email from Google or `"unknown"` if not retrieved |
| `status` | `SUCCESS`, `FAILURE`, `INVALID_EMAIL` |
| `type` | Always `GOOGLE` for these endpoints |
| `ip_address` | Client IP (supports `X-Forwarded-For`) |
| `browser` | Parsed browser name from User-Agent |
| `os` | Parsed OS name from User-Agent |
| `device_type` | Parsed device type from User-Agent |
| `city` | Geo-IP lookup result |
| `region` | Geo-IP lookup result |
| `country` | Geo-IP lookup result |
| `location` | Lat/lon from Geo-IP |

---

## 8. Integration Examples

### 8.1 Web (JavaScript / React)

```javascript
// Step 1: Get the Google auth URL and redirect
async function loginWithGoogle() {
    const res = await fetch('http://127.0.0.1:8000/api/v1/auth/signin-with-google/');
    const data = await res.json();
    
    if (!data.hasError) {
        // Redirect the user to Google's OAuth2 consent screen
        window.location.href = data.response.redirect_url;
    }
}

// Step 2: After Google redirects back, the callback is handled server-side.
// Store the tokens returned from the redirect page:
function handleCallback() {
    // The callback page at /google/login/callback/ returns the tokens directly
    // Parse them from the server's redirect to your frontend
    const params = new URLSearchParams(window.location.search);
    const accessToken = params.get('accessToken');
    const refreshToken = params.get('refreshToken');
    
    localStorage.setItem('accessToken', accessToken);
    localStorage.setItem('refreshToken', refreshToken);
}
```

### 8.2 Mobile (Flutter)

```dart
import 'package:google_sign_in/google_sign_in.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

final GoogleSignIn _googleSignIn = GoogleSignIn(
  scopes: ['email', 'profile'],
);

Future<void> signInWithGoogle() async {
  try {
    // Step 1: Trigger Google Sign-In via the SDK
    final GoogleSignInAccount? account = await _googleSignIn.signIn();
    if (account == null) return; // User cancelled

    // Step 2: Get the ID token
    final GoogleSignInAuthentication auth = await account.authentication;
    final String? idToken = auth.idToken;

    if (idToken == null) {
      print('Failed to get ID token');
      return;
    }

    // Step 3: Send the ID token to our backend
    final response = await http.post(
      Uri.parse('http://127.0.0.1:8000/api/v1/auth/google-mobile/'),
      headers: {'Content-Type': 'application/json'},
      body: jsonEncode({'id_token': idToken}),
    );

    final data = jsonDecode(response.body);

    if (!data['hasError']) {
      final String accessToken = data['response']['accessToken'];
      final String refreshToken = data['response']['refreshToken'];
      final Map<String, dynamic> user = data['response']['user'];

      print('Logged in as: ${user['fullName']} (${user['muid']})');
      // Store tokens securely using flutter_secure_storage
    } else {
      print('Error: ${data['message']['general'][0]}');
    }
  } catch (e) {
    print('Google Sign-In failed: $e');
  }
}
```

### 8.3 Mobile (React Native)

```javascript
import { GoogleSignin } from '@react-native-google-signin/google-signin';

// Configure once (e.g., in App.js)
GoogleSignin.configure({
  webClientId: 'YOUR_WEB_CLIENT_ID.apps.googleusercontent.com',
});

async function googleSignIn() {
  try {
    // Step 1: Sign in with Google SDK
    await GoogleSignin.hasPlayServices();
    const userInfo = await GoogleSignin.signIn();
    const idToken = userInfo.idToken;

    // Step 2: Send to backend
    const response = await fetch('http://127.0.0.1:8000/api/v1/auth/google-mobile/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ id_token: idToken }),
    });

    const data = await response.json();

    if (!data.hasError) {
      const { accessToken, refreshToken, user } = data.response;
      console.log('Signed in:', user.fullName, user.muid);
      // Save tokens to AsyncStorage or SecureStore
    } else {
      console.error('Login failed:', data.message.general[0]);
    }
  } catch (error) {
    console.error('Google Sign-In error:', error);
  }
}
```

### 8.4 cURL (Testing)

**Web Flow — Get redirect URL:**
```bash
curl -X GET http://127.0.0.1:8000/api/v1/auth/signin-with-google/
```

**Mobile Flow — Authenticate with ID token:**
```bash
curl -X POST http://127.0.0.1:8000/api/v1/auth/google-mobile/ \
  -H "Content-Type: application/json" \
  -d '{"id_token": "YOUR_GOOGLE_ID_TOKEN_HERE"}'
```

---

## 9. Environment Configuration

The following environment variables must be set in `.env` for Google auth to work:

| Variable | Description | Example |
|---|---|---|
| `SOCIAL_AUTH_GOOGLE_OAUTH2_KEY` | Google OAuth2 Client ID | `123456789-abc.apps.googleusercontent.com` |
| `SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET` | Google OAuth2 Client Secret | `GOCSPX-abc123` |
| `SECRET_KEY` | Django secret key (used to sign JWTs) | `your-secret-key-here` |

### How to Obtain Google Credentials

1. Go to [Google Cloud Console](https://console.cloud.google.com/).
2. Navigate to **APIs & Services → Credentials**.
3. Click **Create Credentials → OAuth 2.0 Client IDs**.
4. For **Web**: Set the application type to **Web application** and add your callback URL:
   ```
   http://127.0.0.1:8000/api/v1/auth/google/login/callback/
   ```
5. For **Mobile (Android)**: Set the application type to **Android** and provide your package name and SHA-1 fingerprint.
6. For **Mobile (iOS)**: Set the application type to **iOS** and provide your bundle ID.
7. Copy the `Client ID` and `Client Secret` into your `.env` file.

---

## 10. Security Considerations

| Topic | Detail |
|---|---|
| **Account creation** | These endpoints do **not** create new accounts. The user's email must already exist in the system. |
| **Token audience validation** | The mobile endpoint (`google-mobile/`) does not currently enforce `aud` (audience) validation on the Google ID token. Validate the `aud` field against your own Client IDs in production. |
| **HTTPS in production** | All token exchanges must happen over HTTPS. Never use HTTP in production. |
| **JWT secret** | The `SECRET_KEY` in `.env` signs all JWTs. Keep it secret and rotate it if compromised. |
| **Access token expiry** | Access tokens expire in **3 hours**. Use the refresh token to get a new one via `/get-access-token/`. |
| **Refresh tokens** | Refresh tokens have no server-side expiry. Invalidate them by deleting them client-side or implementing a token blacklist. |
| **Login attempt logging** | All failed attempts are logged with IP, browser, OS, device, and geolocation for security auditing. |
| **Brute-force protection** | The standard login endpoint has brute-force protection, but the Google endpoints do not (since they rely on Google's own auth). |

---

## Quick Reference Card

| Endpoint | Method | Auth Required | Use Case |
|---|---|---|---|
| `/api/v1/auth/signin-with-google/` | `GET` | No | Get Google OAuth2 redirect URL (web) |
| `/api/v1/auth/google/login/callback/` | `GET` | No | OAuth2 callback — called by Google (web) |
| `/api/v1/auth/google-mobile/` | `POST` | No | Verify Google ID token (mobile apps) |
| `/api/v1/auth/get-access-token/` | `POST` | No (refresh token in body) | Refresh expired access token |
