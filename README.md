# API Documentation

This folder contains all Postman API documentation files organized by module.

## 📚 Documentation Index

### Core Modules

- **[admin.md](./admin.md)** - Admin endpoints for user management, content moderation, and analytics
- **[auth.md](./auth.md)** - Authentication and FCM token integration (login, logout, settings)
- **[chat.md](./chat.md)** - Real-time chat system with Socket.io (conversations, messages, typing indicators)
- **[collections.md](./collections.md)** - Collections API documentation
- **[notifications.md](./notifications.md)** - Notifications system with Firebase Cloud Messaging
- **[products.md](./products.md)** - Products API documentation
- **[products-quick-reference.md](./products-quick-reference.md)** - Quick reference for Products API
- **[profiles.md](./profiles.md)** - User profile management APIs

### Social Module

- **[social.md](./social.md)** - Main Social API documentation
- **[social-activity.md](./social-activity.md)** - Activity feed endpoints
- **[social-bookmarks.md](./social-bookmarks.md)** - Bookmarks/saved posts API
- **[social-hashtags.md](./social-hashtags.md)** - Hashtags system (search, trending, discovery)
- **[social-reports.md](./social-reports.md)** - Content reporting system
- **[social-stories.md](./social-stories.md)** - Stories feature API
- **[social-posts.md](./social-posts.md)** - Posts search and explore features
- **[social-engagement.md](./social-engagement.md)** - Engagement features overview (consolidated)
- **[social-new-features.md](./social-new-features.md)** - New social features documentation
- **[social-quick-reference.md](./social-quick-reference.md)** - Quick reference guide
- **[social-testing-guide.md](./social-testing-guide.md)** - Testing guide for social features

## 🚀 Quick Start

1. **Authentication**: Start with [auth.md](./auth.md) to set up authentication
2. **Profiles**: Check [profiles.md](./profiles.md) for user profile management
3. **Chat**: See [chat.md](./chat.md) for real-time messaging between users
4. **Social Features**: Explore [social-engagement.md](./social-engagement.md) for all social features
5. **Admin Tools**: See [admin.md](./admin.md) for moderation and analytics

## 📝 Notes

- All endpoints require authentication unless otherwise specified
- Base URL: `http://localhost:YOUR_PORT/api` (or your server URL)
- Use Bearer token authentication: `Authorization: Bearer <your-token>`
- Most endpoints support pagination with `page` and `size` query parameters

## ➕ Adding New Module Documentation

When creating a new module, automatically generate documentation:

```bash
node scripts/generate-docs.js <module-name>
```

**Example:**
```bash
node scripts/generate-docs.js payments
```

This will:
1. Create `Documentations/payments.md` from the template
2. Replace placeholders with your module name
3. Give you a ready-to-customize documentation file

**After generation:**
1. Open the generated file and fill in module details
2. Document all endpoints following the template structure
3. Update this README.md to include your new module in the index

**Template:** See `Documentations/TEMPLATE.md` for the documentation template.

---

## 🔗 Related Documentation

- For database schema, see `prisma/schema.prisma`
- For route definitions, see `src/modules/*/router.js` files
- For validation schemas, see `src/modules/*/controller/*.validation.js` files
- For script documentation, see `scripts/README.md`
