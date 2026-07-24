# Interest Group Cover / Icon Image API — Manual Test Guide

Base path: `/api/v1/dashboard/ig/`

Auth: `Authorization: Bearer <accessToken>` header on every request.
- Upload/replace/remove require the caller to be an `Admin`, the platform-wide `IG Lead`, or that specific IG's `<CODE> IGLead` role (checked via `_can_manage_ig`) — i.e. the IG's own lead can manage their group's images.

## What changed

- **New:** an Interest Group can now have a **cover image**, uploaded as an image file and stored the same way user profile cover pics and impact project images are (`FileSystemStorage`, served from `/muback-media/…`) — **not** as a DB column.
- **Changed:** the old `icon` field (a short emoji/code string, max 10 chars) is superseded by an **icon image** file upload, stored the same way. `icon` is no longer accepted or required on Create (`POST /api/v1/dashboard/ig/`), Update (`PUT /api/v1/dashboard/ig/{pk}/`, `PATCH /api/v1/dashboard/ig/get/{pk}/`), or the IG Request flow (`POST /api/v1/dashboard/ig/request/`). The legacy `icon` DB column is kept (now nullable) for backward compatibility with existing rows and is still returned read-only in responses, but new/updated IGs should use `icon-image/` instead.
- `InterestGroupSerializer` responses (list/get/list-public/CSV) now include two new read-only fields: `cover_image` and `icon_image` — each is a full URL or `null` if nothing has been uploaded yet.

## DB change

See `alter-scripts/alter-1.65.sql`: makes `interest_group.icon` nullable. No new columns/tables — image files live under `MEDIA_ROOT`, not the DB.

---

## 1. Upload / replace the cover image

`POST /api/v1/dashboard/ig/{pk}/cover-image/`
`Content-Type: multipart/form-data`

**Form fields**
| key | value |
|---|---|
| `image` | (file) a `.png`/`.jpg` under 5 MB |

**Response 200**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": { "cover_image": "https://<BE_DOMAIN_NAME>/muback-media/interest_group/cover/<ig_id>.png" }
}
```

**Error cases:**
- IG not found → `400` `"Interest Group Does Not Exist"`
- Not admin/IG-lead/that IG's lead → `400` `"You do not have permission to manage this Interest Group"`
- No file attached → `400` `"No image provided"`
- Non-image file (by declared `Content-Type`) → `400` `"Expected an image file"`
- File over 5 MB → `400` `"Image must be under 5 MB"`
- File's actual bytes aren't a decodable image (e.g. a spoofed `Content-Type`, corrupted file) → `400` `"Invalid or corrupted image file"`

---

## 2. Remove the cover image

`DELETE /api/v1/dashboard/ig/{pk}/cover-image/`

No body.

**Response 200**
```json
{ "hasError": false, "statusCode": 200, "message": { "general": ["Image removed successfully"] }, "response": {} }
```

**Error cases:**
- IG not found → `400` `"Interest Group Does Not Exist"`
- Not admin/IG-lead/that IG's lead → `400` `"You do not have permission to manage this Interest Group"`
- Nothing uploaded yet → `400` `"No image found"`

---

## 3. Upload / replace the icon image

`POST /api/v1/dashboard/ig/{pk}/icon-image/`
`Content-Type: multipart/form-data`

Same rules as the cover image (5 MB max, `image/*` content-type), stored at a separate path so an IG can have both a cover and an icon at once.

**Form fields**
| key | value |
|---|---|
| `image` | (file) a `.png`/`.jpg` under 5 MB |

**Response 200**
```json
{
  "hasError": false,
  "statusCode": 200,
  "message": { "general": [] },
  "response": { "icon_image": "https://<BE_DOMAIN_NAME>/muback-media/interest_group/icon/<ig_id>.png" }
}
```

**Error cases:** same as cover image upload (see above).

---

## 4. Remove the icon image

`DELETE /api/v1/dashboard/ig/{pk}/icon-image/`

No body. Same response/error shape as removing the cover image (see #2).

---

## 5. Effect on existing IG responses

Any endpoint returning `InterestGroupSerializer` data (`GET /api/v1/dashboard/ig/`, `GET /api/v1/dashboard/ig/get/{pk}/`, `GET /api/v1/dashboard/ig/list/`, `GET /api/v1/dashboard/ig/csv/`, `GET /api/v1/dashboard/ig/request/`) now includes:

```json
{
  "icon": "🤖",
  "icon_image": "https://<BE_DOMAIN_NAME>/muback-media/interest_group/icon/<ig_id>.png",
  "cover_image": "https://<BE_DOMAIN_NAME>/muback-media/interest_group/cover/<ig_id>.png"
}
```

`icon_image`/`cover_image` are `null` until something is uploaded. `icon` is `null`/absent for any IG created after this change, since it's no longer settable through Create/Update/Request.

---

## 6. Other endpoints now returning `cover_image`/`icon_image`

The same two read-only fields were also added to every other serializer that already surfaced an IG's identity, so downstream apps (campus, events) don't need a separate lookup to render an image:

| Endpoint | Serializer | File |
|---|---|---|
| `GET /api/v1/dashboard/campus/ig-chapters/` | `CampusIGChapterListSerializer` — as `ig_cover_image` / `ig_icon_image` (alongside the existing `ig_icon`) | `api/dashboard/campus/serializers.py` |
| Campus IG listing (`CampusIGListSerializer`) | adds `cover_image`, `icon_image` next to `id`/`name`/`code`/`campus_member_count` | `api/dashboard/campus/serializers.py` |
| Events' `MinimalIGSerializer` (IG tagging on events) | adds `cover_image`, `icon_image` next to `id`/`name`/`icon` | `api/dashboard/events/serializers.py` |

**Not changed** (no image field added): `MentorIGDropdownSerializer`, profile's `UserIgListSerializer`, register's `AreaOfInterestAPISerializer` — these are plain `id`/`name` dropdown lists that don't render an icon today, so this is out of scope for now. Also not changed: raw `.values('id','name','icon',...)` dict-based queries in `events/meta_views.py`, `dash_task_view.py`, `mentor/task_views.py` — `.values()` can't read a Python `@property`, so surfacing images there needs a follow-up (switch to a serializer, or annotate) rather than this change.

The company IG-request form (`POST/PATCH /api/v1/dashboard/ig/request/`) needed no separate fix — it already runs through the same `InterestGroupRequestSerializer` covered in section "What changed" above. No standalone campus-level IG *creation* endpoint exists — `campus/ig-chapters/` links an existing IG to a campus (`CampusIGChapter`, its own unrelated `icon_link` URL field) rather than creating an `InterestGroup` row.

---

## Quick curl examples

```bash
# Upload cover image
curl -X POST -H "Authorization: Bearer $TOKEN" \
  -F "image=@/path/to/cover.png" \
  http://localhost:8000/api/v1/dashboard/ig/<ig_id>/cover-image/

# Upload icon image
curl -X POST -H "Authorization: Bearer $TOKEN" \
  -F "image=@/path/to/icon.png" \
  http://localhost:8000/api/v1/dashboard/ig/<ig_id>/icon-image/

# Remove cover image
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  http://localhost:8000/api/v1/dashboard/ig/<ig_id>/cover-image/

# Remove icon image
curl -X DELETE -H "Authorization: Bearer $TOKEN" \
  http://localhost:8000/api/v1/dashboard/ig/<ig_id>/icon-image/
```
