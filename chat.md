# Chat API Documentation

Real-time chat functionality between users using Socket.io for instant messaging.

## 📋 Overview

The chat system supports:
- **Direct conversations** between two users
- **Group conversations** with multiple participants
- **Automatic group conversion** when adding participants to direct chats
- **Text messages**
- **Image messages** (uploaded to Cloudinary) with optional captions
- **File messages** (PDF, DOC, DOCX, ZIP, RAR) with optional captions
- **Real-time messaging** via Socket.io
- **Read receipts**
- **Typing indicators**
- **Online/offline status**
- **Message pagination**
- **Dynamic group naming** (computed from participant names)
- **Role-based permissions** (admin/member in groups)

## 🔐 Authentication

All endpoints require authentication with Bearer token:
```
Authorization: Bearer <your-token>
```

Allowed roles: `USER`, `SELLER`, `MEDIATOR`

---

## 📡 Socket.io Events

### Client → Server Events

#### 1. Connect to Socket
```javascript
const socket = io("http://localhost:3000", {
  query: {
    userId: "your-user-id-uuid"
  }
});
```

#### 2. Join Conversation Room
```javascript
socket.emit("join_conversation", {
  conversationId: "123",
  userId: "your-user-id",
  participantIds: ["user-id-1", "user-id-2"] // All participant IDs for proper room management
});
```

#### 3. Leave Conversation Room
```javascript
socket.emit("leave_conversation", {
  conversationId: "123",
  userId: "your-user-id"
});
```

#### 4. Typing Indicator
```javascript
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: true // or false
});
```

### Server → Client Events

#### 1. New Message
```javascript
socket.on("new_message", (data) => {
  console.log("New message:", data);
  // {
  //   conversationId: "123",
  //   message: {
  //     id: "456",
  //     conversationId: "123",
  //     senderId: "user-id",
  //     type: "TEXT" | "IMAGE" | "FILE",
  //     content: "Message content or file URL",
  //     meta: {
  //       // For IMAGE: imageUrl, publicId, width, height, caption (optional)
  //       // For FILE: imageUrl, publicId, resource_type, fileName, fileSize, size, caption (optional)
  //     },
  //     createdAt: "2024-01-01T00:00:00Z",
  //     readAt: null,
  //     sender: {
  //       id: "user-id",
  //       displayName: "User Name",
  //       avatarUrl: "https://..."
  //     }
  //   }
  // }
});
```
**Note:** Messages are emitted to all participants **except** the sender to prevent duplication.

#### 2. Messages Read
```javascript
socket.on("messages_read", (data) => {
  console.log("Messages read:", data);
  // {
  //   conversationId: "123",
  //   readBy: "user-id",
  //   messageIds: ["msg-1", "msg-2"] // Empty array means all messages marked as read
  // }
});
```

#### 3. User Online
```javascript
socket.on("user_online", (data) => {
  console.log("User online:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 4. User Offline
```javascript
socket.on("user_offline", (data) => {
  console.log("User offline:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 5. Typing Indicator
```javascript
socket.on("typing", (data) => {
  console.log("User typing:", data);
  // {
  //   conversationId: "123",
  //   userId: "user-id",
  //   isTyping: true
  // }
});
```

---

## 📨 Conversation Endpoints

### Create or Get Conversation

Create a new direct conversation with another user, or get existing conversation.

**Endpoint:** `POST /api/conversations`

**Request Body:**
```json
{
  "participantId": "uuid-of-other-user"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Conversation created successfully",
  "data": {
    "id": "123",
    "type": "DIRECT",
    "participants": [
      {
        "profileId": "your-user-id",
        "displayName": "Your Name",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      },
      {
        "profileId": "other-user-id",
        "displayName": "Other User",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      }
    ]
  }
}
```

**Notes:**
- Users must be friends to create a conversation
- If a direct conversation already exists, returns the existing one
- Returns `200` status if conversation already exists, `201` if newly created

---

### Get All Conversations

Get all conversations for the current user with pagination.

**Endpoint:** `GET /api/conversations`

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 50)

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 10,
  "pages": 1,
  "count": 10,
  "data": [
    {
      "id": "123",
      "type": "DIRECT",
      "name": null, // Only for GROUP conversations
      "participant": {
        "id": "uuid",
        "displayName": "User Name",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "participants": [
        {
          "profileId": "uuid",
          "displayName": "User Name",
          "avatarUrl": "https://...",
          "roleInChat": "member"
        }
      ],
      "lastMessage": {
        "id": "456",
        "content": "Last message",
        "type": "TEXT",
        "senderId": "uuid",
        "senderName": "Sender Name",
        "createdAt": "2024-01-01T00:00:00Z"
      },
      "unreadCount": 2
    },
    {
      "id": "124",
      "type": "GROUP",
      "name": "Alice, Bob, Charlie", // Computed from participant names
      "participant": null, // null for group chats
      "participants": [
        {
          "profileId": "uuid-1",
          "displayName": "Alice",
          "avatarUrl": "https://...",
          "roleInChat": "admin"
        },
        {
          "profileId": "uuid-2",
          "displayName": "Bob",
          "avatarUrl": "https://...",
          "roleInChat": "member"
        }
      ],
      "lastMessage": {
        "id": "457",
        "content": "Group message",
        "type": "TEXT",
        "senderId": "uuid-1",
        "senderName": "Alice",
        "createdAt": "2024-01-01T00:00:00Z"
      },
      "unreadCount": 0
    }
  ]
}
```

**Notes:**
- Conversations are ordered by most recent message
- Unread counts are efficiently batched in a single query
- Group names are computed dynamically from participant display names (sorted alphabetically)

---

### Get Conversation by ID

Get detailed information about a specific conversation.

**Endpoint:** `GET /api/conversations/:conversationId`

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "123",
    "type": "DIRECT" | "GROUP",
    "name": "Alice, Bob, Charlie", // null for DIRECT, computed for GROUP
    "createdBy": "creator-user-id",
    "createdAt": "2024-01-01T00:00:00Z",
    "participants": [
      {
        "profileId": "uuid",
        "displayName": "User Name",
        "avatarUrl": "https://...",
        "role": "USER",
        "roleInChat": "admin" | "member",
        "joinedAt": "2024-01-01T00:00:00Z"
      }
    ],
    "lastMessage": {
      "id": "456",
      "content": "Last message",
      "type": "TEXT",
      "senderId": "uuid",
      "senderName": "Sender Name",
      "createdAt": "2024-01-01T00:00:00Z"
    }
  }
}
```

**Notes:**
- Only participants can access conversation details
- Group names are computed on-the-fly from participant names

---

### Add Participants to Conversation

Add friends to a conversation. **Automatically converts direct chats to groups** when participants are added.

**Endpoint:** `POST /api/conversations/:conversationId/participants`

**Request Body:**
```json
{
  "participantIds": ["user-id-1", "user-id-2"]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Chat converted to group and 2 participant(s) added successfully",
  "data": {
    "id": "123",
    "type": "GROUP",
    "name": "Alice, Bob, Charlie, David", // Computed from all participants
    "participants": [
      {
        "profileId": "uuid-1",
        "displayName": "Alice",
        "avatarUrl": "https://...",
        "roleInChat": "admin"
      },
      {
        "profileId": "uuid-2",
        "displayName": "Bob",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      },
      {
        "profileId": "uuid-3",
        "displayName": "Charlie",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      },
      {
        "profileId": "uuid-4",
        "displayName": "David",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      }
    ]
  }
}
```

**Notes:**
- If adding to a **direct chat**, it automatically converts to a **GROUP**
- The conversation creator is automatically promoted to **admin** when converted
- New participants are added as **members**
- For existing groups, only **admins** can add participants
- All users must be friends with the current user
- Duplicate participants are filtered out
- Group name is automatically regenerated from all participant names

---

### Remove Participant from Group

Remove a participant from a group conversation.

**Endpoint:** `DELETE /api/conversations/:conversationId/participants/:participantId`

**Response:**
```json
{
  "success": true,
  "message": "Participant removed successfully" // or "You left the group" if self-removal
}
```

**Notes:**
- Only **admins** can remove other participants
- **Members** can only remove themselves
- Only works for **GROUP** conversations
- After removal, group name is recomputed (but not stored in DB)

---

### Convert Direct Chat to Group (Optional)

Manually convert a direct chat to a group. **Note:** This happens automatically when adding participants, so this endpoint is optional.

**Endpoint:** `POST /api/conversations/:conversationId/convert-to-group`

**Request Body:**
```json
{}
```

**Response:**
```json
{
  "success": true,
  "message": "Conversation converted to group successfully",
  "data": {
    "id": "123",
    "type": "GROUP",
    "name": "Alice, Bob" // Computed from participants
  }
}
```

**Notes:**
- The conversation creator becomes an **admin**
- Other participant becomes a **member**
- Group name is computed from participant names

---

## 💬 Message Endpoints

### Send Text Message

Send a text message in a conversation.

**Endpoint:** `POST /api/messages/:conversationId/text`

**Request Body:**
```json
{
  "content": "Hello, how are you?",
  "type": "TEXT"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Message sent successfully",
  "data": {
    "id": "789",
    "conversationId": "123",
    "senderId": "your-user-id",
    "type": "TEXT",
    "content": "Hello, how are you?",
    "meta": null,
    "createdAt": "2024-01-01T00:00:00Z",
    "readAt": null,
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- Message is automatically emitted via Socket.io to all other participants
- Sender receives the message only through API response (prevents duplication)

---

### Send Image Message

Send an image message with optional caption.

**Endpoint:** `POST /api/messages/:conversationId/image`

**Request:** Form-data (multipart/form-data)
- `image`: Image file (required) - Max 10MB
- `caption`: Optional text caption

**Supported Image Types:**
- JPEG, JPG, PNG, GIF, WebP, BMP, TIFF, ICO

**Response:**
```json
{
  "success": true,
  "message": "Image message sent successfully",
  "data": {
    "id": "790",
    "conversationId": "123",
    "senderId": "your-user-id",
    "type": "IMAGE",
    "content": "https://cloudinary.com/image-url.jpg",
    "meta": {
      "imageUrl": "https://cloudinary.com/image-url.jpg",
      "publicId": "chat/images/123/user_id_timestamp",
      "width": 1920,
      "height": 1080,
      "caption": "Check out this image!" // Optional
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "readAt": null,
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- Images are uploaded to Cloudinary under `chat/images/{conversationId}/`
- Caption is optional and stored in `meta.caption`
- Max file size: 10MB

---

### Send File Message

Send a document file (PDF, DOC, DOCX, ZIP, RAR) with optional caption.

**Endpoint:** `POST /api/messages/:conversationId/file`

**Request:** Form-data (multipart/form-data)
- `file`: File (required) - Max 15MB
- `caption`: Optional text caption

**Supported File Types:**
- PDF (`.pdf`)
- Word Documents (`.doc`, `.docx`)
- Archives (`.zip`, `.rar`)

**Response:**
```json
{
  "success": true,
  "message": "File message sent successfully",
  "data": {
    "id": "791",
    "conversationId": "123",
    "senderId": "your-user-id",
    "type": "IMAGE", // Uses IMAGE type; meta.resource_type distinguishes it
    "content": "https://cloudinary.com/file-url.pdf",
    "meta": {
      "imageUrl": "https://cloudinary.com/file-url.pdf",
      "publicId": "chat/files/123/user_id_timestamp",
      "resource_type": "raw", // "raw" for PDFs/docs, "image" for images
      "fileName": "document.pdf",
      "fileSize": 1048576,
      "size": 1048576,
      "caption": "Here's the document" // Optional
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "readAt": null,
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- Files are uploaded to Cloudinary under `chat/files/{conversationId}/`
- Uses `IMAGE` message type, but `meta.resource_type` distinguishes files from images
- Caption is optional and stored in `meta.caption`
- Max file size: 15MB

---

### Get Messages

Get messages for a conversation with pagination and infinite scroll support.

**Endpoint:** `GET /api/messages/:conversationId`

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 50, max: 50)
- `before` (optional): Message ID to get messages before this ID (for infinite scroll)

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 50,
  "total": 150,
  "pages": 3,
  "count": 50,
  "data": [
    {
      "id": "789",
      "conversationId": "123",
      "senderId": "user-id",
      "type": "TEXT",
      "content": "Message content",
      "meta": null,
      "createdAt": "2024-01-01T00:00:00Z",
      "readAt": null,
      "sender": {
        "id": "user-id",
        "displayName": "User Name",
        "avatarUrl": "https://..."
      }
    },
    {
      "id": "790",
      "conversationId": "123",
      "senderId": "user-id",
      "type": "IMAGE",
      "content": "https://cloudinary.com/image.jpg",
      "meta": {
        "imageUrl": "https://cloudinary.com/image.jpg",
        "publicId": "chat/images/123/user_id_timestamp",
        "width": 1920,
        "height": 1080,
        "caption": "Image caption"
      },
      "createdAt": "2024-01-01T00:00:00Z",
      "readAt": null,
      "sender": {
        "id": "user-id",
        "displayName": "User Name",
        "avatarUrl": "https://..."
      }
    }
  ]
}
```

**Notes:**
- Messages are returned in **reverse chronological order** (newest first)
- Use `before` parameter for infinite scroll (load older messages)
- Pagination parameters are validated (page ≥ 1, size 1-50)

---

### Mark Messages as Read

Mark messages as read in a conversation.

**Endpoint:** `PATCH /api/messages/:conversationId/read`

**Request Body:**
```json
{
  "messageIds": ["789", "790"] // Optional: specific message IDs, or omit/empty array to mark all unread as read
}
```

**Response:**
```json
{
  "success": true,
  "message": "Messages marked as read",
  "data": {
    "updatedCount": 5
  }
}
```

**Notes:**
- If `messageIds` is empty or omitted, marks **all unread messages** in the conversation as read
- Automatically emits `messages_read` event via Socket.io
- Only marks messages that the current user hasn't sent

---

## 🧪 Testing Guide

### Postman Setup

1. **Authentication:**
   - Get token from login endpoint: `POST /api/auth/login`
   - Set as header: `Authorization: Bearer <token>`

2. **Socket.io Connection (JavaScript):**
   ```javascript
   import io from "socket.io-client";
   
   const socket = io("http://localhost:3000", {
     query: { userId: "your-user-id" }
   });
   
   socket.on("connect", () => {
     console.log("Connected to Socket.io");
   });
   
   // Join conversation
   socket.emit("join_conversation", {
     conversationId: "123",
     userId: "your-user-id",
     participantIds: ["user-1", "user-2"]
   });
   
   // Listen for new messages
   socket.on("new_message", (data) => {
     console.log("New message received:", data);
   });
   
   // Listen for typing indicators
   socket.on("typing", (data) => {
     console.log("User typing:", data);
   });
   ```

### Testing Flow

1. **Create/Get Conversation:**
   ```
   POST /api/conversations
   Body: { "participantId": "other-user-uuid" }
   ```

2. **Add Participants (Converts to Group):**
   ```
   POST /api/conversations/{conversationId}/participants
   Body: { "participantIds": ["user-id-1", "user-id-2"] }
   ```

3. **Send Text Message:**
   ```
   POST /api/messages/{conversationId}/text
   Body: { "content": "Hello!", "type": "TEXT" }
   ```

4. **Send Image Message:**
   ```
   POST /api/messages/{conversationId}/image
   Form-data: 
     - image: [file]
     - caption: "Optional caption" (optional)
   ```

5. **Send File Message:**
   ```
   POST /api/messages/{conversationId}/file
   Form-data:
     - file: [pdf/doc/zip file]
     - caption: "Optional caption" (optional)
   ```

6. **Get Messages:**
   ```
   GET /api/messages/{conversationId}?page=1&size=50
   GET /api/messages/{conversationId}?page=1&size=50&before=message-id
   ```

7. **Mark as Read:**
   ```
   PATCH /api/messages/{conversationId}/read
   Body: { "messageIds": ["msg-id-1", "msg-id-2"] }
   // or
   Body: {} // marks all unread as read
   ```

8. **Remove Participant:**
   ```
   DELETE /api/conversations/{conversationId}/participants/{participantId}
   ```

---

## 📝 Implementation Notes

### Conversation Types

- **DIRECT**: One-on-one conversation between two users
- **GROUP**: Multi-participant conversation

### Group Management

- **Automatic Conversion**: Adding participants to a direct chat automatically converts it to a group
- **Group Naming**: Group names are **computed dynamically** from participant display names (sorted alphabetically), not stored in the database
- **Roles**:
  - `admin`: Can add/remove participants (conversation creator automatically becomes admin)
  - `member`: Regular participant
- **Permissions**: Only admins can add participants to existing groups; any member can leave

### Message Types

- **TEXT**: Plain text messages
- **IMAGE**: Image files (with optional caption)
- **FILE**: Document files (PDF, DOC, DOCX, ZIP, RAR) - uses `IMAGE` type but distinguished by `meta.resource_type`

### File Storage

- **Images**: Stored in Cloudinary under `chat/images/{conversationId}/`
- **Files**: Stored in Cloudinary under `chat/files/{conversationId}/`
- **File Size Limits**:
  - Images: 10MB max
  - Files: 15MB max

### Socket.io Behavior

- Messages are emitted to **all participants except the sender** to prevent duplication
- The sender receives the message only through the API response
- Read receipts are automatically emitted when messages are marked as read
- Online/offline status is tracked via Socket.io connection/disconnection

### Data Models

- **Conversation IDs**: BigInt (returned as strings)
- **Message IDs**: BigInt (returned as strings)
- **Pagination**: Messages are in reverse chronological order (newest first)
- **Unread Counts**: Efficiently batched in a single query for multiple conversations

### Validation

- Conversation creation requires users to be friends
- Only friends can be added as participants
- File types are strictly validated
- File sizes are enforced (10MB for images, 15MB for files)
- Pagination parameters are validated (page ≥ 1, size 1-50)

---

## 🔗 Related Documentation

- [Authentication](./auth.md) - How to get authentication token
- [Profiles](./profiles.md) - User profile management
- [Social Features](./social.md) - Social features overview (friends, etc.)

---

## 🐛 Error Codes

- `400`: Bad Request (invalid input, duplicate participants, etc.)
- `403`: Forbidden (not a participant, insufficient permissions)
- `404`: Not Found (conversation not found, message not found)
- `500`: Internal Server Error (upload failures, database errors)

Common error messages:
- `"You can only create conversations with friends"`
- `"You can only add friends to conversations"`
- `"Only admins can add participants to groups"`
- `"You are not a participant in this conversation"`
- `"Conversation not found"`
