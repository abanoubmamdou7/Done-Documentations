# Admin Permissions (RBAC)

This document describes the Done Buddy admin permission model: resources, actions, roles, and how access is enforced. Use it together with [admin.md](./admin.md) for moderation/analytics endpoints.

**Base URL (RBAC):** `/api/admin` (auth, roles, permissions)

---

## Overview

- **Admin users** log in via `/api/admin/auth/login` and receive a JWT. They are stored in `admin_users` and are **separate** from app `Profile` (which has `Role.ADMIN` for app-level admin).
- Each admin user has one **Admin Role** (`admin_roles`). Roles have a **permission matrix** per resource (read, create, update, delete, restore).
- **Permission resources** (`permission_resources`) define what can be controlled (e.g. `orders`, `users`, `roles`). Each resource can be toggled on/off globally (`is_active`).
- Access is **deny by default**: no role/resource or missing permission → 403.

---

## Table of Contents

1. [Actions](#actions)
2. [Permission Resources](#permission-resources)
3. [Super Admin](#super-admin)
4. [API Endpoints and Required Permissions](#api-endpoints-and-required-permissions)
5. [Postman Collection](#postman-collection)
6. [Behavior Summary](#behavior-summary)
7. [Related Docs](#related-docs)

---

## Actions

For each resource, a role can have:

| Action   | Flag        | Meaning                          |
|----------|-------------|----------------------------------|
| **read** | `canRead`   | List, view, get details          |
| **create** | `canCreate` | Create new records               |
| **update** | `canUpdate` | Edit existing records            |
| **delete** | `canDelete` | Soft-delete (where applicable)   |
| **restore** | `canRestore` | Restore soft-deleted records   |

APIs use `requireRbac(resourceKey, action)`; the middleware checks the corresponding flag (or Super Admin bypass).

---

## Permission Resources

Resources are seeded in `scripts/seed-admin-rbac.js`. Below is the full list by group.

### Overview

| Key         | Label     | Description                          |
|-------------|-----------|--------------------------------------|
| `dashboard` | Dashboard | Main overview, KPIs and quick stats  |

### Management

| Key           | Label        | Description                                |
|---------------|--------------|--------------------------------------------|
| `users`       | Users        | App users and profiles (list, block, view) |
| `orders`      | Orders       | Orders from app and Shopify                 |
| `products`    | Products     | Product catalog and variants               |
| `collections` | Collections  | Product collections and categories          |
| `sellers`     | Sellers      | Seller accounts and Shopify connections     |
| `customers`   | Customers    | Shopify customers synced per seller         |
| `inventory`   | Inventory    | Stock levels and inventory sync             |

### Social & Content

| Key        | Label     | Description                      |
|------------|-----------|----------------------------------|
| `posts`    | Posts     | Social feed posts and moderation |
| `comments` | Comments  | Post comments and replies        |
| `stories`  | Stories   | Stories and ephemeral content    |
| `reports`  | Reports   | User reports and moderation queue|
| `hashtags` | Hashtags  | Hashtag management and trending  |

### Finance

| Key        | Label    | Description                    |
|------------|----------|--------------------------------|
| `payments` | Payments | Payments and transactions     |
| `refunds`  | Refunds  | Refunds and disputes          |
| `payouts`  | Payouts  | Seller payouts and balances   |

### Chat

| Key             | Label         | Description                    |
|-----------------|---------------|--------------------------------|
| `conversations` | Conversations | Chat conversations and DMs    |
| `messages`      | Messages      | Chat messages (moderation)     |

### Monitoring

| Key         | Label      | Description                      |
|-------------|------------|----------------------------------|
| `suppliers` | Suppliers  | Suppliers and dropship sources   |
| `sync_jobs` | Sync Jobs  | Shopify sync jobs and webhooks   |
| `webhooks`  | Webhooks   | Webhook logs and retries         |

### Reports

| Key         | Label     | Description                |
|-------------|-----------|----------------------------|
| `analytics` | Analytics | Analytics and dashboards   |
| `exports`   | Exports   | Data exports and reports   |

### Settings

| Key             | Label         | Description                         |
|-----------------|---------------|-------------------------------------|
| `countries`     | Countries     | Countries and regions config        |
| `banners`       | Banners       | Home and promo banners              |
| `notifications` | Notifications | Push and in-app notification config |

### Security

| Key           | Label        | Description                          |
|---------------|--------------|--------------------------------------|
| `audit_logs`  | Audit Logs   | Admin action audit trail             |
| `roles`       | Roles        | Admin roles and assignment           |
| `permissions` | Permissions  | Permission resources and feature toggles |
| `admin_users` | Admin Users  | Admin panel user accounts            |

---

## Super Admin

- A role with **`isSuper: true`** (or name `"Super Admin"`) **bypasses** the permission matrix: all actions are allowed for all resources.
- Super Admin **still respects** `PermissionResource.is_active`: if a resource is disabled, nobody (including Super Admin) can use it.
- The seed creates one "Super Admin" role and assigns it to the bootstrap admin user when `ADMIN_BOOTSTRAP_EMAIL` / `ADMIN_BOOTSTRAP_PASSWORD` are set.

---

## API Endpoints and Required Permissions

| Method | Path | Required resource | Required action |
|--------|------|-------------------|-----------------|
| POST   | `/api/admin/auth/login` | — | Public |
| GET    | `/api/admin/roles` | `roles` | read |
| POST   | `/api/admin/roles` | `roles` | create |
| GET    | `/api/admin/roles/:id` | `roles` | read |
| PATCH  | `/api/admin/roles/:id` | `roles` | update |
| DELETE | `/api/admin/roles/:id` | `roles` | delete |
| GET    | `/api/admin/roles/:id/permissions` | `roles` | read |
| PUT    | `/api/admin/roles/:id/permissions` | `roles` | update |
| GET    | `/api/admin/permissions/resources` | `permissions` | read |
| PATCH  | `/api/admin/permissions/resources/:key` | `permissions` | update |

Other admin endpoints (e.g. orders, users) should use `requireRbac("<resourceKey>", "<action>")` with the appropriate resource and action.

---

## Postman Collection

**Done Buddy Admin RBAC** – Admin auth, roles, and permissions APIs. Set collection variables and use **Login** to get `accessToken`, then use it for all other requests.

### Collection variables

| Variable      | Example                | Description |
|---------------|------------------------|-------------|
| `baseUrl`     | `http://localhost:3000`| API base URL |
| `accessToken` | *(empty until Login)*  | JWT from **Auth → Login** response; used in `Authorization: Bearer {{accessToken}}` |
| `roleId`      | *(empty until needed)* | UUID of a role; set from **List roles** response when calling role-by-id or permissions endpoints |

### Auth

| Request | Method | Path | Body |
|---------|--------|------|------|
| **Login** | POST | `{{baseUrl}}/api/admin/auth/login` | `{"email":"...","password":"..."}` |

- **Headers:** `Content-Type: application/json`
- **Response:** `accessToken`, `admin` (id, email, roleId, role). Copy `accessToken` into the collection variable for subsequent requests.

### Roles

| Request | Method | Path | Body |
|---------|--------|------|------|
| **List roles** | GET | `{{baseUrl}}/api/admin/roles` | — |
| **Create role** | POST | `{{baseUrl}}/api/admin/roles` | `{"name":"Manager","description":"Can manage orders and users"}` |
| **Get role** | GET | `{{baseUrl}}/api/admin/roles/{{roleId}}` | — |
| **Update role** | PATCH | `{{baseUrl}}/api/admin/roles/{{roleId}}` | `{"name":"Manager","description":"Updated","is_active":true}` |
| **Delete role** | DELETE | `{{baseUrl}}/api/admin/roles/{{roleId}}` | — |
| **Get role permissions** | GET | `{{baseUrl}}/api/admin/roles/{{roleId}}/permissions` | — |
| **Set role permissions** | PUT | `{{baseUrl}}/api/admin/roles/{{roleId}}/permissions` | See below |

- **Headers (all except Login):** `Authorization: Bearer {{accessToken}}`, and `Content-Type: application/json` for requests with body.
- **Set role permissions** body: array of `permissions`, each with `resourceKey` and boolean flags `read`, `create`, `update`, `delete`, `restore`. Use resource keys from [Permission Resources](#permission-resources).

**Example – Set role permissions:**
```json
{
  "permissions": [
    { "resourceKey": "orders", "read": true, "create": false, "update": true, "delete": false, "restore": false },
    { "resourceKey": "users", "read": true, "create": false, "update": false, "delete": false, "restore": false }
  ]
}
```

### Permissions

| Request | Method | Path | Body |
|---------|--------|------|------|
| **List resources** | GET | `{{baseUrl}}/api/admin/permissions/resources` | — |
| **Toggle resource** | PATCH | `{{baseUrl}}/api/admin/permissions/resources/:key` | `{"is_active":true}` or `{"is_active":false}` |

- **Toggle resource:** replace `:key` with a resource key (e.g. `orders`). Example path: `.../permissions/resources/orders`.

---

## Behavior Summary

- **Deny by default**: Missing permission, missing role, or disabled resource/role → **403 Forbidden**.
- **Feature toggle**: Set `PermissionResource.is_active = false` to disable a resource for **everyone** (including Super Admin).
- **Role active**: If `AdminRole.is_active` is false, the role cannot perform any action.
- **Audit**: Admin actions (e.g. role create/update, permissions update) are logged in `audit_logs` with actor, action, entity type/id, and optional metadata.

---

## Error Responses

| Status | Meaning |
|--------|---------|
| **401** | Missing or invalid token; or admin not found/inactive. |
| **403** | Forbidden – RBAC: no permission for resource/action, or role/resource disabled. |
| **404** | Role or resource not found. |
| **409** | Conflict (e.g. duplicate role name). |

---

## Related Docs

- [admin.md](./admin.md) – Admin API (reports, users, block, delete post/comment, analytics).
- [../docs/ADMIN_RBAC_README.md](../docs/ADMIN_RBAC_README.md) – Setup, migrations, seed, and login flow for the admin RBAC backend.
