# Social APIs - Postman Quick Reference

## 🔐 Authentication
All endpoints need: `Authorization: <token>` header

---

## 👥 FRIENDS

### Get Friends
```
GET /api/friends
```

### Send Friend Request
```
POST /api/friends/requests
Body: { "targetUserId": "uuid" }
```

### Get Pending Requests
```
GET /api/friends/requests/pending
```

### Accept Request
```
POST /api/friends/requests/:id/accept
```

### Reject Request
```
POST /api/friends/requests/:id/reject
```

### Remove Friend
```
DELETE /api/friends/:friendId
```

---

## 📢 FOLLOWS

### Follow User
```
POST /api/follows
Body: { "targetUserId": "uuid" }
```

### Unfollow
```
DELETE /api/follows/:targetUserId
```

### Get Following
```
GET /api/follows/following
```

### Get Followers (SELLER/MEDIATOR only)
```
GET /api/follows/followers
```

---

## 📸 POSTS

### Create Post (with Images/Video)
```
POST /api/posts
Content-Type: multipart/form-data
Body:
  - caption: "text"
  - type: "FEED" | "STORY" | "REEL"
  - postImages: [files] (up to 5, 10MB each)
  - postVideos: [file] (1 file, 10MB max)
```

### Get Feed
```
GET /api/posts/feed?page=1&size=25
```

### Get Single Post
```
GET /api/posts/:postId
```

### Update Post
```
PATCH /api/posts/:postId
Body: { "caption": "...", "mediaUrl": "..." }
```

### Delete Post
```
DELETE /api/posts/:postId
```

---

## ❤️ LIKES

### Toggle Post Like
```
POST /api/likes/posts/:postId
```

### Get Post Likes
```
GET /api/likes/posts/:postId
```

### Toggle Comment Like
```
POST /api/likes/comments/:commentId
```

---

## 💬 COMMENTS

### Create Comment
```
POST /api/comments/posts/:postId
Body: {
  "content": "comment text",
  "parentCommentId": "uuid" (optional, for replies)
}
```

### Get Comments
```
GET /api/comments/posts/:postId
```

### Delete Comment
```
DELETE /api/comments/:commentId
```

---

## 📋 Example Bodies

### Send Friend Request
```json
{
  "targetUserId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Follow User
```json
{
  "targetUserId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Create Comment
```json
{
  "content": "This is my comment",
  "parentCommentId": null
}
```

### Reply to Comment
```json
{
  "content": "This is a reply",
  "parentCommentId": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## 🎯 File Upload in Postman

### For Post Creation:
1. Set method: **POST**
2. URL: `/api/posts`
3. Headers: `Authorization: <token>` (remove Content-Type - Postman sets it)
4. Body tab → Select **form-data**
5. Add fields:
   - `caption` (Text): "My post caption"
   - `type` (Text): "FEED" (optional)
   - `postImages` (File): Select files (up to 5)
     - Or `postVideos` (File): Select 1 video file

---

## 🔑 Quick Copy-Paste URLs

Replace `<base_url>` with your server URL (e.g., `http://localhost:3000`)

```
<base_url>/api/friends
<base_url>/api/friends/requests
<base_url>/api/friends/requests/pending
<base_url>/api/follows
<base_url>/api/follows/following
<base_url>/api/follows/followers
<base_url>/api/posts
<base_url>/api/posts/feed
<base_url>/api/posts/:postId
<base_url>/api/likes/posts/:postId
<base_url>/api/comments/posts/:postId
```

---

**Note:** All `:id` or `:postId` values should be replaced with actual UUIDs in your requests.

