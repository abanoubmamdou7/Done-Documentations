# Banner API Documentation

Admin banner management system with public access for active banners.

## 📋 Overview

The banner system supports:
- **Admin banner management** - Create, update, delete banners (authenticated)
- **Public banner display** - Get active banners (no authentication required)
- **Multiple banner types** - HOME, PRODUCT, CATEGORY, PROMO
- **Display order control** - Custom ordering for banner display
- **Active/Inactive status** - Toggle banner visibility

## 🔐 Authentication

**Public Endpoints:**
- `GET /api/banners` - No authentication required

**Admin Endpoints:**
- All other endpoints require authentication with Bearer token:
```
Authorization: Bearer <your-token>
```

Allowed roles: `USER`, `SELLER`, `MEDIATOR`

---

## 📡 Banner Endpoints

### Get Active Banners (Public)

Get all active banners for display to users.

**Endpoint:** `GET /api/banners`

**Authentication:** ❌ None Required

**Query Parameters:**
- `type` (optional): Filter by banner type (`HOME`, `PRODUCT`, `CATEGORY`, `PROMO`)

**Example Request:**
```bash
# Get all active banners
GET /api/banners

# Get only HOME banners
GET /api/banners?type=HOME
```

**Response:**
```json
{
  "success": true,
  "count": 3,
  "data": {
    "banners": [
      {
        "id": "1",
        "title": "Summer Sale",
        "description": "Get 50% off on all items",
        "imageUrl": "https://example.com/banner1.jpg",
        "type": "PROMO",
        "linkType": null,
        "linkId": null,
        "order": 0
      },
      {
        "id": "2",
        "title": "New Arrivals",
        "description": "Check out our latest products",
        "imageUrl": "https://example.com/banner2.jpg",
        "type": "HOME",
        "linkType": null,
        "linkId": null,
        "order": 1
      },
      {
        "id": "3",
        "title": "iPhone 15 Pro",
        "description": "Pre-order now",
        "imageUrl": "https://example.com/banner3.jpg",
        "type": "PRODUCT",
        "linkType": "ORDER",
        "linkId": "order-iphone-123",
        "order": 2
      }
    ]
  }
}
```

**Notes:**
- Returns only active banners (`isActive: true`)
- Ordered by `order` field (ascending), then by creation date (descending)
- No authentication required - perfect for public display

---

### Get All Banners (Admin)

Get all banners with pagination and filters (includes inactive banners).

**Endpoint:** `GET /api/banners/admin`

**Authentication:** ✅ Required

**Query Parameters:**
- `type` (optional): Filter by type (`HOME`, `PRODUCT`, `CATEGORY`, `PROMO`)
- `isActive` (optional): Filter by status (`true` or `false`)
- `page` (optional): Page number (default: 1)
- `size` (optional): Items per page (default: 10)

**Example Request:**
```bash
# Get all banners (page 1)
GET /api/banners/admin?page=1&size=10

# Get only active HOME banners
GET /api/banners/admin?type=HOME&isActive=true
```

**Response:**
```json
{
  "success": true,
  "page": 1,
  "size": 10,
  "total": 5,
  "pages": 1,
  "data": {
    "banners": [
      {
        "id": "1",
        "title": "Summer Sale",
        "description": "Get 50% off on all items",
        "imageUrl": "https://example.com/banner1.jpg",
        "type": "PROMO",
        "isActive": true,
        "order": 0,
        "createdAt": "2024-01-01T00:00:00Z",
        "updatedAt": "2024-01-01T00:00:00Z"
      }
    ]
  }
}
```

---

### Get Banner by ID (Admin)

Get details of a specific banner.

**Endpoint:** `GET /api/banners/admin/:bannerId`

**Authentication:** ✅ Required

**Response:**
```json
{
  "success": true,
  "data": {
    "banner": {
      "id": "1",
      "title": "Summer Sale",
      "description": "Get 50% off on all items",
      "imageUrl": "https://example.com/banner1.jpg",
      "type": "PROMO",
      "isActive": true,
      "order": 0,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z"
    }
  }
}
```

**Error Responses:**
- `404`: Banner not found

---

### Create Banner (Admin)

Create a new banner with image upload.

**Endpoint:** `POST /api/banners`

**Authentication:** ✅ Required

**Content-Type:** `multipart/form-data`

**Request Body (Form-Data):**
- `bannerImage` (required): Image file (PNG, JPEG, JPG, WebP) - Max 5MB
- `title` (required): String, 1-255 characters
- `description` (optional): String, max 1000 characters
- `type` (optional): `HOME`, `PRODUCT`, `CATEGORY`, or `PROMO` (default: `HOME`)
- `linkType` (optional): `ORDER` or `CATEGORY` - Only for PRODUCT/CATEGORY banners
- `linkId` (optional): String, max 255 characters - Required if linkType is set
- `isActive` (optional): Boolean (default: `true`)
- `order` (optional): Integer ≥ 0 (default: `0`)

**Example using cURL:**
```bash
# Basic PROMO banner (no link)
curl -X POST http://localhost:3000/api/banners \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "bannerImage=@/path/to/image.jpg" \
  -F "title=Summer Sale" \
  -F "description=Get 50% off on all items" \
  -F "type=PROMO" \
  -F "isActive=true" \
  -F "order=0"

# PRODUCT banner with ORDER link
curl -X POST http://localhost:3000/api/banners \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "bannerImage=@/path/to/iphone.jpg" \
  -F "title=iPhone 15 Pro - Pre-Order" \
  -F "description=Order now for exclusive deals" \
  -F "type=PRODUCT" \
  -F "linkType=ORDER" \
  -F "linkId=order-iphone-123" \
  -F "isActive=true"
```

**Field Validation:**
- `bannerImage` (required): Image file (PNG, JPEG, JPG, WebP), max 5MB
- `title` (required): String, 1-255 characters
- `description` (optional): String, max 1000 characters
- `type` (optional): `HOME`, `PRODUCT`, `CATEGORY`, or `PROMO` (default: `HOME`)
- `linkType` (optional): `ORDER` or `CATEGORY` - Only allowed for PRODUCT/CATEGORY types
- `linkId` (optional): String, max 255 characters - Must be provided with linkType
- `isActive` (optional): Boolean (default: `true`)
- `order` (optional): Integer ≥ 0 (default: `0`)

**Response:**
```json
{
  "success": true,
  "message": "Banner created successfully",
  "data": {
    "banner": {
      "id": "1",
      "title": "iPhone 15 Pro - Pre-Order",
      "description": "Order now for exclusive deals",
      "imageUrl": "https://res.cloudinary.com/your-cloud/image/upload/v1234567890/DoneBuddy/banners/banner_1234567890.jpg",
      "type": "PRODUCT",
      "linkType": "ORDER",
      "linkId": "order-iphone-123",
      "isActive": true,
      "order": 0,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-01T00:00:00Z"
    }
  }
}
```

**Notes:**
- Image is automatically uploaded to Cloudinary
- Image URL is stored in database
- Supported formats: PNG, JPEG, JPG, WebP
- Maximum file size: 5MB
- `linkType` and `linkId` only allowed for PRODUCT/CATEGORY banners
- Both link fields must be provided together or both null

---

### Update Banner (Admin)

Update an existing banner.

**Endpoint:** `PATCH /api/banners/:bannerId`

**Authentication:** ✅ Required

**Content-Type:** `multipart/form-data`

**Request Body (Form-Data - all fields optional):**
- `bannerImage` (optional): New image file - will replace old image
- `title` (optional): String, 1-255 characters
- `description` (optional): String, max 1000 characters
- `type` (optional): `HOME`, `PRODUCT`, `CATEGORY`, or `PROMO`
- `linkType` (optional): `ORDER` or `CATEGORY` - Only for PRODUCT/CATEGORY banners
- `linkId` (optional): String, max 255 characters - Required if linkType is set
- `isActive` (optional): Boolean
- `order` (optional): Integer ≥ 0

**Example using cURL:**
```bash
# Update with new image
curl -X PATCH http://localhost:3000/api/banners/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "bannerImage=@/path/to/new-image.jpg" \
  -F "title=Winter Sale" \
  -F "order=5"

# Add clickable link to existing PRODUCT banner
curl -X PATCH http://localhost:3000/api/banners/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "linkType=ORDER" \
  -F "linkId=order-winter-sale-456"

# Remove link from banner
curl -X PATCH http://localhost:3000/api/banners/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "linkType=" \
  -F "linkId="

# Update without changing image
curl -X PATCH http://localhost:3000/api/banners/1 \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "title=Winter Sale" \
  -F "isActive=false"
```

**Notes:**
- At least one field must be provided
- Only provided fields will be updated
- If new image is uploaded, old image is automatically deleted from Cloudinary
- Link fields can be added, updated, or removed (set to empty/null)
- All validation rules from create apply

**Response:**
```json
{
  "success": true,
  "message": "Banner updated successfully",
  "data": {
    "banner": {
      "id": "1",
      "title": "Winter Sale",
      "description": "Updated description",
      "imageUrl": "https://example.com/new-banner.jpg",
      "type": "PRODUCT",
      "linkType": "ORDER",
      "linkId": "order-winter-sale-456",
      "isActive": false,
      "order": 5,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-02T00:00:00Z"
    }
  }
}
```

**Error Responses:**
- `404`: Banner not found
- `400`: No fields provided for update

---

### Toggle Banner Status (Admin)

Quick toggle between active and inactive status.

**Endpoint:** `PATCH /api/banners/:bannerId/toggle`

**Authentication:** ✅ Required

**Response:**
```json
{
  "success": true,
  "message": "Banner activated successfully",
  "data": {
    "banner": {
      "id": "1",
      "title": "Summer Sale",
      "isActive": true,
      "createdAt": "2024-01-01T00:00:00Z",
      "updatedAt": "2024-01-02T00:00:00Z"
    }
  }
}
```

**Notes:**
- Automatically toggles `isActive` to opposite value
- Useful for quick enable/disable without full update

---

### Delete Banner (Admin)

Permanently delete a banner.

**Endpoint:** `DELETE /api/banners/:bannerId`

**Authentication:** ✅ Required

**Response:**
```json
{
  "success": true,
  "message": "Banner deleted successfully"
}
```

**Error Responses:**
- `404`: Banner not found

**Warning:** This is a permanent deletion. Consider toggling `isActive` instead if you might need the banner later.

---

## 🎨 Banner Types

### HOME
- Homepage banners
- General promotional content
- Featured announcements
- **No clickable links allowed**

### PRODUCT
- Product-specific promotions
- New product launches
- Product highlights
- **Can link to orders or categories** ✨

### CATEGORY
- Category-level promotions
- Collection highlights
- Seasonal category banners
- **Can link to orders or categories** ✨

### PROMO
- Special offers
- Sales and discounts
- Limited-time deals
- **No clickable links allowed**

---

## 🔗 Clickable Links (NEW!)

PRODUCT and CATEGORY banners can include clickable links that direct users to specific content.

### Link Types

**ORDER Link:**
- Links to a specific product order page
- Perfect for "Buy Now" or "Pre-Order" banners
- Example: Direct link to iPhone 15 Pro order page

**CATEGORY Link:**
- Links to a specific category page
- Ideal for browsing collections
- Example: Link to "Electronics" category

### Link Fields

- **`linkType`** (optional): Either `ORDER` or `CATEGORY`
- **`linkId`** (optional): The ID of the order or category to link to

### Rules & Validation

✅ **Allowed:**
- PRODUCT banner with ORDER link
- PRODUCT banner with CATEGORY link
- CATEGORY banner with ORDER link
- CATEGORY banner with CATEGORY link
- Any banner with NO link (linkType and linkId both null)

❌ **Not Allowed:**
- HOME banner with any link
- PROMO banner with any link
- linkType without linkId (must provide both or neither)
- linkId without linkType (must provide both or neither)

### Example Usage

**Create PRODUCT Banner with Order Link:**
```json
{
  "title": "iPhone 15 Pro - Pre-Order Now",
  "type": "PRODUCT",
  "linkType": "ORDER",
  "linkId": "order-iphone-15-pro-123"
}
```

**Create CATEGORY Banner with Category Link:**
```json
{
  "title": "Gaming Accessories",
  "type": "CATEGORY",
  "linkType": "CATEGORY",
  "linkId": "gaming-category-456"
}
```

**Remove Link from Banner:**
```json
{
  "linkType": null,
  "linkId": null
}
```

---

## 📝 Implementation Notes

### Display Order
- Banners are sorted by `order` (ascending) then `createdAt` (descending)
- Lower `order` values appear first
- Same `order` values: newer banners appear first

### Active/Inactive Status
- `isActive: true` - Banner is visible to public
- `isActive: false` - Banner is hidden (admin only)
- Use toggle endpoint for quick status changes

### Image Upload
- Images uploaded via `bannerImage` field (multipart/form-data)
- Automatically uploaded to Cloudinary
- Stored in folder: `DoneBuddy/banners/`
- Supported formats: PNG, JPEG, JPG, WebP
- Maximum size: 5MB
- Old images automatically deleted when updating

### URL Fields
- `imageUrl` - Generated automatically from Cloudinary upload
- Stored in database for reference
- Served via Cloudinary CDN for fast delivery

### Pagination (Admin)
- Default page size: 10
- Maximum depends on your data
- Use `page` and `size` query parameters

---

## 🧪 Testing Guide

### Postman Collection

**1. Create Banner (Admin):**
```
POST /api/banners
Headers: 
  Authorization: Bearer <token>
  Content-Type: multipart/form-data
Form-Data:
  bannerImage: [Select Image File]
  title: "Test Banner"
  type: "HOME"
```

**2. Get Public Banners (No Auth):**
```
GET /api/banners
```

**3. Get Admin Banners:**
```
GET /api/banners/admin?page=1&size=10
Headers: 
  Authorization: Bearer <token>
```

**4. Toggle Status:**
```
PATCH /api/banners/1/toggle
Headers: 
  Authorization: Bearer <token>
```

**5. Update Banner:**
```
PATCH /api/banners/1
Headers: 
  Authorization: Bearer <token>
  Content-Type: multipart/form-data
Form-Data:
  bannerImage: [Select New Image File] (optional)
  title: "Updated Title"
  order: 5
```

**6. Delete Banner:**
```
DELETE /api/banners/1
Headers: 
  Authorization: Bearer <token>
```

---

## 🚀 Usage Examples

### Frontend Display (Public)
```javascript
// Get all active banners for homepage
fetch('/api/banners?type=HOME')
  .then(res => res.json())
  .then(data => {
    const banners = data.data.banners;
    // Display banners in carousel/slider
    // imageUrl is Cloudinary CDN URL - fast and optimized
  });
```

### Create Banner with File Upload (Admin)
```javascript
// Create banner with image upload
const formData = new FormData();
formData.append('bannerImage', imageFile); // File from input
formData.append('title', 'Summer Sale');
formData.append('description', 'Get 50% off!');
formData.append('type', 'PROMO');
formData.append('isActive', 'true');
formData.append('order', '0');

fetch('/api/banners', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${token}`
    // Don't set Content-Type - browser sets it automatically with boundary
  },
  body: formData
})
  .then(res => res.json())
  .then(data => {
    console.log('Banner created:', data.data.banner);
    // imageUrl will be Cloudinary URL
  });
```

### Admin Dashboard
```javascript
// Get all banners with filters
fetch('/api/banners/admin?page=1&size=20', {
  headers: {
    'Authorization': `Bearer ${token}`
  }
})
  .then(res => res.json())
  .then(data => {
    const banners = data.data.banners;
    // Display in admin table
  });
```

### Quick Toggle
```javascript
// Toggle banner status
fetch(`/api/banners/${bannerId}/toggle`, {
  method: 'PATCH',
  headers: {
    'Authorization': `Bearer ${token}`
  }
})
  .then(res => res.json())
  .then(data => {
    console.log(data.message); // "Banner activated successfully"
  });
```

---

## 🔗 Related Documentation

- [Authentication](./auth.md) - How to get authentication token
- [Admin](./admin.md) - Admin functionality overview

---

## 🐛 Error Codes

- `400`: Bad Request (invalid input, missing fields)
- `401`: Unauthorized (missing or invalid token)
- `404`: Not Found (banner not found)
- `500`: Internal Server Error

### Common Error Messages

**Banners:**
- `"Banner title is required"`
- `"Banner image is required"`
- `"File type '...' not allowed (dangerous extension)"`
- `"MIME type '...' is not allowed for field 'bannerImage'"`
- `"File too large"` (max 5MB)
- `"Type must be one of: HOME, PRODUCT, CATEGORY, PROMO"`
- `"Banner not found"`
- `"At least one field must be provided for update"`

---

**Version:** 1.0.0  
**Last Updated:** January 12, 2026
