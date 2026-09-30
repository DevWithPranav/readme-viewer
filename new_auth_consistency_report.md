# "Sign in with muLearn": Dashboard Changes Needed

> **What this doc answers:** the backend and the auth server changed for the new auth system. **What must change in the dashboard because of that, and how much of it is already done?**
> **Repos:** authserver `feat/new-auth` · mulearnbackend `feat/new-auth` · mulearn-dashboard `feat/sign-in-with-mulearn` (gtech-mulearn)
> **Review date:** 2026-09-30
> **Type:** read-only review. No code was changed in any of the three repos.

---

## Table of Contents

1. [Summary](#1-summary)
2. [Branches Reviewed](#2-branches-reviewed)
3. [How the New Auth Works (Short Version)](#3-how-the-new-auth-works-short-version)
4. [Dashboard Change List (Main Section)](#4-dashboard-change-list-main-section)
5. [Details: What to Do for Each Open Item](#5-details-what-to-do-for-each-open-item)
6. [Also Needed Outside the Dashboard](#6-also-needed-outside-the-dashboard)
7. [Change Log: Auth Server](#7-change-log-auth-server)
8. [Change Log: Backend](#8-change-log-backend)
9. [Change Log: Dashboard (What the Frontend Dev Already Did)](#9-change-log-dashboard-what-the-frontend-dev-already-did)
10. [Contract Check: What Matches](#10-contract-check-what-matches)
11. [Environment Variables: Must Match Across Repos](#11-environment-variables-must-match-across-repos)
12. [Setup & Deploy Order](#12-setup--deploy-order)
13. [Test Checklist](#13-test-checklist)

---

## 1. Summary

Status key: ✅ Done · 🟡 Partly done · ❌ Not done · ➖ No change needed

| | Count |
|---|---|
| ✅ Already done in the dashboard | **9** |
| 🟡 Partly done (started, but has a gap or bug) | **4** |
| ❌ Not done yet | **9** |
| ➖ Backend or auth server changed, but the dashboard needs no change | **7** |

The dashboard branch already has the core of "Sign in with muLearn":
- PKCE sign-in with a server-side code exchange.
- Refresh token stored httpOnly and rotated on every refresh.
- The edge proxy reads the new token format.
- The onboarding field.
- The Google `state` fix.

What is missing is mostly **session handling around the switch**: logout, refresh, the flag, register and password change.

**One item needs attention even with the flag OFF** ([D1](#d1)). The backend branch changed the error body it returns for an expired or missing token. The dashboard no longer recognises it, so **it never refreshes**. As soon as that backend is deployed, users see "Token expired" errors instead of a silent refresh.

**Must fix before turning `OIDC_ENABLED` on:** [D1](#d1), [D2](#d2), [D3](#d3), [D4](#d4), [D5](#d5).

---

## 2. Branches Reviewed

| Repo | Branch | Head commit | Compared against | Size of change |
|---|---|---|---|---|
| authserver | `DevWithPranav/authserver` → `feat/new-auth` | `b790491` (2026-09-24) | `dev` | 80 files, +6677 / −203 |
| mulearnbackend | `DevWithPranav/mulearnbackend` → `feat/new-auth` | `205a9acf` (2026-09-24) | `gtech-mulearn/mulearnbackend` `dev` | 26 files, +2047 / −168 |
| mulearn-dashboard | `gtech-mulearn/mulearn-dashboard` → `feat/sign-in-with-mulearn` | `c3a01d1` (2026-08-25) | `gtech-mulearn/mulearn-dashboard` `dev` | 22 files, +1168 / −45 |

**Timing matters here.** The dashboard work is **one commit from 2026-08-25**. The backend and authserver got more commits on **2026-09-24**: session revocation, the admin console, password moving to authserver, and the token refactor. So the dashboard was built against an older version of the other two, and several ❌ items below come from those later commits.

The dashboard branch is also **133 commits behind** dashboard `dev`. Only `src/api/endpoints.ts` changed on both sides, so expect one small merge conflict.

---

## 3. How the New Auth Works (Short Version)

```
Browser                Dashboard server (Next.js BFF)        authserver (auth.mulearn.org)        mulearnbackend
   |                              |                                   |                                  |
   | GET /login                   |                                   |                                  |
   |----------------------------->| OIDC_ENABLED=true                  |                                  |
   |                              | -> /api/auth/oidc/start            |                                  |
   |                              |   make PKCE verifier + state       |                                  |
   |                              |   store in httpOnly cookies        |                                  |
   |<-- 307 to /oauth/authorize/ -|                                   |                                  |
   |------------------------------------------------------------------>| login / signup page               |
   |                              |                                   | (signup -> backend                |
   |                              |                                   |  /protected/identity/             |
   |                              |                                   |  provision-member/)  ------------>|
   |<-- 302 /api/auth/oidc/callback?code&state ------------------------|                                  |
   |----------------------------->| check state, POST /oauth/token/    |                                  |
   |                              |---------------------------------->| RS256 access token (15 min)       |
   |                              |<----------------------------------| + opaque refresh token (7 days)   |
   |<-- cookies: accessToken (JS) | refreshToken (httpOnly)            |                                  |
   |                                                                                                      |
   | API call  Authorization: Bearer <RS256 access token>  ---------------------------------------------->|
   |                                                                   JWKS fetch (cached 1h) <-----------| verify iss/aud/exp/sig
   |                                                                                                      | check scope, load roles from DB
```

- **authserver** is the only service that mints tokens (private RSA key).
- **backend** only verifies them (public keys from JWKS). It accepts **both** old HS256 and new RS256 tokens during the move.
- **New tokens carry no roles and no muid.** The backend loads them from its DB on every request.

---

## 4. Dashboard Change List (Main Section)

Each row is a change in the **backend** or **authserver**, what the **dashboard** must do because of it, and whether the frontend dev has **already done it**.

### 4.1 Caused by authserver changes

| ID | authserver change | What the dashboard must do | Status | Where in dashboard |
|---|---|---|---|---|
| A1 | New OIDC provider: `/oauth/authorize/`, `/oauth/token/`, PKCE **S256 required** | Start the sign-in on the server: make the verifier and state, keep them in httpOnly cookies, exchange the code on the server | 🟡 Built, but the redirect URI is wrong on Netlify → [D4](#d4). A failed sign-in loops → [D8](#d8) | `api/auth/oidc/start`, `api/auth/oidc/callback` |
| A2 | Access token is **RS256** with `exp` (seconds), **no `roles` claim** | Edge proxy must read `exp`, and must not treat "no roles" as "no access" | ✅ Done | `src/proxy.ts` |
| A3 | Refresh token is **rotated** on every use. Reusing an old one **revokes the whole session**. Grace period is 0 | Save the new refresh token after every refresh, and refresh **single-flight** (never two at once) | 🟡 Saving is ✅. Single-flight is ❌ → [D7](#d7) | `api/auth/refresh/route.ts` |
| A4 | New refresh token is an **opaque string** that only `/oauth/token/` accepts. The old JWT refresh token only works on the legacy endpoint | Choose the refresh / logout method by **token type**, not by the flag | ❌ → [D3](#d3) | `refresh/route.ts`, `logout/route.ts`, `api/server.ts`, `api/refresh.client.ts` |
| A5 | Refresh token is **httpOnly** (JS cannot read it) | Browser-side refresh must go through a same-origin dashboard route | ❌ → [D6](#d6) | `api/refresh.client.ts`, `api/client.ts` |
| A6 | `/oauth/revoke_token/` and **RP-initiated logout** `/oauth/logout/` are enabled | On logout, revoke the token **and** end the auth.mulearn.org session | 🟡 Revoke is ✅. Provider logout is ❌ → [D2](#d2) | `logout/route.ts`, topbar, sidebar |
| A7 | `mulearn.read` / `mulearn.write` allowed for **first-party** clients only | Request `openid profile email mulearn.read mulearn.write` | ✅ Done (client must be registered `--public --skip-authorization`, see [§6](#6-also-needed-outside-the-dashboard)) | `oidc/start/route.ts` |
| A8 | Signup now lives on authserver (`/accounts/signup/`) and creates the member through the backend | When the flag is on, `/register` should send people to the provider signup | ❌ → [D9](#d9) | `(auth)/register` |
| A9 | Legacy Google web sign-in now **issues and checks `state`** (F5). The response includes `state` | Remember `state`, check it on return, and send it to the callback | ✅ Done | `oauth-state.ts`, `use-google-login.ts`, `callback-page-client.tsx`, `endpoints.ts`, schema |
| A10 | Legacy Apple web sign-in also uses `state` now | – | ➖ No change: the Apple button is commented out in the dashboard | – |
| A11 | `request-otp` now always answers *"If an account exists…, an OTP has been sent"* (F7) | – | ➖ No change: the dashboard shows the backend message | `use-request-otp.ts` |
| A12 | Legacy refresh `/api/v1/auth/get-access-token/` now returns **503** when Redis is down (fails closed, F6) | Treat 503 as "try again", not "log out" | ❌ → part of [D5](#d5) | `refresh.server.ts`, `refresh.client.ts` |
| A13 | `/oauth/token/` is **rate limited per IP** (120/min) | Treat **429** as "try again", not "log out". authserver must also change its limit ([§6](#6-also-needed-outside-the-dashboard)) | ❌ → [D5](#d5) | `oidc-refresh.ts`, `refresh/route.ts` |
| A14 | New config: issuer and client id per environment | Add env vars, and fail at build time if they are missing | 🟡 In `.env.example` ✅. Not validated, and there is no fixed redirect URI var ❌ → [D12](#d12) | `.env.example`, `config/env.ts` |
| A15 | CORS changed from allow-all to an allowlist (F17) | – (config only: add every dashboard origin) | ➖ No code change | – |

### 4.2 Caused by backend changes

| ID | backend change | What the dashboard must do | Status | Where in dashboard |
|---|---|---|---|---|
| B1 | **Auth error body changed.** Expired or missing token now returns `403 {"detail": "Token expired"}`. It was `403 {"hasError": true, "message": {"general": [...]}, "statusCode": 1000}` | The dashboard must still see this as "token expired" and refresh. Best fixed in the **backend** by restoring the old body | ❌ **New, and breaks refresh even with the flag OFF** → [D1](#d1) | `api/client.ts → isTokenExpired` |
| B2 | Backend accepts **both** token formats, and checks **scopes** on new tokens | – | ➖ No change (the dashboard asks for both scopes) | – |
| B3 | `GET /dashboard/user/info/` adds `onboarding: {state, missing, exempt}` | Use the server's onboarding answer, with a fallback for an older backend | ✅ Done | `onboarding-guard.tsx`, `auth.schema.ts` |
| B4 | Roles for new tokens come from the DB, not the token | Get roles from `/user/info/`, not from the token | ✅ Already so (the UI uses `/user/info/`, and only the proxy read the token) | `lib/auth/server.ts`, `proxy.ts` |
| B5 | **Change password** now goes through authserver and **signs the user out everywhere**, including this session. A partial failure returns a **failure** response even though the password changed | After success: tell the user and sign them out. Treat "Password changed, but…" as success with a warning | ❌ → [D10](#d10) | `settings/account/change-password-form.tsx` |
| B6 | **Reset password** (email link) now goes through authserver. The success text may say *"Signing you out of your other devices may take a few minutes"*. Can return 503 | Show the backend message | ➖ Already shows the backend message | `use-reset-password.ts` |
| B7 | **Forgot password** always returns the same success text (F7) | – | ➖ No change | `use-forgot-password.ts` |
| B8 | New **throttles** (per IP): email check 20/min, password reset 5/hour, registration 10/hour. They return **429** `{"detail": "Request was throttled. Expected available in N seconds."}` | Show a friendly "Too many attempts, please try again later" | ❌ (low) → [D13](#d13) | `api/client.ts`, `use-get-error` |
| B9 | New **admin API** `/api/v1/dashboard/auth-admin/*` (8 endpoints, Admin only) | Build the admin screens | ❌ → [D11](#d11) | new pages |
| B10 | Legacy signups are **counted** (legacy path is removed after 7 days of zero) | Stop using the legacy signup when the flag is on | ❌ same as [D9](#d9) | `(auth)/register` |
| B11 | `user_domains` is still returned in user info | – | ➖ No change | – |

### 4.3 Already done by the frontend dev (for credit and review)

| ✅ Done | Files |
|---|---|
| PKCE (S256), `state` and the server-side code exchange | `lib/auth/pkce.ts`, `api/auth/oidc/start`, `api/auth/oidc/callback` |
| Refresh token in an **httpOnly** cookie, access token JS-readable (matches the current app) | `oidc/callback/route.ts` |
| Save the **rotated** refresh token on every refresh | `api/auth/refresh/route.ts`, `lib/auth/oidc-refresh.ts` |
| Revoke the refresh token at the provider on logout | `api/auth/logout/route.ts` |
| Proxy reads both `exp` (new) and `expiry` (old). No `roles` claim → let the server decide | `src/proxy.ts` |
| Onboarding from the server field, with the old rule as fallback | `onboarding-guard.tsx`, `auth.schema.ts` |
| Google `state` (browser half of F5) | `lib/auth/oauth-state.ts`, `use-google-login.ts` |
| Feature flag `OIDC_ENABLED` (read on the server only) + env docs | `(auth)/login/page.tsx`, `.env.example` |
| Unit tests (PKCE, state, token expiry) + a CI test job | `*.test.ts`, `.github/workflows/ci.yml`, `package.json` |

---

## 5. Details: What to Do for Each Open Item

Priority: 🔴 must fix before the flag goes on (D1 even before the backend deploys) · 🟠 should fix before going live · 🟡 can follow

---

<a id="d1"></a>
### D1 🔴 Dashboard no longer sees "token expired" from the backend (B1)

**Status:** ❌ Not done. **This one breaks even with `OIDC_ENABLED` off**, as soon as the new backend is deployed.

**What changed in the backend:** `utils/permission.py` now raises `UnauthorizedAccessException(str(exc))` with a plain string. DRF turns that into:

| | HTTP | Body |
|---|---|---|
| Before (upstream `dev`) | 403 | `{"hasError": true, "message": {"general": ["Token Expired or Invalid"]}, "statusCode": 1000}` |
| Now (`feat/new-auth`) | 403 | `{"detail": "Token expired"}` or `{"detail": "Invalid token header"}` |

I checked this by running the old and new exception code through DRF's real exception handler.

**What breaks in the dashboard:** `src/api/client.ts → isTokenExpired()` returns true only for **401**, or **`statusCode === 1000`**, or `message.general` containing "token expired". The new body matches **none** of these, so:
- the tab sits idle for 15 min, the access-token cookie expires, and the next API call is sent with no token;
- the backend answers `403 {"detail": "Invalid token header"}`;
- the dashboard does **not** refresh and shows an error toast instead. It keeps failing until the user reloads the page.

The mobile apps probably rely on `statusCode: 1000` in the same way.

**Fix (recommended, backend):** keep the old error body for auth failures in `JWTUtils._validated`:
```python
except TokenError as exc:
    raise UnauthorizedAccessException(
        {"hasError": True, "message": {"general": [str(exc)]}, "statusCode": 1000}
    ) from exc
```
Do the same for "Invalid token header", "Token has no subject" and the scope error. Then every client keeps working unchanged.

**Fix (extra safety, dashboard):** in `isTokenExpired`, also return true for `403` when `detail` matches `/token (expired|invalid)|invalid token|token header|signing keys/i`.

---

<a id="d2"></a>
### D2 🔴 Logout must end the auth.mulearn.org session (A6)

**Status:** 🟡 Partly done. Revoking the refresh token is ✅. Ending the provider session is ❌.

**What goes wrong now:** Log out → revoke → `window.location = "/login"` → (flag on) `/api/auth/oidc/start` → `/oauth/authorize/`. The **auth.mulearn.org session is still alive**, and the dashboard is first-party (`skip_authorization`), so a code is issued with no screen at all. **The user is signed straight back in.** This is a risk on shared college lab computers.

**What to do:**
1. In `oidc/callback/route.ts`, also save `tokens.id_token` in an **httpOnly** cookie (for example `idToken`, 7 days).
2. In `logout/route.ts`: revoke as now, clear cookies, and return
   `{ logoutUrl: "{issuer}/oauth/logout/?id_token_hint=<id>&client_id=<client>&post_logout_redirect_uri=<origin>/login?logged_out=1" }`.
3. In `app-topbar.tsx`, `app-sidebar.tsx` and `account-settings-modal.tsx`, navigate to `logoutUrl` (when present) instead of `/login`.
4. In `login/page.tsx`, do **not** auto-redirect when `logged_out=1`. Show a "You are signed out · Sign in" button.
5. Register the post-logout URI on the client (see [§6](#6-also-needed-outside-the-dashboard)).

Why the `id_token` is needed: authserver sets `OIDC_RP_INITIATED_LOGOUT_ALWAYS_PROMPT=False`. It skips its confirmation screen **only** when a valid `id_token_hint` is sent.

---

<a id="d3"></a>
### D3 🔴 Choose the refresh / logout path by token type, not by the flag (A4)

**Status:** ❌ Not done.

**What goes wrong now:** `refresh/route.ts` uses the OIDC refresh when `OIDC_ENABLED=true` and the legacy one otherwise. After the flag goes on, every signed-in user still holds a **legacy JWT** refresh token. It is sent to `/oauth/token/`, rejected with `invalid_grant`, and the user is logged out. **Everyone is logged out within 15 minutes**, although `.env.example` says *"Flipping this does NOT log anyone out"*. Turning the flag off does the same to OIDC users. Logout has the same mix-up.

**What to do:** add one helper and use it in all four places:
```ts
// src/lib/auth/token-kind.ts
/** Legacy refresh tokens are HS256 JWTs (3 dot-separated parts). OIDC ones are opaque. */
export function isLegacyRefreshToken(token: string): boolean {
  return token.split(".").length === 3;
}
```
| File | Change |
|---|---|
| `app/api/auth/refresh/route.ts` | legacy token → `refreshAccessTokenServer`; opaque token → `refreshOidcSession` (needs issuer + client id, whatever the flag) |
| `app/api/auth/logout/route.ts` | legacy token → backend logout; opaque token → `/oauth/revoke_token/` + [D2](#d2) |
| `api/server.ts → refreshAndSetToken()` | same split (it only knows the legacy path today) |
| `api/refresh.client.ts` | replaced by [D6](#d6) |

After this, the flag only decides **where new sign-ins go**, which is what the comments already promise.

---

<a id="d4"></a>
### D4 🔴 Use a fixed redirect URI from env (A1)

**Status:** 🟡 The flow is built, but the URI is taken from the request.

**What goes wrong now:** `start` and `callback` both use `${request.nextUrl.origin}/api/auth/oidc/callback`. The dashboard's own comment in `refresh/route.ts` says that **on Netlify the request URL is the deploy permalink** (`<deploy-id>--<site>.netlify.app`), not `app.mulearn.org`. authserver matches redirect URIs **exactly**, so production sign-in will very likely be refused. Even if that URI were allowed, the PKCE cookies would be on the other domain. It works on localhost, so it is easy to miss.

**What to do:**
- Add a server env var `OIDC_REDIRECT_URI` (for example `https://app.mulearn.org/api/auth/oidc/callback`).
- Use it in **both** `start/route.ts` and `callback/route.ts`. They must send the same value.
- Register exactly that value on the client, one per environment.

---

<a id="d5"></a>
### D5 🔴 Do not log users out on temporary errors (A12, A13)

**Status:** ❌ Not done.

**What goes wrong now:** `refreshOidcSession` turns **any** non-2xx or timeout into `RefreshFailed`, and the refresh route then clears the cookies. The legacy refresh does the same. But:
- `/oauth/token/` returns **429** when its per-IP limit (120/min) is hit. **Every dashboard refresh comes from the dashboard server's IP**, so at scale everyone shares one bucket.
- The legacy refresh returns **503** when Redis is down.

In both cases users are logged out for a temporary problem.

**What to do:**
- In `oidc-refresh.ts`, return a separate `RefreshTemporary` error for **429, 5xx and timeouts**. Keep `RefreshFailed` for **400 `invalid_grant`** (and 401) only.
- In `refresh/route.ts`, on `RefreshTemporary`, **keep the cookies** and send the user to a small "Connection problem, retrying…" page (or retry once after `Retry-After`). Clear the session only on `RefreshFailed`.
- Do the same for the legacy path (503).
- authserver must also change its `/oauth/token/` limit ([§6](#6-also-needed-outside-the-dashboard)).

---

<a id="d6"></a>
### D6 🟠 Browser-side refresh through a dashboard route (A5)

**Status:** ❌ Not done.

**What goes wrong now:** `refresh.client.ts` reads the refresh token with `js-cookie`. The OIDC one is **httpOnly**, so it gets `undefined`. It then calls the **legacy** endpoint anyway. So in an open tab, the first API call after 15 min redirects to `/login` → proxy → `/dashboard` → refresh. **The user loses the page they were on and any unsaved form.**

**What to do:**
- Add `POST /api/auth/refresh/session` (same origin). It reads the httpOnly cookie, refreshes on the server (with the [D3](#d3) split), sets the cookies, and returns `{ accessToken }`.
- Make `refresh.client.ts` call it with `credentials: "include"`, and keep its existing single-flight guard.
- Also make `api/server.ts` use the same server helper. It currently sets `accessToken` as httpOnly with a 24 h life, which does not match the rest.

---

<a id="d7"></a>
### D7 🟠 Refresh single-flight: no parallel refreshes (A3)

**Status:** ❌ Not done.

**Why:** authserver revokes the **whole session** if the same refresh token is used twice. Its settings comment says: *"Clients must refresh single-flight instead (the dashboard BFF's one same-origin refresh route)."* Next.js `<Link>` **prefetches** of protected pages right after expiry each go through `proxy.ts` → `/api/auth/refresh` at the same moment, and so do two tabs.

**What to do:**
- In `proxy.ts`, do not redirect prefetch requests to `/api/auth/refresh`. Check for the `Next-Router-Prefetch: 1` or `Purpose: prefetch` headers and just let them through.
- Refresh a little **before** expiry (for example 60 s early) from one place. After [D6](#d6), that is the client single-flight.
- Test with two tabs (see [§13](#13-test-checklist)).

---

<a id="d8"></a>
### D8 🟠 Show an error on failed sign-in instead of looping (A1)

**Status:** ❌ Not done.

**What goes wrong now:** the callback sends failures to `/login?error=…`. With the flag on, `login/page.tsx` redirects straight back to the provider and **ignores `error`**. A lasting error (such as a client registered with wrong flags) loops until the browser shows "Too many redirects".

**What to do:** in `login/page.tsx`, if `params.error` (or `logged_out`, from [D2](#d2)) is present, render a short message and a **Sign in** button linking to `/api/auth/oidc/start`. Do not redirect in that case.

| `error` value | Message |
|---|---|
| `signin_expired`, `signin_mismatch` | "Your sign-in took too long or was started in another tab. Please try again." |
| `signin_unavailable` | "Sign-in is temporarily unavailable. Please try again in a moment." |
| `signin_failed` | "We could not sign you in. Please try again." |

---

<a id="d9"></a>
### D9 🟠 `/register` → provider signup when the flag is on (A8, B10)

**Status:** ❌ Not done.

**What goes wrong now:** only `/login` switches. `/register` still uses the legacy signup and gets **legacy tokens**. Until [D3](#d3) is done, those users are logged out within 15 min. It also keeps the backend's legacy-signup counter above zero, so the legacy path can never be removed.

**What to do:**
- When `OIDC_ENABLED=true`, make `(auth)/register/page.tsx` redirect to `/api/auth/oidc/start?signup=1`.
- In `start/route.ts`, when `signup=1`, build the usual `/oauth/authorize/?…` path (with PKCE and state), but redirect to `{issuer}/accounts/signup/?next=<url-encoded authorize path>`.
- authserver's signup page follows a safe `next` after the account is created. It has **no `prompt=create`**, so `next` is the way to do it.
- New users then land in the dashboard with `onboarding.state = "INCOMPLETE"` and are sent to `/onboarding/interests`. That part is already ✅.

---

<a id="d10"></a>
### D10 🟠 Change password → "please sign in again" (B5)

**Status:** ❌ Not done.

**What changed in the backend:**
- `ResetPasswordAPI` now asks authserver to change the password, and authserver **revokes every session, including this one**. The dashboard form only resets itself, and the user is kicked out up to 15 min later with no reason given.
- If revocation partly fails, the backend returns a **failure** with *"Password changed, but existing sessions could not be signed out…"*. The dashboard shows it as an error, although the password **did** change.

**What to do in `change-password-form.tsx`:**
- On success: show "Password changed. Please sign in again with your new password." and run the logout (with [D2](#d2)).
- If the error message starts with "Password changed", show it as a **warning**, not an error, then log out the same way.
- (Better: the backend returns success-with-warning for that case. See [§6](#6-also-needed-outside-the-dashboard).)

---

<a id="d11"></a>
### D11 🟡 Admin console screens (B9)

**Status:** ❌ Not done. The backend and authserver sides are ready.

| Screen | Backend endpoint (Admin role) |
|---|---|
| Overview / security posture | `GET /api/v1/dashboard/auth-admin/security-posture/` |
| Connected apps: list, create | `GET` / `POST /api/v1/dashboard/auth-admin/clients/` |
| Connected app: view, edit | `GET` / `PATCH /api/v1/dashboard/auth-admin/clients/<client_id>/` |
| Connected app: disable / enable | `POST …/clients/<client_id>/disable/`, `POST …/clients/<client_id>/enable/` |
| Sign-in policy (attempt limit, block minutes) | `GET` / `PUT /api/v1/dashboard/auth-admin/signin-policy/` |
| Login attempts log | `GET /api/v1/dashboard/auth-admin/login-attempts/` |
| Revoke a member's sessions | `POST /api/v1/dashboard/auth-admin/sessions/revoke/` (button on user management) |

Suggested route: `/dashboard/admin/auth/*`. Add it to `route-access.ts` with the Admin role.

---

<a id="d12"></a>
### D12 🟡 Validate the new env vars (A14)

**Status:** 🟡 They are in `.env.example`. They are not validated, and `OIDC_REDIRECT_URI` does not exist yet.

**What to do:** add `NEXT_PUBLIC_OIDC_ISSUER`, `NEXT_PUBLIC_OIDC_CLIENT_ID`, `OIDC_ENABLED` and `OIDC_REDIRECT_URI` to `config/env.ts` (the t3 `createEnv` schema). They can be optional, but make them required together when `OIDC_ENABLED=true`, so a bad deploy fails at build time and not at sign-in.

---

<a id="d13"></a>
### D13 🟡 Friendly message for 429 throttles (B8)

**Status:** ❌ Not done.

**What changes:** the backend now throttles email check (20/min), password reset (5/hour) and registration (10/hour) per IP. DRF answers `429 {"detail": "Request was throttled. Expected available in 3456 seconds."}`, and the dashboard shows that text as it is.

**What to do:** in the shared error helper, map **429** to "Too many attempts. Please wait a bit and try again."

Note for the backend team: at campus events, many students share one college IP. **10 registrations/hour per IP** may block a workshop signup. See [§6](#6-also-needed-outside-the-dashboard).

---

## 6. Also Needed Outside the Dashboard

These are not dashboard code, but the dashboard items above depend on them.

| Where | What | Why |
|---|---|---|
| **backend** | Keep the old auth error body (`statusCode: 1000`) in `JWTUtils._validated` | [D1](#d1): refresh detection in the dashboard (and probably the mobile apps) |
| **backend** | Password change: return **success with a warning** when the password changed but some sessions were not revoked | [D10](#d10) |
| **backend** | Review the `registration: 10/hour` per-IP throttle for campus NAT | [D13](#d13) |
| **authserver** | `/oauth/token/` limit: rate-limit only the `authorization_code` grant per IP, **or** key it by `client_id + IP` with a higher limit for the dashboard, **or** allowlist the dashboard's server IPs | [D5](#d5): every dashboard refresh comes from one server IP |
| **authserver** | Add a "Forgot password?" link on `/accounts/login/` (for example to the dashboard's `/forgot-password`). `password/forgot` emails link to `{AUTH_PUBLIC_URL}/reset-password`, which only exists in the separate auth UI | Users on the new sign-in page cannot reset their password yet |
| **config** | Register the dashboard client per environment (below) | [D2](#d2), [D4](#d4), A7 |
| **config** | Issuer string **identical** in all three repos (bare origin, no trailing `/`) | The backend checks `iss` exactly. A mismatch rejects every new token |
| **config** | `CORS_ALLOWED_ORIGINS` in backend **and** authserver lists every dashboard origin (including Netlify previews if used) | Both defaults include `mulearn-dashboard.vercel.app` but not Netlify |
| **DB** | db-scripts `alter-1.89.sql` + enum alter for the 6 new `SystemActionLog` types | authserver tables. Admin console audit rows |
| **known** | After "sign out everywhere", already-issued access tokens still work at the backend for **up to 15 min** (both formats) | The backend verifies locally. Accepted trade-off; tell admins |

**Registering the dashboard client:**
```bash
python manage.py register_oauth_client \
  --name "muLearn Dashboard (prod)" \
  --client-id mulearn-dashboard-prod \
  --public \
  --skip-authorization \
  --redirect-uri https://app.mulearn.org/api/auth/oidc/callback \
  --post-logout-redirect-uri "https://app.mulearn.org/login?logged_out=1"
```
- Without `--public`, the code exchange fails, because the dashboard sends no secret.
- Without `--skip-authorization`, the `mulearn.*` scopes are refused, sign-in fails and loops ([D8](#d8)).
- Use one client **per environment**.

---

## 7. Change Log: Auth Server

Branch `feat/new-auth`, 7 commits (Pranav P, awindsr).

### 7.1 New: OIDC provider ("Sign in with muLearn")
- Uses **django-oauth-toolkit**, mounted at `/oauth/`: `authorize/`, `token/`, `revoke_token/`, `introspect/`, `userinfo/`, `logout/`, `.well-known/openid-configuration`, `.well-known/jwks.json`. Also `/.well-known/openid-configuration` at the root.
  - **Not mounted on purpose:** self-service client registration, dynamic registration, device flow, token pages.
- **Access token:** RS256 JWT (`typ: at+jwt`), **15 min**. Claims: `iss, sub, aud, client_id, scope, iat, exp, jti`. **No roles, no muid.**
- **Refresh token:** opaque, **7 days**, **rotated** on each use. Reuse **revokes the whole family**. Grace period is 0.
- **ID token:** 15 min, with `name`, `email` and `muid` (display only).
- **PKCE S256 required.** Only the `code` response type. Implicit and password grants are off. All RFC 9700 flags are on.
- **Scopes:** `openid`, `profile`, `email`, `mulearn.read`, `mulearn.write` (the last two for first-party clients only).
- Suspended users cannot refresh. `prompt=none` is supported.
- `/oauth/token/` rate limit: 120/min per IP. A parallel-refresh race returns `invalid_grant`, not a 500.
- Redirect URIs: `https`, `mulearn://`, or `http://localhost` (only when `ALLOW_LOCALHOST_REDIRECTS=True`). Matching is exact.

### 7.2 New: Pages & JSON API for sign-in
- Pages: `/accounts/login/`, `/accounts/signup/`, `/accounts/logout/`.
- JSON API for the upcoming auth UI (`/accounts/api/`): `csrf`, `login`, `otp/request`, `signup`, `signup-context`, `google/start`, `google/callback`, `client`, `me`, `logout`, `consent`, `password/forgot`, `password/verify`, `password/reset`.
- Signup calls backend `POST /api/v1/protected/identity/provision-member/`.

### 7.3 New: Internal API for the backend (`/api/v1/internal/`, `protectionKey`)
`password/change/`, `password/set/`, `sessions/revoke/`, `maintenance/cleartokens/`, `admin/login-attempts/`, `admin/security-posture/`, `admin/signin-policy/`, `admin/clients/`, `admin/clients/<id>/`, `admin/clients/<id>/disable/`, `admin/clients/<id>/enable/`

### 7.4 Data & sessions
- The `user` table is `AUTH_USER_MODEL`. OAuth tables are `managed = False`, owned by db-scripts `alter-1.89.sql`.
- Sessions are stored in Redis. `SessionRevocationMiddleware` ends auth.mulearn.org sessions after "revoke all".
- `revoke_all_sessions` covers the legacy `global_logout`, browser sessions, and all OIDC tokens.
- Commands: `register_oauth_client`, `verify_oidc_flow`.

### 7.5 Security fixes to legacy `/api/v1/auth/*`
| Fix | What changed |
|---|---|
| F5 | Google and Apple web: single-use `state`, checked on callback. The response includes `state` |
| F4 | Google mobile: ID token audience checked (`GOOGLE_ALLOWED_CLIENT_IDS`) |
| F1 | Apple: token signature verified. The client-sent email fallback was removed |
| F18 | Removed the hidden `flag_register_*` login bypass |
| F7 | `request-otp` no longer shows whether an account exists |
| F6 | Legacy refresh fails **closed** (503) if Redis is down |
| F9 | `token-verification`: constant-time key check, every use logged |
| F14 | Geolocation has a timeout and uses the real client IP |
| F17 | CORS allowlist |
| OTP | An OTP can be used once (race fixed) |
| Policy | Brute-force limits can be edited by an admin |

### 7.6 Other
- Per-IP limits on the new pages: login 30/min, OTP 10/10 min, password forgot 10/h, signup 20/h, token 120/min, Google 30/min.
- `security.log` (for example `token.reuse_detected`). Large test suite plus CI.

---

## 8. Change Log: Backend

Branch `feat/new-auth`, compared with upstream `dev`.

### 8.1 Token verification: both formats
- **RS256 (new):** verified with authserver's JWKS (`{OIDC_ISSUER}/oauth/.well-known/jwks.json`). Checks `iss`, `aud`, `exp` and the signature. The JWKS cache lasts 1 h and keeps the last good keys if authserver is down.
- **HS256 (legacy):** checked with `SECRET_KEY` and the `expiry` field.
- The format is picked from the token header. HS256 is never checked with a public key.
- **Scopes (new tokens only):** GET needs `mulearn.read` or `mulearn.write`. Everything else needs `mulearn.write`.
- Roles and muid for new tokens come from the DB.
- **F10:** the token is verified once per request, and every accessor uses that result.
- ⚠️ **The error body changed** (see [D1](#d1)).
- Per-format counter in `logs/auth_migration.log`.

### 8.2 Password owned by authserver
- Change password (`ResetPasswordAPI`) → authserver `password/change/`.
- Reset link (`ResetPasswordConfirmAPI`) → authserver `password/set/`.
- Both sign the user out everywhere (F3). A Celery retry task plus a sweep (at :07/:22/:37/:52) finish partial failures.

### 8.3 New endpoints
| Endpoint | Caller |
|---|---|
| `POST /api/v1/protected/identity/provision-member/` | authserver signup (`protectionKey`) |
| `/api/v1/dashboard/auth-admin/*`: `login-attempts/`, `security-posture/`, `signin-policy/`, `clients/`, `clients/<id>/`, `clients/<id>/disable/`, `clients/<id>/enable/`, `sessions/revoke/` | dashboard, Admin only. Audit rows in `SystemActionLog` (6 new enum values) |

### 8.4 Visible to the dashboard
- `user/info/` adds `onboarding: {state, missing, exempt}`. `Company` → exempt.
- Forgot password always returns the same success text.
- Throttles: email check 20/min, password reset 5/hour, registration 10/hour (per IP).
- CORS allowlist.

### 8.5 Other
- Celery beat: clear expired tokens hourly at :50.
- `db/user.py`: `deleted_at` and `deleted_by` declared. `last_login` not declared on purpose.
- Side change: achievement views no longer send raw exception text.

---

## 9. Change Log: Dashboard (What the Frontend Dev Already Did)

One commit `c3a01d1` (awindsr, 2026-08-25).

| File | What it does |
|---|---|
| `(auth)/login/page.tsx` | Flag on → redirect to `/api/auth/oidc/start?ruri=…`. Flag off → the old form |
| `api/auth/oidc/start/route.ts` | PKCE verifier, challenge and state in httpOnly cookies (10 min). Redirects to `/oauth/authorize/` with scope `openid profile email mulearn.read mulearn.write` |
| `api/auth/oidc/callback/route.ts` | Clears the flow cookies, checks state, exchanges the code. Sets `refreshToken` (httpOnly, 7 d), `accessToken` (JS, `expires_in`) and `isAuthenticated` |
| `api/auth/refresh/route.ts` | Flag on → OIDC refresh, saves the rotated token. Flag off → legacy |
| `api/auth/logout/route.ts` | Flag on → `/oauth/revoke_token/`. Flag off → legacy logout |
| `lib/auth/pkce.ts`, `lib/auth/oidc-refresh.ts` | PKCE helpers. Refresh with a 10 s timeout |
| `src/proxy.ts` | Reads `exp` and `expiry`. No `roles` claim → defers to the server |
| `onboarding-guard.tsx`, `auth.schema.ts` | Server onboarding field with fallback. Google URL response requires `state` |
| `lib/auth/oauth-state.ts`, `use-google-login.ts`, `callback-page-client.tsx`, `endpoints.ts` | Google `state` remember, check and forward |
| `.env.example`, `package.json`, `ci.yml`, tests | Env docs, vitest, CI test job |

---

## 10. Contract Check: What Matches

✅ consistent · ⚠️ only with the right config · ❌ mismatch (see the item)

| Item | authserver | backend | dashboard | Status |
|---|---|---|---|---|
| OAuth URLs | `/oauth/authorize/`, `/oauth/token/`, `/oauth/revoke_token/` | – | same | ✅ |
| JWKS URL | `/oauth/.well-known/jwks.json` | same | – | ✅ |
| Signing + `kid` | RS256, `kid` = thumbprint | looks up key by `kid` | – | ✅ |
| `iss` | `OIDC_ISSUER` | exact match | `NEXT_PUBLIC_OIDC_ISSUER` | ⚠️ must be identical |
| `aud` | `mulearn-api` | `mulearn-api` | – | ✅ |
| User id | `sub` = `user.id` | uses `sub` | – | ✅ |
| Expiry | `exp` | enforced | proxy reads `exp` | ✅ |
| Roles | not in token | from DB | from `/user/info/` | ✅ |
| Scopes | `mulearn.*` first-party only | read / write check | asks for both | ⚠️ register `--skip-authorization` |
| PKCE | S256 required | – | S256 | ✅ |
| Client type | public / confidential | – | public | ⚠️ register `--public` |
| Token lifetimes | 15 min / 7 days | – | same cookie lifetimes | ✅ |
| Refresh rotation | on, grace 0 | – | saves new token | ✅ (single-flight ❌ [D7](#d7)) |
| Google `state` | issued + consumed | – | stored, checked, forwarded | ✅ |
| Onboarding | – | `{state, missing, exempt}` | same schema | ✅ |
| Auth error body | – | `403 {"detail"}` | expects `statusCode: 1000` or 401 | ❌ [D1](#d1) |
| Old ↔ new token switch | – | accepts both | picks by flag | ❌ [D3](#d3) |
| Provider logout | enabled | – | not used | ❌ [D2](#d2) |
| Password change sign-out | yes | yes | not handled | ❌ [D10](#d10) |
| Admin console | ready | ready | no UI | ❌ [D11](#d11) |
| Internal API paths (backend ↔ authserver) | 11 routes + provision | all match | – | ✅ |

---

## 11. Environment Variables: Must Match Across Repos

| Meaning | authserver | backend | dashboard | Rule |
|---|---|---|---|---|
| Issuer | `OIDC_ISSUER` | `OIDC_ISSUER` | `NEXT_PUBLIC_OIDC_ISSUER` | **Identical.** Bare origin, no trailing `/` |
| Audience | `OIDC_ACCESS_TOKEN_AUDIENCE` | `OIDC_AUDIENCE` | – | Both `mulearn-api` |
| Internal key | `PROTECTED_API_KEY` | `PROTECTED_API_KEY` | – | Identical |
| Backend URL | `MULEARN_BACKEND_URL` | – | – | Points to the backend |
| authserver internal URL | – | `AUTH_DOMAIN` | – | Direct to authserver, not via the auth UI proxy |
| Signing key | `OIDC_RSA_PRIVATE_KEY` (+ `_INACTIVE`) | – | – | Never shared |
| Client id | registered | – | `NEXT_PUBLIC_OIDC_CLIENT_ID` | One per environment |
| Redirect URI | registered | – | `OIDC_REDIRECT_URI` *(new, [D4](#d4))* | Exact match |
| Flag | – | – | `OIDC_ENABLED` | Dev first |
| CORS | `CORS_ALLOWED_ORIGINS` | `CORS_ALLOWED_ORIGINS` | – | Every dashboard origin |
| Proxies | `RATE_LIMIT_TRUSTED_PROXIES` | – | – | 1 = nginx, 2 = auth UI + nginx |
| Public auth URL | `AUTH_PUBLIC_URL` | – | – | Reset emails, Google redirect |

---

## 12. Setup & Deploy Order

1. **Backend fix for [D1](#d1) first.** Do not deploy the backend branch to production before it, or the dashboard stops refreshing tokens.
2. **DB:** `alter-1.89.sql` + the `SystemActionLog` enum alter.
3. **authserver:** deploy. Check `/.well-known/openid-configuration`, `/oauth/.well-known/jwks.json` and `manage.py verify_oidc_flow`.
4. **backend:** deploy with the same `OIDC_ISSUER`. Legacy users are not affected.
5. **Register the dashboard client** per environment ([§6](#6-also-needed-outside-the-dashboard)).
6. **Dashboard:** finish D2 – D10 (at least the 🔴 ones). Deploy with `OIDC_ENABLED=false`.
7. Turn `OIDC_ENABLED=true` in **dev** and run [§13](#13-test-checklist).
8. Turn it on in **prod**. Watch backend `logs/auth_migration.log` and authserver `security.log` (`token.reuse_detected`, 429 on `/oauth/token/`).
9. After **7 days of zero** legacy tokens and legacy signups, remove the legacy paths.

---

## 13. Test Checklist

**Flag OFF, new backend deployed**
- [ ] Leave a tab idle for 16 min, then click something that calls the API → it refreshes silently, with no "Token expired" toast ([D1](#d1)).

**Sign in / out (flag ON)**
- [ ] Sign in on the **real domain**. `redirect_uri` is `app.mulearn.org` ([D4](#d4)).
- [ ] Log out, then Sign in → **the password is asked for** ([D2](#d2)).
- [ ] Wrong client config → an error screen appears, not a redirect loop ([D8](#d8)).
- [ ] `/register` → provider signup → back in the dashboard → sent to interests ([D9](#d9)).
- [ ] Company user → not sent to interests.

**Switching**
- [ ] Sign in with the flag OFF, turn it ON, wait 16 min → still signed in ([D3](#d3)).
- [ ] The reverse (ON → OFF) → still signed in.

**Token life**
- [ ] Idle for 16 min, then submit a form → the form is not lost ([D6](#d6)).
- [ ] Two tabs, token expired, click both at once → still signed in, and no `token.reuse_detected` ([D7](#d7)).
- [ ] authserver returns 429 / is down during a refresh → the user is **not** logged out ([D5](#d5)).
- [ ] Remove a role in the DB → the next API call is refused right away.

**Password**
- [ ] Change the password → "please sign in again" message, then signed out ([D10](#d10)).
- [ ] Forgot password → email → reset works.

**Legacy Google (flag OFF)**
- [ ] Google sign-in works, and the callback carries `state`.
- [ ] Open the callback URL in another browser → refused.
