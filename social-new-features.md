# New Features API Documentation - Stories, Search & Explore

This document provides comprehensive API documentation for the newly added features:
- **Stories Feature** - Create, view, like, and manage stories
- **Search Posts** - Search posts by caption and hashtags
- **Explore/Discover** - Trending posts, popular content, and suggested users

## 📋 Table of Contents
1. [Authentication](#authentication)
2. [Stories APIs](#stories-apis)
3. [Search Posts APIs](#search-posts-apis)
4. [Trending & Popular Posts APIs](#trending--popular-posts-apis)
5. [Explore & Discover APIs](#explore--discover-apis)

---

## 🔐 Authentication

All endpoints require authentication via JWT token in the Authorization header.

**Header:**
```
Authorization: <your_jwt_token>
```

**Base URLs:**
- Stories: `/api/stories`
- Search/Explore Posts: `/api/posts`
- Explore Feed: `/api/explore`

---

## 📸 Stories APIs

### 1. Create Story
**POST** `/api/stories`

**Description:** Create a new story with media upload. Stories expire after 24 hours.

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

**Response (201):**
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

---

### 2. Get Active Stories
**GET** `/api/stories/active`

**Description:** Get all active stories from friends and users you follow, grouped by author.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/stories/active?page=1&size=25
```

**Response (200):**
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

---

### 3. Get My Stories
**GET** `/api/stories/my-stories`

**Description:** Get all active stories created by the authenticated user.

**Headers:**
```
Authorization: <token>
```

**Response (200):**
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

---

### 4. Get Story by ID
**GET** `/api/stories/:storyId`

**Description:** Get a specific story by its ID.

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Headers:**
```
Authorization: <token>
```

**Response (200):**
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
- `404` - Story not found
- `410` - Story has expired

---

### 5. View Story (Mark as Viewed)
**POST** `/api/stories/:storyId/view`

**Description:** Mark a story as viewed by the current user.

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Headers:**
```
Authorization: <token>
```

**Response (201):**
```json
{
  "success": true,
  "message": "Story marked as viewed",
  "data": {
    "viewedAt": "2024-01-01T12:00:00.000Z"
  }
}
```

---

### 6. Like/Unlike Story
**POST** `/api/stories/:storyId/like`

**Description:** Toggle like status for a story.

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Headers:**
```
Authorization: <token>
```

**Response (200):**
```json
{
  "success": true,
  "data": {
    "liked": true,
    "totalLikes": 6
  }
}
```

---

### 7. Delete Story
**DELETE** `/api/stories/:storyId`

**Description:** Delete a story (only the author can delete their own story).

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| storyId | String | Yes | Story ID (numeric) |

**Headers:**
```
Authorization: <token>
```

**Response (200):**
```json
{
  "success": true,
  "message": "Story deleted successfully"
}
```

**Error Responses:**
- `403` - You are not authorized to delete this story

---

## 🔍 Search Posts APIs

### 8. Search Posts
**GET** `/api/posts/search`

**Description:** Search posts by caption content with optional filtering by post type.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| q | String | Yes | Search query (1-100 characters) |
| type | String | No | Filter by post type: `FEED`, `STORY`, or `REEL` |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/posts/search?q=summer&type=FEED&page=1&size=25
```

**Response (200):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 15,
  "query": "summer",
  "data": [
    {
      "id": "uuid",
      "type": "FEED",
      "caption": "Summer vibes! ☀️",
      "mediaUrl": "https://res.cloudinary.com/...",
      "createdAt": "2024-01-01T12:00:00.000Z",
      "updatedAt": "2024-01-01T12:00:00.000Z",
      "author": {
        "id": "uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "counts": {
        "likes": 25,
        "comments": 5,
        "shares": 3
      },
      "likedByCurrentUser": true,
      "isMine": false,
      "isFollow": true,
      "repostedByCurrentUser": false,
      "isRepostedByMine": false
    }
  ]
}
```

**Error Responses:**
- `400` - Search query is required

---

## 🔥 Trending & Popular Posts APIs

### 9. Get Trending Posts
**GET** `/api/posts/trending`

**Description:** Get posts sorted by engagement score (likes×2 + comments×3 + shares×4) from the last N days.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| days | String | No | Number of days to look back (default: 7) |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/posts/trending?days=7&page=1&size=25
```

**Response (200):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 50,
  "days": 7,
  "data": [
    {
      "id": "uuid",
      "type": "FEED",
      "caption": "Trending post!",
      "mediaUrl": "https://res.cloudinary.com/...",
      "createdAt": "2024-01-01T12:00:00.000Z",
      "updatedAt": "2024-01-01T12:00:00.000Z",
      "author": {
        "id": "uuid",
        "displayName": "Jane Doe",
        "avatarUrl": "https://...",
        "role": "SELLER"
      },
      "counts": {
        "likes": 100,
        "comments": 20,
        "shares": 15
      },
      "likedByCurrentUser": false,
      "isMine": false,
      "isFollow": false,
      "repostedByCurrentUser": false,
      "isRepostedByMine": false
    }
  ]
}
```

**Notes:**
- Engagement score calculation: `likes × 2 + comments × 3 + shares × 4`
- Posts are sorted by engagement score (highest first)
- Default time period is 7 days

---

### 10. Get Popular Posts
**GET** `/api/posts/popular`

**Description:** Get posts with the most total engagement (likes + comments + shares) from the last N days.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| days | String | No | Number of days to look back (default: 30) |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/posts/popular?days=30&page=1&size=25
```

**Response (200):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 75,
  "days": 30,
  "data": [
    {
      "id": "uuid",
      "type": "REEL",
      "caption": "Popular content!",
      "mediaUrl": "https://res.cloudinary.com/...",
      "createdAt": "2024-01-01T12:00:00.000Z",
      "updatedAt": "2024-01-01T12:00:00.000Z",
      "author": {
        "id": "uuid",
        "displayName": "Bob Smith",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "counts": {
        "likes": 500,
        "comments": 50,
        "shares": 30
      },
      "likedByCurrentUser": true,
      "isMine": false,
      "isFollow": true,
      "repostedByCurrentUser": true,
      "isRepostedByMine": true
    }
  ]
}
```

**Notes:**
- Only posts with engagement > 0 are included
- Sorted by total engagement (likes + comments + shares)
- Default time period is 30 days (vs 7 days for trending)

---

## 🌟 Explore & Discover APIs

### 11. Get Explore Feed
**GET** `/api/explore`

**Description:** Get a combined feed with trending posts, popular posts, and suggested users.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/explore?page=1&size=25
```

**Response (200):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 10,
  "data": {
    "posts": [
      {
        "id": "uuid",
        "type": "FEED",
        "caption": "Trending content",
        "mediaUrl": "https://res.cloudinary.com/...",
        "createdAt": "2024-01-01T12:00:00.000Z",
        "updatedAt": "2024-01-01T12:00:00.000Z",
        "author": {
          "id": "uuid",
          "displayName": "John Doe",
          "avatarUrl": "https://...",
          "role": "USER"
        },
        "counts": {
          "likes": 100,
          "comments": 20,
          "shares": 15
        },
        "likedByCurrentUser": false,
        "isMine": false,
        "isFollow": false,
        "repostedByCurrentUser": false,
        "isRepostedByMine": false,
        "type": "trending"
      }
    ],
    "suggestedUsers": [
      {
        "id": "uuid",
        "username": "jane_doe",
        "displayName": "Jane Doe",
        "avatarUrl": "https://...",
        "bio": "Content creator",
        "role": "SELLER",
        "stats": {
          "followers": 5000,
          "posts": 150
        },
        "isFollow": false
      }
    ]
  }
}
```

**Notes:**
- Combines top 5 trending posts and top 5 popular posts
- Includes top 5 suggested users
- Removes duplicate posts if they appear in both trending and popular

---

### 12. Get Suggested Users
**GET** `/api/explore/suggested-users`

**Description:** Get users to follow based on activity, follower count, and mutual connections.

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 20) |
| limit | String | No | Maximum suggestions to fetch (default: 20) |

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET /api/explore/suggested-users?page=1&size=20&limit=20
```

**Response (200):**
```json
{
  "success": true,
  "page": "1",
  "size": "20",
  "total": 15,
  "data": [
    {
      "id": "uuid",
      "username": "jane_doe",
      "email": null,
      "displayName": "Jane Doe",
      "avatarUrl": "https://...",
      "bio": "Content creator and influencer",
      "role": "SELLER",
      "stats": {
        "followers": 5000,
        "posts": 150
      },
      "mutualConnections": 5
    },
    {
      "id": "uuid",
      "username": "bob_smith",
      "email": null,
      "displayName": "Bob Smith",
      "avatarUrl": "https://...",
      "bio": "Photographer",
      "role": "USER",
      "stats": {
        "followers": 1200,
        "posts": 45
      },
      "mutualConnections": 2
    }
  ]
}
```

**Notes:**
- Excludes users you already follow, are friends with, or have blocked
- Scoring algorithm:
  - Active users (posted in last 30 days): +100 points
  - Follower count: ×2 points
  - Post count: ×1 point
- Shows mutual connection count (users who follow both you and the suggested user)
- Sorted by score (highest first)

---

## 📝 Response Fields Explanation

### Post Object Fields:
- `id`: Post UUID
- `type`: Post type (`FEED`, `STORY`, `REEL`)
- `caption`: Post caption text
- `mediaUrl`: URL to post media
- `author`: Author profile information
- `counts`: Engagement counts (likes, comments, shares)
- `likedByCurrentUser`: Whether current user liked this post
- `isMine`: Whether post belongs to current user
- `isFollow`: Whether current user follows the author
- `repostedByCurrentUser`: Whether current user reposted this
- `isRepostedByMine`: Whether current user reposted this (alias)

### Story Object Fields:
- `id`: Story ID (BigInt as string)
- `mediaUrl`: URL to story media
- `caption`: Story caption
- `expiresAt`: When the story expires (24 hours after creation)
- `createdAt`: Story creation timestamp
- `counts`: Engagement counts (views, likes)
- `isViewed`: Whether current user has viewed this story
- `isLiked`: Whether current user has liked this story

### Suggested User Object Fields:
- `id`: Profile UUID
- `username`: User's username
- `displayName`: Display name
- `avatarUrl`: Profile avatar URL
- `bio`: User bio
- `role`: User role (`USER`, `SELLER`, `MEDIATOR`)
- `stats`: User statistics (followers, posts)
- `mutualConnections`: Number of mutual followers

---

## 🎯 Best Practices

### Stories:
1. **Expiration:** Stories automatically expire after 24 hours
2. **Media Upload:** Maximum file size is 50MB, supports images and videos
3. **Views:** Each user can only view a story once (tracked per user)
4. **Grouping:** Active stories are grouped by author for easier browsing

### Search:
1. Use specific keywords for better results
2. Filter by type if you only want specific post types
3. Use pagination for large result sets

### Trending vs Popular:
1. Use **Trending** for recent viral content (last 7 days)
2. Use **Popular** for all-time best content (last 30 days)

### Suggested Users:
1. Higher `limit` returns more suggestions but may be slower
2. Check `mutualConnections` to prioritize users with mutual friends
3. Consider user's `stats` to find active creators

### Pagination:
1. Default page size is 25, adjust based on your needs
2. Use `total` field to determine total pages
3. Always check if `data` array is empty before processing

---

## 📦 Postman Collection Examples

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
GET {{baseUrl}}/api/stories/active
Authorization: {{token}}
```

### Search Posts
```
GET {{baseUrl}}/api/posts/search?q=summer&type=FEED&page=1&size=25
Authorization: {{token}}
```

### Get Trending Posts
```
GET {{baseUrl}}/api/posts/trending?days=7&page=1&size=25
Authorization: {{token}}
```

### Get Popular Posts
```
GET {{baseUrl}}/api/posts/popular?days=30&page=1&size=25
Authorization: {{token}}
```

### Get Explore Feed
```
GET {{baseUrl}}/api/explore?page=1&size=25
Authorization: {{token}}
```

### Get Suggested Users
```
GET {{baseUrl}}/api/explore/suggested-users?page=1&size=20&limit=20
Authorization: {{token}}
```

---

## ⚠️ Error Responses

All endpoints may return these common errors:

- `400` - Bad Request (invalid parameters, missing required fields)
- `401` - Unauthorized (missing or invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource doesn't exist)
- `410` - Gone (story has expired)
- `500` - Internal Server Error

Error response format:
```json
{
  "message": "Error message",
  "status_code": 400,
  "error": {}
}
```

---

## 🔗 Related Documentation

- [Profile API Documentation](../profiles/POSTMAN_API_DOCUMENTATION.md)
- [Social Module API Documentation](./POSTMAN_API_DOCUMENTATION.md)
- [Stories API Documentation](./stories/POSTMAN_API_DOCUMENTATION.md)
- [Search & Explore API Documentation](./posts/POSTMAN_SEARCH_EXPLORE_DOCUMENTATION.md)

