# Admin Banner Management API

This document covers the **Admin Banner Management** endpoints — creating, listing, updating, toggling, and deleting banners from the admin panel. Banners support image uploads via Cloudinary and can be linked to orders or categories.

**Base URL:** `/api/banners`

**Authentication:** Admin endpoints require `Authorization: Bearer <accessToken>` — obtained from `POST /api/admin/auth/login`.

**RBAC Resource Key:** `banners`

---

## Table of Contents

1. [Get Active Banners (Public)](#get-active-banners-public)
2. [List All Banners (Admin)](#list-all-banners-admin)
3. [Get Banner by ID (Admin)](#get-banner-by-id-admin)
4. [Create Banner](#create-banner)
5. [Update Banner](#update-banner)
6. [Toggle Banner Status](#toggle-banner-status)
7. [Delete Banner](#delete-banner)
8. [Enums & Schema Reference](#enums--schema-reference)
9. [Business Rules](#business-rules)
10. [Error Responses](#error-responses)

---

## Get Active Banners (Public)

**Endpoint:** `GET /api/banners`

**Authentication:** None — this is a public endpoint.

Returns all active banners, optionally filtered by type. Sorted by display order (ascending), then newest first.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `type` | String | No | — | Filter by banner type: `HOME`, `PRODUCT`, `CATEGORY`, `PROMO` |

**Example Request:**
```http
GET /api/banners?type=HOME
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "count": 2,
  "data": {
    "banners": [
      {
        "id": "1",
        "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "title": "Summer Sale",
        "description": "Up to 50% off on all items",
        "imageUrl": "https://res.cloudinary.com/.../banner_1234567890.jpg",
        "linkType": null,
        "linkId": null,
        "type": "HOME",
        "order": 0
      },
      {
        "id": "2",
        "uuid": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "title": "New Arrivals",
        "description": null,
        "imageUrl": "https://res.cloudinary.com/.../banner_1234567891.jpg",
        "linkType": "CATEGORY",
        "linkId": "electronics",
        "type": "CATEGORY",
        "order": 1
      }
    ]
  }
}
```

---

## List All Banners (Admin)

**Endpoint:** `GET /api/banners/admin`

**Authentication:** Required — `banners.read`

Returns a paginated list of all banners (active and inactive), with optional filters.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `size` | Integer | No | `10` | Items per page |
| `type` | String | No | — | Filter by type: `HOME`, `PRODUCT`, `CATEGORY`, `PROMO` |
| `isActive` | String | No | — | Filter by status: `true` or `false` |

**Example Request:**
```http
GET /api/banners/admin?type=HOME&isActive=true&page=1&size=10
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
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
        "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "title": "Summer Sale",
        "description": "Up to 50% off on all items",
        "imageUrl": "https://res.cloudinary.com/.../banner_1234567890.jpg",
        "type": "HOME",
        "isActive": true,
        "order": 0,
        "linkType": null,
        "linkId": null,
        "createdAt": "2026-01-10T10:00:00Z",
        "updatedAt": "2026-01-10T10:00:00Z"
      }
    ]
  }
}
```

---

## Get Banner by ID (Admin)

**Endpoint:** `GET /api/banners/admin/:bannerId`

**Authentication:** Required — `banners.read`

Returns full banner details by UUID.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `bannerId` | UUID | Yes | Banner UUID |

**Example Request:**
```http
GET /api/banners/admin/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "banner": {
      "id": "1",
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "Summer Sale",
      "description": "Up to 50% off on all items",
      "imageUrl": "https://res.cloudinary.com/.../banner_1234567890.jpg",
      "type": "HOME",
      "isActive": true,
      "order": 0,
      "linkType": null,
      "linkId": null,
      "createdAt": "2026-01-10T10:00:00Z",
      "updatedAt": "2026-01-10T10:00:00Z"
    }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | Banner not found |

---

## Create Banner

**Endpoint:** `POST /api/banners`

**Authentication:** Required — `banners.create`

Creates a new banner with an uploaded image. Sent as `multipart/form-data`.

**Request Body (form-data):**

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `bannerImage` | File | **Yes** | — | Banner image file (max 5 MB; JPEG, PNG, WebP, GIF) |
| `title` | String | **Yes** | — | Banner title (1–255 chars) |
| `description` | String | No | `null` | Banner description (max 1000 chars) |
| `type` | String | No | `HOME` | `HOME`, `PRODUCT`, `CATEGORY`, `PROMO` |
| `isActive` | Boolean | No | `true` | Whether the banner is active |
| `order` | Integer | No | `0` | Display order (lower = shown first) |
| `linkType` | String | No | `null` | `ORDER` or `CATEGORY` — only for `PRODUCT`/`CATEGORY` type banners |
| `linkId` | String | No | `null` | ID of the linked order or category (max 255 chars) |

**Example Request:**
```http
POST /api/banners
Authorization: Bearer <accessToken>
Content-Type: multipart/form-data

bannerImage: (binary file)
title: Summer Sale
description: Up to 50% off on all items
type: HOME
isActive: true
order: 0
```

**Example Response (201 Created):**
```json
{
  "success": true,
  "message": "Banner created successfully",
  "data": {
    "banner": {
      "id": "1",
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "Summer Sale",
      "description": "Up to 50% off on all items",
      "imageUrl": "https://res.cloudinary.com/.../banner_1234567890.jpg",
      "type": "HOME",
      "isActive": true,
      "order": 0,
      "linkType": null,
      "linkId": null,
      "createdAt": "2026-03-24T10:00:00Z",
      "updatedAt": "2026-03-24T10:00:00Z"
    }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Banner image is required |
| `400` | Link fields are only allowed for PRODUCT and CATEGORY banners |
| `400` | `linkId` is required when `linkType` is provided (and vice versa) |
| `422` | Validation error (title missing, type invalid, etc.) |

---

## Update Banner

**Endpoint:** `PATCH /api/banners/:bannerId`

**Authentication:** Required — `banners.update`

Updates an existing banner. All fields are optional. If a new `bannerImage` is uploaded, the old image is deleted from Cloudinary. Sent as `multipart/form-data`.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `bannerId` | UUID | Yes | Banner UUID |

**Request Body (form-data):**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `bannerImage` | File | No | New banner image (max 5 MB) — replaces the existing image |
| `title` | String | No | Updated title (1–255 chars) |
| `description` | String | No | Updated description (max 1000 chars, send empty to clear) |
| `type` | String | No | `HOME`, `PRODUCT`, `CATEGORY`, `PROMO` |
| `isActive` | Boolean | No | Toggle active status |
| `order` | Integer | No | Updated display order |
| `linkType` | String | No | `ORDER`, `CATEGORY`, or empty to clear |
| `linkId` | String | No | Updated link ID or empty to clear |

**Example Request:**
```http
PATCH /api/banners/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Authorization: Bearer <accessToken>
Content-Type: multipart/form-data

title: Updated Summer Sale
isActive: false
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Banner updated successfully",
  "data": {
    "banner": {
      "id": "1",
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "Updated Summer Sale",
      "description": "Up to 50% off on all items",
      "imageUrl": "https://res.cloudinary.com/.../banner_1234567890.jpg",
      "type": "HOME",
      "isActive": false,
      "order": 0,
      "linkType": null,
      "linkId": null,
      "createdAt": "2026-01-10T10:00:00Z",
      "updatedAt": "2026-03-24T10:30:00Z"
    }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Link fields are only allowed for PRODUCT and CATEGORY banners |
| `400` | Both `linkType` and `linkId` must be provided together or both null |
| `404` | Banner not found |
| `422` | Validation error |

---

## Toggle Banner Status

**Endpoint:** `PATCH /api/banners/:bannerId/toggle`

**Authentication:** Required — `banners.update`

Flips the banner's `isActive` flag. No request body required.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `bannerId` | UUID | Yes | Banner UUID |

**Example Request:**
```http
PATCH /api/banners/a1b2c3d4-e5f6-7890-abcd-ef1234567890/toggle
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Banner deactivated successfully",
  "data": {
    "banner": {
      "id": "1",
      "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "title": "Summer Sale",
      "isActive": false,
      "updatedAt": "2026-03-24T11:00:00Z"
    }
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | Banner not found |

---

## Delete Banner

**Endpoint:** `DELETE /api/banners/:bannerId`

**Authentication:** Required — `banners.delete`

Permanently deletes the banner and removes the image from Cloudinary.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `bannerId` | UUID | Yes | Banner UUID |

**Example Request:**
```http
DELETE /api/banners/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Banner deleted successfully"
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | Banner not found |

---

## Enums & Schema Reference

### BannerType

| Value | Description |
|-------|-------------|
| `HOME` | Homepage banner |
| `PRODUCT` | Product-related banner (supports linking) |
| `CATEGORY` | Category-related banner (supports linking) |
| `PROMO` | Promotional banner |

### BannerLinkType

| Value | Description |
|-------|-------------|
| `ORDER` | Links to a specific order |
| `CATEGORY` | Links to a product category |

### Banner Model

| Field | Type | Description |
|-------|------|-------------|
| `id` | BigInt | Auto-incremented primary key |
| `uuid` | UUID | Unique identifier used in URLs |
| `title` | String (255) | Banner title |
| `description` | Text | Optional description |
| `imageUrl` | Text | Cloudinary image URL |
| `type` | BannerType | Banner type (default: `HOME`) |
| `isActive` | Boolean | Whether the banner is visible publicly (default: `true`) |
| `order` | Integer | Display sort order (default: `0`, lower = first) |
| `linkType` | BannerLinkType | Optional link type for PRODUCT/CATEGORY banners |
| `linkId` | String (255) | Optional linked entity ID |
| `createdAt` | DateTime | Creation timestamp |
| `updatedAt` | DateTime | Last update timestamp |

---

## Business Rules

| Rule | Enforced at |
|------|-------------|
| Banner image is required on create | `POST /api/banners` controller |
| `linkType` and `linkId` must be provided together or both null | `POST` and `PATCH` controller |
| Link fields (`linkType`, `linkId`) only allowed for `PRODUCT` and `CATEGORY` type banners | `POST` and `PATCH` controller |
| Old image is deleted from Cloudinary when a new image is uploaded | `PATCH` controller |
| Image is deleted from Cloudinary on banner deletion | `DELETE` controller |
| Public endpoint only returns active banners | `GET /api/banners` query filter |
| Banners are ordered by `order` ASC, then `createdAt` DESC | All list endpoints |
| Max upload size is 5 MB | Multer middleware |
| UUIDs are used in all admin URL paths (not BigInt IDs) | All admin endpoints |

---

## Error Responses

| Status | Meaning |
|--------|---------|
| `400` | Business rule violation (see per-endpoint table) |
| `401` | Missing or invalid/expired admin token |
| `403` | RBAC permission denied (`banners.read`, `banners.create`, `banners.update`, or `banners.delete` required) |
| `404` | Banner not found |
| `422` | Validation error — response includes field-level detail |

**Validation error example (422):**
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "title": "Banner title is required",
    "type": "Type must be one of: HOME, PRODUCT, CATEGORY, PROMO"
  }
}
```

---

## Related Docs

- [banners.md](./banners.md) — Original banners reference.
- [admin-permissions.md](./admin-permissions.md) — RBAC, roles, and permission resources.
- [admin.md](./admin.md) — App-user management, moderation, and analytics.
