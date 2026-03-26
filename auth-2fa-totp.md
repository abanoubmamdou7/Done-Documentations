# Two-Factor Authentication (2FA) — Authenticator App API Documentation

This document covers the TOTP-based 2FA feature that allows users to secure their accounts with any authenticator app (Google Authenticator, Apple Passwords, Authy, Microsoft Authenticator, etc.).

## Table of Contents
1. [Overview](#overview)
2. [How It Works](#how-it-works)
3. [Authentication](#authentication)
4. [Modified Login Flow](#modified-login-flow)
5. [Endpoints](#endpoints)
   - [GET /2fa/status](#get-2fastatus)
   - [GET /2fa/setup](#get-2fasetup)
   - [POST /2fa/enable](#post-2faenable)
   - [POST /2fa/disable](#post-2fadisable)
   - [POST /2fa/verify](#post-2faverify)
6. [Error Responses](#error-responses)
7. [Best Practices](#best-practices)
8. [Postman Collection](#postman-collection)

---

## Overview

TOTP (Time-based One-Time Password) 2FA adds a second layer of security to user accounts. After enabling it, users must provide a 6-digit rotating code from their authenticator app every time they log in. The code refreshes every 30 seconds and works **offline** — no SMS or internet required on the user's phone.

**Base URL:** `/api/auth`

**Compatible Apps:**
- Google Authenticator (Android / iOS)
- Apple Passwords (built-in iOS 17+)
- Authy
- Microsoft Authenticator
- Any app that supports TOTP (RFC 6238)

---

## How It Works

```
Setup Flow:
  User (logged in) → GET /2fa/setup → receives QR code
  User scans QR in authenticator app
  User enters first 6-digit code → POST /2fa/enable
  Server verifies code → 2FA enabled
  Server returns 8 one-time backup codes (shown ONCE)

Login Flow (2FA enabled):
  User → POST /auth/login (email + password)
  Server verifies credentials → returns { requires2fa: true, tempToken }
  User enters 6-digit code from app → POST /2fa/verify (tempToken + totpCode)
  Server verifies code → returns full JWT token

Login Flow (2FA disabled):
  User → POST /auth/login (email + password)
  Server returns full JWT token immediately (no change)
```

---

## Authentication

Endpoints marked **Required** need a valid JWT token in the Authorization header.
The `/2fa/verify` endpoint is **public** — it uses a short-lived `tempToken` instead.

```
Authorization: <your_jwt_token>
```

---

## Modified Login Flow

When a user with 2FA enabled calls `POST /api/auth/login`, the response changes:

### Without 2FA (unchanged)
```json
{
  "message": "Login successful",
  "data": {
    "token": "eyJhbGci...",
    "profile": { ... },
    "user": { ... }
  }
}
```

### With 2FA enabled (new)
```json
{
  "message": "2FA required. Use the tempToken to complete login via /api/auth/2fa/verify",
  "data": {
    "requires2fa": true,
    "tempToken": "eyJhbGci..."
  }
}
```

The `tempToken` is a **short-lived JWT valid for 5 minutes**. The client must use it immediately with `POST /2fa/verify` to complete login.

**Frontend logic:**
```
if (response.data.requires2fa === true) {
  // Save tempToken temporarily
  // Show 2FA code input screen
  // On submit → call POST /api/auth/2fa/verify
} else {
  // Normal login — save token and proceed
}
```

---

## Endpoints

---

### GET /2fa/status

**Description:** Returns whether 2FA is currently enabled for the authenticated user and how many backup codes remain.

**Authentication:** Required

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```http
GET /api/auth/2fa/status
Authorization: Bearer <your-jwt-token>
```

**Response (200 OK):**
```json
{
  "data": {
    "totpEnabled": true,
    "backupCodesRemaining": 7
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `totpEnabled` | Boolean | Whether 2FA is currently active |
| `backupCodesRemaining` | Number | Number of unused backup codes (0–8) |

**Notes:**
- Use this on the security settings screen to show the current 2FA state.
- If `backupCodesRemaining` is 2 or fewer, prompt the user to disable and re-enable 2FA to generate fresh backup codes.

---

### GET /2fa/setup

**Description:** Generates a new TOTP secret and returns a QR code for the user to scan. **Does NOT enable 2FA** — the user must confirm with a valid code via `/2fa/enable`.

**Authentication:** Required

**Headers:**
```
Authorization: <token>
```

**Example Request:**
```http
GET /api/auth/2fa/setup
Authorization: Bearer <your-jwt-token>
```

**Response (200 OK):**
```json
{
  "message": "Scan the QR code with your authenticator app, then confirm with a code",
  "data": {
    "qrCode": "data:image/png;base64,iVBORw0KGgoAAAANS...",
    "secret": "JBSWY3DPEHPK3PXP"
  }
}
```

**Response Fields:**

| Field | Type | Description |
|-------|------|-------------|
| `qrCode` | String | Base64 PNG data URL — render as `<img src="...">` |
| `secret` | String | Base32 secret for manual entry (if camera unavailable) |

**Error Responses:**

| Status | Message | Cause |
|--------|---------|-------|
| 409 | `2FA is already enabled. Disable it first to reconfigure` | User already has 2FA on |

**Notes:**
- Render `qrCode` directly as an image: `<img src={data.qrCode} />`
- Show `secret` as a fallback for users who can't scan (they type it manually into the app)
- The secret is saved temporarily but 2FA is **not active** until `/2fa/enable` succeeds

---

### POST /2fa/enable

**Description:** Confirms the 2FA setup by verifying the first code from the authenticator app. Enables 2FA and returns 8 one-time backup codes.

**Authentication:** Required

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "totpCode": "123456"
}
```

**Request Body Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `totpCode` | String | Yes | 6-digit code currently shown in the authenticator app |

**Response (200 OK):**
```json
{
  "message": "2FA enabled successfully",
  "data": {
    "backupCodes": [
      "A1B2-C3D4",
      "E5F6-G7H8",
      "I9J0-K1L2",
      "M3N4-O5P6",
      "Q7R8-S9T0",
      "U1V2-W3X4",
      "Y5Z6-A7B8",
      "C9D0-E1F2"
    ],
    "warning": "Save these backup codes in a safe place. They will not be shown again."
  }
}
```

**Error Responses:**

| Status | Message | Cause |
|--------|---------|-------|
| 400 | `No 2FA setup in progress. Call /2fa/setup first` | Setup was not initiated |
| 400 | `Invalid authenticator code. Please try again` | Wrong or expired code |
| 409 | `2FA is already enabled` | Already configured |

**Notes:**
- Backup codes are shown **exactly once** — prompt the user to save them (copy, screenshot, or write down)
- Each backup code is in `XXXX-XXXX` format and can be used **once** if the authenticator app is unavailable
- After this call, all subsequent logins will require a TOTP code

---

### POST /2fa/disable

**Description:** Disables 2FA. Requires both the current password and a valid TOTP code as confirmation.

**Authentication:** Required

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
```json
{
  "password": "currentPassword123",
  "totpCode": "654321"
}
```

**Request Body Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `password` | String | Yes | User's current account password |
| `totpCode` | String | Yes | 6-digit code from the authenticator app |

**Response (200 OK):**
```json
{
  "message": "2FA disabled successfully"
}
```

**Error Responses:**

| Status | Message | Cause |
|--------|---------|-------|
| 400 | `2FA is not enabled` | Nothing to disable |
| 400 | `Invalid authenticator code` | Wrong TOTP code |
| 401 | `Incorrect password` | Password mismatch |

**Notes:**
- Both password and TOTP code are required to prevent unauthorized disabling
- After disabling, the stored secret and backup codes are deleted permanently
- The user can re-enable 2FA at any time via `/2fa/setup`

---

### POST /2fa/verify

**Description:** Completes login for accounts with 2FA enabled. Exchanges the `tempToken` (received from `/login`) plus a TOTP code or backup code for a full JWT token.

**Authentication:** None (uses `tempToken` instead)

**Headers:**
```
Content-Type: application/json
```

**Request Body (with authenticator code):**
```json
{
  "tempToken": "eyJhbGci...",
  "totpCode": "123456"
}
```

**Request Body (with backup code):**
```json
{
  "tempToken": "eyJhbGci...",
  "backupCode": "A1B2-C3D4"
}
```

**Request Body Fields:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tempToken` | String | Yes | The `tempToken` returned by `POST /login` |
| `totpCode` | String | One of | 6-digit code from the authenticator app |
| `backupCode` | String | One of | One-time backup code (format: `XXXX-XXXX`) |

At least one of `totpCode` or `backupCode` must be provided.

**Response (200 OK):**
```json
{
  "message": "2FA verified. Login successful",
  "data": {
    "token": "eyJhbGci...",
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

**Response Headers (when using backup code with ≤ 2 remaining):**
```
X-Backup-Codes-Remaining: 1
```

**Error Responses:**

| Status | Message | Cause |
|--------|---------|-------|
| 400 | `Provide either totpCode or backupCode` | Neither field sent |
| 400 | `Invalid token type` | Token is not a 2FA pending token |
| 400 | `Invalid authenticator code` | Wrong TOTP code |
| 400 | `Invalid backup code` | Wrong or already-used backup code |
| 401 | `Invalid or expired session. Please log in again` | tempToken expired (> 5 min) or invalid |

**Notes:**
- The `tempToken` expires after **5 minutes** — if it expires the user must log in again
- Backup codes are consumed on use (each code works once only)
- If `X-Backup-Codes-Remaining` header is present and ≤ 2, warn the user to generate new codes
- The FCM token passed during login is automatically registered on successful 2FA verification

---

## Error Responses

Common error format across all 2FA endpoints:

```json
{
  "message": "Error description here",
  "status_code": 400,
  "error": {}
}
```

| Status | Meaning |
|--------|---------|
| 400 | Bad request — invalid input or wrong code |
| 401 | Unauthorized — expired/invalid token or wrong password |
| 404 | User not found |
| 409 | Conflict — 2FA already in the requested state |

---

## Best Practices

### Enable 2FA Flow
```
1. User goes to Settings > Security > Enable 2FA
2. Call GET /api/auth/2fa/setup
3. Display QR code as <img src={qrCode} />
4. Show manual secret as fallback text
5. User scans QR in their authenticator app
6. User enters the 6-digit code shown in the app
7. Call POST /api/auth/2fa/enable with { totpCode }
8. On success → show backup codes with a "Copy All" button
9. Force user to acknowledge before dismissing (checkbox: "I've saved my codes")
10. 2FA is now active
```

### Login with 2FA Flow
```
1. User enters email/password → POST /api/auth/login
2. Check response:
   - If requires2fa === true → save tempToken, show TOTP input screen
   - If no requires2fa → normal login, save token
3. User opens authenticator app, reads 6-digit code
4. POST /api/auth/2fa/verify with { tempToken, totpCode }
5. On success → save token, proceed to app
6. On error "Invalid authenticator code" → let user retry (code refreshes every 30s)
7. On error "Invalid or expired session" → redirect back to login
```

### Backup Code Flow
```
1. User lost their phone / can't access authenticator app
2. On 2FA screen, show "Use a backup code" option
3. User enters one of their saved XXXX-XXXX codes
4. POST /api/auth/2fa/verify with { tempToken, backupCode }
5. Check X-Backup-Codes-Remaining response header
6. If ≤ 2 remaining → prompt user to disable and re-enable 2FA to get fresh codes
```

### Disable 2FA Flow
```
1. User goes to Settings > Security > Disable 2FA
2. Ask for current password + current TOTP code
3. POST /api/auth/2fa/disable with { password, totpCode }
4. On success → update UI to show 2FA as disabled
```

---

## Frontend UX Recommendations

- **Setup screen:** Show QR code prominently. Show manual secret in a collapsed section labeled "Can't scan? Enter manually".
- **Backup codes screen:** Display codes in a grid. Provide "Copy All" and "Download as .txt" buttons. Show a warning that these will not be displayed again.
- **Login screen:** After password verification, show a "Enter authenticator code" screen with a numeric keypad. Add a small "Use backup code" link below.
- **Countdown hint:** Authenticator codes are valid for 30 seconds. The API allows ±30 seconds drift, so codes remain valid for up to 60 seconds total.
- **Low backup codes:** If the status endpoint returns `backupCodesRemaining ≤ 2`, show a warning banner in the security settings.

---

## Postman Collection

### Environment Variables
- `base_url`: `http://localhost:3000`
- `auth_token`: Valid JWT from login
- `temp_token`: Short-lived token from login (when requires2fa is true)

---

### 1. Get 2FA Status
```
GET {{base_url}}/api/auth/2fa/status
Authorization: {{auth_token}}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Has totpEnabled", () => {
    const data = pm.response.json().data;
    pm.expect(data).to.have.property("totpEnabled");
    pm.expect(data).to.have.property("backupCodesRemaining");
});
```

---

### 2. Setup 2FA (Get QR Code)
```
GET {{base_url}}/api/auth/2fa/setup
Authorization: {{auth_token}}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Has qrCode and secret", () => {
    const data = pm.response.json().data;
    pm.expect(data).to.have.property("qrCode");
    pm.expect(data).to.have.property("secret");
    pm.expect(data.qrCode).to.include("data:image/png;base64");
});
```

---

### 3. Enable 2FA
```
POST {{base_url}}/api/auth/2fa/enable
Authorization: {{auth_token}}
Content-Type: application/json

{
  "totpCode": "123456"
}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Has backup codes", () => {
    const data = pm.response.json().data;
    pm.expect(data.backupCodes).to.be.an("array").with.lengthOf(8);
    pm.environment.set("backup_code", data.backupCodes[0]);
});
```

---

### 4. Login (2FA account)
```
POST {{base_url}}/api/auth/login
Content-Type: application/json

{
  "emailOrUsername": "user@example.com",
  "password": "password123"
}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
const data = pm.response.json().data;
if (data.requires2fa) {
    pm.environment.set("temp_token", data.tempToken);
    pm.test("Has tempToken", () => pm.expect(data.tempToken).to.be.a("string"));
} else {
    pm.environment.set("auth_token", data.token);
    pm.test("Has token", () => pm.expect(data.token).to.be.a("string"));
}
```

---

### 5. Verify 2FA (complete login)
```
POST {{base_url}}/api/auth/2fa/verify
Content-Type: application/json

{
  "tempToken": "{{temp_token}}",
  "totpCode": "123456"
}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Has token", () => {
    const data = pm.response.json().data;
    pm.expect(data.token).to.be.a("string");
    pm.environment.set("auth_token", data.token);
});
```

---

### 6. Verify 2FA with Backup Code
```
POST {{base_url}}/api/auth/2fa/verify
Content-Type: application/json

{
  "tempToken": "{{temp_token}}",
  "backupCode": "{{backup_code}}"
}
```

---

### 7. Disable 2FA
```
POST {{base_url}}/api/auth/2fa/disable
Authorization: {{auth_token}}
Content-Type: application/json

{
  "password": "password123",
  "totpCode": "123456"
}
```

**Test Script:**
```javascript
pm.test("Status 200", () => pm.response.to.have.status(200));
pm.test("Disabled successfully", () => {
    pm.expect(pm.response.json().message).to.include("disabled");
});
```

---

## Notes

- **Clock sync:** TOTP codes are time-based. The server allows ±30 seconds drift. If users report frequent failures, check that their phone clock is synchronized.
- **Secret security:** The TOTP secret stored in the database is as sensitive as a password. It is stored in plaintext in the current implementation — treat DB access accordingly.
- **tempToken expiry:** The 2FA pending token expires in 5 minutes. If the user takes too long, they must log in again. Show a countdown timer in the UI.
- **Backup codes:** Each backup code works exactly once. After use it is deleted from the database. Warn users to treat them like passwords.
- **Re-setup:** To get new backup codes, the user must disable 2FA and re-enable it (new QR scan required).
- **All roles supported:** 2FA works for USER, SELLER, and MEDIATOR roles.

---

## Related Documentation

- [Auth API (Login, Logout, FCM)](./auth.md)
- [Profile API](./profiles.md)
