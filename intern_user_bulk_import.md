# Intern Management - Bulk Import API Documentation

This document outlines the usage of the endpoints for bulk importing users as interns to specific guilds.

---

## 1. Download Bulk Import Template

Downloads an empty `.xlsx` file containing the required headers for the bulk import process.

- **URL**: `/api/v1/dashboard/manage-interns/interns/import/template/`
- **Method**: `GET`
- **Authentication**: Required
- **Role Required**: `Admin`

### Response
- **Content-Type**: `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
- **Body**: An Excel file named `intern_bulk_import_template.xlsx`.
- **Columns Included**:
  - `muid`: The MuID of the user to assign as an intern.
  - `guild`: The name of the guild the intern will belong to.

---

## 2. Bulk Import Interns

Upload the filled `.xlsx` file to bulk assign users as interns. The system processes each row independently. The user will be assigned the `Intern` role by default.

- **URL**: `/api/v1/dashboard/manage-interns/interns/import/`
- **Method**: `POST`
- **Authentication**: Required
- **Role Required**: `Admin`
- **Content-Type**: `multipart/form-data`

### Request Body
| Key | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `excel_file` | File (.xlsx) | Yes | The Excel file containing the `muid` and `guild` columns. |

### Edge Cases Handled
- **Invalid File**: If the file is not a valid `.xlsx` or is corrupted, the request fails entirely (400 Bad Request).
- **Missing Headers**: If `muid` or `guild` headers are missing, the request fails entirely.
- **Missing Values**: If a row has an empty `muid` or `guild`, the row is skipped and logged as a failure.
- **User Not Found**: If the `muid` does not match an existing user, the row is skipped.
- **Already an Intern**: If the user is already assigned as an intern, the row is skipped.
- **Duplicates in File**: If the same `muid` appears multiple times in the upload, the first valid occurrence succeeds, and subsequent ones fail.
- **Empty Rows**: Completely empty rows are silently ignored.

### Success Response
- **Status Code**: `200 OK`
- **Content-Type**: `application/json`

```json
{
    "hasError": false,
    "statusCode": 200,
    "message": {
        "general": [
            "Bulk intern import completed."
        ]
    },
    "response": {
        "success_count": 5,
        "failed_count": 2,
        "failed_rows": [
            {
                "row": 3,
                "muid": "user@mulearn",
                "reason": "No user found with this muid."
            },
            {
                "row": 6,
                "muid": "existing@mulearn",
                "reason": "User is already onboarded as an intern."
            }
        ]
    }
}
```

### Error Response (Validation Failures)
- **Status Code**: `400 Bad Request`

```json
{
    "hasError": true,
    "statusCode": 400,
    "message": {
        "general": [
            "Missing required column(s): guild. Expected headers: ['muid', 'guild']."
        ]
    },
    "response": {}
}
```
