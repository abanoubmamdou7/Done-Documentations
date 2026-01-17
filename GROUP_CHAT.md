# Group Chat Feature

## Overview
Added full group chat support with the ability to create groups, manage participants with admin/member roles, and real-time updates via Socket.IO.

## 📋 Features

- ✅ **Create Groups** - Create group chats with multiple friends
- ✅ **Admin Roles** - Group creators are admins with management permissions
- ✅ **Add/Remove Members** - Admins can manage participant list
- ✅ **Promote to Admin** - Promote trusted members to admin role
- ✅ **Group Details** - Update name, description, and avatar (admin only)
- ✅ **Real-time Updates** - All changes broadcast via Socket.IO
- ✅ **Leave Group** - Any member can leave at any time
- ✅ **Auto-delete** - Group deleted when last member leaves

## Database Changes

Run this SQL to add group chat fields (or use `npx prisma db push`):

\`\`\`sql
-- Add group chat fields to conversations table
ALTER TABLE "conversations" 
ADD COLUMN IF NOT EXISTS "name" VARCHAR(255),
ADD COLUMN IF NOT EXISTS "avatar_url" TEXT,
ADD COLUMN IF NOT EXISTS "description" TEXT;

-- Add index for conversation type
CREATE INDEX IF NOT EXISTS "conversations_type_idx" ON "conversations"("type");

-- Add index for conversation participants profile_id
CREATE INDEX IF NOT EXISTS "conversation_participants_profile_id_idx" ON "conversation_participants"("profile_id");
\`\`\`

Or simply run:
\`\`\`bash
npx prisma db push
\`\`\`

## Backend API Endpoints

### 1. **Create Group Conversation** ⭐ NEW
- **Endpoint:** `POST /api/conversations/groups`
- **Body:** 
  ```json
  {
    "name": "Study Group",
    "participantIds": ["uuid-1", "uuid-2", "uuid-3"],
    "avatarUrl": "https://...",
    "description": "A group for studying together"
  }
  ```
- **Response:**
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
- **Notes:**
  - Group name is **required** (1-255 characters)
  - At least **1 participant** required (besides creator)
  - All participants must be **friends** with creator
  - Creator automatically becomes **admin**
  - Emits `conversation_created` socket event to all participants

### 2. **Create Direct Conversation**
- **Endpoint:** `POST /api/conversations`
- **Body:** `{ "participantId": "uuid" }`
- Creates or returns existing conversation with a friend
- **Note:** For creating groups, use the `/groups` endpoint above

### 3. **Get All Conversations**
- **Endpoint:** `GET /api/conversations?page=1&size=25`
- Returns paginated list of user's conversations (both DIRECT and GROUP)
- Includes unread counts and last message for each

### 4. **Get Conversation by ID**
- **Endpoint:** `GET /api/conversations/:conversationId`
- Returns conversation details with participants and their roles

### 5. **Update Group Details** (Admin Only)
- **Endpoint:** `PATCH /api/conversations/:conversationId/groups`
- **Body:** 
  ```json
  {
    "name": "Updated Group Name",
    "description": "Updated description",
    "avatarUrl": "https://..."
  }
  ```
- All fields are optional (only update what you provide)
- **Emits:** `conversation_updated` socket event to all participants

### 6. **Add Participants to Group** (Admin Only)
- **Endpoint:** `POST /api/conversations/:conversationId/participants`
- **Body:** `{ "participantIds": ["uuid-4", "uuid-5"] }`
- New members must be friends with admin
- New members added as **members** (not admin)
- **Emits:** `participant_added` socket event for each new participant

### 7. **Remove Participant from Group** (Admin or Self)
- **Endpoint:** `DELETE /api/conversations/:conversationId/participants/:participantId`
- **Admins** can remove any member
- **Any member** can remove themselves (leave group)
- Cannot remove **last admin** (must promote someone first)
- If last member leaves, **group is deleted**
- **Emits:** `participant_removed` socket event to all participants

### 8. **Promote Member to Admin** (Admin Only)
- **Endpoint:** `PATCH /api/conversations/:conversationId/participants/:participantId/promote`
- Promotes a member to admin role
- Groups can have **multiple admins**
- **Emits:** Socket event notifying all participants

## 📡 Socket.IO Events for Groups

### Server → Client Events

Group chats use the same Socket.IO infrastructure as direct chats, with additional events for group management.

#### 1. **conversation_created**
Emitted when a new group is created or you're added to a group.

```javascript
socket.on("conversation_created", (data) => {
  console.log("New group created:", data);
  // {
  //   conversation: {
  //     id: "123",
  //     type: "GROUP",
  //     name: "Study Group",
  //     avatarUrl: "https://...",
  //     description: "A group for studying",
  //     createdBy: "creator-id",
  //     createdAt: "2024-01-01T00:00:00Z",
  //     participants: [...]
  //   }
  // }
  
  // Add to your conversation list
  addConversationToList(data.conversation);
});
```

#### 2. **conversation_updated**
Emitted when group details change (name, avatar, description).

```javascript
socket.on("conversation_updated", (data) => {
  console.log("Group updated:", data);
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
  
  // Update in conversation list
  updateConversationInList(data.conversation);
});
```

#### 3. **participant_added**
Emitted when a new member is added to the group.

```javascript
socket.on("participant_added", (data) => {
  console.log("Member added:", data);
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
  
  // Update participants list
  addParticipantToConversation(data.conversationId, data.participant);
});
```

#### 4. **participant_removed**
Emitted when someone is removed from the group or leaves.

```javascript
socket.on("participant_removed", (data) => {
  console.log("Member removed:", data);
  // {
  //   conversationId: "123",
  //   participantId: "user-id"
  // }
  
  // If it's you, remove the conversation
  if (data.participantId === currentUserId) {
    removeConversationFromList(data.conversationId);
  } else {
    // Otherwise, just update participants
    removeParticipantFromConversation(data.conversationId, data.participantId);
  }
});
```

#### 5. **conversation_deleted**
Emitted when the group is deleted (last member leaves).

```javascript
socket.on("conversation_deleted", (data) => {
  console.log("Group deleted:", data);
  // {
  //   conversationId: "123"
  // }
  
  // Remove from conversation list
  removeConversationFromList(data.conversationId);
});
```

#### 6. **typing** (with userName for groups)
Shows who is typing in group chats.

```javascript
socket.on("typing", (data) => {
  console.log("Typing in group:", data);
  // {
  //   conversationId: "123",
  //   userId: "user-id",
  //   userName: "John Doe",  // ⭐ Shows who is typing in groups
  //   isTyping: true,
  //   isRecording: false
  // }
  
  // Display: "John Doe is typing..." in group chats
  // Display: "Typing..." in direct chats
  if (data.isTyping) {
    const message = data.userName 
      ? `${data.userName} is typing...`
      : "Someone is typing...";
    showTypingIndicator(message);
  }
});
```

### Client → Server Events

#### Joining a Group Conversation
```javascript
// Join the group room
socket.emit("join_conversation", {
  conversationId: "123",
  userId: currentUserId,
  participantIds: allParticipantIds // All members in the group
});
```

#### Typing in a Group
```javascript
// Emit typing event with userName for group chats
socket.emit("typing", {
  conversationId: "123",
  userId: currentUserId,
  userName: currentUserDisplayName, // ⭐ Important for group chats
  isTyping: true,
  isRecording: false,
  participantIds: allParticipantIds // Required for chat list indicators
});
```

### Real-time Features in Groups

1. **Messages** - All members receive `new_message` events instantly
2. **Typing Indicators** - Shows "John is typing..." with user names
3. **Read Receipts** - Shows who has read messages
4. **Reactions** - All members see reactions in real-time
5. **Member Changes** - Instant updates when members join/leave
6. **Group Updates** - Name/avatar changes appear immediately
7. **Last Message** - Chat list updates with latest message and unread counts

### Multiple Typing Users in Groups

```javascript
// Track multiple users typing
const typingUsers = new Map(); // userId → { userName, timestamp }

socket.on("typing", (data) => {
  if (data.isTyping) {
    typingUsers.set(data.userId, {
      userName: data.userName,
      timestamp: Date.now()
    });
  } else {
    typingUsers.delete(data.userId);
  }
  
  updateTypingIndicator(typingUsers);
});

function updateTypingIndicator(typingUsers) {
  const users = Array.from(typingUsers.values());
  
  if (users.length === 0) {
    hideTypingIndicator();
  } else if (users.length === 1) {
    showTypingIndicator(`${users[0].userName} is typing...`);
  } else if (users.length === 2) {
    showTypingIndicator(`${users[0].userName} and ${users[1].userName} are typing...`);
  } else {
    showTypingIndicator(`${users.length} people are typing...`);
  }
}
```

---

## Frontend Components

### Required Components

1. **CreateGroupModal.jsx** ⭐ NEW
   - Modal to create a new group chat
   - Friend selection with search/filter
   - Group name input (required)
   - Group description input (optional)
   - Avatar URL input (optional)

2. **GroupSettingsModal.jsx** ⭐ NEW
   - Modal to manage group settings (admin only)
   - Update group name, description, avatar
   - View and manage members
   - Promote members to admin
   - Remove members (admin only)

3. **AddParticipantsModal.jsx**
   - Modal to add friends to a group
   - Shows friends list with selection
   - Filters out existing participants
   - Admin only for groups

### Updated Components

1. **ChatWindow.jsx**
   - Shows group name and member count for groups
   - Displays group avatar or initials
   - "Add Members" button (➕) for admins
   - "Group Settings" button (⚙️) for admins
   - Shows typing indicator with user names in groups

2. **ConversationList.jsx**
   - Displays group icon for group chats
   - Shows member count instead of online status
   - Displays last message sender name for groups

3. **MessageBubble.jsx**
   - Shows sender name above messages in groups
   - Different color for different senders
   - Your messages aligned right, others aligned left

4. **App.jsx**
   - Added `onConversationUpdated` callback
   - Handles group creation, updates, and deletions
   - Manages socket events for groups

5. **api.js**
   - `conversationAPI.createGroup()` ⭐ NEW
   - `conversationAPI.updateGroup()` ⭐ NEW
   - `conversationAPI.addParticipants()`
   - `conversationAPI.removeParticipant()`
   - `conversationAPI.promoteMember()` ⭐ NEW

## Features

### 1. **Create New Group**
   - Click "New Group" button in conversation list
   - Select 1 or more friends to add
   - Enter group name (required)
   - Add optional description and avatar URL
   - Creator automatically becomes admin
   - **Socket Event:** `conversation_created` emitted to all participants

### 2. **Add Members to Group** (Admin Only)
   - Click the ➕ button in group chat header
   - Select friends from your friends list (non-participants only)
   - Click "Add (N)" to add selected friends
   - New members receive `participant_added` event
   - New members can see full chat history
   - **Socket Event:** `participant_added` for each new member

### 3. **Group Management** (Admin Only)
   - Click ⚙️ settings button in group chat
   - **Update Details:**
     - Change group name
     - Update description
     - Change avatar URL
     - **Socket Event:** `conversation_updated` to all participants
   - **Manage Members:**
     - View all participants with roles
     - Remove members (except last admin)
     - Promote members to admin
     - **Socket Event:** `participant_removed` when removing

### 4. **Promote to Admin** (Admin Only)
   - Open group settings
   - Click "Promote to Admin" on a member
   - Member gains all admin permissions
   - Groups can have multiple admins
   - Cannot demote admins (not implemented)

### 5. **Leave Group** (Any Member)
   - Click "Leave Group" in settings
   - Confirmation required
   - You are removed from the group
   - Group disappears from your conversation list
   - **Socket Event:** `participant_removed` with your ID
   - **Special:** If last member leaves, group is deleted
     - **Socket Event:** `conversation_deleted` to remaining participants

### 6. **Group Chat Display**
   - Shows group name in header (not participant names)
   - Displays member count: "5 members"
   - Group avatar or generated initials
   - Sender names shown above each message
   - Different colors for different senders
   - Typing shows user names: "John is typing..."

### 7. **Real-time Updates**
   - All changes broadcast instantly via Socket.IO
   - New messages appear immediately
   - Member changes update participant list
   - Group details update in real-time
   - Typing indicators show who is typing
   - Last message updates in conversation list

---

## 🔐 Permissions Matrix

| Action | Direct Chat | Group (Admin) | Group (Member) |
|--------|-------------|---------------|----------------|
| Send Messages | ✅ Both | ✅ All | ✅ All |
| Add Participants | ✅ Both | ✅ Yes | ❌ No |
| Remove Participants | ❌ N/A | ✅ Yes | ❌ No |
| Leave Group | ❌ N/A | ✅ Yes | ✅ Yes |
| Update Group Details | ❌ N/A | ✅ Yes | ❌ No |
| Promote to Admin | ❌ N/A | ✅ Yes | ❌ No |
| Delete Group | ❌ N/A | ✅ Last member | ✅ Last member |
| View Messages | ✅ Both | ✅ All | ✅ All |
| React to Messages | ✅ Both | ✅ All | ✅ All |
| Delete Own Messages | ✅ Both | ✅ All | ✅ All |

### Special Rules

1. **Last Admin Protection**: Cannot remove the last admin from a group
   - Must promote another member to admin first
   - Prevents groups from having no admins

2. **Auto-delete**: Group is deleted when last member leaves
   - All participants receive `conversation_deleted` event
   - Messages are soft-deleted (preserved in database)

3. **Friend Requirement**: Can only add users who are your friends
   - Prevents spam and unwanted group additions
   - Maintains privacy and trust

4. **Creator Privilege**: Group creator automatically becomes admin
   - Cannot be changed
   - Can be demoted if feature implemented in future

---

## Schema Updates

\`\`\`prisma
model Conversation {
  id          BigInt           @id @default(autoincrement()) @db.BigInt
  type        ConversationType @default(DIRECT)
  name        String?          @db.VarChar(255) // Group name
  avatarUrl   String?          @map("avatar_url") @db.Text // Group avatar
  description String?          @db.Text // Group description
  createdBy   String           @map("created_by") @db.Uuid
  createdAt   DateTime         @default(now()) @map("created_at") @db.Timestamptz(6)

  creator           Profile                   @relation("ConversationCreator", fields: [createdBy], references: [id], onDelete: Cascade)
  participants      ConversationParticipant[]
  messages          Message[]
  coShoppingSession CoShoppingSession?
  calls             CallSession[]

  @@index([type])
  @@map("conversations")
}
\`\`\`

## 🚀 Usage Flow

### Creating a New Group:
1. Click **"New Group"** or **"+"** button in conversation list
2. Select friends to add (minimum 1 friend)
3. Enter a **group name** (required)
4. Optionally add:
   - Group description
   - Avatar URL
5. Click **"Create Group"**
6. You become the admin automatically
7. All participants receive `conversation_created` socket event
8. Group appears in everyone's conversation list

### Sending Messages in Groups:
1. Open the group chat
2. Type your message
3. All members receive the message via Socket.IO
4. Message shows your name above it (for others)
5. Others see typing indicator: "Your Name is typing..."

### Adding More Members (Admin Only):
1. Open the group chat
2. Click the **➕** button in header
3. Select friends from the list (non-participants only)
4. Click **"Add (N)"** where N is the count
5. New members receive `participant_added` event
6. New members can see full chat history
7. Welcome message appears in chat (optional)

### Managing Group Settings (Admin Only):
1. Click the **⚙️ Settings** button in group header
2. **Update Group Info:**
   - Change name, description, or avatar
   - Click "Save Changes"
   - All members see updated info via `conversation_updated` event
3. **Manage Members:**
   - View list of all participants with roles
   - Click **"Promote to Admin"** to make someone admin
   - Click **"Remove"** to remove a member
   - Cannot remove last admin

### Leaving a Group (Any Member):
1. Open group settings
2. Scroll to bottom
3. Click **"Leave Group"**
4. Confirm the action
5. You're removed from group
6. Group disappears from your list
7. Other members see you left
8. If you're the last member:
   - Group is automatically deleted
   - `conversation_deleted` event sent to all

### Real-time Experience:
- **New Message**: Appears instantly for all members
- **Typing**: "John is typing..." shows while typing
- **Member Join**: "Sarah was added to the group" notification
- **Member Leave**: "Mike left the group" notification
- **Name Change**: Group name updates everywhere immediately
- **Read Receipts**: See who read your messages (✓✓)

---

## 📝 Notes & Best Practices

### Important Notes:
- ✅ All participants in direct chats can add friends
- ✅ Group creator automatically becomes admin
- ✅ Groups can have multiple admins
- ✅ Only admins can manage group and members
- ✅ Any member can leave at any time
- ✅ Friends list automatically filters out existing participants
- ✅ Group chats show member count instead of online status
- ✅ Can only add users who are your friends
- ✅ All changes broadcast via Socket.IO instantly

### Best Practices:
1. **Group Names**: Use clear, descriptive names (1-255 characters)
2. **Member Limits**: Consider adding 50-100 member limit (not enforced yet)
3. **Admin Management**: Promote trusted members to admin for redundancy
4. **Permissions**: Clearly communicate admin vs member permissions to users
5. **Socket Events**: Always listen to all group-related socket events
6. **Error Handling**: Handle "not admin" errors gracefully in UI
7. **Optimistic UI**: Show changes immediately, then confirm via socket

### Socket.IO Integration Checklist:
- [ ] Listen to `conversation_created` event
- [ ] Listen to `conversation_updated` event  
- [ ] Listen to `conversation_deleted` event
- [ ] Listen to `participant_added` event
- [ ] Listen to `participant_removed` event
- [ ] Emit `join_conversation` when opening group
- [ ] Emit `typing` with `userName` and `participantIds`
- [ ] Update UI immediately on socket events
- [ ] Re-join rooms after reconnection
- [ ] Handle connection errors gracefully

---

## 🧪 Testing

### Manual Testing Steps

#### Test 1: Create Group
1. **Create a new group** via API or UI
2. **Verify:**
   - ✅ Group appears in your conversation list
   - ✅ You are marked as admin
   - ✅ All added friends receive `conversation_created` event
   - ✅ Group shows correct name and member count

#### Test 2: Send Messages
1. **Send a message** in the group
2. **Verify:**
   - ✅ All members receive message via Socket.IO
   - ✅ Sender name appears above message (for others)
   - ✅ Message shows in correct order
   - ✅ Read receipts work correctly

#### Test 3: Typing Indicator
1. **Start typing** in the group
2. **Verify:**
   - ✅ Others see "Your Name is typing..."
   - ✅ Indicator appears in chat window
   - ✅ Indicator appears in conversation list
   - ✅ Indicator disappears after 4 seconds of inactivity

#### Test 4: Add Member (Admin)
1. **Click ➕ and add a friend**
2. **Verify:**
   - ✅ New member receives `participant_added` event
   - ✅ New member sees group in their list
   - ✅ New member can see chat history
   - ✅ Notification appears in chat (optional)
   - ✅ Member count updates

#### Test 5: Update Group Details (Admin)
1. **Open settings** and change group name
2. **Verify:**
   - ✅ All members receive `conversation_updated` event
   - ✅ Name updates everywhere immediately
   - ✅ Chat list shows new name
   - ✅ Chat header shows new name

#### Test 6: Promote to Admin
1. **Promote a member** to admin
2. **Verify:**
   - ✅ Member's role changes to "admin"
   - ✅ Member can now access admin features
   - ✅ Member can add/remove participants
   - ✅ Member can update group details

#### Test 7: Remove Member (Admin)
1. **Remove a member** from group
2. **Verify:**
   - ✅ Member receives `participant_removed` event
   - ✅ Group disappears from their list
   - ✅ Member cannot send messages
   - ✅ Notification appears in chat (optional)
   - ✅ Member count updates

#### Test 8: Leave Group (Member)
1. **Click "Leave Group"** as a member
2. **Verify:**
   - ✅ You receive `participant_removed` event
   - ✅ Group disappears from your list
   - ✅ Others see you left
   - ✅ Cannot send/receive messages

#### Test 9: Last Admin Protection
1. **Try to remove** the last admin
2. **Verify:**
   - ✅ Request is rejected with error
   - ✅ Error message explains the issue
   - ✅ Admin remains in group

#### Test 10: Auto-Delete Group
1. **Remove all members** or last member leaves
2. **Verify:**
   - ✅ All participants receive `conversation_deleted` event
   - ✅ Group disappears from all lists
   - ✅ Messages are soft-deleted (in DB)

### Socket.IO Testing

#### Connect and Listen:
```javascript
// Connect to Socket.IO
const socket = io("http://localhost:3000", {
  query: { userId: "your-user-id" }
});

// Listen to all group events
socket.on("conversation_created", (data) => {
  console.log("✅ Group created:", data);
});

socket.on("conversation_updated", (data) => {
  console.log("✅ Group updated:", data);
});

socket.on("participant_added", (data) => {
  console.log("✅ Member added:", data);
});

socket.on("participant_removed", (data) => {
  console.log("✅ Member removed:", data);
});

socket.on("conversation_deleted", (data) => {
  console.log("✅ Group deleted:", data);
});

socket.on("typing", (data) => {
  console.log("✅ Typing:", data.userName, "is typing");
});
```

#### Test Scenarios:
1. Create group → verify `conversation_created` event
2. Add member → verify `participant_added` event
3. Update name → verify `conversation_updated` event
4. Remove member → verify `participant_removed` event
5. Type message → verify `typing` event with userName
6. Last member leaves → verify `conversation_deleted` event

---

## 🎯 Future Enhancements

### Planned Features
- [ ] **Group Avatar Upload** - Upload custom images via Cloudinary
- [ ] **Demote Admin** - Ability to demote admins back to members
- [ ] **Group Settings Page** - Dedicated page for all group settings
- [ ] **Mute Notifications** - Per-group notification preferences
- [ ] **Pin Messages** - Pin important messages in group
- [ ] **Group Invite Links** - Share invite links for easy joining
- [ ] **Maximum Group Size** - Enforce member limits (e.g., 100 members)
- [ ] **Group Description Rich Text** - Markdown support for descriptions
- [ ] **Member Permissions** - Granular permissions (can send media, etc.)
- [ ] **Group Archive** - Archive inactive groups
- [ ] **Group Analytics** - Admin dashboard with stats
- [ ] **Message Threads** - Reply threads within groups
- [ ] **Polls** - Create polls for group decisions
- [ ] **File Sharing Limits** - Per-group storage limits

### Technical Improvements
- [ ] **Batch Operations** - Add/remove multiple members at once
- [ ] **Optimistic Updates** - UI updates before server confirmation
- [ ] **Offline Support** - Queue actions when offline
- [ ] **Push Notifications** - Mobile push for group messages
- [ ] **Search Within Group** - Search messages in specific group
- [ ] **Export Chat History** - Download group chat history
- [ ] **Member Search** - Search for members by name
- [ ] **Typing Indicators Optimization** - Better handling of many typists
- [ ] **Message Delivery Status** - Track delivery per member
- [ ] **Read Receipts per Member** - See who read each message

---

## 📚 Related Documentation

- **[Chat API Documentation](./chat.md)** - Complete chat API reference with all socket events
- **[Socket.IO Events Guide](../docs/chat/SOCKET_EVENTS_GUIDE.md)** - Detailed socket event documentation
- **[Chat Features Guide](../docs/chat/CHAT_FEATURES_README.md)** - All chat features explained
- **[Authentication](./auth.md)** - How to authenticate for API requests
- **[Profiles](./profiles.md)** - User profile management

---

## 🐛 Troubleshooting

### Common Issues

#### Issue 1: "Not authorized" when adding members
**Cause:** Only admins can add members to groups  
**Solution:** Check user's `roleInChat` - must be "admin"

#### Issue 2: Socket events not received
**Cause:** Not joined to conversation room  
**Solution:** Emit `join_conversation` event when opening group

#### Issue 3: Typing indicator not showing in chat list
**Cause:** Missing `participantIds` in typing event  
**Solution:** Always include `participantIds` array in typing emissions

#### Issue 4: Cannot remove last admin
**Cause:** Groups must always have at least one admin  
**Solution:** Promote another member to admin first, then remove

#### Issue 5: Group not deleted when last member leaves
**Cause:** Server-side auto-delete logic issue  
**Solution:** Check server logs - should emit `conversation_deleted` event

#### Issue 6: Typing shows "Someone is typing..." instead of name
**Cause:** `userName` not provided in typing event  
**Solution:** Always include `userName` field when emitting typing event

### Debug Checklist

1. **Socket Connection:**
   ```javascript
   console.log("Connected:", socket.connected);
   console.log("Socket ID:", socket.id);
   ```

2. **Room Membership:**
   ```javascript
   // Check if joined conversation room
   socket.emit("join_conversation", {
     conversationId,
     userId,
     participantIds
   });
   ```

3. **Event Listeners:**
   ```javascript
   // Verify all listeners are set up
   const events = [
     "conversation_created",
     "conversation_updated",
     "conversation_deleted",
     "participant_added",
     "participant_removed",
     "new_message",
     "typing"
   ];
   
   events.forEach(event => {
     console.log(`Listener for ${event}:`, socket.hasListeners(event));
   });
   ```

4. **Backend Logs:**
   - Check server console for socket emissions
   - Look for "📤 Emitting" messages
   - Verify participant IDs are correct

---

**Version:** 2.0.0  
**Last Updated:** January 17, 2026  
**Production URL:** https://done-buddy-production.up.railway.app

