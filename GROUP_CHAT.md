# Group Chat Feature

## Overview
Added full group chat support with the ability to convert 1-on-1 chats to groups and manage participants.

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

### 1. **Create Conversation**
- `POST /api/conversations`
- Body: `{ "participantId": "uuid" }`
- Creates or returns existing conversation with a friend

### 2. **Get All Conversations**
- `GET /api/conversations?page=1&size=25`
- Returns paginated list of user's conversations

### 3. **Get Conversation by ID**
- `GET /api/conversations/:conversationId`
- Returns conversation details with participants

### 4. **Convert to Group**
- `POST /api/conversations/:conversationId/convert-to-group`
- Body: `{ "name": "Group Name", "description": "Optional", "avatarUrl": "Optional" }`
- Converts a direct chat to a group chat

### 5. **Add Participants**
- `POST /api/conversations/:conversationId/participants`
- Body: `{ "participantIds": ["uuid1", "uuid2"] }`
- Adds friends to an existing conversation (admin only for groups)

### 6. **Remove Participant**
- `DELETE /api/conversations/:conversationId/participants/:participantId`
- Removes a participant from a group (admin or self)

### 7. **Update Group Details**
- `PATCH /api/conversations/:conversationId`
- Body: `{ "name": "New Name", "description": "New desc", "avatarUrl": "url" }`
- Updates group information (admin only)

## Frontend Components

### New Components

1. **GroupConvertModal.jsx**
   - Modal to convert a direct chat to a group
   - Prompts for group name and description

2. **AddParticipantsModal.jsx**
   - Modal to add friends to a conversation
   - Shows friends list with selection
   - Filters out existing participants

### Updated Components

1. **ChatWindow.jsx**
   - Added "Convert to Group" button (👥) for direct chats
   - Added "Add Friends" button (➕) for all conversations
   - Shows group name and member count for groups
   - Displays group avatar or initials

2. **App.jsx**
   - Added `onConversationUpdated` callback to handle conversation updates

3. **api.js**
   - Added `conversationAPI.convertToGroup()`
   - Added `conversationAPI.addParticipants()`
   - Added `conversationAPI.removeParticipant()`
   - Added `conversationAPI.updateGroup()`

## Features

### 1. **Direct Chat → Group Conversion**
   - Click the 👥 button in a direct chat
   - Enter group name and optional description
   - Chat is converted to a group
   - Creator becomes admin

### 2. **Add Friends to Conversation**
   - Click the ➕ button
   - Select friends from your friends list
   - Friends who are already participants are filtered out
   - Only friends can be added

### 3. **Group Management**
   - Admins can add/remove participants
   - Admins can update group name, description, and avatar
   - Members can leave the group themselves
   - Group shows member count instead of "Online" status

### 4. **Permissions**
   - **Direct chats**: Both participants can add friends and convert to group
   - **Groups**: 
     - Only admins can add participants
     - Only admins can remove participants
     - Only admins can update group details
     - Any member can leave

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

## Usage Flow

### Converting to Group:
1. Open a direct chat with a friend
2. Click the "👥" button in the chat header
3. Enter a group name (required) and description (optional)
4. Click "Convert to Group"
5. The chat is now a group and you can add more friends

### Adding Friends:
1. In any chat (direct or group), click the "➕" button
2. Select one or more friends from the list
3. Click "Add (N)" where N is the number selected
4. For direct chats: this automatically converts it to a group
5. For groups: only admins can add friends

## Notes

- All participants in direct chats can convert to group and add friends
- Once converted to a group, only admins can manage participants
- The creator of a group is automatically set as admin
- Friends list automatically filters out existing participants
- Group chats show member count instead of online status
- You can only add users who are your friends

## Testing

1. **Create a direct chat** with a friend
2. **Send some messages** to verify it works
3. **Convert to group**: Click 👥 and enter a name
4. **Add more friends**: Click ➕ and select friends
5. **Verify**: Check that the chat header shows the group name and member count
6. **Send messages**: Ensure all participants receive messages

## Future Enhancements

- Group avatar upload
- Promote/demote admin roles
- Group settings page
- Mute/notification preferences per group
- Group invite links
- Maximum group size limits

