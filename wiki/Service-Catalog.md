# Service Catalog

Complete reference for all 16 services in the xshopai platform.

---

## Service Summary

| #   | Service                                             | Port | Language    | Framework       | Database               | Dapr Role     |
| :-- | :-------------------------------------------------- | :--- | :---------- | :-------------- | :--------------------- | :------------ |
| 1   | [product-service](#product-service)                 | 8001 | Python 3.12 | FastAPI         | MongoDB (27019)        | Hybrid        |
| 2   | [user-service](#user-service)                       | 8002 | Node.js 20  | Express 5       | MongoDB (27018)        | Publisher     |
| 3   | [admin-service](#admin-service)                     | 8003 | Node.js 20  | Express 5       | — (calls user-service) | Publisher     |
| 4   | [auth-service](#auth-service)                       | 8004 | Node.js 20  | Express 5       | — (calls user-service) | Publisher     |
| 5   | [inventory-service](#inventory-service)             | 8005 | Python 3.12 | Flask 3         | MySQL (3306)           | Hybrid        |
| 6   | [order-service](#order-service)                     | 8006 | C# 12       | ASP.NET Core 8  | SQL Server (1434)      | Publisher     |
| 7   | [order-processor-service](#order-processor-service) | 8007 | Java 17     | Spring Boot 3.3 | PostgreSQL (5435)      | Hybrid (Saga) |
| 8   | [cart-service](#cart-service)                       | 8008 | TypeScript  | Express         | Redis (6379)           | Publisher     |
| 9   | [payment-service](#payment-service)                 | 8009 | C# 12       | ASP.NET Core 8  | SQL Server (1433)      | Hybrid        |
| 10  | [review-service](#review-service)                   | 8010 | Node.js 20  | Express         | MongoDB (27020)        | Publisher     |
| 11  | [notification-service](#notification-service)       | 8011 | TypeScript  | Express         | — (stateless)          | Hybrid        |
| 12  | [audit-service](#audit-service)                     | 8012 | TypeScript  | Express         | PostgreSQL (5434)      | Consumer      |
| 13  | [chat-service](#chat-service)                       | 8013 | TypeScript  | Express         | — (stateless)          | None          |
| 14  | [web-bff](#web-bff)                                 | 8014 | TypeScript  | Express         | — (stateless)          | None          |
| 15  | [customer-ui](#customer-ui)                         | 3000 | JavaScript  | React 18 (CRA)  | —                      | —             |
| 16  | [admin-ui](#admin-ui)                               | 3001 | TypeScript  | React 18        | —                      | —             |

---

## Backend Services

### product-service

| Attribute            | Value                                                                                                                               |
| :------------------- | :---------------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/product-service](https://github.com/xshopai/product-service)                                                               |
| **Port**             | 8001                                                                                                                                |
| **Language**         | Python 3.12                                                                                                                         |
| **Framework**        | FastAPI with Uvicorn                                                                                                                |
| **Database**         | MongoDB (port 27019) via Motor async driver                                                                                         |
| **Dapr App ID**      | `product-service`                                                                                                                   |
| **Purpose**          | Product catalog CRUD, search, categories, images, review aggregation                                                                |
| **Events Published** | `product.created`, `product.updated`, `product.deleted`, `product.price.changed`, `product.badge.assigned`, `product.badge.removed` |
| **Events Consumed**  | `review.created`, `review.updated`, `review.deleted`, `inventory.stock.updated`, `inventory.out.of.stock`                           |

---

### user-service

| Attribute            | Value                                                                                                                          |
| :------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/user-service](https://github.com/xshopai/user-service)                                                                |
| **Port**             | 8002                                                                                                                           |
| **Language**         | Node.js 20 (JavaScript ESM)                                                                                                    |
| **Framework**        | Express 5.1+                                                                                                                   |
| **Database**         | MongoDB (port 27018) via Mongoose                                                                                              |
| **Dapr App ID**      | `user-service`                                                                                                                 |
| **Purpose**          | User profiles, addresses, payment methods, wishlists, preferences                                                              |
| **Events Published** | `user.created`, `user.updated`, `user.deleted`, `user.logged_in`, `user.logged_out`, `user.deactivated`, `user.email_verified` |
| **Events Consumed**  | None                                                                                                                           |

---

### admin-service

| Attribute            | Value                                                                  |
| :------------------- | :--------------------------------------------------------------------- |
| **Repo**             | [xshopai/admin-service](https://github.com/xshopai/admin-service)      |
| **Port**             | 8003                                                                   |
| **Language**         | Node.js 20 (JavaScript ESM)                                            |
| **Framework**        | Express 5.1+                                                           |
| **Database**         | None — calls user-service via Dapr for user operations                 |
| **Dapr App ID**      | `admin-service`                                                        |
| **Purpose**          | Administrative operations, user management, role administration        |
| **Events Published** | `admin.action.performed`, `admin.user.created`, `admin.config.changed` |
| **Events Consumed**  | None                                                                   |

---

### auth-service

| Attribute            | Value                                                                                                                                                           |
| :------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/auth-service](https://github.com/xshopai/auth-service)                                                                                                 |
| **Port**             | 8004                                                                                                                                                            |
| **Language**         | Node.js 20 (JavaScript ESM)                                                                                                                                     |
| **Framework**        | Express 5.1+                                                                                                                                                    |
| **Database**         | None — stateless, calls user-service via Dapr                                                                                                                   |
| **Dapr App ID**      | `auth-service`                                                                                                                                                  |
| **Purpose**          | Authentication gateway — login, registration, JWT token management                                                                                              |
| **Events Published** | `auth.login`, `auth.register`, `auth.email.verification.required`, `auth.password.reset.requested`, `auth.password.reset.completed`, `auth.account.reactivated` |
| **Events Consumed**  | None                                                                                                                                                            |

---

### inventory-service

| Attribute            | Value                                                                                                                                               |
| :------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/inventory-service](https://github.com/xshopai/inventory-service)                                                                           |
| **Port**             | 8005                                                                                                                                                |
| **Language**         | Python 3.12                                                                                                                                         |
| **Framework**        | Flask 3.0 with Flask-RESTX (Swagger auto-generated)                                                                                                 |
| **Database**         | MySQL (port 3306) via SQLAlchemy + Flask-Migrate                                                                                                    |
| **Dapr App ID**      | `inventory-service`                                                                                                                                 |
| **Purpose**          | Stock levels, reservations, stock movements, reorder alerts                                                                                         |
| **Events Published** | `inventory.stock.updated`, `inventory.reserved`, `inventory.released`, `inventory.low.stock`, `inventory.out.of.stock`, `inventory.created`         |
| **Events Consumed**  | `order.created`, `order.cancelled`, `order.completed`, `order.refunded`, `payment.received`, `payment.failed`, `product.created`, `product.deleted` |

---

### order-service

| Attribute            | Value                                                                                                                          |
| :------------------- | :----------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/order-service](https://github.com/xshopai/order-service)                                                              |
| **Port**             | 8006                                                                                                                           |
| **Language**         | C# 12 / .NET 8                                                                                                                 |
| **Framework**        | ASP.NET Core 8 Web API                                                                                                         |
| **Database**         | SQL Server (port 1434) via Entity Framework Core 8                                                                             |
| **Dapr App ID**      | `order-service`                                                                                                                |
| **Purpose**          | Order CRUD, lifecycle tracking, returns, cancellations                                                                         |
| **Events Published** | `order.created`, `order.confirmed`, `order.shipped`, `order.delivered`, `order.completed`, `order.cancelled`, `order.refunded` |
| **Events Consumed**  | None                                                                                                                           |

---

### order-processor-service

| Attribute            | Value                                                                                                                                         |
| :------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/order-processor-service](https://github.com/xshopai/order-processor-service)                                                         |
| **Port**             | 8007                                                                                                                                          |
| **Language**         | Java 17                                                                                                                                       |
| **Framework**        | Spring Boot 3.3                                                                                                                               |
| **Database**         | PostgreSQL (port 5435) via Spring Data JPA + Flyway                                                                                           |
| **Dapr App ID**      | `order-processor-service`                                                                                                                     |
| **Purpose**          | Choreography-based saga orchestrator for order fulfillment                                                                                    |
| **Events Published** | `order.processing`, `order.completed`, `order.failed`, `payment.processing`, `inventory.reserve`, `inventory.release`, `shipping.preparation` |
| **Events Consumed**  | `order.created`, `payment.received`, `payment.failed`, `inventory.reserved`, `inventory.released`, `shipping.prepared`, `shipping.failed`     |

---

### cart-service

| Attribute            | Value                                                                                               |
| :------------------- | :-------------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/cart-service](https://github.com/xshopai/cart-service)                                     |
| **Port**             | 8008                                                                                                |
| **Language**         | Node.js 20 (TypeScript)                                                                             |
| **Framework**        | Express 4.18+                                                                                       |
| **Database**         | Redis (port 6379) via ioredis / Dapr state store                                                    |
| **Dapr App ID**      | `cart-service`                                                                                      |
| **Purpose**          | Shopping cart management — user carts, guest carts, cart transfer                                   |
| **Events Published** | `cart.item.added`, `cart.item.removed`, `cart.cleared`, `cart.abandoned`                            |
| **Events Consumed**  | None                                                                                                |
| **Notes**            | Dual storage: Dapr state store for Container Apps, direct ioredis for App Service (`PLATFORM_MODE`) |

---

### payment-service

| Attribute            | Value                                                                        |
| :------------------- | :--------------------------------------------------------------------------- |
| **Repo**             | [xshopai/payment-service](https://github.com/xshopai/payment-service)        |
| **Port**             | 8009                                                                         |
| **Language**         | C# 12 / .NET 8                                                               |
| **Framework**        | ASP.NET Core 8 Web API                                                       |
| **Database**         | SQL Server (port 1433) via Entity Framework Core 8                           |
| **Dapr App ID**      | `payment-service`                                                            |
| **Purpose**          | Multi-provider payment processing (Stripe, PayPal, Square, Simulation)       |
| **Events Published** | `payment.processing`, `payment.received`, `payment.failed`, `payment.refund` |
| **Events Consumed**  | `order.created`, `order.cancelled`                                           |

---

### review-service

| Attribute            | Value                                                               |
| :------------------- | :------------------------------------------------------------------ |
| **Repo**             | [xshopai/review-service](https://github.com/xshopai/review-service) |
| **Port**             | 8010                                                                |
| **Language**         | Node.js 20 (JavaScript ESM)                                         |
| **Framework**        | Express with Mongoose                                               |
| **Database**         | MongoDB (port 27020)                                                |
| **Dapr App ID**      | `review-service`                                                    |
| **Purpose**          | Product reviews, ratings, moderation, aggregated statistics         |
| **Events Published** | `review.created`, `review.updated`, `review.deleted`                |
| **Events Consumed**  | None                                                                |

---

### notification-service

| Attribute            | Value                                                                                            |
| :------------------- | :----------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/notification-service](https://github.com/xshopai/notification-service)                  |
| **Port**             | 8011                                                                                             |
| **Language**         | Node.js 20 (TypeScript)                                                                          |
| **Framework**        | Express                                                                                          |
| **Database**         | None (stateless)                                                                                 |
| **Dapr App ID**      | `notification-service`                                                                           |
| **Purpose**          | Event-driven notification delivery — email via SMTP (Mailpit dev) / Azure Communication Services |
| **Events Published** | `notification.sent`, `notification.failed`                                                       |
| **Events Consumed**  | 20+ events from auth, user, order, payment, inventory, cart domains                              |

---

### audit-service

| Attribute            | Value                                                                                          |
| :------------------- | :--------------------------------------------------------------------------------------------- |
| **Repo**             | [xshopai/audit-service](https://github.com/xshopai/audit-service)                              |
| **Port**             | 8012                                                                                           |
| **Language**         | Node.js 20 (TypeScript)                                                                        |
| **Framework**        | Express                                                                                        |
| **Database**         | PostgreSQL (port 5434) via Knex.js                                                             |
| **Dapr App ID**      | `audit-service`                                                                                |
| **Purpose**          | Terminal event consumer — immutable audit trail for all platform events                        |
| **Events Published** | None                                                                                           |
| **Events Consumed**  | 40+ events from all domains (universal audit sink)                                             |
| **Notes**            | Append-only — no UPDATE or DELETE operations. Idempotent handlers with event ID deduplication. |

---

### chat-service

| Attribute       | Value                                                                                                  |
| :-------------- | :----------------------------------------------------------------------------------------------------- |
| **Repo**        | [xshopai/chat-service](https://github.com/xshopai/chat-service)                                        |
| **Port**        | 8013                                                                                                   |
| **Language**    | Node.js 20 (TypeScript)                                                                                |
| **Framework**   | Express                                                                                                |
| **Database**    | None (stateless, in-memory conversation history)                                                       |
| **Dapr App ID** | `chat-service`                                                                                         |
| **Purpose**     | AI-powered shopping assistant using Azure OpenAI GPT-4o with function calling                          |
| **AI Tools**    | `searchProducts`, `getProductDetails`, `getCategories`, `getMyOrders`, `getOrderDetails`, `trackOrder` |
| **Events**      | None (HTTP-only)                                                                                       |

---

### web-bff

| Attribute       | Value                                                                                           |
| :-------------- | :---------------------------------------------------------------------------------------------- |
| **Repo**        | [xshopai/web-bff](https://github.com/xshopai/web-bff)                                           |
| **Port**        | 8014                                                                                            |
| **Language**    | Node.js 20 (TypeScript)                                                                         |
| **Framework**   | Express 4.19+                                                                                   |
| **Database**    | None (stateless gateway)                                                                        |
| **Dapr App ID** | `web-bff`                                                                                       |
| **Purpose**     | Backend for Frontend — aggregates microservice calls into UI-optimized responses                |
| **Downstream**  | auth (8004), user (8002), product (8001), order (8006), cart (8008), inventory (8005)           |
| **Notes**       | Supports `PLATFORM_MODE=dapr` (Dapr SDK) or `PLATFORM_MODE=direct` (HTTP with service resolver) |

---

## Frontend Applications

### customer-ui

| Attribute     | Value                                                                         |
| :------------ | :---------------------------------------------------------------------------- |
| **Repo**      | [xshopai/customer-ui](https://github.com/xshopai/customer-ui)                 |
| **Port**      | 3000                                                                          |
| **Language**  | JavaScript (JSX)                                                              |
| **Framework** | React 18.2 with Create React App                                              |
| **State**     | Redux Toolkit + Zustand + TanStack Query v5                                   |
| **Styling**   | TailwindCSS 3 + Heroicons                                                     |
| **API**       | Axios → web-bff (port 8014)                                                   |
| **Purpose**   | Customer-facing storefront — product browsing, cart, checkout, order tracking |

---

### admin-ui

| Attribute     | Value                                                                            |
| :------------ | :------------------------------------------------------------------------------- |
| **Repo**      | [xshopai/admin-ui](https://github.com/xshopai/admin-ui)                          |
| **Port**      | 3001                                                                             |
| **Language**  | TypeScript (TSX)                                                                 |
| **Framework** | React 18.2 with react-app-rewired                                                |
| **State**     | Redux Toolkit + Zustand + TanStack Query v5                                      |
| **Styling**   | TailwindCSS 3 + HeadlessUI + Heroicons                                           |
| **Charts**    | Recharts                                                                         |
| **Tables**    | TanStack Table v8                                                                |
| **API**       | Axios → web-bff (port 8014)                                                      |
| **Purpose**   | Admin portal — user management, order processing, inventory oversight, analytics |

---

## Supporting Repositories

| Repo                                                                | Purpose                                                    |
| :------------------------------------------------------------------ | :--------------------------------------------------------- |
| [xshopai/dev](https://github.com/xshopai/dev)                       | Docker Compose for local infrastructure, VS Code workspace |
| [xshopai/docs](https://github.com/xshopai/docs)                     | Central documentation, architecture guides, wiki           |
| [xshopai/infrastructure](https://github.com/xshopai/infrastructure) | IaC — Bicep templates, ACA scripts, deployment workflows   |
| [xshopai/db-seeder](https://github.com/xshopai/db-seeder)           | Database seeding utility for demo/development environments |
| [xshopai/.github](https://github.com/xshopai/.github)               | Org profile, issue templates, shared GitHub config         |
