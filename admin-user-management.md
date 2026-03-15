# Admin User Management API

This document covers the **Admin User Management** endpoints — creating, listing, updating, and deleting admin panel accounts, plus self-profile management and audit log retrieval.

Use it together with [admin-permissions.md](./admin-permissions.md) for roles/permissions context.

**Base URL:** `/api/admin`

**Authentication:** All endpoints (except Login) require `Authorization: Bearer <accessToken>` — obtained from `POST /api/admin/auth/login`.

---

## Table of Contents

1. [Authentication](#authentication)
2. [Me (Self-Profile)](#me-self-profile)
3. [Admin Users CRUD](#admin-users-crud)
4. [Audit Logs](#audit-logs)
5. [Postman Collection](#postman-collection)
6. [Business Rules](#business-rules)
7. [Error Responses](#error-responses)

---

## Authentication

### Login

**Endpoint:** `POST /api/admin/auth/login`

Public — no token required.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | String | Yes | Admin account email |
| `password` | String | Yes | Account password |

**Example Request:**
```http
POST /api/admin/auth/login
Content-Type: application/json

{
  "email": "admin@example.com",
  "password": "StrongPass1!"
}
```

**Example Response (200 OK):**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "admin": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "email": "admin@example.com",
    "fullName": "Alice Smith",
    "phone": "+1234567890",
    "avatarUrl": null,
    "status": "ACTIVE",
    "isActive": true,
    "isSuperAdmin": false,
    "lastLoginAt": "2026-03-14T09:00:00Z",
    "roleId": "660e8400-e29b-41d4-a716-446655440001",
    "role": {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "name": "Super Admin",
      "isActive": true,
      "isSuper": true
    },
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-03-15T10:00:00Z"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `401` | Wrong email / password, or account was deleted |
| `403` | Account is inactive or suspended |

> **Note:** `lastLoginAt` is automatically recorded on every successful login.

---

## Me (Self-Profile)

Any authenticated admin can manage their own profile — no extra RBAC permission required.

---

### Get My Profile

**Endpoint:** `GET /api/admin/me`

**Authentication:** Required (any admin JWT)

**Example Request:**
```http
GET /api/admin/me
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Profile retrieved",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "fullName": "Alice Smith",
    "email": "admin@example.com",
    "phone": "+1234567890",
    "avatarUrl": "https://cdn.example.com/avatar.png",
    "status": "ACTIVE",
    "isActive": true,
    "isSuperAdmin": false,
    "lastLoginAt": "2026-03-15T10:00:00Z",
    "roleId": "660e8400-e29b-41d4-a716-446655440001",
    "role": {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "name": "Super Admin",
      "isActive": true,
      "isSuper": true
    },
    "createdAt": "2026-01-01T00:00:00Z",
    "updatedAt": "2026-03-15T10:00:00Z",
    "deletedAt": null
  }
}
```

---

### Update My Profile

**Endpoint:** `PATCH /api/admin/me`

**Authentication:** Required (any admin JWT)

**Request Body** (all optional, at least one required):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fullName` | String | No | Display name (max 255 chars) |
| `phone` | String | No | Phone number (7–50 chars) |
| `avatarUrl` | String | No | Valid URL for profile picture |

**Example Request:**
```http
PATCH /api/admin/me
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "fullName": "Alice J. Smith",
  "phone": "+9876543210"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Profile updated",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "fullName": "Alice J. Smith",
    "phone": "+9876543210",
    "email": "admin@example.com",
    "status": "ACTIVE"
  }
}
```

---

### Change My Password

**Endpoint:** `PATCH /api/admin/me/password`

**Authentication:** Required (any admin JWT)

Requires the current password for verification.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `currentPassword` | String | Yes | The admin's existing password |
| `newPassword` | String | Yes | New password (min 8, max 128 chars) |

**Example Request:**
```http
PATCH /api/admin/me/password
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "currentPassword": "OldPass1!",
  "newPassword": "NewPass2!"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Password changed successfully"
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | `currentPassword` is incorrect |

---

## Admin Users CRUD

All endpoints in this section require the `admin_users` permission resource. See [admin-permissions.md](./admin-permissions.md) for how permissions work.

---

### List Admin Users

**Endpoint:** `GET /api/admin/users`

**Authentication:** Required — `admin_users.read`

Soft-deleted users are never returned. Results are paginated.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `limit` | Integer | No | `20` | Items per page (max 100) |
| `search` | String | No | — | Search across `email`, `fullName`, `phone` (case-insensitive) |
| `roleId` | UUID | No | — | Filter by role ID |
| `status` | String | No | — | Filter by status: `ACTIVE`, `INACTIVE`, `SUSPENDED` |
| `sortBy` | String | No | `createdAt` | Sort field: `createdAt`, `updatedAt`, `email`, `fullName`, `status` |
| `sortOrder` | String | No | `desc` | Sort direction: `asc` or `desc` |

**Example Request:**
```http
GET /api/admin/users?status=ACTIVE&search=alice&sortBy=createdAt&sortOrder=desc&page=1&limit=20
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Admin users retrieved",
  "data": {
    "users": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "fullName": "Alice Smith",
        "email": "alice@example.com",
        "phone": "+1234567890",
        "avatarUrl": null,
        "status": "ACTIVE",
        "isActive": true,
        "isSuperAdmin": false,
        "lastLoginAt": "2026-03-15T10:00:00Z",
        "roleId": "660e8400-e29b-41d4-a716-446655440001",
        "role": {
          "id": "660e8400-e29b-41d4-a716-446655440001",
          "name": "Super Admin",
          "isActive": true,
          "isSuper": true
        },
        "createdAt": "2026-01-01T00:00:00Z",
        "updatedAt": "2026-03-15T10:00:00Z",
        "deletedAt": null
      }
    ],
    "total": 5,
    "page": 1,
    "limit": 20,
    "totalPages": 1
  }
}
```

---

### Create Admin User

**Endpoint:** `POST /api/admin/users`

**Authentication:** Required — `admin_users.create`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `email` | String | Yes | Must be unique |
| `password` | String | Yes | min 8, max 128 chars |
| `fullName` | String | No | Display name |
| `phone` | String | No | Phone number (7–50 chars) |
| `avatarUrl` | String | No | Must be a valid URL |
| `roleId` | UUID | No | Must reference an existing `AdminRole` |
| `isSuperAdmin` | Boolean | No | Defaults to `false` |

**Example Request:**
```http
POST /api/admin/users
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "email": "bob@example.com",
  "password": "SecurePass1!",
  "fullName": "Bob Jones",
  "phone": "+9876543210",
  "roleId": "660e8400-e29b-41d4-a716-446655440001"
}
```

**Example Response (201 Created):**
```json
{
  "message": "Admin user created",
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "fullName": "Bob Jones",
    "email": "bob@example.com",
    "phone": "+9876543210",
    "avatarUrl": null,
    "status": "ACTIVE",
    "isActive": true,
    "isSuperAdmin": false,
    "lastLoginAt": null,
    "roleId": "660e8400-e29b-41d4-a716-446655440001",
    "role": { "id": "...", "name": "Super Admin", "isActive": true, "isSuper": true },
    "createdAt": "2026-03-15T10:30:00Z",
    "updatedAt": "2026-03-15T10:30:00Z",
    "deletedAt": null
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `409` | Email already in use |
| `404` | `roleId` does not exist |

---

### Get Admin User

**Endpoint:** `GET /api/admin/users/:id`

**Authentication:** Required — `admin_users.read`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Example Request:**
```http
GET /api/admin/users/770e8400-e29b-41d4-a716-446655440002
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Admin user retrieved",
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "fullName": "Bob Jones",
    "email": "bob@example.com",
    "status": "ACTIVE",
    "role": { "id": "...", "name": "Super Admin" }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | User not found or soft-deleted |

---

### Update Admin User

**Endpoint:** `PATCH /api/admin/users/:id`

**Authentication:** Required — `admin_users.update`

Updates profile fields only (`fullName`, `phone`, `avatarUrl`). To change role use [Assign Role](#assign-role). To change status use [Toggle Status](#toggle-status).

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Request Body** (all optional, at least one required):

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fullName` | String | No | Display name (max 255 chars) |
| `phone` | String | No | Phone number (7–50 chars) |
| `avatarUrl` | String | No | Valid URL for avatar |

**Example Request:**
```http
PATCH /api/admin/users/770e8400-e29b-41d4-a716-446655440002
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "fullName": "Robert Jones",
  "phone": "+1112223333"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Admin user updated",
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "fullName": "Robert Jones",
    "phone": "+1112223333",
    "email": "bob@example.com",
    "status": "ACTIVE"
  }
}
```

---

### Reset Admin Password

**Endpoint:** `PATCH /api/admin/users/:id/password`

**Authentication:** Required — `admin_users.update`

Resets another admin's password. No current-password verification — the actor's RBAC permission is the gate.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `password` | String | Yes | New password (min 8, max 128 chars) |

**Example Request:**
```http
PATCH /api/admin/users/770e8400-e29b-41d4-a716-446655440002/password
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "password": "ResetPass1!"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Password reset successfully"
}
```

---

### Assign Role

**Endpoint:** `PATCH /api/admin/users/:id/roles`

**Authentication:** Required — `admin_users.update`

Assign or remove a role from an admin user. Pass `null` to remove the role without assigning a new one.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `roleId` | UUID or null | No | Role to assign. `null` removes the current role |

**Example Request — assign:**
```http
PATCH /api/admin/users/770e8400-e29b-41d4-a716-446655440002/roles
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "roleId": "660e8400-e29b-41d4-a716-446655440001"
}
```

**Example Request — remove role:**
```http
PATCH /api/admin/users/770e8400-e29b-41d4-a716-446655440002/roles
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "roleId": null
}
```

**Example Response (200 OK):**
```json
{
  "message": "Role assigned",
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "roleId": "660e8400-e29b-41d4-a716-446655440001",
    "role": { "id": "...", "name": "Super Admin" }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | `roleId` does not exist |

---

### Toggle Status

**Endpoint:** `PATCH /api/admin/users/:id/toggle-status`

**Authentication:** Required — `admin_users.update`

Change an admin account's status. `isActive` is automatically kept in sync.

| Status | isActive | Effect |
|--------|----------|--------|
| `ACTIVE` | `true` | Full access |
| `INACTIVE` | `false` | Cannot log in or call APIs |
| `SUSPENDED` | `false` | Cannot log in or call APIs |

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | String | Yes | `ACTIVE`, `INACTIVE`, or `SUSPENDED` |

**Example Request:**
```http
PATCH /api/admin/users/770e8400-e29b-41d4-a716-446655440002/toggle-status
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "status": "SUSPENDED"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Status updated",
  "data": {
    "id": "770e8400-e29b-41d4-a716-446655440002",
    "status": "SUSPENDED",
    "isActive": false
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Cannot change your own status |
| `400` | Cannot deactivate the last super admin |
| `404` | User not found |

---

### Delete Admin User

**Endpoint:** `DELETE /api/admin/users/:id`

**Authentication:** Required — `admin_users.delete`

Soft-delete — sets `deletedAt`, marks account `INACTIVE`. The record is never returned in list/get queries after deletion. This action cannot be undone via the API.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Example Request:**
```http
DELETE /api/admin/users/770e8400-e29b-41d4-a716-446655440002
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Admin user deleted"
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Cannot delete yourself |
| `400` | Cannot delete the last super admin |
| `404` | User not found |

---

## Audit Logs

### Get User Audit Logs

**Endpoint:** `GET /api/admin/users/:id/audit-logs`

**Authentication:** Required — `audit_logs.read`

Returns a paginated list of audit log entries recording **actions performed by** this admin user.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | UUID | Yes | Admin user ID |

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `limit` | Integer | No | `20` | Items per page (max 100) |

**Example Request:**
```http
GET /api/admin/users/550e8400-e29b-41d4-a716-446655440000/audit-logs?page=1&limit=20
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Audit logs retrieved",
  "data": {
    "logs": [
      {
        "id": "880e8400-e29b-41d4-a716-446655440003",
        "actorAdminId": "550e8400-e29b-41d4-a716-446655440000",
        "action": "ADMIN_USER_CREATE",
        "entityType": "AdminUser",
        "entityId": "770e8400-e29b-41d4-a716-446655440002",
        "meta": {
          "email": "bob@example.com",
          "fullName": "Bob Jones",
          "roleId": "660e8400-e29b-41d4-a716-446655440001"
        },
        "ip": "192.168.1.1",
        "userAgent": "PostmanRuntime/7.36.0",
        "createdAt": "2026-03-15T10:30:00Z"
      },
      {
        "id": "990e8400-e29b-41d4-a716-446655440004",
        "actorAdminId": "550e8400-e29b-41d4-a716-446655440000",
        "action": "ADMIN_USER_STATUS_CHANGE",
        "entityType": "AdminUser",
        "entityId": "770e8400-e29b-41d4-a716-446655440002",
        "meta": {
          "before": { "status": "ACTIVE" },
          "after": { "status": "SUSPENDED" }
        },
        "ip": "192.168.1.1",
        "userAgent": "PostmanRuntime/7.36.0",
        "createdAt": "2026-03-15T11:00:00Z"
      }
    ],
    "total": 2,
    "page": 1,
    "limit": 20,
    "totalPages": 1
  }
}
```

**Audit actions logged for admin users:**

| Action | Trigger |
|--------|---------|
| `ADMIN_USER_CREATE` | New admin user created |
| `ADMIN_USER_UPDATE` | Profile fields updated |
| `ADMIN_USER_DELETE` | Soft-deleted |
| `ADMIN_USER_PASSWORD_RESET` | Password reset by another admin |
| `ADMIN_USER_PASSWORD_CHANGE` | Admin changes own password |
| `ADMIN_USER_ROLE_ASSIGN` | Role assigned or removed |
| `ADMIN_USER_STATUS_CHANGE` | Status toggled |

---

## Postman Collection

Set `baseUrl` to your server URL (e.g. `http://localhost:3000`) and `accessToken` after running Login.

### Auth

| Request | Method | Path | Body |
|---------|--------|------|------|
| **Login** | POST | `{{baseUrl}}/api/admin/auth/login` | `{"email":"...","password":"..."}` |

Copy `accessToken` from the response and set it as a collection variable.

### Me

| Request | Method | Path | Body |
|---------|--------|------|------|
| **Get my profile** | GET | `{{baseUrl}}/api/admin/me` | — |
| **Update my profile** | PATCH | `{{baseUrl}}/api/admin/me` | `{"fullName":"Alice B.","phone":"+447700900000"}` |
| **Change my password** | PATCH | `{{baseUrl}}/api/admin/me/password` | `{"currentPassword":"OldPass1!","newPassword":"NewPass2!"}` |

### Admin Users

| Request | Method | Path | Body |
|---------|--------|------|------|
| **List users** | GET | `{{baseUrl}}/api/admin/users` | — |
| **Create user** | POST | `{{baseUrl}}/api/admin/users` | See Create body below |
| **Get user** | GET | `{{baseUrl}}/api/admin/users/{{userId}}` | — |
| **Update user** | PATCH | `{{baseUrl}}/api/admin/users/{{userId}}` | `{"fullName":"New Name"}` |
| **Reset password** | PATCH | `{{baseUrl}}/api/admin/users/{{userId}}/password` | `{"password":"NewPass1!"}` |
| **Assign role** | PATCH | `{{baseUrl}}/api/admin/users/{{userId}}/roles` | `{"roleId":"{{roleId}}"}` |
| **Toggle status** | PATCH | `{{baseUrl}}/api/admin/users/{{userId}}/toggle-status` | `{"status":"SUSPENDED"}` |
| **Delete user** | DELETE | `{{baseUrl}}/api/admin/users/{{userId}}` | — |
| **Get audit logs** | GET | `{{baseUrl}}/api/admin/users/{{userId}}/audit-logs` | — |

**Create user body:**
```json
{
  "email": "bob@example.com",
  "password": "SecurePass1!",
  "fullName": "Bob Jones",
  "phone": "+9876543210",
  "roleId": "{{roleId}}"
}
```

> **Headers for all requests (except Login):**
> ```
> Authorization: Bearer {{accessToken}}
> Content-Type: application/json
> ```

---

## Business Rules

| Rule | Enforced at |
|------|-------------|
| Cannot delete yourself | `DELETE /api/admin/users/:id` |
| Cannot deactivate / delete the last super admin | `PATCH .../toggle-status` and `DELETE /api/admin/users/:id` |
| Soft-deleted users are excluded from all list/get queries | Service layer (`deletedAt: null` filter) |
| Soft-deleted admins cannot log in or use the API | Login controller + `adminAuth` middleware |
| Email must be unique across non-deleted admins | `POST /api/admin/users` |
| `roleId` must reference an existing role | `POST /api/admin/users` and `PATCH .../roles` |

---

## Error Responses

| Status | Meaning |
|--------|---------|
| `401` | Missing or invalid/expired token; or admin was deleted |
| `403` | Account is inactive / suspended; or RBAC permission denied |
| `404` | Admin user or role not found |
| `409` | Email already in use |
| `400` | Business rule violation (see per-endpoint table) |
| `422` | Validation error — response includes field-level detail |

**Validation error example (422):**
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "email": "\"email\" must be a valid email",
    "password": "\"password\" length must be at least 8 characters long"
  }
}
```

---

## Related Docs

- [admin-permissions.md](./admin-permissions.md) — RBAC, roles, and permission resources.
- [admin.md](./admin.md) — App-user management, moderation, and analytics.
