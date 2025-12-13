# Reports API Documentation

## Overview
The Reports API allows users to report inappropriate content (posts, comments, profiles, stories) for moderation.

**Base URL:** `/api/reports`

---

## Endpoints

### 1. Create Report

Submit a report for inappropriate content.

**Endpoint:** `POST /api/reports`

**Authentication:** Required (Bearer Token)

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `type` | String | Yes | Type of content: `POST`, `COMMENT`, `PROFILE`, `STORY` |
| `reason` | String | Yes | Reason: `SPAM`, `HARASSMENT`, `INAPPROPRIATE`, `COPYRIGHT`, `FALSE_INFO`, `OTHER` |
| `description` | String | No | Additional details (max 1000 chars) |
| `postId` | UUID | Conditional* | Post ID (required if type is POST) |
| `commentId` | UUID | Conditional* | Comment ID (required if type is COMMENT) |
| `profileId` | UUID | Conditional* | Profile ID (required if type is PROFILE) |
| `storyId` | String | Conditional* | Story ID (required if type is STORY) |

\* Exactly one of `postId`, `commentId`, `profileId`, or `storyId` must be provided.

**Example Request:**
```http
POST /api/reports
Authorization: Bearer <your-token>
Content-Type: application/json

{
  "type": "POST",
  "reason": "SPAM",
  "description": "This post contains spam content",
  "postId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Report submitted successfully",
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440000",
    "reporterId": "123e4567-e89b-12d3-a456-426614174000",
    "type": "POST",
    "reason": "SPAM",
    "description": "This post contains spam content",
    "status": "PENDING",
    "postId": "550e8400-e29b-41d4-a716-446655440000",
    "commentId": null,
    "profileId": null,
    "storyId": null,
    "reviewedBy": null,
    "reviewedAt": null,
    "resolution": null,
    "createdAt": "2024-01-15T10:30:00Z",
    "updatedAt": "2024-01-15T10:30:00Z"
  }
}
```

**Report Types:**
- `POST`: Report a post
- `COMMENT`: Report a comment
- `PROFILE`: Report a user profile
- `STORY`: Report a story

**Report Reasons:**
- `SPAM`: Spam content
- `HARASSMENT`: Harassment or bullying
- `INAPPROPRIATE`: Inappropriate content
- `COPYRIGHT`: Copyright violation
- `FALSE_INFO`: False information
- `OTHER`: Other reason

**Report Status:**
- `PENDING`: Awaiting review
- `REVIEWED`: Under review
- `RESOLVED`: Issue resolved
- `DISMISSED`: Report dismissed

**Error Responses:**

**400 Bad Request:**
```json
{
  "message": "Exactly one target (postId, commentId, profileId, or storyId) must be provided",
  "status_code": 400
}
```

**404 Not Found:**
```json
{
  "message": "Target not found",
  "status_code": 404
}
```

**409 Conflict:**
```json
{
  "message": "You have already reported this content",
  "status_code": 409
}
```

---

### 2. Get My Reports

Get all reports submitted by the authenticated user.

**Endpoint:** `GET /api/reports/my-reports`

**Authentication:** Required (Bearer Token)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `status` | String | No | - | Filter by status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/reports/my-reports?status=PENDING&page=1&size=25
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 5,
  "data": [
    {
      "id": "990e8400-e29b-41d4-a716-446655440000",
      "reporterId": "123e4567-e89b-12d3-a456-426614174000",
      "type": "POST",
      "reason": "SPAM",
      "description": "This post contains spam content",
      "status": "PENDING",
      "postId": "550e8400-e29b-41d4-a716-446655440000",
      "commentId": null,
      "profileId": null,
      "storyId": null,
      "reviewedBy": null,
      "reviewedAt": null,
      "resolution": null,
      "createdAt": "2024-01-15T10:30:00Z",
      "updatedAt": "2024-01-15T10:30:00Z",
      "post": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Check out this amazing product!",
        "author": {
          "id": "223e4567-e89b-12d3-a456-426614174001",
          "displayName": "Jane Smith"
        }
      },
      "reporter": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "displayName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg"
      }
    }
  ]
}
```

---

## Postman Collection Examples

### Create Report - Post

**Request:**
```
POST {{base_url}}/api/reports
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "type": "POST",
  "reason": "SPAM",
  "description": "This post contains spam content",
  "postId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Create Report - Comment

**Request:**
```
POST {{base_url}}/api/reports
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "type": "COMMENT",
  "reason": "HARASSMENT",
  "description": "This comment contains harassment",
  "commentId": "660e8400-e29b-41d4-a716-446655440001"
}
```

### Create Report - Profile

**Request:**
```
POST {{base_url}}/api/reports
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "type": "PROFILE",
  "reason": "INAPPROPRIATE",
  "description": "Inappropriate profile content",
  "profileId": "223e4567-e89b-12d3-a456-426614174001"
}
```

### Create Report - Story

**Request:**
```
POST {{base_url}}/api/reports
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "type": "STORY",
  "reason": "COPYRIGHT",
  "description": "This story violates copyright",
  "storyId": "1234567890"
}
```

### Get My Reports

**Request:**
```
GET {{base_url}}/api/reports/my-reports?status=PENDING&page=1&size=25
Authorization: Bearer {{auth_token}}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has reports array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});
```

---

## Notes

- Users cannot report themselves
- Each user can only report the same content once
- Reports start with `PENDING` status
- Admin users can review and update report status (see Admin API docs)
- Reports are automatically linked to the content being reported

