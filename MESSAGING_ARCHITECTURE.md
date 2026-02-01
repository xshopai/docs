# xshopai Messaging Architecture

> **Document Version**: 1.0  
> **Last Updated**: February 1, 2026  
> **Status**: Active

## Table of Contents

- [Overview](#overview)
- [Message Broker Infrastructure](#message-broker-infrastructure)
- [Service Communication Matrix](#service-communication-matrix)
- [Event Catalog](#event-catalog)
- [Architecture Diagrams](#architecture-diagrams)
- [Event Flow Patterns](#event-flow-patterns)
- [Configuration Reference](#configuration-reference)
- [Troubleshooting](#troubleshooting)

---

## Overview

The xshopai platform uses **event-driven architecture** for asynchronous inter-service communication. All services communicate through **Dapr Pub/Sub** abstraction layer, with **RabbitMQ** as the underlying message broker.

### Key Principles

| Principle                  | Description                                                                  |
| -------------------------- | ---------------------------------------------------------------------------- |
| **Dapr Abstraction**       | All pub/sub operations go through Dapr sidecars, enabling broker portability |
| **CloudEvents Format**     | All events follow CloudEvents specification (v1.0) for interoperability      |
| **Topic-Based Routing**    | Events are published to topics; consumers subscribe to relevant topics       |
| **Fire-and-Forget**        | Publishers don't wait for consumer acknowledgment (async decoupling)         |
| **At-Least-Once Delivery** | RabbitMQ ensures messages are delivered at least once                        |
| **Idempotent Handlers**    | All event handlers must be idempotent to handle retries safely               |

### Communication Patterns

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         xshopai Event-Driven Architecture                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │   Service   │     │    Dapr     │     │  RabbitMQ   │                   │
│  │  (Publisher)│────▶│   Sidecar   │────▶│   Broker    │                   │
│  └─────────────┘     └─────────────┘     └──────┬──────┘                   │
│                                                  │                          │
│                                                  │ Topic Routing            │
│                                                  ▼                          │
│  ┌─────────────┐     ┌─────────────┐     ┌─────────────┐                   │
│  │   Service   │◀────│    Dapr     │◀────│   Fanout    │                   │
│  │  (Consumer) │     │   Sidecar   │     │   Exchange  │                   │
│  └─────────────┘     └─────────────┘     └─────────────┘                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Message Broker Infrastructure

### Dapr Pub/Sub Configuration

All services use a consistent Dapr pub/sub component named `xshopai-pubsub`:

```yaml
# Standard Dapr Pub/Sub Component Configuration
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: xshopai-pubsub
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
    - name: connectionString
      value: 'amqp://admin:admin123@127.0.0.1:5672'
    - name: exchangeName
      value: 'xshopai.events'
    - name: exchangeKind
      value: 'topic'
    - name: durable
      value: 'true'
    - name: autoAck
      value: 'false'
    - name: deliveryMode
      value: '2' # Persistent
    - name: requeueInFailure
      value: 'true'
    - name: prefetchCount
      value: '10'
```

### RabbitMQ Configuration

| Setting           | Value                                     | Description              |
| ----------------- | ----------------------------------------- | ------------------------ |
| **Host**          | `127.0.0.1` (local) / `rabbitmq` (Docker) | Broker hostname          |
| **Port**          | `5672` (AMQP) / `15672` (Management)      | Connection ports         |
| **Username**      | `admin`                                   | Default username         |
| **Password**      | `admin123`                                | Default password         |
| **Exchange**      | `xshopai.events`                          | Main topic exchange      |
| **Exchange Type** | `topic`                                   | Enables wildcard routing |

### Service Ports Reference

| Service                 | App Port | Dapr HTTP Port | Dapr gRPC Port |
| ----------------------- | -------- | -------------- | -------------- |
| admin-service           | 8080     | 3500           | 50001          |
| audit-service           | 8015     | 3500           | 50001          |
| auth-service            | 8003     | 3500           | 50001          |
| cart-service            | 8004     | 3500           | 50001          |
| inventory-service       | 8005     | 3500           | 50001          |
| notification-service    | 8010     | 3500           | 50001          |
| order-processor-service | 8007     | 3500           | 50001          |
| order-service           | 8006     | 3500           | 50001          |
| payment-service         | 8009     | 3500           | 50001          |
| product-service         | 8001     | 3500           | 50001          |
| review-service          | 8008     | 3500           | 50001          |
| user-service            | 8002     | 3500           | 50001          |

---

## Service Communication Matrix

### Publishers vs Consumers Overview

```
                            ┌───────────────────────────────────────────────────────────────┐
                            │                    SERVICE ROLE MATRIX                         │
                            ├───────────────────────────────────────────────────────────────┤
                            │                                                                │
    PURE PUBLISHERS         │    HYBRID (PUB + SUB)        │    PURE CONSUMERS              │
    (Fire & Forget)         │    (Saga Participants)       │    (Event Sinks)               │
                            │                              │                                 │
    ┌─────────────────┐     │    ┌─────────────────┐      │    ┌─────────────────┐         │
    │  auth-service   │     │    │inventory-service│      │    │  audit-service  │         │
    │  (6 events)     │     │    │  (6 pub/6 sub)  │      │    │  (40+ events)   │         │
    └─────────────────┘     │    └─────────────────┘      │    └─────────────────┘         │
                            │                              │                                 │
    ┌─────────────────┐     │    ┌─────────────────┐      │                                │
    │  user-service   │     │    │ product-service │      │                                │
    │  (7 events)     │     │    │  (6 pub/5 sub)  │      │                                │
    └─────────────────┘     │    └─────────────────┘      │                                │
                            │                              │                                 │
    ┌─────────────────┐     │    ┌─────────────────┐      │                                │
    │  order-service  │     │    │payment-service  │      │                                │
    │  (5 events)     │     │    │  (4 pub/2 sub)  │      │                                │
    └─────────────────┘     │    └─────────────────┘      │                                │
                            │                              │                                 │
    ┌─────────────────┐     │    ┌─────────────────┐      │                                │
    │ review-service  │     │    │order-processor  │      │                                │
    │  (3 events)     │     │    │ (12 pub/7 sub)  │      │                                │
    └─────────────────┘     │    └─────────────────┘      │                                │
                            │                              │                                 │
    ┌─────────────────┐     │    ┌─────────────────┐      │                                │
    │  admin-service  │     │    │notification-svc │      │                                │
    │  (3 events)     │     │    │  (2 pub/20 sub) │      │                                │
    └─────────────────┘     │    └─────────────────┘      │                                │
                            │                              │                                 │
    ┌─────────────────┐     │                              │                                │
    │  cart-service   │     │                              │                                │
    │  (4 events)     │     │                              │                                │
    └─────────────────┘     │                              │                                │
                            └───────────────────────────────────────────────────────────────┘
```

### Detailed Publisher/Consumer Matrix

| Service                     | Technology | Role      | Events Published | Events Consumed |
| --------------------------- | ---------- | --------- | ---------------- | --------------- |
| **admin-service**           | Node.js    | Publisher | 3                | 0               |
| **audit-service**           | TypeScript | Consumer  | 0                | 40+             |
| **auth-service**            | Node.js    | Publisher | 6                | 0               |
| **cart-service**            | Java       | Publisher | 4                | 0               |
| **inventory-service**       | Python     | Hybrid    | 6                | 6               |
| **notification-service**    | TypeScript | Hybrid    | 2                | 20+             |
| **order-processor-service** | Java       | Hybrid    | 12               | 7               |
| **order-service**           | C# .NET    | Publisher | 5                | 0               |
| **payment-service**         | C# .NET    | Hybrid    | 4                | 2               |
| **product-service**         | Python     | Hybrid    | 6                | 5               |
| **review-service**          | Node.js    | Publisher | 3                | 0               |
| **user-service**            | Node.js    | Publisher | 7                | 0               |
| **web-bff**                 | TypeScript | None      | 0                | 0               |
| **chat-service**            | TypeScript | None      | 0                | 0               |

---

## Event Catalog

### Authentication Events (auth-service)

| Event Type                         | Topic                              | Description                 | Consumer(s)                         |
| ---------------------------------- | ---------------------------------- | --------------------------- | ----------------------------------- |
| `auth.login`                       | `auth.login`                       | User successfully logged in | audit-service, notification-service |
| `auth.register`                    | `auth.register`                    | New user registered         | audit-service, notification-service |
| `auth.email.verification.required` | `auth.email.verification.required` | Email verification needed   | audit-service, notification-service |
| `auth.password.reset.requested`    | `auth.password.reset.requested`    | Password reset initiated    | audit-service, notification-service |
| `auth.password.reset.completed`    | `auth.password.reset.completed`    | Password successfully reset | audit-service, notification-service |
| `auth.account.reactivated`         | `auth.account.reactivated`         | Account reactivated         | audit-service, notification-service |

### User Events (user-service)

| Event Type            | Topic                 | Description              | Consumer(s)                         |
| --------------------- | --------------------- | ------------------------ | ----------------------------------- |
| `user.created`        | `user.created`        | New user profile created | audit-service, notification-service |
| `user.updated`        | `user.updated`        | User profile updated     | audit-service, notification-service |
| `user.deleted`        | `user.deleted`        | User account deleted     | audit-service, notification-service |
| `user.logged_in`      | `user.logged_in`      | User login recorded      | audit-service                       |
| `user.logged_out`     | `user.logged_out`     | User logout recorded     | audit-service                       |
| `user.deactivated`    | `user.deactivated`    | Account deactivated      | audit-service, notification-service |
| `user.email_verified` | `user.email_verified` | Email verified           | audit-service, notification-service |

### Order Events (order-service)

| Event Type             | Topic                  | Description          | Consumer(s)                                                                                      |
| ---------------------- | ---------------------- | -------------------- | ------------------------------------------------------------------------------------------------ |
| `order.created`        | `order.created`        | New order placed     | order-processor-service, payment-service, inventory-service, audit-service, notification-service |
| `order.updated`        | `order.updated`        | Order modified       | audit-service, notification-service                                                              |
| `order.cancelled`      | `order.cancelled`      | Order cancelled      | payment-service, inventory-service, audit-service, notification-service                          |
| `order.completed`      | `order.completed`      | Order fulfilled      | audit-service, notification-service                                                              |
| `order.status.changed` | `order.status.changed` | Order status updated | audit-service, notification-service                                                              |

### Payment Events (payment-service)

| Event Type           | Topic                | Description             | Consumer(s)                                                                     |
| -------------------- | -------------------- | ----------------------- | ------------------------------------------------------------------------------- |
| `payment.processing` | `payment.processing` | Payment being processed | audit-service                                                                   |
| `payment.received`   | `payment.received`   | Payment successful      | order-processor-service, inventory-service, audit-service, notification-service |
| `payment.failed`     | `payment.failed`     | Payment failed          | order-processor-service, audit-service, notification-service                    |
| `payment.refund`     | `payment.refund`     | Refund processed        | audit-service, notification-service                                             |

### Inventory Events (inventory-service)

| Event Type                | Topic                     | Description              | Consumer(s)                                          |
| ------------------------- | ------------------------- | ------------------------ | ---------------------------------------------------- |
| `inventory.stock.updated` | `inventory.stock.updated` | Stock quantity changed   | product-service, audit-service                       |
| `inventory.reserved`      | `inventory.reserved`      | Stock reserved for order | order-processor-service, audit-service               |
| `inventory.released`      | `inventory.released`      | Reserved stock released  | order-processor-service, audit-service               |
| `inventory.low.stock`     | `inventory.low.stock`     | Low stock alert          | notification-service, audit-service                  |
| `inventory.out.of.stock`  | `inventory.out.of.stock`  | Out of stock alert       | product-service, notification-service, audit-service |
| `inventory.created`       | `inventory.created`       | New inventory record     | audit-service                                        |

### Product Events (product-service)

| Event Type               | Topic                    | Description       | Consumer(s)                         |
| ------------------------ | ------------------------ | ----------------- | ----------------------------------- |
| `product.created`        | `product.created`        | New product added | inventory-service, audit-service    |
| `product.updated`        | `product.updated`        | Product modified  | inventory-service, audit-service    |
| `product.deleted`        | `product.deleted`        | Product removed   | inventory-service, audit-service    |
| `product.price.changed`  | `product.price.changed`  | Price updated     | audit-service, notification-service |
| `product.badge.assigned` | `product.badge.assigned` | Badge added       | audit-service                       |
| `product.badge.removed`  | `product.badge.removed`  | Badge removed     | audit-service                       |

### Review Events (review-service)

| Event Type       | Topic            | Description          | Consumer(s)                    |
| ---------------- | ---------------- | -------------------- | ------------------------------ |
| `review.created` | `review.created` | New review submitted | product-service, audit-service |
| `review.updated` | `review.updated` | Review modified      | product-service, audit-service |
| `review.deleted` | `review.deleted` | Review removed       | product-service, audit-service |

### Cart Events (cart-service)

| Event Type          | Topic               | Description            | Consumer(s)                         |
| ------------------- | ------------------- | ---------------------- | ----------------------------------- |
| `cart.item.added`   | `cart.item.added`   | Item added to cart     | audit-service                       |
| `cart.item.removed` | `cart.item.removed` | Item removed from cart | audit-service                       |
| `cart.cleared`      | `cart.cleared`      | Cart emptied           | audit-service                       |
| `cart.abandoned`    | `cart.abandoned`    | Cart abandoned         | notification-service, audit-service |

### Admin Events (admin-service)

| Event Type               | Topic                    | Description            | Consumer(s)                         |
| ------------------------ | ------------------------ | ---------------------- | ----------------------------------- |
| `admin.action.performed` | `admin.action.performed` | Admin action logged    | audit-service                       |
| `admin.user.created`     | `admin.user.created`     | Admin created user     | audit-service, notification-service |
| `admin.config.changed`   | `admin.config.changed`   | Configuration modified | audit-service                       |

### Notification Events (notification-service)

| Event Type            | Topic                 | Description            | Consumer(s)   |
| --------------------- | --------------------- | ---------------------- | ------------- |
| `notification.sent`   | `notification.sent`   | Notification delivered | audit-service |
| `notification.failed` | `notification.failed` | Delivery failed        | audit-service |

---

## Architecture Diagrams

### Complete Event Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                    xshopai EVENT FLOW ARCHITECTURE                                       │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                          │
│    ╔═══════════════════════════════════════════════════════════════════════════════════════════════╗    │
│    ║                                     RABBITMQ MESSAGE BROKER                                     ║    │
│    ║                                  (xshopai.events topic exchange)                                ║    │
│    ╚═══════════════════════════════════════════════════════════════════════════════════════════════╝    │
│           ▲           ▲           ▲           ▲           ▲           ▲           │           │         │
│           │           │           │           │           │           │           │           │         │
│    ┌──────┴───┐ ┌─────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐     │           │         │
│    │  auth.*  │ │  user.*  │ │ order.* │ │payment.*│ │inventory│ │product.*│     ▼           ▼         │
│    │  events  │ │  events  │ │ events  │ │ events  │ │  .*     │ │ events  │ ┌───────┐ ┌───────────┐   │
│    └──────┬───┘ └─────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘ │audit- │ │notificat- │   │
│           │           │           │           │           │           │       │service│ │ion-service│   │
│           │           │           │           │           │           │       │(40+)  │ │  (20+)    │   │
│    ┌──────┴───┐ ┌─────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ ┌────┴────┐ └───────┘ └───────────┘   │
│    │  AUTH    │ │   USER   │ │  ORDER  │ │PAYMENT  │ │INVENTORY│ │PRODUCT  │                            │
│    │ SERVICE  │ │ SERVICE  │ │ SERVICE │ │ SERVICE │ │ SERVICE │ │ SERVICE │                            │
│    │(Node.js) │ │(Node.js) │ │ (.NET)  │ │ (.NET)  │ │(Python) │ │(Python) │                            │
│    └──────────┘ └──────────┘ └────┬────┘ └────┬────┘ └────┬────┘ └────┬────┘                            │
│                                   │           │           │           │                                  │
│                                   │           │           │           │                                  │
│                                   ▼           ▼           ▼           ▼                                  │
│                              ┌─────────────────────────────────────────────┐                            │
│                              │          ORDER PROCESSOR SERVICE             │                            │
│                              │              (Saga Coordinator)              │                            │
│                              │                  (Java)                      │                            │
│                              │                                              │                            │
│                              │  Subscribes to:                              │                            │
│                              │  • order.created → Start payment saga        │                            │
│                              │  • payment.received → Reserve inventory      │                            │
│                              │  • payment.failed → Cancel order             │                            │
│                              │  • inventory.reserved → Prepare shipping     │                            │
│                              │  • inventory.released → Handle failure       │                            │
│                              │  • shipping.prepared → Complete order        │                            │
│                              │  • shipping.failed → Compensate              │                            │
│                              │                                              │                            │
│                              │  Publishes:                                  │                            │
│                              │  • payment.processing                        │                            │
│                              │  • inventory.reserve / release               │                            │
│                              │  • shipping.preparation                      │                            │
│                              │  • order.completed / order.failed            │                            │
│                              └─────────────────────────────────────────────┘                            │
│                                                                                                          │
│    ┌──────────┐ ┌──────────┐ ┌──────────┐                                                               │
│    │  REVIEW  │ │   CART   │ │  ADMIN   │                                                               │
│    │ SERVICE  │ │ SERVICE  │ │ SERVICE  │                                                               │
│    │(Node.js) │ │  (Java)  │ │(Node.js) │                                                               │
│    └────┬─────┘ └────┬─────┘ └────┬─────┘                                                               │
│         │            │            │                                                                      │
│         ▼            ▼            ▼                                                                      │
│    ┌─────────────────────────────────────┐                                                              │
│    │         ALL EVENTS → AUDIT          │                                                              │
│    └─────────────────────────────────────┘                                                              │
│                                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Order Processing Saga Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                      ORDER PROCESSING SAGA FLOW                                          │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                          │
│    Customer                                                                                              │
│       │                                                                                                  │
│       │ POST /api/orders                                                                                 │
│       ▼                                                                                                  │
│  ┌───────────┐                                                                                          │
│  │  ORDER    │──────── order.created ────────┐                                                          │
│  │  SERVICE  │                               │                                                          │
│  └───────────┘                               ▼                                                          │
│                                        ┌───────────┐                                                    │
│                                        │  ORDER    │                                                    │
│                                        │ PROCESSOR │                                                    │
│                                        └─────┬─────┘                                                    │
│                                              │                                                          │
│                           ┌──────────────────┼──────────────────┐                                       │
│                           │                  │                  │                                       │
│                           ▼                  │                  │                                       │
│                    ┌───────────┐             │                  │                                       │
│                    │  PAYMENT  │◀── payment.processing ─────────┘                                       │
│                    │  SERVICE  │                                                                        │
│                    └─────┬─────┘                                                                        │
│                          │                                                                              │
│              ┌───────────┴───────────┐                                                                  │
│              │                       │                                                                  │
│              ▼                       ▼                                                                  │
│       payment.received         payment.failed                                                           │
│              │                       │                                                                  │
│              ▼                       ▼                                                                  │
│        ┌───────────┐          ┌───────────┐                                                            │
│        │ INVENTORY │          │  ORDER    │──── order.cancelled ────▶ (End - Failure)                  │
│        │  SERVICE  │          │ PROCESSOR │                                                            │
│        └─────┬─────┘          └───────────┘                                                            │
│              │                                                                                          │
│              │◀── inventory.reserve ──────────────────────┐                                             │
│              │                                            │                                             │
│    ┌─────────┴─────────┐                                  │                                             │
│    │                   │                                  │                                             │
│    ▼                   ▼                                  │                                             │
│ inventory.reserved  inventory.failed                      │                                             │
│    │                   │                                  │                                             │
│    │                   ▼                                  │                                             │
│    │            ┌───────────┐                             │                                             │
│    │            │  PAYMENT  │──── payment.refund ────▶ (Compensation)                                  │
│    │            │  SERVICE  │                                                                          │
│    │            └───────────┘                                                                          │
│    │                                                                                                    │
│    ▼                                                                                                    │
│  ┌───────────┐                                                                                          │
│  │  SHIPPING │◀── shipping.preparation ───────────────────┐                                             │
│  │  (future) │                                            │                                             │
│  └─────┬─────┘                                            │                                             │
│        │                                                  │                                             │
│        ▼                                                  │                                             │
│  shipping.prepared                                        │                                             │
│        │                                                  │                                             │
│        ▼                                                  │                                             │
│  ┌───────────┐                                            │                                             │
│  │  ORDER    │──── order.completed ────▶ (End - Success)  │                                             │
│  │ PROCESSOR │                                            │                                             │
│  └───────────┘                                            │                                             │
│                                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Product-Review-Inventory Sync Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                 PRODUCT DATA SYNCHRONIZATION FLOW                                        │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                          │
│                              ┌─────────────────────────────────────────┐                                │
│                              │          PRODUCT SERVICE                 │                                │
│                              │     (Denormalized Data Aggregator)       │                                │
│                              │                                          │                                │
│                              │  Stores:                                 │                                │
│                              │  • Product base info                     │                                │
│                              │  • Review aggregates (avg rating, count) │                                │
│                              │  • Availability status (from inventory)  │                                │
│                              │  • Sales rank (from analytics)           │                                │
│                              │                                          │                                │
│                              └────────────────┬────────────────────────┘                                │
│                                               │                                                          │
│                                               │ Subscribes to:                                           │
│                        ┌──────────────────────┼──────────────────────┐                                  │
│                        │                      │                      │                                  │
│                        ▼                      ▼                      ▼                                  │
│               ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐                        │
│               │  review.created │    │ inventory.stock │    │   analytics.*   │                        │
│               │  review.updated │    │    .updated     │    │ (future)        │                        │
│               │  review.deleted │    │                 │    │                 │                        │
│               └────────┬────────┘    └────────┬────────┘    └────────┬────────┘                        │
│                        │                      │                      │                                  │
│                        │                      │                      │                                  │
│               ┌────────┴────────┐    ┌────────┴────────┐    ┌────────┴────────┐                        │
│               │     REVIEW      │    │   INVENTORY     │    │   ANALYTICS     │                        │
│               │    SERVICE      │    │    SERVICE      │    │   (future)      │                        │
│               │   (Node.js)     │    │    (Python)     │    │                 │                        │
│               └─────────────────┘    └─────────────────┘    └─────────────────┘                        │
│                                                                                                          │
│  ═══════════════════════════════════════════════════════════════════════════════════════════════════   │
│                                                                                                          │
│  Example Event Flow:                                                                                     │
│                                                                                                          │
│  1. Customer submits review                                                                              │
│     │                                                                                                    │
│     ▼                                                                                                    │
│  ┌─────────────────┐                                                                                    │
│  │ review-service  │ ──── review.created ────▶ RabbitMQ                                                 │
│  └─────────────────┘              │                                                                     │
│                                   │                                                                     │
│                                   ▼                                                                     │
│                          ┌─────────────────┐                                                            │
│                          │ product-service │                                                            │
│                          │                 │                                                            │
│                          │ Updates:        │                                                            │
│                          │ • reviewCount   │                                                            │
│                          │ • averageRating │                                                            │
│                          │ • ratingDist[]  │                                                            │
│                          └─────────────────┘                                                            │
│                                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

### Audit Trail Collection Flow

```
┌─────────────────────────────────────────────────────────────────────────────────────────────────────────┐
│                                        AUDIT TRAIL COLLECTION                                            │
├─────────────────────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                                          │
│                                    ╔═════════════════════════════════╗                                  │
│                                    ║       AUDIT SERVICE              ║                                  │
│                                    ║     (Central Event Sink)         ║                                  │
│                                    ║                                  ║                                  │
│                                    ║  Stores ALL platform events      ║                                  │
│                                    ║  for compliance and debugging    ║                                  │
│                                    ╚═════════════════════════════════╝                                  │
│                                                   ▲                                                      │
│                                                   │                                                      │
│              ┌────────────────────────────────────┼────────────────────────────────────┐                │
│              │                    │               │               │                    │                │
│              │                    │               │               │                    │                │
│       ┌──────┴──────┐      ┌──────┴──────┐ ┌─────┴─────┐  ┌──────┴──────┐      ┌──────┴──────┐        │
│       │   auth.*    │      │   user.*    │ │  order.*  │  │ payment.*   │      │ inventory.* │        │
│       │   events    │      │   events    │ │  events   │  │   events    │      │   events    │        │
│       └──────┬──────┘      └──────┬──────┘ └─────┬─────┘  └──────┬──────┘      └──────┬──────┘        │
│              │                    │               │               │                    │                │
│       ┌──────┴──────┐      ┌──────┴──────┐ ┌─────┴─────┐  ┌──────┴──────┐      ┌──────┴──────┐        │
│       │  product.*  │      │  review.*   │ │  cart.*   │  │   admin.*   │      │notification*│        │
│       │   events    │      │   events    │ │  events   │  │   events    │      │   events    │        │
│       └─────────────┘      └─────────────┘ └───────────┘  └─────────────┘      └─────────────┘        │
│                                                                                                          │
│  ═══════════════════════════════════════════════════════════════════════════════════════════════════   │
│                                                                                                          │
│  Subscribed Topics (40+):                                                                                │
│                                                                                                          │
│  Auth Domain:        User Domain:         Order Domain:        Payment Domain:                          │
│  • auth.login        • user.created       • order.created      • payment.received                       │
│  • auth.register     • user.updated       • order.updated      • payment.failed                         │
│  • auth.logout       • user.deleted       • order.cancelled    • payment.refund                         │
│  • auth.password.*   • user.deactivated   • order.completed                                             │
│                                                                                                          │
│  Inventory Domain:   Product Domain:      Review Domain:       Cart Domain:                             │
│  • inventory.*       • product.created    • review.created     • cart.item.added                        │
│                      • product.updated    • review.updated     • cart.item.removed                      │
│                      • product.deleted    • review.deleted     • cart.cleared                           │
│                      • product.price.*                         • cart.abandoned                         │
│                                                                                                          │
└─────────────────────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Event Flow Patterns

### Pattern 1: Fire-and-Forget (Most Common)

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Service A  │────▶│   Dapr      │────▶│  RabbitMQ   │
│ (Publisher) │     │  Sidecar    │     │   Broker    │
└─────────────┘     └─────────────┘     └─────────────┘
      │                                        │
      │ HTTP 200 OK (immediate)                │
      ◀────────────────────────────────────────┘
                                               │
                                               ▼
                                    (Async delivery to consumers)
```

**Used by:** auth-service, user-service, order-service, review-service, admin-service, cart-service

### Pattern 2: Saga Coordination

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Order     │     │   Order     │     │  Payment    │
│  Service    │────▶│  Processor  │────▶│  Service    │
└─────────────┘     └──────┬──────┘     └──────┬──────┘
                          │                    │
                          │◀─── payment.* ─────┘
                          │
                          ▼
                    ┌─────────────┐
                    │  Inventory  │
                    │   Service   │
                    └──────┬──────┘
                          │
                          │◀─── inventory.* ───┘
                          │
                    (Continue saga...)
```

**Used by:** order-processor-service (saga coordinator)

### Pattern 3: Data Synchronization

```
┌─────────────┐                      ┌─────────────┐
│   Review    │──── review.* ───────▶│   Product   │
│  Service    │                      │   Service   │
└─────────────┘                      │             │
                                     │  Updates:   │
┌─────────────┐                      │  - ratings  │
│  Inventory  │──── inventory.* ────▶│  - stock    │
│  Service    │                      │  - badges   │
└─────────────┘                      └─────────────┘
```

**Used by:** product-service (data aggregation)

---

## Configuration Reference

### Dapr Component: xshopai-pubsub

Every service uses this standard configuration in `.dapr/components/event-bus.yaml`:

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: xshopai-pubsub
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
    - name: connectionString
      value: 'amqp://admin:admin123@127.0.0.1:5672'
    - name: exchangeName
      value: 'xshopai.events'
    - name: exchangeKind
      value: 'topic'
    - name: durable
      value: 'true'
    - name: autoAck
      value: 'false'
    - name: deliveryMode
      value: '2'
    - name: requeueInFailure
      value: 'true'
    - name: prefetchCount
      value: '10'
```

### Dapr Subscription Example

Services that consume events have a `subscriptions.yaml`:

```yaml
apiVersion: dapr.io/v2alpha1
kind: Subscription
metadata:
  name: order-events-subscription
spec:
  pubsubname: xshopai-pubsub
  topic: order.created
  routes:
    default: /events/order-created
scopes:
  - order-processor-service
```

### CloudEvents Payload Format

All events follow CloudEvents v1.0 specification:

```json
{
  "specversion": "1.0",
  "type": "com.xshopai.user.created",
  "source": "user-service",
  "id": "1706790000000-abc123",
  "time": "2026-02-01T10:00:00Z",
  "datacontenttype": "application/json",
  "data": {
    "userId": "user-123",
    "email": "john@example.com",
    "firstName": "John",
    "lastName": "Doe"
  },
  "metadata": {
    "traceId": "trace-abc-123",
    "correlationId": "corr-xyz-456",
    "environment": "production"
  }
}
```

---

## Troubleshooting

### Common Issues

#### 1. Events Not Being Received

**Symptoms:** Consumer service not receiving events

**Checklist:**

- [ ] Verify Dapr sidecar is running (`dapr list`)
- [ ] Check pub/sub component name matches (`xshopai-pubsub`)
- [ ] Verify topic name in subscription matches publisher
- [ ] Check RabbitMQ management UI for queue bindings
- [ ] Review Dapr sidecar logs for errors

```bash
# Check Dapr sidecar logs
dapr logs --app-id <service-name>

# Verify pub/sub component
curl http://localhost:3500/v1.0/metadata
```

#### 2. RabbitMQ Connection Failed

**Symptoms:** "Connection refused" errors

**Checklist:**

- [ ] Verify RabbitMQ is running: `docker ps | grep rabbitmq`
- [ ] Check connection string in component YAML
- [ ] Verify credentials (admin/admin123)
- [ ] Check port 5672 is accessible

```bash
# Test RabbitMQ connection
curl -u admin:admin123 http://localhost:15672/api/overview
```

#### 3. Duplicate Event Processing

**Symptoms:** Events processed multiple times

**Solution:** Ensure event handlers are idempotent:

- Use unique event IDs for deduplication
- Check if operation already applied before processing
- Use database transactions with idempotency keys

#### 4. Event Schema Mismatch

**Symptoms:** Consumer fails to parse event data

**Checklist:**

- [ ] Verify CloudEvents format
- [ ] Check `datacontenttype` is `application/json`
- [ ] Ensure `data` field matches expected schema
- [ ] Review producer logs for serialization errors

### Monitoring Commands

```bash
# List all running Dapr applications
dapr list

# View Dapr dashboard
dapr dashboard

# Check RabbitMQ queues
curl -u admin:admin123 http://localhost:15672/api/queues

# View RabbitMQ exchanges
curl -u admin:admin123 http://localhost:15672/api/exchanges

# Publish test event via Dapr
curl -X POST http://localhost:3500/v1.0/publish/xshopai-pubsub/test.topic \
  -H "Content-Type: application/json" \
  -d '{"test": "message"}'
```

---

## Future Enhancements

The following services may benefit from pub/sub integration in the future. GitHub issues have been created to track these:

| Service          | Current State  | GitHub Issue                                                               |
| ---------------- | -------------- | -------------------------------------------------------------------------- |
| **cart-service** | Pure publisher | [xshopai/cart-service#1](https://github.com/xshopai/cart-service/issues/1) |
| **chat-service** | No pub/sub     | [xshopai/chat-service#1](https://github.com/xshopai/chat-service/issues/1) |

**Services that intentionally don't consume events:**

- **order-service** - Pure publisher (saga initiator)
- **web-bff** - HTTP gateway only, no async messaging

> **Note:** All services must use `admin:admin123` credentials for local RabbitMQ (configured in `docker-compose.infrastructure.yml`). Production configurations are managed separately via infrastructure code and use managed identity.

---

## Appendix

### A. Complete Event Type Reference

```
auth.login
auth.register
auth.logout
auth.email.verification.required
auth.password.reset.requested
auth.password.reset.completed
auth.account.reactivated

user.created
user.updated
user.deleted
user.logged_in
user.logged_out
user.deactivated
user.email_verified

order.created
order.updated
order.cancelled
order.completed
order.status.changed
order.failed

payment.processing
payment.received
payment.failed
payment.refund

inventory.stock.updated
inventory.reserved
inventory.released
inventory.low.stock
inventory.out.of.stock
inventory.created

product.created
product.updated
product.deleted
product.price.changed
product.badge.assigned
product.badge.removed

review.created
review.updated
review.deleted

cart.item.added
cart.item.removed
cart.cleared
cart.abandoned

admin.action.performed
admin.user.created
admin.config.changed

notification.sent
notification.failed
```

### B. Service Technology Stack Summary

| Service                 | Language   | Framework       | Database   |
| ----------------------- | ---------- | --------------- | ---------- |
| admin-service           | JavaScript | Node.js/Express | MongoDB    |
| audit-service           | TypeScript | Node.js/Express | MongoDB    |
| auth-service            | JavaScript | Node.js/Express | MongoDB    |
| cart-service            | Java       | Spring Boot     | Redis      |
| inventory-service       | Python     | Flask           | PostgreSQL |
| notification-service    | TypeScript | Node.js/Express | MongoDB    |
| order-processor-service | Java       | Spring Boot     | PostgreSQL |
| order-service           | C#         | .NET 8          | PostgreSQL |
| payment-service         | C#         | .NET 8          | SQL Server |
| product-service         | Python     | FastAPI         | MongoDB    |
| review-service          | JavaScript | Node.js/Express | MongoDB    |
| user-service            | JavaScript | Node.js/Express | MongoDB    |
| web-bff                 | TypeScript | Node.js/Express | -          |
| chat-service            | TypeScript | Node.js/Express | -          |

---

_Document maintained by the xshopai Platform Team_
