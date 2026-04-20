# Legal Documents API

Manages **Privacy Policy** and **Terms of Service** content for both the user app and admin app.  
Each document is stored as a single row identified by `type + app`. Updating a document overwrites the existing row (upsert).

---

## Database Model

| Column       | Type        | Description                              |
|-------------|-------------|------------------------------------------|
| `id`         | UUID        | Primary key                              |
| `type`       | enum        | `PRIVACY_POLICY` \| `TERMS_OF_SERVICE`   |
| `app`        | enum        | `USER` \| `ADMIN`                        |
| `content`    | Text        | Full document content                    |
| `version`    | String(20)  | Version label e.g. `"1.0"`, `"2024-01"` |
| `updated_by` | UUID (FK)   | Admin who last updated it                |
| `created_at` | Timestamptz |                                          |
| `updated_at` | Timestamptz |                                          |

**Unique constraint:** `(type, app)` — one row per document per app.

---

## Endpoints

### 1. Get User App — Privacy Policy

```
GET /api/legal/user/privacy-policy
```

**Auth:** None  
**Response `200`:**
```json
{
  "message": "Privacy Policy",
  "data": {
    "content": "...",
    "version": "1.0",
    "updatedAt": "2026-04-20T10:00:00.000Z"
  }
}
```

---

### 2. Get User App — Terms of Service

```
GET /api/legal/user/terms-of-service
```

**Auth:** None  
**Response `200`:** same shape as above.

---

### 3. Get Admin App — Privacy Policy

```
GET /api/legal/admin-app/privacy-policy
```

**Auth:** None  
**Response `200`:** same shape as above.

---

### 4. Get Admin App — Terms of Service

```
GET /api/legal/admin-app/terms-of-service
```

**Auth:** None  
**Response `200`:** same shape as above.

---

### 5. Create / Update a Legal Document

```
PUT /api/legal/admin
```

**Auth:** Admin JWT (`Authorization: Bearer <token>`)  
**Body:**

| Field     | Type   | Required | Notes                                    |
|-----------|--------|----------|------------------------------------------|
| `type`    | string | yes      | `PRIVACY_POLICY` or `TERMS_OF_SERVICE`   |
| `app`     | string | yes      | `USER` or `ADMIN`                        |
| `content` | string | yes      | Full document text                       |
| `version` | string | yes      | Max 20 chars, e.g. `"1.0"`, `"2026-04"` |

**Request example:**
```json
{
  "type": "PRIVACY_POLICY",
  "app": "USER",
  "content": "Your full privacy policy text here...",
  "version": "1.0"
}
```

**Response `200`:**
```json
{
  "message": "Legal document saved",
  "data": {
    "id": "uuid",
    "type": "PRIVACY_POLICY",
    "app": "USER",
    "content": "...",
    "version": "1.0",
    "updatedBy": "admin-uuid",
    "createdAt": "2026-04-20T10:00:00.000Z",
    "updatedAt": "2026-04-20T10:00:00.000Z"
  }
}
```

---

## Error Responses

| Status | Message                    | Cause                          |
|--------|----------------------------|--------------------------------|
| `400`  | Validation error           | Missing or invalid body fields |
| `401`  | Authorization is required  | Missing admin token            |
| `404`  | Privacy Policy not found   | Document not yet created       |
| `404`  | Terms of Service not found | Document not yet created       |

---

## Document Matrix

There are 4 independent documents — all must be seeded via `PUT /api/legal/admin` before the GET endpoints return data:

| `type`              | `app`   | GET endpoint                            |
|---------------------|---------|-----------------------------------------|
| `PRIVACY_POLICY`    | `USER`  | `GET /api/legal/user/privacy-policy`    |
| `TERMS_OF_SERVICE`  | `USER`  | `GET /api/legal/user/terms-of-service`  |
| `PRIVACY_POLICY`    | `ADMIN` | `GET /api/legal/admin-app/privacy-policy`   |
| `TERMS_OF_SERVICE`  | `ADMIN` | `GET /api/legal/admin-app/terms-of-service` |

---

## Quick Seed (curl)

```bash
ADMIN_TOKEN="your_admin_jwt"

# User app — Privacy Policy
curl -X PUT http://localhost:3000/api/legal/admin \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"PRIVACY_POLICY","app":"USER","content":"...","version":"1.0"}'

# User app — Terms of Service
curl -X PUT http://localhost:3000/api/legal/admin \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"TERMS_OF_SERVICE","app":"USER","content":"...","version":"1.0"}'

# Admin app — Privacy Policy
curl -X PUT http://localhost:3000/api/legal/admin \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"PRIVACY_POLICY","app":"ADMIN","content":"...","version":"1.0"}'

# Admin app — Terms of Service
curl -X PUT http://localhost:3000/api/legal/admin \
  -H "Authorization: Bearer $ADMIN_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"TERMS_OF_SERVICE","app":"ADMIN","content":"...","version":"1.0"}'
```

---

## File Structure

```
src/modules/legal/
├── legal.router.js
└── controller/
    ├── legal.controller.js
    └── legal.validation.js
```
