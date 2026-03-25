# Admin Reports & Moderation API

This document covers the **Admin Reports Management** endpoints — listing, filtering, and reviewing user-submitted reports for content moderation. Reports can target posts, comments, profiles, or stories.

**Base URL:** `/api/admin`

**Authentication:** All endpoints require `Authorization: Bearer <accessToken>` — obtained from `POST /api/admin/auth/login`.

**RBAC Resource Key:** `reports`

---

## Table of Contents

1. [List Reports](#list-reports)
2. [Review Report](#review-report)
3. [User-Facing Endpoints](#user-facing-endpoints)
   - [Submit Report](#submit-report)
   - [Get My Reports](#get-my-reports)
4. [Enums & Schema Reference](#enums--schema-reference)
5. [Business Rules](#business-rules)
6. [Error Responses](#error-responses)

---

## List Reports

**Endpoint:** `GET /api/admin/reports`

**Authentication:** Required — `reports.read`

Returns a paginated list of all reports with optional status and type filters. Includes the reporter, reported content, and reviewer details. Sorted newest-first.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `size` | Integer | No | `25` | Items per page |
| `status` | String | No | — | Filter by status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |
| `type` | String | No | — | Filter by type: `POST`, `COMMENT`, `PROFILE`, `STORY` |

**Example Request:**
```http
GET /api/admin/reports?status=PENDING&type=POST&page=1&size=25
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 12,
  "data": [
    {
      "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
      "reporterId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "type": "POST",
      "reason": "SPAM",
      "description": "This post contains spam links",
      "status": "PENDING",
      "postId": "d4e5f6a7-b890-1234-cdef-456789012345",
      "commentId": null,
      "profileId": null,
      "storyId": null,
      "commentLikeId": null,
      "reviewedBy": null,
      "reviewedAt": null,
      "resolution": null,
      "createdAt": "2026-03-20T14:00:00Z",
      "updatedAt": "2026-03-20T14:00:00Z",
      "reporter": {
        "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "displayName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg"
      },
      "post": {
        "id": "d4e5f6a7-b890-1234-cdef-456789012345",
        "caption": "Check out this amazing deal...",
        "author": {
          "id": "e5f6a7b8-9012-3456-def0-567890123456",
          "displayName": "Spammer User"
        }
      },
      "comment": null,
      "reportedProfile": null,
      "reviewer": null
    }
  ]
}
```

### Response includes (when applicable):

| Relation | Fields | Included when |
|----------|--------|---------------|
| `reporter` | `id`, `displayName`, `avatarUrl` | Always |
| `post` | `id`, `caption`, `post.author.id`, `post.author.displayName` | Report type is `POST` |
| `comment` | `id`, `content`, `comment.author.id`, `comment.author.displayName` | Report type is `COMMENT` |
| `reportedProfile` | `id`, `displayName`, `avatarUrl` | Report type is `PROFILE` |
| `reviewer` | `id`, `displayName` | Report has been reviewed |

---

## Review Report

**Endpoint:** `PATCH /api/admin/reports/:reportId`

**Authentication:** Required — `reports.update`

Updates a report's status and/or adds a resolution note. Automatically records the reviewing admin and timestamp.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reportId` | UUID | Yes | Report UUID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | String | No | New status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |
| `resolution` | String | No | Admin's resolution note (max 1000 chars) |

**Example Request — mark as reviewed:**
```http
PATCH /api/admin/reports/c3d4e5f6-a7b8-9012-cdef-345678901234
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "status": "REVIEWED",
  "resolution": "Content verified — does contain spam links. User warned."
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Report reviewed successfully",
  "data": {
    "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
    "reporterId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "type": "POST",
    "reason": "SPAM",
    "description": "This post contains spam links",
    "status": "REVIEWED",
    "postId": "d4e5f6a7-b890-1234-cdef-456789012345",
    "commentId": null,
    "profileId": null,
    "storyId": null,
    "commentLikeId": null,
    "reviewedBy": "550e8400-e29b-41d4-a716-446655440000",
    "reviewedAt": "2026-03-24T15:30:00Z",
    "resolution": "Content verified — does contain spam links. User warned.",
    "createdAt": "2026-03-20T14:00:00Z",
    "updatedAt": "2026-03-24T15:30:00Z",
    "reporter": {
      "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "displayName": "John Doe"
    },
    "reviewer": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "displayName": "Admin Alice"
    }
  }
}
```

**Example Request — dismiss:**
```http
PATCH /api/admin/reports/c3d4e5f6-a7b8-9012-cdef-345678901234
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "status": "DISMISSED",
  "resolution": "Content reviewed and found acceptable. No action needed."
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Invalid report ID format (not a valid UUID) |
| `404` | Report not found |
| `422` | Validation error (invalid status value, resolution too long) |

---

## User-Facing Endpoints

These endpoints are used by regular app users (not admins) to submit and view their own reports.

**Base URL:** `/api/reports`

**Authentication:** Standard user auth — `Authorization: Bearer <accessToken>` from `POST /api/auth/login`.

---

### Submit Report

**Endpoint:** `POST /api/reports`

**Authentication:** Required — `USER`, `SELLER`, or `MEDIATOR` role.

Submits a report against a post, comment, profile, or story. Exactly one target must be provided.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | **Yes** | Report type: `POST`, `COMMENT`, `PROFILE`, `STORY` |
| `reason` | String | **Yes** | Reason: `SPAM`, `HARASSMENT`, `INAPPROPRIATE`, `COPYRIGHT`, `FALSE_INFO`, `OTHER` |
| `description` | String | No | Additional details (max 1000 chars) |
| `postId` | UUID | Conditional | Target post ID (required if `type` is `POST`) |
| `commentId` | UUID | Conditional | Target comment ID (required if `type` is `COMMENT`) |
| `profileId` | UUID | Conditional | Target profile ID (required if `type` is `PROFILE`) |
| `storyId` | String (numeric) | Conditional | Target story ID as BigInt string (required if `type` is `STORY`) |

> **Note:** Exactly one of `postId`, `commentId`, `profileId`, or `storyId` must be provided.

**Example Request — report a post:**
```http
POST /api/reports
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "type": "POST",
  "reason": "SPAM",
  "description": "This post contains spam links and misleading content",
  "postId": "d4e5f6a7-b890-1234-cdef-456789012345"
}
```

**Example Request — report a profile:**
```http
POST /api/reports
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "type": "PROFILE",
  "reason": "HARASSMENT",
  "profileId": "e5f6a7b8-9012-3456-def0-567890123456"
}
```

**Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Report submitted successfully",
  "data": {
    "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
    "reporterId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "type": "POST",
    "reason": "SPAM",
    "description": "This post contains spam links and misleading content",
    "status": "PENDING",
    "postId": "d4e5f6a7-b890-1234-cdef-456789012345",
    "commentId": null,
    "profileId": null,
    "storyId": null,
    "reviewedBy": null,
    "reviewedAt": null,
    "resolution": null,
    "createdAt": "2026-03-24T14:00:00Z",
    "updatedAt": "2026-03-24T14:00:00Z"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Exactly one target must be provided |
| `400` | You cannot report yourself |
| `404` | Target not found |
| `409` | You have already reported this content |
| `422` | Validation error (missing type/reason, invalid values) |

---

### Get My Reports

**Endpoint:** `GET /api/reports/my-reports`

**Authentication:** Required — `USER`, `SELLER`, or `MEDIATOR` role.

Returns the authenticated user's own reports with pagination and optional status filter.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `size` | Integer | No | `25` | Items per page |
| `status` | String | No | — | Filter by status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |

**Example Request:**
```http
GET /api/reports/my-reports?status=PENDING&page=1&size=25
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 3,
  "data": [
    {
      "id": "c3d4e5f6-a7b8-9012-cdef-345678901234",
      "reporterId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "type": "POST",
      "reason": "SPAM",
      "description": "This post contains spam links",
      "status": "PENDING",
      "postId": "d4e5f6a7-b890-1234-cdef-456789012345",
      "commentId": null,
      "profileId": null,
      "storyId": null,
      "commentLikeId": null,
      "reviewedBy": null,
      "reviewedAt": null,
      "resolution": null,
      "createdAt": "2026-03-20T14:00:00Z",
      "updatedAt": "2026-03-20T14:00:00Z",
      "post": {
        "id": "d4e5f6a7-b890-1234-cdef-456789012345",
        "caption": "Check out this amazing deal...",
        "author": {
          "id": "e5f6a7b8-9012-3456-def0-567890123456",
          "displayName": "Spammer User"
        }
      },
      "comment": null,
      "reportedProfile": null,
      "reviewer": null
    }
  ]
}
```

---

## Enums & Schema Reference

### ReportType

| Value | Description |
|-------|-------------|
| `POST` | Report targeting a post |
| `COMMENT` | Report targeting a comment |
| `PROFILE` | Report targeting a user profile |
| `STORY` | Report targeting a story |

### ReportReason

| Value | Description |
|-------|-------------|
| `SPAM` | Spam or unwanted content |
| `HARASSMENT` | Harassment or bullying |
| `INAPPROPRIATE` | Inappropriate or offensive content |
| `COPYRIGHT` | Copyright or intellectual property violation |
| `FALSE_INFO` | False or misleading information |
| `OTHER` | Other reason (use `description` for details) |

### ReportStatus

| Value | Description |
|-------|-------------|
| `PENDING` | Newly submitted, awaiting admin review |
| `REVIEWED` | Admin has reviewed but action may still be needed |
| `RESOLVED` | Issue has been resolved (content removed, user warned, etc.) |
| `DISMISSED` | Report was reviewed and dismissed (no action taken) |

### Report Model

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Primary key |
| `reporterId` | UUID | Profile ID of the user who submitted the report |
| `type` | ReportType | Type of content being reported |
| `reason` | ReportReason | Reason for the report |
| `description` | Text | Optional additional description |
| `status` | ReportStatus | Current review status (default: `PENDING`) |
| `postId` | UUID | Reported post ID (if type is `POST`) |
| `commentId` | UUID | Reported comment ID (if type is `COMMENT`) |
| `profileId` | UUID | Reported profile ID (if type is `PROFILE`) |
| `storyId` | BigInt | Reported story ID (if type is `STORY`) |
| `commentLikeId` | UUID | Reported comment like ID (reserved) |
| `reviewedBy` | UUID | Profile ID of the admin who reviewed |
| `reviewedAt` | DateTime | Timestamp of admin review |
| `resolution` | Text | Admin's resolution note |
| `createdAt` | DateTime | Creation timestamp |
| `updatedAt` | DateTime | Last update timestamp |

### Report Relations

| Relation | Model | FK Field | On Delete |
|----------|-------|----------|-----------|
| `reporter` | Profile | `reporterId` | Cascade |
| `reportedProfile` | Profile | `profileId` | Cascade |
| `post` | Post | `postId` | Cascade |
| `comment` | Comment | `commentId` | Cascade |
| `story` | Story | `storyId` | Cascade |
| `reviewer` | Profile | `reviewedBy` | Set Null |
| `commentLike` | CommentLike | `commentLikeId` | Cascade |

---

## Business Rules

| Rule | Enforced at |
|------|-------------|
| Exactly one target (`postId`, `commentId`, `profileId`, or `storyId`) must be provided | `POST /api/reports` validation + controller |
| A user cannot report themselves | `POST /api/reports` controller |
| Target entity must exist before a report can be created | `POST /api/reports` controller |
| Duplicate reports (same reporter + same target) are rejected | `POST /api/reports` controller |
| All new reports start with status `PENDING` | `POST /api/reports` controller |
| `reviewedBy` and `reviewedAt` are automatically set when admin reviews | `PATCH /api/admin/reports/:reportId` controller |
| Users can only view their own reports | `GET /api/reports/my-reports` query filter |
| Report IDs are UUIDs in all URL paths | All endpoints |

---

## Error Responses

### Admin Endpoints

| Status | Meaning |
|--------|---------|
| `400` | Invalid UUID format or business rule violation |
| `401` | Missing or invalid/expired admin token |
| `403` | RBAC permission denied (`reports.read` or `reports.update` required) |
| `404` | Report not found |
| `422` | Validation error — response includes field-level detail |

### User Endpoints

| Status | Meaning |
|--------|---------|
| `400` | Invalid target specification or self-report attempt |
| `401` | Missing or invalid/expired user token |
| `404` | Target entity not found |
| `409` | Duplicate report — content already reported by this user |
| `422` | Validation error — response includes field-level detail |

**Validation error example (422):**
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "type": "\"type\" must be one of [POST, COMMENT, PROFILE, STORY]",
    "reason": "\"reason\" is required"
  }
}
```

---

## Related Docs

- [social-reports.md](./social-reports.md) — Original social reports reference.
- [admin.md](./admin.md) — App-user management, content moderation, and analytics.
- [admin-permissions.md](./admin-permissions.md) — RBAC, roles, and permission resources.
