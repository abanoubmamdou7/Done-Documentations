# [Module Name] API Documentation

## Overview
[Brief description of what this module does]

**Base URL:** `/api/[module-name]`

---

## Table of Contents
1. [Authentication](#authentication)
2. [Endpoints](#endpoints)
   - [Endpoint 1](#endpoint-1)
   - [Endpoint 2](#endpoint-2)
3. [Error Responses](#error-responses)
4. [Postman Collection](#postman-collection)

---

## Authentication

All endpoints in this module require authentication unless otherwise specified.

**Header:**
```
Authorization: Bearer <your-jwt-token>
```

---

## Endpoints

### Endpoint 1

**Description:** [What this endpoint does]

**Endpoint:** `[METHOD] /api/[module-name]/[path]`

**Authentication:** Required / Optional

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `paramName` | Type | Yes/No | Description |

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `queryParam` | Type | Yes/No | Default | Description |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fieldName` | Type | Yes/No | Description |

**Example Request:**
```http
[METHOD] /api/[module-name]/[path]
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "fieldName": "value"
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "...",
    "field": "value"
  }
}
```

**Error Responses:**

**400 Bad Request:**
```json
{
  "message": "Error message",
  "status_code": 400
}
```

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

**404 Not Found:**
```json
{
  "message": "Resource not found",
  "status_code": 404
}
```

---

## Postman Collection

### Environment Variables
- `base_url`: `http://localhost:3000` (or your server URL)
- `auth_token`: Your JWT token

### Example Request

**Request:**
```
[METHOD] {{base_url}}/api/[module-name]/[path]
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "fieldName": "value"
}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has success field", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('success');
    pm.expect(jsonData.success).to.be.true;
});
```

---

## Notes

- [Any important notes about this module]
- [Usage tips]
- [Common issues and solutions]

