# API Reference

REST endpoint reference for all xshopai backend services. All services use JSON request/response bodies and follow RESTful conventions.

> **Authentication**: Most endpoints require a JWT Bearer token in the `Authorization` header. Service-specific auth requirements are noted below.

---

## web-bff (Gateway) — Port 8014

The web-bff is the single entry point for all frontend applications. It proxies and composes data from downstream services.

### Auth Routes

| Method | Endpoint             | Auth   | Description          |
| :----- | :------------------- | :----- | :------------------- |
| POST   | `/api/auth/register` | —      | Register a new user  |
| POST   | `/api/auth/login`    | —      | Login, returns JWT   |
| POST   | `/api/auth/refresh`  | Cookie | Refresh access token |
| POST   | `/api/auth/logout`   | JWT    | Logout, clear tokens |

### User Routes

| Method | Endpoint                  | Auth | Description                 |
| :----- | :------------------------ | :--- | :-------------------------- |
| GET    | `/api/users/me`           | JWT  | Get current user profile    |
| PUT    | `/api/users/me`           | JWT  | Update current user profile |
| GET    | `/api/users/me/addresses` | JWT  | List user addresses         |
| POST   | `/api/users/me/addresses` | JWT  | Add an address              |

### Product Routes

| Method | Endpoint               | Auth | Description                           |
| :----- | :--------------------- | :--- | :------------------------------------ |
| GET    | `/api/products`        | —    | List products (paginated, filterable) |
| GET    | `/api/products/:id`    | —    | Get product by ID                     |
| GET    | `/api/products/search` | —    | Search products                       |
| GET    | `/api/categories`      | —    | List categories                       |

### Cart Routes

| Method | Endpoint              | Auth | Description                      |
| :----- | :-------------------- | :--- | :------------------------------- |
| GET    | `/api/cart`           | JWT  | Get current user's cart          |
| POST   | `/api/cart/items`     | JWT  | Add item to cart                 |
| PUT    | `/api/cart/items/:id` | JWT  | Update cart item quantity        |
| DELETE | `/api/cart/items/:id` | JWT  | Remove item from cart            |
| DELETE | `/api/cart`           | JWT  | Clear cart                       |
| POST   | `/api/cart/transfer`  | JWT  | Transfer guest cart to user cart |

### Order Routes

| Method | Endpoint                 | Auth | Description        |
| :----- | :----------------------- | :--- | :----------------- |
| POST   | `/api/orders`            | JWT  | Create a new order |
| GET    | `/api/orders`            | JWT  | List user's orders |
| GET    | `/api/orders/:id`        | JWT  | Get order details  |
| POST   | `/api/orders/:id/cancel` | JWT  | Cancel an order    |
| POST   | `/api/orders/:id/return` | JWT  | Request a return   |

### Review Routes

| Method | Endpoint                    | Auth | Description         |
| :----- | :-------------------------- | :--- | :------------------ |
| GET    | `/api/products/:id/reviews` | —    | Get product reviews |
| POST   | `/api/products/:id/reviews` | JWT  | Submit a review     |
| PUT    | `/api/reviews/:id`          | JWT  | Update a review     |
| DELETE | `/api/reviews/:id`          | JWT  | Delete a review     |

### Admin Routes

| Method | Endpoint                       | Auth      | Description         |
| :----- | :----------------------------- | :-------- | :------------------ |
| GET    | `/api/admin/users`             | Admin JWT | List all users      |
| PUT    | `/api/admin/users/:id`         | Admin JWT | Update user         |
| DELETE | `/api/admin/users/:id`         | Admin JWT | Delete user         |
| GET    | `/api/admin/orders`            | Admin JWT | List all orders     |
| PUT    | `/api/admin/orders/:id/status` | Admin JWT | Update order status |

---

## Direct Service APIs

When running services individually or via Dapr service invocation, each service exposes its own API.

### product-service — Port 8001

FastAPI auto-generates OpenAPI docs at `http://localhost:8001/docs`.

| Method | Endpoint                  | Auth      | Description               |
| :----- | :------------------------ | :-------- | :------------------------ |
| GET    | `/api/v1/products`        | —         | List products (paginated) |
| GET    | `/api/v1/products/:id`    | —         | Get product by ID         |
| POST   | `/api/v1/products`        | Admin JWT | Create product            |
| PUT    | `/api/v1/products/:id`    | Admin JWT | Update product            |
| DELETE | `/api/v1/products/:id`    | Admin JWT | Delete product            |
| GET    | `/api/v1/products/search` | —         | Search products           |
| GET    | `/api/v1/categories`      | —         | List categories           |

### user-service — Port 8002

| Method | Endpoint                           | Auth | Description         |
| :----- | :--------------------------------- | :--- | :------------------ |
| GET    | `/api/v1/users/:id`                | JWT  | Get user profile    |
| PUT    | `/api/v1/users/:id`                | JWT  | Update user profile |
| GET    | `/api/v1/users/:id/addresses`      | JWT  | List addresses      |
| POST   | `/api/v1/users/:id/addresses`      | JWT  | Add address         |
| PUT    | `/api/v1/users/:id/addresses/:aid` | JWT  | Update address      |
| DELETE | `/api/v1/users/:id/addresses/:aid` | JWT  | Delete address      |

### auth-service — Port 8004

| Method | Endpoint                | Auth          | Description           |
| :----- | :---------------------- | :------------ | :-------------------- |
| POST   | `/api/v1/auth/register` | —             | Register user         |
| POST   | `/api/v1/auth/login`    | —             | Login                 |
| POST   | `/api/v1/auth/refresh`  | Refresh Token | Refresh access token  |
| POST   | `/api/v1/auth/logout`   | JWT           | Logout                |
| GET    | `/api/v1/auth/verify`   | JWT           | Verify token validity |

### inventory-service — Port 8005

Flask-RESTX auto-generates Swagger docs at `http://localhost:8005/api/docs`.

| Method | Endpoint                         | Auth          | Description          |
| :----- | :------------------------------- | :------------ | :------------------- |
| GET    | `/api/v1/inventory`              | Service Token | List inventory items |
| GET    | `/api/v1/inventory/:sku`         | Service Token | Get inventory by SKU |
| PUT    | `/api/v1/inventory/:sku`         | Admin JWT     | Update stock         |
| POST   | `/api/v1/inventory/:sku/reserve` | Service Token | Reserve stock        |
| POST   | `/api/v1/inventory/:sku/release` | Service Token | Release reservation  |
| GET    | `/api/v1/inventory/low-stock`    | Admin JWT     | Get low stock items  |

### order-service — Port 8006

Swagger docs at `http://localhost:8006/swagger`.

| Method | Endpoint                    | Auth      | Description             |
| :----- | :-------------------------- | :-------- | :---------------------- |
| POST   | `/api/v1/orders`            | JWT       | Create order            |
| GET    | `/api/v1/orders`            | JWT       | List user's orders      |
| GET    | `/api/v1/orders/:id`        | JWT       | Get order by ID         |
| PUT    | `/api/v1/orders/:id/status` | Admin JWT | Update order status     |
| POST   | `/api/v1/orders/:id/cancel` | JWT       | Cancel order            |
| POST   | `/api/v1/orders/:id/return` | JWT       | Request return          |
| GET    | `/api/v1/admin/orders`      | Admin JWT | List all orders (admin) |

### cart-service — Port 8008

| Method | Endpoint                      | Auth      | Description                |
| :----- | :---------------------------- | :-------- | :------------------------- |
| GET    | `/api/v1/cart`                | X-User-Id | Get user cart              |
| POST   | `/api/v1/cart/items`          | X-User-Id | Add item                   |
| PUT    | `/api/v1/cart/items/:id`      | X-User-Id | Update quantity            |
| DELETE | `/api/v1/cart/items/:id`      | X-User-Id | Remove item                |
| DELETE | `/api/v1/cart`                | X-User-Id | Clear cart                 |
| POST   | `/api/v1/cart/transfer`       | X-User-Id | Transfer guest → user cart |
| POST   | `/api/v1/guest/cart`          | —         | Create guest cart          |
| GET    | `/api/v1/guest/cart/:guestId` | —         | Get guest cart             |

### payment-service — Port 8009

Swagger docs at `http://localhost:8009/swagger`.

| Method | Endpoint                      | Auth          | Description        |
| :----- | :---------------------------- | :------------ | :----------------- |
| POST   | `/api/v1/payments`            | Service Token | Process payment    |
| GET    | `/api/v1/payments/:id`        | JWT           | Get payment status |
| POST   | `/api/v1/payments/:id/refund` | Admin JWT     | Refund payment     |
| GET    | `/api/v1/admin/payments`      | Admin JWT     | List all payments  |

### review-service — Port 8010

| Method | Endpoint                                   | Auth | Description           |
| :----- | :----------------------------------------- | :--- | :-------------------- |
| GET    | `/api/v1/reviews/product/:productId`       | —    | Get product reviews   |
| POST   | `/api/v1/reviews`                          | JWT  | Create review         |
| PUT    | `/api/v1/reviews/:id`                      | JWT  | Update review         |
| DELETE | `/api/v1/reviews/:id`                      | JWT  | Delete review         |
| GET    | `/api/v1/reviews/product/:productId/stats` | —    | Get rating statistics |

### audit-service — Port 8012

| Method | Endpoint              | Auth      | Description                      |
| :----- | :-------------------- | :-------- | :------------------------------- |
| GET    | `/api/v1/audit`       | Admin JWT | Query audit records (filterable) |
| GET    | `/api/v1/audit/:id`   | Admin JWT | Get audit record by ID           |
| GET    | `/api/v1/audit/stats` | Admin JWT | Get audit statistics             |

### chat-service — Port 8013

| Method | Endpoint               | Auth | Description                   |
| :----- | :--------------------- | :--- | :---------------------------- |
| POST   | `/api/v1/chat`         | JWT  | Send message, get AI response |
| GET    | `/api/v1/chat/history` | JWT  | Get conversation history      |
| DELETE | `/api/v1/chat/history` | JWT  | Clear conversation            |

---

## Common Patterns

### Health Endpoints

All services expose health check endpoints:

| Endpoint            | Purpose                                                   |
| :------------------ | :-------------------------------------------------------- |
| `GET /health/live`  | Liveness probe — is the process running?                  |
| `GET /health/ready` | Readiness probe — is the service ready to accept traffic? |
| `GET /health/info`  | Service info — name, version, uptime                      |

### Error Response Format

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid input",
    "details": [{ "field": "email", "message": "Must be a valid email address" }]
  }
}
```

### Pagination

```
GET /api/v1/products?page=1&limit=20&sort=createdAt&order=desc
```

Response includes pagination metadata:

```json
{
  "data": [...],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 150,
    "pages": 8
  }
}
```

### Request Headers

| Header             | Value              | Description                        |
| :----------------- | :----------------- | :--------------------------------- |
| `Authorization`    | `Bearer <JWT>`     | Authentication token               |
| `X-Correlation-ID` | UUID               | Request correlation for tracing    |
| `X-User-Id`        | User ID            | Set by BFF for downstream services |
| `Content-Type`     | `application/json` | Request body format                |
