# Admin Order Management API

This document covers the **Admin Order Management** endpoints — listing, viewing, and managing orders from the admin panel, including status updates, payment tracking, fulfillment, assignment, notes, timeline, and bulk operations.

**Base URL:** `/api/admin`

**Authentication:** All endpoints require `Authorization: Bearer <accessToken>` — obtained from `POST /api/admin/auth/login`.

**RBAC Resource Key:** `orders`

---

## Table of Contents

1. [List Orders](#list-orders)
2. [Get Order Detail](#get-order-detail)
3. [Update Order Status](#update-order-status)
4. [Update Payment Status](#update-payment-status)
5. [Update Fulfillment Status](#update-fulfillment-status)
6. [Assign Order](#assign-order)
7. [Update Priority](#update-priority)
8. [Update Tags](#update-tags)
9. [Mark Fraud Review](#mark-fraud-review)
10. [Cancel Order](#cancel-order)
11. [Initiate Refund](#initiate-refund)
12. [Notes](#notes)
13. [Timeline](#timeline)
14. [Bulk Operations](#bulk-operations)
15. [Status Transition Rules](#status-transition-rules)
16. [Business Rules](#business-rules)
17. [Error Responses](#error-responses)

---

## List Orders

**Endpoint:** `GET /api/admin/orders`

**Authentication:** Required — `orders.read`

Returns a paginated, filterable, sortable list of orders.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `limit` | Integer | No | `20` | Items per page (max 100) |
| `search` | String | No | — | Search by order name, Shopify order ID, buyer name, customer email/phone, or order number |
| `status` | String | No | — | `PENDING`, `PAID`, `SHIPPED`, `COMPLETED`, `CANCELLED` |
| `paymentStatus` | String | No | — | `UNPAID`, `PAID`, `PARTIALLY_PAID`, `REFUNDED`, `PARTIALLY_REFUNDED` |
| `fulfillmentStatus` | String | No | — | `UNFULFILLED`, `PARTIALLY_FULFILLED`, `FULFILLED` |
| `source` | String | No | — | Order source identifier |
| `assignedTo` | UUID | No | — | Filter by assigned admin ID |
| `priority` | String | No | — | `LOW`, `MEDIUM`, `HIGH`, `URGENT` |
| `customerId` | String | No | — | Filter by customer ID |
| `fromDate` | ISO Date | No | — | Filter orders created on or after this date |
| `toDate` | ISO Date | No | — | Filter orders created on or before this date |
| `minTotal` | Number | No | — | Minimum order total |
| `maxTotal` | Number | No | — | Maximum order total |
| `sortBy` | String | No | `createdAt` | `createdAt`, `updatedAt`, `total`, `status`, `priority` |
| `sortOrder` | String | No | `desc` | `asc` or `desc` |

**Example Request:**
```http
GET /api/admin/orders?status=PENDING&priority=HIGH&page=1&limit=20&sortBy=createdAt&sortOrder=desc
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Orders retrieved",
  "data": {
    "orders": [
      {
        "id": "12345",
        "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
        "shopifyOrderNumber": 1001,
        "shopifyOrderName": "#1001",
        "total": "149.99",
        "currency": "USD",
        "status": "PENDING",
        "paymentStatus": "UNPAID",
        "fulfillmentStatus": "UNFULFILLED",
        "priority": "HIGH",
        "source": "shopify",
        "tags": ["vip"],
        "assignedToAdminId": null,
        "assignedToAdmin": null,
        "buyer": {
          "id": "98765",
          "displayName": "John Buyer",
          "avatarUrl": null,
          "phone": "+1234567890"
        },
        "customer": {
          "id": "11111",
          "firstName": "John",
          "lastName": "Doe",
          "email": "john@example.com",
          "phone": "+1234567890"
        },
        "_count": { "items": 3, "notes": 0 },
        "createdAt": "2026-03-15T10:00:00Z",
        "updatedAt": "2026-03-15T10:00:00Z"
      }
    ],
    "total": 42,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

---

## Get Order Detail

**Endpoint:** `GET /api/admin/orders/:uuid`

**Authentication:** Required — `orders.read`

Returns full order detail including items, seller, and product variant information.

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | UUID | Yes | Order UUID (not the BigInt `id`) |

**Example Request:**
```http
GET /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Order retrieved",
  "data": {
    "id": "12345",
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "shopifyOrderNumber": 1001,
    "shopifyOrderName": "#1001",
    "shopifyOrderId": "gid://shopify/Order/12345",
    "total": "149.99",
    "currency": "USD",
    "status": "PENDING",
    "paymentStatus": "UNPAID",
    "fulfillmentStatus": "UNFULFILLED",
    "priority": "HIGH",
    "source": "shopify",
    "tags": ["vip"],
    "cancelReason": null,
    "assignedToAdminId": null,
    "assignedToAdmin": null,
    "buyer": { "id": "98765", "displayName": "John Buyer", "avatarUrl": null, "phone": "+1234567890" },
    "customer": { "id": "11111", "firstName": "John", "lastName": "Doe", "email": "john@example.com", "phone": "+1234567890" },
    "seller": {
      "id": "55555",
      "shopName": "My Store",
      "shopifyDomain": "mystore.myshopify.com",
      "shopEmail": "shop@example.com",
      "shopCurrency": "USD"
    },
    "items": [
      {
        "id": "999",
        "quantity": 2,
        "price": "49.99",
        "productVariant": {
          "id": "888",
          "title": "Blue / L",
          "sku": "SHIRT-BLU-L",
          "price": "49.99",
          "currency": "USD",
          "product": {
            "id": "777",
            "title": "Classic T-Shirt",
            "mainImageUrl": "https://cdn.shopify.com/...",
            "vendor": "BrandName"
          }
        }
      }
    ],
    "_count": { "items": 1, "notes": 2 },
    "createdAt": "2026-03-15T10:00:00Z",
    "updatedAt": "2026-03-15T10:00:00Z"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `404` | Order not found |

---

## Update Order Status

**Endpoint:** `PATCH /api/admin/orders/:uuid/status`

**Authentication:** Required — `orders.update`

Transition the order to a new status. Only valid transitions are accepted (see [Status Transition Rules](#status-transition-rules)).

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | UUID | Yes | Order UUID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | String | Yes | Target status: `PENDING`, `PAID`, `SHIPPED`, `COMPLETED`, `CANCELLED` |
| `reason` | String | No | Optional reason (stored in `cancelReason` if status is `CANCELLED`) |

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/status
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "status": "PAID",
  "reason": "Payment confirmed via bank transfer"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Order status updated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "PAID",
    "paymentStatus": "UNPAID",
    "updatedAt": "2026-03-15T10:30:00Z"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Invalid status transition (e.g. PENDING → COMPLETED) |
| `404` | Order not found |

---

## Update Payment Status

**Endpoint:** `PATCH /api/admin/orders/:uuid/payment-status`

**Authentication:** Required — `orders.update`

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `uuid` | UUID | Yes | Order UUID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `paymentStatus` | String | Yes | `UNPAID`, `PAID`, `PARTIALLY_PAID`, `REFUNDED`, `PARTIALLY_REFUNDED` |
| `reason` | String | No | Optional reason for the change |

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/payment-status
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "paymentStatus": "PAID",
  "reason": "COD collected by driver"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Payment status updated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "paymentStatus": "PAID",
    "updatedAt": "2026-03-15T10:35:00Z"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Invalid payment status transition |
| `404` | Order not found |

---

## Update Fulfillment Status

**Endpoint:** `PATCH /api/admin/orders/:uuid/shipping-status`

**Authentication:** Required — `orders.update`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `fulfillmentStatus` | String | Yes | `UNFULFILLED`, `PARTIALLY_FULFILLED`, `FULFILLED` |

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/shipping-status
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "fulfillmentStatus": "FULFILLED"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Fulfillment status updated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "fulfillmentStatus": "FULFILLED",
    "updatedAt": "2026-03-15T11:00:00Z"
  }
}
```

---

## Assign Order

**Endpoint:** `PATCH /api/admin/orders/:uuid/assign`

**Authentication:** Required — `orders.update`

Assign the order to an admin user for handling. Pass `null` to unassign.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `assignedToAdminId` | UUID or null | No | Admin user UUID. `null` removes assignment |

**Example Request — assign:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/assign
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "assignedToAdminId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Example Request — unassign:**
```http
{
  "assignedToAdminId": null
}
```

**Example Response (200 OK):**
```json
{
  "message": "Order assigned",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "assignedToAdminId": "550e8400-e29b-41d4-a716-446655440000",
    "assignedToAdmin": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "fullName": "Alice Smith",
      "email": "alice@example.com",
      "avatarUrl": null
    }
  }
}
```

---

## Update Priority

**Endpoint:** `PATCH /api/admin/orders/:uuid/priority`

**Authentication:** Required — `orders.update`

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `priority` | String | Yes | `LOW`, `MEDIUM`, `HIGH`, `URGENT` |

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/priority
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "priority": "URGENT"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Order priority updated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "priority": "URGENT"
  }
}
```

---

## Update Tags

**Endpoint:** `PATCH /api/admin/orders/:uuid/tags`

**Authentication:** Required — `orders.update`

Replaces the entire tags array. Send an empty array `[]` to remove all tags.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tags` | String[] | Yes | Array of tag strings (max 20 tags, each max 100 chars) |

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/tags
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "tags": ["vip", "gift", "express-shipping"]
}
```

**Example Response (200 OK):**
```json
{
  "message": "Order tags updated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "tags": ["vip", "gift", "express-shipping"]
  }
}
```

---

## Mark Fraud Review

**Endpoint:** `PATCH /api/admin/orders/:uuid/mark-fraud-review`

**Authentication:** Required — `orders.update`

Convenience endpoint that sets priority to `URGENT` and appends the tag `fraud-review`. No body required. Idempotent — calling it multiple times does not duplicate the tag.

**Example Request:**
```http
PATCH /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/mark-fraud-review
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Order flagged for fraud review",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "priority": "URGENT",
    "tags": ["fraud-review"]
  }
}
```

---

## Cancel Order

**Endpoint:** `POST /api/admin/orders/:uuid/cancel`

**Authentication:** Required — `orders.update`

Cancels the order by setting status to `CANCELLED`. Cannot cancel a completed or already-cancelled order.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `reason` | String | No | Cancellation reason (stored in `cancelReason`, max 500 chars) |

**Example Request:**
```http
POST /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/cancel
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "reason": "Customer requested cancellation"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Order cancelled",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "status": "CANCELLED",
    "cancelReason": "Customer requested cancellation"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Order is already cancelled |
| `400` | Cannot cancel a completed order |
| `404` | Order not found |

---

## Initiate Refund

**Endpoint:** `POST /api/admin/orders/:uuid/refund`

**Authentication:** Required — `orders.update`

Records a refund for the order and updates `paymentStatus` automatically:
- Full refund (no `amount` or `amount` = total) → `REFUNDED`
- Partial refund (`amount` < total) → `PARTIALLY_REFUNDED`

> **Note:** This records the refund in the admin system. Actual payment gateway refunds must be handled separately in Shopify.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `amount` | Number | No | Refund amount. Omit for full refund. Must not exceed order total |
| `reason` | String | No | Reason for refund (max 500 chars) |
| `refundType` | String | No | `full` or `partial` |

**Example Request — full refund:**
```http
POST /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/refund
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "reason": "Defective item",
  "refundType": "full"
}
```

**Example Request — partial refund:**
```http
{
  "amount": 50.00,
  "reason": "Item arrived damaged, partial compensation agreed",
  "refundType": "partial"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Refund initiated",
  "data": {
    "uuid": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "paymentStatus": "REFUNDED"
  }
}
```

**Error Responses:**

| Code | Reason |
|------|--------|
| `400` | Cannot refund an unpaid order |
| `400` | Order has already been fully refunded |
| `400` | Refund amount exceeds order total |
| `404` | Order not found |

---

## Notes

### Add Note

**Endpoint:** `POST /api/admin/orders/:uuid/notes`

**Authentication:** Required — `orders.update`

Adds an internal note to the order. Notes are attributed to the authenticated admin.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `note` | String | Yes | Note content (1–2000 chars) |

**Example Request:**
```http
POST /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/notes
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "note": "Customer called — confirmed delivery address. Driver will call 30 mins before arrival."
}
```

**Example Response (201 Created):**
```json
{
  "message": "Note added",
  "data": {
    "id": "note-uuid-here",
    "note": "Customer called — confirmed delivery address. Driver will call 30 mins before arrival.",
    "isInternal": true,
    "createdAt": "2026-03-15T11:30:00Z",
    "updatedAt": "2026-03-15T11:30:00Z",
    "authorAdmin": {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "fullName": "Alice Smith",
      "email": "alice@example.com",
      "avatarUrl": null
    }
  }
}
```

---

### List Notes

**Endpoint:** `GET /api/admin/orders/:uuid/notes`

**Authentication:** Required — `orders.read`

Returns all internal notes for the order, ordered newest-first.

**Example Request:**
```http
GET /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/notes
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Notes retrieved",
  "data": [
    {
      "id": "note-uuid-here",
      "note": "Customer called — confirmed delivery address.",
      "isInternal": true,
      "createdAt": "2026-03-15T11:30:00Z",
      "updatedAt": "2026-03-15T11:30:00Z",
      "authorAdmin": {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "fullName": "Alice Smith",
        "email": "alice@example.com",
        "avatarUrl": null
      }
    }
  ]
}
```

---

## Timeline

**Endpoint:** `GET /api/admin/orders/:uuid/timeline`

**Authentication:** Required — `orders.read`

Returns a paginated audit trail of all admin actions performed on this order, ordered newest-first. Reuses the `AuditLog` table — filtered by `entityType = "Order"`.

**Query Parameters:**

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| `page` | Integer | No | `1` | Page number |
| `limit` | Integer | No | `30` | Items per page (max 100) |

**Example Request:**
```http
GET /api/admin/orders/a1b2c3d4-e5f6-7890-abcd-ef1234567890/timeline?page=1&limit=30
Authorization: Bearer <accessToken>
```

**Example Response (200 OK):**
```json
{
  "message": "Timeline retrieved",
  "data": {
    "logs": [
      {
        "id": "log-uuid-1",
        "action": "ORDER_STATUS_CHANGE",
        "meta": {
          "before": { "status": "PENDING" },
          "after": { "status": "PAID" },
          "reason": "Payment confirmed"
        },
        "ip": "192.168.1.1",
        "createdAt": "2026-03-15T10:30:00Z",
        "actorAdmin": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "fullName": "Alice Smith",
          "email": "alice@example.com",
          "avatarUrl": null
        }
      },
      {
        "id": "log-uuid-2",
        "action": "ORDER_NOTE_ADDED",
        "meta": { "noteId": "note-uuid-here" },
        "ip": "192.168.1.1",
        "createdAt": "2026-03-15T11:30:00Z",
        "actorAdmin": {
          "id": "550e8400-e29b-41d4-a716-446655440000",
          "fullName": "Alice Smith",
          "email": "alice@example.com",
          "avatarUrl": null
        }
      }
    ],
    "total": 5,
    "page": 1,
    "limit": 30,
    "totalPages": 1
  }
}
```

**Timeline action types:**

| Action | Trigger |
|--------|---------|
| `ORDER_STATUS_CHANGE` | Order status updated |
| `ORDER_PAYMENT_STATUS_CHANGE` | Payment status updated |
| `ORDER_FULFILLMENT_STATUS_CHANGE` | Fulfillment status updated |
| `ORDER_ASSIGNED` | Order assigned or unassigned |
| `ORDER_PRIORITY_CHANGE` | Priority changed |
| `ORDER_TAGS_UPDATED` | Tags replaced |
| `ORDER_CANCELLED` | Order cancelled |
| `ORDER_REFUND_INITIATED` | Refund recorded |
| `ORDER_NOTE_ADDED` | Note added |

---

## Bulk Operations

### Bulk Update Status

**Endpoint:** `PATCH /api/admin/orders/bulk/status`

**Authentication:** Required — `orders.update`

Updates status for multiple orders at once. Each order is processed independently — failures do not block others. Returns a results object listing which orders succeeded and which failed with reasons.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuids` | UUID[] | Yes | Array of order UUIDs (1–100 items) |
| `status` | String | Yes | Target status for all orders |

**Example Request:**
```http
PATCH /api/admin/orders/bulk/status
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "uuids": [
    "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "b2c3d4e5-f6a7-8901-bcde-f12345678901"
  ],
  "status": "PAID"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Bulk status update complete",
  "data": {
    "updated": ["a1b2c3d4-e5f6-7890-abcd-ef1234567890"],
    "failed": [
      {
        "uuid": "b2c3d4e5-f6a7-8901-bcde-f12345678901",
        "reason": "Cannot transition from CANCELLED to PAID"
      }
    ]
  }
}
```

---

### Bulk Assign

**Endpoint:** `PATCH /api/admin/orders/bulk/assign`

**Authentication:** Required — `orders.update`

Assigns (or unassigns) multiple orders to an admin user in one request.

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `uuids` | UUID[] | Yes | Array of order UUIDs (1–100 items) |
| `assignedToAdminId` | UUID or null | No | Admin to assign. `null` removes assignment from all |

**Example Request:**
```http
PATCH /api/admin/orders/bulk/assign
Authorization: Bearer <accessToken>
Content-Type: application/json

{
  "uuids": [
    "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "b2c3d4e5-f6a7-8901-bcde-f12345678901"
  ],
  "assignedToAdminId": "550e8400-e29b-41d4-a716-446655440000"
}
```

**Example Response (200 OK):**
```json
{
  "message": "Bulk assign complete",
  "data": {
    "updated": [
      "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
      "b2c3d4e5-f6a7-8901-bcde-f12345678901"
    ],
    "failed": []
  }
}
```

---

## Status Transition Rules

### Order Status

```
PENDING  → PAID        ✓
PENDING  → CANCELLED   ✓
PAID     → SHIPPED     ✓
PAID     → CANCELLED   ✓
SHIPPED  → COMPLETED   ✓
SHIPPED  → CANCELLED   ✓
COMPLETED → (any)      ✗  (terminal)
CANCELLED → (any)      ✗  (terminal)
```

### Payment Status

```
UNPAID           → PAID, PARTIALLY_PAID              ✓
PARTIALLY_PAID   → PAID, REFUNDED, PARTIALLY_REFUNDED ✓
PAID             → REFUNDED, PARTIALLY_REFUNDED       ✓
PARTIALLY_REFUNDED → REFUNDED                         ✓
REFUNDED         → (any)                              ✗  (terminal)
```

---

## Business Rules

| Rule | Enforced at |
|------|-------------|
| Only valid status transitions are accepted | `PATCH .../status` and bulk status |
| Only valid payment transitions are accepted | `PATCH .../payment-status` |
| Cannot refund an UNPAID order | `POST .../refund` |
| Cannot refund a fully REFUNDED order | `POST .../refund` |
| Refund amount cannot exceed order total | `POST .../refund` |
| Cannot cancel a COMPLETED order | `POST .../cancel` |
| Cannot cancel an already CANCELLED order | `POST .../cancel` |
| Bulk operations isolate per-item failures | `PATCH /bulk/status` and `/bulk/assign` |
| Order UUIDs are used in URLs (not BigInt IDs) | All endpoints |
| All mutations are recorded in the audit timeline | Service layer |

---

## Error Responses

| Status | Meaning |
|--------|---------|
| `401` | Missing or invalid/expired admin token |
| `403` | RBAC permission denied (`orders.read` or `orders.update` required) |
| `404` | Order not found |
| `400` | Business rule violation (see per-endpoint table) |
| `422` | Validation error — response includes field-level detail |

**Validation error example (422):**
```json
{
  "message": "Validation Error",
  "status_code": 422,
  "errors": {
    "status": "\"status\" must be one of [PENDING, PAID, SHIPPED, COMPLETED, CANCELLED]",
    "uuids": "\"uuids\" must contain at least 1 items"
  }
}
```

---

## Related Docs

- [admin-user-management.md](./admin-user-management.md) — Admin user CRUD and audit logs.
- [admin-permissions.md](./admin-permissions.md) — RBAC, roles, and permission resources.
- [admin.md](./admin.md) — App-user management, moderation, and analytics.
