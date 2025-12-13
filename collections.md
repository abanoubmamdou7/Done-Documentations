# Collections API Documentation

## Base URL
```
/api/collections
```

## Authentication
- **Public Endpoints**: No authentication required
- **Protected Endpoints**: Require Bearer token in `Authorization` header
  ```
  Authorization: Bearer <token>
  ```
- **Role-Based Access**: Some endpoints require specific roles (SELLER, USER, MEDIATOR)

---

## 📋 Table of Contents
1. [Get All Collections](#1-get-all-collections)
2. [Get Root Collections](#2-get-root-collections)
3. [Get Collection Tree](#3-get-collection-tree)
4. [Get Collection by ID](#4-get-collection-by-id)
5. [Get Sub-Collections](#5-get-sub-collections)
6. [Get Collection Breadcrumb](#6-get-collection-breadcrumb)
7. [Set Collection Parent](#7-set-collection-parent)
8. [Set Collection Hierarchy](#8-set-collection-hierarchy)
9. [Set Parent Bulk](#9-set-parent-bulk)
10. [Auto-Detect Hierarchy](#10-auto-detect-hierarchy)

---

## 1. Get All Collections

Get all collections with optional filtering and hierarchy support.

**Endpoint:** `GET /api/collections`

**Authentication:** Public (No auth required)

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `sellerId` | string | No | - | Filter by seller ID |
| `rootOnly` | string | No | - | If `"true"`, returns only root collections (no parent) |
| `tree` | string | No | - | If `"true"`, returns hierarchical tree structure |
| `page` | string | No | `"1"` | Page number for pagination |
| `size` | string | No | `"25"` | Items per page (max 25) |

**Example Request:**
```http
GET /api/collections?sellerId=1&rootOnly=true&page=1&size=10
```

**Response:**
```json
{
  "success": true,
  "pagination": {
    "page": 1,
    "size": 10,
    "total": 50,
    "pages": 5
  },
  "data": [
    {
      "id": "45",
      "sellerId": "1",
      "shopifyCollectionId": "451263561983",
      "handle": "home",
      "title": "HOME",
      "bodyHtml": "",
      "collectionType": "CUSTOM",
      "published": true,
      "publishedAt": "2025-02-26T13:24:12.000Z",
      "sortOrder": "best-selling",
      "imageUrl": "https://cdn.shopify.com/..."
    }
  ]
}
```

**With Tree Structure (`tree=true`):**
```json
{
  "success": true,
  "pagination": { ... },
  "data": [
    {
      "id": "45",
      "title": "HOME",
      "children": [
        {
          "id": "46",
          "title": "HOME ACCESSORIES",
          "children": []
        }
      ]
    }
  ]
}
```

---

## 2. Get Root Collections

Get only root collections (collections with no parent).

**Endpoint:** `GET /api/collections/roots`

**Authentication:** Public (No auth required)

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `sellerId` | string | No | - | Filter by seller ID |
| `page` | string | No | `"1"` | Page number for pagination |
| `size` | string | No | `"25"` | Items per page (max 25) |

**Example Request:**
```http
GET /api/collections/roots?sellerId=1&page=1&size=10
```

**Response:**
```json
{
  "success": true,
  "pagination": {
    "page": 1,
    "size": 10,
    "total": 15,
    "pages": 2
  },
  "data": [
    {
      "id": "45",
      "sellerId": "1",
      "shopifyCollectionId": "451263561983",
      "handle": "home",
      "title": "HOME",
      "bodyHtml": "",
      "collectionType": "CUSTOM",
      "published": true,
      "publishedAt": "2025-02-26T13:24:12.000Z",
      "sortOrder": "best-selling",
      "imageUrl": "https://cdn.shopify.com/..."
    }
  ]
}
```

---

## 3. Get Collection Tree

Get hierarchical tree structure for a seller's collections.

**Endpoint:** `GET /api/collections/tree/:sellerId`

**Authentication:** Public (No auth required)

**URL Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `sellerId` | string | Yes | Seller ID |

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | string | No | `"1"` | Page number for pagination |
| `size` | string | No | `"25"` | Items per page (max 25) |

**Example Request:**
```http
GET /api/collections/tree/1?page=1&size=10
```

**Response:**
```json
{
  "success": true,
  "count": 5,
  "pagination": {
    "page": 1,
    "size": 10,
    "total": 5,
    "pages": 1
  },
  "data": [
    {
      "id": "45",
      "title": "HOME",
      "handle": "home",
      "imageUrl": "https://cdn.shopify.com/...",
      "productCount": 25,
      "children": [
        {
          "id": "46",
          "title": "HOME ACCESSORIES",
          "handle": "home-accessories",
          "imageUrl": "https://cdn.shopify.com/...",
          "productCount": 10,
          "children": []
        }
      ]
    }
  ]
}
```

---

## 4. Get Collection by ID

Get a single collection by its ID with optional children and products.

**Endpoint:** `GET /api/collections/:id`

**Authentication:** Public (No auth required)

**URL Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Collection ID |

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `includeChildren` | string | No | - | If `"true"`, includes sub-collections |
| `includeProducts` | string | No | - | If `"true"`, includes products in collection |

**Example Request:**
```http
GET /api/collections/45?includeChildren=true
```

**Response:**
```json
{
  "success": true,
  "data": {
    "id": "45",
    "sellerId": "1",
    "shopifyCollectionId": "451263561983",
    "handle": "home",
    "title": "HOME",
    "bodyHtml": "",
    "collectionType": "CUSTOM",
    "published": true,
    "publishedAt": "2025-02-26T13:24:12.000Z",
    "sortOrder": "best-selling",
    "imageUrl": "https://cdn.shopify.com/...",
    "parent": null,
    "children": [
      {
        "id": "46",
        "title": "HOME ACCESSORIES",
        "_count": {
          "products": 10
        }
      }
    ],
    "_count": {
      "products": 25,
      "children": 3
    }
  }
}
```

---

## 5. Get Sub-Collections

Get all sub-collections (children) of a parent collection.

**Endpoint:** `GET /api/collections/:id/children`

**Authentication:** Public (No auth required)

**URL Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Parent collection ID |

**Query Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | string | No | `"1"` | Page number for pagination |
| `size` | string | No | `"25"` | Items per page (max 25) |

**Example Request:**
```http
GET /api/collections/45/children?page=1&size=10
```

**Response:**
```json
{
  "success": true,
  "count": 5,
  "pagination": {
    "page": 1,
    "size": 10,
    "total": 5,
    "pages": 1
  },
  "parent": {
    "id": "45",
    "title": "HOME",
    "handle": "home"
  },
  "data": [
    {
      "id": "46",
      "sellerId": "1",
      "shopifyCollectionId": "451867377919",
      "handle": "home-accessories",
      "title": "HOME ACCESSORIES",
      "bodyHtml": "",
      "collectionType": "CUSTOM",
      "published": true,
      "publishedAt": "2025-02-26T13:24:12.000Z",
      "sortOrder": "best-selling",
      "imageUrl": "https://cdn.shopify.com/...",
      "_count": {
        "products": 10,
        "children": 0
      }
    }
  ]
}
```

---

## 6. Get Collection Breadcrumb

Get the breadcrumb path (navigation trail) from root to the specified collection.

**Endpoint:** `GET /api/collections/:id/breadcrumb`

**Authentication:** Public (No auth required)

**URL Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Collection ID |

**Example Request:**
```http
GET /api/collections/123/breadcrumb
```

**Response:**
```json
{
  "success": true,
  "data": [
    {
      "id": "45",
      "title": "HOME",
      "handle": "home"
    },
    {
      "id": "46",
      "title": "HOME ACCESSORIES",
      "handle": "home-accessories"
    },
    {
      "id": "123",
      "title": "Samsung",
      "handle": "samsung"
    }
  ]
}
```

---

## 7. Set Collection Parent

Set or update the parent of a single collection.

**Endpoint:** `PATCH /api/collections/:id/parent`

**Authentication:** Required (Any authenticated user)

**Headers:**
```
Authorization: Bearer <token>
```

**URL Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `id` | string | Yes | Collection ID to update |

**Request Body:**
```json
{
  "parentId": "45"  // Parent collection ID, or null to make it root
}
```

**Example Request:**
```http
PATCH /api/collections/46/parent
Authorization: Bearer <token>
Content-Type: application/json

{
  "parentId": "45"
}
```

**Response:**
```json
{
  "success": true,
  "message": "Parent set successfully",
  "data": {
    "id": "46",
    "title": "HOME ACCESSORIES",
    "parentId": "45",
    "parent": {
      "id": "45",
      "title": "HOME"
    },
    "children": []
  }
}
```

**To remove parent (make root):**
```json
{
  "parentId": null
}
```

---

## 8. Set Collection Hierarchy

Set parent-child relationships for multiple collections (each can have different parent).

**Endpoint:** `POST /api/collections/hierarchy`

**Authentication:** Required (SELLER role only)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "hierarchy": [
    {
      "id": "46",
      "parentId": "45"
    },
    {
      "id": "47",
      "parentId": "45"
    },
    {
      "id": "48",
      "parentId": null
    }
  ]
}
```

**Example Request:**
```http
POST /api/collections/hierarchy
Authorization: Bearer <token>
Content-Type: application/json

{
  "hierarchy": [
    { "id": "46", "parentId": "45" },
    { "id": "47", "parentId": "45" },
    { "id": "48", "parentId": null }
  ]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Updated 3 collections",
  "updated": [
    { "id": "46", "parentId": "45", "success": true },
    { "id": "47", "parentId": "45", "success": true },
    { "id": "48", "parentId": null, "success": true }
  ],
  "errors": []
}
```

**Error Response (if any errors):**
```json
{
  "success": true,
  "message": "Updated 2 collections",
  "updated": [
    { "id": "46", "parentId": "45", "success": true },
    { "id": "47", "parentId": "45", "success": true }
  ],
  "errors": [
    { "id": "48", "error": "Collection not found" }
  ]
}
```

---

## 9. Set Parent Bulk

Set the same parent for multiple collections at once.

**Endpoint:** `POST /api/collections/set-parent-bulk`

**Authentication:** Required (Any authenticated user)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "parentId": "45",
  "childIds": ["46", "47", "48", "49", "50"]
}
```

**Example Request:**
```http
POST /api/collections/set-parent-bulk
Authorization: Bearer <token>
Content-Type: application/json

{
  "parentId": "45",
  "childIds": ["46", "47", "48", "49", "50"]
}
```

**Response:**
```json
{
  "success": true,
  "message": "Set 5 collections as children of \"HOME\"",
  "parent": {
    "id": "45",
    "title": "HOME"
  },
  "updated": [
    { "id": "46", "title": "HOME ACCESSORIES", "success": true },
    { "id": "47", "title": "HOME DECOR", "success": true },
    { "id": "48", "title": "Home Supplies", "success": true },
    { "id": "49", "title": "Household Appliances", "success": true },
    { "id": "50", "title": "interior car accessories", "success": true }
  ],
  "errors": []
}
```

---

## 10. Auto-Detect Hierarchy

Automatically detect parent-child relationships from collection naming patterns.

**Endpoint:** `POST /api/collections/auto-hierarchy`

**Authentication:** Required (SELLER role only)

**Headers:**
```
Authorization: Bearer <token>
```

**Request Body:**
```json
{
  "sellerId": "1",
  "useHandlePrefix": true,
  "separator": " > ",
  "handleSeparator": "-",
  "useHandle": false
}
```

**Request Body Parameters:**
| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `sellerId` | string/number | Yes | - | Seller ID |
| `useHandlePrefix` | boolean | No | `true` | Detect hierarchy from handle prefixes (e.g., "home-accessories" → parent "HOME") |
| `separator` | string | No | `" > "` | Separator used in titles (e.g., "Electronics > Laptops") |
| `handleSeparator` | string | No | `"-"` | Separator used in handles (e.g., "electronics-laptops") |
| `useHandle` | boolean | No | `false` | Use handle instead of title for detection |

**Example Request:**
```http
POST /api/collections/auto-hierarchy
Authorization: Bearer <token>
Content-Type: application/json

{
  "sellerId": "1",
  "useHandlePrefix": true
}
```

**Response:**
```json
{
  "success": true,
  "message": "Auto-detected hierarchy for 5 collections",
  "data": [
    {
      "id": "46",
      "title": "HOME ACCESSORIES",
      "handle": "home-accessories",
      "parentId": "45",
      "parentTitle": "HOME",
      "parentHandle": "home"
    },
    {
      "id": "47",
      "title": "HOME DECOR",
      "handle": "home-decor",
      "parentId": "45",
      "parentTitle": "HOME",
      "parentHandle": "home"
    }
  ]
}
```

**How it works:**
- Detects that collections with handles like `home-accessories`, `home-decor` should be children of the `HOME` collection
- Uses fuzzy matching to find parent collections
- Works best when collections follow naming conventions

---

## Error Responses

All endpoints may return the following error responses:

### 400 Bad Request
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "sellerId": "Seller ID is required"
  }
}
```

### 404 Not Found
```json
{
  "message": "Collection not found",
  "status_code": 404,
  "error": {}
}
```

### 401 Unauthorized
```json
{
  "message": "Unauthorized",
  "status_code": 401,
  "error": {}
}
```

### 403 Forbidden
```json
{
  "message": "Access denied. SELLER role required",
  "status_code": 403,
  "error": {}
}
```

### 500 Internal Server Error
```json
{
  "message": "Internal server error",
  "status_code": 500,
  "error": {}
}
```

---

## Data Types

### Collection Object
```typescript
{
  id: string;                    // BigInt as string
  sellerId: string;              // BigInt as string
  shopifyCollectionId: string;   // Shopify collection ID
  handle: string | null;          // URL-friendly identifier
  title: string;                  // Collection name
  bodyHtml: string | null;        // HTML description
  collectionType: "CUSTOM" | "SMART";
  published: boolean;
  publishedAt: string | null;     // ISO 8601 date
  sortOrder: string | null;
  imageUrl: string | null;
  parentId: string | null;       // Parent collection ID (if has parent)
}
```

### Pagination Object
```typescript
{
  page: number;
  size: number;
  total: number;
  pages: number;
}
```

---

## Notes for Flutter Developers

1. **BigInt Handling**: All IDs are returned as strings (not numbers) because they're BigInt values. Always treat them as strings.

2. **Pagination**: Use `page` and `size` query parameters. Maximum `size` is 25.

3. **Authentication**: Include Bearer token in headers:
   ```dart
   headers: {
     'Authorization': 'Bearer $token',
     'Content-Type': 'application/json',
   }
   ```

4. **Query Parameters**: All query parameters are strings, even numbers (e.g., `page="1"` not `page=1`).

5. **Boolean Query Parameters**: Use string `"true"` or `"false"` (e.g., `tree="true"`).

6. **Error Handling**: Always check the `success` field in responses. Handle errors based on `status_code`.

7. **Tree Structure**: When using `tree=true`, the response includes nested `children` arrays. Handle recursively in your UI.

8. **Breadcrumb**: Use the breadcrumb endpoint to show navigation paths like "Home > Electronics > Mobiles".

---

## Example Flutter Usage

```dart
// Get root collections
final response = await http.get(
  Uri.parse('$baseUrl/api/collections/roots?sellerId=1&page=1&size=10'),
);

// Get collection tree
final treeResponse = await http.get(
  Uri.parse('$baseUrl/api/collections/tree/1'),
);

// Set parent (authenticated)
final setParentResponse = await http.patch(
  Uri.parse('$baseUrl/api/collections/46/parent'),
  headers: {
    'Authorization': 'Bearer $token',
    'Content-Type': 'application/json',
  },
  body: jsonEncode({
    'parentId': '45',
  }),
);

// Auto-detect hierarchy (SELLER only)
final autoDetectResponse = await http.post(
  Uri.parse('$baseUrl/api/collections/auto-hierarchy'),
  headers: {
    'Authorization': 'Bearer $sellerToken',
    'Content-Type': 'application/json',
  },
  body: jsonEncode({
    'sellerId': '1',
    'useHandlePrefix': true,
  }),
);
```

---

## Version
**API Version:** 1.0  
**Last Updated:** 2025-11-29

