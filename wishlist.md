# 🎁 Wishlist API Documentation

Complete API documentation for the Wishlist endpoints in DoneBuddy.

---

## 📋 Overview

The Wishlist API allows users to:
- Add products to their wishlist
- Remove products from their wishlist
- View their own wishlist or other users' wishlists
- Generate shareable links to share wishlists with others

**Base URL:** `/api/wishlist`

---

## 🔐 Authentication

Most endpoints require authentication. Include the authorization token in the request headers:

```
Authorization: <your-jwt-token>
```

**Note:** The `GET /api/wishlist/shared/:shareToken` endpoint is public and does not require authentication.

---

## 📚 Endpoints

### 1. Add Item to Wishlist

Add a product to the authenticated user's wishlist.

**Endpoint:** `POST /api/wishlist`

**Access:** Private (Requires authentication)

**Request Body:**
```json
{
  "productId": "123456789"
}
```

**Response (201 Created):**
```json
{
  "success": true,
  "message": "Product added to wishlist successfully",
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "profileId": "user-profile-id",
    "productId": "123456789",
    "createdAt": "2025-12-28T10:00:00.000Z",
    "updatedAt": "2025-12-28T10:00:00.000Z",
    "product": {
      "id": "123456789",
      "title": "Product Name",
      "mainImageUrl": "https://example.com/image.jpg",
      "seller": {
        "id": "1",
        "shopName": "Seller Shop"
      },
      "images": [
        {
          "id": "1",
          "src": "https://example.com/image.jpg",
          "altText": "Product image",
          "position": 0
        }
      ],
      "variants": [
        {
          "id": "1",
          "price": "29.99",
          "currency": "USD"
        }
      ]
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Product ID is required
- `404 Not Found` - Product not found
- `409 Conflict` - Product is already in wishlist
- `401 Unauthorized` - Invalid or missing authentication token

---

### 2. Remove Item from Wishlist

Remove a product from the authenticated user's wishlist.

**Endpoint:** `DELETE /api/wishlist/:itemId`

**Access:** Private (Requires authentication)

**URL Parameters:**
- `itemId` (UUID, required) - The wishlist item ID to remove

**Example:**
```
DELETE /api/wishlist/550e8400-e29b-41d4-a716-446655440000
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Product removed from wishlist successfully"
}
```

**Error Responses:**
- `400 Bad Request` - Invalid item ID format
- `404 Not Found` - Wishlist item not found
- `403 Forbidden` - You can only remove your own wishlist items
- `401 Unauthorized` - Invalid or missing authentication token

---

### 3. Get User Wishlist

Retrieve all wishlist items for a specific user with pagination support.

**Endpoint:** `GET /api/wishlist/:userId`

**Access:** Private (Requires authentication)

**URL Parameters:**
- `userId` (UUID, required) - The user's profile ID

**Query Parameters:**
- `page` (string, optional) - Page number (default: "1")
- `size` (string, optional) - Items per page (default: "25")

**Example:**
```
GET /api/wishlist/550e8400-e29b-41d4-a716-446655440000?page=1&size=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": "1",
  "size": "10",
  "total": 15,
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "profileId": "user-profile-id",
      "productId": "123456789",
      "createdAt": "2025-12-28T10:00:00.000Z",
      "updatedAt": "2025-12-28T10:00:00.000Z",
      "product": {
        "id": "123456789",
        "title": "Product Name",
        "handle": "product-name",
        "bodyHtml": "<p>Product description</p>",
        "vendor": "Vendor Name",
        "productType": "Type",
        "status": "active",
        "mainImageUrl": "https://example.com/image.jpg",
        "seller": {
          "id": "1",
          "shopName": "Seller Shop",
          "shopifyDomain": "shop.myshopify.com"
        },
        "images": [
          {
            "id": "1",
            "src": "https://example.com/image.jpg",
            "altText": "Product image",
            "position": 0
          }
        ],
        "variants": [
          {
            "id": "1",
            "title": "Default Title",
            "price": "29.99",
            "currency": "USD",
            "compareAtPrice": "39.99",
            "inventoryQuantity": 10,
            "sku": "SKU-001"
          }
        ],
        "collections": [
          {
            "collection": {
              "id": "1",
              "title": "Collection Name",
              "handle": "collection-name"
            }
          }
        ],
        "_count": {
          "variants": 1,
          "images": 3
        }
      }
    }
  ]
}
```

**Error Responses:**
- `400 Bad Request` - Invalid user ID format
- `404 Not Found` - User not found
- `401 Unauthorized` - Invalid or missing authentication token

---

### 4. Share Wishlist

Generate a shareable link/token for a user's wishlist. Users can only share their own wishlist.

**Endpoint:** `POST /api/wishlist/share/:wishlistId`

**Access:** Private (Requires authentication)

**URL Parameters:**
- `wishlistId` (UUID, required) - The user's profile ID (same as userId, since each user has one wishlist)

**Note:** The `wishlistId` parameter represents the userId. Users can only share their own wishlist.

**Example:**
```
POST /api/wishlist/share/550e8400-e29b-41d4-a716-446655440000
```

**Response (200 OK):**
```json
{
  "success": true,
  "message": "Wishlist share link generated successfully",
  "data": {
    "shareToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "shareLink": "http://localhost:5000/api/wishlist/shared/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expiresIn": "30 days",
    "wishlistOwner": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "displayName": "User Name"
    }
  }
}
```

**Error Responses:**
- `400 Bad Request` - Invalid wishlist ID format
- `403 Forbidden` - You can only share your own wishlist
- `404 Not Found` - User not found
- `401 Unauthorized` - Invalid or missing authentication token

**Share Token Details:**
- Share tokens are JWT tokens that expire in 30 days
- The token contains the userId and a unique share ID
- Share tokens can be used with the `GET /api/wishlist/shared/:shareToken` endpoint

---

### 5. Get Shared Wishlist

Retrieve a wishlist using a share token. This endpoint is public and does not require authentication.

**Endpoint:** `GET /api/wishlist/shared/:shareToken`

**Access:** Public (No authentication required)

**URL Parameters:**
- `shareToken` (string, required) - Share token generated from `POST /api/wishlist/share/:wishlistId`

**Query Parameters:**
- `page` (string, optional) - Page number (default: "1")
- `size` (string, optional) - Items per page (default: "25")

**Example:**
```
GET /api/wishlist/shared/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...?page=1&size=10
```

**Response (200 OK):**
```json
{
  "success": true,
  "page": "1",
  "size": "10",
  "total": 15,
  "wishlistOwner": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "displayName": "User Name",
    "avatarUrl": "https://example.com/avatar.jpg"
  },
  "data": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "profileId": "user-profile-id",
      "productId": "123456789",
      "createdAt": "2025-12-28T10:00:00.000Z",
      "updatedAt": "2025-12-28T10:00:00.000Z",
      "product": {
        "id": "123456789",
        "title": "Product Name",
        "mainImageUrl": "https://example.com/image.jpg",
        "seller": {
          "id": "1",
          "shopName": "Seller Shop"
        },
        "images": [
          {
            "id": "1",
            "src": "https://example.com/image.jpg",
            "altText": "Product image",
            "position": 0
          }
        ],
        "variants": [
          {
            "id": "1",
            "title": "Default Title",
            "price": "29.99",
            "currency": "USD",
            "compareAtPrice": "39.99",
            "inventoryQuantity": 10
          }
        ],
        "collections": [
          {
            "collection": {
              "id": "1",
              "title": "Collection Name"
            }
          }
        ]
      }
    }
  ]
}
```

**Error Responses:**
- `400 Bad Request` - Invalid share token format
- `401 Unauthorized` - Invalid or expired share token

---

## 🔒 Security & Authorization

1. **Authentication Required**: Most endpoints require a valid JWT token in the Authorization header
2. **Ownership Validation**: Users can only remove their own wishlist items
3. **Share Restrictions**: Users can only share their own wishlist
4. **Share Token Expiry**: Share tokens expire after 30 days
5. **Public Access**: The shared wishlist endpoint is public but requires a valid share token

---

## 📝 Data Models

### WishlistItem
```typescript
{
  id: string (UUID)
  profileId: string (UUID)
  productId: string (BigInt as string)
  createdAt: string (ISO 8601 date)
  updatedAt: string (ISO 8601 date)
  product: Product (with seller, images, variants, collections)
}
```

---

## 🎯 Usage Examples

### Example 1: Add Product to Wishlist

```bash
curl -X POST http://localhost:5000/api/wishlist \
  -H "Authorization: your-jwt-token" \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "123456789"
  }'
```

### Example 2: Get User's Wishlist

```bash
curl -X GET "http://localhost:5000/api/wishlist/550e8400-e29b-41d4-a716-446655440000?page=1&size=10" \
  -H "Authorization: your-jwt-token"
```

### Example 3: Remove Item from Wishlist

```bash
curl -X DELETE http://localhost:5000/api/wishlist/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: your-jwt-token"
```

### Example 4: Share Wishlist

```bash
curl -X POST http://localhost:5000/api/wishlist/share/550e8400-e29b-41d4-a716-446655440000 \
  -H "Authorization: your-jwt-token"
```

### Example 5: View Shared Wishlist (No Auth Required)

```bash
curl -X GET "http://localhost:5000/api/wishlist/shared/eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...?page=1&size=10"
```

---

## ⚠️ Important Notes

1. **BigInt Serialization**: Product IDs are stored as BigInt in the database but are serialized as strings in JSON responses
2. **Pagination**: All list endpoints support pagination with `page` and `size` query parameters
3. **Unique Constraint**: Each user can only add a product to their wishlist once (enforced by unique constraint on `profileId_productId`)
4. **Cascade Deletion**: Wishlist items are automatically deleted when:
   - The user (profile) is deleted
   - The product is deleted
5. **Share Token Format**: Share tokens are JWT tokens and should be included in the URL path, not as a query parameter

---

## 🐛 Error Handling

All endpoints follow consistent error response format:

```json
{
  "success": false,
  "message": "Error message",
  "statusCode": 400
}
```

Common HTTP status codes:
- `200` - Success
- `201` - Created
- `400` - Bad Request (validation error, missing parameters)
- `401` - Unauthorized (invalid or missing token)
- `403` - Forbidden (insufficient permissions)
- `404` - Not Found (resource doesn't exist)
- `409` - Conflict (resource already exists)
- `500` - Internal Server Error

---

## 🔄 Related APIs

- **Products API**: `/api/products` - Manage products
- **Bookmarks API**: `/api/bookmarks` - Save/bookmark posts (different from wishlist)
- **Cart API**: `/api/cart` - Shopping cart functionality

---

## 📅 Changelog

### Version 1.0.0 (2025-12-28)
- Initial release
- Add item to wishlist
- Remove item from wishlist
- Get user wishlist
- Share wishlist
- Get shared wishlist

