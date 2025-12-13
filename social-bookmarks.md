# Bookmarks API Documentation

## Overview
The Bookmarks API allows users to save posts for later viewing. Users can bookmark posts, view their saved posts, and remove bookmarks.

**Base URL:** `/api/bookmarks`

---

## Endpoints

### 1. Save Post (Bookmark)

Save a post to your bookmarks.

**Endpoint:** `POST /api/bookmarks/:postId`

**Authentication:** Required (Bearer Token)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | UUID | Yes | ID of the post to save |

**Example Request:**
```http
POST /api/bookmarks/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <your-token>
```

**Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Post saved successfully",
  "data": {
    "id": "aa0e8400-e29b-41d4-a716-446655440000",
    "profileId": "123e4567-e89b-12d3-a456-426614174000",
    "postId": "550e8400-e29b-41d4-a716-446655440000",
    "createdAt": "2024-01-15T10:30:00Z",
    "post": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "type": "FEED",
      "caption": "Beautiful sunset today! #sunset #nature",
      "mediaUrl": "https://example.com/image.jpg",
      "visibility": "PUBLIC",
      "createdAt": "2024-01-15T09:00:00Z",
      "author": {
        "id": "223e4567-e89b-12d3-a456-426614174001",
        "displayName": "Jane Smith",
        "avatarUrl": "https://example.com/avatar.jpg",
        "role": "USER"
      },
      "_count": {
        "likes": 15,
        "comments": 3,
        "shares": 2
      }
    }
  }
}
```

**Error Responses:**

**404 Not Found:**
```json
{
  "message": "Post not found",
  "status_code": 404
}
```

**409 Conflict:**
```json
{
  "message": "Post is already saved",
  "status_code": 409
}
```

---

### 2. Unsave Post (Remove Bookmark)

Remove a post from your bookmarks.

**Endpoint:** `DELETE /api/bookmarks/:postId`

**Authentication:** Required (Bearer Token)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | UUID | Yes | ID of the post to unsave |

**Example Request:**
```http
DELETE /api/bookmarks/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Post unsaved successfully"
}
```

**Error Responses:**

**404 Not Found:**
```json
{
  "message": "Post is not saved",
  "status_code": 404
}
```

---

### 3. Get Saved Posts

Get all posts saved by the authenticated user.

**Endpoint:** `GET /api/bookmarks`

**Authentication:** Required (Bearer Token)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/bookmarks?page=1&size=25
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 10,
  "data": [
    {
      "id": "aa0e8400-e29b-41d4-a716-446655440000",
      "savedAt": "2024-01-15T10:30:00Z",
      "post": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "type": "FEED",
        "caption": "Beautiful sunset today! #sunset #nature",
        "mediaUrl": "https://example.com/image.jpg",
        "visibility": "PUBLIC",
        "createdAt": "2024-01-15T09:00:00Z",
        "updatedAt": "2024-01-15T09:00:00Z",
        "author": {
          "id": "223e4567-e89b-12d3-a456-426614174001",
          "displayName": "Jane Smith",
          "avatarUrl": "https://example.com/avatar.jpg",
          "role": "USER"
        },
        "counts": {
          "likes": 15,
          "comments": 3,
          "shares": 2
        },
        "hashtags": [
          {
            "id": 1,
            "name": "sunset"
          },
          {
            "id": 2,
            "name": "nature"
          }
        ]
      }
    }
  ]
}
```

---

### 4. Check if Post is Saved

Check whether a specific post is saved by the authenticated user.

**Endpoint:** `GET /api/bookmarks/:postId/check`

**Authentication:** Required (Bearer Token)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postId` | UUID | Yes | ID of the post to check |

**Example Request:**
```http
GET /api/bookmarks/550e8400-e29b-41d4-a716-446655440000/check
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "isSaved": true
  }
}
```

---

## Postman Collection Examples

### Save Post

**Request:**
```
POST {{base_url}}/api/bookmarks/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{auth_token}}
```

**Tests:**
```javascript
pm.test("Status code is 201", function () {
    pm.response.to.have.status(201);
});

pm.test("Post is saved", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData.success).to.be.true;
    pm.expect(jsonData.data).to.have.property('postId');
});
```

### Unsave Post

**Request:**
```
DELETE {{base_url}}/api/bookmarks/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{auth_token}}
```

### Get Saved Posts

**Request:**
```
GET {{base_url}}/api/bookmarks?page=1&size=25
Authorization: Bearer {{auth_token}}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has saved posts", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});
```

### Check if Saved

**Request:**
```
GET {{base_url}}/api/bookmarks/550e8400-e29b-41d4-a716-446655440000/check
Authorization: Bearer {{auth_token}}
```

---

## Notes

- Each user can save a post only once (duplicate saves return 409)
- Saved posts are sorted by `savedAt` timestamp (most recent first)
- The `isSaved` field is also included in the feed response for convenience
- Bookmarks are user-specific and private
- Saved posts respect visibility settings (you can only see posts you have permission to view)

