# Postman Testing Guide - Step by Step

## 🚀 Setup

### 1. Environment Variables in Postman
Create a new environment with:
- `base_url`: `http://localhost:3000` (or your server URL)
- `token`: (will be set after login)

Then use in requests: `{{base_url}}/api/posts`

---

## 📝 Step-by-Step Testing

### Step 1: Get Authentication Token
**If you don't have a token yet:**
1. Test your login endpoint (from auth module)
2. Copy the token from response
3. Set as environment variable `{{token}}`

**In Postman:**
- Create header: `Authorization: {{token}}`

---

### Step 2: Create a Post with Images

1. **Method:** POST
2. **URL:** `{{base_url}}/api/posts`
3. **Headers:**
   ```
   Authorization: {{token}}
   ```
   (Don't set Content-Type - Postman will set it automatically for form-data)
4. **Body Tab:**
   - Select **form-data**
   - Add fields:
     - Key: `caption` | Type: **Text** | Value: `My first post with images!`
     - Key: `type` | Type: **Text** | Value: `FEED`
     - Key: `postImages` | Type: **File** | Value: Select image file (repeat for up to 5 images)
5. **Click Send**

**Expected:** 201 Created with post data including Cloudinary URLs

---

### Step 3: Create a Post with Video

1. **Method:** POST
2. **URL:** `{{base_url}}/api/posts`
3. **Headers:** `Authorization: {{token}}`
4. **Body Tab:**
   - Select **form-data**
   - Add:
     - Key: `caption` | Type: **Text** | Value: `Check out my video!`
     - Key: `postVideos` | Type: **File** | Value: Select video file (1 file max)
5. **Click Send**

**Expected:** 201 Created, type automatically set to "REEL"

---

### Step 4: Create Text-Only Post

1. **Method:** POST
2. **URL:** `{{base_url}}/api/posts`
3. **Headers:**
   ```
   Authorization: {{token}}
   Content-Type: application/json
   ```
4. **Body Tab:**
   - Select **raw** → **JSON**
   - Body:
   ```json
   {
     "caption": "Just a text post, no media",
     "type": "FEED"
   }
   ```
5. **Click Send**

---

### Step 5: Get Feed

1. **Method:** GET
2. **URL:** `{{base_url}}/api/posts/feed?page=1&size=10`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

**Expected:** List of posts from friends/followed users

---

### Step 6: Get Single Post

1. **Method:** GET
2. **URL:** `{{base_url}}/api/posts/<post-id-from-step-2>`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

---

### Step 7: Like a Post

1. **Method:** POST
2. **URL:** `{{base_url}}/api/likes/posts/<post-id>`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

**Expected:** `{ "liked": true, "totalLikes": 1 }`

5. **Send again** - should unlike: `{ "liked": false, "totalLikes": 0 }`

---

### Step 8: Create Comment

1. **Method:** POST
2. **URL:** `{{base_url}}/api/comments/posts/<post-id>`
3. **Headers:**
   ```
   Authorization: {{token}}
   Content-Type: application/json
   ```
4. **Body (raw JSON):**
   ```json
   {
     "content": "Great post!",
     "parentCommentId": null
   }
   ```
5. **Click Send**

---

### Step 9: Reply to Comment (Threaded)

1. **Method:** POST
2. **URL:** `{{base_url}}/api/comments/posts/<post-id>`
3. **Headers:** `Authorization: {{token}}`, `Content-Type: application/json`
4. **Body:**
   ```json
   {
     "content": "I totally agree!",
     "parentCommentId": "<comment-id-from-step-8>"
   }
   ```
5. **Click Send**

---

### Step 10: Get Post Comments

1. **Method:** GET
2. **URL:** `{{base_url}}/api/comments/posts/<post-id>`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

**Expected:** List of comments with replies

---

### Step 11: Send Friend Request

**Prerequisites:** You need two USER accounts (not SELLER/MEDIATOR)

1. **Method:** POST
2. **URL:** `{{base_url}}/api/friends/requests`
3. **Headers:** `Authorization: {{token}}`, `Content-Type: application/json`
4. **Body:**
   ```json
   {
     "targetUserId": "<another-user-id>"
   }
   ```
5. **Click Send**

---

### Step 12: Get Pending Requests

1. **Method:** GET
2. **URL:** `{{base_url}}/api/friends/requests/pending`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

**Expected:** `{ "received": [...], "sent": [...] }`

---

### Step 13: Accept Friend Request

1. **Method:** POST
2. **URL:** `{{base_url}}/api/friends/requests/<request-id>/accept`
3. **Headers:** `Authorization: {{token}}` (must be the receiver)
4. **Click Send**

---

### Step 14: Get Friends List

1. **Method:** GET
2. **URL:** `{{base_url}}/api/friends`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

---

### Step 15: Follow a Seller/Mediator

**Prerequisites:** You need a USER account and a SELLER or MEDIATOR account ID

1. **Method:** POST
2. **URL:** `{{base_url}}/api/follows`
3. **Headers:** `Authorization: {{token}}`, `Content-Type: application/json`
4. **Body:**
   ```json
   {
     "targetUserId": "<seller-or-mediator-id>"
   }
   ```
5. **Click Send**

---

### Step 16: Get Following List

1. **Method:** GET
2. **URL:** `{{base_url}}/api/follows/following`
3. **Headers:** `Authorization: {{token}}`
4. **Click Send**

---

### Step 17: Get Followers (SELLER/MEDIATOR only)

1. **Method:** GET
2. **URL:** `{{base_url}}/api/follows/followers`
3. **Headers:** `Authorization: {{token}}` (must be SELLER or MEDIATOR)
4. **Click Send**

---

## 🐛 Common Issues

### Issue: "Authorization is required"
**Solution:** Make sure you have `Authorization` header with valid token

### Issue: "You cannot send a friend request to yourself"
**Solution:** Use a different user ID

### Issue: "Friend requests are limited to USER accounts only"
**Solution:** Both users must have role="USER"

### Issue: "Only USER accounts can follow others"
**Solution:** Only USER role can follow SELLER/MEDIATOR

### Issue: File upload fails
**Solution:** 
- Make sure file size < 10MB
- Check file format is supported
- Use `form-data` not `raw` in Postman body

### Issue: "Post not found"
**Solution:** Use valid UUID format for post ID

---

## ✅ Testing Checklist

- [ ] Create post with images (up to 5)
- [ ] Create post with video
- [ ] Create text-only post
- [ ] Get feed (paginated)
- [ ] Get single post
- [ ] Like/unlike post
- [ ] Create comment
- [ ] Reply to comment (threaded)
- [ ] Get comments for post
- [ ] Delete comment
- [ ] Send friend request
- [ ] Accept friend request
- [ ] Get friends list
- [ ] Follow seller/mediator
- [ ] Get following list
- [ ] Unfollow user
- [ ] Remove friend

---

## 📸 Postman Collection Tips

1. **Create Collection:** "Social APIs"
2. **Create Folders:**
   - Friends
   - Follows
   - Posts
   - Likes
   - Comments
3. **Set Collection Variables:**
   - `base_url`: `http://localhost:3000`
4. **Save tokens** in environment variables
5. **Use Pre-request Scripts** to auto-set Authorization header:
   ```javascript
   pm.request.headers.add({
       key: 'Authorization',
       value: pm.environment.get('token')
   });
   ```

---

**Happy Testing! 🎉**

