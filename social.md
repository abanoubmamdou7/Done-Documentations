# Social Module API Documentation for Postman

## 📋 Table of Contents
1. [Authentication](#authentication)
2. [Friends APIs](#friends-apis)
3. [Follows APIs](#follows-apis)
4. [Posts APIs](#posts-apis)
5. [Likes APIs](#likes-apis)
6. [Comments APIs](#comments-apis)
7. [Reposts APIs](#reposts-apis)

---

## 🔐 Authentication

All protected endpoints require authentication via JWT token in the Authorization header.

**Header:**
```
Authorization: <your_jwt_token>
```

**Note:** Get your token from the auth/login endpoint first.

---

## 👥 Friends APIs

**Base URL:** `/api/friends`

### 1. Get Friends List
**GET** `/api/friends`

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "success": true,
  "count": 2,
  "data": [
    {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER",
      "since": "2024-01-15T10:30:00.000Z"
    }
  ]
}
```

---

### 2. Send Friend Request
**POST** `/api/friends/requests`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "targetUserId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "requesterId": "your-user-id",
    "addresseeId": "target-user-id",
    "status": "PENDING",
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

### 3. Get Pending Friend Requests
**GET** `/api/friends/requests/pending`

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "success": true,
  "data": {
    "received": [
      {
        "id": "uuid",
        "requesterId": "user-id",
        "addresseeId": "your-id",
        "status": "PENDING",
        "requester": {
          "id": "uuid",
          "displayName": "John Doe",
          "avatarUrl": "https://..."
        }
      }
    ],
    "sent": []
  }
}
```

---

### 4. Accept Friend Request
**POST** `/api/friends/requests/:id/accept`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `id` - Friend request ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": {
    "friendship": {
      "id": "uuid",
      "user1Id": "...",
      "user2Id": "...",
      "createdAt": "2024-01-15T10:30:00.000Z"
    },
    "request": {
      "id": "uuid",
      "status": "ACCEPTED",
      "respondedAt": "2024-01-15T10:35:00.000Z"
    }
  }
}
```

---

### 5. Reject Friend Request
**POST** `/api/friends/requests/:id/reject`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `id` - Friend request ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "REJECTED",
    "respondedAt": "2024-01-15T10:35:00.000Z"
  }
}
```

---

### 6. Remove Friend
**DELETE** `/api/friends/:friendId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `friendId` - Friend's user ID (UUID)

**Response:**
```json
{
  "success": true,
  "message": "Friend removed successfully"
}
```

---

## 📢 Follows APIs

**Base URL:** `/api/follows`

### 1. Follow User (SELLER/MEDIATOR)
**POST** `/api/follows`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "targetUserId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "followerId": "your-user-id",
    "targetUserId": "target-user-id",
    "targetType": "SELLER",
    "createdAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

### 2. Unfollow User
**DELETE** `/api/follows/:targetUserId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `targetUserId` - User ID to unfollow (UUID)

**Response:**
```json
{
  "success": true,
  "message": "Unfollowed successfully"
}
```

---

### 3. Get Following List
**GET** `/api/follows/following`

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "success": true,
  "count": 2,
  "data": [
    {
      "id": "uuid",
      "followerId": "your-id",
      "targetUserId": "target-id",
      "targetType": "SELLER",
      "targetUser": {
        "id": "uuid",
        "displayName": "Shop Name",
        "avatarUrl": "https://...",
        "role": "SELLER"
      }
    }
  ]
}
```

---

### 4. Get Followers (SELLER/MEDIATOR only)
**GET** `/api/follows/followers`

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "success": true,
  "count": 5,
  "data": [
    {
      "id": "uuid",
      "followerId": "follower-id",
      "targetUserId": "your-id",
      "follower": {
        "id": "uuid",
        "displayName": "Follower Name",
        "avatarUrl": "https://...",
        "role": "USER"
      }
    }
  ]
}
```

---

## 📸 Posts APIs

**Base URL:** `/api/posts`

### 1. Create Post (with Images/Video)
**POST** `/api/posts`

**Headers:**
```
Authorization: <token>
Content-Type: multipart/form-data
```

**Body (Form Data):**
- `caption` (text): "My awesome post caption"
- `type` (text, optional): "FEED", "STORY", or "REEL" (default: "FEED")
- `postImages` (file): Upload up to 5 images
  - Supported: PNG, JPEG, JPG, WEBP
  - Max size: 10MB per file
- `postVideos` (file): Upload 1 video (max)
  - Supported: MP4, MPEG, QuickTime, WebM, AVI, WMV
  - Max size: 10MB per file

**Note:** 
- You can upload either images OR video, not both
- If you upload images, they will be stored as comma-separated URLs
- If you upload video, type will automatically be set to "REEL"

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "authorId": "your-user-id",
    "type": "FEED",
    "caption": "My awesome post caption",
    "mediaUrl": "https://res.cloudinary.com/.../image1.jpg,https://res.cloudinary.com/.../image2.jpg",
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

---

### 2. Create Post (Text Only - No Files)
**POST** `/api/posts`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "caption": "Just a text post",
  "type": "FEED"
}
```

---

### 3. Get Feed
**GET** `/api/posts/feed?page=1&size=25&feedType=following`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 25)
- `feedType` (optional): Feed type - `"for_you"` or `"following"` (default: `"following"`)
  - **`"following"`**: Shows only posts from friends and users you follow (personalized feed)
  - **`"for_you"`**: Shows all posts from all users (discovery feed, similar to TikTok's "For You" page)

**Examples:**

**Following Feed (default):**
```
GET /api/posts/feed?feedType=following
GET /api/posts/feed  (defaults to following)
```

**For You Feed:**
```
GET /api/posts/feed?feedType=for_you
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 50,
  "feedType": "following",
  "data": [
    {
      "id": "uuid",
      "type": "FEED",
      "caption": "Post caption",
      "mediaUrl": "https://...",
      "createdAt": "2024-01-15T10:30:00.000Z",
      "updatedAt": "2024-01-15T10:30:00.000Z",
      "author": {
        "id": "uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      },
    "counts": {
      "likes": 10,
      "comments": 5,
      "shares": 3
    },
    "likedByCurrentUser": false,
    "repostedByCurrentUser": false
    }
  ]
}
```

**Notes:**
- The `feedType` field in the response indicates which feed type is being returned
- "Following" feed only includes posts from users you're friends with or follow
- "For you" feed includes all posts from all users for content discovery

---

### 4. Get Single Post
**GET** `/api/posts/:postId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "type": "FEED",
    "caption": "Post caption",
    "mediaUrl": "https://...",
    "author": {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER"
    },
    "counts": {
      "likes": 10,
      "comments": 5,
      "shares": 3
    },
    "likedByCurrentUser": true,
    "repostedByCurrentUser": false
  }
}
```

---

### 5. Update Post
**PATCH** `/api/posts/:postId`

**Headers:**
```
Authorization: <token>
```
> **Note:** When uploading files, use `multipart/form-data`. Postman/curl will set this automatically. For JSON-only updates, use `Content-Type: application/json`.

**URL Parameters:**
- `postId` - Post ID (UUID)

**Body Options:**

**Option 1: Update with File Uploads (multipart/form-data)**
- `caption` (optional) - Updated caption text
- `postImages` (optional) - Multiple image files (max 5). Old images will be automatically deleted from Cloudinary.
- `postVideos` (optional) - Single video file (max 1). Old video will be automatically deleted from Cloudinary.

> **Note:** When uploading new files, the old media files are automatically deleted from Cloudinary. The post type will be automatically set to `FEED` for images or `REEL` for videos.

**Option 2: Update Caption Only (JSON)**
```json
{
  "caption": "Updated caption"
}
```

**Option 3: Update with External Media URL (JSON)**
```json
{
  "caption": "Updated caption",
  "mediaUrl": "https://external-url.com/image.jpg"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "type": "FEED",
    "caption": "Updated caption",
    "mediaUrl": "https://res.cloudinary.com/.../image.jpg",
    "authorId": "uuid",
    "createdAt": "2024-01-15T10:00:00.000Z",
    "updatedAt": "2024-01-15T11:00:00.000Z"
  }
}
```

**Notes:**
- Only the post author can update their post
- Uploading new images/videos automatically replaces old media in Cloudinary
- Post type is automatically set based on uploaded media (FEED for images, REEL for videos)
- You can update caption without changing media
- Maximum 5 images or 1 video per update

---

### 6. Delete Post
**DELETE** `/api/posts/:postId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "message": "Post deleted successfully"
}
```

---

### 7. Repost a Post
**POST** `/api/posts/:postId/repost`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Body (JSON):**
```json
{
  "caption": "Check out this amazing post! (Optional caption)"
}
```

**Note:** The `caption` field is optional. If not provided, the repost will be created without a caption.

**Response:**
```json
{
  "success": true,
  "message": "Post reposted successfully",
  "data": {
    "id": "1234567890",
    "postId": "uuid",
    "caption": "Check out this amazing post!",
    "createdAt": "2024-01-15T11:00:00.000Z",
    "originalPost": {
      "id": "uuid",
      "type": "FEED",
      "caption": "Original post caption",
      "mediaUrl": "https://...",
      "author": {
        "id": "uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      }
    },
    "repostedBy": {
      "id": "uuid",
      "displayName": "Your Name",
      "avatarUrl": "https://...",
      "role": "USER"
    }
  }
}
```

**Error Responses:**
- `400` - You have already reposted this post
- `404` - Post not found

---

### 8. Undo Repost (Remove Repost)
**DELETE** `/api/posts/:postId/repost`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "message": "Repost removed successfully"
}
```

**Error Responses:**
- `404` - Post not found or You have not reposted this post

---

### 9. Get Reposts for a Post
**GET** `/api/posts/:postId/reposts?page=1&size=25`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 25)

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 10,
  "data": [
    {
      "id": "1234567890",
      "postId": "uuid",
      "caption": "Great post!",
      "createdAt": "2024-01-15T11:00:00.000Z",
      "repostedBy": {
        "id": "uuid",
        "displayName": "Jane Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      }
    }
  ]
}
```

**Notes:**
- Returns paginated list of users who reposted the post
- Includes optional captions added by users when reposting
- Ordered by most recent reposts first

---

## ❤️ Likes APIs

**Base URL:** `/api/likes`

### 1. Toggle Post Like
**POST** `/api/likes/posts/:postId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": {
    "liked": true,
    "totalLikes": 11
  }
}
```

**Note:** If already liked, this will unlike (liked: false)

---

### 2. Get Post Likes (Users who liked)
**GET** `/api/likes/posts/:postId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "count": 11,
  "data": [
    {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER"
    }
  ]
}
```

---

### 3. Toggle Comment Like
**POST** `/api/likes/comments/:commentId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `commentId` - Comment ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": {
    "liked": true,
    "totalLikes": 5
  }
}
```

---

## 💬 Comments APIs

**Base URL:** `/api/comments`

### 1. Create Comment
**POST** `/api/comments/posts/:postId`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Body (JSON):**
```json
{
  "content": "This is a great post!",
  "parentCommentId": null
}
```

**For Reply (Threaded Comment):**
```json
{
  "content": "I agree with you!",
  "parentCommentId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "uuid",
    "postId": "post-uuid",
    "authorId": "your-user-id",
    "content": "This is a great post!",
    "parentId": null,
    "createdAt": "2024-01-15T10:30:00.000Z",
    "author": {
      "id": "uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "role": "USER"
    }
  }
}
```

---

### 2. Get Post Comments
**GET** `/api/comments/posts/:postId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `postId` - Post ID (UUID)

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "postId": "post-uuid",
      "authorId": "user-id",
      "content": "Great post!",
      "parentId": null,
      "createdAt": "2024-01-15T10:30:00.000Z",
      "author": {
        "id": "uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "_count": {
        "replies": 2,
        "likes": 3
      },
      "replies": [
        {
          "id": "uuid",
          "content": "I agree!",
          "author": {
            "id": "uuid",
            "displayName": "Jane Doe",
            "avatarUrl": "https://..."
          }
        }
      ]
    }
  ]
}
```

---

### 3. Delete Comment
**DELETE** `/api/comments/:commentId`

**Headers:**
```
Authorization: <token>
```

**URL Parameters:**
- `commentId` - Comment ID (UUID)

**Response:**
```json
{
  "success": true,
  "message": "Comment deleted successfully"
}
```

---

## 📝 Quick Reference

### Base URLs
- Friends: `http://localhost:3000/api/friends`
- Follows: `http://localhost:3000/api/follows`
- Posts: `http://localhost:3000/api/posts`
- Likes: `http://localhost:3000/api/likes`
- Comments: `http://localhost:3000/api/comments`

### Common Headers
```
Authorization: <jwt_token>
Content-Type: application/json  (for JSON requests)
Content-Type: multipart/form-data  (for file uploads)
```

### File Upload Limits
- **Images:** Up to 5 files, 10MB each
  - Formats: PNG, JPEG, JPG, WEBP
- **Videos:** 1 file, 10MB
  - Formats: MP4, MPEG, QuickTime, WebM, AVI, WMV

---

## 🧪 Testing Workflow

### Suggested Testing Order:

1. **Authentication** - Login to get token
2. **Create Post** - Upload images/video
3. **Get Feed** - View posts in feed
4. **Like Post** - Toggle like on post
5. **Create Comment** - Add comment to post
6. **Like Comment** - Toggle like on comment
7. **Send Friend Request** - Request friendship
8. **Accept Friend Request** - Accept request
9. **Follow Seller** - Follow a seller/mediator
10. **Get Following** - View who you follow

---

## ⚠️ Important Notes

1. **Authentication Required:** All endpoints require valid JWT token
2. **Role Restrictions:**
   - Friend requests: Only USER role
   - Follow: Only USER can follow SELLER/MEDIATOR
   - Get Followers: Only SELLER/MEDIATOR
3. **File Uploads:** Use `multipart/form-data` in Postman
4. **UUID Format:** All IDs are UUIDs (e.g., `550e8400-e29b-41d4-a716-446655440000`)
5. **Pagination:** Default page=1, size=25 (max 25)

---

**Happy Testing! 🚀**

