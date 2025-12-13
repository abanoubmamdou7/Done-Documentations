# Hashtags API Documentation

## Overview
The Hashtags API allows users to search hashtags, view trending hashtags, and browse posts by hashtag.

**Base URL:** `/api/hashtags`

---

## Endpoints

### 1. Search Hashtags

Search for hashtags by name.

**Endpoint:** `GET /api/hashtags/search`

**Authentication:** Required (Bearer Token)

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (1-100 characters) |
| `page` | String | No | Page number (default: 1) |
| `size` | String | No | Items per page (default: 25) |

**Example Request:**
```http
GET /api/hashtags/search?q=sunset&page=1&size=25
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
      "id": 1,
      "name": "sunset",
      "postCount": 150,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-15T10:00:00Z"
    },
    {
      "id": 2,
      "name": "sunsetphotography",
      "postCount": 45,
      "createdAt": "2024-01-05T00:00:00Z",
      "updatedAt": "2024-01-14T09:00:00Z"
    }
  ]
}
```

**Error Responses:**

**400 Bad Request:**
```json
{
  "message": "Search query is required",
  "status_code": 400
}
```

---

### 2. Get Trending Hashtags

Get trending hashtags based on recent post activity.

**Endpoint:** `GET /api/hashtags/trending`

**Authentication:** Required (Bearer Token)

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `days` | String | No | `7` | Number of days to look back for trending calculation |
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/hashtags/trending?days=7&page=1&size=25
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 20,
  "data": [
    {
      "id": 1,
      "name": "sunset",
      "postCount": 150,
      "recentPostCount": 45
    },
    {
      "id": 3,
      "name": "nature",
      "postCount": 200,
      "recentPostCount": 38
    },
    {
      "id": 5,
      "name": "photography",
      "postCount": 300,
      "recentPostCount": 32
    }
  ]
}
```

**Notes:**
- `postCount`: Total number of posts with this hashtag
- `recentPostCount`: Number of posts created in the last N days (specified by `days` parameter)
- Results are sorted by `recentPostCount` (most trending first)

---

### 3. Get Posts by Hashtag

Get all posts that contain a specific hashtag.

**Endpoint:** `GET /api/hashtags/:hashtagId/posts`

**Authentication:** Required (Bearer Token)

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `hashtagId` | Number | Yes | ID of the hashtag |

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | String | No | `1` | Page number |
| `size` | String | No | `25` | Items per page |

**Example Request:**
```http
GET /api/hashtags/1/posts?page=1&size=25
Authorization: Bearer <your-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 150,
  "data": [
    {
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
      "likedByCurrentUser": false,
      "repostedByCurrentUser": false,
      "isMine": false
    }
  ]
}
```

**Error Responses:**

**400 Bad Request:**
```json
{
  "message": "Invalid hashtag ID",
  "status_code": 400
}
```

**404 Not Found:**
```json
{
  "message": "Hashtag not found",
  "status_code": 404
}
```

**Notes:**
- Posts respect visibility settings (only shows PUBLIC posts or FOLLOWERS-only posts from users you follow)
- Results are sorted by creation date (most recent first)

---

## Postman Collection Examples

### Search Hashtags

**Request:**
```
GET {{base_url}}/api/hashtags/search?q=sunset&page=1&size=25
Authorization: Bearer {{auth_token}}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has hashtags", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});
```

### Get Trending Hashtags

**Request:**
```
GET {{base_url}}/api/hashtags/trending?days=7&page=1&size=25
Authorization: Bearer {{auth_token}}
```

### Get Posts by Hashtag

**Request:**
```
GET {{base_url}}/api/hashtags/1/posts?page=1&size=25
Authorization: Bearer {{auth_token}}
```

**Tests:**
```javascript
pm.test("Status code is 200", function () {
    pm.response.to.have.status(200);
});

pm.test("Response has posts", function () {
    var jsonData = pm.response.json();
    pm.expect(jsonData).to.have.property('data');
    pm.expect(jsonData.data).to.be.an('array');
});
```

---

## Hashtag Usage in Posts

### Creating Posts with Hashtags

When creating a post, hashtags can be included in two ways:

1. **In the caption** (automatically extracted):
```json
{
  "caption": "Beautiful sunset today! #sunset #nature #photography",
  "visibility": "PUBLIC"
}
```

2. **As a separate array**:
```json
{
  "caption": "Beautiful sunset today!",
  "hashtags": ["sunset", "nature", "photography"],
  "visibility": "PUBLIC"
}
```

Hashtags are automatically:
- Extracted from captions (format: `#hashtag`)
- Normalized to lowercase
- Deduplicated
- Linked to the post

### Hashtag Format

- Must start with `#`
- Can contain letters, numbers, and underscores
- Case-insensitive (automatically converted to lowercase)
- Examples: `#sunset`, `#nature_photography`, `#travel2024`

---

## Notes

- Hashtag search is case-insensitive
- Hashtags are automatically created when first used in a post
- `postCount` is automatically updated when posts are created/deleted
- Trending hashtags are calculated based on recent activity (last N days)
- Posts by hashtag respect visibility and follow relationships

