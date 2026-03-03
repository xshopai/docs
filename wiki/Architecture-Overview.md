# Architecture Overview

## Platform Overview

xshopai is a cloud-native, microservices-based e-commerce platform built with a polyglot architecture. The platform follows Domain-Driven Design (DDD) principles with event-driven communication patterns powered by Dapr.

### Design Principles

| Principle          | Description                                                      |
| :----------------- | :--------------------------------------------------------------- |
| **Microservices**  | Each service owns its domain and data store                      |
| **Event-Driven**   | Asynchronous communication via Dapr pub/sub (RabbitMQ)           |
| **API-First**      | RESTful APIs with standardized contracts                         |
| **Polyglot**       | Best technology chosen per service requirement                   |
| **Cloud-Native**   | Containerized, deployable to Azure App Service or Container Apps |
| **Observability**  | Distributed tracing (Zipkin/OpenTelemetry), structured logging   |
| **Security-First** | JWT authentication, RBAC, service-to-service tokens              |

---

## High-Level Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        Client Layer                           │
│                                                               │
│   ┌─────────────┐            ┌─────────────┐                │
│   │ customer-ui │            │  admin-ui   │                │
│   │  React :3000│            │ React :3001 │                │
│   └──────┬──────┘            └──────┬──────┘                │
└──────────┼──────────────────────────┼────────────────────────┘
           │                          │
           └──────────┬───────────────┘
                      ▼
┌──────────────────────────────────────────────────────────────┐
│                     API Gateway Layer                         │
│                                                               │
│                  ┌──────────────┐                             │
│                  │   web-bff    │                             │
│                  │   TS :8014   │                             │
│                  └──────┬───────┘                             │
└─────────────────────────┼────────────────────────────────────┘
                          │
           ┌──────────────┼──────────────────┐
           ▼              ▼                  ▼
┌──────────────────────────────────────────────────────────────┐
│                     Service Layer                             │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌─────────────────┐        │
│  │   auth     │  │   user     │  │    admin         │        │
│  │  JS :8004  │  │  JS :8002  │  │   JS :8003       │        │
│  └────────────┘  └────────────┘  └─────────────────┘        │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌─────────────────┐        │
│  │  product   │  │ inventory  │  │    review        │        │
│  │  Py :8001  │  │  Py :8005  │  │   JS :8010       │        │
│  └────────────┘  └────────────┘  └─────────────────┘        │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌─────────────────┐        │
│  │   order    │  │  payment   │  │ order-processor  │        │
│  │  C# :8006  │  │  C# :8009  │  │  Java :8007      │        │
│  └────────────┘  └────────────┘  └─────────────────┘        │
│                                                               │
│  ┌────────────┐  ┌────────────┐  ┌─────────────────┐        │
│  │   cart     │  │   chat     │  │  notification    │        │
│  │  TS :8008  │  │  TS :8013  │  │   TS :8011       │        │
│  └────────────┘  └────────────┘  └─────────────────┘        │
│                                                               │
│                  ┌────────────┐                               │
│                  │   audit    │                               │
│                  │  TS :8012  │                               │
│                  └────────────┘                               │
└──────────────────────────────────────────────────────────────┘
           │              │                  │
           ▼              ▼                  ▼
┌──────────────────────────────────────────────────────────────┐
│                  Infrastructure Layer                         │
│                                                               │
│  ┌─────────┐  ┌──────┐  ┌───────────┐  ┌────────┐          │
│  │ MongoDB │  │ PgSQL│  │ SQL Server│  │ MySQL  │          │
│  │ ×3      │  │ ×2   │  │ ×2        │  │ ×1     │          │
│  └─────────┘  └──────┘  └───────────┘  └────────┘          │
│                                                               │
│  ┌─────────┐  ┌──────────┐  ┌────────┐  ┌────────┐         │
│  │  Redis  │  │ RabbitMQ │  │ Zipkin │  │Mailpit │         │
│  └─────────┘  └──────────┘  └────────┘  └────────┘         │
└──────────────────────────────────────────────────────────────┘
```

---

## Communication Patterns

### Synchronous (HTTP)

All client requests flow through the **web-bff** gateway, which fans out to downstream services.

```
Browser → web-bff → auth-service    (login, register)
Browser → web-bff → product-service (catalog, search)
Browser → web-bff → order-service   (place order, track)
Browser → web-bff → cart-service    (add/remove items)
```

Services also call each other directly for privileged operations:

| Caller        | Target          | Purpose                            |
| :------------ | :-------------- | :--------------------------------- |
| auth-service  | user-service    | Validate credentials, create users |
| admin-service | user-service    | Administrative user management     |
| chat-service  | product-service | AI product search                  |
| chat-service  | order-service   | AI order tracking                  |

### Asynchronous (Dapr Pub/Sub)

Services publish events to Dapr pub/sub topics. Other services subscribe to topics they care about. RabbitMQ is the underlying message broker.

**Service Roles:**

| Role                | Services                                                   | Description                              |
| :------------------ | :--------------------------------------------------------- | :--------------------------------------- |
| **Pure Publishers** | auth, user, order, review, admin, cart                     | Publish events, never consume            |
| **Hybrid**          | payment, inventory, product, order-processor, notification | Both publish and consume                 |
| **Pure Consumer**   | audit                                                      | Subscribes to all events for audit trail |
| **No Pub/Sub**      | web-bff, chat                                              | HTTP-only services                       |

See the [Event Catalog](Event-Catalog.md) for the complete event reference.

---

## Saga Pattern — Order Processing

The platform uses a **choreography-based saga** for order fulfillment, coordinated by `order-processor-service`:

```
1. order-service publishes → order.created
2. order-processor-service starts saga → PENDING_PAYMENT_CONFIRMATION
3. Admin confirms payment → payment-service publishes → payment.received
4. order-processor-service → requests inventory reservation
5. inventory-service publishes → inventory.reserved
6. order-processor-service → initiates shipping
7. Saga completes → COMPLETED
```

**Compensating Transactions:** If any step fails, the saga reverses previous steps (refund payment, release inventory).

---

## Security Architecture

### Authentication Flow

```
Client → web-bff → auth-service → user-service (credential validation)
                         │
                         ├── Issues JWT access token (short-lived)
                         └── Issues JWT refresh token (long-lived)
```

### Token Structure

```json
{
  "id": "user-id",
  "email": "user@example.com",
  "roles": ["customer"],
  "iat": 1700000000,
  "exp": 1700003600
}
```

### Access Control

| Role         | Access                                        |
| :----------- | :-------------------------------------------- |
| **customer** | Customer-facing APIs via web-bff              |
| **admin**    | Admin APIs + admin-ui dashboard               |
| **service**  | Service-to-service tokens for Dapr invocation |

---

## Data Architecture

Each service owns its data store — no shared databases:

| Database          | Services                | Purpose                                    |
| :---------------- | :---------------------- | :----------------------------------------- |
| MongoDB (27018)   | user-service            | User profiles, addresses, preferences      |
| MongoDB (27019)   | product-service         | Product catalog, categories                |
| MongoDB (27020)   | review-service          | Reviews, ratings                           |
| PostgreSQL (5434) | audit-service           | Immutable audit records                    |
| PostgreSQL (5435) | order-processor-service | Saga state tracking                        |
| SQL Server (1434) | order-service           | Orders, order items, returns               |
| SQL Server (1433) | payment-service         | Payments, transactions, refunds            |
| MySQL (3306)      | inventory-service       | Stock levels, reservations                 |
| Redis (6379)      | cart-service            | Shopping cart state (via Dapr state store) |

See the [Database Guide](Database-Guide.md) for connection details and schema patterns.

---

## Deployment Architecture

The platform supports three deployment targets:

| Target                   | Description                                              | IaC                                            |
| :----------------------- | :------------------------------------------------------- | :--------------------------------------------- |
| **Local Docker**         | Docker Compose for infrastructure, services run natively | `dev/docker-compose.yml`                       |
| **Azure App Service**    | Managed PaaS with per-service web apps                   | Bicep modules in `infrastructure/app-service/` |
| **Azure Container Apps** | Serverless container platform with Dapr integration      | Bash scripts in `infrastructure/aca/`          |

See the [Deployment Guide](Deployment-Guide.md) for detailed instructions.
