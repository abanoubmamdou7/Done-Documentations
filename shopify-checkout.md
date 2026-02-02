# Shopify Checkout Flow API Documentation

## Overview

Complete API documentation for the Shopify checkout flow in DoneBuddy. This module handles the entire e-commerce flow from adding items to cart, through checkout creation, payment processing, and order fulfillment via Shopify webhooks.

**Base URL:** `/api/cart`, `/api/checkout`, `/api/webhooks/shopify`

**Architecture:**
- **Cart Management**: Local database (not Shopify cart)
- **Checkout**: Shopify Draft Orders
- **Payment**: Shopify Checkout (Stripe integration)
- **Order Updates**: Shopify Webhooks

---

## Table of Contents

1. [Authentication](#authentication)
2. [Cart Management](#cart-management)
   - [Get Cart](#get-cart)
   - [Add Item to Cart](#add-item-to-cart)
   - [Update Cart Item](#update-cart-item)
   - [Remove Cart Item](#remove-cart-item)
   - [Clear Cart](#clear-cart)
3. [Checkout](#checkout)
   - [Create Checkout (Draft Order)](#create-checkout-draft-order)
4. [Payment Flow](#payment-flow)
5. [Webhooks](#webhooks)
   - [Order Create](#order-create-webhook)
   - [Order Paid](#order-paid-webhook)
   - [Order Cancelled](#order-cancelled-webhook)
   - [Fulfillment Create](#fulfillment-create-webhook)
   - [Fulfillment Update](#fulfillment-update-webhook)
6. [Error Responses](#error-responses)
7. [Postman Collection](#postman-collection)

---

## Authentication

All cart and checkout endpoints require authentication.

**Header:**
```
Authorization: Bearer <your-jwt-token>
```

**User Roles:** `USER`, `SELLER`, `MEDIATOR`

---

## Cart Management

### Get Cart

**Description:** Retrieve the current active cart with all items, totals, and product details.

**Endpoint:** `GET /api/cart`

**Authentication:** Required

**Example Request:**
```http
GET /api/cart
Authorization: Bearer <your-jwt-token>
Content-Type: application/json
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "data": {
    "id": "3",
    "uuid": "f726e351-9b62-440c-9291-183f68868d41",
    "status": "ACTIVE",
    "itemCount": 2,
    "items": [
      {
        "id": "8",
        "productVariantId": "250",
        "quantity": 2,
        "price": "226.49",
        "currency": "USD",
        "product": {
          "id": "59",
          "title": "Steelseries 62406 Aerox 5 Wireless Gaming Mouse",
          "image": "https://cdn.shopify.com/s/files/1/0742/5376/2815/files/235593108784-0.jpg",
          "variant": {
            "id": "250",
            "title": "Right",
            "price": "226.49"
          }
        }
      }
    ],
    "totals": {
      "subtotal": 452.98,
      "tax": 0,
      "total": 452.98,
      "currency": "USD",
      "itemCount": 2
    },
    "createdAt": "2026-01-24T14:18:06.621Z",
    "updatedAt": "2026-01-24T14:18:06.621Z"
  }
}
```

**Notes:**
- Returns empty cart if no active cart exists
- Price is stored as snapshot at add-to-cart time
- Tax is calculated by Shopify Checkout (not in app)

---

### Add Item to Cart

**Description:** Add a product variant to the cart. Supports multiple ways to identify variants.

**Endpoint:** `POST /api/cart/items`

**Authentication:** Required

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `productUuid` | string (UUID) | Conditional* | Product UUID |
| `variantOption` | string | Optional | Variant option value (e.g., "black", "WHITE") - used with productUuid |
| `shopifyVariantId` | string | Conditional* | Shopify variant ID (most specific) |
| `productVariantId` | string/number | Conditional* | Database variant ID |
| `quantity` | number | Required | Quantity to add (min: 1) |

\* At least one of `productUuid`, `shopifyVariantId`, or `productVariantId` is required.

**Example Request (Using Product UUID):**
```http
POST /api/cart/items
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "productUuid": "c79f9adf-a07d-49bd-b968-514e956f1830",
  "variantOption": "black",
  "quantity": 2
}
```

**Example Request (Using Shopify Variant ID):**
```http
POST /api/cart/items
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "shopifyVariantId": "47583440404735",
  "quantity": 2
}
```

**Example Request (Using Database Variant ID):**
```http
POST /api/cart/items
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "productVariantId": "250",
  "quantity": 2
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Item added to cart",
  "data": {
    "item": {
      "id": "8",
      "productVariantId": "250",
      "quantity": 2,
      "price": "226.49",
      "currency": "USD",
      "product": {
        "id": "59",
        "title": "Steelseries 62406 Aerox 5 Wireless Gaming Mouse",
        "image": "https://cdn.shopify.com/s/files/1/0742/5376/2815/files/235593108784-0.jpg",
        "variant": {
          "id": "250",
          "title": "Right",
          "price": "226.49"
        }
      }
    },
    "cart": {
      "id": "3",
      "itemCount": 2,
      "totals": {
        "subtotal": 452.98,
        "tax": 0,
        "total": 452.98,
        "currency": "USD",
        "itemCount": 2
      }
    }
  }
}
```

**Behavior:**
- If the same variant already exists in cart, quantity is updated (not duplicated)
- Price is stored as snapshot at add time (preserves price even if variant price changes)
- Cart is scoped to a single Shopify store (all items must be from same seller)

**Error Responses:**

**400 Bad Request - Missing Required Field:**
```json
{
  "message": "Either productVariantId, shopifyVariantId, or productUuid is required",
  "status_code": 400
}
```

**400 Bad Request - Invalid Variant Option:**
```json
{
  "message": "Variant option 'red' not found. Available options: black, WHITE",
  "status_code": 400
}
```

**404 Not Found - Product Not Found:**
```json
{
  "message": "Product not found",
  "status_code": 404
}
```

---

### Update Cart Item

**Description:** Update the quantity of a specific cart item.

**Endpoint:** `PATCH /api/cart/items/:itemId`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `itemId` | string | Required | Cart item ID |

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `quantity` | number | Required | New quantity (min: 1) |

**Example Request:**
```http
PATCH /api/cart/items/8
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "quantity": 3
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Cart item updated",
  "data": {
    "item": {
      "id": "8",
      "productVariantId": "250",
      "quantity": 3,
      "price": "226.49",
      "currency": "USD"
    },
    "cart": {
      "id": "3",
      "itemCount": 3,
      "totals": {
        "subtotal": 679.47,
        "tax": 0,
        "total": 679.47,
        "currency": "USD",
        "itemCount": 3
      }
    }
  }
}
```

---

### Remove Cart Item

**Description:** Remove a specific item from the cart.

**Endpoint:** `DELETE /api/cart/items/:itemId`

**Authentication:** Required

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `itemId` | string | Required | Cart item ID |

**Example Request:**
```http
DELETE /api/cart/items/8
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Item removed from cart",
  "data": {
    "cart": {
      "id": "3",
      "itemCount": 0,
      "totals": {
        "subtotal": 0,
        "tax": 0,
        "total": 0,
        "currency": "USD",
        "itemCount": 0
      }
    }
  }
}
```

---

### Clear Cart

**Description:** Remove all items from the cart.

**Endpoint:** `DELETE /api/cart`

**Authentication:** Required

**Example Request:**
```http
DELETE /api/cart
Authorization: Bearer <your-jwt-token>
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Cart cleared successfully",
  "data": {
    "cart": {
      "id": "3",
      "itemCount": 0,
      "totals": {
        "subtotal": 0,
        "tax": 0,
        "total": 0,
        "currency": "USD",
        "itemCount": 0
      }
    }
  }
}
```

---

## Checkout

### Create Checkout (Draft Order)

**Description:** Create a Shopify Draft Order from the active cart and return the checkout URL. The customer will complete payment on Shopify's checkout page.

**Endpoint:** `POST /api/checkout`

**Authentication:** Required

**Request Body:**

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cartItemIds` | array | Optional | Cart item IDs to checkout. If omitted or empty, **all cart items** are checked out (default). |

**Example Request (checkout all - default):**
```http
POST /api/checkout
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{}
```

**Example Request (checkout specific items):**
```http
POST /api/checkout
Authorization: Bearer <your-jwt-token>
Content-Type: application/json

{
  "cartItemIds": ["1", "3", "5"]
}
```

**Example Response (200 OK):**
```json
{
  "success": true,
  "message": "Checkout created successfully",
  "data": {
    "checkoutUrl": "https://www.doneshops.com/74253762815/invoices/b566204c20b58ef4476df12b0b39fa76",
    "draftOrderId": "1256751169791",
    "subtotal": "452.98",
    "currency": "USD"
  }
}
```

**Flow:**
1. Loads active cart
2. If `cartItemIds` provided: filters to only those items; otherwise uses all items
3. Verifies all selected items belong to the same Shopify store
4. Converts cart items to Shopify Draft Order `line_items`
5. Creates Draft Order via Shopify Admin REST API
6. **Full checkout** (no cartItemIds): marks cart as `CHECKED_OUT`
7. **Partial checkout** (cartItemIds): removes checked-out items from cart; marks `CHECKED_OUT` if cart empty
8. Returns `invoice_url` for customer to complete checkout

**Important Notes:**
- **Shipping**: Calculated by Shopify Checkout (manual rates only - Basic Shopify)
- **Tax**: Calculated by Shopify Checkout
- **Payment**: Handled by Shopify Checkout (Stripe integration)
- **Price**: Uses cart snapshot prices (preserves price at add-to-cart time)

**Error Responses:**

**400 Bad Request - Empty Cart:**
```json
{
  "message": "Cart is empty",
  "status_code": 400
}
```

**400 Bad Request - Multiple Stores:**
```json
{
  "message": "All cart items must be from the same Shopify store",
  "status_code": 400
}
```

**400 Bad Request - Invalid cartItemIds:**
```json
{
  "message": "No valid cart items found for the provided cartItemIds",
  "status_code": 400
}
```

```json
{
  "message": "Some cartItemIds are not in your cart: 99, 100",
  "status_code": 400
}
```

**404 Not Found - No Active Cart:**
```json
{
  "message": "No active cart found",
  "status_code": 404
}
```

**500 Internal Server Error - Shopify API Error:**
```json
{
  "message": "Failed to create checkout: Shopify API error",
  "status_code": 500
}
```

---

## Payment Flow

### Overview

Payment is handled entirely by Shopify Checkout. The app does NOT:
- ❌ Process payments directly
- ❌ Handle credit card data
- ❌ Integrate with Stripe directly
- ❌ Calculate shipping or tax

### Flow Diagram

```
1. Customer adds items to cart (local database)
   ↓
2. Customer clicks "Checkout"
   ↓
3. App creates Shopify Draft Order
   ↓
4. App returns checkoutUrl (Shopify invoice URL)
   ↓
5. Customer redirected to Shopify Checkout
   ↓
6. Shopify handles:
   - Address collection
   - Shipping selection
   - Tax calculation
   - Payment processing (Stripe)
   ↓
7. After payment, Shopify sends webhooks:
   - orders/create
   - orders/paid (triggers inventory update)
   ↓
8. App updates local database:
   - Order status
   - Inventory quantities
   - Customer information
```

### Shipping Policy

**Supported Shipping Methods (Basic Shopify):**
- ✅ Flat-rate shipping
- ✅ Weight-based shipping
- ✅ Price-based shipping
- ❌ Carrier Calculated Shipping (DHL, FedEx, UPS, Aramex) - NOT supported

**Note:** Shipping rates are configured in Shopify Admin → Settings → Shipping. The app does not calculate shipping.

---

## Webhooks

All webhooks require HMAC signature verification for security.

**Required Headers:**
```
X-Shopify-Hmac-Sha256: <hmac-signature>
X-Shopify-Shop-Domain: <shop-domain>
X-Shopify-Topic: <webhook-topic>
Content-Type: application/json
```

**Base URL:** `/api/webhooks/shopify`

---

### Order Create Webhook

**Description:** Triggered when a Draft Order is converted to a real Order (after customer completes checkout).

**Endpoint:** `POST /api/webhooks/shopify/orders/create`

**Webhook Topic:** `orders/create`

**Example Payload:**
```json
{
  "id": 1234567890,
  "order_number": 1001,
  "name": "#1001",
  "email": "customer@example.com",
  "created_at": "2024-01-01T12:00:00Z",
  "updated_at": "2024-01-01T12:00:00Z",
  "financial_status": "paid",
  "fulfillment_status": null,
  "currency": "USD",
  "total_price": "452.98",
  "subtotal_price": "452.98",
  "total_tax": "0.00",
  "customer": {
    "id": 9876543210,
    "email": "customer@example.com",
    "first_name": "John",
    "last_name": "Doe",
    "phone": "+1234567890",
    "created_at": "2024-01-01T10:00:00Z",
    "updated_at": "2024-01-01T12:00:00Z"
  },
  "line_items": [
    {
      "id": 1111111111,
      "variant_id": "47583440404735",
      "product_id": "3333333333",
      "title": "Product Name",
      "variant_title": "Variant Name",
      "quantity": 2,
      "price": "226.49",
      "sku": "SKU-123"
    }
  ],
  "shipping_address": {
    "first_name": "John",
    "last_name": "Doe",
    "address1": "123 Main St",
    "city": "New York",
    "province": "NY",
    "country": "United States",
    "zip": "10001"
  },
  "billing_address": {
    "first_name": "John",
    "last_name": "Doe",
    "address1": "123 Main St",
    "city": "New York",
    "province": "NY",
    "country": "United States",
    "zip": "10001"
  }
}
```

**What Happens:**
1. Order is saved to local database
2. Customer is created/updated
3. Order items are linked to product variants
4. Order status is set based on `financial_status` and `fulfillment_status`

**Response:**
```json
{
  "success": true
}
```

---

### Order Paid Webhook

**Description:** Triggered when payment is completed for an order. **This is when inventory is updated.**

**Endpoint:** `POST /api/webhooks/shopify/orders/paid`

**Webhook Topic:** `orders/paid`

**Example Payload:**
```json
{
  "id": 1234567890,
  "order_number": 1001,
  "name": "#1001",
  "financial_status": "paid",
  "currency": "USD",
  "total_price": "452.98",
  "line_items": [
    {
      "variant_id": "47583440404735",
      "quantity": 2,
      "price": "226.49"
    }
  ],
  "customer": {
    "id": 9876543210,
    "email": "customer@example.com",
    "first_name": "John",
    "last_name": "Doe"
  }
}
```

**What Happens:**
1. Order status is updated to `PAID`
2. **Inventory is updated** (both local database and Shopify):
   - Local: `ProductVariant.inventoryQuantity` decreased
   - Shopify: Inventory levels updated via API
   - Local: `InventoryLevel` records updated (if exists)
3. Inventory update is non-blocking (errors don't fail webhook)

**Response:**
```json
{
  "success": true
}
```

**Important:** Inventory is only updated after successful payment. This ensures inventory is only reduced when customer actually pays.

---

### Order Cancelled Webhook

**Description:** Triggered when an order is cancelled.

**Endpoint:** `POST /api/webhooks/shopify/orders/cancelled`

**Webhook Topic:** `orders/cancelled`

**Example Payload:**
```json
{
  "id": 1234567890,
  "order_number": 1001,
  "name": "#1001",
  "financial_status": "voided",
  "cancelled_at": "2024-01-01T13:00:00Z",
  "cancel_reason": "customer"
}
```

**What Happens:**
1. Order status is updated to `CANCELLED`
2. Order cancellation reason is saved

**Note:** Inventory is NOT automatically restored. Manual restoration required if needed.

**Response:**
```json
{
  "success": true
}
```

---

### Fulfillment Create Webhook

**Description:** Triggered when an order is fulfilled (shipped).

**Endpoint:** `POST /api/webhooks/shopify/fulfillments/create`

**Webhook Topic:** `fulfillments/create`

**Example Payload:**
```json
{
  "id": 4444444444,
  "order_id": 1234567890,
  "status": "success",
  "created_at": "2024-01-02T10:00:00Z",
  "updated_at": "2024-01-02T10:00:00Z",
  "tracking_company": "USPS",
  "tracking_number": "9400111899223197428490",
  "tracking_url": "https://tools.usps.com/go/TrackConfirmAction?tLabels=9400111899223197428490",
  "line_items": [
    {
      "id": 1111111111,
      "quantity": 2
    }
  ]
}
```

**What Happens:**
1. Fulfillment is saved to local database
2. Order status is updated to `SHIPPED` (if all items fulfilled)
3. Tracking information is stored

**Response:**
```json
{
  "success": true
}
```

---

### Fulfillment Update Webhook

**Description:** Triggered when fulfillment status changes (e.g., tracking updated).

**Endpoint:** `POST /api/webhooks/shopify/fulfillments/update`

**Webhook Topic:** `fulfillments/update`

**Example Payload:**
```json
{
  "id": 4444444444,
  "order_id": 1234567890,
  "status": "success",
  "updated_at": "2024-01-02T11:00:00Z",
  "tracking_company": "USPS",
  "tracking_number": "9400111899223197428490"
}
```

**What Happens:**
1. Fulfillment record is updated
2. Tracking information is updated

**Response:**
```json
{
  "success": true
}
```

---

## Error Responses

### Common Error Codes

**400 Bad Request:**
```json
{
  "message": "Error message",
  "status_code": 400
}
```

**401 Unauthorized:**
```json
{
  "message": "Unauthorized",
  "status_code": 401
}
```

**404 Not Found:**
```json
{
  "message": "Resource not found",
  "status_code": 404
}
```

**500 Internal Server Error:**
```json
{
  "message": "Internal server error",
  "status_code": 500
}
```

### Webhook-Specific Errors

**401 Unauthorized - Invalid Signature:**
```json
{
  "error": "Invalid webhook signature",
  "status_code": 401
}
```

**401 Unauthorized - Missing Headers:**
```json
{
  "error": "Missing webhook headers",
  "status_code": 401
}
```

**404 Not Found - Seller Not Found:**
```json
{
  "error": "Seller not found",
  "status_code": 404
}
```

---

## Postman Collection

### Environment Variables

The Postman collection includes the following variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `baseUrl` | API base URL | `http://localhost:3000` |
| `authToken` | JWT authentication token | `Bearer eyJhbGc...` |
| `shopDomain` | Shopify shop domain | `your-shop.myshopify.com` |
| `webhookSignature` | HMAC signature for webhooks | Generated via script |
| `productUuid` | Product UUID | `c79f9adf-a07d-49bd-b968-514e956f1830` |
| `variantOption` | Variant option value | `black` or `WHITE` |
| `shopifyVariantId` | Shopify variant ID | `47583440404735` |
| `productVariantId` | Database variant ID | `250` |
| `cartItemId` | Cart item ID | `8` |

### Generating Webhook Signatures

Use the provided script to generate valid HMAC signatures:

```bash
node scripts/generate-webhook-signature.js
```

This will output:
- Webhook payload (JSON)
- HMAC signature (base64)
- Postman headers to use

### Collection Structure

```
DoneBuddy - Shopify Checkout Flow
├── Cart Management
│   ├── Get Cart
│   ├── Add Item to Cart
│   ├── Update Cart Item
│   ├── Remove Cart Item
│   └── Clear Cart
├── Checkout
│   └── Create Checkout (Draft Order)
└── Webhooks
    ├── Order Create
    ├── Order Paid
    ├── Order Cancelled
    ├── Fulfillment Create
    └── Fulfillment Update
```

### Import Collection

1. Open Postman
2. Click "Import"
3. Select `docs/features/Shopify_Checkout_Flow.postman_collection.json`
4. Set environment variables
5. Start testing!

---

## Testing Guide

### Complete Flow Test

1. **Add Items to Cart:**
   ```http
   POST /api/cart/items
   {
     "productUuid": "c79f9adf-a07d-49bd-b968-514e956f1830",
     "variantOption": "black",
     "quantity": 2
   }
   ```

2. **Get Cart:**
   ```http
   GET /api/cart
   ```
   Verify items and totals are correct.

3. **Create Checkout:**
   ```http
   POST /api/checkout
   ```
   Copy the `checkoutUrl` from response.

4. **Complete Payment:**
   - Open `checkoutUrl` in browser
   - Complete Shopify checkout
   - Use test payment method

5. **Verify Webhooks:**
   - Check server logs for webhook processing
   - Verify order created in database
   - Verify inventory updated (after `orders/paid` webhook)

### Testing Webhooks Locally

1. **Start ngrok:**
   ```bash
   ngrok http 3000
   ```

2. **Register Webhook in Shopify:**
   - Go to Shopify Admin → Settings → Notifications → Webhooks
   - Create webhook: `orders/paid` → `https://your-ngrok-url.ngrok.io/api/webhooks/shopify/orders/paid`

3. **Complete Test Order:**
   - Create checkout
   - Complete payment
   - Webhook will be triggered automatically

4. **Monitor Logs:**
   ```bash
   # Look for:
   # "Inventory updated after checkout: { orderId: '...', updated: 2, failed: 0 }"
   ```

See [WEBHOOK_TESTING_GUIDE.md](../docs/features/WEBHOOK_TESTING_GUIDE.md) for detailed webhook testing instructions.

---

## Important Notes

### Cart Behavior

- ✅ **Local Cart**: Cart is stored in local database (not Shopify cart)
- ✅ **Price Snapshots**: Prices are stored at add-to-cart time (preserves price even if variant price changes)
- ✅ **Single Store**: All cart items must be from the same Shopify store
- ✅ **Auto-Consolidation**: Adding the same variant multiple times updates quantity (doesn't create duplicates)

### Checkout Behavior

- ✅ **Draft Orders**: Uses Shopify Draft Orders (not direct Order creation)
- ✅ **Price Preservation**: Uses cart snapshot prices (not current variant prices)
- ✅ **Shopify Handles**: Shipping, tax, and payment are handled by Shopify Checkout
- ✅ **No Stripe Integration**: App does NOT integrate Stripe directly

### Inventory Updates

- ✅ **After Payment Only**: Inventory is updated only after successful payment (`orders/paid` webhook)
- ✅ **Both Databases**: Updates both local database and Shopify inventory
- ✅ **Non-Blocking**: Inventory update errors don't fail the webhook
- ✅ **Shopify-Managed Only**: Only updates variants with `inventoryManagement === "shopify"`

### Webhook Security

- ✅ **HMAC Verification**: All webhooks require valid HMAC signature
- ✅ **Shop Domain Verification**: Verifies webhook is from correct shop
- ✅ **Signature Validation**: Uses timing-safe comparison to prevent timing attacks

---

## Related Documentation

- [WEBHOOK_TESTING_GUIDE.md](../docs/features/WEBHOOK_TESTING_GUIDE.md) - Detailed webhook testing instructions
- [INVENTORY_UPDATE_AFTER_CHECKOUT.md](../docs/features/INVENTORY_UPDATE_AFTER_CHECKOUT.md) - Inventory update documentation
- [SHOPIFY_CHECKOUT_FLOW.md](../docs/features/SHOPIFY_CHECKOUT_FLOW.md) - Complete flow documentation

---

## Support

For issues or questions:
1. Check server logs for error messages
2. Verify webhook signatures are correct
3. Ensure Shopify store is connected
4. Verify all required environment variables are set

**Last Updated:** January 19, 2026  
**Version:** 1.0
