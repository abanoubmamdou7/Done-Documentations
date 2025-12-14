# Posts Search & Explore API Documentation

This document provides comprehensive API documentation for the Search Posts and Explore/Discover endpoints.

## Base URLs
```
/api/posts (for search, trending, popular)
/api/explore (for explore feed and suggested users)
```

## Authentication
All endpoints require authentication. Include the JWT token in the Authorization header:
```
Authorization: <your-jwt-token>
```

---

## SEARCH POSTS

### 1. Search Posts

Search posts by caption content with optional filtering by post type.

**Endpoint:** `GET /api/posts/search`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| q | String | Yes | Search query (1-100 characters) |
| type | String | No | Filter by post type: `FEED`, `STORY`, or `REEL` |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/posts/search?q=summer&type=FEED&page=1&size=25
```

**Response (200 OK):**
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
- `401` - Unauthorized

---

## TRENDING & POPULAR POSTS

### 2. Get Trending Posts

Get posts sorted by engagement score (likes×2 + comments×3 + shares×4) from the last N days.

**Endpoint:** `GET /api/posts/trending`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| days | String | No | Number of days to look back (default: 7) |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/posts/trending?days=7&page=1&size=25
```

**Response (200 OK):**
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

**Error Responses:**
- `401` - Unauthorized

**Notes:**
- Engagement score calculation: `likes × 2 + comments × 3 + shares × 4`
- Posts are sorted by engagement score (highest first)

---

### 3. Get Popular Posts

Get posts with the most total engagement (likes + comments + shares) from the last N days.

**Endpoint:** `GET /api/posts/popular`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| days | String | No | Number of days to look back (default: 30) |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/posts/popular?days=30&page=1&size=25
```

**Response (200 OK):**
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

**Error Responses:**
- `401` - Unauthorized

**Notes:**
- Only posts with engagement > 0 are included
- Sorted by total engagement (likes + comments + shares)
- Default time period is 30 days (vs 7 days for trending)

---

## EXPLORE & DISCOVER

### 4. Get Explore Feed

Get a combined feed with trending posts, popular posts, and suggested users.

**Endpoint:** `GET /api/explore`

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
GET /api/explore?page=1&size=25
```

**Response (200 OK):**
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

**Error Responses:**
- `401` - Unauthorized

**Notes:**
- Combines top 5 trending posts and top 5 popular posts
- Includes top 5 suggested users
- Removes duplicate posts if they appear in both trending and popular

---

### 5. Get Suggested Users

Get users to follow based on activity, follower count, and mutual connections.

**Endpoint:** `GET /api/explore/suggested-users`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 20) |
| limit | String | No | Maximum suggestions to fetch (default: 20) |

**Example Request:**
```
GET /api/explore/suggested-users?page=1&size=20&limit=20
```

**Response (200 OK):**
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

**Error Responses:**
- `401` - Unauthorized
- `404` - Profile not found

**Notes:**
- Excludes users you already follow, are friends with, or have blocked
- Scoring algorithm:
  - Active users (posted in last 30 days): +100 points
  - Follower count: ×2 points
  - Post count: ×1 point
- Shows mutual connection count (users who follow both you and the suggested user)
- Sorted by score (highest first)

---

## Response Fields Explanation

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

## Example Postman Collection

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

## GET POSTS BY PROFILE

### 6. Get Posts by Profile ID

Get all posts created by a specific user or seller profile.

**Endpoint:** `GET /api/posts/profile/:profileId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `profileId` | UUID | Yes | Profile ID of the user or seller |

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `page` | String | No | Page number (default: 1) |
| `size` | String | No | Items per page (default: 25) |
| `includeReposts` | String | No | Set to `"true"` to include reposts mixed with original posts |

**Visibility Rules:**
- **Own Profile:** Shows all posts (PUBLIC and FOLLOWERS-only)
- **Other Profiles:**
  - Shows **PUBLIC** posts to everyone
  - Shows **FOLLOWERS-only** posts only if you follow them or are friends

**Example Request (Original Posts Only):**
```
GET /api/posts/profile/4afd7a15-1f48-40fe-86f8-58ba012ebba6?page=1&size=25
```

**Example Request (Include Reposts):**
```
GET /api/posts/profile/4afd7a15-1f48-40fe-86f8-58ba012ebba6?page=1&size=25&includeReposts=true
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 10,
  "data": [
    {
      "id": "f5674766-bb0c-4ccf-bfa3-6c6f4b57b5de",
      "type": "REEL",
      "caption": "Beautiful sunset today! #sunset",
      "mediaUrl": "https://res.cloudinary.com/...",
      "visibility": "PUBLIC",
      "createdAt": "2025-12-14T18:34:22.562Z",
      "updatedAt": "2025-12-14T18:34:22.562Z",
      "author": {
        "id": "4afd7a15-1f48-40fe-86f8-58ba012ebba6",
        "displayName": "Abanoub Youssef 2",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "counts": {
        "likes": 1,
        "comments": 3,
        "shares": 0
      },
      "likedByCurrentUser": false,
      "isMine": false,
      "isFollow": false,
      "isFriend": true,
      "repostedByCurrentUser": false,
      "isRepostedByMine": false,
      "isSaved": false
    }
  ]
}
```

**Error Responses:**
- `400` - Invalid profile ID format
- `401` - Unauthorized
- `404` - Profile not found
- `403` - Profile is not active

**Notes:**
- Works for both USER and SELLER profiles
- **Default:** Shows only original posts created by the profile
- **With `includeReposts=true`:** Shows both original posts and reposts mixed together
- Reposts include `isRepost: true` and `repostInfo` object with repost details
- Respects post visibility settings (PUBLIC vs FOLLOWERS) for both posts and reposts
- Friend detection checks both friendships table and accepted friend requests
- All items sorted by creation date (most recent first)

**Postman Examples:**

Original Posts Only:
```
GET {{baseUrl}}/api/posts/profile/4afd7a15-1f48-40fe-86f8-58ba012ebba6?page=1&size=25
Authorization: {{token}}
```

Include Reposts:
```
GET {{baseUrl}}/api/posts/profile/4afd7a15-1f48-40fe-86f8-58ba012ebba6?page=1&size=25&includeReposts=true
Authorization: {{token}}
```

---

## Best Practices

1. **Search Posts:**
   - Use specific keywords for better results
   - Filter by type if you only want specific post types
   - Use pagination for large result sets

2. **Trending vs Popular:**
   - Use **Trending** for recent viral content (last 7 days)
   - Use **Popular** for all-time best content (last 30 days)

3. **Suggested Users:**
   - Higher `limit` returns more suggestions but may be slower
   - Check `mutualConnections` to prioritize users with mutual friends
   - Consider user's `stats` to find active creators

4. **Pagination:**
   - Default page size is 25, adjust based on your needs
   - Use `total` field to determine total pages
   - Always check if `data` array is empty before processing

