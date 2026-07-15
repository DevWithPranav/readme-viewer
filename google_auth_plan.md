# Redesign Google Sign-In / Sign-Up Flow — Final Plan

## Decisions Made

| Question | Decision |
|---|---|
| **Password for Google users** | `NULL` — no password is set. Django standard. User can set one later via "Forgot Password". |
| **How tokens are issued after Google sign-up** | Use the already-existing `TokenVerificationAPI` (`/token-verification/<user_id>/`) with `PROTECTED_API_KEY` — no password needed. |
| **Registration form for Google sign-ups** | Email pre-filled + read-only. Password field hidden. All other fields normal. |

---

## Key Insight — The Token Problem With NULL Password

The existing post-registration flow in mulearnbackend is:

```python
password = request.data["user"]["password"]
get_auth_token(user.muid, password)   # calls auth server with muid + password
```

With `password = None`, this call fails because the auth server does:
```python
if user.password and check_password(password, user.password):  # user.password is None → False
```

**Solution:** For Google sign-ups, skip `get_auth_token()` and instead call the **already-existing** `TokenVerificationAPI`:

```
POST /api/v1/auth/token-verification/<user_id>/
Headers: protectionKey: <PROTECTED_API_KEY>
→ Returns { accessToken, refreshToken } without needing any password
```

This endpoint already exists, is already protected, and was designed for exactly this internal use case.

---

## How It Works

```
Frontend                  Auth Server                 Google          mulearnbackend
   |                           |                        |                   |
   |-- GET /signin-with-google/ ->|                      |                   |
   |<-- { redirect_url } --------|                      |                   |
   |                           |                        |                   |
   |-- User goes to Google ----------------------->|                        |
   |<-- Redirect to /callback?code=... ------------|                        |
   |                           |                        |                   |
   |-- GET /google/login/callback/?code=... ->|          |                   |
   |                           |-- exchange code ------->|                   |
   |                           |<-- access_token --------|                   |
   |                           |-- GET /userinfo ------->|                   |
   |                           |<-- { email, name } -----|                   |
   |                           |                                             |
   |                           |-- DB lookup by email                        |
   |                           |                                             |
   |         [EXISTING USER]                                                 |
   |                           |-- generate JWT                              |
   |<-- { accessToken, refreshToken, isNewUser: false } ---|                 |
   |                           |                                             |
   |         [NEW USER]                                                      |
   |                           |-- generate tempToken (signed JWT, 15 min)   |
   |<-- { tempToken, isNewUser: true, email, fullName } ---|                 |
   |                           |                                             |
   | [Frontend shows registration form]                                      |
   |   - email: pre-filled, read-only                                        |
   |   - fullName: pre-filled, editable                                      |
   |   - password: HIDDEN                                                    |
   |   - district, dob, gender, referral: normal                             |
   |                           |                                             |
   |-- POST /api/v1/user/register/ { tempToken, district, ... } ----------->|
   |                           |<-- POST /google/verify-temp-token/ ---------|
   |                           |-- returns { email, fullName } ------------->|
   |                           |   user created (password = NULL)            |
   |                           |   Wallet, Socials, UserSettings created     |
   |                           |                                             |
   |                           |<-- POST /token-verification/<user_id>/ -----|
   |                           |-- returns { accessToken, refreshToken } --->|
   |<-- { accessToken, refreshToken, data: { muid, email, ... } } ------<|  |
```

---

## Proposed Changes

---

### Part 1 — Auth Server

#### [MODIFY] [views.py](file:///c:/Users/prana/Desktop/internship/authserver/muauth/views.py)

**1a. Add helper function `generate_google_temp_token`:**

```python
def generate_google_temp_token(email: str, full_name: str) -> str:
    """
    Short-lived signed JWT (15 min) carrying Google-verified identity.
    Used during sign-up so mulearnbackend can trust the email without
    the user being able to spoof it.
    """
    now = get_current_utc_time()
    payload = {
        "email": email,
        "full_name": full_name,
        "tokenType": "google_signup",
        "iat": int(now.timestamp()),
        "exp": int((now + timedelta(minutes=15)).timestamp()),
    }
    return jwt.encode(payload, decouple.config("SECRET_KEY"), algorithm="HS256")
```

---

**1b. Update new-user branch in `GoogleLoginCallbackAPIView`:**

```diff
  if not user:
      first_name = user_info.get("given_name", "")
      full_name = user_info.get("name", first_name).strip() or email.split("@")[0]
-     return CustomResponse(
-         general_message="New user",
-         response={
-             "isNewUser": True,
-             "email": email,
-             "fullName": full_name,
-         },
-     ).get_success_response()

+     temp_token = generate_google_temp_token(email=email, full_name=full_name)
+     return CustomResponse(
+         general_message="New user",
+         response={
+             "isNewUser": True,
+             "tempToken": temp_token,
+             "email": email,        # for pre-filling the UI only
+             "fullName": full_name, # for pre-filling the UI only
+         },
+     ).get_success_response()
```

---

**1c. Same change in `GoogleMobileAuthAPIView`:**

```diff
  if not user:
      first_name = token_info.get("given_name", "")
      full_name = token_info.get("name", first_name).strip() or email.split("@")[0]
-     return CustomResponse(
-         general_message="New user",
-         response={
-             "isNewUser": True,
-             "email": email,
-             "fullName": full_name,
-         },
-     ).get_success_response()

+     temp_token = generate_google_temp_token(email=email, full_name=full_name)
+     return CustomResponse(
+         general_message="New user",
+         response={
+             "isNewUser": True,
+             "tempToken": temp_token,
+             "email": email,
+             "fullName": full_name,
+         },
+     ).get_success_response()
```

---

**1d. Add new internal endpoint `GoogleTempTokenVerifyAPIView`:**

```python
class GoogleTempTokenVerifyAPIView(APIView):
    """
    Called internally by mulearnbackend during Google sign-up to validate
    the tempToken and get the Google-verified email and name.
    Protected by PROTECTED_API_KEY — not callable by external clients.
    """
    def post(self, request):
        protection_key = request.headers.get("protectionKey")
        if not protection_key or protection_key != config("PROTECTED_API_KEY"):
            return CustomResponse(general_message="Invalid Key").get_failure_response()

        temp_token = request.data.get("tempToken")
        if not temp_token:
            return CustomResponse(
                general_message="tempToken is required"
            ).get_failure_response()

        try:
            payload = jwt.decode(
                temp_token,
                decouple.config("SECRET_KEY"),
                algorithms=["HS256"],
            )
        except jwt.ExpiredSignatureError:
            return CustomResponse(
                general_message="Google session expired. Please sign in with Google again."
            ).get_failure_response()
        except Exception:
            return CustomResponse(
                general_message="Invalid temp token."
            ).get_failure_response()

        if payload.get("tokenType") != "google_signup":
            return CustomResponse(
                general_message="Invalid token type."
            ).get_failure_response()

        return CustomResponse(
            response={
                "email": payload["email"],
                "fullName": payload["full_name"],
            }
        ).get_success_response()
```

---

#### [MODIFY] [urls.py](file:///c:/Users/prana/Desktop/internship/authserver/muauth/urls.py)

```diff
  path('google/login/callback/', views.GoogleLoginCallbackAPIView.as_view(), name='google_login_callback'),
+ path('google/verify-temp-token/', views.GoogleTempTokenVerifyAPIView.as_view(), name='google_verify_temp_token'),
```

---

### Part 2 — mulearnbackend

#### [MODIFY] [register_views.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/register_views.py)

**Full updated `RegisterDataAPI.post`:**

```python
class RegisterDataAPI(APIView):
    def post(self, request):
        data = request.data.copy()
        data = {key: value for key, value in data.items() if value}

        is_google_signup = False
        temp_token = data.pop("tempToken", None)

        if temp_token:
            # --- Google Sign-Up path ---
            # Verify the tempToken with the auth server.
            # This proves the email was authenticated by Google.
            AUTH_DOMAIN = config("AUTH_DOMAIN")
            PROTECTED_API_KEY = config("PROTECTED_API_KEY")

            verify_resp = requests.post(
                f"{AUTH_DOMAIN}/api/v1/auth/google/verify-temp-token/",
                data={"tempToken": temp_token},
                headers={"protectionKey": PROTECTED_API_KEY},
            )
            if verify_resp.status_code != 200:
                return CustomResponse(
                    general_message="Invalid or expired Google session. Please sign in with Google again."
                ).get_failure_response()

            google_data = verify_resp.json().get("response", {})
            user_data = dict(data.get("user", {}))

            # These values come from Google — cannot be overridden by the client
            user_data["email"] = google_data["email"]
            user_data["full_name"] = google_data["fullName"]
            user_data.pop("password", None)  # no password for Google users

            data["user"] = user_data
            is_google_signup = True
            # --- End Google Sign-Up path ---

        create_user = serializers.RegisterSerializer(
            data=data, context={"request": request}
        )
        if not create_user.is_valid():
            return CustomResponse(message=create_user.errors).get_failure_response()

        user = create_user.save()
        cache.set(f"db_user_{user.muid}", user, timeout=60)

        if is_google_signup:
            # NULL password — cannot use muid+password to get tokens.
            # Use the internal token-verification endpoint instead.
            res_data = get_auth_token_by_id(user.id)
        else:
            password = request.data["user"]["password"]
            cache.set(f"flag_register_{user.muid}", True, timeout=5)
            res_data = get_auth_token(user.muid, password)

        response_data = serializers.UserDetailSerializer(user, many=False).data

        send_email.delay(
            response_data,
            "YOUR TICKET TO µFAM IS HERE!",
            ["user_registration.html"],
        )

        res_data["data"] = response_data
        return CustomResponse(response=res_data).get_success_response()
```

---

#### [MODIFY] [register_helper.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/register_helper.py)

Add a new helper `get_auth_token_by_id` that uses `TokenVerificationAPI` instead of `user-authentication`:

```python
def get_auth_token_by_id(user_id):
    """
    Gets JWT tokens for a user by their ID using the internal
    token-verification endpoint. Used for Google sign-ups where
    no password is set (NULL).
    """
    AUTH_DOMAIN = decouple.config("AUTH_DOMAIN")
    PROTECTED_API_KEY = decouple.config("PROTECTED_API_KEY")

    response = requests.post(
        f"{AUTH_DOMAIN}/api/v1/auth/token-verification/{user_id}/",
        headers={"protectionKey": PROTECTED_API_KEY},
    )
    response = response.json()
    if response.get("statusCode") != 200:
        raise CustomException(response.get("message"))

    return response.get("response")
```

---

#### [MODIFY] [serializers.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/serializers.py) — `UserSerializer`

Make `password` optional so it doesn't fail validation when not provided for Google sign-ups:

```diff
  class UserSerializer(serializers.ModelSerializer):
      ...
      class Meta:
          model = User
          fields = [
              "full_name",
              "email",
-             # "mobile",
              "password",
+             # "mobile",
              "dob",
              "gender",
              "role",
              "district",
              "area_of_interest",
          ]
+         extra_kwargs = {
+             "password": {"required": False, "allow_null": True},
+         }
```

And in `UserSerializer.create()`, skip hashing if password is None:

```diff
- password = validated_data.pop("password")
- hashed_password = make_password(password)
- validated_data["password"] = hashed_password

+ password = validated_data.pop("password", None)
+ validated_data["password"] = make_password(password) if password else None
```

---

## Full Summary of All File Changes

### Auth Server

| File | Change |
|---|---|
| [views.py](file:///c:/Users/prana/Desktop/internship/authserver/muauth/views.py) | Add `generate_google_temp_token()`; update new-user branch in `GoogleLoginCallbackAPIView` + `GoogleMobileAuthAPIView`; add `GoogleTempTokenVerifyAPIView` |
| [urls.py](file:///c:/Users/prana/Desktop/internship/authserver/muauth/urls.py) | Add `google/verify-temp-token/` route |

### mulearnbackend

| File | Change |
|---|---|
| [register_views.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/register_views.py) | Handle `tempToken` in `RegisterDataAPI.post`; call `get_auth_token_by_id()` for Google sign-ups |
| [register_helper.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/register_helper.py) | Add `get_auth_token_by_id()` helper |
| [serializers.py](file:///C:/Users/prana/Desktop/company_v2/mulearnbackend/api/register/serializers.py) | Make `password` optional in `UserSerializer`; skip `make_password()` when password is `None` |

---

## Frontend Summary

| Scenario | Behaviour |
|---|---|
| Google callback → `isNewUser: false` | Store tokens, go to dashboard. Done. |
| Google callback → `isNewUser: true` | Show registration form: email pre-filled + read-only, password field hidden, all other fields normal. Submit with `tempToken` as hidden field. |

---

## Verification Checklist

- [ ] New Google user → `isNewUser: true` + `tempToken` returned (no `accessToken`)
- [ ] mulearnbackend register with valid `tempToken` → user created, `password = NULL` in DB
- [ ] DB: `Wallet`, `Socials`, `UserSettings`, `UserRoleLink` all created normally
- [ ] JWT tokens returned to frontend via `TokenVerificationAPI`
- [ ] Existing Google user → `isNewUser: false` + tokens returned directly
- [ ] Expired `tempToken` → clear error, user prompted to sign in with Google again
- [ ] Tampered `tempToken` → rejected by JWT signature check
- [ ] Normal (non-Google) registration → completely unchanged behaviour
