# Activity Feed API Documentation

## Overview
The Activity Feed API provides endpoints to view recent activity from friends and following users, including posts, likes, comments, and reposts.

**Base URL:** `/api/activity`

---

## Endpoints

### 1. Get Activity Feed

Get a timeline of recent activity from friends and following users.

**Endpoint:** `GET /api/activity/feed`

**Authentication:** Required (Bearer Token)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | String | No | `all` | Filter by activity type: `all`, `posts`, `likes`, `comments`, `reposts` |
| `page` | String | No | `1` | Page number for pagination |
| `size` | String | No | `25` | Number of items per page |

**Example Request:**
```http
GET /api/activity/feed?type=all&page=1&size=25
Authorization: Bearer <your-token>
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
      "type": "POST",
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "activityId": "550e8400-e29b-41d4-a716-446655440000",
      "actor": {
        "id": "123e4567-e89b-12d3-a456-426614174000",
        "displayName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg",
        "role": "USER"
      },
      "target": {
        "type": "post",
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Beautiful sunset today! #sunset #nature",
        "mediaUrl": "https://example.com/image.jpg"
      },
      "timestamp": "2024-01-15T10:30:00Z",
      "metadata": {
        "likes": 15,
        "comments": 3,
        "shares": 2
      }
    },
    {
      "type": "LIKE",
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "activityId": "660e8400-e29b-41d4-a716-446655440001",
      "actor": {
        "id": "223e4567-e89b-12d3-a456-426614174001",
        "displayName": "Jane Smith",
        "avatarUrl": "https://example.com/avatar2.jpg",
        "role": "USER"
      },
      "target": {
        "type": "post",
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Beautiful sunset today!",
        "author": {
          "id": "123e4567-e89b-12d3-a456-426614174000",
          "displayName": "John Doe",
          "avatarUrl": "https://example.com/avatar.jpg"
        }
      },
      "timestamp": "2024-01-15T10:25:00Z"
    },
    {
      "type": "COMMENT",
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "activityId": "770e8400-e29b-41d4-a716-446655440002",
      "actor": {
        "id": "323e4567-e89b-12d3-a456-426614174002",
        "displayName": "Bob Wilson",
        "avatarUrl": "https://example.com/avatar3.jpg",
        "role": "USER"
      },
      "target": {
        "type": "post",
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Beautiful sunset today!",
        "author": {
          "id": "123e4567-e89b-12d3-a456-426614174000",
          "displayName": "John Doe"
        }
      },
      "content": "Amazing photo!",
      "timestamp": "2024-01-15T10:20:00Z"
    },
    {
      "type": "REPOST",
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "activityId": "880e8400-e29b-41d4-a716-446655440003",
      "actor": {
        "id": "423e4567-e89b-12d3-a456-426614174003",
        "displayName": "Alice Brown",
        "avatarUrl": "https://example.com/avatar4.jpg",
        "role": "USER"
      },
      "target": {
        "type": "post",
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "caption": "Beautiful sunset today!",
        "author": {
          "id": "123e4567-e89b-12d3-a456-426614174000",
          "displayName": "John Doe"
        }
      },
      "timestamp": "2024-01-15T10:15:00Z"
    }
  ]
}
```

**Activity Types:**
- `POST`: New post created
- `LIKE`: Post liked by a friend/following
- `COMMENT`: Comment added to a post
- `REPOST`: Post reposted by a friend/following

**Error Responses:**

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

**400 Bad Request:**
```json
{
  "message": "Invalid query parameters",
  "status_code": 400
}
```

---

## Postman Collection Setup

### Environment Variables
- `base_url`: `http://localhost:3000` (or your server URL)
- `auth_token`: Your JWT token

### Example Postman Request

**Request:**
```
GET {{base_url}}/api/activity/feed?type=all&page=1&size=25
Authorization: Bearer {{auth_token}}
```

**Tests (Postman):**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has success field", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('success');
    pm.expect(jsonData.success).to.be.true;
});

pm.test("Response has data array", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});
```

---

## Notes

- Activities are sorted by timestamp (most recent first)
- Only shows activity from users you follow or are friends with
- Activities are mixed together in chronological order
- Empty feed returns an empty array with `total: 0`

