# Advanced Search Implementation

## Overview
Implemented comprehensive search functionality across messages, conversations, and groups with advanced filtering options.

**Date:** January 17, 2026  
**Version:** 2.0.0

---

## ✅ Implemented Features

### 1. Enhanced Message Search
**Endpoint:** `GET /api/messages/search/messages`

**Query Parameters:**
- `query` (required) - Search term (2-200 chars)
- `conversationId` (optional) - Filter by conversation
- `type` (optional) - Filter by message type (TEXT, IMAGE, FILE, VOICE)
- `senderId` (optional) - Filter by sender UUID
- `dateFrom` (optional) - Start date (ISO 8601)
- `dateTo` (optional) - End date (ISO 8601)
- `hasReactions` (optional) - Filter messages with reactions (true/false)
- `hasReplies` (optional) - Filter reply messages (true/false)
- `page` (optional) - Page number (default: 1)
- `size` (optional) - Items per page (default: 25, max: 50)

**Example Requests:**
```bash
# Basic search
GET /api/messages/search/messages?query=meeting

# Search with type filter
GET /api/messages/search/messages?query=report&type=FILE

# Search with date range
GET /api/messages/search/messages?query=project&dateFrom=2024-01-01&dateTo=2024-01-31

# Search messages with reactions
GET /api/messages/search/messages?query=great&hasReactions=true

# Combined filters
GET /api/messages/search/messages?query=meeting&conversationId=123&type=TEXT&hasReactions=true
```

**Response Fields:**
- Full message data with reactions
- Reply information (replyTo)
- Mentions array
- Sender information
- Conversation context (type, name, avatar)
- Edit history (editedAt)

---

### 2. Conversation Search ⭐ NEW
**Endpoint:** `GET /api/conversations/search`

**Query Parameters:**
- `query` (required) - Search term (2-200 chars)
- `type` (optional) - Filter by type (DIRECT, GROUP, CO_SHOPPING)
- `hasUnread` (optional) - Filter conversations with unread messages (true/false)
- `page` (optional) - Page number (default: 1)
- `size` (optional) - Items per page (default: 25, max: 50)

**Search Criteria:**
- Group names (case-insensitive)
- Group descriptions
- Participant display names
- Direct chat contact names

**Example Requests:**
```bash
# Search all conversations
GET /api/conversations/search?query=project

# Search only groups
GET /api/conversations/search?query=team&type=GROUP

# Search with unread filter
GET /api/conversations/search?query=john&hasUnread=true

# Search direct chats only
GET /api/conversations/search?query=sarah&type=DIRECT
```

**Response Fields:**
- `matchedOn` - Indicates what matched (name, description, participant)
- Unread count per conversation
- Participant count
- Last message preview
- Full participant list (for groups)
- Other participant info (for direct chats)

---

### 3. Group-Specific Search ⭐ NEW
**Endpoint:** `GET /api/conversations/search/groups`

**Query Parameters:**
- `query` (required) - Search term (2-200 chars)
- `hasAdmin` (optional) - Filter groups where you're admin (true/false)
- `minMembers` (optional) - Minimum member count
- `maxMembers` (optional) - Maximum member count
- `createdAfter` (optional) - Filter by creation date (ISO 8601)
- `page` (optional) - Page number (default: 1)
- `size` (optional) - Items per page (default: 25, max: 50)

**Example Requests:**
```bash
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

**Response Fields:**
- `yourRole` - Your role in the group (admin/member)
- `adminCount` - Number of admins
- `participantCount` - Total members
- Full participant list with roles
- Creation date and creator
- Unread count
- Last message

---

### 4. Global Search ⭐ NEW
**Endpoint:** `GET /api/conversations/search/global`

**Query Parameters:**
- `query` (required) - Search term (2-200 chars)
- `searchIn` (optional) - Comma-separated types (messages, conversations, groups)
- `size` (optional) - Items per type (default: 10, max: 25)

**Example Requests:**
```bash
# Search everything
GET /api/conversations/search/global?query=project

# Search messages and groups only
GET /api/conversations/search/global?query=meeting&searchIn=messages,groups

# Search conversations only
GET /api/conversations/search/global?query=john&searchIn=conversations
```

**Response Structure:**
```json
{
  "success": true,
  "query": "project",
  "results": {
    "messages": {
      "total": 25,
      "data": [...]
    },
    "conversations": {
      "total": 5,
      "data": [...]
    },
    "groups": {
      "total": 3,
      "data": [...]
    }
  }
}
```

**Features:**
- Unified search across all types
- Categorized results
- Totals per category
- Flexible filtering (choose what to search)
- Parallel queries for performance
- Deduplicated results

---

## 🔧 Technical Implementation

### Files Modified

1. **`src/modules/chat/controller/message.controller.js`**
   - Enhanced `searchMessages()` with 7 new filters
   - Added reactions, replies, date range, type, sender filters
   - Improved response with full message data

2. **`src/modules/chat/controller/conversation.controller.js`**
   - Added `searchConversations()` - Search all conversation types
   - Added `searchGroups()` - Group-specific advanced search
   - Added `globalSearch()` - Unified search across all types

3. **`src/modules/chat/conversation.router.js`**
   - Added 3 new search routes
   - All protected with auth middleware
   - Validation middleware applied

4. **`src/modules/chat/controller/message.validation.js`**
   - Enhanced searchMessages validation with all new parameters
   - Added date validation (ISO 8601)
   - Added boolean flag validation

5. **`src/modules/chat/controller/conversation.validation.js`**
   - Added `searchConversations` validation
   - Added `searchGroups` validation  
   - Added `globalSearch` validation
   - Regex patterns for searchIn parameter

### Database Queries

**Performance Optimizations:**
- PostgreSQL full-text search for messages
- Case-insensitive LIKE queries with indexes
- Parallel queries for global search
- Efficient unread count batching
- Proper use of Prisma includes

**Privacy & Security:**
- Only searches user's own conversations
- Validates conversation participation
- Excludes soft-deleted messages
- Auth middleware on all routes
- Input validation and sanitization

---

## 📊 Search Features Matrix

| Feature | Message Search | Conversation Search | Group Search | Global Search |
|---------|---------------|---------------------|--------------|---------------|
| **Full-text search** | ✅ | ✅ | ✅ | ✅ |
| **Filter by type** | ✅ (message) | ✅ (conv) | ❌ (groups only) | ✅ (searchIn) |
| **Date range** | ✅ | ❌ | ✅ (createdAfter) | ❌ |
| **Sender filter** | ✅ | ❌ | ❌ | ❌ |
| **Reaction filter** | ✅ | ❌ | ❌ | ❌ |
| **Reply filter** | ✅ | ❌ | ❌ | ❌ |
| **Unread filter** | ❌ | ✅ | ✅ (included) | ✅ (included) |
| **Admin filter** | ❌ | ❌ | ✅ | ❌ |
| **Member count** | ❌ | ❌ | ✅ | ❌ |
| **Match indicator** | ❌ | ✅ | ❌ | ✅ |
| **Pagination** | ✅ | ✅ | ✅ | ❌ (fixed size) |
| **Context included** | ✅ (conv) | ✅ (last msg) | ✅ (full) | ✅ (all) |

---

## 🚀 Usage Examples

### Frontend Integration

```javascript
// Advanced Search Class
class ChatSearch {
  constructor(apiClient) {
    this.api = apiClient;
  }

  // Message search with filters
  async searchMessages(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(
      `/api/messages/search/messages?${params}`
    );
    return response.data;
  }

  // Conversation search
  async searchConversations(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(
      `/api/conversations/search/conversations?${params}`
    );
    return response.data;
  }

  // Group search
  async searchGroups(query, filters = {}) {
    const params = new URLSearchParams({
      query,
      ...filters
    });

    const response = await this.api.get(
      `/api/conversations/search/groups?${params}`
    );
    return response.data;
  }

  // Global search
  async globalSearch(query, options = {}) {
    const params = new URLSearchParams({
      query,
      ...options
    });

    const response = await this.api.get(
      `/api/conversations/search/global?${params}`
    );
    return response.data;
  }
}

// Usage Examples
const search = new ChatSearch(apiClient);

// Search messages from last week
const recentMessages = await search.searchMessages("meeting", {
  dateFrom: "2024-01-10",
  dateTo: "2024-01-17",
  type: "TEXT"
});

// Find groups you manage
const adminGroups = await search.searchGroups("project", {
  hasAdmin: true,
  minMembers: 5
});

// Global search
const results = await search.globalSearch("john", {
  searchIn: "messages,conversations"
});
```

### React Hook Example

```javascript
import { useState, useCallback } from 'react';
import { debounce } from 'lodash';

function useAdvancedSearch(apiClient) {
  const [results, setResults] = useState(null);
  const [loading, setLoading] = useState(false);

  const search = useCallback(
    debounce(async (query, options = {}) => {
      if (query.length < 2) {
        setResults(null);
        return;
      }

      setLoading(true);
      try {
        const search = new ChatSearch(apiClient);
        const data = await search.globalSearch(query, options);
        setResults(data);
      } catch (error) {
        console.error('Search error:', error);
        setResults(null);
      } finally {
        setLoading(false);
      }
    }, 300),
    [apiClient]
  );

  return { results, loading, search };
}

// Component usage
function SearchBar() {
  const { results, loading, search } = useAdvancedSearch(apiClient);

  return (
    <div>
      <input
        type="text"
        placeholder="Search messages, chats, groups..."
        onChange={(e) => search(e.target.value)}
      />
      {loading && <Spinner />}
      {results && <SearchResults results={results} />}
    </div>
  );
}
```

---

## 🧪 Testing

### Manual Testing

```bash
# 1. Test message search with filters
curl -H "authorization: YOUR_TOKEN" \
  "http://localhost:3000/api/messages/search/messages?query=meeting&type=TEXT&hasReactions=true"

# 2. Test conversation search
curl -H "authorization: YOUR_TOKEN" \
  "http://localhost:3000/api/conversations/search/conversations?query=john&type=DIRECT"

# 3. Test group search
curl -H "authorization: YOUR_TOKEN" \
  "http://localhost:3000/api/conversations/search/groups?query=team&hasAdmin=true&minMembers=5"

# 4. Test global search
curl -H "authorization: YOUR_TOKEN" \
  "http://localhost:3000/api/conversations/search/global?query=project&searchIn=messages,groups"
```

### Postman Collection

```json
{
  "name": "Advanced Search",
  "requests": [
    {
      "name": "Search Messages",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/api/messages/search/messages",
        "params": [
          { "key": "query", "value": "meeting" },
          { "key": "type", "value": "TEXT" },
          { "key": "hasReactions", "value": "true" }
        ]
      }
    },
    {
      "name": "Search Conversations",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/api/conversations/search/conversations",
        "params": [
          { "key": "query", "value": "project" },
          { "key": "type", "value": "GROUP" }
        ]
      }
    },
    {
      "name": "Search Groups",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/api/conversations/search/groups",
        "params": [
          { "key": "query", "value": "team" },
          { "key": "hasAdmin", "value": "true" },
          { "key": "minMembers", "value": "5" }
        ]
      }
    },
    {
      "name": "Global Search",
      "request": {
        "method": "GET",
        "url": "{{baseUrl}}/api/conversations/search/global",
        "params": [
          { "key": "query", "value": "project" },
          { "key": "searchIn", "value": "messages,conversations,groups" }
        ]
      }
    }
  ]
}
```

---

## 📝 Notes

### Performance Considerations

1. **Database Indexes**
   - Full-text search indexes on message content
   - Indexes on conversation names and descriptions
   - Indexes on profile displayName for participant search

2. **Query Optimization**
   - Parallel queries in global search
   - Efficient unread count batching
   - Proper use of Prisma includes
   - Pagination to limit result sets

3. **Caching Strategy** (Recommended)
   - Cache recent searches for 5 minutes
   - Cache conversation lists
   - Invalidate on new messages/updates

### Security

- ✅ All routes protected with auth middleware
- ✅ Only searches user's own conversations
- ✅ Input validation and sanitization
- ✅ Excludes soft-deleted messages
- ✅ Privacy-respecting participant filtering

### Future Enhancements

- [ ] Highlight matched text in results
- [ ] Search history and suggestions
- [ ] Advanced filters UI
- [ ] Export search results
- [ ] Save custom search queries
- [ ] Search within specific date ranges (UI)
- [ ] Filter by media type (images only, files only)
- [ ] Search by hashtags or keywords
- [ ] Full-text search ranking/relevance scores

---

## 🔗 Related Documentation

- [Chat API Documentation](../../Documentations/chat.md)
- [Group Chat Feature](../../Documentations/GROUP_CHAT.md)
- [Socket.IO Events](./SOCKET_EVENTS_GUIDE.md)

---

**Version:** 2.0.0  
**Last Updated:** January 17, 2026  
**Author:** Development Team
