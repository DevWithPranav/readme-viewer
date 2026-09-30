# "Sign in with muLearn": Change Log & Consistency Check

> **Scope:** the new auth system across the three repos: **authserver**, **mulearnbackend** and **mulearn-dashboard**.
> **Goal:** list what changed in each repo, then check that the dashboard work matches what the backend and the auth server actually do.
> **Review date:** 2026-09-30
> **Type:** read-only review. No code was changed in any of the three repos.

---

## Table of Contents

1. [Summary](#1-summary)
2. [Branches Reviewed](#2-branches-reviewed)
3. [How the New Auth Works (Short Version)](#3-how-the-new-auth-works-short-version)
4. [Change Log: Auth Server](#4-change-log-auth-server)
5. [Change Log: Backend](#5-change-log-backend)
6. [Change Log: Dashboard](#6-change-log-dashboard)
7. [Contract Check: What Matches](#7-contract-check-what-matches)
8. [Issues Found](#8-issues-found)
9. [Missing Pieces in the Dashboard](#9-missing-pieces-in-the-dashboard)
10. [Environment Variables: Must Match Across Repos](#10-environment-variables-must-match-across-repos)
11. [Setup & Deploy Order](#11-setup--deploy-order)
12. [Test Checklist Before Turning It On](#12-test-checklist-before-turning-it-on)

---

## 1. Summary

The **token contract is consistent**. The token format, JWKS location, audience, `sub` claim, scopes, PKCE and refresh rotation all line up across the three repos. The backend accepts both the old and new token formats. The onboarding field and the Google `state` fix also match on both sides.

The problems are in the **dashboard session handling around the flag (`OIDC_ENABLED`)**. These do not show up in local dev. They will show up the day the flag is turned on:

| # | Severity | Issue (short) | Repo to fix |
|---|---|---|---|
| 1 | 🔴 High | Logout does not log out. The user is signed straight back in with no password. | dashboard |
| 2 | 🔴 High | Turning the flag on logs out **every** existing user within 15 minutes, although the comments say it does not. | dashboard |
| 3 | 🔴 High | `redirect_uri` is built from the request origin. On Netlify that is the deploy permalink, not `app.mulearn.org`. | dashboard |
| 4 | 🔴 High | `/oauth/token/` has a per-IP limit of 120/min, and every dashboard refresh comes from the dashboard server's IP. At scale users get 429, and the dashboard then logs them out. | authserver + dashboard |
| 5 | 🟠 Medium | The browser-side refresh cannot work with OIDC (the refresh token is httpOnly, and the call goes to the legacy endpoint). The user is bounced and loses the current page every 15 min. | dashboard |
| 6 | 🟠 Medium | No single-flight on refresh. Parallel refreshes look like token theft, and authserver revokes the whole session. | dashboard |
| 7 | 🟠 Medium | A failed sign-in makes `/login` loop straight back to the provider, and the error is never shown. | dashboard |
| 8 | 🟠 Medium | `/register` still uses legacy signup when the flag is on, so new users get legacy tokens (then issue #2 hits them). | dashboard |
| 9 | 🟠 Medium | A password change now signs the user out everywhere, but the dashboard does not tell them or log them out. | dashboard |
| 10–17 | 🟡 Low / ℹ️ Info | Config must match exactly, CORS lists, revocation window, missing admin UI, forgot-password link, env validation, branch drift. | all |

Full details and fixes are in [Section 8](#8-issues-found).

---

## 2. Branches Reviewed

| Repo | Branch | Head commit | Compared against | Size of change |
|---|---|---|---|---|
| authserver | `DevWithPranav/authserver` → `feat/new-auth` | `b790491` (2026-09-24) | `dev` | 80 files, +6677 / −203 |
| mulearnbackend | `DevWithPranav/mulearnbackend` → `feat/new-auth` | `205a9acf` (2026-09-24) | `gtech-mulearn/mulearnbackend` `dev` | 26 files, +2047 / −168 |
| mulearn-dashboard | `gtech-mulearn/mulearn-dashboard` → `feat/sign-in-with-mulearn` | `c3a01d1` (2026-08-25) | `gtech-mulearn/mulearn-dashboard` `dev` | 22 files, +1168 / −45 |

**Timing matters here.** The dashboard work is **one commit from 2026-08-25**. The backend and authserver got several more commits on **2026-09-24** (session revocation, admin console, password ownership, token refactor). So the dashboard was built against an older version of the other two. Every issue below was checked against the **latest** backend and authserver code.

Other notes:
- The backend branch already merges `gtech-mulearn/mulearnbackend` → `feat/sign-in-with-mulearn` (`f0132c5`, the onboarding + revocation helpers).
- The dashboard branch is **133 commits behind** dashboard `dev`. Only `src/api/endpoints.ts` changed on both sides, so expect one small merge conflict.

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

Key ideas:
- **authserver** is the only service that **mints** tokens. It signs with a private RSA key.
- **backend** only **verifies** tokens, using the public keys from JWKS. It no longer holds a secret that can mint new-format tokens.
- **New tokens carry no roles and no muid.** The backend reads them from its own DB on every request, so a role change takes effect right away.
- **Both token formats work in the backend** during the move. The old HS256 path is removed once `logs/auth_migration.log` shows zero legacy use for 7 days.

---

## 4. Change Log: Auth Server

Branch `feat/new-auth`, 7 commits (Pranav P, awindsr).

### 4.1 New: OIDC provider ("Sign in with muLearn")
- Uses **django-oauth-toolkit**, mounted at `/oauth/`:
  - `/oauth/authorize/`, `/oauth/token/`, `/oauth/revoke_token/`, `/oauth/introspect/`, `/oauth/userinfo/`, `/oauth/logout/`
  - `/oauth/.well-known/openid-configuration`, `/oauth/.well-known/jwks.json`
  - also `/.well-known/openid-configuration` at the root
  - **Not mounted on purpose:** self-service client registration, dynamic registration, device flow, token management pages.
- **Access token:** RS256 JWT (RFC 9068, `typ: at+jwt`), **15 min**. Claims: `iss, sub, aud, client_id, scope, iat, exp, jti`. **No roles and no muid.**
- **Refresh token:** opaque random string, **7 days**. **Rotated on every use.** Reusing an old one **revokes the whole family**. Grace period is 0.
- **ID token:** 15 min. It adds `name`, `email` and `muid` (display only).
- **PKCE S256 required** for every client. Only the `code` response type. Implicit and password grants are off. All RFC 9700 flags are on.
- **Scopes:** `openid`, `profile`, `email`, `mulearn.read`, `mulearn.write`. **`mulearn.*` only for first-party clients** (`skip_authorization=True`).
- **Suspended users cannot refresh.** `prompt=none` (silent sign-in) is supported.
- `/oauth/token/` is **rate limited per IP** (120/min). A parallel-refresh race returns a normal `invalid_grant`, not a 500.
- Redirect URIs must be `https`, `mulearn://`, or `http://localhost` (only when `ALLOW_LOCALHOST_REDIRECTS=True`). Matching is exact.

### 4.2 New: Pages & JSON API for sign-in
- Server-rendered pages: `/accounts/login/`, `/accounts/signup/`, `/accounts/logout/`.
- JSON API for the upcoming auth UI (`/accounts/api/`): `csrf`, `login`, `otp/request`, `signup`, `signup-context`, `google/start`, `google/callback`, `client`, `me`, `logout`, `consent`, `password/forgot`, `password/verify`, `password/reset`.
- Signup calls **backend** `POST /api/v1/protected/identity/provision-member/`, so muid, wallet, level and role are still created in one place.

### 4.3 New: Internal API for the backend (`/api/v1/internal/`, `protectionKey`)
`password/change/`, `password/set/`, `sessions/revoke/`, `maintenance/cleartokens/`, `admin/login-attempts/`, `admin/security-posture/`, `admin/signin-policy/`, `admin/clients/`, `admin/clients/<id>/`, `admin/clients/<id>/disable/`, `admin/clients/<id>/enable/`

### 4.4 Data & sessions
- The `user` table is now `AUTH_USER_MODEL`. The OAuth tables are `managed = False` and owned by **db-scripts `alter-1.89.sql`**.
- Django sessions are stored in **Redis**. `SessionRevocationMiddleware` ends auth.mulearn.org sessions after "revoke all sessions".
- `revoke_all_sessions` covers the legacy `global_logout` key, auth.mulearn.org browser sessions, and all OIDC refresh and access tokens.
- Management commands: `register_oauth_client`, `verify_oidc_flow`.

### 4.5 Security fixes to the legacy `/api/v1/auth/*`
| Fix | What changed |
|---|---|
| F5 | Google and Apple web sign-in now issue a single-use `state` and check it on the callback. The response now includes `state`. |
| F4 | The Google mobile ID token **audience** is checked against `GOOGLE_ALLOWED_CLIENT_IDS`. |
| F1 | The Apple identity token **signature** is now verified. The client-sent email fallback was removed. |
| F18 | Removed the hidden `flag_register_*` login bypass. |
| F7 | `request-otp` no longer tells whether an account exists. |
| F6 | Refresh-token revocation check **fails closed** (503) if Redis is down. |
| F9 | `token-verification` uses a constant-time key check and logs every use. |
| F14 | Geolocation lookup has a timeout and uses the real client IP. |
| F17 | CORS allowlist instead of `CORS_ALLOW_ALL_ORIGINS`. |
| OTP | An OTP can be used once (race fixed). |
| Policy | Brute-force limits can be edited by an admin (shared by legacy and OIDC login). |

### 4.6 Other
- Per-IP limits: login 30/min, OTP 10/10 min, password forgot 10/h, signup 20/h, token 120/min, Google 30/min.
- A separate `security.log` records things like `token.reuse_detected`.
- A large test suite plus a CI workflow.

---

## 5. Change Log: Backend

Branch `feat/new-auth`, compared with upstream `dev`.

### 5.1 Token verification: both formats (`utils/token_verification.py`, `utils/permission.py`)
- **RS256 (new):** verified against authserver's **JWKS** at `{OIDC_ISSUER}/oauth/.well-known/jwks.json`. Checks `iss`, `aud` (`OIDC_AUDIENCE`), `exp` and the signature.
  - The JWKS cache lasts 1 h. There is at most 1 fetch per 5 s, and the last good keys are kept if authserver is down.
- **HS256 (legacy):** verified with `SECRET_KEY`, including the custom `expiry` field.
- The format is picked from the token header. An HS256 token is **never** checked with a public key, which blocks algorithm confusion.
- **Scope check (new tokens only):** `GET/HEAD/OPTIONS` needs `mulearn.read` or `mulearn.write`. Everything else needs `mulearn.write`.
- **Roles and muid** for new tokens come from the DB (`UserRoleLink`, `User`).
- **F10 fix:** the token is verified **once per request** and cached. `fetch_role`, `fetch_user_id` and `fetch_muid` all read that one result. Before, they skipped the expiry check.
- A per-format counter goes to `logs/auth_migration.log` (`mulearn.token_format`).

### 5.2 Password now owned by authserver
- `ResetPasswordAPI` (change password in settings) calls authserver `password/change/`.
- `ResetPasswordConfirmAPI` (forgot-password link) calls authserver `password/set/`.
- Both **sign the user out everywhere** (F3). If that partly fails, a Celery task and a sweep keep retrying (`mu_celery/auth_session_tasks.py`, at :07/:22/:37/:52).
- `utils/authserver_client.py` makes these calls with a timeout (3 s connect, 10 s read).

### 5.3 New endpoints
| Endpoint | Who calls it | Notes |
|---|---|---|
| `POST /api/v1/protected/identity/provision-member/` | authserver signup | `protectionKey`, constant-time check, runs in a transaction, returns `already_exists` for a duplicate email |
| `/api/v1/dashboard/auth-admin/login-attempts/` | dashboard (admin) | Admin role only |
| `/api/v1/dashboard/auth-admin/security-posture/` | dashboard (admin) | |
| `/api/v1/dashboard/auth-admin/signin-policy/` | dashboard (admin) | GET / PUT |
| `/api/v1/dashboard/auth-admin/clients/` | dashboard (admin) | GET / POST |
| `/api/v1/dashboard/auth-admin/clients/<id>/` | dashboard (admin) | GET / PATCH |
| `/api/v1/dashboard/auth-admin/clients/<id>/disable/` and `enable/` | dashboard (admin) | POST |
| `/api/v1/dashboard/auth-admin/sessions/revoke/` | dashboard (admin) | POST |

Every admin action is written to `SystemActionLog`. It uses **6 new enum values** (`AUTH_CLIENT_*`, `AUTH_POLICY_UPDATE`, `AUTH_SESSION_REVOKE`), so the **DB ENUM must be altered first**, or MySQL stores `''`.

### 5.4 Changes the dashboard can see
- `GET /api/v1/dashboard/user/info/` now also returns:
  ```json
  "onboarding": { "state": "COMPLETE" | "INCOMPLETE", "missing": ["interests"], "exempt": false }
  ```
  `Company` role → `{"state": "COMPLETE", "missing": [], "exempt": true}`. `user_domains` is still returned.
- `forgot-password` now always says *"If an account exists for that ID, a password reset link has been sent."*
- Throttles (per IP): email check 20/min, password reset 5/hour, registration 10/hour.
- The change-password response can be a **failure** even when the password **did** change. See [issue #9](#issue-9).
- CORS allowlist (`CORS_ALLOWED_ORIGINS`) replaces allow-all.

### 5.5 Other
- Celery beat: `clear-expired-auth-tokens-cron` runs hourly at :50 and calls authserver `maintenance/cleartokens/`.
- `db/user.py`: `deleted_at` and `deleted_by` are declared but not enforced. `last_login` is **not** declared on purpose (see the comment there).
- Legacy signups are counted (`mulearn.legacy_signup`).
- Unrelated side change: achievement views no longer send raw exception text to the client.

---

## 6. Change Log: Dashboard

Branch `feat/sign-in-with-mulearn`, one commit `c3a01d1` (awindsr, 2026-08-25).

### 6.1 New "Sign in with muLearn" flow (behind a flag)
| File | What it does |
|---|---|
| `src/app/(auth)/login/page.tsx` | If `OIDC_ENABLED === "true"`, redirect to `/api/auth/oidc/start?ruri=…`. Otherwise show the old form. |
| `src/app/api/auth/oidc/start/route.ts` | Creates the PKCE verifier, challenge and state. Stores `oidc_verifier`, `oidc_state` and `oidc_return` in httpOnly cookies (10 min). Redirects to `{issuer}/oauth/authorize/` with scope `openid profile email mulearn.read mulearn.write`. |
| `src/app/api/auth/oidc/callback/route.ts` | Clears the flow cookies, checks `state`, and POSTs to `{issuer}/oauth/token/` with the verifier. Sets `refreshToken` (httpOnly, 7 d), `accessToken` (JS-readable, `expires_in`) and `isAuthenticated`. |
| `src/app/api/auth/refresh/route.ts` | If the flag is on, calls `refreshOidcSession` and **stores the rotated refresh token**. Otherwise uses the legacy refresh. |
| `src/app/api/auth/logout/route.ts` | If the flag is on, POSTs the refresh token to `{issuer}/oauth/revoke_token/`. Otherwise uses the legacy logout. |
| `src/lib/auth/pkce.ts` | Verifier, S256 challenge, state, constant-time compare. |
| `src/lib/auth/oidc-refresh.ts` | Refresh-token grant with rotation and a 10 s timeout. |

### 6.2 Changes that work with or without the flag
| File | What changed |
|---|---|
| `src/proxy.ts` | Reads both `exp` (new) and `expiry` (legacy). If the token has **no `roles` claim**, it returns `null` and **lets the request through**, because the server checks roles. |
| `src/app/(dashboard)/onboarding-guard.tsx` | Uses the server's `onboarding.state`. Falls back to the old `user_domains` rule. |
| `src/features/auth/schemas/auth.schema.ts` | Optional `onboarding` field. Google auth URL response now **requires** `state`. |
| `src/lib/auth/oauth-state.ts` + `use-google-login.ts` + `callback-page-client.tsx` + `endpoints.ts` | Legacy Google flow: remembers `state` in sessionStorage, checks it on return, and sends it to the callback (F5). |
| `.env.example` | `NEXT_PUBLIC_OIDC_ISSUER`, `NEXT_PUBLIC_OIDC_CLIENT_ID`, `OIDC_ENABLED`. |
| `package.json`, `.github/workflows/ci.yml` | `vitest` test job (3 known-broken suites excluded by name). |
| tests | `pkce.test.ts`, `oauth-state.test.ts`, `token-expiry.test.ts`. |

---

## 7. Contract Check: What Matches

✅ = consistent  ⚠️ = works only with the right config  ❌ = mismatch (see Section 8)

| Item | authserver | backend | dashboard | Status |
|---|---|---|---|---|
| Authorize / token / revoke URLs | `/oauth/authorize/`, `/oauth/token/`, `/oauth/revoke_token/` | – | same paths | ✅ |
| JWKS URL | `/oauth/.well-known/jwks.json` | `{OIDC_ISSUER}/oauth/.well-known/jwks.json` | – | ✅ |
| Signing | RS256, `kid` = key thumbprint (same as JWKS) | RS256 via PyJWKClient, looks up key by `kid` | – | ✅ |
| `iss` | `OIDC_ISSUER` | `OIDC_ISSUER`, exact match | `NEXT_PUBLIC_OIDC_ISSUER` (used for URLs) | ⚠️ must be the **same string** ([#10](#issue-10)) |
| `aud` | `OIDC_ACCESS_TOKEN_AUDIENCE` = `mulearn-api` | `OIDC_AUDIENCE` = `mulearn-api` | – | ✅ (defaults match) |
| User id | `sub` = `user.id` | `sub` → `user_id` (same `user` table) | not read | ✅ |
| Expiry claim | `exp` (seconds) | enforced by PyJWT | proxy reads `exp * 1000` | ✅ |
| Roles / muid in token | not included | loaded from DB | proxy defers on `null`; UI gets roles from `/user/info/` | ✅ |
| Scopes | `mulearn.*` for first-party only | GET needs read, others need write | asks for `openid profile email mulearn.read mulearn.write` | ⚠️ client must be `--skip-authorization` ([#11](#issue-11)) |
| PKCE | required, S256 only | – | S256 | ✅ |
| Client type | public or confidential | – | public (no secret sent) | ⚠️ must register as `--public` ([#11](#issue-11)) |
| Access token life | 900 s | – | cookie = `expires_in` | ✅ |
| Refresh rotation + reuse detection | on, grace 0 | – | stores new token | ✅ (but see [#6](#issue-6)) |
| Refresh token life | 7 days | – | cookie 7 days | ✅ |
| Legacy Google `state` | issued in `signin-with-google`, consumed in callback | – | stored, checked, forwarded | ✅ |
| Onboarding object | – | `{state, missing, exempt}` | `z.object({state: enum, missing: string[], exempt: boolean})` | ✅ |
| Company role title | – | `"Company"` | `ROLES.COMPANY = "Company"` | ✅ |
| Backend → authserver internal paths | 11 routes | `password/change`, `password/set`, `sessions/revoke`, `maintenance/cleartokens`, `admin/*` | – | ✅ all match |
| authserver → backend provision path | `/api/v1/protected/identity/provision-member/` | same | – | ✅ |
| `PROTECTED_API_KEY` | used both ways | used both ways | – | ⚠️ must be the same value |
| RP logout (`/oauth/logout/`) | enabled | – | **not used** | ❌ [#1](#issue-1) |
| Old ↔ new token switch | – | accepts both | refresh picks path by **flag**, not by token | ❌ [#2](#issue-2) |
| Password change signs you out | yes | yes | not handled | ❌ [#9](#issue-9) |
| Admin console | internal API ready | 8 endpoints ready | **no UI** | ❌ [#14](#issue-14) |

---

## 8. Issues Found

Each issue has: **where**, **what goes wrong**, and **how to fix**.

---

<a id="issue-1"></a>
### #1 🔴 Logout does not log out (user is signed straight back in)

**Where:** `mulearn-dashboard/src/app/api/auth/logout/route.ts`, `src/components/dashboard/app-topbar.tsx`

**What goes wrong:**
1. The user clicks **Log out**. The route revokes the refresh token at `/oauth/revoke_token/` and deletes the cookies.
2. The UI then does `window.location.href = "/login"`.
3. With the flag on, `/login` redirects to `/api/auth/oidc/start` and then to `/oauth/authorize/`.
4. The **auth.mulearn.org session cookie is still alive** (Django session in Redis). The dashboard is a first-party client (`skip_authorization`), so authserver issues a code **with no screen at all**.
5. The user lands back on `/dashboard`, still signed in.

The comment in `logout/route.ts` says it prevents exactly this: *"'sign out' followed by 'sign in' would walk straight back in without asking for a password. That is not a logout."* But revoking a token does **not** end the provider session. Only `/oauth/logout/` (RP-initiated logout) does that. This is a real risk for students on shared college lab computers.

**Fix (dashboard + one registration flag):**
- In the callback, also save `id_token` from the token response in an **httpOnly** cookie.
- In logout: revoke the refresh token as now, then send the browser to
  `{issuer}/oauth/logout/?id_token_hint=<id_token>&client_id=<id>&post_logout_redirect_uri=<app>/login?logged_out=1`.
  The easiest way is for the route to return `{ logoutUrl }` and have the UI navigate there instead of `/login`.
- Register the post-logout URI: `register_oauth_client … --post-logout-redirect-uri https://app.mulearn.org/login?logged_out=1`.
- On `/login`, do **not** auto-redirect when `logged_out=1` is present. Show a "Signed out. Sign in again" button.
- authserver has `OIDC_RP_INITIATED_LOGOUT_ALWAYS_PROMPT=False`, so **there is no confirmation screen only when a valid `id_token_hint` is sent**. That is why the id_token must be stored.

---

<a id="issue-2"></a>
### #2 🔴 Turning the flag on logs everyone out (the docs say it does not)

**Where:** `mulearn-dashboard/src/app/api/auth/refresh/route.ts`, `logout/route.ts`, `src/api/server.ts`, `.env.example`, `login/page.tsx`

**What goes wrong:**
`.env.example` and the login page say: *"Flipping this does NOT log anyone out … Existing sessions keep working."* But the refresh route picks its path **by the flag, not by the token**:

```ts
if (useOidc) { refreshOidcSession(refreshToken) }   // flag on  -> /oauth/token/
else         { refreshAccessTokenServer(refreshToken) } // flag off -> legacy
```

After the flag is turned on, each existing user still has a **legacy** (HS256 JWT) refresh token. At their next access-token expiry (**15 min or less**), it is sent to `/oauth/token/`. authserver answers `invalid_grant`, the route clears all cookies, and the user goes to `/login`. So **every signed-in user is logged out within 15 minutes.** Turning the flag **off** later does the same to every OIDC user, in the other direction.

The same mix-up exists in logout: with the flag on, a legacy token is sent to `/oauth/revoke_token/` (a silent no-op), so the legacy `global_logout` is never written.

**Fix:** choose the path by the **shape of the refresh token**, not by the flag. The flag should only decide where **new** sign-ins go.
- Legacy refresh token = a JWT: 3 dot-separated parts, header `alg: HS256`.
- OIDC refresh token = an opaque random string with no dots.

```ts
const isLegacy = refreshToken.split(".").length === 3;
if (isLegacy) { /* legacy refresh / legacy logout */ }
else          { /* OIDC refresh (needs issuer + clientId) / revoke */ }
```

Apply this in `refresh/route.ts`, `logout/route.ts` and `src/api/server.ts → refreshAndSetToken()`.

---

<a id="issue-3"></a>
### #3 🔴 `redirect_uri` uses the request origin, which on Netlify is the deploy permalink

**Where:** `mulearn-dashboard/src/app/api/auth/oidc/start/route.ts` and `callback/route.ts`

```ts
`${request.nextUrl.origin}/api/auth/oidc/callback`
```

**What goes wrong:**
The dashboard's own comment in `refresh/route.ts` says: *"on Netlify … `request.url` inside a route handler is rebuilt from the deploy permalink — `<deploy-id>--<site>.netlify.app` — not the custom domain."* `request.nextUrl` comes from `request.url`, so in production the `redirect_uri` will very likely be `https://<deploy-id>--<site>.netlify.app/api/auth/oidc/callback`.
- authserver uses **exact matching** of registered redirect URIs, so it rejects the request, and sign-in fails for everyone.
- Even if that URI were registered, the PKCE and state cookies were set on `app.mulearn.org`. The callback would arrive on another origin with no cookies and fail with `signin_expired`.

This does not show up on localhost, where the origin is correct.

**Fix:** add a server env var, for example `OIDC_REDIRECT_URI=https://app.mulearn.org/api/auth/oidc/callback`. Use that exact value in **both** start and callback (they must send the same value), and register that exact value with `register_oauth_client`. Use one value per environment (dev, prod).

---

<a id="issue-4"></a>
### #4 🔴 `/oauth/token/` per-IP rate limit vs the dashboard's server-side refresh

**Where:** `authserver/muauth/security/ratelimit.py` (`"token": (120, 60)`), `authserver/muauth/views/oauth.py`. On the dashboard side: `refresh/route.ts` and `oidc-refresh.ts`.

**What goes wrong:**
The dashboard does the code exchange and every refresh **from its server** (the BFF design, which is correct for security). So to authserver, **every dashboard user shares the dashboard server's IP bucket**: 120 requests per minute.
- Each active user refreshes about every 15 minutes, so the limit is hit at roughly **1,800 active users per server IP**, and sooner at peak times such as exam results or event launches.
- When it hits, authserver returns **429**. `refreshOidcSession` treats any non-2xx as `RefreshFailed`, and the refresh route **clears the cookies and logs the user out**.
- The limit's own comment says it is sized for "a real person, or a classroom behind one NAT address", not a whole BFF.

**Fix (authserver, pick one):**
- Rate-limit only the `authorization_code` grant per IP. The `refresh_token` grant cannot be brute-forced (it is a long random value, and reuse revokes the family).
- Or key the limit by `client_id + IP` and give the dashboard client a much higher limit.
- Or allowlist the dashboard's server egress IPs.

**Fix (dashboard):** treat **429 / 5xx / timeout** from `/oauth/token/` as "try again later". Do **not** clear the session on them. Only `400 invalid_grant` should mean "sign in again".

---

<a id="issue-5"></a>
### #5 🟠 Browser-side refresh cannot work with OIDC

**Where:** `mulearn-dashboard/src/api/refresh.client.ts`, `src/api/client.ts`

**What goes wrong:**
When an API call from the browser gets a 401, `client.ts` calls `refreshAccessToken()`. That function:
1. reads the refresh token with `js-cookie`. The OIDC `refreshToken` cookie is **httpOnly**, so it gets `undefined`.
2. would post to the **legacy** `/api/v1/auth/get-access-token/` anyway.

So it returns `null`, and `client.ts` clears the JS cookies and does `window.location.href = "/login"`. The proxy then bounces `/login` → `/dashboard` → `/api/auth/refresh` → `/dashboard`. The user **loses the page they were on and any unsaved form input**. This happens whenever a tab is idle for more than 15 minutes and the user then clicks something that calls the API.

`src/api/server.ts → refreshAndSetToken()` has the same problem on the server side. It only knows the legacy refresh, and it sets `accessToken` as **httpOnly with 24 h expiry**, which does not match the rest.

**Fix:** add a same-origin JSON endpoint, for example `POST /api/auth/refresh` that returns `{ accessToken }`. It does the refresh on the server (using the token-shape check from [#2](#issue-2)) and sets the cookies. `refresh.client.ts` should call it with `credentials: "include"` and keep its single-flight guard. Make `server.ts` use the same helper.

---

<a id="issue-6"></a>
### #6 🟠 Parallel refreshes can revoke the whole session

**Where:** `mulearn-dashboard/src/app/api/auth/refresh/route.ts` together with authserver's `REFRESH_TOKEN_GRACE_PERIOD_SECONDS: 0`

**What goes wrong:**
authserver rotates refresh tokens with **no grace window**. Its settings comment says: *"Clients must refresh single-flight instead (the dashboard BFF's one same-origin refresh route)."* The dashboard refresh route has **no lock**. If two requests reach it with the same refresh token at the same moment, the second one counts as **reuse**, and authserver revokes the **whole token family** (the user is signed out on every device).

Ways this can happen: Next.js `<Link>` prefetches of protected pages right after the token expires (each one goes through `proxy.ts` → `/api/auth/refresh`), or two tabs refreshing at the same second.

**Fix:**
- In `proxy.ts`, do not redirect **prefetch** requests to `/api/auth/refresh`. Check for the `Next-Router-Prefetch` or `Purpose: prefetch` header and just let them through or return 204.
- Refresh a little **before** expiry (for example, 60 s early), from one place.
- Test this: two tabs, token expired, click both.
- For the long term, consider a short grace window in authserver. That needs a storage decision, because the settings comment explains it conflicts with hashed tokens.

---

<a id="issue-7"></a>
### #7 🟠 A failed sign-in loops back to the provider and the error is never shown

**Where:** `mulearn-dashboard/src/app/(auth)/login/page.tsx`, `oidc/callback/route.ts`

**What goes wrong:**
On any failure the callback sends the user to `/login?error=signin_failed` (or `signin_expired`, `signin_mismatch`, `signin_unavailable`). With the flag on, `/login` **always** redirects straight back to `/api/auth/oidc/start` and ignores `error`. If the error keeps happening (for example `invalid_scope` because the client is not registered as first-party, or a misconfigured client), the browser loops until it shows *"Too many redirects"*, and the real reason is never shown.

**Fix:** in `login/page.tsx`, when `params.error` is set, **do not redirect**. Show a short message ("Sign-in failed. Please try again.") and a **Try again** button that links to `/api/auth/oidc/start`.

---

<a id="issue-8"></a>
### #8 🟠 `/register` still uses the legacy signup when the flag is on

**Where:** `mulearn-dashboard/src/app/(auth)/register/*`

**What goes wrong:**
Only `/login` is switched. `/register` (and the Google sign-up path through `tempToken`) still calls the legacy `RegisterDataAPI` and gets **legacy tokens**. Because of [#2](#issue-2), those new users are logged out within 15 minutes. Each such signup also adds to the backend's `mulearn.legacy_signup` counter, which has to reach zero before the legacy path can be removed.

**Fix:** when `OIDC_ENABLED` is on, send `/register` to `/api/auth/oidc/start?signup=1`. In the start route, when `signup=1` is set, build the usual `/oauth/authorize/?…` URL (with PKCE and state as now), but redirect to
`{issuer}/accounts/signup/?next=<url-encoded /oauth/authorize/?… path>` instead of going to authorize directly.
authserver's signup page already follows a safe `next` after the account is created, so the user comes back through the normal code flow. authserver has no `prompt=create` support, so `next` is the way to do it.

Keep the old form only for when the flag is off.

---

<a id="issue-9"></a>
### #9 🟠 Changing the password signs the user out, but the dashboard does not say so

**Where:** backend `api/dashboard/profile/profile_view.py → ResetPasswordAPI`. Dashboard `src/app/(dashboard)/dashboard/settings/account/change-password-form.tsx`

**What goes wrong:**
- The backend now asks authserver to change the password, and authserver **revokes every session, including the current one**. The dashboard form just resets itself. The user carries on until the next refresh (up to 15 min) and is then suddenly logged out with no reason given.
- If revocation only partly succeeds, the backend returns a **failure** response ("Password changed, but existing sessions could not be signed out…") even though the password **did** change. The dashboard shows it as an error, so the user may think the change failed and try again with the old password.

**Fix:**
- Dashboard: on success, show "Password changed. Please sign in again." and log out (use the fixed logout from [#1](#issue-1)).
- Backend (small): return **success with a warning** for the partial case (as `ResetPasswordConfirmAPI` already does), or add a clear code the dashboard can check.

---

<a id="issue-10"></a>
### #10 🟡 The issuer string must be exactly the same in all three repos

- authserver writes `OIDC_ISSUER` into `iss` on every token.
- The backend checks `iss == OIDC_ISSUER` **exactly**. A trailing `/` in one and not the other means **every new-format token is rejected**.
- The dashboard builds URLs with `new URL("/oauth/token/", issuer)`, which **drops any path** in the issuer. So the issuer must be a **bare origin**, like `https://auth.mulearn.org`.

**Fix:** use the same value everywhere, with no trailing slash and no path. See [Section 10](#10-environment-variables-must-match-across-repos). Optional: have the backend check it at startup against `/.well-known/openid-configuration`.

---

<a id="issue-11"></a>
### #11 🟡 The dashboard client must be registered exactly right

The dashboard sends only `client_id` (no secret) and asks for `mulearn.read mulearn.write`. So its client must be:
```bash
python manage.py register_oauth_client \
  --name "muLearn Dashboard (prod)" \
  --client-id mulearn-dashboard-prod \
  --public \
  --skip-authorization \
  --redirect-uri https://app.mulearn.org/api/auth/oidc/callback \
  --post-logout-redirect-uri "https://app.mulearn.org/login?logged_out=1"
```
- Without `--public`, the token exchange fails, because a confidential client needs a secret.
- Without `--skip-authorization`, the validator **refuses the `mulearn.*` scopes**, and every sign-in fails (and loops, see [#7](#issue-7)).
- Create a **separate client per environment**. The dashboard's `.env.example` says the same.

---

<a id="issue-12"></a>
### #12 ℹ️ Tokens keep working at the backend for up to 15 minutes after "sign out everywhere"

After a password change or admin "revoke sessions", authserver revokes refresh tokens and deletes access-token **rows**. But the backend verifies access tokens **locally** (JWKS), and it does not check `global_logout` for either format. So an access token that was already issued still works at the backend until it expires (**15 min at most**). Legacy and new tokens behave the same way here, so it is consistent. Treat it as a known trade-off and document it for admins ("revocation takes effect within 15 minutes").

---

<a id="issue-13"></a>
### #13 🟡 CORS allowlists: set them per environment

The backend and authserver both default `CORS_ALLOWED_ORIGINS` to `localhost:3000, dev.mulearn.org, mulearn-dashboard.vercel.app, app.mulearn.org`. The dashboard code now talks about **Netlify**, so Netlify deploy previews (`*--<site>.netlify.app`) are **not** in the list. Browser calls from a preview will fail CORS. Set `CORS_ALLOWED_ORIGINS` explicitly in every environment, in both services.

---

<a id="issue-14"></a>
### #14 ℹ️ No admin console UI in the dashboard

The backend has 8 new `/api/v1/dashboard/auth-admin/*` endpoints (connected apps, sign-in policy, login attempts, security posture, revoke sessions), and authserver has the internal API behind them. The dashboard branch has **no screens** for these. See [Section 9](#9-missing-pieces-in-the-dashboard).

---

<a id="issue-15"></a>
### #15 🟡 Forgot-password on the new sign-in page

- The server-rendered `/accounts/login/` page in authserver has **no "Forgot password?" link**.
- authserver's `password/forgot` emails a link to `{AUTH_PUBLIC_URL}/reset-password`. That page only exists in the **separate auth UI**, not in authserver. Until the auth UI is deployed, that link is a 404.
- The dashboard's own `/forgot-password` (backend flow → authserver `password/set/`) still works.

**Fix (short term):** add a "Forgot password?" link on `/accounts/login/` that points to the dashboard's `/forgot-password`, or add a simple reset page in authserver.

---

<a id="issue-16"></a>
### #16 🟡 Dashboard env vars are not validated at build time

`NEXT_PUBLIC_OIDC_ISSUER`, `NEXT_PUBLIC_OIDC_CLIENT_ID` and `OIDC_ENABLED` are read with `process.env` directly. They are not in `config/env.ts` (the t3 `createEnv` schema). If one is missing, you only find out at runtime (`start` returns *500 "Sign-in is not configured"*). Add them to the schema (as optional, but required together when `OIDC_ENABLED=true`), along with the new `OIDC_REDIRECT_URI` from [#3](#issue-3).

---

<a id="issue-17"></a>
### #17 ℹ️ Branch drift

- The dashboard branch is **133 commits behind** `dev`. Only `src/api/endpoints.ts` overlaps, so expect one small conflict.
- The dashboard commit is from **2026-08-25**. The later backend and authserver changes (password ownership, session revocation, admin console, token refactor) came after it. That is where issues #9 and #14 come from.

---

## 9. Missing Pieces in the Dashboard

| Missing | Backend / authserver ready? | Suggested page |
|---|---|---|
| Admin: connected apps (list / create / edit / disable / enable) | ✅ `auth-admin/clients/*` | `/dashboard/admin/auth/clients` |
| Admin: sign-in policy (attempt limit, block minutes) | ✅ `auth-admin/signin-policy/` | `/dashboard/admin/auth/policy` |
| Admin: login attempts log | ✅ `auth-admin/login-attempts/` | `/dashboard/admin/auth/attempts` |
| Admin: security posture | ✅ `auth-admin/security-posture/` | `/dashboard/admin/auth` |
| Admin: revoke a member's sessions | ✅ `auth-admin/sessions/revoke/` | button on the user management page |
| Store `id_token` + RP logout | ✅ `/oauth/logout/` | see [#1](#issue-1) |
| OIDC-aware browser refresh endpoint | ✅ | see [#5](#issue-5) |
| Register → provider signup when flag on | ✅ `/accounts/signup/` | see [#8](#issue-8) |
| "Sign in again" after password change | ✅ | see [#9](#issue-9) |
| Error screen for failed OIDC sign-in | – | see [#7](#issue-7) |

---

## 10. Environment Variables: Must Match Across Repos

| Meaning | authserver | backend | dashboard | Rule |
|---|---|---|---|---|
| Issuer | `OIDC_ISSUER` | `OIDC_ISSUER` | `NEXT_PUBLIC_OIDC_ISSUER` | **Identical.** Bare origin, no trailing `/` |
| Audience | `OIDC_ACCESS_TOKEN_AUDIENCE` (`mulearn-api`) | `OIDC_AUDIENCE` (`mulearn-api`) | – | Identical |
| Shared internal key | `PROTECTED_API_KEY` | `PROTECTED_API_KEY` | – | Identical |
| Backend URL (for provisioning) | `MULEARN_BACKEND_URL` | – | – | Points to the backend |
| authserver internal URL | – | `AUTH_DOMAIN` | – | Direct to authserver; **must not** go through the auth UI proxy |
| Signing key | `OIDC_RSA_PRIVATE_KEY` (+ `_INACTIVE`) | – (uses JWKS) | – | Never shared |
| Client id | registered with `register_oauth_client` | – | `NEXT_PUBLIC_OIDC_CLIENT_ID` | One per environment |
| Redirect URI | registered | – | `OIDC_REDIRECT_URI` *(new, see #3)* | Exact match |
| Flag | – | – | `OIDC_ENABLED` | Turn on in dev first |
| CORS | `CORS_ALLOWED_ORIGINS` | `CORS_ALLOWED_ORIGINS` | – | List every dashboard origin |
| Proxy count | `RATE_LIMIT_TRUSTED_PROXIES` | – | – | 1 = nginx only, 2 = auth UI + nginx |
| Public auth URL | `AUTH_PUBLIC_URL` | – | – | Used in reset emails and the Google redirect |
| Google | `SOCIAL_AUTH_GOOGLE_OAUTH2_*`, `GOOGLE_ALLOWED_CLIENT_IDS` | – | – | Also register `{AUTH_PUBLIC_URL}/accounts/api/google/callback/` in Google Cloud |

---

## 11. Setup & Deploy Order

Do these steps in this order. Each step is safe on its own.

1. **DB:** apply db-scripts `alter-1.89.sql` (OAuth tables, `user.last_login`), and the enum alter for the 6 new `SystemActionLog` action types.
2. **authserver:** deploy with `OIDC_ISSUER`, `OIDC_RSA_PRIVATE_KEY`, `MULEARN_BACKEND_URL`, `PROTECTED_API_KEY` and `CORS_ALLOWED_ORIGINS`. Check:
   - `GET {issuer}/.well-known/openid-configuration` returns the right `issuer`.
   - `GET {issuer}/oauth/.well-known/jwks.json` returns a key.
   - `python manage.py verify_oidc_flow` passes.
3. **backend:** deploy with `OIDC_ISSUER` (identical to authserver), `OIDC_AUDIENCE` and `CORS_ALLOWED_ORIGINS`. Legacy tokens keep working, and nothing changes for users yet.
4. **Register the dashboard client** per environment ([#11](#issue-11)).
5. **Fix dashboard issues #1 – #3 and #5 – #8** (and authserver #4) before going further.
6. **dashboard:** deploy with `NEXT_PUBLIC_OIDC_ISSUER`, `NEXT_PUBLIC_OIDC_CLIENT_ID`, `OIDC_REDIRECT_URI` and `OIDC_ENABLED=false`.
7. Turn on `OIDC_ENABLED=true` in **dev** and run the [test checklist](#12-test-checklist-before-turning-it-on).
8. Turn it on in **prod**. Watch `logs/auth_migration.log` (backend) and `security.log` (authserver, especially `token.reuse_detected` and 429s on `/oauth/token/`).
9. After **7 days of zero** `format=legacy_hs256` and zero `legacy_signup`, remove the legacy code paths.

---

## 12. Test Checklist Before Turning It On

**Sign in / out**
- [ ] Sign in with the flag on, on the **real deployed domain** (not localhost). `redirect_uri` shows `app.mulearn.org` ([#3](#issue-3)).
- [ ] New user signs up on auth.mulearn.org → lands on `/onboarding/interests` (onboarding `INCOMPLETE`).
- [ ] Company user → **not** sent to interests (`exempt: true`).
- [ ] Log out, then click Sign in → **the password is asked for** ([#1](#issue-1)).
- [ ] Sign in with a broken config (wrong client id) → an error screen appears, not a redirect loop ([#7](#issue-7)).

**Migration**
- [ ] Sign in with the flag **off**, turn the flag **on**, wait 16 min, click around → **still signed in** ([#2](#issue-2)).
- [ ] The reverse: sign in with the flag on, turn it off → still signed in.
- [ ] `/register` with the flag on → goes to the provider signup ([#8](#issue-8)).

**Token life**
- [ ] Leave a tab idle for 16 min, then submit a form → the form is **not** lost and you are not bounced ([#5](#issue-5)).
- [ ] Two tabs, token expired, click in both at once → still signed in, and **no** `token.reuse_detected` in `security.log` ([#6](#issue-6)).
- [ ] Role-gated page (e.g. Campus Lead) with an OIDC token → opens (the proxy defers, and the server checks roles).
- [ ] Remove a role in the DB → the next API call is refused right away (roles are not in the token).
- [ ] Load test: a few thousand refreshes per minute from one IP → no 429 ([#4](#issue-4)).

**Password**
- [ ] Change the password in settings → a clear message, then sign in again ([#9](#issue-9)).
- [ ] Forgot password from the dashboard → email → reset works → old sessions are ended.
- [ ] Forgot password from the auth.mulearn.org sign-in page → a link exists and works ([#15](#issue-15)).

**Legacy Google (flag off)**
- [ ] Google sign-in works, and the callback carries `state`.
- [ ] Open the callback URL in another browser → refused ("No sign-in was started in this browser").

**Backend**
- [ ] A new-format token with only `openid profile email` (partner app) → `GET` and `POST` to the backend are refused.
- [ ] Stop authserver for a few minutes → existing new-format tokens still verify at the backend (cached JWKS).
- [ ] Issuer with and without a trailing `/` → confirm all three configs use the same string ([#10](#issue-10)).
