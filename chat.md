# Chat API Documentation

Real-time chat functionality between users using Socket.io for instant messaging.

## 📋 Overview

The chat system supports:
- **Direct conversations** between two users (friends only)
- **Group chats** with multiple participants (NEW!)
- **Group management** with admin/member roles (NEW!)
- **Text messages**
- **Image messages** (uploaded to Cloudinary) with optional captions
- **File messages** (PDF, DOC, DOCX, ZIP, RAR) with optional captions
- **Message reactions** with emoji (👍❤️😂😮😢🙏 and more)
- **Delete messages** (soft delete - sender only)
- **Real-time messaging** via Socket.io
- **Read receipts** with checkmarks
- **Typing indicators** with multi-user support
- **Online/offline status**
- **Message pagination** with infinite scroll
- **Message grouping** by date and sender
- **Optimistic UI** for instant feedback

## 🔐 Authentication

All endpoints require authentication with token in the authorization header:
```
authorization: <your-token>
```

**Note:** The backend now accepts tokens **without** the "Bearer " prefix. Simply send the token directly in the `authorization` header (lowercase).

Allowed roles: `USER`, `SELLER`, `MEDIATOR`, `ADMIN`

---

## 📡 Socket.io Events

### Client → Server Events

#### 1. Connect to Socket
```javascript
// For local development
const socket = io("http://localhost:3000", {
  query: {
    userId: "your-user-id-uuid"
  }
});

// For production
const socket = io("https://done-buddy-production.up.railway.app", {
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

#### 4. Typing/Recording Indicator
```javascript
// Typing indicator
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: true,
  isRecording: false,
  participantIds: ["user-id-1", "user-id-2"] // Optional: for chat list updates
});

// Recording indicator (voice message)
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: false,
  isRecording: true,
  participantIds: ["user-id-1", "user-id-2"] // Optional: for chat list updates
});

// Both can be sent together (edge case)
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: true,
  isRecording: true,
  participantIds: ["user-id-1", "user-id-2"] // Optional: for chat list updates
});

// Stop all indicators
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: false,
  isRecording: false,
  participantIds: ["user-id-1", "user-id-2"] // Optional: for chat list updates
});
```

**Note:** The `participantIds` parameter is optional but **recommended**. When provided:
- ✅ Typing indicators appear in the **chat list** (conversation list view)
- ✅ Users can see activity even when **outside** the conversation
- ✅ Better UX - know which conversations have active users

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
  //     editedAt: null,
  //     deletedAt: null,
  //     reactions: [
  //       {
  //         id: "1",
  //         emoji: "👍",
  //         profileId: "user-id",
  //         profile: { id: "user-id", displayName: "User", avatarUrl: "..." },
  //         createdAt: "2024-01-01T00:00:00Z"
  //       }
  //     ],
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

#### 3. Typing/Recording Indicator
```javascript
socket.on("typing", (data) => {
  console.log("User typing/recording:", data);
  // {
  //   conversationId: "123",
  //   userId: "user-id",
  //   isTyping: true,      // User is typing text
  //   isRecording: false   // User is NOT recording voice
  // }
  
  // Handle different states
  if (data.isTyping) {
    // Show "User is typing..." indicator
    // In active chat: show at bottom of messages
    // In chat list: show next to conversation
  }
  if (data.isRecording) {
    // Show "User is recording..." indicator
    // In active chat: show at bottom of messages
    // In chat list: show "🎙️ Recording..." next to conversation
  }
});
```

**Usage in Different Views:**

**Inside Active Chat:**
```javascript
// Show indicator at bottom of chat
if (data.conversationId === currentConversationId) {
  if (data.isTyping) showTypingIndicator(data.userId);
  if (data.isRecording) showRecordingIndicator(data.userId);
}
```

**In Chat List (Outside Conversation):**
```javascript
// Show indicator next to conversation in list
socket.on("typing", (data) => {
  updateConversationIndicator(data.conversationId, {
    isTyping: data.isTyping,
    isRecording: data.isRecording,
    userId: data.userId
  });
});
```

#### 4. Message Deleted (NEW)
```javascript
socket.on("message_deleted", (data) => {
  console.log("Message deleted:", data);
  // {
  //   conversationId: "123",
  //   messageId: "456"
  // }
});
```

#### 5. Message Reaction (NEW)
```javascript
socket.on("message_reaction", (data) => {
  console.log("Message reaction:", data);
  // {
  //   conversationId: "123",
  //   messageId: "456",
  //   reaction: {
  //     id: "1",
  //     emoji: "👍",
  //     profileId: "user-id",
  //     profile: { id: "user-id", displayName: "User", avatarUrl: "..." },
  //     createdAt: "2024-01-01T00:00:00Z"
  //   }
  // }
});
```

#### 6. Message Reaction Removed (NEW)
```javascript
socket.on("message_reaction_removed", (data) => {
  console.log("Reaction removed:", data);
  // {
  //   conversationId: "123",
  //   messageId: "456",
  //   reactionId: "1",
  //   profileId: "user-id"
  // }
});
```

#### 7. User Online
```javascript
socket.on("user_online", (data) => {
  console.log("User online:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 8. User Offline
```javascript
socket.on("user_offline", (data) => {
  console.log("User offline:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 9. Last Message Update (NEW)
```javascript
socket.on("last_message_update", (data) => {
  console.log("Last message updated:", data);
  // {
  //   conversationId: "123",
  //   lastMessage: {
  //     id: "456",
  //     content: "Last message content",
  //     type: "TEXT",
  //     senderId: "user-id",
  //     senderName: "User Name",
  //     createdAt: "2024-01-01T00:00:00Z"
  //   },
  //   unreadCount: 3
  // }
  
  // Update conversation in your chat list
  updateConversationInList(data.conversationId, {
    lastMessage: data.lastMessage,
    unreadCount: data.unreadCount,
    timestamp: new Date()
  });
});
```

**Note:** This event is emitted to all participants whenever a new message is sent. It provides the last message details and the unread count for each participant. This is useful for updating the conversation list outside of the active chat.

#### 10. Conversation Created (NEW)
```javascript
socket.on("conversation_created", (data) => {
  console.log("New conversation:", data);
  // {
  //   conversation: {
  //     id: "123",
  //     type: "DIRECT" | "GROUP",
  //     name: "Group Name" (if GROUP),
  //     avatarUrl: "https://...",
  //     createdAt: "2024-01-01T00:00:00Z",
  //     participants: [...]
  //   }
  // }
  
  // Add new conversation to your chat list
  addConversationToList(data.conversation);
});
```

**Note:** Emitted when someone creates a new conversation with you or adds you to a group. Automatically appears in your chat list.

#### 11. Conversation Updated (NEW)
```javascript
socket.on("conversation_updated", (data) => {
  console.log("Conversation updated:", data);
  // {
  //   conversation: {
  //     id: "123",
  //     type: "GROUP",
  //     name: "Updated Group Name",
  //     avatarUrl: "https://...",
  //     description: "New description",
  //     participants: [...]
  //   }
  // }
  
  // Update conversation in your chat list
  updateConversationInList(data.conversation);
});
```

**Note:** Emitted when group details change (name, avatar, description). All participants receive this update.

#### 12. Conversation Deleted (NEW)
```javascript
socket.on("conversation_deleted", (data) => {
  console.log("Conversation deleted:", data);
  // {
  //   conversationId: "123"
  // }
  
  // Remove conversation from your chat list
  removeConversationFromList(data.conversationId);
});
```

**Note:** Emitted when a group is deleted or when you're removed from a conversation. Removes it from your chat list.

#### 13. Participant Added (NEW)
```javascript
socket.on("participant_added", (data) => {
  console.log("Participant added:", data);
  // {
  //   conversationId: "123",
  //   participant: {
  //     profileId: "user-id",
  //     displayName: "User Name",
  //     avatarUrl: "https://...",
  //     roleInChat: "member",
  //     joinedAt: "2024-01-01T00:00:00Z"
  //   }
  // }
  
  // Update participants list for this conversation
  addParticipantToConversation(data.conversationId, data.participant);
});
```

**Note:** Emitted when a new member is added to a group. All existing participants receive this.

#### 14. Participant Removed (NEW)
```javascript
socket.on("participant_removed", (data) => {
  console.log("Participant removed:", data);
  // {
  //   conversationId: "123",
  //   participantId: "user-id"
  // }
  
  // If it's you, remove conversation from list
  if (data.participantId === currentUserId) {
    removeConversationFromList(data.conversationId);
  } else {
    // Otherwise, just update participants
    removeParticipantFromConversation(data.conversationId, data.participantId);
  }
});
```

**Note:** Emitted when someone is removed from a group. If you're removed, the conversation disappears from your list.

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
- Only supports **DIRECT** conversations (for groups, use Create Group endpoint)

---

### Create Group Conversation (NEW)

Create a new group conversation with multiple friends.

**Endpoint:** `POST /api/conversations/groups`

**Request Body:**
```json
{
  "name": "Study Group",
  "participantIds": ["uuid-1", "uuid-2", "uuid-3"],
  "avatarUrl": "https://...",
  "description": "A group for studying together"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Group created successfully",
  "data": {
    "id": "123",
    "type": "GROUP",
    "name": "Study Group",
    "avatarUrl": "https://...",
    "description": "A group for studying together",
    "createdBy": "your-user-id",
    "createdAt": "2024-01-01T00:00:00Z",
    "participants": [
      {
        "profileId": "your-user-id",
        "displayName": "Your Name",
        "avatarUrl": "https://...",
        "roleInChat": "admin"
      },
      {
        "profileId": "uuid-1",
        "displayName": "User 1",
        "avatarUrl": "https://...",
        "roleInChat": "member"
      }
    ]
  }
}
```

**Notes:**
- Group name is **required** (1-255 characters)
- At least **1 participant** required (besides creator)
- All participants must be **friends** with creator
- Creator automatically becomes **admin**
- avatarUrl and description are **optional**

---

### Update Group Details (NEW)

Update group name, description, or avatar (admin only).

**Endpoint:** `PATCH /api/conversations/:conversationId/groups`

**Request Body:**
```json
{
  "name": "Updated Group Name",
  "description": "Updated description",
  "avatarUrl": "https://..."
}
```

**Response:**
```json
{
  "success": true,
  "message": "Group updated successfully",
  "data": {
    "id": "123",
    "type": "GROUP",
    "name": "Updated Group Name",
    "avatarUrl": "https://...",
    "description": "Updated description",
    "participants": [...]
  }
}
```

**Notes:**
- **Admin only** - only group admins can update
- All fields are optional
- Omitted fields remain unchanged
- Real-time updates via Socket.io

**Error Responses:**
- `403`: If user is not an admin

---

### Add Participants to Group (NEW)

Add new members to a group (admin only).

**Endpoint:** `POST /api/conversations/:conversationId/participants`

**Request Body:**
```json
{
  "participantIds": ["uuid-4", "uuid-5"]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Participants added successfully",
  "data": {
    "id": "123",
    "type": "GROUP",
    "name": "Study Group",
    "participants": [
      {
        "profileId": "uuid-4",
        "displayName": "User 4",
        "avatarUrl": "https://...",
        "roleInChat": "member",
        "joinedAt": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

**Notes:**
- **Admin only** - only group admins can add members
- New members must be **friends** with admin
- New members added as **members** (not admin)
- Duplicate participants are ignored

**Error Responses:**
- `403`: If user is not an admin
- `400`: If participants are not friends

---

### Remove Participant from Group (NEW)

Remove a member from group (admin) or leave group (self).

**Endpoint:** `DELETE /api/conversations/:conversationId/participants/:participantId`

**Response:**
```json
{
  "success": true,
  "message": "Participant removed successfully"
}
```

**Notes:**
- **Admins** can remove any member
- **Any member** can remove themselves (leave group)
- Cannot remove **last admin** (must promote someone first)
- If last member leaves, **group is deleted**

**Special Response (Last Member):**
```json
{
  "success": true,
  "message": "Group deleted (last member left)"
}
```

**Error Responses:**
- `403`: If trying to remove others without admin permission
- `400`: If trying to remove last admin

---

### Promote Member to Admin (NEW)

Promote a group member to admin role (admin only).

**Endpoint:** `PATCH /api/conversations/:conversationId/participants/:participantId/promote`

**Response:**
```json
{
  "success": true,
  "message": "Member promoted to admin successfully",
  "data": {
    "profileId": "uuid-1",
    "displayName": "User 1",
    "roleInChat": "admin"
  }
}
```

**Notes:**
- **Admin only** - only current admins can promote
- Promoted member gains **all admin permissions**
- Groups can have **multiple admins**
- Cannot demote admins (feature not implemented)

**Error Responses:**
- `403`: If user is not an admin
- `404`: If participant not found

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
      "name": null,
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
    }
  ]
}
```

**Notes:**
- Conversations are ordered by most recent message
- Unread counts are efficiently batched in a single query
- Returns both **DIRECT** and **GROUP** conversations
- For groups, `name` field contains the group name
- For direct chats, `name` is null and `participant` contains the other user

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
    "type": "DIRECT",
    "name": null,
    "createdBy": "creator-user-id",
    "createdAt": "2024-01-01T00:00:00Z",
    "participants": [
      {
        "profileId": "uuid",
        "displayName": "User Name",
        "avatarUrl": "https://...",
        "role": "USER",
        "roleInChat": "member",
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
- Works for both **DIRECT** and **GROUP** conversations
- Group conversations include all participants with roles
- Direct conversations show the other participant info

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
    "editedAt": null,
    "deletedAt": null,
    "reactions": [],
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
- Supports **optimistic UI** - can display immediately before server confirms

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
      "caption": "Check out this image!"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "readAt": null,
    "editedAt": null,
    "deletedAt": null,
    "reactions": [],
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
- Images use **lazy loading** for better performance

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
    "type": "IMAGE",
    "content": "https://cloudinary.com/file-url.pdf",
    "meta": {
      "imageUrl": "https://cloudinary.com/file-url.pdf",
      "publicId": "chat/files/123/user_id_timestamp",
      "resource_type": "raw",
      "fileName": "document.pdf",
      "fileSize": 1048576,
      "size": 1048576,
      "caption": "Here's the document"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "readAt": null,
    "editedAt": null,
    "deletedAt": null,
    "reactions": [],
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
      "editedAt": null,
      "deletedAt": null,
      "reactions": [
        {
          "id": "1",
          "emoji": "👍",
          "profileId": "user-id",
          "profile": {
            "id": "user-id",
            "displayName": "User Name",
            "avatarUrl": "https://..."
          },
          "createdAt": "2024-01-01T00:00:00Z"
        }
      ],
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
- **Deleted messages are excluded** from results
- Messages include **all reactions** with profile info
- Frontend automatically groups messages by date and sender

---

### Mark Messages as Read

Mark messages as read in a conversation.

**Endpoint:** `PATCH /api/messages/:conversationId/read`

**Request Body:**
```json
{
  "messageIds": ["789", "790"]
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
- Shows double checkmark (✓✓) on read messages

---

### Delete Message (NEW)

Delete a message (soft delete - message preserved but marked as deleted).

**Endpoint:** `DELETE /api/messages/:conversationId/:messageId`

**Response:**
```json
{
  "success": true,
  "message": "Message deleted successfully"
}
```

**Notes:**
- **Only the sender** can delete their own messages
- **Soft delete** - message is marked as deleted but preserved in database
- Content replaced with "This message was deleted"
- Automatically emits `message_deleted` event via Socket.io
- Deleted messages show "🚫 This message was deleted" to all users
- Cannot be undone - requires confirmation in UI

**Error Responses:**
- `403`: If trying to delete someone else's message
- `404`: If message not found or already deleted

---

### Add Reaction to Message (NEW)

Add an emoji reaction to a message.

**Endpoint:** `POST /api/messages/:conversationId/:messageId/reactions`

**Request Body:**
```json
{
  "emoji": "👍"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Reaction added successfully",
  "data": {
    "id": "1",
    "messageId": "789",
    "emoji": "👍",
    "profileId": "your-user-id",
    "profile": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    },
    "createdAt": "2024-01-01T00:00:00Z"
  }
}
```

**Notes:**
- Supports any emoji (1-10 characters)
- Quick emojis in UI: 👍 ❤️ 😂 😮 😢 🙏
- **One emoji per user per message** - duplicate reactions update existing
- Automatically emits `message_reaction` event via Socket.io
- Reactions grouped by emoji with counts
- Click your own reaction to remove it
- Hover reactions to see who reacted

---

### Remove Reaction from Message (NEW)

Remove your emoji reaction from a message.

**Endpoint:** `DELETE /api/messages/:conversationId/:messageId/reactions/:reactionId`

**Response:**
```json
{
  "success": true,
  "message": "Reaction removed successfully"
}
```

**Notes:**
- **Only the reaction owner** can remove their reaction
- Automatically emits `message_reaction_removed` event via Socket.io
- In UI, simply click your highlighted reaction to remove

**Error Responses:**
- `403`: If trying to remove someone else's reaction
- `404`: If reaction not found

---

## 🎨 UI Features

### Message Grouping
- Messages **automatically grouped** by sender and time
- Groups created for messages within **5 minutes** by same sender
- **Date dividers** separate messages by day ("Today", "Yesterday", etc.)
- Reduces visual clutter and improves readability

### Optimistic UI
- Messages appear **instantly** when sent
- Shows **loading state** while server confirms
- Replaced with real message once confirmed
- Provides responsive, instant feedback

### Performance Optimizations
- **React.memo** prevents unnecessary re-renders
- **Smart scrolling** preserves position when loading history
- **Lazy loading** for images
- **Debounced** typing indicators
- **50-70% reduction** in component re-renders

### Typing & Recording Indicators
- **Typing:** Shows "**User is typing...**" when `isTyping: true`
- **Recording:** Shows "**User is recording...**" when `isRecording: true`
- **Multiple users:** Shows "**3 people are typing...**" or "**2 people are recording...**"
- **Combined event:** Single `typing` event handles both states
- **Auto-timeout:** Recommended 3 seconds of inactivity
- **Real-time updates** via Socket.io
- **Flexible:** Can show both indicators simultaneously or separately
- **Multi-view support:** Works both **inside conversations** and in **chat list** (NEW!)
  - Inside chat: Shows indicator at bottom of messages
  - In chat list: Shows indicator next to conversation name
  - Requires `participantIds` parameter for chat list updates

### Last Message Updates (NEW)
- **Real-time conversation list updates** when new messages arrive
- Automatically updates **last message preview** in conversation list
- Shows **unread message count** for each conversation
- Updates even when user is **outside the active chat**
- Enables **live conversation list** without polling

---

## 🧪 Testing Guide

### Postman Setup

1. **Authentication:**
   - Get token from login endpoint: `POST /api/auth/login`
   - Set as header: `authorization: <token>` (lowercase, no "Bearer " prefix)
   - Example: `authorization: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...`

2. **Base URLs:**
   - **Local Development:** `http://localhost:3000`
   - **Production:** `https://done-buddy-production.up.railway.app`

3. **Socket.io Connection (JavaScript):**
   ```javascript
   import io from "socket.io-client";
   
   // Use environment-specific URL
   const SOCKET_URL = process.env.NODE_ENV === 'production' 
     ? "https://done-buddy-production.up.railway.app"
     : "http://localhost:3000";
   
   const socket = io(SOCKET_URL, {
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
   
   // Listen for reactions
   socket.on("message_reaction", (data) => {
     console.log("Reaction added:", data);
   });
   
   // Listen for deletions
   socket.on("message_deleted", (data) => {
     console.log("Message deleted:", data);
   });
   
   // Listen for typing/recording indicators
   socket.on("typing", (data) => {
     if (data.isTyping) {
       console.log("User is typing...");
     }
     if (data.isRecording) {
       console.log("User is recording...");
     }
   });
   
   // Listen for last message updates
   socket.on("last_message_update", (data) => {
     console.log("Last message updated:", data);
     // Update conversation list UI
   });
   ```

### Testing Flow

#### Direct Chat Flow

1. **Create/Get Conversation:**
   ```
   POST /api/conversations
   Body: { "participantId": "other-user-uuid" }
   ```

2. **Send Text Message:**
   ```
   POST /api/messages/{conversationId}/text
   Body: { "content": "Hello!", "type": "TEXT" }
   ```

3. **Add Reaction:**
   ```
   POST /api/messages/{conversationId}/{messageId}/reactions
   Body: { "emoji": "👍" }
   ```

4. **Remove Reaction:**
   ```
   DELETE /api/messages/{conversationId}/{messageId}/reactions/{reactionId}
   ```

5. **Delete Message:**
   ```
   DELETE /api/messages/{conversationId}/{messageId}
   ```

6. **Send Image Message:**
   ```
   POST /api/messages/{conversationId}/image
   Form-data: 
     - image: [file]
     - caption: "Optional caption" (optional)
   ```

7. **Get Messages:**
   ```
   GET /api/messages/{conversationId}?page=1&size=50
   GET /api/messages/{conversationId}?page=1&size=50&before=message-id
   ```

8. **Mark as Read:**
   ```
   PATCH /api/messages/{conversationId}/read
   Body: { "messageIds": ["msg-id-1", "msg-id-2"] }
   ```

#### Group Chat Flow (NEW)

1. **Create Group:**
   ```
   POST /api/conversations/groups
   Body: {
     "name": "Test Group",
     "participantIds": ["uuid-1", "uuid-2"],
     "description": "Testing group chat"
   }
   ```

2. **Send Message to Group:**
   ```
   POST /api/messages/{groupId}/text
   Body: { "content": "Hello group!" }
   ```

3. **Update Group Info (Admin):**
   ```
   PATCH /api/conversations/{groupId}/groups
   Body: { "name": "Updated Group Name" }
   ```

4. **Add Member (Admin):**
   ```
   POST /api/conversations/{groupId}/participants
   Body: { "participantIds": ["uuid-3"] }
   ```

5. **Promote to Admin:**
   ```
   PATCH /api/conversations/{groupId}/participants/{userId}/promote
   ```

6. **Remove Member (Admin):**
   ```
   DELETE /api/conversations/{groupId}/participants/{memberId}
   ```

7. **Leave Group:**
   ```
   DELETE /api/conversations/{groupId}/participants/{yourUserId}
   ```

---

## 📝 Implementation Notes

### Conversation Types
- **DIRECT**: One-on-one conversation between two users (friends only)
- **GROUP**: Multi-participant group chat with admin/member roles
- **CO_SHOPPING**: Reserved for future co-shopping feature (not implemented)

### Message Types
- **TEXT**: Plain text messages
- **IMAGE**: Image files (with optional caption)
- **FILE**: Document files (PDF, DOC, DOCX, ZIP, RAR)

### Message States
- **Normal**: Active message visible to all
- **Deleted**: Soft deleted, shows "This message was deleted"
- **Read**: Message has been read (shows ✓✓)
- **Unread**: Message not yet read (shows ✓)

### Reactions
- **Unique per user**: One emoji reaction per message per user
- **Grouped display**: Same emojis grouped with count
- **Real-time**: Instant updates via Socket.io
- **Profile info**: Shows who reacted on hover

### File Storage
- **Images**: Stored in Cloudinary under `chat/images/{conversationId}/`
- **Files**: Stored in Cloudinary under `chat/files/{conversationId}/`
- **File Size Limits**:
  - Images: 10MB max
  - Files: 15MB max

### Socket.io Behavior
- Messages emitted to **all participants except sender**
- Reactions broadcast to **all participants**
- Deletions broadcast to **all participants**
- Read receipts emitted when messages marked as read
- Typing/recording indicators emitted to **all participants except sender**
- Typing/recording auto-timeout recommended (3 seconds)
- Last message updates sent to **all participants** with personalized unread counts

### Data Models
- **Conversation IDs**: BigInt (returned as strings)
- **Message IDs**: BigInt (returned as strings)
- **Reaction IDs**: BigInt (returned as strings)
- **Pagination**: Messages in reverse chronological order
- **Soft Deletes**: deletedAt timestamp instead of hard delete

### Performance Features
- **Component memoization**: React.memo for Message components
- **Smart state updates**: Optimistic UI with server confirmation
- **Efficient queries**: Reactions loaded with messages in single query
- **Lazy loading**: Images load only when visible
- **Debounced events**: Typing indicators throttled

### Validation
- Conversation creation requires users to be friends
- File types strictly validated
- File sizes enforced (10MB images, 15MB files)
- Pagination validated (page ≥ 1, size 1-50)
- Only message sender can delete their messages
- Only reaction owner can remove their reaction

---

## 🔗 Related Documentation

- [Chat Improvements Summary](../docs/CHAT_IMPROVEMENTS_SUMMARY.md) - Technical details of recent upgrades
- [Chat Features Quick Reference](../docs/CHAT_FEATURES_QUICK_REFERENCE.md) - Developer quick reference
- [Authentication](./auth.md) - How to get authentication token
- [Profiles](./profiles.md) - User profile management
- [Social Features](./social.md) - Social features overview (friends, etc.)

---

## 🐛 Error Codes

- `400`: Bad Request (invalid input, file too large, etc.)
- `403`: Forbidden (not a participant, insufficient permissions, not message owner)
- `404`: Not Found (conversation/message/reaction not found)
- `500`: Internal Server Error (upload failures, database errors)

### Common Error Messages

**Conversations:**
- `"You can only create conversations with friends"`
- `"You are not a participant in this conversation"`
- `"Conversation not found"`

**Messages:**
- `"Message content cannot be empty"`
- `"Message not found"`
- `"You can only delete your own messages"`

**Reactions:**
- `"Emoji is required"`
- `"You can only remove your own reactions"`
- `"Reaction not found"`

---

## 🚀 Recent Updates (v3.2)

### Latest Features (v3.2)
- ✅ **Recording Indicator** - Show when users are recording voice messages
- ✅ **Last Message Updates** - Real-time conversation list updates with unread counts
- ✅ **Live Conversation List** - Updates automatically without polling

### Previous Features (v2.0-3.1)
- ✅ **Message Reactions** - Add emoji reactions to any message
- ✅ **Delete Messages** - Soft delete your own messages
- ✅ **Message Grouping** - Automatic grouping by date/sender
- ✅ **Optimistic UI** - Instant feedback when sending
- ✅ **Performance Boost** - 50-70% fewer re-renders

### Group Chat Features
- ✅ **Create Groups** - Create group chats with multiple friends
- ✅ **Group Management** - Edit name, description, avatar (admin only)
- ✅ **Add Members** - Invite more friends to groups (admin only)
- ✅ **Remove Members** - Remove members or leave group
- ✅ **Promote to Admin** - Promote trusted members to admin
- ✅ **Role-Based Permissions** - Admin vs Member roles

### Group Chat Roles
- **Admin**: Can edit group, add/remove members, promote members
- **Member**: Can chat, view info, and leave group

### Breaking Changes
- None - All changes are backwards compatible
- Existing direct conversations work as before
- New fields (reactions, deletedAt, group fields) are optional
- Frontend supports both direct and group chats seamlessly

---

## 🎯 Group Chat Quick Start

### 1. Create a Group
```bash
POST /api/conversations/groups
{
  "name": "My Group",
  "participantIds": ["uuid-1", "uuid-2"],
  "description": "Optional description"
}
```

### 2. Send Message to Group
```bash
POST /api/messages/:groupId/text
{
  "content": "Hello everyone!"
}
```

### 3. Manage Group (Admin Only)
```bash
# Update group info
PATCH /api/conversations/:groupId/groups
{ "name": "New Name" }

# Add members
POST /api/conversations/:groupId/participants
{ "participantIds": ["uuid-3"] }

# Promote to admin
PATCH /api/conversations/:groupId/participants/:userId/promote

# Remove member
DELETE /api/conversations/:groupId/participants/:userId
```

### 4. Leave Group
```bash
# Remove yourself
DELETE /api/conversations/:groupId/participants/:yourUserId
```

---

## 🌐 Production Deployment

### Environment Configuration

The chat system is deployed on **Railway** at:
- **Production API:** `https://done-buddy-production.up.railway.app`
- **Socket.io:** Same URL as API (automatic upgrade)

### Client Configuration

**React/Vite Setup:**
```javascript
// config.js
export const API_BASE_URL = import.meta.env.VITE_API_URL || 'https://done-buddy-production.up.railway.app';
export const SOCKET_URL = import.meta.env.VITE_SOCKET_URL || 'https://done-buddy-production.up.railway.app';
```

**Environment Variables (.env):**
```env
# Production (default)
VITE_API_URL=https://done-buddy-production.up.railway.app
VITE_SOCKET_URL=https://done-buddy-production.up.railway.app

# Local Development (override)
# VITE_API_URL=http://localhost:3000
# VITE_SOCKET_URL=http://localhost:3000
```

### Authentication Headers

**Important:** The backend accepts tokens **without** the "Bearer " prefix:

```javascript
// ✅ Correct
api.defaults.headers.common['authorization'] = token;

// ❌ Old way (no longer required)
api.defaults.headers.common['Authorization'] = `Bearer ${token}`;
```

### Global Error Handling

Implement automatic token refresh on auth errors:

```javascript
api.interceptors.response.use(
  (response) => response,
  (error) => {
    // Handle expired/invalid tokens
    if (
      error.response?.status === 401 ||
      error.response?.data?.error?.name === 'JsonWebTokenError'
    ) {
      localStorage.removeItem('token');
      localStorage.removeItem('user');
      window.location.reload();
    }
    return Promise.reject(error);
  }
);
```

### CORS Configuration

The backend has CORS enabled for all origins. No additional configuration needed for localhost development.

---

---

## 📝 Socket Events Summary

### Client → Server Events
| Event | Purpose | Data |
|-------|---------|------|
| `join_conversation` | Join a conversation room | `{ conversationId, userId, participantIds }` |
| `leave_conversation` | Leave a conversation room | `{ conversationId, userId }` |
| `typing` | Indicate typing or recording | `{ conversationId, userId, isTyping, isRecording }` |

### Server → Client Events
| Event | Purpose | Data |
|-------|---------|------|
| `new_message` | New message received | `{ conversationId, message }` |
| `messages_read` | Messages marked as read | `{ conversationId, readBy, messageIds }` |
| `typing` | User typing/recording status | `{ conversationId, userId, isTyping, isRecording }` |
| `message_deleted` | Message was deleted | `{ conversationId, messageId }` |
| `message_reaction_added` | Reaction added | `{ conversationId, messageId, reaction }` |
| `message_reaction_removed` | Reaction removed | `{ conversationId, messageId, reactionId, profileId }` |
| `user_online` | User came online | `{ userId, conversationId }` |
| `user_offline` | User went offline | `{ userId, conversationId }` |
| `last_message_update` | Last message & unread count | `{ conversationId, lastMessage, unreadCount }` |
| `conversation_created` | New conversation created (NEW) | `{ conversation }` |
| `conversation_updated` | Conversation details changed (NEW) | `{ conversation }` |
| `conversation_deleted` | Conversation deleted (NEW) | `{ conversationId }` |
| `participant_added` | Member added to group (NEW) | `{ conversationId, participant }` |
| `participant_removed` | Member removed from group (NEW) | `{ conversationId, participantId }` |

---

**Version:** 3.2.0  
**Last Updated:** January 16, 2026  
**Production URL:** https://done-buddy-production.up.railway.app
