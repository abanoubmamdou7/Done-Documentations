# Advanced Search & Discovery API Documentation

This document provides comprehensive API documentation for the Advanced Search & Discovery feature, which enables global search across e-commerce and social content with filters, search history, trending searches, and suggestions.

## 📋 Table of Contents

1. [Overview](#overview)
2. [Base URL](#base-url)
3. [Authentication](#authentication)
4. [E-commerce Global Search](#1-e-commerce-global-search)
5. [Social Global Search](#2-social-global-search)
6. [Customized Searches](#3-customized-searches)
   - [Products Search](#31-products-search)
   - [Users Search](#32-users-search)
   - [Posts Search](#33-posts-search)
7. [Search History](#4-search-history)
8. [Trending Searches](#5-trending-searches)
9. [Search Suggestions](#6-search-suggestions)
10. [Clear Search History](#7-clear-search-history)
11. [Response Formats](#response-formats)
12. [Error Responses](#error-responses)
13. [Best Practices](#best-practices)

---

## Overview

The Advanced Search & Discovery system provides:

- **E-commerce Global Search**: Search across products, categories, collections, and sellers
- **Social Global Search**: Search across posts, reels, users, friends, and sellers
- **Customized Searches**: Focused searches for specific content types
- **Search History**: Track user search history for personalized suggestions
- **Trending Searches**: Discover what's popular across the platform
- **Search Suggestions**: Autocomplete functionality with intelligent suggestions

---

## Base URL

```
/api/search
```

---

## Authentication

Most search endpoints support **optional authentication**:
- **Without authentication**: Returns public content only
- **With authentication**: Returns personalized results including private content you have access to

**Header (Optional):**
```
Authorization: <your-jwt-token>
```

**Note:** Search history and clear history endpoints require authentication.

---

## 1. E-commerce Global Search

Search across all e-commerce content: products, collections, and sellers.

**Endpoint:** `GET /api/search/ecommerce`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 1 character) |
| `sellerId` | String | No | Filter by seller ID |
| `collectionId` | String | No | Filter by collection ID |
| `minPrice` | Number | No | Minimum price filter |
| `maxPrice` | Number | No | Maximum price filter |
| `category` | String | No | Filter by product category/type |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/ecommerce?q=laptop&minPrice=500&maxPrice=2000&page=1&size=25
```

**Response (200 OK):**
```json
{
  "success": true,
  "query": "laptop",
  "filters": {
    "minPrice": "500",
    "maxPrice": "2000"
  },
  "products": {
    "data": [
      {
        "id": "123",
        "uuid": "550e8400-e29b-41d4-a716-446655440000",
        "title": "Gaming Laptop",
        "vendor": "TechBrand",
        "productType": "Electronics",
        "status": "active",
        "featuredInApp": true,
        "mainImageUrl": "https://...",
        "seller": {
          "id": "1",
          "shopName": "TechStore",
          "avatarUrl": "https://..."
        },
        "images": [
          {
            "src": "https://...",
            "altText": "Gaming Laptop"
          }
        ],
        "variants": [
          {
            "id": "456",
            "title": "16GB RAM",
            "price": "1299.99",
            "currency": "USD"
          }
        ],
        "collections": [
          {
            "collection": {
              "id": "789",
              "title": "Electronics",
              "handle": "electronics"
            }
          }
        ]
      }
    ],
    "total": 45,
    "page": 1,
    "size": 25
  },
  "collections": {
    "data": [
      {
        "id": "789",
        "title": "Electronics Collection",
        "handle": "electronics",
        "description": "All electronics products",
        "products": [
          {
            "id": "123",
            "images": [
              {
                "src": "https://..."
              }
            ]
          }
        ]
      }
    ],
    "total": 3
  },
  "sellers": {
    "data": [
      {
        "id": "1",
        "shopName": "TechStore",
        "description": "Best tech products",
        "profile": {
          "id": "uuid",
          "displayName": "Tech Store",
          "avatarUrl": "https://..."
        },
        "_count": {
          "products": 150
        }
      }
    ],
    "total": 5
  },
  "total": 53
}
```

**Notes:**
- Searches across product title, handle, vendor, product type, tags, and description
- Returns up to 10 collections and 10 sellers in addition to paginated products
- Products are ordered by featured status, then by creation date

---

## 2. Social Global Search

Search across all social content: posts, reels, users, and sellers.

**Endpoint:** `GET /api/search/social`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 1 character) |
| `visibility` | String | No | Filter by visibility (PUBLIC, PRIVATE, FOLLOWERS, CUSTOM) |
| `type` | String | No | Filter by post type (FEED, REEL, STORY) |
| `friendsOnly` | Boolean | No | Filter to friends only (true/false) |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/social?q=vacation&type=REEL&page=1&size=25
```nn

**Response (200 OK):**
```json
{
  "success": true,
  "query": "vacation",
  "filters": {
    "type": "REEL"
  },
  "posts": {
    "data": [
      {
        "id": "post-uuid",
        "type": "REEL",
        "caption": "Amazing vacation destination!",
        "mediaUrl": "https://...",
        "visibility": "PUBLIC",
        "author": {
          "id": "user-uuid",
          "displayName": "John Doe",
          "avatarUrl": "https://...",
          "role": "USER"
        },
        "_count": {
          "likes": 150,
          "comments": 25,
          "shares": 10
        },
        "createdAt": "2024-01-15T10:30:00.000Z"
      }
    ],
    "total": 12,
    "page": 1,
    "size": 25
  },
  "users": {
    "data": [
      {
        "id": "user-uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "bio": "Travel enthusiast",
        "role": "USER",
        "_count": {
          "posts": 45,
          "followsAsFollower": 120,
          "followsAsTarget": 500
        }
      }
    ],
    "total": 8
  },
  "sellers": {
    "data": [
      {
        "id": "1",
        "shopName": "Travel Gear",
        "profile": {
          "id": "seller-uuid",
          "displayName": "Travel Gear Store",
          "avatarUrl": "https://..."
        }
      }
    ],
    "total": 2
  },
  "total": 22
}
```

**Notes:**
- Searches post captions, user display names, and bios
- Returns up to 20 users and 10 sellers in addition to paginated posts
- Visibility filtering respects user permissions (logged-in users see more content)

---

## 3. Customized Searches

### 3.1 Products Search

Focused search for products only with advanced filtering.

**Endpoint:** `GET /api/search/products`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 1 character) |
| `sellerId` | String | No | Filter by seller ID |
| `collectionId` | String | No | Filter by collection ID |
| `minPrice` | Number | No | Minimum price filter |
| `maxPrice` | Number | No | Maximum price filter |
| `category` | String | No | Filter by product category/type |
| `featured` | Boolean | No | Filter featured products only (true/false) |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/products?q=wireless%20mouse&minPrice=20&maxPrice=100&featured=true
```

**Response (200 OK):**
```json
{
  "success": true,
  "query": "wireless mouse",
  "filters": {
    "minPrice": "20",
    "maxPrice": "100",
    "featured": "true"
  },
  "data": [
    {
      "id": "123",
      "uuid": "550e8400-e29b-41d4-a716-446655440000",
      "title": "Wireless Gaming Mouse",
      "vendor": "TechBrand",
      "productType": "Electronics",
      "status": "active",
      "featuredInApp": true,
      "mainImageUrl": "https://...",
      "seller": {
        "id": "1",
        "shopName": "TechStore",
        "avatarUrl": "https://..."
      },
      "images": [
        {
          "src": "https://...",
          "altText": "Wireless Gaming Mouse"
        }
      ],
      "variants": [
        {
          "id": "456",
          "title": "Black",
          "price": "49.99",
          "currency": "USD"
        }
      ]
    }
  ],
  "total": 15,
  "page": 1,
  "size": 25
}
```

---

### 3.2 Users Search

Focused search for users/profiles only.

**Endpoint:** `GET /api/search/users`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 1 character) |
| `role` | String | No | Filter by role (USER, SELLER, MEDIATOR, ADMIN) |
| `friendsOnly` | Boolean | No | Filter to friends only (true/false) |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/users?q=john&role=USER&friendsOnly=false
```

**Response (200 OK):**
```json
{
  "success": true,
  "query": "john",
  "filters": {
    "role": "USER",
    "friendsOnly": "false"
  },
  "data": [
    {
      "id": "user-uuid",
      "displayName": "John Doe",
      "avatarUrl": "https://...",
      "bio": "Software developer",
      "role": "USER",
      "_count": {
        "posts": 45,
        "followsAsFollower": 120,
        "followsAsTarget": 500
      }
    }
  ],
  "total": 8,
  "page": 1,
  "size": 25
}
```

---

### 3.3 Posts Search

Focused search for posts/reels only.

**Endpoint:** `GET /api/search/posts`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 1 character) |
| `visibility` | String | No | Filter by visibility (PUBLIC, PRIVATE, FOLLOWERS, CUSTOM) |
| `type` | String | No | Filter by post type (FEED, REEL, STORY) |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/posts?q=travel&type=REEL&visibility=PUBLIC
```

**Response (200 OK):**
```json
{
  "success": true,
  "query": "travel",
  "filters": {
    "type": "REEL",
    "visibility": "PUBLIC"
  },
  "data": [
    {
      "id": "post-uuid",
      "type": "REEL",
      "caption": "Amazing travel destination!",
      "mediaUrl": "https://...",
      "visibility": "PUBLIC",
      "author": {
        "id": "user-uuid",
        "displayName": "John Doe",
        "avatarUrl": "https://...",
        "role": "USER"
      },
      "_count": {
        "likes": 150,
        "comments": 25,
        "shares": 10
      },
      "createdAt": "2024-01-15T10:30:00.000Z"
    }
  ],
  "total": 25,
  "page": 1,
  "size": 25
}
```

---

## 4. Search History

Get user's search history for personalized suggestions.

**Endpoint:** `GET /api/search/history`

**Authentication:** Required

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | String | No | Filter by search type (ECOMMERCE, SOCIAL, PRODUCTS, USERS, POSTS, etc.) |
| `page` | Number | No | Page number (default: 1) |
| `size` | Number | No | Items per page (default: 25) |

**Example Request:**
```
GET /api/search/history?type=ECOMMERCE&page=1&size=25
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "query": "laptop",
      "searchType": "ECOMMERCE",
      "filters": {
        "minPrice": "500",
        "maxPrice": "2000"
      },
      "resultCount": 45,
      "createdAt": "2024-01-15T10:30:00.000Z"
    },
    {
      "id": "2",
      "query": "wireless mouse",
      "searchType": "PRODUCTS",
      "filters": null,
      "resultCount": 15,
      "createdAt": "2024-01-14T15:20:00.000Z"
    }
  ],
  "total": 50,
  "page": 1,
  "size": 25
}
```

**Notes:**
- Returns unique queries (deduplicated)
- Ordered by most recent first
- Only returns history for authenticated user

---

## 5. Trending Searches

Get trending/popular searches across the platform.

**Endpoint:** `GET /api/search/trending`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | String | No | Filter by search type (ECOMMERCE, SOCIAL, PRODUCTS, USERS, POSTS, etc.) |
| `limit` | Number | No | Number of results (default: 20, max: 50) |

**Example Request:**
```
GET /api/search/trending?type=ECOMMERCE&limit=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "id": "1",
      "query": "laptop",
      "searchType": "ECOMMERCE",
      "count": 1250,
      "lastSearchedAt": "2024-01-15T10:30:00.000Z",
      "createdAt": "2024-01-10T08:00:00.000Z"
    },
    {
      "id": "2",
      "query": "wireless mouse",
      "searchType": "ECOMMERCE",
      "count": 890,
      "lastSearchedAt": "2024-01-15T09:15:00.000Z",
      "createdAt": "2024-01-12T14:20:00.000Z"
    }
  ]
}
```

**Notes:**
- Ordered by search count (descending), then by last searched date
- Shows what's popular across the platform
- Updates automatically as users search

---

## 6. Search Suggestions

Get autocomplete suggestions for search queries.

**Endpoint:** `GET /api/search/suggestions`

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | String | Yes | Search query (min 2 characters) |
| `type` | String | No | Filter by search type (ECOMMERCE, SOCIAL, PRODUCTS, USERS, POSTS, etc.) |
| `limit` | Number | No | Number of suggestions (default: 10, max: 20) |

**Example Request:**
```
GET /api/search/suggestions?q=lap&type=ECOMMERCE&limit=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "data": [
    {
      "query": "laptop",
      "type": "trending",
      "count": 1250
    },
    {
      "query": "laptop bag",
      "type": "trending",
      "count": 450
    },
    {
      "query": "laptop stand",
      "type": "history"
    }
  ]
}
```

**Notes:**
- Combines trending searches and user's search history
- Prioritizes trending searches
- Minimum 2 characters required
- Useful for autocomplete/search-as-you-type functionality

---

## 7. Clear Search History

Clear user's search history.

**Endpoint:** `DELETE /api/search/history`

**Authentication:** Required

**Query Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `type` | String | No | Clear specific search type only (if not provided, clears all) |

**Example Request:**
```
DELETE /api/search/history?type=ECOMMERCE
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Search history cleared successfully"
}
```

**Notes:**
- Clears all history if `type` is not provided
- Clears only specific type if `type` is provided
- Cannot be undone

---

## Response Formats

### Success Response
All successful responses follow this format:
```json
{
  "success": true,
  "data": [...],
  "total": 100,
  "page": 1,
  "size": 25
}
```

### Pagination
- `page`: Current page number (default: 1)
- `size`: Items per page (default: 25)
- `total`: Total number of results

---

## Error Responses

### 400 Bad Request
```json
{
  "message": "Search query is required",
  "status_code": 400
}
```

### 401 Unauthorized
```json
{
  "message": "Authentication required",
  "status_code": 401
}
```

### 422 Validation Error
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "q": "\"q\" is required",
    "minPrice": "\"minPrice\" must be a valid number"
  }
}
```

---

## Best Practices

### 1. Search Query Optimization
- Use specific keywords for better results
- Combine multiple filters for refined searches
- Use autocomplete suggestions for better UX

### 2. Performance
- Implement debouncing for search-as-you-type (minimum 300ms)
- Cache trending searches on the frontend
- Use pagination for large result sets

### 3. User Experience
- Show search suggestions as user types
- Display trending searches on search page
- Show search history for quick access
- Provide clear filters for refinement

### 4. Search Types
- Use **E-commerce Search** for shopping/product discovery
- Use **Social Search** for content discovery
- Use **Customized Searches** for focused searches

### 5. Filtering
- Start with broad searches, then apply filters
- Combine multiple filters for better results
- Remember that filters are optional

### 6. Search History
- Respect user privacy (clear history option)
- Use history for personalized suggestions
- Don't store sensitive information in search queries

---

## Example Workflows

### E-commerce Shopping Flow
1. User types "laptop" in search
2. Frontend calls `/api/search/suggestions?q=lap`
3. User selects "laptop" from suggestions
4. Frontend calls `/api/search/ecommerce?q=laptop&minPrice=500&maxPrice=2000`
5. Display results with filters
6. User clicks on a product

### Social Discovery Flow
1. User types "travel" in search
2. Frontend calls `/api/search/suggestions?q=trav&type=SOCIAL`
3. User selects "travel" from suggestions
4. Frontend calls `/api/search/social?q=travel&type=REEL`
5. Display posts, users, and sellers
6. User interacts with content

---

## Notes

- All search queries are case-insensitive
- Search history is automatically tracked for authenticated users
- Trending searches update in real-time
- BigInt values are automatically serialized to strings in responses
- Date values are returned in ISO 8601 format
- Search results respect user permissions and visibility settings
