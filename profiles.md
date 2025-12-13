# Profile Module API Documentation for Postman

## 📋 Table of Contents
1. [Authentication](#authentication)
2. [Profile Management APIs](#profile-management-apis)
3. [Profile Discovery APIs](#profile-discovery-apis)
4. [Profile Relationships APIs](#profile-relationships-apis)
5. [Profile Settings APIs](#profile-settings-apis)
6. [Category Preferences APIs](#category-preferences-apis)
7. [Posts by Profile APIs](#posts-by-profile-apis)

---

## 🔐 Authentication

All protected endpoints require authentication via JWT token in the Authorization header.

**Header:**
```
Authorization: <your_jwt_token>
```

**Note:** Get your token from the auth/login endpoint first.

**Base URL:** `http://localhost:YOUR_PORT/api/profile` (or your server URL)

---

## 👤 Profile Management APIs

### 1. Get My Profile
**GET** `/api/profile/me`

**Description:** Retrieve the authenticated user's own profile with statistics

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "message": "Profile retrieved successfully",
  "data": {
    "profile": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "email": "john@example.com",
      "displayName": "John Doe",
      "avatarUrl": "https://cloudinary.com/avatar.jpg",
      "phone": "1234567890",
      "countryCode": "+1",
      "gender": "MALE",
      "bio": "Fashion enthusiast | Love sharing my style with the world ✨",
      "role": "USER",
      "status": "Active",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z",
      "stats": {
        "posts": 42,
        "friends": 1250,
        "following": 856
      }
    }
  }
}
```

**Note:** For SELLER/MEDIATOR roles, stats will have `followers` instead of `friends`:
```json
{
  "stats": {
    "posts": 42,
    "followers": 1250,
    "following": 856
  }
}
```

---

### 2. Get Profile by ID
**GET** `/api/profile/:profileId`

**Description:** Retrieve a specific user's profile by their profile ID

**URL Parameters:**
```
profileId: string (required) - UUID of the profile
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET http://localhost:3000/api/profile/550e8400-e29b-41d4-a716-446655440000
```

**Response:**
```json
{
  "message": "Profile retrieved successfully",
  "data": {
    "profile": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "displayName": "John Doe",
      "avatarUrl": "https://cloudinary.com/avatar.jpg",
      "phone": "1234567890",
      "countryCode": "+1",
      "gender": "MALE",
      "bio": "Fashion enthusiast | Love sharing my style with the world ✨",
      "role": "USER",
      "status": "Active",
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z",
      "stats": {
        "posts": 42,
        "friends": 1250,
        "following": 856
      },
      "relationship": {
        "isFollowing": false,
        "isFriend": true
      }
    }
  }
}
```

---

### 3. Update Profile (Simple)
**PATCH** `/api/profile`

**Description:** Update profile information without requiring avatar upload

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "displayName": "John Doe Updated",
  "phone": "9876543210",
  "countryCode": "+1",
  "gender": "MALE",
  "bio": "Updated bio text"
}
```

**Note:** All fields are optional. Only include fields you want to update.

**Response:**
```json
{
  "message": "Profile updated successfully",
  "data": {
    "profile": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "username": "johndoe",
      "email": "john@example.com",
      "displayName": "John Doe Updated",
      "avatarUrl": "https://cloudinary.com/avatar.jpg",
      "phone": "9876543210",
      "countryCode": "+1",
      "gender": "MALE",
      "bio": "Updated bio text",
      "role": "USER",
      "status": "Active",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

---

### 4. Complete Profile (with Avatar Upload)
**PUT** `/api/profile/complete`

**Description:** Update profile information with optional avatar upload

**Headers:**
```
Authorization: <token>
Content-Type: multipart/form-data
```

**Body (Form Data):**
```
displayName: "John Doe" (optional)
phone: "1234567890" (optional)
countryCode: "+1" (optional)
gender: "MALE" | "FEMALE" | "OTHER" (optional)
bio: "Bio text" (optional)
avatar: <file> (optional, max 5MB, image file)
```

**Note:** In Postman, use form-data and set `avatar` as File type.

**Response:**
```json
{
  "message": "Profile updated successfully",
  "data": {
    "profile": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "role": "USER",
      "displayName": "John Doe",
      "avatarUrl": "https://cloudinary.com/new-avatar.jpg",
      "phone": "1234567890",
      "countryCode": "+1",
      "gender": "MALE",
      "bio": "Bio text",
      "status": "Active",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-15T10:30:00.000Z"
    }
  }
}
```

---

## 🔍 Profile Discovery APIs

### 5. Search Profiles
**GET** `/api/profile/search`

**Description:** Search for profiles by username or display name

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
```
q: string (required) - Search query (min 1, max 100 characters)
role: "USER" | "SELLER" | "MEDIATOR" (optional) - Filter by role
page: string (optional, default: "1") - Page number
size: string (optional, default: "25") - Items per page
```

**Example Request:**
```
GET http://localhost:3000/api/profile/search?q=john&role=USER&page=1&size=25
```

**Response:**
```json
{
  "message": "Profiles retrieved successfully",
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 10,
    "pages": 1
  },
  "data": {
    "profiles": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "username": "johndoe",
        "displayName": "John Doe",
        "avatarUrl": "https://cloudinary.com/avatar.jpg",
        "bio": "Fashion enthusiast",
        "role": "USER"
      }
    ]
  }
}
```

---

## 👥 Profile Relationships APIs

### 6. Get Followers List
**GET** `/api/profile/:profileId/followers`

**Description:** Get list of users following a specific profile (only for SELLER/MEDIATOR)

**URL Parameters:**
```
profileId: string (required) - UUID of the profile
```

**Query Parameters:**
```
page: string (optional, default: "1") - Page number
size: string (optional, default: "25") - Items per page
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET http://localhost:3000/api/profile/550e8400-e29b-41d4-a716-446655440000/followers?page=1&size=25
```

**Response:**
```json
{
  "message": "Followers retrieved successfully",
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 1250,
    "pages": 50
  },
  "data": {
    "followers": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "username": "follower1",
        "displayName": "Follower One",
        "avatarUrl": "https://cloudinary.com/avatar.jpg",
        "bio": "Bio text",
        "role": "USER",
        "followedAt": "2024-01-10T00:00:00.000Z",
        "isFollowing": false,
        "isFriend": true
      }
    ]
  }
}
```

**Error Response (if profile is USER):**
```json
{
  "message": "Users don't have followers. Use friends instead."
}
```

---

### 7. Get Following List
**GET** `/api/profile/:profileId/following`

**Description:** Get list of profiles that a specific profile is following

**URL Parameters:**
```
profileId: string (required) - UUID of the profile
```

**Query Parameters:**
```
page: string (optional, default: "1") - Page number
size: string (optional, default: "25") - Items per page
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET http://localhost:3000/api/profile/550e8400-e29b-41d4-a716-446655440000/following?page=1&size=25
```

**Response:**
```json
{
  "message": "Following retrieved successfully",
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 856,
    "pages": 35
  },
  "data": {
    "following": [
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "username": "seller1",
        "displayName": "Seller One",
        "avatarUrl": "https://cloudinary.com/avatar.jpg",
        "bio": "Shop bio",
        "role": "SELLER",
        "followedAt": "2024-01-05T00:00:00.000Z",
        "isFollowing": true,
        "isFriend": false
      }
    ]
  }
}
```

---

### 8. Block User
**POST** `/api/profile/:profileId/block`

**Description:** Block a user profile. This will automatically remove friendships and follow relationships.

**URL Parameters:**
```
profileId: string (required) - UUID of the profile to block
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
POST http://localhost:3000/api/profile/550e8400-e29b-41d4-a716-446655440000/block
```

**Response:**
```json
{
  "message": "User blocked successfully"
}
```

**Error Responses:**
- `400`: "You cannot block yourself"
- `400`: "User is already blocked"
- `404`: "Profile not found"

---

### 9. Unblock User
**DELETE** `/api/profile/:profileId/block`

**Description:** Unblock a previously blocked user

**URL Parameters:**
```
profileId: string (required) - UUID of the profile to unblock
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
DELETE http://localhost:3000/api/profile/550e8400-e29b-41d4-a716-446655440000/block
```

**Response:**
```json
{
  "message": "User unblocked successfully"
}
```

**Error Response:**
```json
{
  "message": "User is not blocked"
}
```

---

### 10. Get Blocked Users List
**GET** `/api/profile/blocked/list`

**Description:** Get list of all users blocked by the authenticated user

**Query Parameters:**
```
page: string (optional, default: "1") - Page number
size: string (optional, default: "25") - Items per page
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET http://localhost:3000/api/profile/blocked/list?page=1&size=25
```

**Response:**
```json
{
  "message": "Blocked users retrieved successfully",
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 5,
    "pages": 1
  },
  "data": {
    "blockedUsers": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "username": "blockeduser",
        "displayName": "Blocked User",
        "avatarUrl": "https://cloudinary.com/avatar.jpg",
        "role": "USER",
        "blockedAt": "2024-01-10T00:00:00.000Z"
      }
    ]
  }
}
```

---

## ⚙️ Profile Settings APIs

### 11. Update Profile Settings
**PATCH** `/api/profile/settings`

**Description:** Update profile settings (privacy, notifications, etc.)

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{}
```

**Note:** Currently no user-editable settings are available. The `isBlocking` field is admin-only and cannot be changed by users. More settings can be added in the future.

**Response:**
```json
{
  "message": "Profile settings updated successfully",
  "data": {
    "settings": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "updatedAt": "2024-01-15T10:30:00.000Z"
    },
    "note": "No user-editable settings available. isBlocking is admin-only."
  }
}
```

**Important Notes:**
- `isBlocking` is an **admin-only** field used to block accounts
- Users cannot modify `isBlocking` themselves
- If your account shows "Account is blocked by admin", contact support
- This endpoint is reserved for future user-editable settings (privacy, notifications, etc.)

---

## 📁 Category Preferences APIs

### 12. Select Category Preferences
**POST** `/api/profile/categories/preferences`

**Description:** Select category preferences for the user

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Body (JSON):**
```json
{
  "collectionIds": [1, 2, 3, 4, 5]
}
```

**Response:**
```json
{
  "message": "Preferences saved successfully",
  "data": {
    "preferences": [
      {
        "id": 1,
        "profileId": "550e8400-e29b-41d4-a716-446655440000",
        "collectionId": 1,
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    ]
  }
}
```

---

### 13. Get User Category Preferences
**GET** `/api/profile/categories/preferences`

**Description:** Get the category preferences selected by the authenticated user

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "message": "Preferences retrieved successfully",
  "data": {
    "collections": [
      {
        "id": 1,
        "title": "Fashion",
        "handle": "fashion",
        "imageUrl": "https://cloudinary.com/image.jpg",
        "description": "Fashion collection"
      }
    ],
    "totalSelected": 5
  }
}
```

---

### 14. Get All Categories
**GET** `/api/profile/categories`

**Description:** Get all available categories/collections that users can select from

**Headers:**
```
Authorization: <token>
```

**Response:**
```json
{
  "message": "Categories retrieved successfully",
  "data": {
    "categories": [
      {
        "id": 1,
        "title": "Fashion",
        "handle": "fashion",
        "imageUrl": "https://cloudinary.com/image.jpg",
        "description": "Fashion collection"
      }
    ],
    "total": 20
  }
}
```

---

## 📸 Posts by Profile APIs

### 15. Get Posts by Profile ID
**GET** `/api/posts/profile/:profileId`

**Description:** Get all posts created by a specific profile (for profile page "All Posts" section)

**URL Parameters:**
```
profileId: string (required) - UUID of the profile
```

**Query Parameters:**
```
page: string (optional, default: "1") - Page number
size: string (optional, default: "25") - Items per page
```

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```
GET http://localhost:3000/api/posts/profile/550e8400-e29b-41d4-a716-446655440000?page=1&size=25
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 42,
  "data": [
    {
      "id": "880e8400-e29b-41d4-a716-446655440003",
      "type": "FEED",
      "caption": "Check out my new style!",
      "mediaUrl": "https://cloudinary.com/post-image.jpg",
      "createdAt": "2024-01-15T10:30:00.000Z",
      "updatedAt": "2024-01-15T10:30:00.000Z",
      "author": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "displayName": "John Doe",
        "avatarUrl": "https://cloudinary.com/avatar.jpg",
        "role": "USER"
      },
      "counts": {
        "likes": 150,
        "comments": 25,
        "shares": 10
      },
      "likedByCurrentUser": true,
      "isMine": false,
      "isFollow": true,
      "repostedByCurrentUser": false
    }
  ]
}
```

**Error Responses:**
- `404`: "Profile not found"
- `403`: "Profile is not active"

---

## 📝 Notes

### Role-Based Stats
- **USER role:** Stats include `posts`, `friends`, `following`
- **SELLER/MEDIATOR roles:** Stats include `posts`, `followers`, `following`

### Pagination
All list endpoints support pagination with:
- `page`: Page number (default: 1)
- `size`: Items per page (default: 25, max: 25)

### Error Responses
All endpoints return errors in the following format:
```json
{
  "message": "Error message here"
}
```

Common HTTP status codes:
- `200`: Success
- `400`: Bad Request (validation errors, invalid input)
- `401`: Unauthorized (missing or invalid token)
- `403`: Forbidden (insufficient permissions, profile inactive)
- `404`: Not Found (resource doesn't exist)
- `409`: Conflict (duplicate resource)
- `500`: Internal Server Error

### Testing Tips
1. Always authenticate first using the auth/login endpoint
2. Use the returned JWT token in the Authorization header
3. For file uploads (avatar), use Postman's form-data with File type
4. Test pagination with different page and size values
5. Test error cases (invalid IDs, missing fields, etc.)

---

## 🔗 Related APIs

- **Friends APIs:** `/api/friends` - For friend requests and friendships
- **Follow APIs:** `/api/follows` - For following/unfollowing sellers/mediators
- **Posts APIs:** `/api/posts` - For creating and managing posts

---

**Last Updated:** 2024-01-15

