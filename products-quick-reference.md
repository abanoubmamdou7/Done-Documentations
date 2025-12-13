# Products API - Quick Reference for Postman

## Base URL
```
http://localhost:3000/api/products
```
(Replace `3000` with your server port)

---

## 1. GET All Products

```
GET {{base_url}}/api/products
```

**Query Params:**
```
sellerId: 123
status: active
featuredInApp: true
hiddenInApp: false
search: laptop
page: 1
size: 25
```

**Full Example:**
```
GET {{base_url}}/api/products?page=1&size=25&status=active&featuredInApp=true
```

---

## 2. GET Product by ID

```
GET {{base_url}}/api/products/1
```

**With Query Params:**
```
GET {{base_url}}/api/products/1?includeVariants=true&includeImages=true&includeCollections=true
```

**Query Params:**
```
includeVariants: true
includeImages: true
includeCollections: true
```

---

## 3. Search Products

```
GET {{base_url}}/api/products/search?q=laptop
```

**Query Params:**
```
q: laptop (required)
sellerId: 123
collectionId: 456
minPrice: 50.00
maxPrice: 1000.00
page: 1
size: 25
```

**Full Example:**
```
GET {{base_url}}/api/products/search?q=laptop&sellerId=123&minPrice=50.00&maxPrice=1000.00&page=1&size=25
```

---

## 4. GET Products by Seller

```
GET {{base_url}}/api/products/seller/123
```

**Query Params:**
```
status: active
featuredInApp: true
hiddenInApp: false
search: shirt
page: 1
size: 25
```

**Full Example:**
```
GET {{base_url}}/api/products/seller/123?status=active&featuredInApp=true&page=1&size=10
```

---

## 5. GET Products by Collection

```
GET {{base_url}}/api/products/collection/456
```

**Query Params:**
```
page: 1
size: 25
```

**Full Example:**
```
GET {{base_url}}/api/products/collection/456?page=1&size=10
```

---

## 6. PATCH Update Product Feature Status

```
PATCH {{base_url}}/api/products/1/feature-status
```

**Headers:**
```
Authorization: Bearer YOUR_TOKEN_HERE
Content-Type: application/json
```

**Body (raw JSON):**
```json
{
  "featuredInApp": true,
  "hiddenInApp": false
}
```

**Body Examples:**

Set Featured:
```json
{
  "featuredInApp": true
}
```

Set Hidden:
```json
{
  "hiddenInApp": false
}
```

Update Both:
```json
{
  "featuredInApp": true,
  "hiddenInApp": false
}
```

---

## Status Values

- `status`: `"draft"` | `"active"` | `"archived"`
- `featuredInApp`: `"true"` | `"false"` (as string)
- `hiddenInApp`: `"true"` | `"false"` (as string)

---

## Quick Test URLs (Replace IDs with actual values)

### 1. Get All Products
```
GET http://localhost:3000/api/products
```

### 2. Get Product by ID
```
GET http://localhost:3000/api/products/1
```

### 3. Search Products
```
GET http://localhost:3000/api/products/search?q=laptop
```

### 4. Get Products by Seller
```
GET http://localhost:3000/api/products/seller/1
```

### 5. Get Products by Collection
```
GET http://localhost:3000/api/products/collection/1
```

### 6. Update Feature Status (Requires Auth)
```
PATCH http://localhost:3000/api/products/1/feature-status
Body: {"featuredInApp": true}
Headers: Authorization: Bearer YOUR_TOKEN
```

---

## Response Format

All successful responses follow this format:
```json
{
  "success": true,
  "pagination": {
    "page": 1,
    "size": 25,
    "total": 100,
    "pages": 4
  },
  "data": [...]
}
```

For single product (GET by ID):
```json
{
  "success": true,
  "data": {...}
}
```

