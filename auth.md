# FCM Token Integration - Login, Logout & Settings API Documentation

This document provides API documentation for the integrated Firebase Cloud Messaging (FCM) token management in login, logout, and profile settings endpoints.

## 📋 Table of Contents
1. [Overview](#overview)
2. [Login with FCM Token](#login-with-fcm-token)
3. [Logout with FCM Token](#logout-with-fcm-token)
4. [Change Password](#change-password)
5. [Profile Settings - Notification Toggle](#profile-settings---notification-toggle)
6. [Best Practices](#best-practices)

---

## 🔐 Authentication

All protected endpoints require authentication via JWT token in the Authorization header.

**Header:**
```
Authorization: <your_jwt_token>
```

**Base URLs:**
- Auth: `/api/auth`
- Profile: `/api/profile`

---

## Overview

FCM token management is now **automatically integrated** into:
- **Login** - Automatically registers FCM token when provided
- **Logout** - Automatically unregisters FCM token when provided
- **Profile Settings** - Toggle notifications on/off

This eliminates the need for separate API calls to register/unregister tokens.

---

## 1. Login with FCM Token

**POST** `/api/auth/login`

**Description:** Login endpoint that automatically registers FCM token if provided. Works for all user roles (USER, SELLER, MEDIATOR).

**Headers:**
```
Content-Type: application/json
```

**Request Body:**
```json
{
  "emailOrUsername": "user@example.com",
  "password": "password123",
  "fcmToken": "fcm-token-from-firebase-sdk",
  "deviceId": "device-unique-id",
  "platform": "ios"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| emailOrUsername | String | Yes | Email address or username |
| password | String | Yes | User password |
| fcmToken | String | No | FCM token from Firebase SDK (auto-registered if provided) |
| deviceId | String | No | Unique device identifier |
| platform | String | No | Platform: `"ios"`, `"android"`, or `"web"` |

**Response (200 OK):**
```json
{
  "message": "Login successful",
  "data": {
    "token": "jwt-token-here",
    "profile": {
      "id": "uuid",
      "displayName": "John Doe",
      "role": "USER",
      "phone": "1234567890",
      "avatarUrl": "https://...",
      "customizedPreference": true
    },
    "user": {
      "id": "uuid",
      "email": "user@example.com",
      "username": "johndoe"
    }
  }
}
```

**Response (401 Unauthorized):**
```json
{
  "message": "Invalid email/username or password",
  "status_code": 401,
  "error": {}
}
```

**Response (403 Forbidden):**
```json
{
  "message": "Account is inactive or suspended",
  "status_code": 403,
  "error": {}
}
```

**Notes:**
- FCM token registration is **non-blocking** - login succeeds even if token registration fails
- If the same FCM token already exists, it will be updated with the current user's profile ID
- Token registration happens automatically in the background
- You can still use the separate `/api/notifications/register-token` endpoint if needed

**Example Request (with FCM token):**
```json
POST /api/auth/login
Content-Type: application/json

{
  "emailOrUsername": "johndoe",
  "password": "SecurePass123!",
  "fcmToken": "cXyZ123abc...",
  "deviceId": "iPhone-14-Pro-ABC123",
  "platform": "ios"
}
```

**Example Request (without FCM token - still works):**
```json
POST /api/auth/login
Content-Type: application/json

{
  "emailOrUsername": "johndoe",
  "password": "SecurePass123!"
}
```

---

## 2. Logout with FCM Token

**POST** `/api/auth/logout`

**Description:** Logout endpoint that automatically unregisters FCM tokens from the database. If `fcmToken` is provided, only that specific token is unregistered. If not provided, ALL tokens for the user are unregistered (logout from all devices).

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body (Optional):**
```json
{
  "fcmToken": "fcm-token-from-firebase-sdk"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| fcmToken | String | No | FCM token to unregister. If not provided, ALL tokens for the user are unregistered |

**Response (200 OK):**
```json
{
  "message": "Logout successful",
  "data": {
    "loggedOut": true
  }
}
```

**Error Responses:**
- `401` - Unauthorized (missing or invalid token)

**Notes:**
- FCM token unregistration is **non-blocking** - logout succeeds even if token unregistration fails
- **If `fcmToken` is provided:** Only that specific token is unregistered (single device logout)
- **If `fcmToken` is NOT provided:** ALL FCM tokens for the user are fetched from database and unregistered (logout from all devices)
- Tokens are automatically fetched from the database - no need to send token if you want to logout from all devices
- You can still use the separate `/api/notifications/unregister-token` endpoint if needed

**Example Request:**
```json
POST /api/auth/logout
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "fcmToken": "cXyZ123abc..."
}
```

**Example Request (without FCM token - unregisters ALL tokens):**
```json
POST /api/auth/logout
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{}
```

**Behavior:**
- **With `fcmToken`:** Unregisters only that specific device token
- **Without `fcmToken`:** Fetches all tokens from database and unregisters all of them (logout from all devices)

---

## 4. Change Password

**POST** `/api/auth/change-password`

**Description:** Allows authenticated users to change their password. Requires the current password for verification, and both new password and confirmation must match.

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "currentPassword": "oldPassword123",
  "newPassword": "newSecurePassword456",
  "confirmNewPassword": "newSecurePassword456"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| currentPassword | String | Yes | User's current password (for verification) |
| newPassword | String | Yes | New password (minimum 8 characters) |
| confirmNewPassword | String | Yes | Must match newPassword exactly |

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Password changed successfully"
}
```

**Error Responses:**

**401 Unauthorized - Incorrect Current Password:**
```json
{
  "message": "Current password is incorrect",
  "status_code": 401,
  "error": {}
}
```

**400 Bad Request - New Password Same as Current:**
```json
{
  "message": "New password must be different from current password",
  "status_code": 400,
  "error": {}
}
```

**400 Bad Request - Validation Error:**
```json
{
  "message": "New password and confirm password must match",
  "status_code": 400,
  "error": {}
}
```

**401 Unauthorized - Missing Token:**
```json
{
  "message": "Authorization is required",
  "status_code": 401,
  "error": {}
}
```

**Notes:**
- ✅ Requires authentication (valid JWT token)
- ✅ Works for all user roles (USER, SELLER, MEDIATOR)
- ✅ Current password is verified before allowing change
- ✅ New password must be different from current password
- ✅ New password and confirm password must match exactly
- ✅ Minimum password length: 8 characters
- ✅ Password is securely hashed using bcrypt before storage

**Example Request:**
```json
POST /api/auth/change-password
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "currentPassword": "MyOldPassword123!",
  "newPassword": "MyNewSecurePassword456!",
  "confirmNewPassword": "MyNewSecurePassword456!"
}
```

**Example cURL:**
```bash
curl -X POST "http://localhost:3000/api/auth/change-password" \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "currentPassword": "oldPassword123",
    "newPassword": "newPassword456",
    "confirmNewPassword": "newPassword456"
  }'
```

**Security Features:**
- 🔒 Current password verification prevents unauthorized changes
- 🔒 Password hashing using bcrypt with configurable salt rounds
- 🔒 Validation ensures new password is different from current
- 🔒 Confirmation field prevents typos in new password

---

## 5. Profile Settings - Notification Toggle

**PATCH** `/api/profile/settings`

**Description:** Update profile settings including notification preferences. Can enable/disable push notifications by managing FCM tokens.

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body (Disable Notifications):**
```json
{
  "notificationsEnabled": false
}
```

**Request Body (Enable Notifications):**
```json
{
  "notificationsEnabled": true,
  "fcmToken": "fcm-token-from-firebase-sdk"
}
```

**Request Body Fields:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| notificationsEnabled | Boolean | No | Toggle notification preferences |
| fcmToken | String | No | FCM token to register when enabling notifications (required if `notificationsEnabled: true`) |

**Response (200 OK):**
```json
{
  "message": "Profile settings updated successfully",
  "data": {
    "settings": {
      "id": "uuid",
      "notificationsEnabled": false,
      "updatedAt": "2024-01-01T12:00:00.000Z"
    }
  }
}
```

**Error Responses:**
- `401` - Unauthorized
- `404` - Profile not found

**Behavior:**

### When `notificationsEnabled: false`
- **Action:** Unregisters **all** FCM tokens for the authenticated user
- **Result:** User will not receive push notifications on any device
- **Use Case:** User wants to disable all notifications

### When `notificationsEnabled: true` + `fcmToken` provided
- **Action:** Registers the provided FCM token
- **Result:** User will receive push notifications on that device
- **Use Case:** User wants to enable notifications for a specific device

**Example Request (Disable Notifications):**
```json
PATCH /api/profile/settings
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "notificationsEnabled": false
}
```

**Example Request (Enable Notifications):**
```json
PATCH /api/profile/settings
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "notificationsEnabled": true,
  "fcmToken": "cXyZ123abc..."
}
```

---

## Best Practices

### 1. Login Flow
```
1. User opens app
2. Firebase SDK generates/retrieves FCM token
3. User enters credentials and logs in
4. Include fcmToken in login request
5. Token automatically registered
6. User receives push notifications
```

### 2. Logout Flow
```
1. User taps logout
2. Get current FCM token from Firebase SDK
3. Include fcmToken in logout request
4. Token automatically unregistered
5. No more notifications sent to that device
```

### 3. Change Password Flow
```
1. User goes to Settings > Change Password
2. User enters current password
3. User enters new password
4. User confirms new password
5. Frontend validates passwords match
6. Send POST /api/auth/change-password
7. Backend verifies current password
8. Backend checks new password is different
9. Backend hashes and updates password
10. User receives success message
11. User can now login with new password
```

### 4. Notification Toggle Flow
```
Scenario A: User wants to disable notifications
1. User goes to Settings
2. Toggles "Notifications" OFF
3. Send: { "notificationsEnabled": false }
4. All tokens unregistered
5. No more notifications

Scenario B: User wants to enable notifications
1. User goes to Settings
2. Get FCM token from Firebase SDK
3. Toggles "Notifications" ON
4. Send: { "notificationsEnabled": true, "fcmToken": "..." }
5. Token registered
6. User receives notifications
```

### 5. Multi-Device Support
- Each device has its own FCM token
- Register token on login for each device
- Unregister token on logout for that specific device
- User can have multiple devices registered simultaneously

### 6. Token Lifecycle
- **Token Changes:** FCM tokens can change (app reinstall, OS update)
- **Handle Changes:** Re-register token when it changes
- **Automatic Cleanup:** Invalid tokens are automatically removed by Firebase

### 7. Error Handling
- All FCM operations are **non-blocking**
- Login/logout succeed even if token operations fail
- Log errors for debugging but don't block user flow
- Frontend can handle token registration separately if needed

---

## Comparison: Integrated vs Separate Endpoints

### Before (Separate Endpoints)
```
1. POST /api/auth/login
2. POST /api/notifications/register-token (separate call)
```

### After (Integrated)
```
1. POST /api/auth/login (with fcmToken in body)
   → Token automatically registered
```

### Benefits of Integration
- ✅ **Fewer API calls** - One request instead of two
- ✅ **Better UX** - Automatic token management
- ✅ **Simpler code** - No need to handle separate token registration
- ✅ **More reliable** - Token registered immediately on login

---

## Example Postman Collection

### Login with FCM Token
```
POST {{baseUrl}}/api/auth/login
Content-Type: application/json

{
  "emailOrUsername": "user@example.com",
  "password": "password123",
  "fcmToken": "{{fcmToken}}",
  "deviceId": "device-123",
  "platform": "ios"
}
```

### Logout with FCM Token
```
POST {{baseUrl}}/api/auth/logout
Authorization: {{token}}
Content-Type: application/json

{
  "fcmToken": "{{fcmToken}}"
}
```

### Disable Notifications
```
PATCH {{baseUrl}}/api/profile/settings
Authorization: {{token}}
Content-Type: application/json

{
  "notificationsEnabled": false
}
```

### Enable Notifications
```
PATCH {{baseUrl}}/api/profile/settings
Authorization: {{token}}
Content-Type: application/json

{
  "notificationsEnabled": true,
  "fcmToken": "{{fcmToken}}"
}
```

### Change Password
```
POST {{baseUrl}}/api/auth/change-password
Authorization: {{token}}
Content-Type: application/json

{
  "currentPassword": "oldPassword123",
  "newPassword": "newPassword456",
  "confirmNewPassword": "newPassword456"
}
```

---

## Migration Guide

If you're currently using separate token registration endpoints:

### Old Approach
```javascript
// 1. Login
const loginResponse = await fetch('/api/auth/login', {
  method: 'POST',
  body: JSON.stringify({ emailOrUsername, password })
});

// 2. Register token separately
await fetch('/api/notifications/register-token', {
  method: 'POST',
  headers: { Authorization: `Bearer ${token}` },
  body: JSON.stringify({ fcmToken, deviceId, platform })
});
```

### New Approach
```javascript
// 1. Login with token (automatic registration)
const loginResponse = await fetch('/api/auth/login', {
  method: 'POST',
  body: JSON.stringify({
    emailOrUsername,
    password,
    fcmToken,      // Add this
    deviceId,      // Add this
    platform       // Add this
  })
});
```

---

## Notes

- **Backward Compatible:** All FCM token fields are optional. Existing code without tokens will continue to work.
- **Non-Blocking:** Token operations don't block login/logout. If token registration fails, login still succeeds.
- **Error Handling:** Token registration errors are logged but don't affect the main operation.
- **Multiple Devices:** Users can have multiple devices registered. Each device needs its own token.
- **Token Updates:** If the same token is registered again, it updates the existing record (useful for account switching).

---

## Related Documentation

- [Notifications API Documentation](../notifications/POSTMAN_API_DOCUMENTATION.md)
- [Profile API Documentation](../profiles/POSTMAN_API_DOCUMENTATION.md)
- [Auth API Documentation](./POSTMAN_API_DOCUMENTATION.md) (if exists)

---

## Support

For issues or questions:
1. Check error logs for token registration failures
2. Verify FCM token is valid and not expired
3. Ensure Firebase is properly configured
4. Test with separate `/api/notifications/register-token` endpoint if needed

