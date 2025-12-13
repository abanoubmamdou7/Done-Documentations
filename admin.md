# Admin API Documentation

## Overview
The Admin API provides endpoints for user management, content moderation, and analytics. **All endpoints require ADMIN or MEDIATOR role.**

**Base URL:** `/api/admin`

**Note:** Currently, admin endpoints use the `MEDIATOR` role. You may need to add an `ADMIN` role to your Role enum and update the middleware accordingly.

---

## Endpoints

### 1. Get All Reports

Get all reports submitted by users (for moderation).

**Endpoint:** `GET /api/admin/reports`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `status` | String | No | - | Filter by status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |
| `type` | String | No | - | Filter by type: `POST`, `COMMENT`, `PROFILE`, `STORY` |
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/admin/reports?status=PENDING&type=POST&page=1&size=25
Authorization: Bearer <admin-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 50,
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
      "reporter": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "displayName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg"
      },
      "post": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Check out this amazing product!",
        "author": {
          "id": "223e4567-e89b-12d3-a456-426614174001",
          "displayName": "Jane Smith"
        }
      }
    }
  ]
}
```

---

### 2. Review Report

Update the status and resolution of a report.

**Endpoint:** `PATCH /api/admin/reports/:reportId`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `reportId` | UUID | Yes | ID of the report to review |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | String | No | New status: `PENDING`, `REVIEWED`, `RESOLVED`, `DISMISSED` |
| `resolution` | String | No | Admin's resolution note (max 1000 chars) |

**Example Request:**
```http
PATCH /api/admin/reports/990e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "status": "RESOLVED",
  "resolution": "Post removed. User warned."
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Report reviewed successfully",
  "data": {
    "id": "990e8400-e29b-41d4-a716-446655440000",
    "status": "RESOLVED",
    "resolution": "Post removed. User warned.",
    "reviewedBy": "333e4567-e89b-12d3-a456-426614174002",
    "reviewedAt": "2024-01-15T11:00:00Z",
    "reviewer": {
      "id": "333e4567-e89b-12d3-a456-426614174002",
      "displayName": "Admin User"
    }
  }
}
```

---

### 3. Get All Users

Get a list of all users in the system.

**Endpoint:** `GET /api/admin/users`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `role` | String | No | - | Filter by role: `USER`, `SELLER`, `MEDIATOR` |
| `search` | String | No | - | Search by display name or phone |
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/admin/users?role=USER&search=john&page=1&size=25
Authorization: Bearer <admin-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 100,
  "data": [
    {
      "id": "123e4567-e89b-12d3-a456-426614174000",
      "role": "USER",
      "displayName": "John Doe",
      "avatarUrl": "https://example.com/avatar.jpg",
      "phone": "+1234567890",
      "bio": "Photography enthusiast",
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T10:00:00Z",
      "user": {
        "id": "111e4567-e89b-12d3-a456-426614174000",
        "email": "john@example.com",
        "username": "johndoe"
      },
      "stats": {
        "posts": 25,
        "sentFriendRequests": 10,
        "receivedFriendRequests": 5,
        "followsAsFollower": 50,
        "followsAsTarget": 30
      }
    }
  ]
}
```

---

### 4. Block/Unblock User

Block or unblock a user account.

**Endpoint:** `PATCH /api/admin/users/:profileId/block`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profileId` | UUID | Yes | ID of the user profile to block/unblock |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `isBlocked` | Boolean | No | Set to `true` to block, `false` to unblock. If not provided, toggles current state |

**Example Request:**
```http
PATCH /api/admin/users/123e4567-e89b-12d3-a456-426614174000/block
Authorization: Bearer <admin-token>
Content-Type: application/json

{
  "isBlocked": true
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "User blocked successfully",
  "data": {
    "id": "123e4567-e89b-12d3-a456-426614174000",
    "isBlocked": true,
    "displayName": "John Doe"
  }
}
```

**Note:** This endpoint requires an `isBlocked` field in the Profile model. If it doesn't exist, you'll need to add it to the schema.

---

### 5. Delete Post (Content Moderation)

Delete a post as an admin.

**Endpoint:** `DELETE /api/admin/posts/:postId`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | UUID | Yes | ID of the post to delete |

**Example Request:**
```http
DELETE /api/admin/posts/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <admin-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Post deleted successfully"
}
```

---

### 6. Delete Comment (Content Moderation)

Delete a comment as an admin.

**Endpoint:** `DELETE /api/admin/comments/:commentId`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `commentId` | UUID | Yes | ID of the comment to delete |

**Example Request:**
```http
DELETE /api/admin/comments/660e8400-e29b-41d4-a716-446655440001
Authorization: Bearer <admin-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Comment deleted successfully"
}
```

---

### 7. Get Analytics

Get platform analytics and statistics.

**Endpoint:** `GET /api/admin/analytics`

**Authentication:** Required (Bearer Token - ADMIN/MEDIATOR role)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `days` | String | No | `30` | Number of days to look back for recent statistics |

**Example Request:**
```http
GET /api/admin/analytics?days=30
Authorization: Bearer <admin-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "overview": {
      "totalUsers": 1000,
      "totalPosts": 5000,
      "totalComments": 15000,
      "totalReports": 50,
      "pendingReports": 10
    },
    "recent": {
      "days": 30,
      "newUsers": 100,
      "newPosts": 500,
      "newComments": 1500,
      "newReports": 5
    },
    "breakdown": {
      "usersByRole": [
        {
          "role": "USER",
          "count": 800
        },
        {
          "role": "SELLER",
          "count": 150
        },
        {
          "role": "MEDIATOR",
          "count": 50
        }
      ],
      "reportsByType": [
        {
          "type": "POST",
          "count": 30
        },
        {
          "type": "COMMENT",
          "count": 15
        },
        {
          "type": "PROFILE",
          "count": 5
        }
      ],
      "reportsByStatus": [
        {
          "status": "PENDING",
          "count": 10
        },
        {
          "status": "RESOLVED",
          "count": 30
        },
        {
          "status": "DISMISSED",
          "count": 10
        }
      ]
    }
  }
}
```

---

## Postman Collection Examples

### Get All Reports

**Request:**
```
GET {{base_url}}/api/admin/reports?status=PENDING&page=1&size=25
Authorization: Bearer {{admin_token}}
```

### Review Report

**Request:**
```
PATCH {{base_url}}/api/admin/reports/990e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{admin_token}}
Content-Type: application/json

{
  "status": "RESOLVED",
  "resolution": "Post removed. User warned."
}
```

### Get All Users

**Request:**
```
GET {{base_url}}/api/admin/users?role=USER&page=1&size=25
Authorization: Bearer {{admin_token}}
```

### Block User

**Request:**
```
PATCH {{base_url}}/api/admin/users/123e4567-e89b-12d3-a456-426614174000/block
Authorization: Bearer {{admin_token}}
Content-Type: application/json

{
  "isBlocked": true
}
```

### Delete Post

**Request:**
```
DELETE {{base_url}}/api/admin/posts/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{admin_token}}
```

### Delete Comment

**Request:**
```
DELETE {{base_url}}/api/admin/comments/660e8400-e29b-41d4-a716-446655440001
Authorization: Bearer {{admin_token}}
```

### Get Analytics

**Request:**
```
GET {{base_url}}/api/admin/analytics?days=30
Authorization: Bearer {{admin_token}}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has analytics data", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.have.property('overview');
    pm.expect(jsonData.data).to.have.property('recent');
    pm.expect(jsonData.data).to.have.property('breakdown');
});
```

---

## Error Responses

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

**403 Forbidden:**
```json
{
  "message": "Access denied. Admin role required.",
  "status_code": 403
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

## Notes

- All admin endpoints require ADMIN or MEDIATOR role
- Currently using MEDIATOR role - consider adding ADMIN role to Role enum
- Reports are automatically linked to the reviewer when status is updated
- Analytics data is calculated in real-time
- User blocking requires `isBlocked` field in Profile model
- Content deletion is permanent and cannot be undone

