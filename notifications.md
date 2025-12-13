# Notifications API Documentation for Postman

This document provides comprehensive API documentation for the Notifications system with Firebase Cloud Messaging (FCM) integration.

## 📋 Table of Contents
1. [Authentication](#authentication)
2. [Notification Types](#-notification-types)
3. [Get Notifications](#1-get-notifications)
4. [Get Unread Count](#2-get-unread-count)
5. [Mark Notification as Read](#3-mark-notification-as-read)
6. [Mark All Notifications as Read](#4-mark-all-notifications-as-read)
7. [Delete Notification](#5-delete-notification)
8. [Register FCM Token](#6-register-fcm-token)
9. [Unregister FCM Token](#7-unregister-fcm-token)
10. [Notification Data Structure](#notification-data-structure)
11. [Integration with Other Features](#integration-with-other-features)
12. [Best Practices](#best-practices)

---

## 🔐 Authentication

All endpoints require authentication via JWT token in the Authorization header.

**Header:**
```
Authorization: <your_jwt_token>
```

**Note:** Get your token from the auth/login endpoint first.

**Base URL:** `http://localhost:YOUR_PORT/api/notifications` (or your server URL)

---

## 📱 Notification Types

The system supports the following notification types:

- `FRIEND_REQUEST` - Someone sent you a friend request
- `FRIEND_REQUEST_ACCEPTED` - Someone accepted your friend request
- `POST_LIKE` - Someone liked your post
- `COMMENT_LIKE` - Someone liked your comment
- `COMMENT` - Someone commented on your post
- `COMMENT_REPLY` - Someone replied to your comment
- `REPOST` - Someone reposted your post
- `MENTION` - Someone mentioned you (@username)
- `STORY_VIEW` - Someone viewed your story (optional, limited)
- `STORY_LIKE` - Someone liked your story

---

## 1. Get Notifications

**GET** `/api/notifications`

**Description:** Get paginated list of notifications for the authenticated user.

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| type | String | No | Filter by notification type. Valid values: `FRIEND_REQUEST`, `FRIEND_REQUEST_ACCEPTED`, `POST_LIKE`, `COMMENT_LIKE`, `COMMENT`, `COMMENT_REPLY`, `REPOST`, `MENTION`, `STORY_VIEW`, `STORY_LIKE` |
| isRead | String | No | Filter by read status: `"true"` or `"false"` |
| page | String | No | Page number (default: 1) |
| size | String | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/notifications?type=POST_LIKE&isRead=false&page=1&size=25
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": "1",
  "size": "25",
  "total": 50,
  "unreadCount": 15,
  "data": [
    {
      "id": "uuid",
      "profileId": "uuid",
      "type": "POST_LIKE",
      "title": "New Like",
      "body": "John Doe liked your post",
      "data": {
        "actorId": "uuid",
        "postId": "uuid",
        "type": "POST_LIKE"
      },
      "isRead": false,
      "readAt": null,
      "createdAt": "2024-01-01T12:00:00.000Z"
    }
  ]
}
```

**Error Responses:**
- `401` - Unauthorized

---

## 2. Get Unread Count

**GET** `/api/notifications/unread-count`

**Description:** Get the count of unread notifications.

**Headers:**
```
Authorization: <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "unreadCount": 15
  }
}
```

**Error Responses:**
- `401` - Unauthorized

---

## 3. Mark Notification as Read

**PATCH** `/api/notifications/:id/read`

**Description:** Mark a specific notification as read.

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | String | Yes | Notification ID (UUID) |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Notification marked as read",
  "data": {
    "id": "uuid",
    "profileId": "uuid",
    "type": "POST_LIKE",
    "title": "New Like",
    "body": "John Doe liked your post",
    "data": {
      "actorId": "uuid",
      "postId": "uuid",
      "type": "POST_LIKE"
    },
    "isRead": true,
    "readAt": "2024-01-01T12:00:00.000Z",
    "createdAt": "2024-01-01T11:00:00.000Z"
  }
}
```

**Error Responses:**
- `400` - Invalid notification ID format
- `401` - Unauthorized
- `403` - You are not authorized to update this notification
- `404` - Notification not found

---

## 4. Mark All Notifications as Read

**PATCH** `/api/notifications/read-all`

**Description:** Mark all notifications as read for the authenticated user.

**Headers:**
```
Authorization: <token>
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "All notifications marked as read",
  "data": {
    "updatedCount": 15
  }
}
```

**Error Responses:**
- `401` - Unauthorized

---

## 5. Delete Notification

**DELETE** `/api/notifications/:id`

**Description:** Delete a specific notification.

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| id | String | Yes | Notification ID (UUID) |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Notification deleted successfully"
}
```

**Error Responses:**
- `400` - Invalid notification ID format
- `401` - Unauthorized
- `403` - You are not authorized to delete this notification
- `404` - Notification not found

---

## 6. Register FCM Token

**POST** `/api/notifications/register-token`

**Description:** Register a Firebase Cloud Messaging (FCM) token for push notifications.

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "fcmToken": "fcm-token-from-firebase",
  "deviceId": "device-unique-id",
  "platform": "ios"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| fcmToken | String | Yes | FCM token from Firebase SDK |
| deviceId | String | No | Unique device identifier |
| platform | String | No | Platform: `"ios"`, `"android"`, or `"web"` |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "FCM token registered successfully",
  "data": {
    "id": "1234567890",
    "platform": "ios",
    "createdAt": "2024-01-01T12:00:00.000Z"
  }
}
```

**Error Responses:**
- `400` - FCM token is required
- `401` - Unauthorized

**Notes:**
- The same FCM token can be registered for multiple users (if device is shared)
- If the token already exists, it will be updated with the new profile ID
- Call this endpoint whenever the FCM token changes (e.g., app reinstall)

---

## 7. Unregister FCM Token

**DELETE** `/api/notifications/unregister-token`

**Description:** Unregister a Firebase Cloud Messaging (FCM) token.

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "fcmToken": "fcm-token-from-firebase"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| fcmToken | String | Yes | FCM token to unregister |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "FCM token unregistered successfully"
}
```

**Error Responses:**
- `400` - FCM token is required
- `401` - Unauthorized
- `404` - FCM token not found

**Notes:**
- Call this endpoint when the user logs out or uninstalls the app
- Only tokens belonging to the authenticated user can be unregistered

---

## Notification Data Structure

Each notification includes a `data` field with additional context:

### Friend Request
```json
{
  "actorId": "uuid",
  "friendRequestId": "uuid",
  "type": "FRIEND_REQUEST"
}
```

### Friend Request Accepted
```json
{
  "actorId": "uuid",
  "type": "FRIEND_REQUEST_ACCEPTED"
}
```

### Post Like
```json
{
  "actorId": "uuid",
  "postId": "uuid",
  "type": "POST_LIKE"
}
```

### Comment Like
```json
{
  "actorId": "uuid",
  "commentId": "uuid",
  "type": "COMMENT_LIKE"
}
```

### Comment
```json
{
  "actorId": "uuid",
  "commentId": "uuid",
  "postId": "uuid",
  "type": "COMMENT"
}
```

### Comment Reply
```json
{
  "actorId": "uuid",
  "commentId": "uuid",
  "postId": "uuid",
  "type": "COMMENT_REPLY"
}
```

### Repost
```json
{
  "actorId": "uuid",
  "postId": "uuid",
  "type": "REPOST"
}
```

### Mention
```json
{
  "actorId": "uuid",
  "postId": "uuid",
  "commentId": "uuid",
  "type": "MENTION"
}
```

### Story Like
```json
{
  "actorId": "uuid",
  "storyId": "string",
  "type": "STORY_LIKE"
}
```

---

## Integration with Other Features

Notifications are automatically sent when:

1. **Friend Requests:**
   - When someone sends a friend request → `FRIEND_REQUEST`
   - When someone accepts your friend request → `FRIEND_REQUEST_ACCEPTED`

2. **Likes:**
   - When someone likes your post → `POST_LIKE`
   - When someone likes your comment → `COMMENT_LIKE`
   - When someone likes your story → `STORY_LIKE`

3. **Comments:**
   - When someone comments on your post → `COMMENT`
   - When someone replies to your comment → `COMMENT_REPLY`

4. **Reposts:**
   - When someone reposts your post → `REPOST`

5. **Mentions:**
   - When someone mentions you in a post caption → `MENTION`
   - When someone mentions you in a comment → `MENTION`

**Note:** Notifications are not sent to yourself (e.g., liking your own post won't trigger a notification).

---

## Example Postman Collection

### Get Notifications
```
GET {{baseUrl}}/api/notifications?page=1&size=25
Authorization: {{token}}
```

### Get Unread Count
```
GET {{baseUrl}}/api/notifications/unread-count
Authorization: {{token}}
```

### Mark Notification as Read
```
PATCH {{baseUrl}}/api/notifications/{{notificationId}}/read
Authorization: {{token}}
```

### Mark All as Read
```
PATCH {{baseUrl}}/api/notifications/read-all
Authorization: {{token}}
```

### Register FCM Token
```
POST {{baseUrl}}/api/notifications/register-token
Authorization: {{token}}
Content-Type: application/json

{
  "fcmToken": "your-fcm-token",
  "deviceId": "device-123",
  "platform": "ios"
}
```

### Unregister FCM Token
```
DELETE {{baseUrl}}/api/notifications/unregister-token
Authorization: {{token}}
Content-Type: application/json

{
  "fcmToken": "your-fcm-token"
}
```

---

## Best Practices

1. **FCM Token Management:**
   - Register the FCM token after user login
   - Update the token if it changes (Firebase SDK handles this)
   - Unregister the token on logout

2. **Notification Polling:**
   - Use the unread count endpoint to show a badge
   - Poll for new notifications periodically (e.g., every 30 seconds)
   - Use push notifications for real-time updates

3. **Marking as Read:**
   - Mark notifications as read when the user views them
   - Provide a "Mark all as read" option for better UX

4. **Filtering:**
   - Use the `type` filter to show specific notification categories
   - Use the `isRead` filter to separate read/unread notifications

5. **Performance:**
   - Use pagination for large notification lists
   - Cache unread count to reduce API calls
   - Delete old notifications periodically (optional cleanup)

---

## Error Handling

All endpoints follow the standard error response format:

```json
{
  "message": "Error message",
  "status_code": 400,
  "error": {}
}
```

Common error codes:
- `400` - Bad Request (invalid parameters)
- `401` - Unauthorized (missing or invalid token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource doesn't exist)

---

## Firebase Setup

To enable push notifications:

1. **Firebase Project:**
   - Create a Firebase project at https://console.firebase.google.com
   - Enable Cloud Messaging (FCM)
   - Download service account key (already configured in `services/firebaseKey.js`)

2. **Client Setup:**
   - Install Firebase SDK in your mobile/web app
   - Request notification permissions
   - Get FCM token and register it via `/api/notifications/register-token`

3. **Testing:**
   - Use Firebase Console to send test notifications
   - Verify tokens are registered correctly
   - Test notification delivery

---

## Notes

- Notifications are stored in the database for history
- Push notifications are sent asynchronously (non-blocking)
- Invalid FCM tokens are automatically cleaned up
- Story view notifications are limited (only first 10 views) to prevent spam
- Mentions are detected using `@username` format in captions and comments

