# Privacy Groups API Documentation

This document provides comprehensive API documentation for the Privacy Groups feature, which allows users to create custom privacy groups and control who can see their posts, reels, and stories.

## Base URL
```
/api/privacy-groups
```

## Authentication
All endpoints require authentication. Include the JWT token in the Authorization header:
```
Authorization: <your-jwt-token>
```

---

## Overview

Privacy Groups allow users to create custom groups of friends (e.g., "Family", "Work", "Classmates") and use these groups to control who can see their content. When creating a post, reel, or story, users can select:

1. **PUBLIC** - Visible to all friends and all app users
2. **PRIVATE** - Visible only to friends
3. **FOLLOWERS** - Visible to followers and friends
4. **CUSTOM** - Visible only to members of a selected privacy group

---

## Table of Contents

1. [Get All Privacy Groups](#1-get-all-privacy-groups)
2. [Get Single Privacy Group](#2-get-single-privacy-group)
3. [Create Privacy Group](#3-create-privacy-group)
4. [Update Privacy Group](#4-update-privacy-group)
5. [Delete Privacy Group](#5-delete-privacy-group)
6. [Add Members to Group](#6-add-members-to-group)
7. [Remove Member from Group](#7-remove-member-from-group)
8. [Using Privacy Groups with Posts](#using-privacy-groups-with-posts)
9. [Using Privacy Groups with Stories](#using-privacy-groups-with-stories)

---

## 1. Get All Privacy Groups

Get all privacy groups created by the authenticated user.

**Endpoint:** `GET /api/privacy-groups`

**Headers:**
```
Authorization: <token>
```

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| page | String | No | 1 | Page number |
| size | String | No | 25 | Items per page |

**Example Request:**
```
GET /api/privacy-groups?page=1&size=25
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": 1,
  "size": 25,
  "total": 3,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "name": "Family",
      "description": "Close family members",
      "memberCount": 10,
      "members": [
        {
          "id": "660e8400-e29b-41d4-a716-446655440001",
          "displayName": "John Doe",
          "avatarUrl": "https://...",
          "role": "USER",
          "addedAt": "2024-01-15T10:30:00.000Z"
        }
      ],
      "createdAt": "2024-01-01T12:00:00.000Z",
      "updatedAt": "2024-01-01T12:00:00.000Z"
    }
  ]
}
```

**Notes:**
- Returns up to 5 members as a preview
- Use the single group endpoint to see all members

---

## 2. Get Single Privacy Group

Get detailed information about a specific privacy group including all members.

**Endpoint:** `GET /api/privacy-groups/:groupId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| groupId | UUID | Yes | Privacy group ID |

**Example Request:**
```
GET /api/privacy-groups/550e8400-e29b-41d4-a716-446655440000
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Family",
    "description": "Close family members",
    "memberCount": 10,
    "members": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER",
        "addedAt": "2024-01-15T10:30:00.000Z"
      },
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "displayName": "Jane Smith",
        "avatarUrl": "https://...",
        "role": "USER",
        "addedAt": "2024-01-15T10:35:00.000Z"
      }
    ],
    "createdAt": "2024-01-01T12:00:00.000Z",
    "updatedAt": "2024-01-01T12:00:00.000Z"
  }
}
```

**Error Responses:**
- `404` - Privacy group not found
- `403` - You are not authorized to view this privacy group (not the owner)

---

## 3. Create Privacy Group

Create a new privacy group and optionally add members.

**Endpoint:** `POST /api/privacy-groups`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | String | Yes | Group name (1-255 characters) |
| description | String | No | Group description (max 1000 characters) |
| memberIds | Array[UUID] | No | Array of friend profile IDs to add initially |

**Example Request:**
```json
{
  "name": "Family",
  "description": "Close family members",
  "memberIds": [
    "660e8400-e29b-41d4-a716-446655440001",
    "770e8400-e29b-41d4-a716-446655440002"
  ]
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Privacy group created successfully",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Family",
    "description": "Close family members",
    "memberCount": 2,
    "members": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER",
        "addedAt": "2024-01-15T10:30:00.000Z"
      },
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "displayName": "Jane Smith",
        "avatarUrl": "https://...",
        "role": "USER",
        "addedAt": "2024-01-15T10:30:00.000Z"
      }
    ],
    "createdAt": "2024-01-15T10:30:00.000Z",
    "updatedAt": "2024-01-15T10:30:00.000Z"
  }
}
```

**Error Responses:**
- `400` - Invalid member IDs (must be friends) or invalid format
- `401` - Unauthorized

**Notes:**
- Only friends can be added to privacy groups
- You can create a group without members and add them later

---

## 4. Update Privacy Group

Update the name or description of a privacy group.

**Endpoint:** `PUT /api/privacy-groups/:groupId`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| groupId | UUID | Yes | Privacy group ID |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| name | String | No | New group name (1-255 characters) |
| description | String | No | New group description (max 1000 characters) |

**Example Request:**
```json
{
  "name": "Extended Family",
  "description": "All family members including extended family"
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Privacy group updated successfully",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "name": "Extended Family",
    "description": "All family members including extended family",
    "memberCount": 10,
    "members": [...],
    "createdAt": "2024-01-01T12:00:00.000Z",
    "updatedAt": "2024-01-15T11:00:00.000Z"
  }
}
```

**Error Responses:**
- `404` - Privacy group not found
- `403` - You are not authorized to update this privacy group

---

## 5. Delete Privacy Group

Delete a privacy group. This will remove the group and all its members, but posts/stories using this group will have their privacyGroupId set to null.

**Endpoint:** `DELETE /api/privacy-groups/:groupId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| groupId | UUID | Yes | Privacy group ID |

**Example Request:**
```
DELETE /api/privacy-groups/550e8400-e29b-41d4-a716-446655440000
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Privacy group deleted successfully"
}
```

**Error Responses:**
- `404` - Privacy group not found
- `403` - You are not authorized to delete this privacy group

**Notes:**
- Deleting a privacy group will set `privacyGroupId` to null for posts/stories using it
- The posts/stories will still exist but may become visible to more people

---

## 6. Add Members to Group

Add one or more friends to an existing privacy group.

**Endpoint:** `POST /api/privacy-groups/:groupId/members`

**Headers:**
```
Authorization: <token>
Content-Type: application/json
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| groupId | UUID | Yes | Privacy group ID |

**Request Body:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| memberIds | Array[UUID] | Yes | Array of friend profile IDs to add (min 1) |

**Example Request:**
```json
{
  "memberIds": [
    "880e8400-e29b-41d4-a716-446655440003",
    "990e8400-e29b-41d4-a716-446655440004"
  ]
}
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Members added successfully",
  "data": {
    "added": 2,
    "alreadyInGroup": 0,
    "totalMembers": 12,
    "members": [...]
  }
}
```

**Error Responses:**
- `400` - Invalid member IDs (must be friends) or members already in group
- `404` - Privacy group not found
- `403` - You are not authorized to modify this privacy group

**Notes:**
- Only friends can be added
- If a member is already in the group, they are skipped (not an error)
- Response shows how many were added vs already in group

---

## 7. Remove Member from Group

Remove a member from a privacy group.

**Endpoint:** `DELETE /api/privacy-groups/:groupId/members/:memberId`

**Headers:**
```
Authorization: <token>
```

**Path Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| groupId | UUID | Yes | Privacy group ID |
| memberId | UUID | Yes | Profile ID of member to remove |

**Example Request:**
```
DELETE /api/privacy-groups/550e8400-e29b-41d4-a716-446655440000/members/660e8400-e29b-41d4-a716-446655440001
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Member removed from privacy group successfully"
}
```

**Error Responses:**
- `404` - Privacy group not found or member not in group
- `403` - You are not authorized to modify this privacy group

---

## Using Privacy Groups with Posts

When creating or updating a post, you can use privacy groups by setting the visibility to `CUSTOM` and providing a `privacyGroupId`.

### Create Post with Custom Privacy

**Endpoint:** `POST /api/posts`

**Request Body:**
```json
{
  "caption": "Family vacation photos",
  "type": "FEED",
  "visibility": "CUSTOM",
  "privacyGroupId": "550e8400-e29b-41d4-a716-446655440000"
}
```

### Update Post Privacy

**Endpoint:** `PATCH /api/posts/:postId`

**Request Body:**
```json
{
  "visibility": "PRIVATE"
}
```

**Available Visibility Options:**
- `PUBLIC` - Visible to everyone (default)
- `PRIVATE` - Only friends can see
- `FOLLOWERS` - Followers and friends can see
- `CUSTOM` - Only members of the specified privacy group can see (requires `privacyGroupId`)

---

## Using Privacy Groups with Stories

Stories support the same privacy options as posts.

### Create Story with Custom Privacy

**Endpoint:** `POST /api/stories`

**Request Body (Form Data):**
```
storyMedia: [file]
caption: "Private story for family"
visibility: CUSTOM
privacyGroupId: 550e8400-e29b-41d4-a716-446655440000
```

**Or JSON:**
```json
{
  "mediaUrl": "https://...",
  "caption": "Private story for family",
  "visibility": "CUSTOM",
  "privacyGroupId": "550e8400-e29b-41d4-a716-446655440000"
}
```

---

## Privacy Rules Summary

### Post/Story Visibility Rules

1. **PUBLIC**
   - Visible to: Everyone (all app users)
   - Use case: Public content for maximum reach

2. **PRIVATE**
   - Visible to: Only friends of the author
   - Use case: Personal content for close friends only

3. **FOLLOWERS**
   - Visible to: Users who follow the author OR are friends
   - Use case: Content for your audience

4. **CUSTOM**
   - Visible to: Only members of the specified privacy group
   - Use case: Content for specific groups (family, work, etc.)
   - Requires: `privacyGroupId` must be provided

### Feed Filtering

- **For You Feed**: Shows PUBLIC posts, FOLLOWERS posts from followed users, PRIVATE posts from friends, and CUSTOM posts from groups you're a member of
- **Following Feed**: Shows posts from friends/following based on visibility settings

---

## Error Responses

### 400 Bad Request
```json
{
  "message": "Cannot add non-friends to privacy group. Invalid member IDs: uuid1, uuid2",
  "status_code": 400
}
```

### 401 Unauthorized
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

### 403 Forbidden
```json
{
  "message": "You are not authorized to view this privacy group",
  "status_code": 403
}
```

### 404 Not Found
```json
{
  "message": "Privacy group not found",
  "status_code": 404
}
```

---

## Best Practices

1. **Group Naming**: Use clear, descriptive names (e.g., "Family", "Work Colleagues", "College Friends")

2. **Member Management**: 
   - Add members when creating the group for convenience
   - Or create an empty group and add members later

3. **Privacy Selection**:
   - Use PUBLIC for content you want maximum visibility
   - Use PRIVATE for personal content
   - Use CUSTOM for content targeted to specific groups

4. **Group Organization**:
   - Create groups based on relationships (family, work, school)
   - Keep groups focused and relevant
   - Regularly review and update group members

5. **Content Privacy**:
   - Remember that deleting a privacy group affects all posts/stories using it
   - Consider the audience before selecting privacy settings

---

## Example Workflow

1. **Create a Privacy Group:**
   ```
   POST /api/privacy-groups
   {
     "name": "Family",
     "memberIds": ["friend-uuid-1", "friend-uuid-2"]
   }
   ```

2. **Create a Post with Custom Privacy:**
   ```
   POST /api/posts
   {
     "caption": "Family dinner!",
     "visibility": "CUSTOM",
     "privacyGroupId": "group-uuid-from-step-1"
   }
   ```

3. **Add More Members Later:**
   ```
   POST /api/privacy-groups/{groupId}/members
   {
     "memberIds": ["friend-uuid-3"]
   }
   ```

---

## Notes

- Only friends can be added to privacy groups
- Users can only use privacy groups they own
- When a privacy group is deleted, posts/stories using it will have their `privacyGroupId` set to null
- Privacy settings are enforced at the API level for security
- The system automatically filters content based on privacy settings in feeds
