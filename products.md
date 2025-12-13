# Products API - Postman Testing Guide

Base URL: `http://localhost:YOUR_PORT/api/products` (or your server URL)

---

## 1. Get All Products

**Method:** `GET`  
**Endpoint:** `/api/products`  
**Access:** Public  
**Description:** Retrieve all products with optional filtering and pagination

### Query Parameters:
```
sellerId: string (optional)
collectionId: string (optional)
status: "draft" | "active" | "archived" (optional)
featuredInApp: "true" | "false" (optional)
hiddenInApp: "true" | "false" (optional)
search: string (optional)
page: string (optional, default: "1")
size: string (optional, default: "25")
```

### Example Request:
```
GET http://localhost:3000/api/products?page=1&size=25
```

### Example Request with Filters:
```
GET http://localhost:3000/api/products?sellerId=123&status=active&featuredInApp=true&page=1&size=10
```

### Example Request with Search:
```
GET http://localhost:3000/api/products?search=laptop&page=1&size=25
```

### Response Example:
```json
{
  "success": true,
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 100,
    "pages": 4
  },
  "data": [
    {
      "id": "1",
      "sellerId": "123",
      "shopifyProductId": "gid://shopify/Product/456",
      "handle": "product-handle",
      "title": "Product Title",
      "bodyHtml": "<p>Product description</p>",
      "vendor": "Vendor Name",
      "productType": "Product Type",
      "status": "active",
      "tags": ["tag1", "tag2"],
      "mainImageUrl": "https://example.com/image.jpg",
      "isGiftCard": false,
      "featuredInApp": true,
      "hiddenInApp": false,
      "createdAt": "2024-01-01T00:00:00.000Z",
      "updatedAt": "2024-01-01T00:00:00.000Z",
      "seller": {
        "id": "123",
        "shopName": "Shop Name"
      },
      "_count": {
        "variants": 3,
        "images": 5,
        "collections": 2
      }
    }
  ]
}
```

---

## 2. Get Product by ID

**Method:** `GET`  
**Endpoint:** `/api/products/:id`  
**Access:** Public  
**Description:** Retrieve detailed information about a specific product

### URL Parameters:
```
id: string (required) - Product ID
```

### Query Parameters:
```
includeVariants: "true" | "false" (optional)
includeImages: "true" | "false" (optional)
includeCollections: "true" | "false" (optional)
```

### Example Request (Basic):
```
GET http://localhost:3000/api/products/1
```

### Example Request (With All Includes):
```
GET http://localhost:3000/api/products/1?includeVariants=true&includeImages=true&includeCollections=true
```

### Response Example:
```json
{
  "success": true,
  "data": {
    "id": "1",
    "sellerId": "123",
    "shopifyProductId": "gid://shopify/Product/456",
    "handle": "product-handle",
    "title": "Product Title",
    "bodyHtml": "<p>Product description</p>",
    "vendor": "Vendor Name",
    "productType": "Product Type",
    "status": "active",
    "tags": ["tag1", "tag2"],
    "mainImageUrl": "https://example.com/image.jpg",
    "isGiftCard": false,
    "featuredInApp": true,
    "hiddenInApp": false,
    "createdAt": "2024-01-01T00:00:00.000Z",
    "updatedAt": "2024-01-01T00:00:00.000Z",
    "seller": {
      "id": "123",
      "shopName": "Shop Name"
    },
    "variants": [
      {
        "id": "1",
        "title": "Default Title",
        "price": "29.99",
        "compareAtPrice": "39.99",
        "sku": "SKU123",
        "inventoryQuantity": 100
      }
    ],
    "images": [
      {
        "id": "1",
        "src": "https://example.com/image1.jpg",
        "altText": "Product Image",
        "position": 1
      }
    ],
    "collections": [
      {
        "collection": {
          "id": "1",
          "title": "Collection Name",
          "handle": "collection-handle",
          "imageUrl": "https://example.com/collection-image.jpg"
        },
        "position": 0
      }
    ],
    "_count": {
      "variants": 3,
      "images": 5,
      "collections": 2
    }
  }
}
```

---

## 3. Search Products

**Method:** `GET`  
**Endpoint:** `/api/products/search`  
**Access:** Public  
**Description:** Search products with advanced filtering including price range

### Query Parameters:
```
q: string (required) - Search query
sellerId: string (optional)
collectionId: string (optional)
minPrice: string (optional) - e.g., "10.99"
maxPrice: string (optional) - e.g., "100.50"
page: string (optional, default: "1")
size: string (optional, default: "25")
```

### Example Request (Basic Search):
```
GET http://localhost:3000/api/products/search?q=laptop
```

### Example Request (Advanced Search):
```
GET http://localhost:3000/api/products/search?q=laptop&sellerId=123&minPrice=50.00&maxPrice=1000.00&page=1&size=20
```

### Example Request (Search with Collection Filter):
```
GET http://localhost:3000/api/products/search?q=shirt&collectionId=456&page=1&size=25
```

### Response Example:
```json
{
  "success": true,
  "query": "laptop",
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 15,
    "pages": 1
  },
  "data": [
    {
      "id": "1",
      "title": "Gaming Laptop",
      "vendor": "Tech Brand",
      "mainImageUrl": "https://example.com/laptop.jpg",
      "variants": [
        {
          "id": "1",
          "price": "999.99",
          "compareAtPrice": "1299.99",
          "currency": "USD",
          "inventoryQuantity": 10
        }
      ],
      "_count": {
        "variants": 2,
        "images": 4,
        "collections": 1
      }
    }
  ]
}
```

---

## 4. Get Products by Seller

**Method:** `GET`  
**Endpoint:** `/api/products/seller/:sellerId`  
**Access:** Public  
**Description:** Get all products belonging to a specific seller

### URL Parameters:
```
sellerId: string (required) - Seller ID
```

### Query Parameters:
```
status: "draft" | "active" | "archived" (optional)
featuredInApp: "true" | "false" (optional)
hiddenInApp: "true" | "false" (optional)
search: string (optional)
page: string (optional, default: "1")
size: string (optional, default: "25")
```

### Example Request (Basic):
```
GET http://localhost:3000/api/products/seller/123
```

### Example Request (With Filters):
```
GET http://localhost:3000/api/products/seller/123?status=active&featuredInApp=true&page=1&size=10
```

### Example Request (With Search):
```
GET http://localhost:3000/api/products/seller/123?search=shirt&status=active&page=1&size=25
```

### Response Example:
```json
{
  "success": true,
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 50,
    "pages": 2
  },
  "data": [
    {
      "id": "1",
      "sellerId": "123",
      "title": "Product Title",
      "status": "active",
      "featuredInApp": true,
      "_count": {
        "variants": 3,
        "images": 5,
        "collections": 2
      }
    }
  ]
}
```

---

## 5. Get Products by Collection

**Method:** `GET`  
**Endpoint:** `/api/products/collection/:collectionId`  
**Access:** Public  
**Description:** Get all products in a specific collection

### URL Parameters:
```
collectionId: string (required) - Collection ID
```

### Query Parameters:
```
page: string (optional, default: "1")
size: string (optional, default: "25")
```

### Example Request:
```
GET http://localhost:3000/api/products/collection/456
```

### Example Request (With Pagination):
```
GET http://localhost:3000/api/products/collection/456?page=2&size=10
```

### Response Example:
```json
{
  "success": true,
  "collection": {
    "id": "456",
    "title": "Summer Collection",
    "handle": "summer-collection"
  },
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 30,
    "pages": 2
  },
  "data": [
    {
      "id": "1",
      "title": "Product Title",
      "position": 0,
      "_count": {
        "variants": 3,
        "images": 5,
        "collections": 2
      }
    }
  ]
}
```

---

## 6. Update Product Feature Status

**Method:** `PATCH`  
**Endpoint:** `/api/products/:id/feature-status`  
**Access:** Private (Requires Authentication)  
**Description:** Update the featured or hidden status of a product

### Headers:
```
Authorization: Bearer YOUR_TOKEN_HERE
Content-Type: application/json
```

### URL Parameters:
```
id: string (required) - Product ID
```

### Request Body:
```json
{
  "featuredInApp": true,
  "hiddenInApp": false
}
```

**Note:** At least one field (`featuredInApp` or `hiddenInApp`) must be provided.

### Example Request (Set Featured):
```
PATCH http://localhost:3000/api/products/1/feature-status

Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  Content-Type: application/json

Body:
{
  "featuredInApp": true
}
```

### Example Request (Set Hidden):
```
PATCH http://localhost:3000/api/products/1/feature-status

Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  Content-Type: application/json

Body:
{
  "hiddenInApp": false
}
```

### Example Request (Update Both):
```
PATCH http://localhost:3000/api/products/1/feature-status

Headers:
  Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
  Content-Type: application/json

Body:
{
  "featuredInApp": true,
  "hiddenInApp": false
}
```

### Response Example:
```json
{
  "success": true,
  "message": "Product feature status updated successfully",
  "data": {
    "id": "1",
    "title": "Product Title",
    "featuredInApp": true,
    "hiddenInApp": false,
    "updatedAt": "2024-01-01T12:00:00.000Z"
  }
}
```

---

## Postman Collection Setup Tips

### 1. Create Environment Variables:
- `base_url`: `http://localhost:3000` (or your server URL)
- `auth_token`: Your authentication token
- `product_id`: Example product ID for testing
- `seller_id`: Example seller ID for testing
- `collection_id`: Example collection ID for testing

### 2. Pre-request Script for Authenticated Requests:
Add this to requests that require authentication:
```javascript
pm.request.headers.add({
    key: 'Authorization',
    value: 'Bearer ' + pm.environment.get('auth_token')
});
```

### 3. Collection Structure:
```
Products API
├── 1. Get All Products
│   ├── Basic
│   ├── With Filters
│   └── With Search
├── 2. Get Product by ID
│   ├── Basic
│   └── With All Includes
├── 3. Search Products
│   ├── Basic Search
│   └── Advanced Search
├── 4. Get Products by Seller
│   ├── Basic
│   └── With Filters
├── 5. Get Products by Collection
│   └── Basic
└── 6. Update Product Feature Status
    ├── Set Featured
    ├── Set Hidden
    └── Update Both
```

---

## Common Response Codes

- `200 OK` - Request successful
- `400 Bad Request` - Invalid request parameters
- `401 Unauthorized` - Missing or invalid authentication token
- `403 Forbidden` - Not authorized to perform action
- `404 Not Found` - Resource not found
- `422 Validation Error` - Request validation failed
- `500 Internal Server Error` - Server error

---

## Testing Checklist

- [ ] Test all GET endpoints without authentication
- [ ] Test GET endpoints with various query parameters
- [ ] Test pagination (page, size)
- [ ] Test filtering options
- [ ] Test search functionality
- [ ] Test PATCH endpoint with authentication
- [ ] Test PATCH endpoint without authentication (should fail)
- [ ] Test invalid IDs (should return 404)
- [ ] Test invalid query parameters (should return 422)
- [ ] Test empty results (should return empty array with pagination)

---

## Notes

1. All pagination defaults: `page=1`, `size=25`
2. Maximum page size is limited by the pagination utility (typically 25)
3. All BigInt values are automatically serialized to strings in responses
4. Date values are returned in ISO 8601 format
5. Search is case-insensitive
6. All query parameters are optional unless marked as required

