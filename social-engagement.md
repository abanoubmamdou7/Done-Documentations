# Engagement Features API Documentation

## Overview
This document consolidates all the new engagement features APIs including Activity Feed, Reports, Bookmarks, Hashtags, and Admin endpoints.

**Base URLs:**
- Activity: `/api/activity`
- Reports: `/api/reports`
- Bookmarks: `/api/bookmarks`
- Hashtags: `/api/hashtags`
- Admin: `/api/admin`

---

## Table of Contents

1. [Activity Feed](#activity-feed)
2. [Reports](#reports)
3. [Bookmarks](#bookmarks)
4. [Hashtags](#hashtags)
5. [Admin Endpoints](#admin-endpoints)
6. [Post Visibility Updates](#post-visibility-updates)

---

## Activity Feed

### Get Activity Feed
**Endpoint:** `GET /api/activity/feed`

View recent activity from friends and following users.

**Query Parameters:**
- `type` (optional): `all`, `posts`, `likes`, `comments`, `reposts`
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25)

**Example:**
```http
GET /api/activity/feed?type=all&page=1&size=25
Authorization: Bearer <token>
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 50,
  "data": [
    {
      "type": "POST",
      "activityId": "550e8400-e29b-41d4-a716-446655440000",
      "actor": { "id": "...", "displayName": "John Doe" },
      "target": { "type": "post", "id": "...", "caption": "..." },
      "timestamp": "2024-01-15T10:30:00Z"
    }
  ]
}
```

**See:** [Full Activity Feed Documentation](./activity/POSTMAN_API_DOCUMENTATION.md)

---

## Reports

### Create Report
**Endpoint:** `POST /api/reports`

Report inappropriate content (posts, comments, profiles, stories).

**Request Body:**
```json
{
  "type": "POST",
  "reason": "SPAM",
  "description": "This post contains spam",
  "postId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Report Types:** `POST`, `COMMENT`, `PROFILE`, `STORY`
**Report Reasons:** `SPAM`, `HARASSMENT`, `INAPPROPRIATE`, `COPYRIGHT`, `FALSE_INFO`, `OTHER`

### Get My Reports
**Endpoint:** `GET /api/reports/my-reports`

Get all reports submitted by the authenticated user.

**Query Parameters:**
- `status` (optional): Filter by status
- `page`, `size` (optional): Pagination

**See:** [Full Reports Documentation](./reports/POSTMAN_API_DOCUMENTATION.md)

---

## Bookmarks

### Save Post
**Endpoint:** `POST /api/bookmarks/:postId`

Save a post to bookmarks.

### Unsave Post
**Endpoint:** `DELETE /api/bookmarks/:postId`

Remove a post from bookmarks.

### Get Saved Posts
**Endpoint:** `GET /api/bookmarks`

Get all saved posts.

**Query Parameters:**
- `page`, `size` (optional): Pagination

### Check if Saved
**Endpoint:** `GET /api/bookmarks/:postId/check`

Check if a post is saved.

**See:** [Full Bookmarks Documentation](./bookmarks/POSTMAN_API_DOCUMENTATION.md)

---

## Hashtags

### Search Hashtags
**Endpoint:** `GET /api/hashtags/search`

Search for hashtags by name.

**Query Parameters:**
- `q` (required): Search query
- `page`, `size` (optional): Pagination

### Get Trending Hashtags
**Endpoint:** `GET /api/hashtags/trending`

Get trending hashtags based on recent activity.

**Query Parameters:**
- `days` (optional): Days to look back (default: 7)
- `page`, `size` (optional): Pagination

### Get Posts by Hashtag
**Endpoint:** `GET /api/hashtags/:hashtagId/posts`

Get all posts with a specific hashtag.

**Query Parameters:**
- `page`, `size` (optional): Pagination

**See:** [Full Hashtags Documentation](./hashtags/POSTMAN_API_DOCUMENTATION.md)

---

## Admin Endpoints

**Note:** All admin endpoints require ADMIN or MEDIATOR role.

### Get All Reports
**Endpoint:** `GET /api/admin/reports`

Get all reports for moderation.

**Query Parameters:**
- `status` (optional): Filter by status
- `type` (optional): Filter by type
- `page`, `size` (optional): Pagination

### Review Report
**Endpoint:** `PATCH /api/admin/reports/:reportId`

Update report status and resolution.

**Request Body:**
```json
{
  "status": "RESOLVED",
  "resolution": "Post removed. User warned."
}
```

### Get All Users
**Endpoint:** `GET /api/admin/users`

Get list of all users.

**Query Parameters:**
- `role` (optional): Filter by role
- `search` (optional): Search by name/phone
- `page`, `size` (optional): Pagination

### Block/Unblock User
**Endpoint:** `PATCH /api/admin/users/:profileId/block`

Block or unblock a user.

**Request Body:**
```json
{
  "isBlocked": true
}
```

### Delete Post
**Endpoint:** `DELETE /api/admin/posts/:postId`

Delete a post as admin.

### Delete Comment
**Endpoint:** `DELETE /api/admin/comments/:commentId`

Delete a comment as admin.

### Get Analytics
**Endpoint:** `GET /api/admin/analytics`

Get platform analytics.

**Query Parameters:**
- `days` (optional): Days to look back (default: 30)

**See:** [Full Admin Documentation](../admin/POSTMAN_API_DOCUMENTATION.md)

---

## Post Visibility Updates

### Create Post with Visibility

When creating a post, you can now specify visibility:

**Endpoint:** `POST /api/posts`

**Request Body:**
```json
{
  "caption": "Beautiful sunset! #sunset #nature",
  "visibility": "FOLLOWERS",
  "hashtags": ["sunset", "nature"]
}
```

**Visibility Options:**
- `PUBLIC`: Visible to everyone (default)
- `FOLLOWERS`: Only visible to followers

### Update Post Visibility

**Endpoint:** `PATCH /api/posts/:postId`

**Request Body:**
```json
{
  "visibility": "PUBLIC",
  "hashtags": ["updated", "hashtags"]
}
```

**Note:** The feed endpoint (`GET /api/posts/feed`) now respects visibility settings:
- Shows PUBLIC posts to everyone
- Shows FOLLOWERS-only posts only to users who follow the author

---

## Postman Environment Variables

Set up these variables in your Postman environment:

```
base_url: http://localhost:3000
auth_token: <your-jwt-token>
admin_token: <admin-jwt-token>
```

---

## Quick Start Examples

### 1. View Activity Feed
```
GET {{base_url}}/api/activity/feed?type=all
Authorization: Bearer {{auth_token}}
```

### 2. Report a Post
```
POST {{base_url}}/api/reports
Authorization: Bearer {{auth_token}}
Content-Type: application/json

{
  "type": "POST",
  "reason": "SPAM",
  "postId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### 3. Save a Post
```
POST {{base_url}}/api/bookmarks/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{auth_token}}
```

### 4. Search Hashtags
```
GET {{base_url}}/api/hashtags/search?q=sunset
Authorization: Bearer {{auth_token}}
```

### 5. Get Trending Hashtags
```
GET {{base_url}}/api/hashtags/trending?days=7
Authorization: Bearer {{auth_token}}
```

### 6. Admin: Review Report
```
PATCH {{base_url}}/api/admin/reports/990e8400-e29b-41d4-a716-446655440000
Authorization: Bearer {{admin_token}}
Content-Type: application/json

{
  "status": "RESOLVED",
  "resolution": "Issue resolved"
}
```

---

## Feature Summary

✅ **Activity Feed** - Timeline of friend/following activity  
✅ **Post Visibility** - Public vs Followers-only posts  
✅ **Reports** - Report inappropriate content  
✅ **Bookmarks** - Save posts for later  
✅ **Hashtags** - Tag, search, and discover content  
✅ **Admin Tools** - User management, moderation, analytics  

All features are fully implemented and ready to use!

