# Chat API Documentation

Real-time chat functionality between users using Socket.io for instant messaging.

## 📋 Overview

The chat system supports:
- **Direct conversations** between two users (friends only)
- **Group chats** with multiple participants
- **Group management** with admin/member roles
- **Text messages** with @mentions support
- **Image messages** (uploaded to Cloudinary) with optional captions
- **File messages** (PDF, DOC, DOCX, ZIP, RAR) with optional captions
- **Voice messages** (audio recordings) with playback controls ⭐ NEW
- **Message reactions** with emoji (👍❤️😂😮😢🙏 and more)
- **Reply/Quote messages** - Reply to previous messages with quote preview ⭐ NEW
- **Edit messages** - Edit sent messages with history tracking ⭐ NEW
- **Delete messages** (soft delete - sender only)
- **Message search** - Full-text search across conversations ⭐ NEW
- **Real-time messaging** via Socket.io
- **Read receipts** with checkmarks
- **Typing indicators** with multi-user support (continuous while typing)
- **Recording indicators** for voice messages (continuous while recording)
- **Online/offline status** with green indicators
- **Message pagination** with infinite scroll
- **Message grouping** by date and sender
- **Optimistic UI** for instant feedback
- **Real-time chat list updates** with unread counts
- **Mobile responsive design** with slide navigation

## 🆕 Recent Updates (v4.0.0)

### Latest Features (January 2026) ⭐ MAJOR UPDATE
- ✅ **Reply/Quote Messages** - Reply to messages with quote preview
- ✅ **Edit Messages** - Edit sent messages with full history tracking
- ✅ **Voice Messages** - Record and send audio messages with playback
- ✅ **@Mentions** - Mention users in messages with autocomplete
- ✅ **Message Search** - Full-text search across all conversations
- ✅ **Recording Indicator** - Shows "🎙️ Recording..." when users record voice messages
- ✅ **Continuous Indicators** - Typing/recording indicators stay visible while active
- ✅ **Mobile Responsive** - Full mobile support with slide navigation and back button
- ✅ **Enhanced Logging** - Comprehensive debugging logs for troubleshooting
- ✅ **Professional Socket.IO** - Security, rate limiting, validation, error handling
- ✅ **Better Performance** - Optimized event emission (throttled to 400ms)
- ✅ **Auto-cleanup** - Stale indicators removed after 4 seconds

### Previous Features (v3.0-3.5)
- ✅ **Message Reactions** - Add emoji reactions to any message
- ✅ **Delete Messages** - Soft delete your own messages
- ✅ **Message Grouping** - Automatic grouping by date/sender
- ✅ **Optimistic UI** - Instant feedback when sending
- ✅ **Performance Boost** - 50-70% fewer re-renders
- ✅ **Group Chat Management** - Create, edit, add/remove members
- ✅ **Real-time Chat List** - Updates without polling

## 🔐 Authentication

All endpoints require authentication with token in the authorization header:
```
authorization: <your-token>
```

**Note:** The backend accepts tokens **without** the "Bearer " prefix. Simply send the token directly in the `authorization` header (lowercase).

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
  participantIds: ["user-id-1", "user-id-2"] // Optional but recommended
});

// Recording indicator (voice message)
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: false,
  isRecording: true,
  participantIds: ["user-id-1", "user-id-2"] // Optional but recommended
});

// Both can be sent together (edge case)
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: true,
  isRecording: true,
  participantIds: ["user-id-1", "user-id-2"]
});

// Stop all indicators
socket.emit("typing", {
  conversationId: "123",
  userId: "your-user-id",
  isTyping: false,
  isRecording: false,
  participantIds: ["user-id-1", "user-id-2"]
});
```

**Important Notes:**
- **Continuous Emission:** Events are emitted **every 400ms** while typing/recording to keep indicator alive
- **Auto-timeout:** Indicators auto-disappear after **3-4 seconds** of inactivity
- **ParticipantIds:** Optional but **highly recommended** for chat list indicators
  - ✅ With participantIds: Indicators appear in **chat list** (outside conversation)
  - ❌ Without participantIds: Indicators only appear **inside active chat**
- **Backend Fallback:** If participantIds not provided, backend fetches from database (slower)

**Behavior:**
```
User types → Emits every 400ms → Indicator stays visible
User stops → Wait 3 seconds → Indicator disappears
User continues → Indicator immediately reappears
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
  //   isTyping: true,        // User is typing text
  //   isRecording: false     // User is NOT recording voice
  // }
  
  // Handle different states
  if (data.isTyping) {
    // Show "User is typing..." indicator
    // In active chat: show at bottom of messages
    // In chat list: show next to conversation (💬 Typing...)
  }
  if (data.isRecording) {
    // Show "User is recording..." indicator
    // In active chat: show at bottom of messages
    // In chat list: show "🎙️ Recording..." next to conversation
  }
  
  // Note: Indicators auto-disappear after 3-4 seconds if no new events received
});
```javascript
socket.on("typing", (data) => {
  console.log("Typing indicator:", data);
  // {
  //   conversationId: "123",
  //   userId: "user-uuid-456",
  //   userName: "John Doe",  // ✨ NEW: User's display name (for group chats)
  //   isTyping: true,
  //   isRecording: false
  // }
});
```

**Usage in Different Views:**

**Inside Active Chat:**
```javascript
// Show indicator at bottom of chat with user's name in groups
if (data.conversationId === currentConversationId) {
  const displayName = data.userName || 'Someone';
  
  if (data.isTyping) {
    showTypingIndicator(`${displayName} is typing...`);
  }
  if (data.isRecording) {
    showRecordingIndicator(`${displayName} is recording...`);
  }
  
  // Auto-clear after 4 seconds (frontend cleanup)
  setTimeout(() => clearIndicator(), 4000);
}
```

**In Chat List (Outside Conversation):**
```javascript
// Show indicator next to conversation in list
socket.on("typing", (data) => {
  updateConversationIndicator(data.conversationId, {
    isTyping: data.isTyping,
    isRecording: data.isRecording,
    userId: data.userId,
    userName: data.userName,  // ✨ Store userName for display
    timestamp: Date.now() // Track when received
  });
  
  // Display in chat list:
  // - Group chat: "💬 John is typing..."
  // - 1-on-1 chat: "💬 Typing..."
  // - Multiple users: "John and Jane are typing..." or "3 people are typing..."
  
  // Frontend cleanup: Remove indicators older than 4 seconds
  setInterval(() => {
    cleanupStaleIndicators();
  }, 1000);
});
```

**Important Notes:**
- Events are throttled to **max 1 per 400ms** (prevent spam)
- Backend emits to **2 places**:
  1. Conversation room (for users inside chat)
  2. Direct to participants (for chat list)
- Frontend should cleanup stale indicators (>4 seconds old)
- Multiple users can be typing/recording simultaneously
- **`userName` is included** to show who is typing in group chats
- **Display logic:**
  - **Group chats**: Show user's name ("John is typing...")
  - **1-on-1 chats**: Just show action ("Typing...")
  - **Multiple users (2-3)**: Show names ("John and Jane are typing...")
  - **Many users (4+)**: Show count ("5 people are typing...")

#### 4. Message Deleted
```javascript
socket.on("message_deleted", (data) => {
  console.log("Message deleted:", data);
  // {
  //   conversationId: "123",
  //   messageId: "456"
  // }
});
```

#### 5. Message Edited (NEW) ⭐
```javascript
socket.on("message_edited", (data) => {
  console.log("Message edited:", data);
  // {
  //   conversationId: "123",
  //   message: {
  //     id: "456",
  //     content: "Updated message content",
  //     editedAt: "2024-01-01T00:10:00Z",
  //     originalContent: "Original message content"
  //   }
  // }
  
  // Update message in UI
  updateMessageInList(data.conversationId, data.message);
});
```

**Note:** Emitted to all participants when a message is edited. The `editedAt` timestamp indicates when the edit occurred. The `originalContent` field preserves the first version of the message.

#### 6. Message Reaction Added
```javascript
socket.on("message_reaction_added", (data) => {
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

#### 7. Message Reaction Removed (NEW)
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

#### 8. User Online
```javascript
socket.on("user_online", (data) => {
  console.log("User online:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 9. User Offline
```javascript
socket.on("user_offline", (data) => {
  console.log("User offline:", data);
  // {
  //   userId: "user-id",
  //   conversationId: "123"
  // }
});
```

#### 10. Last Message Update
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

#### 11. Conversation Created
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

#### 12. Conversation Updated
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

#### 13. Conversation Deleted
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

#### 14. Participant Added
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

#### 15. Participant Removed
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

#### 16. Socket Connection Events

The following events are automatically handled by Socket.IO:

```javascript
// Connection established
socket.on("connect", () => {
  console.log("Connected to Socket.IO server");
  console.log("Socket ID:", socket.id);
});

// Connection error
socket.on("connect_error", (error) => {
  console.error("Connection error:", error);
  // Retry logic is handled automatically by Socket.IO
});

// Disconnection
socket.on("disconnect", (reason) => {
  console.log("Disconnected:", reason);
  // Reasons: 
  // - "io server disconnect": Server forcefully disconnected
  // - "io client disconnect": Client called socket.disconnect()
  // - "ping timeout": Server didn't respond to ping in time
  // - "transport close": Underlying transport was closed
  // - "transport error": Transport error occurred
});

// Reconnection attempt
socket.on("reconnect_attempt", (attemptNumber) => {
  console.log("Reconnection attempt:", attemptNumber);
});

// Reconnected successfully
socket.on("reconnect", (attemptNumber) => {
  console.log("Reconnected after", attemptNumber, "attempts");
  // Re-join rooms after reconnection
  socket.emit("join_conversation", {
    conversationId: currentConversationId,
    userId: currentUserId,
    participantIds: participantIds
  });
});

// Custom error event
socket.on("error", (error) => {
  console.error("Socket error:", error);
});
```

**Important Connection Notes:**
- Socket.IO automatically handles reconnection with exponential backoff
- After reconnection, you must re-join all conversation rooms
- Connection state is managed per socket (multiple tabs = multiple connections)
- The `userId` query parameter links sockets to users in `userSocketMap`
- Each user can have multiple sockets (e.g., mobile + desktop)
- All user's sockets receive events when using `emitToUser()`

---

## 🏗️ Socket.IO Architecture

### Room Management

The chat system uses a sophisticated room-based architecture:

#### Room Types

1. **Conversation Rooms** (`conversation_{conversationId}`)
   - Primary room for each conversation
   - Used for all real-time message events
   - Format: `conversation_123`
   - Joined when: Client emits `join_conversation`
   - Purpose: Broadcast messages to all active participants

2. **Participant Rooms** (`userId_userId`)
   - Legacy support for direct chats
   - Format: `abc123_def456` (sorted alphabetically)
   - Automatically joined for 2-participant conversations
   - Purpose: Backward compatibility

3. **User Socket Mapping** (`userSocketMap`)
   - Maps each `userId` to a Set of `socketIds`
   - Supports multiple connections per user (mobile + desktop)
   - Used for direct user-to-user emissions
   - Format: `Map<userId, Set<socketId>>`

#### Event Emission Patterns

**Pattern 1: Room Broadcast (Excluding Sender)**
```javascript
// Used for: new_message, message_deleted, message_edited
emitToRoomExceptUser(conversationRoom, senderId, "new_message", data);
```
- Sends to all users in room EXCEPT the sender
- Prevents duplicate messages for sender (they get it from API response)

**Pattern 2: Direct User Emission**
```javascript
// Used for: last_message_update, conversation_created, typing (chat list)
emitToUser(userId, "last_message_update", data);
```
- Sends to ALL sockets of a specific user
- Ensures users outside active chat receive updates
- Enables chat list real-time updates

**Pattern 3: Room Broadcast (All Users)**
```javascript
// Used for: typing (inside chat), user_online, user_offline
io.to(conversationRoom).emit("typing", data);
```
- Sends to ALL users in room (including sender optional)
- Used for presence and indicators

**Pattern 4: Participant Iteration**
```javascript
// Used for: participant_added, participant_removed, conversation_updated
participantIds.forEach(pid => {
  emitToUser(pid, "participant_added", data);
});
```
- Explicitly sends to each participant
- Ensures delivery even if not in room
- Used for management operations

### Multi-Connection Support

**Key Feature:** Each user can have multiple socket connections simultaneously.

```javascript
// userSocketMap structure
{
  "user-id-1": Set(["socketId-a", "socketId-b", "socketId-c"]),
  "user-id-2": Set(["socketId-x"])
}
```

**Use Cases:**
- User opens chat on desktop browser
- User opens chat on mobile browser
- User opens multiple tabs
- All connections receive real-time updates

**Benefits:**
- Seamless multi-device experience
- No message loss when switching devices
- Consistent state across all clients

### Connection Lifecycle

```
┌─────────────────────────────────────────────────────────────┐
│ 1. CONNECT                                                   │
│    socket.connect({ query: { userId } })                    │
│    → Added to userSocketMap                                 │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│ 2. JOIN CONVERSATION                                         │
│    socket.emit("join_conversation", { conversationId, ... })│
│    → Joins conversation room                                │
│    → Emits user_online to others                           │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│ 3. ACTIVE COMMUNICATION                                      │
│    • Send/receive messages                                  │
│    • Emit typing indicators                                 │
│    • Receive real-time updates                             │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│ 4. LEAVE CONVERSATION (Optional)                            │
│    socket.emit("leave_conversation", { conversationId })    │
│    → Leaves conversation room                               │
└────────────────────────┬────────────────────────────────────┘
                         │
┌────────────────────────▼────────────────────────────────────┐
│ 5. DISCONNECT                                                │
│    socket.disconnect() or connection lost                   │
│    → Removed from userSocketMap                             │
│    → Emits user_offline if last socket                     │
└─────────────────────────────────────────────────────────────┘
```

### Security & Validation

**Rate Limiting:**
- Typing events throttled to max 1 per 400ms
- Prevents event spam and reduces server load

**Validation:**
- All socket events validate required fields
- Invalid data logged but doesn't crash server
- Malformed events silently ignored

**Authorization:**
- Socket connection requires valid `userId`
- Conversation participation verified before joining rooms
- Only participants receive conversation events

### Error Handling

**Graceful Degradation:**
- Database errors don't crash socket connection
- Missing data falls back to defaults
- Failed emissions logged but don't block other events

**Automatic Cleanup:**
- Stale typing indicators removed after 3-4 seconds
- Disconnected sockets automatically removed from maps
- Empty user entries cleaned from userSocketMap

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
- Now supports **@mentions** - include mentions array in request body (NEW!)

---

### Send Message with Reply/Quote (NEW) ⭐

Reply to a previous message with quote preview.

**Endpoint:** `POST /api/messages/:conversationId/reply`

**Request Body:**
```json
{
  "content": "Great idea! Let's do it.",
  "replyToId": "456",
  "mentions": [
    {
      "userId": "uuid-123",
      "displayName": "John",
      "offset": 0
    }
  ]
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
    "content": "Great idea! Let's do it.",
    "mentions": [
      {
        "userId": "uuid-123",
        "displayName": "John",
        "offset": 0
      }
    ],
    "replyTo": {
      "id": "456",
      "content": "Should we meet tomorrow?",
      "senderId": "uuid-456",
      "senderName": "Jane Doe"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- **replyToId** is optional - if not provided, sends as regular message
- **mentions** array is optional - for @mentioning users
- Reply message must exist in the same conversation
- Socket emits `new_message` with full reply data
- Frontend displays quoted message above reply

---

### Edit Message (NEW) ⭐

Edit a previously sent message with history tracking.

**Endpoint:** `PATCH /api/messages/:conversationId/:messageId/edit`

**Request Body:**
```json
{
  "content": "Corrected message text"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Message edited successfully",
  "data": {
    "id": "789",
    "conversationId": "123",
    "senderId": "your-user-id",
    "content": "Corrected message text",
    "editedAt": "2024-01-01T00:10:00Z",
    "originalContent": "Original message text",
    "editHistory": [
      {
        "content": "Original message text",
        "editedAt": "2024-01-01T00:00:00Z"
      }
    ],
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- **Only the sender** can edit their messages
- **⏱️ 15-minute limit**: Messages can only be edited within 15 minutes of sending
- Cannot edit **deleted messages**
- Original content is preserved in `originalContent`
- Full edit history tracked in `editHistory` array
- Socket emits `message_edited` event to all participants
- Frontend displays **(edited)** indicator

**Error Responses:**
- `403`: If trying to edit someone else's message
- `403`: If message is older than 15 minutes
- `400`: If trying to edit a deleted message

---

### Send Voice Message (NEW) ⭐

Send a voice/audio message recorded from the device.

**Endpoint:** `POST /api/messages/:conversationId/voice`

**Request:** Form-data (multipart/form-data)
- `voice`: Audio file (required) - Max 15MB
- `caption`: Optional text caption

**Supported Audio Formats:**
- WebM (MediaRecorder default)
- MP3, WAV, OGG (Cloudinary converts)

**Response:**
```json
{
  "success": true,
  "message": "Voice message sent successfully",
  "data": {
    "id": "790",
    "conversationId": "123",
    "senderId": "your-user-id",
    "type": "VOICE",
    "content": "Voice message",
    "voiceUrl": "https://cloudinary.com/voice-url.mp3",
    "voiceDuration": 15,
    "meta": {
      "voiceUrl": "https://cloudinary.com/voice-url.mp3",
      "publicId": "chat/voice/123/user_timestamp",
      "duration": 15,
      "resource_type": "video",
      "format": "mp3"
    },
    "createdAt": "2024-01-01T00:00:00Z",
    "sender": {
      "id": "your-user-id",
      "displayName": "Your Name",
      "avatarUrl": "https://..."
    }
  }
}
```

**Notes:**
- Voice files uploaded to Cloudinary under `chat/voice/{conversationId}/`
- Duration automatically extracted (in seconds)
- Caption is optional
- Max file size: **15MB**
- Frontend displays audio player with play/pause controls
- Works with browser MediaRecorder API

---

### Advanced Search (NEW) ⭐

Comprehensive search across messages, conversations, and groups with advanced filtering.

#### 1. Search Messages

Search for messages across all your conversations or within a specific conversation.

**Endpoint:** `GET /api/messages/search/messages`

**Query Parameters:**
- `query` (required): Search query string (2-200 chars)
- `conversationId` (optional): Limit search to specific conversation
- `type` (optional): Filter by message type (`TEXT`, `IMAGE`, `FILE`, `VOICE`)
- `senderId` (optional): Filter by sender user ID
- `dateFrom` (optional): Start date (ISO 8601 format)
- `dateTo` (optional): End date (ISO 8601 format)
- `hasReactions` (optional): Filter messages with reactions (true/false)
- `hasReplies` (optional): Filter messages that are replies (true/false)
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 50)

**Examples:**
```
# Basic search
GET /api/messages/search/messages?query=meeting

# Search in specific conversation
GET /api/messages/search/messages?query=meeting&conversationId=123

# Search by message type
GET /api/messages/search/messages?query=report&type=FILE

# Search with date range
GET /api/messages/search/messages?query=presentation&dateFrom=2024-01-01&dateTo=2024-01-31

# Search messages with reactions
GET /api/messages/search/messages?query=great&hasReactions=true

# Search replies only
GET /api/messages/search/messages?query=thanks&hasReplies=true

# Combined filters
GET /api/messages/search/messages?query=project&conversationId=123&type=TEXT&hasReactions=true
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 5,
  "pages": 1,
  "count": 5,
  "data": [
    {
      "id": "789",
      "conversationId": "123",
      "conversation": {
        "id": "123",
        "type": "GROUP",
        "name": "Project Team",
        "avatarUrl": "https://..."
      },
      "senderId": "uuid-456",
      "type": "TEXT",
      "content": "Meeting tomorrow at 3pm in conference room",
      "mentions": [],
      "replyTo": null,
      "editedAt": null,
      "createdAt": "2024-01-01T00:00:00Z",
      "sender": {
        "id": "uuid-456",
        "displayName": "Jane Doe",
        "avatarUrl": "https://..."
      },
      "reactions": [
        {
          "id": "1",
          "emoji": "👍",
          "profileId": "uuid-789"
        }
      ]
    }
  ]
}
```

**Features:**
- ✅ **Full-text search** using PostgreSQL
- ✅ **Case-insensitive** search with accent support
- ✅ **Filter by type** (text, images, files, voice)
- ✅ **Date range filtering**
- ✅ **Sender filtering**
- ✅ **Reaction filtering**
- ✅ **Reply filtering**
- ✅ **Conversation context** included
- ✅ **Only your conversations** (privacy protected)
- ✅ **Excludes deleted** messages
- ✅ **Paginated results**

---

#### 2. Search Conversations

Search for conversations and groups by name, participant, or content.

**Endpoint:** `GET /api/conversations/search`

**Query Parameters:**
- `query` (required): Search query string (2-200 chars)
- `type` (optional): Filter by type (`DIRECT`, `GROUP`, `CO_SHOPPING`)
- `hasUnread` (optional): Filter conversations with unread messages (true/false)
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 50)

**Examples:**
```
# Search all conversations
GET /api/conversations/search?query=project

# Search only groups
GET /api/conversations/search?query=team&type=GROUP

# Search conversations with unread messages
GET /api/conversations/search?query=john&hasUnread=true

# Search direct chats only
GET /api/conversations/search?query=sarah&type=DIRECT
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 3,
  "pages": 1,
  "count": 3,
  "data": [
    {
      "id": "123",
      "type": "GROUP",
      "name": "Project Team",
      "avatarUrl": "https://...",
      "description": "Main project discussion",
      "participantCount": 8,
      "unreadCount": 3,
      "lastMessage": {
        "id": "456",
        "content": "Last message preview",
        "type": "TEXT",
        "senderId": "uuid",
        "senderName": "John Doe",
        "createdAt": "2024-01-01T00:00:00Z"
      },
      "participants": [
        {
          "profileId": "uuid-1",
          "displayName": "John Doe",
          "avatarUrl": "https://...",
          "roleInChat": "admin"
        }
      ],
      "matchedOn": "name" // What field matched: name, description, participant
    },
    {
      "id": "124",
      "type": "DIRECT",
      "name": null,
      "participant": {
        "id": "uuid-2",
        "displayName": "Sarah Project Manager",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "unreadCount": 0,
      "lastMessage": {
        "id": "789",
        "content": "See you tomorrow",
        "type": "TEXT",
        "senderId": "uuid-2",
        "senderName": "Sarah Project Manager",
        "createdAt": "2024-01-01T00:00:00Z"
      },
      "matchedOn": "participant" // Matched on participant name
    }
  ]
}
```

**Search Criteria:**
- ✅ **Group names** - Search by group name
- ✅ **Group descriptions** - Search in descriptions
- ✅ **Participant names** - Find chats with specific people
- ✅ **Direct chat names** - Search by contact display name
- ✅ **Filter by type** - Groups, direct chats, or co-shopping
- ✅ **Unread filter** - Find conversations with unread messages
- ✅ **Match indicator** - Shows what field matched your query

---

#### 3. Search Groups Only

Search specifically for group chats with advanced filtering.

**Endpoint:** `GET /api/conversations/search/groups`

**Query Parameters:**
- `query` (required): Search query string (2-200 chars)
- `hasAdmin` (optional): Filter groups where you're admin (true/false)
- `minMembers` (optional): Minimum number of members
- `maxMembers` (optional): Maximum number of members
- `createdAfter` (optional): Created after date (ISO 8601)
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 25, max: 50)

**Examples:**
```
# Search all groups
GET /api/conversations/search/groups?query=team

# Search groups where you're admin
GET /api/conversations/search/groups?query=project&hasAdmin=true

# Search groups with 5+ members
GET /api/conversations/search/groups?query=study&minMembers=5

# Search recently created groups
GET /api/conversations/search/groups?query=new&createdAfter=2024-01-01

# Large groups only
GET /api/conversations/search/groups?query=company&minMembers=20&maxMembers=50
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 2,
  "pages": 1,
  "count": 2,
  "data": [
    {
      "id": "123",
      "type": "GROUP",
      "name": "Project Team Alpha",
      "avatarUrl": "https://...",
      "description": "Main project discussion and planning",
      "createdBy": "uuid-creator",
      "createdAt": "2024-01-01T00:00:00Z",
      "participantCount": 12,
      "adminCount": 2,
      "yourRole": "admin",
      "unreadCount": 5,
      "lastMessage": {
        "id": "456",
        "content": "Meeting at 3pm",
        "type": "TEXT",
        "senderId": "uuid",
        "senderName": "John Doe",
        "createdAt": "2024-01-17T00:00:00Z"
      },
      "participants": [
        {
          "profileId": "uuid-1",
          "displayName": "John Doe",
          "avatarUrl": "https://...",
          "roleInChat": "admin"
        }
      ]
    }
  ]
}
```

**Features:**
- ✅ **Group-specific search**
- ✅ **Admin filter** - Find groups you manage
- ✅ **Member count filter** - Small/large groups
- ✅ **Creation date filter** - Recent/old groups
- ✅ **Your role included** - See if you're admin/member
- ✅ **Admin count** - Number of admins
- ✅ **Full group details**

---

#### 4. Global Search (All Types)

Search across messages, conversations, and groups in a single request.

**Endpoint:** `GET /api/search/global`

**Query Parameters:**
- `query` (required): Search query string (2-200 chars)
- `searchIn` (optional): Comma-separated types to search (`messages`, `conversations`, `groups`)
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page per category (default: 10, max: 25)

**Examples:**
```
# Search everything
GET /api/search/global?query=project

# Search messages and groups only
GET /api/search/global?query=meeting&searchIn=messages,groups

# Search conversations only
GET /api/search/global?query=john&searchIn=conversations
```

**Response:**
```json
{
  "success": true,
  "query": "project",
  "results": {
    "messages": {
      "total": 25,
      "data": [
        {
          "id": "789",
          "conversationId": "123",
          "content": "Project deadline is tomorrow",
          "type": "TEXT",
          "createdAt": "2024-01-01T00:00:00Z",
          "sender": {
            "id": "uuid-456",
            "displayName": "Jane Doe",
            "avatarUrl": "https://..."
          },
          "conversation": {
            "id": "123",
            "type": "GROUP",
            "name": "Project Team"
          }
        }
      ]
    },
    "conversations": {
      "total": 5,
      "data": [
        {
          "id": "123",
          "type": "GROUP",
          "name": "Project Team Alpha",
          "participantCount": 12,
          "unreadCount": 3,
          "matchedOn": "name"
        },
        {
          "id": "124",
          "type": "DIRECT",
          "participant": {
            "displayName": "Project Manager Sarah",
            "avatarUrl": "https://..."
          },
          "matchedOn": "participant"
        }
      ]
    },
    "groups": {
      "total": 3,
      "data": [
        {
          "id": "123",
          "name": "Project Team Alpha",
          "description": "Main project discussion",
          "participantCount": 12,
          "yourRole": "admin",
          "unreadCount": 3
        }
      ]
    }
  }
}
```

**Features:**
- ✅ **Unified search** - One query, all results
- ✅ **Categorized results** - Organized by type
- ✅ **Totals per category** - Quick overview
- ✅ **Flexible filtering** - Choose what to search
- ✅ **Fast performance** - Parallel queries
- ✅ **Deduplicated** - Groups don't appear in both sections

---

### Search Best Practices

#### Frontend Implementation

```javascript
class AdvancedSearch {
  constructor(apiClient) {
    this.api = apiClient;
    this.debounceTimer = null;
  }

  // Debounced search (wait for user to stop typing)
  async search(query, options = {}) {
    return new Promise((resolve) => {
      clearTimeout(this.debounceTimer);
      this.debounceTimer = setTimeout(async () => {
        if (query.length < 2) {
          resolve({ results: [] });
          return;
        }

        try {
          const results = await this.globalSearch(query, options);
          resolve(results);
        } catch (error) {
          console.error("Search error:", error);
          resolve({ results: [] });
        }
      }, 300); // Wait 300ms after last keystroke
    });
  }

  // Global search
  async globalSearch(query, options = {}) {
    const params = new URLSearchParams({
      query,
      ...options
    });

    const response = await this.api.get(`/api/search/global?${params}`);
    return response.data;
  }

  // Message search with filters
  async searchMessages(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(`/api/messages/search/messages?${params}`);
    return response.data;
  }

  // Conversation search
  async searchConversations(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(`/api/conversations/search?${params}`);
    return response.data;
  }

  // Group search
  async searchGroups(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(`/api/conversations/search/groups?${params}`);
    return response.data;
  }
}

// Usage example
const search = new AdvancedSearch(apiClient);

// Search as user types
searchInput.addEventListener("input", async (e) => {
  const query = e.target.value;
  const results = await search.search(query, {
    searchIn: "messages,conversations,groups"
  });
  displayResults(results);
});

// Advanced message search with filters
const messageResults = await search.searchMessages("meeting", {
  conversationId: "123",
  type: "TEXT",
  dateFrom: "2024-01-01",
  hasReactions: true
});

// Search groups where you're admin
const adminGroups = await search.searchGroups("project", {
  hasAdmin: true,
  minMembers: 5
});
```

#### Search UI Components

```javascript
// Search Results Component
function SearchResults({ results }) {
  return (
    <div className="search-results">
      {/* Messages Section */}
      {results.messages?.total > 0 && (
        <section className="search-section">
          <h3>Messages ({results.messages.total})</h3>
          {results.messages.data.map(message => (
            <MessageResult 
              key={message.id}
              message={message}
              onSelect={() => jumpToMessage(message)}
            />
          ))}
        </section>
      )}

      {/* Conversations Section */}
      {results.conversations?.total > 0 && (
        <section className="search-section">
          <h3>Conversations ({results.conversations.total})</h3>
          {results.conversations.data.map(conv => (
            <ConversationResult
              key={conv.id}
              conversation={conv}
              onSelect={() => openConversation(conv.id)}
            />
          ))}
        </section>
      )}

      {/* Groups Section */}
      {results.groups?.total > 0 && (
        <section className="search-section">
          <h3>Groups ({results.groups.total})</h3>
          {results.groups.data.map(group => (
            <GroupResult
              key={group.id}
              group={group}
              onSelect={() => openGroup(group.id)}
            />
          ))}
        </section>
      )}
    </div>
  );
}
```

#### Advanced Filter UI

```javascript
function SearchFilters({ filters, onChange }) {
  return (
    <div className="search-filters">
      {/* Search Type */}
      <select 
        value={filters.searchIn} 
        onChange={(e) => onChange({ searchIn: e.target.value })}
      >
        <option value="messages,conversations,groups">All</option>
        <option value="messages">Messages Only</option>
        <option value="conversations">Conversations Only</option>
        <option value="groups">Groups Only</option>
      </select>

      {/* Message Type Filter (if searching messages) */}
      {filters.searchIn.includes("messages") && (
        <select 
          value={filters.type} 
          onChange={(e) => onChange({ type: e.target.value })}
        >
          <option value="">All Types</option>
          <option value="TEXT">Text</option>
          <option value="IMAGE">Images</option>
          <option value="FILE">Files</option>
          <option value="VOICE">Voice</option>
        </select>
      )}

      {/* Date Range Filter */}
      <input 
        type="date" 
        value={filters.dateFrom}
        onChange={(e) => onChange({ dateFrom: e.target.value })}
        placeholder="From Date"
      />
      <input 
        type="date" 
        value={filters.dateTo}
        onChange={(e) => onChange({ dateTo: e.target.value })}
        placeholder="To Date"
      />

      {/* Additional Filters */}
      <label>
        <input 
          type="checkbox" 
          checked={filters.hasReactions}
          onChange={(e) => onChange({ hasReactions: e.target.checked })}
        />
        Has Reactions
      </label>

      <label>
        <input 
          type="checkbox" 
          checked={filters.hasUnread}
          onChange={(e) => onChange({ hasUnread: e.target.checked })}
        />
        Unread Only
      </label>
    </div>
  );
}
```

### Search Performance Tips

1. **Debounce Input**: Wait 300ms after typing stops before searching
2. **Minimum Query Length**: Require at least 2 characters
3. **Limit Results**: Use pagination (max 50 per page)
4. **Cache Results**: Cache recent searches for 5 minutes
5. **Progressive Loading**: Load messages first, then conversations
6. **Index Optimization**: Backend uses PostgreSQL full-text search indexes
7. **Parallel Queries**: Global search runs queries in parallel

### Search Notes

- ✅ All searches respect **privacy** - only your conversations
- ✅ **Case-insensitive** and **accent-insensitive** search
- ✅ **Soft-deleted messages** excluded from results
- ✅ **Full-text indexes** for optimal performance
- ✅ **Pagination** for large result sets
- ✅ **Context included** - conversation info with messages
- ✅ **Match indicators** - shows why results matched

---

### Send Text Message (Updated)

**Note:** Text message endpoint now supports @mentions!

**Endpoint:** `POST /api/messages/:conversationId/text`

**Request Body:**
```json
{
  "content": "Hey @John, check this out!",
  "type": "TEXT",
  "mentions": [
    {
      "userId": "uuid-123",
      "displayName": "John",
      "offset": 4
    }
  ]
}
```

**Response:** (same as before, now includes `mentions` field)

---

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
      "mentions": [
        {
          "userId": "uuid-123",
          "displayName": "John",
          "offset": 4
        }
      ],
      "voiceUrl": null,
      "voiceDuration": null,
      "replyTo": {
        "id": "456",
        "content": "Original message",
        "senderId": "uuid-456",
        "senderName": "Jane Doe"
      },
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
- Messages now include **replyTo** data for quoted messages (NEW)
- Messages include **mentions** array for @mentions (NEW)
- Messages include **voiceUrl** and **voiceDuration** for voice messages (NEW)
- Messages include **editedAt** timestamp for edited messages (NEW)
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
- **⏱️ 15-minute limit**: Messages can only be deleted within 15 minutes of sending
- **Soft delete** - message is marked as deleted but preserved in database
- Content replaced with "This message was deleted"
- Automatically emits `message_deleted` event via Socket.io
- Deleted messages show "🚫 This message was deleted" to all users
- Cannot be undone - requires confirmation in UI

**Error Responses:**
- `403`: If trying to delete someone else's message
- `403`: If message is older than 15 minutes
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
- **Continuous:** Emits every **400ms** while active to keep indicator alive
- **Auto-timeout:** **3-4 seconds** of inactivity before disappearing
- **Real-time updates** via Socket.io
- **Flexible:** Can show both indicators simultaneously or separately
- **Multi-view support:** Works both **inside conversations** and in **chat list**
  - Inside chat: Shows indicator at bottom of messages
  - In chat list: Shows indicator next to conversation name
  - Requires `participantIds` parameter for chat list updates
- **Throttled:** Max 1 event per 400ms to prevent spam
- **Auto-cleanup:** Frontend removes stale indicators (>4 seconds old)

### Last Message Updates
- **Real-time conversation list updates** when new messages arrive
- Automatically updates **last message preview** in conversation list
- Shows **unread message count** for each conversation
- Updates even when user is **outside the active chat**
- Enables **live conversation list** without polling

### Reply/Quote Messages (NEW) ⭐
- **Quote Preview:** Shows quoted message above reply with blue accent bar
- **Sender Name:** Displayed in blue above quoted text
- **Truncated Preview:** Shows first 100 characters of quoted message
- **Cancel Button:** Remove quote before sending
- **Click to Reply:** Tap any message to quote and reply
- **Visual Design:** Light gray background with left border accent

### Edit Messages (NEW) ⭐
- **Edit Indicator:** Shows "(edited)" text in gray after edited messages
- **Edit History:** Tracks all versions of edited messages
- **Only Sender:** Only message sender can edit their own messages
- **Edit Button:** Pencil icon appears on hover for own messages
- **Cannot Edit Deleted:** Deleted messages cannot be edited
- **Real-time Updates:** All participants see edits instantly via Socket.io

### Voice Messages (NEW) ⭐
- **Voice Player:** Play/pause button with circular design
- **Progress Bar:** Visual waveform showing playback progress
- **Duration Display:** Shows current time / total duration (e.g., "0:15 / 0:30")
- **Recording Interface:** Press and hold microphone button
- **Recording Indicator:** Pulsing red dot while recording
- **Duration Counter:** Shows recording time while active
- **Auto-upload:** Automatically uploads when recording stops

### @Mentions (NEW) ⭐
- **Autocomplete Dropdown:** Appears when typing "@" symbol
- **User Suggestions:** Filters participants by display name
- **Avatar Display:** Shows user avatar in dropdown
- **Highlighted in Message:** Mentions appear in blue text
- **Clickable:** Mentions can link to user profile (future feature)
- **Real-time Suggestions:** Updates as you type

### Message Search (NEW) ⭐
- **Full-Text Search:** Searches across all conversation content
- **Conversation Filter:** Optionally limit search to one conversation
- **Results with Context:** Shows conversation name with each result
- **Pagination:** Load more results with pagination
- **Highlighted Matches:** Query terms highlighted in results (future)
- **Jump to Message:** Click result to view message in context (future)

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
   socket.on("message_reaction_added", (data) => {
     console.log("Reaction added:", data);
   });
   
   socket.on("message_reaction_removed", (data) => {
     console.log("Reaction removed:", data);
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

9. **Send Message with Reply:**
   ```
   POST /api/messages/{conversationId}/reply
   Body: {
     "content": "Great idea!",
     "replyToId": "456",
     "mentions": [{"userId": "uuid", "displayName": "John", "offset": 0}]
   }
   ```

10. **Edit Message:**
   ```
   PATCH /api/messages/{conversationId}/{messageId}/edit
   Body: { "content": "Updated message text" }
   ```

11. **Send Voice Message:**
   ```
   POST /api/messages/{conversationId}/voice
   Form-data:
     - voice: [audio file]
     - caption: "Voice message" (optional)
   ```

12. **Search Messages:**
   ```
   GET /api/messages/search/messages?query=keyword&conversationId=123&page=1
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
- **VOICE**: Audio/voice messages (WebM, MP3, WAV, OGG) ⭐ NEW

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
- **Voice Messages**: Stored in Cloudinary under `chat/voice/{conversationId}/` ⭐ NEW
- **File Size Limits**:
  - Images: 10MB max
  - Files: 15MB max
  - Voice: 15MB max ⭐ NEW

### Socket.io Behavior
- Messages emitted to **all participants except sender**
- Reactions broadcast to **all participants**
- Deletions broadcast to **all participants**
- **Edits broadcast to all participants** via `message_edited` event ⭐ NEW
- Read receipts emitted when messages marked as read
- Typing/recording indicators:
  - Emitted to **conversation room** (for users inside chat)
  - Emitted **directly to participants** (for chat list updates)
  - Throttled to **max 1 per 400ms** to prevent spam
  - Auto-timeout after **3-4 seconds** of inactivity
- Last message updates sent to **all participants** with personalized unread counts
- Connection state managed with `userSocketMap` (userId → Set of socketIds)
- Supports multiple connections per user (e.g., mobile + desktop)
- Automatic cleanup on disconnect
- Online/offline status broadcast to conversation participants

### Socket Event Names (IMPORTANT)
**⚠️ Correct Event Names to Use:**
- ✅ `message_reaction_added` - When a reaction is added
- ✅ `message_reaction_removed` - When a reaction is removed
- ❌ `message_reaction` - **NOT USED** (old/incorrect name)

**All Socket Events:**
```javascript
// Client → Server
- join_conversation
- leave_conversation  
- typing

// Server → Client
- new_message
- messages_read
- typing
- message_deleted
- message_edited
- message_reaction_added      // ✅ Use this
- message_reaction_removed
- user_online
- user_offline
- last_message_update
- conversation_created
- conversation_updated
- conversation_deleted
- participant_added
- participant_removed

// Connection Events (auto-handled)
- connect
- disconnect
- connect_error
- reconnect
- reconnect_attempt
- error
```

### Data Models
- **Conversation IDs**: BigInt (returned as strings)
- **Message IDs**: BigInt (returned as strings)
- **Reaction IDs**: BigInt (returned as strings)
- **Pagination**: Messages in reverse chronological order
- **Soft Deletes**: deletedAt timestamp instead of hard delete
- **Reply Relations**: Messages can reference other messages via replyToId ⭐ NEW
- **Edit History**: JSONB array tracking all message edits ⭐ NEW
- **Mentions**: JSONB array of mentioned users with offsets ⭐ NEW

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

## 🎯 Socket.IO Best Practices

### Frontend Implementation

#### 1. Connection Management

```javascript
import io from "socket.io-client";

class ChatSocket {
  constructor(userId, apiUrl) {
    this.userId = userId;
    this.socket = null;
    this.reconnectAttempts = 0;
    this.currentConversationId = null;
    this.participantIds = [];
  }

  connect() {
    this.socket = io(apiUrl, {
      query: { userId: this.userId },
      transports: ["websocket", "polling"],
      reconnection: true,
      reconnectionDelay: 1000,
      reconnectionDelayMax: 5000,
      reconnectionAttempts: 5
    });

    this.setupEventListeners();
  }

  setupEventListeners() {
    this.socket.on("connect", () => {
      console.log("✅ Connected:", this.socket.id);
      // Re-join conversation after reconnection
      if (this.currentConversationId) {
        this.joinConversation(this.currentConversationId, this.participantIds);
      }
    });

    this.socket.on("disconnect", (reason) => {
      console.log("❌ Disconnected:", reason);
    });

    this.socket.on("connect_error", (error) => {
      console.error("Connection error:", error);
    });
  }

  joinConversation(conversationId, participantIds) {
    this.currentConversationId = conversationId;
    this.participantIds = participantIds;
    
    this.socket.emit("join_conversation", {
      conversationId,
      userId: this.userId,
      participantIds
    });
  }

  leaveConversation(conversationId) {
    this.socket.emit("leave_conversation", {
      conversationId,
      userId: this.userId
    });
    this.currentConversationId = null;
    this.participantIds = [];
  }

  disconnect() {
    if (this.socket) {
      this.socket.disconnect();
    }
  }
}

// Usage
const chatSocket = new ChatSocket(currentUserId, SOCKET_URL);
chatSocket.connect();
```

#### 2. Typing Indicator (Throttled)

```javascript
import { debounce } from "lodash";

class TypingIndicator {
  constructor(socket, conversationId, userId, participantIds) {
    this.socket = socket;
    this.conversationId = conversationId;
    this.userId = userId;
    this.participantIds = participantIds;
    this.userName = getUserName(); // Get current user's display name
    
    // Throttle emissions to 400ms
    this.emitTyping = debounce(this.sendTypingEvent.bind(this), 400, {
      leading: true,
      trailing: true
    });
  }

  sendTypingEvent(isTyping, isRecording = false) {
    this.socket.emit("typing", {
      conversationId: this.conversationId,
      userId: this.userId,
      userName: this.userName, // ⭐ Important for group chats
      isTyping,
      isRecording,
      participantIds: this.participantIds // ⭐ Required for chat list indicators
    });
  }

  startTyping() {
    this.emitTyping(true, false);
  }

  stopTyping() {
    this.emitTyping(false, false);
  }

  startRecording() {
    this.emitTyping(false, true);
  }

  stopRecording() {
    this.emitTyping(false, false);
  }
}

// Usage in input component
const typingIndicator = new TypingIndicator(socket, conversationId, userId, participantIds);

// On input change
inputElement.addEventListener("input", () => {
  typingIndicator.startTyping();
});

// On input blur or send
inputElement.addEventListener("blur", () => {
  typingIndicator.stopTyping();
});
```

#### 3. Indicator Display (with Auto-Cleanup)

```javascript
class IndicatorManager {
  constructor() {
    this.indicators = new Map(); // conversationId → { users: Set, timestamp }
  }

  addIndicator(conversationId, userId, userName, type) {
    if (!this.indicators.has(conversationId)) {
      this.indicators.set(conversationId, { 
        typing: new Map(),
        recording: new Map()
      });
    }

    const conv = this.indicators.get(conversationId);
    const map = type === "typing" ? conv.typing : conv.recording;
    
    map.set(userId, {
      userName,
      timestamp: Date.now()
    });

    this.updateUI(conversationId);
  }

  removeIndicator(conversationId, userId, type) {
    const conv = this.indicators.get(conversationId);
    if (!conv) return;

    const map = type === "typing" ? conv.typing : conv.recording;
    map.delete(userId);

    this.updateUI(conversationId);
  }

  // Auto-cleanup stale indicators (>4 seconds old)
  cleanupStale() {
    const now = Date.now();
    const STALE_TIMEOUT = 4000;

    for (const [conversationId, conv] of this.indicators) {
      for (const [userId, data] of conv.typing) {
        if (now - data.timestamp > STALE_TIMEOUT) {
          conv.typing.delete(userId);
        }
      }
      for (const [userId, data] of conv.recording) {
        if (now - data.timestamp > STALE_TIMEOUT) {
          conv.recording.delete(userId);
        }
      }
      this.updateUI(conversationId);
    }
  }

  updateUI(conversationId) {
    const conv = this.indicators.get(conversationId);
    if (!conv) return;

    const typingUsers = Array.from(conv.typing.values());
    const recordingUsers = Array.from(conv.recording.values());

    // Update UI based on indicator counts
    if (typingUsers.length > 0) {
      this.showTypingIndicator(conversationId, typingUsers);
    } else {
      this.hideTypingIndicator(conversationId);
    }

    if (recordingUsers.length > 0) {
      this.showRecordingIndicator(conversationId, recordingUsers);
    } else {
      this.hideRecordingIndicator(conversationId);
    }
  }

  showTypingIndicator(conversationId, users) {
    const count = users.length;
    let text;

    if (count === 1) {
      text = `${users[0].userName} is typing...`;
    } else if (count === 2) {
      text = `${users[0].userName} and ${users[1].userName} are typing...`;
    } else {
      text = `${count} people are typing...`;
    }

    // Update DOM
    document.querySelector(`#typing-${conversationId}`).textContent = text;
  }

  showRecordingIndicator(conversationId, users) {
    const count = users.length;
    let text;

    if (count === 1) {
      text = `🎙️ ${users[0].userName} is recording...`;
    } else {
      text = `🎙️ ${count} people are recording...`;
    }

    // Update DOM
    document.querySelector(`#recording-${conversationId}`).textContent = text;
  }
}

// Initialize and auto-cleanup
const indicatorManager = new IndicatorManager();
setInterval(() => indicatorManager.cleanupStale(), 1000);

// Listen to socket events
socket.on("typing", (data) => {
  if (data.isTyping) {
    indicatorManager.addIndicator(
      data.conversationId,
      data.userId,
      data.userName,
      "typing"
    );
  } else {
    indicatorManager.removeIndicator(
      data.conversationId,
      data.userId,
      "typing"
    );
  }

  if (data.isRecording) {
    indicatorManager.addIndicator(
      data.conversationId,
      data.userId,
      data.userName,
      "recording"
    );
  } else {
    indicatorManager.removeIndicator(
      data.conversationId,
      data.userId,
      "recording"
    );
  }
});
```

#### 4. Message Handling (Optimistic UI)

```javascript
class MessageManager {
  constructor(socket, conversationId) {
    this.socket = socket;
    this.conversationId = conversationId;
    this.messages = [];
    this.optimisticMessages = new Map(); // tempId → message
  }

  async sendMessage(content, type = "TEXT") {
    // Generate temporary ID for optimistic UI
    const tempId = `temp-${Date.now()}`;
    
    // Add optimistic message to UI
    const optimisticMessage = {
      id: tempId,
      content,
      type,
      senderId: currentUserId,
      sender: currentUserProfile,
      createdAt: new Date().toISOString(),
      isOptimistic: true // Flag for UI styling
    };

    this.addOptimisticMessage(tempId, optimisticMessage);

    try {
      // Send to API
      const response = await fetch(`/api/messages/${this.conversationId}/text`, {
        method: "POST",
        headers: {
          "Content-Type": "application/json",
          "authorization": token
        },
        body: JSON.stringify({ content, type })
      });

      const result = await response.json();

      // Replace optimistic message with real one
      this.replaceOptimisticMessage(tempId, result.data);

    } catch (error) {
      console.error("Failed to send message:", error);
      // Mark optimistic message as failed
      this.markOptimisticMessageFailed(tempId);
    }
  }

  addOptimisticMessage(tempId, message) {
    this.optimisticMessages.set(tempId, message);
    this.messages.push(message);
    this.updateUI();
  }

  replaceOptimisticMessage(tempId, realMessage) {
    const index = this.messages.findIndex(m => m.id === tempId);
    if (index !== -1) {
      this.messages[index] = realMessage;
      this.optimisticMessages.delete(tempId);
      this.updateUI();
    }
  }

  markOptimisticMessageFailed(tempId) {
    const message = this.messages.find(m => m.id === tempId);
    if (message) {
      message.failed = true;
      this.updateUI();
    }
  }

  // Listen to socket for new messages from others
  onNewMessage(messageData) {
    // Don't add if it's from current user (already have via API)
    if (messageData.message.senderId === currentUserId) {
      return;
    }

    this.messages.push(messageData.message);
    this.updateUI();
  }
}

// Usage
const messageManager = new MessageManager(socket, conversationId);

socket.on("new_message", (data) => {
  messageManager.onNewMessage(data);
});
```

#### 5. Room Management

```javascript
class ConversationManager {
  constructor(socket, userId) {
    this.socket = socket;
    this.userId = userId;
    this.activeConversations = new Set();
  }

  joinConversation(conversationId, participantIds) {
    this.socket.emit("join_conversation", {
      conversationId,
      userId: this.userId,
      participantIds
    });

    this.activeConversations.add(conversationId);
    console.log(`Joined conversation: ${conversationId}`);
  }

  leaveConversation(conversationId) {
    this.socket.emit("leave_conversation", {
      conversationId,
      userId: this.userId
    });

    this.activeConversations.delete(conversationId);
    console.log(`Left conversation: ${conversationId}`);
  }

  // Clean up when navigating away
  cleanup() {
    for (const conversationId of this.activeConversations) {
      this.leaveConversation(conversationId);
    }
  }
}

// React example with useEffect
useEffect(() => {
  if (conversationId && participantIds) {
    conversationManager.joinConversation(conversationId, participantIds);
  }

  return () => {
    if (conversationId) {
      conversationManager.leaveConversation(conversationId);
    }
  };
}, [conversationId]);
```

### Common Pitfalls & Solutions

#### ❌ Pitfall 1: Not Providing `participantIds` in Typing Event
```javascript
// BAD - indicators won't appear in chat list
socket.emit("typing", {
  conversationId,
  userId,
  isTyping: true
});

// GOOD - indicators work everywhere
socket.emit("typing", {
  conversationId,
  userId,
  userName: currentUserName, // ⭐ Important for group chats
  isTyping: true,
  participantIds: allParticipantIds // ⭐ Required
});
```

#### ❌ Pitfall 2: Not Cleaning Up Stale Indicators
```javascript
// BAD - indicators never disappear
socket.on("typing", (data) => {
  if (data.isTyping) {
    showTypingIndicator(data.userId);
  }
});

// GOOD - auto-cleanup after 4 seconds
socket.on("typing", (data) => {
  if (data.isTyping) {
    showTypingIndicator(data.userId, Date.now());
  }
  // Run cleanup every second
  setInterval(() => cleanupStaleIndicators(), 1000);
});
```

#### ❌ Pitfall 3: Not Re-joining Rooms After Reconnection
```javascript
// BAD - lose connection to room after reconnect
socket.on("connect", () => {
  console.log("Connected");
});

// GOOD - automatically re-join
socket.on("reconnect", () => {
  console.log("Reconnected - re-joining rooms");
  if (currentConversationId) {
    socket.emit("join_conversation", {
      conversationId: currentConversationId,
      userId: currentUserId,
      participantIds: currentParticipantIds
    });
  }
});
```

#### ❌ Pitfall 4: Using Wrong Event Names
```javascript
// BAD - wrong event name (old/incorrect)
socket.on("message_reaction", (data) => {
  updateReaction(data);
});

// GOOD - correct event name
socket.on("message_reaction_added", (data) => {
  updateReaction(data);
});
```

#### ❌ Pitfall 5: Not Throttling Typing Events
```javascript
// BAD - emits on every keystroke (spam)
input.addEventListener("input", () => {
  socket.emit("typing", { isTyping: true });
});

// GOOD - throttled to 400ms
const throttledTyping = debounce(() => {
  socket.emit("typing", { isTyping: true });
}, 400);

input.addEventListener("input", throttledTyping);
```

### Performance Optimization

1. **Debounce Typing Events**: Throttle to 400ms maximum
2. **Memoize Components**: Use React.memo for message components
3. **Virtual Scrolling**: Implement for large message lists
4. **Lazy Loading**: Load images only when visible
5. **Batch Updates**: Group state updates together
6. **Cleanup Listeners**: Remove socket listeners on unmount

### Security Considerations

1. **Validate User IDs**: Always verify userId matches authenticated user
2. **Check Permissions**: Verify user is participant before emitting
3. **Sanitize Input**: Clean message content before sending
4. **Rate Limiting**: Backend throttles events to prevent spam
5. **Token Refresh**: Handle expired tokens gracefully

---

## 🔗 Related Documentation

- **[NEW! Chat Features Implementation](../docs/CHAT_FEATURES_README.md)** ⭐ 5 NEW FEATURES!
  - Reply/Quote Messages
  - Edit Messages
  - Voice Messages
  - @Mentions
  - Message Search
- [Complete Features Summary](../docs/CHAT_FEATURES_COMPLETE_SUMMARY.md) - Full implementation details
- [Backend Documentation](../docs/CHAT_FEATURES_BACKEND_COMPLETE.md) - API & Database
- [Frontend Guide](../docs/CHAT_FEATURES_FRONTEND_GUIDE.md) - React Components
- [Socket Improvements Guide](../docs/SOCKET_IMPROVEMENTS.md) - Professional Socket.IO enhancements
- [Typing Indicator Debugging](../chat-client/TYPING_INDICATOR_DEBUG.md) - Debug typing issues
- [Recording Feature Guide](../chat-client/RECORDING_FEATURE.md) - Recording indicator details
- [Mobile Improvements](../chat-client/MOBILE_IMPROVEMENTS.md) - Mobile responsive design
- [Chat Features Added](../chat-client/FEATURES_ADDED.md) - Implementation summary
- [Chat Improvements Summary](../docs/CHAT_IMPROVEMENTS_SUMMARY.md) - Technical details
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

## 🚀 Recent Updates (v3.5)

### Latest Features (v3.5)
- ✅ **Recording Indicator** - Show when users are recording voice messages (🎙️)
- ✅ **Continuous Indicators** - Typing/recording stay visible while active (emit every 400ms)
- ✅ **Mobile Responsive** - Full mobile support with slide navigation
- ✅ **Enhanced Debugging** - Comprehensive console logs for troubleshooting
- ✅ **Professional Socket.IO** - Security, rate limiting, validation improvements
- ✅ **Better Performance** - Optimized emission frequency and auto-cleanup
- ✅ **Last Message Updates** - Real-time conversation list updates with unread counts

### Previous Features (v2.0-3.4)
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
| Event | Purpose | Data | Notes |
|-------|---------|------|-------|
| `join_conversation` | Join a conversation room | `{ conversationId, userId, participantIds }` | Auto-emit online status |
| `leave_conversation` | Leave a conversation room | `{ conversationId, userId }` | - |
| `typing` | Indicate typing or recording | `{ conversationId, userId, userName, isTyping, isRecording, participantIds }` | Throttled to 400ms, continuous while active |

### Server → Client Events
| Event | Purpose | Data | Notes |
|-------|---------|------|-------|
| `new_message` | New message received | `{ conversationId, message }` | Excludes sender, includes reply, mentions, voice |
| `messages_read` | Messages marked as read | `{ conversationId, readBy, messageIds }` | - |
| `typing` | User typing/recording status | `{ conversationId, userId, userName, isTyping, isRecording }` | Emitted every 400ms while active |
| `message_deleted` | Message was deleted | `{ conversationId, messageId }` | - |
| `message_edited` | Message was edited | `{ conversationId, message: { id, content, editedAt, originalContent } }` | ⭐ NEW |
| `message_reaction_added` | Reaction added | `{ conversationId, messageId, reaction }` | Correct event name |
| `message_reaction_removed` | Reaction removed | `{ conversationId, messageId, reactionId, profileId }` | - |
| `user_online` | User came online | `{ userId, conversationId }` | Auto-emitted on join_conversation |
| `user_offline` | User went offline | `{ userId, conversationId }` | Auto-emitted on disconnect |
| `last_message_update` | Last message & unread count | `{ conversationId, lastMessage, unreadCount }` | Personalized per user |
| `conversation_created` | New conversation created | `{ conversation }` | - |
| `conversation_updated` | Conversation details changed | `{ conversation }` | Group name, avatar, etc. |
| `conversation_deleted` | Conversation deleted | `{ conversationId }` | - |
| `participant_added` | Member added to group | `{ conversationId, participant }` | - |
| `participant_removed` | Member removed from group | `{ conversationId, participantId }` | - |

### Connection Events (Auto-handled by Socket.IO)
| Event | Purpose | Notes |
|-------|---------|-------|
| `connect` | Connection established | Provides `socket.id` |
| `disconnect` | Connection lost | Provides disconnect reason |
| `connect_error` | Connection failed | Auto-retry enabled |
| `reconnect` | Reconnected successfully | Must re-join rooms |
| `reconnect_attempt` | Attempting reconnection | Shows attempt number |
| `error` | Custom error event | Application-level errors |

---

**Version:** 4.0.0  
**Last Updated:** January 17, 2026  
**Production URL:** https://done-buddy-production.up.railway.app
