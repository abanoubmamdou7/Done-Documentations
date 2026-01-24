# Shared Cart (Group Shopping Session) API Documentation

## Overview

Complete API documentation for the Shared Cart feature in DoneBuddy. This module enables multiple users in a chat conversation to shop together with a shared cart. When the shopping session ends, all items are automatically copied to each participant's personal cart, allowing them to checkout individually.

**Base URL:** `/api/shopping-sessions`

**Architecture:**
- **Shopping Session**: Group shopping session tied to a conversation
- **Shared Cart**: Items stored separately from personal carts during active session
- **Session End**: Automatic copy of items to personal carts
- **Real-time Updates**: Socket.io events for live cart synchronization
- **Like Reactions**: LIKE-only reactions on shared cart items

---

## Table of Contents

1. [Authentication](#authentication)
2. [Session Lifecycle](#session-lifecycle)
   - [Start Shopping Session](#start-shopping-session)
   - [Join Shopping Session](#join-shopping-session)
   - [End Shopping Session](#end-shopping-session)
3. [Shared Cart Management](#shared-cart-management)
   - [Get Session Cart](#get-session-cart)
   - [Add Item to Session Cart](#add-item-to-session-cart)
   - [Update Session Cart Item](#update-session-cart-item)
   - [Remove Session Cart Item](#remove-session-cart-item)
4. [Item Reactions](#item-reactions)
   - [Toggle Item Like](#toggle-item-like)
5. [Real-time Events (Socket.io)](#real-time-events-socketio)
6. [Flow Diagram](#flow-diagram)
7. [Error Responses](#error-responses)
8. [Important Notes](#important-notes)

---

## Authentication

All shopping session endpoints require authentication.

**Header:**
```
Authorization: Bearer <your-jwt-token>
```

**User Roles:** `USER`, `SELLER`, `MEDIATOR`

---

## Session Lifecycle

### Start Shopping Session

**Description:** Start a new shopping session for a conversation. All conversation participants are automatically added to the session.

**Endpoint:** `POST /api/shopping-sessions/start`

**Authentication:** Required

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `conversationId` | string/number | Required | Conversation ID where session will be created |
| `sellerId` | string/number | Required | Seller ID (all items must be from this seller) |

**Example Request:**
```http
POST /api/shopping-sessions/start
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "conversationId": "123",
  "sellerId": "1"
}
```

**Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Shopping session started successfully",
  "data": {
    "session": {
      "id": "1",
      "conversationId": "123",
      "sellerId": "1",
      "hostId": "550e8400-e29b-41d4-a716-446655440000",
      "status": "ACTIVE",
      "startedAt": "2026-01-24T14:00:00.000Z",
      "participants": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "displayName": "John Doe",
          "avatarUrl": "https://example.com/avatar.jpg",
          "joinedAt": "2026-01-24T14:00:00.000Z"
        },
        {
          "id": "660e8400-e29b-41d4-a716-446655440001",
          "displayName": "Jane Smith",
          "avatarUrl": "https://example.com/avatar2.jpg",
          "joinedAt": "2026-01-24T14:00:00.000Z"
        }
      ],
      "items": []
    }
  }
}
```

**Behavior:**
- Validates that user is a conversation participant
- Ensures no active session already exists for the conversation
- Automatically adds all conversation participants to the session
- Sets the requester as the session host

**Socket Event:** `session:started` emitted to `conversation_{conversationId}` room

**Error Responses:**

**400 Bad Request - Active Session Exists:**
```json
{
  "message": "An active shopping session already exists for this conversation",
  "status_code": 400
}
```

**403 Forbidden - Not a Participant:**
```json
{
  "message": "User is not a participant in this conversation",
  "status_code": 403
}
```

---

### Join Shopping Session

**Description:** Join an active shopping session. Useful if a user joins the conversation after the session has started.

**Endpoint:** `POST /api/shopping-sessions/:sessionId/join`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |

**Example Request:**
```http
POST /api/shopping-sessions/1/join
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Joined shopping session successfully",
  "data": {
    "session": {
      "id": "1",
      "conversationId": "123",
      "status": "ACTIVE",
      "participants": [
        {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "displayName": "John Doe",
          "avatarUrl": "https://example.com/avatar.jpg",
          "joinedAt": "2026-01-24T14:00:00.000Z"
        }
      ],
      "items": [
        {
          "id": "1",
          "shopifyVariantId": "47583440404735",
          "quantity": 2,
          "price": "226.49",
          "currency": "USD",
          "addedBy": "550e8400-e29b-41d4-a716-446655440000",
          "createdAt": "2026-01-24T14:05:00.000Z"
        }
      ]
    }
  }
}
```

**Behavior:**
- Validates session is active
- Validates user is a conversation participant
- Upserts participant (rejoins if previously left)

**Socket Event:** `session:joined` emitted to `shopping_session:{sessionId}` room

---

### End Shopping Session

**Description:** End the shopping session and automatically copy all items to each participant's personal cart. This operation is **idempotent** - calling it multiple times returns the same result.

**Endpoint:** `POST /api/shopping-sessions/:sessionId/end`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |

**Example Request:**
```http
POST /api/shopping-sessions/1/end
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Shopping session ended successfully",
  "data": {
    "sessionId": "1",
    "status": "ENDED",
    "copiedItemsCount": 5,
    "generatedCarts": [
      {
        "profileId": "550e8400-e29b-41d4-a716-446655440000",
        "cartId": "10"
      },
      {
        "profileId": "660e8400-e29b-41d4-a716-446655440001",
        "cartId": "11"
      }
    ]
  }
}
```

**Behavior:**
- **Idempotent**: If already finalized, returns existing generated cart IDs
- Uses Prisma transaction for atomicity
- Marks session as `ENDED` with `endedAt` timestamp
- For each participant:
  - Gets or creates their active personal cart
  - Copies all session items to personal cart
  - If cart already has same variant, increments quantity (doesn't duplicate)
  - Sets `cart.sourceShoppingSessionId` to track origin
- Sets `finalizedAt` timestamp to prevent duplicate processing

**Important:** Only the session host can end the session.

**Socket Event:** `session:ended` emitted to `shopping_session:{sessionId}` room

**Error Responses:**

**403 Forbidden - Not Host:**
```json
{
  "message": "Only the session host can end the session",
  "status_code": 403
}
```

**400 Bad Request - Session Not Active:**
```json
{
  "message": "Shopping session is not active",
  "status_code": 400
}
```

---

## Shared Cart Management

### Get Session Cart

**Description:** Retrieve the current shared cart for an active shopping session, including all items, who added them, and like counts.

**Endpoint:** `GET /api/shopping-sessions/:sessionId/cart`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |

**Example Request:**
```http
GET /api/shopping-sessions/1/cart
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "sessionId": "1",
    "status": "ACTIVE",
    "items": [
      {
        "id": "1",
        "shopifyVariantId": "47583440404735",
        "quantity": 2,
        "price": "226.49",
        "currency": "USD",
        "addedBy": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "displayName": "John Doe",
          "avatarUrl": "https://example.com/avatar.jpg"
        },
        "product": {
          "id": "59",
          "title": "Steelseries 62406 Aerox 5 Wireless Gaming Mouse",
          "image": "https://cdn.shopify.com/s/files/1/0742/5376/2815/files/235593108784-0.jpg",
          "variant": {
            "id": "250",
            "title": "Right",
            "price": "226.49"
          }
        },
        "likesCount": 3,
        "isLiked": true,
        "createdAt": "2026-01-24T14:05:00.000Z"
      },
      {
        "id": "2",
        "shopifyVariantId": "47583440437503",
        "quantity": 1,
        "price": "41.49",
        "currency": "USD",
        "addedBy": {
          "id": "660e8400-e29b-41d4-a716-446655440001",
          "displayName": "Jane Smith",
          "avatarUrl": "https://example.com/avatar2.jpg"
        },
        "product": {
          "id": "67",
          "title": "yeah magic V1 PAW3311 12000DPI Gaming Mouse",
          "image": "https://cdn.shopify.com/s/files/1/0742/5376/2815/files/S208df27a19194baf9bfe41928bcaaf9bK.webp",
          "variant": {
            "id": "251",
            "title": "black",
            "price": "41.49"
          }
        },
        "likesCount": 1,
        "isLiked": false,
        "createdAt": "2026-01-24T14:10:00.000Z"
      }
    ]
  }
}
```

**Notes:**
- `isLiked` indicates if the current user has liked this item
- `likesCount` shows total number of likes from all participants
- Items are ordered by creation time (oldest first)

---

### Add Item to Session Cart

**Description:** Add a product variant to the shared session cart. If the same variant already exists, the quantity is incremented instead of creating a duplicate.

**Endpoint:** `POST /api/shopping-sessions/:sessionId/cart/items`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `shopifyVariantId` | string | Required | Shopify variant ID |
| `quantity` | number | Required | Quantity to add (min: 1) |

**Example Request:**
```http
POST /api/shopping-sessions/1/cart/items
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "shopifyVariantId": "47583440404735",
  "quantity": 2
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Item added to session cart",
  "data": {
    "item": {
      "id": "1",
      "shopifyVariantId": "47583440404735",
      "quantity": 2,
      "price": "226.49",
      "currency": "USD",
      "addedBy": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "displayName": "John Doe",
        "avatarUrl": "https://example.com/avatar.jpg"
      },
      "product": {
        "id": "59",
        "title": "Steelseries 62406 Aerox 5 Wireless Gaming Mouse",
        "image": "https://cdn.shopify.com/s/files/1/0742/5376/2815/files/235593108784-0.jpg",
        "variant": {
          "id": "250",
          "title": "Right",
          "price": "226.49"
        }
      },
      "likesCount": 0,
      "createdAt": "2026-01-24T14:05:00.000Z"
    }
  }
}
```

**Behavior:**
- Validates session is active
- Validates user is a session participant
- Validates variant belongs to session's seller
- If item already exists (same `shopifyVariantId`), increments quantity
- Otherwise creates new item with price snapshot
- Stores price at add time (preserves price even if variant price changes)

**Socket Event:** `session:item_added` emitted to `shopping_session:{sessionId}` room

**Error Responses:**

**400 Bad Request - Session Not Active:**
```json
{
  "message": "Shopping session is not active",
  "status_code": 400
}
```

**400 Bad Request - Wrong Seller:**
```json
{
  "message": "Product variant does not belong to session's seller",
  "status_code": 400
}
```

**404 Not Found - Variant Not Found:**
```json
{
  "message": "Product variant not found",
  "status_code": 404
}
```

---

### Update Session Cart Item

**Description:** Update the quantity of a session cart item. If quantity is set to 0 or less, the item is removed.

**Endpoint:** `PATCH /api/shopping-sessions/:sessionId/cart/items/:itemId`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |
| `itemId` | string | Required | Session cart item ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `quantity` | number | Required | New quantity (min: 0) |

**Example Request:**
```http
PATCH /api/shopping-sessions/1/cart/items/1
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "quantity": 3
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Cart item updated",
  "data": {
    "item": {
      "id": "1",
      "shopifyVariantId": "47583440404735",
      "quantity": 3,
      "price": "226.49",
      "currency": "USD",
      "likesCount": 2
    }
  }
}
```

**Example Response (Item Removed):**
```json
{
  "success": true,
  "message": "Cart item removed",
  "data": {}
}
```

**Behavior:**
- If `quantity <= 0`, item is deleted
- Otherwise, quantity is updated
- Validates session is active and user is participant

**Socket Events:**
- `session:item_updated` if quantity > 0
- `session:item_removed` if quantity <= 0

---

### Remove Session Cart Item

**Description:** Remove an item from the shared session cart.

**Endpoint:** `DELETE /api/shopping-sessions/:sessionId/cart/items/:itemId`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |
| `itemId` | string | Required | Session cart item ID |

**Example Request:**
```http
DELETE /api/shopping-sessions/1/cart/items/1
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Item removed from session cart",
  "data": {}
}
```

**Socket Event:** `session:item_removed` emitted to `shopping_session:{sessionId}` room

---

## Item Reactions

### Toggle Item Like

**Description:** Toggle a LIKE reaction on a session cart item. This is a LIKE-only reaction system (no other emoji reactions).

**Endpoint:** `POST /api/shopping-sessions/:sessionId/cart/items/:itemId/like`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sessionId` | string | Required | Shopping session ID |
| `itemId` | string | Required | Session cart item ID |

**Example Request:**
```http
POST /api/shopping-sessions/1/cart/items/1/like
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK - Liked):**
```json
{
  "success": true,
  "message": "Item liked",
  "data": {
    "liked": true,
    "likesCount": 5
  }
}
```

**Example Response (200 OK - Unliked):**
```json
{
  "success": true,
  "message": "Item unliked",
  "data": {
    "liked": false,
    "likesCount": 4
  }
}
```

**Behavior:**
- Toggle: If like exists, removes it; otherwise creates it
- Returns updated like status and total count
- Validates session is active and user is participant

**Socket Event:** `session:item_like_toggled` emitted to `shopping_session:{sessionId}` room

---

## Real-time Events (Socket.io)

### Room Convention

- **Shopping Session Room:** `shopping_session:{sessionId}`
- **Conversation Room:** `conversation_{conversationId}` (for session:started event)

### Client → Server Events

#### Join Shopping Session Room

**Event:** `join_shopping_session`

**Description:** Join a shopping session room to receive real-time updates for that specific session.

**Payload:**
```javascript
{
  sessionId: "1",  // Shopping session ID (string)
  userId: "550e8400-e29b-41d4-a716-446655440000"  // Current user's profile ID (optional, can be inferred from socket)
}
```

**Example:**
```javascript
socket.emit("join_shopping_session", {
  sessionId: "1",
  userId: "550e8400-e29b-41d4-a716-446655440000"
});
```

**When to Use:**
- When user opens/views a shopping session
- After successfully starting or joining a session
- When navigating to session cart view

**Behavior:**
- Socket joins the room `shopping_session:{sessionId}`
- User will receive all events for this session
- Multiple users can join the same session room
- User can be in multiple session rooms simultaneously

---

#### Leave Shopping Session Room

**Event:** `leave_shopping_session`

**Description:** Leave a shopping session room to stop receiving updates for that session.

**Payload:**
```javascript
{
  sessionId: "1",  // Shopping session ID (string)
  userId: "550e8400-e29b-41d4-a716-446655440000"  // Current user's profile ID (optional)
}
```

**Example:**
```javascript
socket.emit("leave_shopping_session", {
  sessionId: "1",
  userId: "550e8400-e29b-41d4-a716-446655440000"
});
```

**When to Use:**
- When user closes/leaves the shopping session view
- When navigating away from session cart
- When session is ended

**Behavior:**
- Socket leaves the room `shopping_session:{sessionId}`
- User stops receiving events for this session
- Other users in the room are not affected

---

### Session Socket Best Practices

#### 1. Join on Session Start/View

```javascript
// After starting a session
const response = await fetch('/api/shopping-sessions/start', {...});
const { session } = await response.json();

// Immediately join the session room
socket.emit("join_shopping_session", {
  sessionId: session.id,
  userId: currentUserId
});
```

#### 2. Leave on Component Unmount

```javascript
// React example
useEffect(() => {
  // Join on mount
  socket.emit("join_shopping_session", { sessionId, userId });
  
  // Leave on unmount
  return () => {
    socket.emit("leave_shopping_session", { sessionId, userId });
  };
}, [sessionId, userId]);
```

#### 3. Handle Reconnection

```javascript
socket.on("connect", () => {
  // Rejoin all active sessions after reconnection
  activeSessionIds.forEach(sessionId => {
    socket.emit("join_shopping_session", { sessionId, userId });
  });
});
```

#### 4. Error Handling

```javascript
socket.on("error", (error) => {
  console.error("Socket error:", error);
  // Implement retry logic or show user notification
});

socket.on("disconnect", () => {
  console.log("Disconnected from server");
  // Show connection status to user
});
```

### Server → Client Events

#### session:started

Emitted when a shopping session is started.

```json
{
  "sessionId": "1",
  "conversationId": "123",
  "hostId": "550e8400-e29b-41d4-a716-446655440000",
  "status": "ACTIVE",
  "participants": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "displayName": "John Doe",
      "avatarUrl": "https://example.com/avatar.jpg",
      "joinedAt": "2026-01-24T14:00:00.000Z"
    }
  ]
}
```

**Room:** `conversation_{conversationId}`

---

#### session:joined

Emitted when a user joins a shopping session.

```json
{
  "sessionId": "1",
  "profileId": "550e8400-e29b-41d4-a716-446655440000",
  "participant": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "displayName": "John Doe",
    "avatarUrl": "https://example.com/avatar.jpg"
  }
}
```

**Room:** `shopping_session:{sessionId}`

---

#### session:item_added

Emitted when an item is added to the shared cart.

```json
{
  "sessionId": "1",
  "item": {
    "id": "1",
    "shopifyVariantId": "47583440404735",
    "quantity": 2,
    "price": "226.49",
    "currency": "USD",
    "addedBy": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "displayName": "John Doe",
      "avatarUrl": "https://example.com/avatar.jpg"
    },
    "product": {
      "id": "59",
      "title": "Product Name",
      "image": "https://...",
      "variant": {
        "id": "250",
        "title": "Variant Name",
        "price": "226.49"
      }
    },
    "likesCount": 0,
    "createdAt": "2026-01-24T14:05:00.000Z"
  }
}
```

**Room:** `shopping_session:{sessionId}`

---

#### session:item_updated

Emitted when an item quantity is updated.

```json
{
  "sessionId": "1",
  "item": {
    "id": "1",
    "shopifyVariantId": "47583440404735",
    "quantity": 3,
    "price": "226.49",
    "currency": "USD",
    "likesCount": 2
  }
}
```

**Room:** `shopping_session:{sessionId}`

---

#### session:item_removed

Emitted when an item is removed from the cart.

```json
{
  "sessionId": "1",
  "itemId": "1"
}
```

**Room:** `shopping_session:{sessionId}`

---

#### session:item_like_toggled

Emitted when a like is toggled on an item.

```json
{
  "sessionId": "1",
  "itemId": "1",
  "profileId": "550e8400-e29b-41d4-a716-446655440000",
  "liked": true,
  "likesCount": 5
}
```

**Room:** `shopping_session:{sessionId}`

---

#### session:ended

Emitted when a shopping session is ended.

```json
{
  "sessionId": "1",
  "status": "ENDED",
  "copiedItemsCount": 5,
  "generatedCarts": [
    {
      "profileId": "550e8400-e29b-41d4-a716-446655440000",
      "cartId": "10"
    },
    {
      "profileId": "660e8400-e29b-41d4-a716-446655440001",
      "cartId": "11"
    }
  ]
}
```

**Room:** `shopping_session:{sessionId}`

---

### Complete Socket.io Integration Example

**Full implementation example for a shopping session:**

```javascript
import io from 'socket.io-client';

class ShoppingSessionSocket {
  constructor(userId) {
    this.userId = userId;
    this.socket = io('http://localhost:3000', {
      query: { userId }
    });
    this.activeSessions = new Set();
    this.setupEventHandlers();
  }

  // Join a shopping session room
  joinSession(sessionId) {
    if (this.activeSessions.has(sessionId)) {
      return; // Already joined
    }

    this.socket.emit("join_shopping_session", {
      sessionId,
      userId: this.userId
    });

    this.activeSessions.add(sessionId);
  }

  // Leave a shopping session room
  leaveSession(sessionId) {
    if (!this.activeSessions.has(sessionId)) {
      return; // Not joined
    }

    this.socket.emit("leave_shopping_session", {
      sessionId,
      userId: this.userId
    });

    this.activeSessions.delete(sessionId);
  }

  // Setup all event handlers
  setupEventHandlers() {
    // Session lifecycle events
    this.socket.on("session:started", (data) => {
      console.log("Session started:", data);
      // Auto-join the session room
      this.joinSession(data.sessionId);
      // Update UI to show session is active
      this.onSessionStarted(data);
    });

    this.socket.on("session:joined", (data) => {
      console.log("User joined session:", data);
      // Update participant list in UI
      this.onParticipantJoined(data);
    });

    this.socket.on("session:ended", (data) => {
      console.log("Session ended:", data);
      // Leave the session room
      this.leaveSession(data.sessionId);
      // Show notification that items were copied to personal cart
      this.onSessionEnded(data);
    });

    // Cart item events
    this.socket.on("session:item_added", (data) => {
      console.log("Item added:", data);
      // Add item to cart UI in real-time
      this.onItemAdded(data.item);
    });

    this.socket.on("session:item_updated", (data) => {
      console.log("Item updated:", data);
      // Update item quantity in cart UI
      this.onItemUpdated(data.item);
    });

    this.socket.on("session:item_removed", (data) => {
      console.log("Item removed:", data);
      // Remove item from cart UI
      this.onItemRemoved(data.itemId);
    });

    this.socket.on("session:item_like_toggled", (data) => {
      console.log("Like toggled:", data);
      // Update like count and status in UI
      this.onLikeToggled(data);
    });

    // Connection events
    this.socket.on("connect", () => {
      console.log("Connected to server");
      // Rejoin all active sessions
      this.activeSessions.forEach(sessionId => {
        this.joinSession(sessionId);
      });
    });

    this.socket.on("disconnect", () => {
      console.log("Disconnected from server");
      // Show connection status
      this.onDisconnected();
    });

    this.socket.on("error", (error) => {
      console.error("Socket error:", error);
      this.onError(error);
    });
  }

  // UI update callbacks (implement these in your component)
  onSessionStarted(data) {
    // Update UI to show session is active
  }

  onParticipantJoined(data) {
    // Update participant list
  }

  onSessionEnded(data) {
    // Show notification: "Session ended. Items copied to your cart!"
  }

  onItemAdded(item) {
    // Add item to cart list in UI
  }

  onItemUpdated(item) {
    // Update item quantity in UI
  }

  onItemRemoved(itemId) {
    // Remove item from UI
  }

  onLikeToggled(data) {
    // Update like button and count
  }

  onDisconnected() {
    // Show "Reconnecting..." status
  }

  onError(error) {
    // Show error notification
  }

  // Cleanup
  disconnect() {
    // Leave all sessions
    this.activeSessions.forEach(sessionId => {
      this.leaveSession(sessionId);
    });
    this.socket.disconnect();
  }
}

// Usage example
const sessionSocket = new ShoppingSessionSocket(currentUserId);

// Start a session and auto-join
const startSession = async () => {
  const response = await fetch('/api/shopping-sessions/start', {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      conversationId: "123",
      sellerId: "1"
    })
  });
  
  const { session } = await response.json();
  sessionSocket.joinSession(session.id);
};

// Cleanup on component unmount
// sessionSocket.disconnect();
```

---

### Socket Event Flow Diagram

```
┌─────────────────────────────────────────────────────────────┐
│                    Shopping Session Socket Flow              │
└─────────────────────────────────────────────────────────────┘

1. User Opens Session View
   ↓
2. socket.emit("join_shopping_session", { sessionId, userId })
   ↓
3. Socket joins room: shopping_session:{sessionId}
   ↓
4. User can now receive real-time events:
   ├─ session:item_added      (when someone adds item)
   ├─ session:item_updated    (when quantity changes)
   ├─ session:item_removed    (when item deleted)
   ├─ session:item_like_toggled (when someone likes/unlikes)
   ├─ session:joined          (when new participant joins)
   └─ session:ended           (when host ends session)
   ↓
5. User Leaves Session View
   ↓
6. socket.emit("leave_shopping_session", { sessionId, userId })
   ↓
7. Socket leaves room: shopping_session:{sessionId}
   ↓
8. User stops receiving events for this session
```

---

## Flow Diagram

### Complete Shopping Session Flow

```
1. User A starts shopping session in conversation
   ↓
2. POST /api/shopping-sessions/start
   - Creates ShoppingSession (ACTIVE)
   - Auto-adds all conversation participants
   - Emits session:started event
   ↓
3. Participants add items to shared cart
   ↓
4. POST /api/shopping-sessions/:sessionId/cart/items
   - Adds item to ShoppingSessionCartItem
   - Emits session:item_added event (real-time)
   ↓
5. Participants can:
   - Update quantities (PATCH)
   - Remove items (DELETE)
   - Like items (POST /like)
   - All actions emit real-time events
   ↓
6. Host ends session
   ↓
7. POST /api/shopping-sessions/:sessionId/end
   - Marks session as ENDED
   - Uses Prisma transaction
   - For each participant:
     * Gets/creates personal ACTIVE cart
     * Copies all session items
     * Merges quantities if variant exists
     * Sets cart.sourceShoppingSessionId
   - Emits session:ended event
   ↓
8. Each participant checks out individually
   ↓
9. POST /api/checkout (existing endpoint)
   - Creates Shopify Draft Order
   - Returns invoice_url
   - User completes payment in Shopify Checkout
```

---

## Error Responses

### Common Error Codes

**400 Bad Request:**
```json
{
  "message": "Error message",
  "status_code": 400
}
```

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

**403 Forbidden:**
```json
{
  "message": "User is not a participant in this conversation",
  "status_code": 403
}
```

**404 Not Found:**
```json
{
  "message": "Shopping session not found",
  "status_code": 404
}
```

**500 Internal Server Error:**
```json
{
  "message": "Internal server error",
  "status_code": 500
}
```

---

## Important Notes

### Session Behavior

- ✅ **One Active Session Per Conversation**: Only one active session can exist per conversation
- ✅ **Auto-Participant Addition**: All conversation participants are automatically added when session starts
- ✅ **Host-Only End**: Only the session host can end the session (configurable)
- ✅ **Idempotent End**: Ending session multiple times returns the same result

### Shared Cart Behavior

- ✅ **Separate Storage**: Shared cart items stored in `ShoppingSessionCartItem` (not personal carts)
- ✅ **Price Snapshots**: Prices stored at add time (preserves price even if variant price changes)
- ✅ **Single Seller**: All items must be from the same seller
- ✅ **Auto-Consolidation**: Adding same variant multiple times increments quantity (doesn't duplicate)
- ✅ **Real-time Sync**: All cart changes emit Socket.io events for live updates

### Session End Behavior

- ✅ **Idempotent**: Can be called multiple times safely (returns existing generated carts)
- ✅ **Transaction Safety**: Uses Prisma transaction for atomicity
- ✅ **Copy to Personal Carts**: All session items copied to each participant's personal cart
- ✅ **Quantity Merge**: If personal cart already has variant, quantities are added (not duplicated)
- ✅ **Source Tracking**: `cart.sourceShoppingSessionId` tracks which session generated the cart

### Personal Cart After Session End

- ✅ **Individual Checkout**: Each participant checks out their personal cart separately
- ✅ **Existing Flow**: Uses existing `POST /api/checkout` endpoint (unchanged)
- ✅ **No Breaking Changes**: Personal cart → checkout flow remains intact
- ✅ **Cart Management**: Users can modify or delete items in their personal cart before checkout

### Like Reactions

- ✅ **LIKE-Only**: Only LIKE reactions supported (no other emoji reactions)
- ✅ **Toggle Behavior**: Clicking like again removes the like
- ✅ **Real-time Updates**: Like toggles emit Socket.io events

### Authorization

- ✅ **Conversation Participants Only**: Only conversation participants can start/join sessions
- ✅ **Session Participants Only**: Only session participants can modify session cart
- ✅ **Host-Only End**: Only session host can end session (by default)

---

## Testing Guide

### Complete Flow Test

1. **Start Shopping Session:**
   ```http
   POST /api/shopping-sessions/start
   {
     "conversationId": "123",
     "sellerId": "1"
   }
   ```
   Copy `sessionId` from response.

2. **Join Session Room (Socket.io):**
   ```javascript
   socket.emit("join_shopping_session", {
     sessionId: "1",
     userId: "your-user-id"
   });
   ```

3. **Add Items to Shared Cart:**
   ```http
   POST /api/shopping-sessions/1/cart/items
   {
     "shopifyVariantId": "47583440404735",
     "quantity": 2
   }
   ```
   Verify `session:item_added` event is received.

4. **Get Shared Cart:**
   ```http
   GET /api/shopping-sessions/1/cart
   ```
   Verify items are returned.

5. **Like an Item:**
   ```http
   POST /api/shopping-sessions/1/cart/items/1/like
   ```
   Verify `session:item_like_toggled` event is received.

6. **End Session:**
   ```http
   POST /api/shopping-sessions/1/end
   ```
   Verify `generatedCarts` array shows all participants' cart IDs.

7. **Check Personal Cart:**
   ```http
   GET /api/cart
   ```
   Verify items from session are now in personal cart.

8. **Checkout Personal Cart:**
   ```http
   POST /api/checkout
   ```
   Uses existing checkout flow (unchanged).

---

## Related Documentation

- [shopify-checkout.md](./shopify-checkout.md) - Personal cart checkout flow
- [SHOPPING_SESSION_IMPLEMENTATION.md](../docs/features/SHOPPING_SESSION_IMPLEMENTATION.md) - Technical implementation details

---

## Support

For issues or questions:
1. Check server logs for error messages
2. Verify user is a conversation participant
3. Ensure session is ACTIVE before modifying cart
4. Verify Socket.io connection is established

**Last Updated:** January 24, 2026  
**Version:** 1.0
