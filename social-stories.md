# Stories API Documentation

This document provides comprehensive API documentation for the Stories feature endpoints.

## Base URL
```
/api/stories
```

## Authentication
All endpoints require authentication. Include the JWT token in the Authorization header:
```
Authorization: <your-jwt-token>
```

---

## 1. Create Story

Create a new story with media upload.

**Endpoint:** `POST /api/stories`

**Headers:**
```
Authorization: <token>
Content-Type: multipart/form-data
```

**Request Body (Form Data):**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| storyMedia | File | Yes | Image or video file (max 50MB) |
| caption | String | No | Story caption (max 500 characters) |

**Alternative (JSON with URL):**
```json
{
  "mediaUrl": "https://example.com/image.jpg",
  "caption": "My story caption"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Story created successfully",
  "data": {
    "id": "1234567890",
    "mediaUrl": "https://res.cloudinary.com/...",
    "caption": "My story caption",
    "expiresAt": "2024-01-02T12:00:00.000Z",
    "createdAt": "2024-01-01T12:00:00.000Z",
    "author": {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER"
    },
    "counts": {
      "views": 0,
      "likes": 0
    }
  }
}
```

**Error Responses:**
- `400` - Story media file (storyMedia) or mediaUrl is required
- `401` - Unauthorized
- `500` - Failed to upload story media

---

## 2. Get Active Stories

Get all active stories from friends and users you follow, grouped by author.

**Endpoint:** `GET /api/stories/active`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/stories/active?page=1&size=25
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 10,
  "count": 3,
  "data": [
    {
      "author": {
        "id": "uuid",
        "displayName": "Jane Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "stories": [
        {
          "id": "1234567890",
          "mediaUrl": "https://res.cloudinary.com/...",
          "caption": "Story caption",
          "expiresAt": "2024-01-02T12:00:00.000Z",
          "createdAt": "2024-01-01T12:00:00.000Z",
          "counts": {
            "views": 15,
            "likes": 5
          },
          "isViewed": false,
          "isLiked": false
        }
      ]
    }
  ]
}
```

**Error Responses:**
- `401` - Unauthorized

---

## 3. Get My Stories

Get all active stories created by the authenticated user.

**Endpoint:** `GET /api/stories/my-stories`

**Headers:**
```
Authorization: <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "count": 2,
  "data": [
    {
      "id": "1234567890",
      "mediaUrl": "https://res.cloudinary.com/...",
      "caption": "My story",
      "expiresAt": "2024-01-02T12:00:00.000Z",
      "createdAt": "2024-01-01T12:00:00.000Z",
      "counts": {
        "views": 10,
        "likes": 3
      }
    }
  ]
}
```

**Error Responses:**
- `401` - Unauthorized

---

## 4. Get Story by ID

Get a specific story by its ID.

**Endpoint:** `GET /api/stories/:storyId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "1234567890",
    "mediaUrl": "https://res.cloudinary.com/...",
    "caption": "Story caption",
    "expiresAt": "2024-01-02T12:00:00.000Z",
    "createdAt": "2024-01-01T12:00:00.000Z",
    "author": {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER"
    },
    "counts": {
      "views": 15,
      "likes": 5
    },
    "isViewed": true,
    "isLiked": false
  }
}
```

**Error Responses:**
- `400` - Invalid story ID
- `401` - Unauthorized
- `404` - Story not found
- `410` - Story has expired

---

## 5. View Story (Mark as Viewed)

Mark a story as viewed by the current user.

**Endpoint:** `POST /api/stories/:storyId/view`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Story marked as viewed",
  "data": {
    "viewedAt": "2024-01-01T12:00:00.000Z"
  }
}
```

**Response (200 OK - Already Viewed):**
```json
{
  "success": true,
  "message": "Story already viewed",
  "data": {
    "viewedAt": "2024-01-01T11:00:00.000Z"
  }
}
```

**Error Responses:**
- `400` - Invalid story ID
- `401` - Unauthorized
- `404` - Story not found
- `410` - Story has expired

---

## 6. Like/Unlike Story

Toggle like status for a story.

**Endpoint:** `POST /api/stories/:storyId/like`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "liked": true,
    "totalLikes": 6
  }
}
```

**Error Responses:**
- `400` - Invalid story ID
- `401` - Unauthorized
- `404` - Story not found
- `410` - Story has expired

---

## 7. Delete Story

Delete a story (only the author can delete their own story).

**Endpoint:** `DELETE /api/stories/:storyId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Story deleted successfully"
}
```

**Error Responses:**
- `400` - Invalid story ID
- `401` - Unauthorized
- `403` - You are not authorized to delete this story
- `404` - Story not found

---

## Notes

1. **Story Expiration:** Stories automatically expire after 24 hours from creation. Expired stories cannot be viewed, liked, or interacted with.

2. **Media Upload:** 
   - Maximum file size: 50MB
   - Supported formats: Images (jpg, png, gif, webp) and Videos (mp4, mov, avi)
   - Files are uploaded to Cloudinary

3. **Story Views:** Each user can only view a story once (tracked per user).

4. **Story Likes:** Users can like/unlike stories. The like count is included in the response.

5. **Story Grouping:** Active stories endpoint groups stories by author for easier browsing in a story viewer UI.

---

## Example Postman Collection

### Create Story
```
POST {{baseUrl}}/api/stories
Authorization: {{token}}
Content-Type: multipart/form-data

storyMedia: [Select File]
caption: "Check out my new story!"
```

### Get Active Stories
```
GET {{baseUrl}}/api/stories/active?page=1&size=25
Authorization: {{token}}
```

### View Story
```
POST {{baseUrl}}/api/stories/1234567890/view
Authorization: {{token}}
```

### Like Story
```
POST {{baseUrl}}/api/stories/1234567890/like
Authorization: {{token}}
```

