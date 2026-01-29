# Service Architecture Document Template

## Table of Contents

1. [Overview](#1-overview)
   - 1.1 [Purpose](#11-purpose)
   - 1.2 [Scope](#12-scope)
   - 1.3 [Service Summary](#13-service-summary)
   - 1.4 [Directory Structure](#14-directory-structure)
   - 1.5 [References](#15-references)
2. [System Context](#2-system-context)
   - 2.1 [Context Diagram](#21-context-diagram)
   - 2.2 [External Interfaces](#22-external-interfaces)
   - 2.3 [Dependencies](#23-dependencies)
3. [Data Architecture](#3-data-architecture)
   - 3.1 [Database Selection](#31-database-selection)
   - 3.2 [Data Model](#32-data-model)
   - 3.3 [Data Patterns](#33-data-patterns)
   - 3.4 [Caching Strategy](#34-caching-strategy)
   - 3.5 [Data Migration Strategy](#35-data-migration-strategy)
4. [API Design](#4-api-design)
   - 4.1 [API Overview](#41-api-overview)
   - 4.2 [Endpoint Summary](#42-endpoint-summary)
   - 4.3 [Endpoints](#43-endpoints)
5. [Event Architecture](#5-event-architecture)
   - 5.1 [Event Overview](#51-event-overview)
   - 5.2 [Event Summary](#52-event-summary)
   - 5.3 [Published Events](#53-published-events)
   - 5.4 [Consumed Events](#54-consumed-events)
   - 5.5 [CloudEvents Envelope](#55-cloudevents-envelope)
   - 5.6 [Dapr Configuration](#56-dapr-configuration)
   - 5.7 [Event Processing Patterns](#57-event-processing-patterns)
   - 5.8 [Event Monitoring](#58-event-monitoring)
6. [Configuration](#6-configuration)
   - 6.1 [Environment Variables](#61-environment-variables)
   - 6.2 [Configuration Files](#62-configuration-files)
   - 6.3 [Secrets Management](#63-secrets-management)
   - 6.4 [Feature Flags](#64-feature-flags)
   - 6.5 [Configuration Validation](#65-configuration-validation)
7. [Deployment](#7-deployment)
   - 7.1 [Container Configuration](#71-container-configuration)
   - 7.2 [Resource Requirements](#72-resource-requirements)
   - 7.3 [Deployment Topology](#73-deployment-topology)
   - 7.4 [Azure Container Apps Configuration](#74-azure-container-apps-configuration)
   - 7.5 [CI/CD Pipeline](#75-cicd-pipeline)
   - 7.6 [Rollback Procedure](#76-rollback-procedure)
8. [Observability](#8-observability)
   - 8.1 [Logging Strategy](#81-logging-strategy)
   - 8.2 [Metrics Collection](#82-metrics-collection)
   - 8.3 [Distributed Tracing](#83-distributed-tracing)
   - 8.4 [Health Checks](#84-health-checks)
   - 8.5 [Alerting Rules](#85-alerting-rules)
   - 8.6 [Dashboard Requirements](#86-dashboard-requirements)
9. [Error Handling](#9-error-handling)
   - 9.1 [Error Categories](#91-error-categories)
   - 9.2 [Circuit Breaker Pattern](#92-circuit-breaker-pattern)
   - 9.3 [Retry Policies](#93-retry-policies)
   - 9.4 [Graceful Degradation](#94-graceful-degradation)
   - 9.5 [Error Logging Requirements](#95-error-logging-requirements)
   - 9.6 [Dead Letter Queue (DLQ)](#96-dead-letter-queue-dlq)
10. [Security](#10-security)
    - 10.1 [Authentication](#101-authentication)
    - 10.2 [Authorization (RBAC)](#102-authorization-rbac)
    - 10.3 [Data Protection](#103-data-protection)
    - 10.4 [Input Validation](#104-input-validation)
    - 10.5 [Rate Limiting](#105-rate-limiting)
    - 10.6 [Security Headers](#106-security-headers)
    - 10.7 [Secrets Management](#107-secrets-management)
11. [Appendix](#11-appendix)
    - A. [Glossary](#a-glossary)
    - B. [Related Documents](#b-related-documents)

---

## 1. Overview

<!-- REQUIRED: All subsections in Overview are mandatory -->

### 1.1 Purpose

<!--
Describe what this service does and why it exists.
Reference PRD section if applicable.
Example: "The Inventory Service manages stock levels, reservations, and warehouse locations for all products in the xshopai platform (PRD Section 1.1)."
-->

### 1.2 Scope

**In Scope:**

<!-- List what this service handles -->

-
- **Out of Scope:**

<!-- List what this service does NOT handle and who handles it -->

-

### 1.3 Service Summary

| Attribute          | Value                                                |
| ------------------ | ---------------------------------------------------- |
| **Service Name**   | <!-- e.g., inventory-service -->                     |
| **Tech Stack**     | <!-- e.g., Python 3.11 / FastAPI -->                 |
| **Database**       | <!-- e.g., PostgreSQL 15 -->                         |
| **Cache**          | <!-- e.g., Redis 7.x / None -->                      |
| **Messaging**      | <!-- Dapr Pub/Sub (RabbitMQ) -->                     |
| **Main Port**      | <!-- e.g., 8084 -->                                  |
| **Dapr HTTP Port** | 3500                                                 |
| **Dapr gRPC Port** | 50001                                                |
| **Pattern**        | <!-- Publisher / Consumer / Publisher & Consumer --> |

> **Note:** All services now use the standard Dapr ports (3500 for HTTP, 50001 for gRPC). This simplifies configuration and works consistently whether running via Docker Compose or individual service runs.

### 1.4 Directory Structure

<!--
REQUIRED: Provide the actual directory structure of your service.
This helps coding agents understand where to create/find files.
-->

```
service-name/
├── src/                          # Source code
│   ├── api/                      # API routes/controllers
│   │   ├── routes/               # Route definitions
│   │   └── middleware/           # Request middleware
│   ├── services/                 # Business logic layer
│   ├── repositories/             # Data access layer
│   ├── models/                   # Data models/entities
│   ├── schemas/                  # Request/response schemas
│   ├── events/                   # Event handlers and publishers
│   │   ├── handlers/             # Event consumption handlers
│   │   └── publishers/           # Event publishing logic
│   ├── core/                     # Core utilities
│   │   ├── config.py             # Configuration
│   │   ├── logger.py             # Logging setup
│   │   └── exceptions.py         # Custom exceptions
│   └── utils/                    # Helper utilities
├── tests/                        # Test files
│   ├── unit/                     # Unit tests
│   ├── integration/              # Integration tests
│   └── fixtures/                 # Test fixtures
├── dapr/                         # Dapr configuration
│   └── components/               # Dapr component YAML files
├── docs/                         # Documentation
│   ├── ARCHITECTURE.md           # This file
│   └── PRD.md                    # Product requirements
├── docker-compose.yml            # Local development
├── Dockerfile                    # Container definition
├── requirements.txt              # Dependencies (Python)
├── package.json                  # Dependencies (Node.js)
├── pom.xml                       # Dependencies (Java)
└── README.md                     # Quick start guide
```

### 1.5 References

| Document             | Link                                                                  | Description          |
| -------------------- | --------------------------------------------------------------------- | -------------------- |
| PRD                  | [docs/PRD.md](./PRD.md)                                               | Product requirements |
| API Spec             | <!-- link -->                                                         | OpenAPI/Swagger spec |
| Runbook              | <!-- link -->                                                         | Operational runbook  |
| Copilot Instructions | [.github/copilot-instructions.md](../.github/copilot-instructions.md) | AI coding guidelines |

---

## 2. System Context

<!-- REQUIRED: System context is mandatory for understanding service boundaries -->

### 2.1 Context Diagram

<!--
Show the service in context with:
- Other xshopai services it interacts with
- External systems (databases, message brokers, third-party APIs)
- UI clients that call this service
-->

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              xshopai Platform                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                             │
│  ┌──────────────┐          ┌──────────────────┐          ┌──────────────┐  │
│  │  customer-ui │────────► │     web-bff      │ ◄───────►│   admin-ui   │  │
│  └──────────────┘   REST   └────────┬─────────┘   REST   └──────────────┘  │
│                                     │                                       │
│                                     │ REST                                  │
│                                     ▼                                       │
│                           ┌──────────────────┐                             │
│                           │  [THIS SERVICE]  │                             │
│                           └────────┬─────────┘                             │
│                                    │                                        │
│            ┌───────────────────────┼───────────────────────┐               │
│            │                       │                       │               │
│            ▼                       ▼                       ▼               │
│  ┌──────────────────┐   ┌──────────────────┐   ┌──────────────────┐       │
│  │    Database      │   │   Dapr Pub/Sub   │   │  Other Services  │       │
│  │  (PostgreSQL/    │   │   (RabbitMQ)     │   │  (list them)     │       │
│  │   MongoDB)       │   │                  │   │                  │       │
│  └──────────────────┘   └──────────────────┘   └──────────────────┘       │
│                                                                             │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 2.2 External Interfaces

| System           | Direction | Protocol    | Description                  |
| ---------------- | --------- | ----------- | ---------------------------- |
| web-bff          | Inbound   | REST/HTTP   | BFF aggregates calls from UI |
| <!-- service --> | Outbound  | REST/HTTP   | <!-- description -->         |
| <!-- service --> | Pub/Sub   | Dapr Events | <!-- description -->         |
| Database         | Outbound  | TCP         | Primary data store           |

### 2.3 Dependencies

#### 2.3.1 Upstream Dependencies

<!-- Services this service CALLS (synchronous dependencies) -->

| Service          | Purpose             | Failure Impact                |
| ---------------- | ------------------- | ----------------------------- |
| <!-- service --> | <!-- why needed --> | <!-- what happens if down --> |

#### 2.3.2 Downstream Consumers

<!-- Services that CALL this service -->

| Consumer         | Endpoints Used | SLA Requirement        |
| ---------------- | -------------- | ---------------------- |
| web-bff          | <!-- list -->  | <!-- response time --> |
| <!-- service --> | <!-- list -->  | <!-- response time --> |

#### 2.3.3 Event Dependencies

<!-- Events this service publishes or consumes -->

| Direction | Event Type     | Partner Service    |
| --------- | -------------- | ------------------ |
| Publishes | <!-- event --> | <!-- consumers --> |
| Consumes  | <!-- event --> | <!-- producer -->  |

---

## 3. Data Architecture

### 3.1 Database Selection

| Aspect            | Decision                                    |
| ----------------- | ------------------------------------------- |
| **Database**      | <!-- PostgreSQL / MongoDB / Redis -->       |
| **Justification** | <!-- Why this database for this service --> |
| **Version**       | <!-- Specific version -->                   |
| **Driver**        | <!-- Client library used -->                |

### 3.2 Data Model

#### 3.2.1 Entity Relationship Diagram

```
<!--
ERD using ASCII art or Mermaid.
Show tables/collections, relationships, cardinality.
Example:
┌─────────────────┐       ┌─────────────────┐
│   inventory     │       │    warehouse    │
├─────────────────┤       ├─────────────────┤
│ id (PK)         │───────│ id (PK)         │
│ product_id      │       │ name            │
│ warehouse_id(FK)│◄──────│ location        │
│ quantity        │       │ is_active       │
└─────────────────┘       └─────────────────┘
-->
```

#### 3.2.2 Schema Definitions

<!--
For each entity, document:
- Table/Collection name
- Fields with types
- Constraints (PK, FK, UNIQUE, NOT NULL)
- Indexes
-->

**Entity: <!-- EntityName -->**

| Field          | Type                   | Constraints          | Description           |
| -------------- | ---------------------- | -------------------- | --------------------- |
| id             | <!-- UUID/ObjectId --> | PK                   | Primary identifier    |
| <!-- field --> | <!-- type -->          | <!-- constraints --> | <!-- description -->  |
| created_at     | timestamp              | NOT NULL             | Creation timestamp    |
| updated_at     | timestamp              | NOT NULL             | Last update timestamp |

**Indexes:**

```sql
-- Performance indexes
CREATE INDEX idx_<!-- table -->_<!-- field --> ON <!-- table -->(<!-- field -->);

-- Unique constraints
CREATE UNIQUE INDEX idx_<!-- table -->_unique ON <!-- table -->(<!-- fields -->);
```

### 3.3 Data Patterns

#### 3.3.1 Soft Delete Pattern

<!-- If using soft delete, document the pattern -->

```python
# Soft delete fields (add to all deletable entities)
is_deleted: boolean (default: false)
deleted_at: timestamp (nullable)
deleted_by: string (nullable)
```

**Query Pattern:** All queries MUST include `WHERE is_deleted = false` unless explicitly retrieving deleted records.

#### 3.3.2 Audit Trail Pattern

<!-- If tracking changes, document the approach -->

```python
# Audit fields (add to all entities)
created_at: timestamp
created_by: string
updated_at: timestamp
updated_by: string
```

#### 3.3.3 Optimistic Locking Pattern

<!-- If using version-based concurrency control -->

```python
# Version field for optimistic locking
version: integer (default: 1)
# Increment on each update, reject if version mismatch
```

### 3.4 Caching Strategy

| Cache Layer      | Technology                 | TTL                | Invalidation                |
| ---------------- | -------------------------- | ------------------ | --------------------------- |
| L1 (In-Memory)   | <!-- e.g., Python dict --> | <!-- e.g., 60s --> | <!-- e.g., TTL expiry -->   |
| L2 (Distributed) | <!-- e.g., Redis -->       | <!-- e.g., 5m -->  | <!-- e.g., Event-driven --> |

**Cache Key Pattern:**

```
{service}:{entity}:{id}
{service}:{query}:{hash}
```

### 3.5 Data Migration Strategy

| Aspect                | Value                                                |
| --------------------- | ---------------------------------------------------- |
| **Tool**              | <!-- Alembic / Flyway / EF Migrations / Mongoose --> |
| **Location**          | `<!-- path to migrations folder -->`                 |
| **Naming Convention** | `<!-- e.g., V001__description.sql -->`               |

**Migration Commands:**

```bash
# Run migrations
<!-- command -->

# Create new migration
<!-- command -->

# Rollback
<!-- command -->
```

---

## 4. API Design

### 4.1 API Overview

| Attribute          | Value                       |
| ------------------ | --------------------------- |
| **Base Path**      | `/api/v1/<!-- resource -->` |
| **Protocol**       | REST over HTTP/1.1          |
| **Content Type**   | `application/json`          |
| **Authentication** | JWT Bearer Token            |
| **Versioning**     | URL path (`/api/v1/`)       |

### 4.2 Endpoint Summary

| Method | Endpoint                         | Description                 | Auth Required |
| ------ | -------------------------------- | --------------------------- | ------------- |
| POST   | `/api/v1/<!-- resource -->`      | Create <!-- resource -->    | JWT           |
| GET    | `/api/v1/<!-- resource -->/{id}` | Get <!-- resource --> by ID | Optional      |
| GET    | `/api/v1/<!-- resource -->`      | List <!-- resources -->     | Optional      |
| PUT    | `/api/v1/<!-- resource -->/{id}` | Update <!-- resource -->    | JWT           |
| DELETE | `/api/v1/<!-- resource -->/{id}` | Delete <!-- resource -->    | Admin         |

**Auth Legend:** `JWT` = Valid JWT token required | `Admin` = Admin role required | `Service` = Service-to-service auth | `Optional` = Works with or without auth | `None` = Public endpoint

---

### 4.3 Endpoints

<!-- Detailed endpoint specifications -->

#### 4.3.1 Create <!-- Resource -->

Creates a new <!-- resource --> in the system.

**Request**

```
POST /api/v1/<!-- resource -->
```

**Headers**

| Header          | Required | Description                 | Example                  |
| --------------- | -------- | --------------------------- | ------------------------ |
| `Authorization` | Yes      | JWT Bearer token            | `Bearer eyJhbGciOiJI...` |
| `Content-Type`  | Yes      | Request body format         | `application/json`       |
| `X-Request-ID`  | No       | Client-generated request ID | `req-123-456-789`        |
| `X-Trace-ID`    | No       | Distributed tracing ID      | `trace-abc-def`          |

**Request Body**

```json
{
  "<!-- field1 -->": "<!-- value1 -->",
  "<!-- field2 -->": <!-- value2 -->,
  "<!-- field3 -->": "<!-- value3 -->"
}
```

**Request Body Fields**

| Field             | Type    | Required | Validation                     | Description                 |
| ----------------- | ------- | -------- | ------------------------------ | --------------------------- |
| `<!-- field1 -->` | string  | Yes      | 1-100 characters               | <!-- field1 description --> |
| `<!-- field2 -->` | integer | Yes      | > 0                            | <!-- field2 description --> |
| `<!-- field3 -->` | string  | No       | Valid enum: `value1`, `value2` | <!-- field3 description --> |

**Success Response**

```
HTTP/1.1 201 Created
Location: /api/v1/<!-- resource -->/<!-- id -->
```

```json
{
  "success": true,
  "data": {
    "id": "<!-- generated-id -->",
    "<!-- field1 -->": "<!-- value1 -->",
    "<!-- field2 -->": <!-- value2 -->,
    "<!-- field3 -->": "<!-- value3 -->",
    "createdAt": "2025-01-15T10:30:00Z",
    "updatedAt": "2025-01-15T10:30:00Z"
  },
  "message": "<!-- Resource --> created successfully"
}
```

**Error Responses**

| Status | Code                 | Condition                            | Example Message                          |
| ------ | -------------------- | ------------------------------------ | ---------------------------------------- |
| 400    | `VALIDATION_ERROR`   | Invalid or missing required fields   | "<!-- field1 --> is required"            |
| 400    | `INVALID_FORMAT`     | Field format validation failed       | "<!-- field2 --> must be greater than 0" |
| 401    | `UNAUTHORIZED`       | Missing or invalid JWT token         | "Authentication required"                |
| 403    | `FORBIDDEN`          | User lacks permission                | "Insufficient permissions"               |
| 409    | `DUPLICATE_RESOURCE` | Resource with same identifier exists | "<!-- Resource --> already exists"       |
| 500    | `INTERNAL_ERROR`     | Unexpected server error              | "Internal server error"                  |

---

#### 4.3.2 Get <!-- Resource --> by ID

Retrieves a single <!-- resource --> by its unique identifier.

**Request**

```
GET /api/v1/<!-- resource -->/{id}
```

**Path Parameters**

| Parameter | Type   | Required | Description                         | Example           |
| --------- | ------ | -------- | ----------------------------------- | ----------------- |
| `id`      | string | Yes      | Unique <!-- resource --> identifier | `res-123-456-789` |

**Headers**

| Header          | Required | Description                 | Example                  |
| --------------- | -------- | --------------------------- | ------------------------ |
| `Authorization` | No       | JWT Bearer token (optional) | `Bearer eyJhbGciOiJI...` |
| `X-Request-ID`  | No       | Client-generated request ID | `req-123-456-789`        |

**Success Response**

```
HTTP/1.1 200 OK
```

```json
{
  "success": true,
  "data": {
    "id": "<!-- id -->",
    "<!-- field1 -->": "<!-- value1 -->",
    "<!-- field2 -->": <!-- value2 -->,
    "<!-- field3 -->": "<!-- value3 -->",
    "createdAt": "2025-01-15T10:30:00Z",
    "updatedAt": "2025-01-15T10:30:00Z"
  }
}
```

**Error Responses**

| Status | Code             | Condition                        | Example Message                       |
| ------ | ---------------- | -------------------------------- | ------------------------------------- |
| 400    | `INVALID_ID`     | ID format is invalid             | "Invalid <!-- resource --> ID format" |
| 404    | `NOT_FOUND`      | <!-- Resource --> does not exist | "<!-- Resource --> not found"         |
| 500    | `INTERNAL_ERROR` | Unexpected server error          | "Internal server error"               |

---

#### 4.3.3 List <!-- Resources --> (Paginated)

Retrieves a paginated list of <!-- resources --> with optional filtering and sorting.

**Request**

```
GET /api/v1/<!-- resource -->?page=1&limit=20&sort=-createdAt&status=active
```

**Query Parameters**

| Parameter | Type    | Required | Default      | Validation  | Description                      |
| --------- | ------- | -------- | ------------ | ----------- | -------------------------------- |
| `page`    | integer | No       | 1            | >= 1        | Page number                      |
| `limit`   | integer | No       | 20           | 1-100       | Items per page                   |
| `sort`    | string  | No       | `-createdAt` | Valid field | Sort field (prefix `-` for desc) |
| `status`  | string  | No       | -            | Valid enum  | Filter by status                 |
| `search`  | string  | No       | -            | 1-100 chars | Search term for text fields      |

**Headers**

| Header          | Required | Description                 | Example                  |
| --------------- | -------- | --------------------------- | ------------------------ |
| `Authorization` | No       | JWT Bearer token (optional) | `Bearer eyJhbGciOiJI...` |
| `X-Request-ID`  | No       | Client-generated request ID | `req-123-456-789`        |

**Success Response**

```
HTTP/1.1 200 OK
```

```json
{
  "success": true,
  "data": [
    {
      "id": "<!-- id-1 -->",
      "<!-- field1 -->": "<!-- value1 -->",
      "<!-- field2 -->": <!-- value2 -->,
      "createdAt": "2025-01-15T10:30:00Z"
    },
    {
      "id": "<!-- id-2 -->",
      "<!-- field1 -->": "<!-- value1 -->",
      "<!-- field2 -->": <!-- value2 -->,
      "createdAt": "2025-01-14T09:15:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalItems": 150,
    "totalPages": 8,
    "hasNextPage": true,
    "hasPrevPage": false
  }
}
```

**Error Responses**

| Status | Code                 | Condition                         | Example Message                    |
| ------ | -------------------- | --------------------------------- | ---------------------------------- |
| 400    | `INVALID_PARAMETER`  | Query parameter validation failed | "limit must be between 1 and 100"  |
| 400    | `INVALID_SORT_FIELD` | Sort field does not exist         | "Invalid sort field: unknownField" |
| 500    | `INTERNAL_ERROR`     | Unexpected server error           | "Internal server error"            |

<!-- Add more endpoint specifications as needed following the same pattern -->

---

## 5. Event Architecture

<!--
CRITICAL: This section documents ALL events published and consumed by this service.
Coding agents use this for:
- Implementing event publishers/consumers
- Understanding async communication patterns
- Debugging message flows
-->

### 5.1 Event Overview

| Attribute              | Value                             |
| ---------------------- | --------------------------------- |
| **Message Broker**     | Dapr Pub/Sub with RabbitMQ        |
| **Event Format**       | CloudEvents 1.0                   |
| **Serialization**      | JSON                              |
| **Delivery Guarantee** | At-least-once                     |
| **Ordering**           | Per-partition (by correlation ID) |

```
<!-- Event flow diagram showing:
     - This service's position in event topology
     - Published event topics
     - Consumed event topics
     - Connected services -->
```

### 5.2 Event Summary

#### 5.2.1 Published Events

| Event Type                         | Topic                 | Trigger                 | Consumers                           |
| ---------------------------------- | --------------------- | ----------------------- | ----------------------------------- |
| `<!-- service -->.<!-- action -->` | `<!-- topic-name -->` | <!-- When triggered --> | <!-- List of consuming services --> |

#### 5.2.2 Consumed Events

| Event Type                        | Source                  | Handler                 | Idempotency Key      |
| --------------------------------- | ----------------------- | ----------------------- | -------------------- |
| `<!-- source -->.<!-- action -->` | <!-- source-service --> | `<!-- handler-path -->` | `<!-- key-field -->` |

---

### 5.3 Published Events

<!--
Document ALL events this service publishes.
Include trigger conditions and full payload schemas.
-->

#### 5.3.1 <!-- Event Name --> Event

**Trigger:** <!-- When this event is published -->

**Payload Schema:**

```json
{
  "specversion": "1.0",
  "type": "<!-- service -->.<!-- action -->",
  "source": "<!-- service-name -->",
  "id": "evt-uuid-here",
  "time": "2025-01-15T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    // Event-specific payload
  },
  "correlationid": "req-uuid-here",
  "traceparent": "00-trace-id-span-id-01"
}
```

### 5.4 Consumed Events

<!--
Document ALL events this service consumes.
Include handler location and processing logic.
-->

#### 5.4.1 <!-- Event Name --> Handler

**Source Service:** <!-- Which service publishes this -->

**Handler Location:** `src/events/handlers/<!-- handler -->.js` (or `.py`, `.java`, `.cs`)

**Processing Logic:**

1. <!-- Step 1 -->
2. <!-- Step 2 -->
3. <!-- Step 3 -->

**Idempotency Strategy:** <!-- How duplicates are handled -->

### 5.5 CloudEvents Envelope

<!--
CRITICAL: All events MUST follow CloudEvents 1.0 specification.
This envelope structure is used by ALL xshopai services.
-->

```json
{
  "specversion": "1.0",
  "type": "<!-- service -->.<!-- entity -->.<!-- action -->",
  "source": "<!-- service-name -->",
  "id": "evt-550e8400-e29b-41d4-a716-446655440000",
  "time": "2025-01-15T10:30:00.000Z",
  "datacontenttype": "application/json",
  "subject": "<!-- optional: entity-id -->",
  "data": {
    // Event-specific payload
  },
  "correlationid": "req-550e8400-e29b-41d4-a716-446655440001",
  "traceparent": "00-4bf92f3577b34da6a3ce929d0e0e4736-00f067aa0ba902b7-01",
  "causationid": "evt-previous-event-id"
}
```

**Event Type Naming Convention:** `{service}.{entity}.{action}`

- Examples: `user.created`, `order.completed`, `inventory.reserved`

### 5.6 Dapr Configuration

#### 5.6.1 Pub/Sub Component

**File:** `dapr/components/pubsub.yaml`

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <!-- service -->-pubsub
  namespace: xshopai
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
    - name: connectionString
      secretKeyRef:
        name: rabbitmq-connection
        key: connectionString
    - name: durable
      value: 'true'
    - name: deletedWhenUnused
      value: 'false'
    - name: autoAck
      value: 'false'
    - name: deliveryMode
      value: '2' # Persistent
    - name: prefetchCount
      value: '10'
```

#### 5.6.2 Subscription Configuration

**File:** `dapr/components/subscription.yaml`

```yaml
apiVersion: dapr.io/v2alpha1
kind: Subscription
metadata:
  name: <!-- service -->-subscriptions
spec:
  topic: <!-- topic-name -->
  routes:
    default: /events/<!-- handler-path -->
  pubsubname: <!-- service -->-pubsub
  deadLetterTopic: <!-- topic-name -->-dlq
```

### 5.7 Event Processing Patterns

#### 5.7.1 Publisher Pattern

```
<!-- Language-specific example based on service tech stack -->
```

#### 5.7.2 Consumer Pattern

```
<!-- Language-specific example based on service tech stack -->
```

### 5.8 Event Monitoring

| Metric                              | Type      | Description                      |
| ----------------------------------- | --------- | -------------------------------- |
| `events_published_total`            | Counter   | Total events published by type   |
| `events_consumed_total`             | Counter   | Total events consumed by type    |
| `event_processing_duration_seconds` | Histogram | Event processing time            |
| `event_processing_errors_total`     | Counter   | Failed event processing attempts |
| `dlq_depth`                         | Gauge     | Dead letter queue message count  |

---

## 6. Configuration

<!--
Split from Infrastructure to focus on service configuration.
Environment variables, feature flags, secrets management.
-->

### 6.1 Environment Variables

<!--
CRITICAL: Document ALL environment variables.
Coding agents need this for deployment and local development.
-->

#### 6.1.1 Required Variables

| Variable                           | Description                | Example         | Validation              |
| ---------------------------------- | -------------------------- | --------------- | ----------------------- |
| `PORT`                             | Service port               | `8080`          | Integer, 1-65535        |
| `NODE_ENV` / `ENVIRONMENT`         | Environment name           | `production`    | development, production |
| `DATABASE_URL`                     | Database connection string | `mongodb://...` | Valid URI               |
| `JWT_SECRET`                       | JWT signing key            | (from secret)   | Min 32 chars            |
| <!-- Add service-specific vars --> |                            |                 |                         |

#### 6.1.2 Optional Variables

| Variable                           | Description          | Default | Notes                    |
| ---------------------------------- | -------------------- | ------- | ------------------------ |
| `LOG_LEVEL`                        | Logging verbosity    | `info`  | debug, info, warn, error |
| `CORS_ORIGINS`                     | Allowed CORS origins | `*`     | Comma-separated URLs     |
| `REQUEST_TIMEOUT_MS`               | Request timeout      | `30000` | Milliseconds             |
| <!-- Add service-specific vars --> |                      |         |                          |

### 6.2 Configuration Files

| File                     | Purpose                | Environment |
| ------------------------ | ---------------------- | ----------- |
| `.env.example`           | Template for local dev | Local       |
| `config/default.json`    | Default configuration  | All         |
| `config/production.json` | Production overrides   | Production  |
| `dapr/components/*.yaml` | Dapr components        | All         |

### 6.3 Secrets Management

<!--
How secrets are stored, accessed, and rotated.
-->

| Secret                                | Storage         | Rotation | Access Method     |
| ------------------------------------- | --------------- | -------- | ----------------- |
| `DATABASE_PASSWORD`                   | Azure Key Vault | 90 days  | Dapr Secret Store |
| `JWT_SECRET`                          | Azure Key Vault | 180 days | Dapr Secret Store |
| `RABBITMQ_PASSWORD`                   | Azure Key Vault | 90 days  | Dapr Secret Store |
| <!-- Add service-specific secrets --> |                 |          |                   |

**Dapr Secret Store Configuration:**

```yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: secretstore
spec:
  type: secretstores.azure.keyvault
  version: v1
  metadata:
    - name: vaultName
      value: 'xshopai-secrets'
    - name: azureClientId
      value: '<!-- managed-identity-client-id -->'
```

### 6.4 Feature Flags

| Flag          | Description          | Default             | Scope                       |
| ------------- | -------------------- | ------------------- | --------------------------- |
| <!-- flag --> | <!-- description --> | <!-- true/false --> | <!-- user/tenant/global --> |

### 6.5 Configuration Validation

<!--
How configuration is validated at startup.
-->

- **Startup Validation:** All required variables checked before service starts
- **Type Checking:** Variables validated against expected types
- **Failure Behavior:** Service fails fast with clear error message if validation fails

---

## 7. Deployment

<!--
Container configuration, CI/CD, environment topology.
-->

### 7.1 Container Configuration

**Dockerfile:**

```dockerfile
# Example structure - adjust for actual service
FROM <!-- base-image -->

WORKDIR /app
COPY <!-- files -->
RUN <!-- build commands -->

EXPOSE <!-- port -->
CMD ["<!-- entrypoint -->"]
```

| Setting          | Value                                           |
| ---------------- | ----------------------------------------------- |
| **Base Image**   | <!-- e.g., node:18-alpine, python:3.11-slim --> |
| **Exposed Port** | <!-- e.g., 8080 -->                             |
| **Health Check** | `<!-- health check command -->`                 |
| **User**         | `<!-- non-root user -->`                        |

### 7.2 Resource Requirements

| Environment | CPU Request | CPU Limit | Memory Request | Memory Limit | Replicas |
| ----------- | ----------- | --------- | -------------- | ------------ | -------- |
| Development | 100m        | 500m      | 128Mi          | 512Mi        | 1        |
| Production  | 500m        | 2000m     | 512Mi          | 2Gi          | 3-10     |

### 7.3 Deployment Topology

```
<!-- ASCII diagram showing:
     - Load balancer
     - Service replicas
     - Database connections
     - Message broker connections
     - External service connections -->
```

### 7.4 Azure Container Apps Configuration

```yaml
# container-app.yaml
properties:
  configuration:
    activeRevisionsMode: Multiple
    ingress:
      external: true
      targetPort: <!-- port -->
      traffic:
        - latestRevision: true
          weight: 100
    dapr:
      enabled: true
      appId: <!-- service-name -->
      appPort: <!-- port -->
      appProtocol: http
  template:
    containers:
      - name: <!-- service-name -->
        image: <!-- image -->
        resources:
          cpu: <!-- cpu -->
          memory: <!-- memory -->
        env:
          - name: <!-- var -->
            value: <!-- value -->
    scale:
      minReplicas: <!-- min -->
      maxReplicas: <!-- max -->
      rules:
        - name: http-scaling
          http:
            metadata:
              concurrentRequests: '100'
```

### 7.5 CI/CD Pipeline

| Stage          | Trigger         | Actions                   | Duration |
| -------------- | --------------- | ------------------------- | -------- |
| Build          | Push to main/PR | Lint, Test, Build image   | ~5 min   |
| Security Scan  | After Build     | Container scan, SAST      | ~3 min   |
| Deploy to Dev  | Merge to main   | Deploy to dev environment | ~2 min   |
| Deploy to Prod | Manual approval | Blue-green deployment     | ~5 min   |

### 7.6 Rollback Procedure

1. Identify failed deployment via monitoring alerts
2. Execute rollback command: `az containerapp revision activate --name <!-- service --> --revision <!-- previous-revision -->`
3. Verify health checks pass
4. Investigate root cause
5. Document in incident report

---

## 8. Observability

Comprehensive monitoring and observability following the three pillars: logs, metrics, and traces.

### 8.1 Logging Strategy

#### 8.1.1 Log Levels

| Level   | Usage                                              | Example                                           |
| ------- | -------------------------------------------------- | ------------------------------------------------- |
| `ERROR` | System errors requiring immediate attention        | Database connection failure, unhandled exceptions |
| `WARN`  | Unexpected conditions that don't prevent operation | Deprecated API usage, retry attempts              |
| `INFO`  | Significant business events                        | Request received, order created, user logged in   |
| `DEBUG` | Detailed diagnostic information                    | Method entry/exit, variable values                |
| `TRACE` | Fine-grained debugging (disabled in production)    | Full request/response payloads                    |

#### 8.1.2 Structured Log Format

All services MUST use structured JSON logging:

```json
{
  "timestamp": "2025-01-15T10:30:45.123Z",
  "level": "INFO",
  "service": "<!-- service-name -->",
  "version": "<!-- service-version -->",
  "environment": "production",
  "correlationId": "<!-- uuid -->",
  "traceId": "<!-- opentelemetry-trace-id -->",
  "spanId": "<!-- opentelemetry-span-id -->",
  "userId": "<!-- authenticated-user-id -->",
  "operation": "<!-- operation-name -->",
  "message": "<!-- human-readable-message -->",
  "duration": 45,
  "metadata": {
    "<!-- context-specific-fields -->"
  },
  "error": {
    "code": "<!-- error-code -->",
    "message": "<!-- error-message -->",
    "stack": "<!-- stack-trace-if-error -->"
  }
}
```

#### 8.1.3 Environment-Specific Logging

| Environment | Level                        | Stack Traces                | Request Bodies | Retention |
| ----------- | ---------------------------- | --------------------------- | -------------- | --------- |
| Development | DEBUG                        | Full                        | Yes            | 7 days    |
| Production  | INFO (WARN for high-traffic) | Error tracking service only | No             | 90 days   |

### 8.2 Metrics Collection

#### 8.2.1 Required Metrics

| Metric Name                     | Type      | Description                 | Labels                      |
| ------------------------------- | --------- | --------------------------- | --------------------------- |
| `http_requests_total`           | Counter   | Total HTTP requests         | method, path, status        |
| `http_request_duration_seconds` | Histogram | Request latency             | method, path                |
| `http_requests_in_flight`       | Gauge     | Current active requests     | -                           |
| `db_query_duration_seconds`     | Histogram | Database query latency      | operation, collection/table |
| `db_connections_active`         | Gauge     | Active database connections | -                           |
| `events_published_total`        | Counter   | Events published            | event_type, status          |
| `events_consumed_total`         | Counter   | Events consumed             | event_type, status          |
| `cache_hits_total`              | Counter   | Cache hit count             | cache_name                  |
| `cache_misses_total`            | Counter   | Cache miss count            | cache_name                  |

#### 8.2.2 Metrics Endpoint

```yaml
# Prometheus scrape configuration
endpoint: /metrics
port: <!-- metrics-port -->
interval: 15s
```

### 8.3 Distributed Tracing

#### 8.3.1 OpenTelemetry Configuration

```yaml
# OpenTelemetry SDK configuration
OTEL_SERVICE_NAME: '<!-- service-name -->'
OTEL_EXPORTER_OTLP_ENDPOINT: 'http://otel-collector:4318'
OTEL_TRACES_SAMPLER: 'parentbased_traceidratio'
OTEL_TRACES_SAMPLER_ARG: '0.1' # 10% sampling in production
```

#### 8.3.2 Span Attributes

All spans MUST include:

| Attribute                | Description            |
| ------------------------ | ---------------------- |
| `service.name`           | Service identifier     |
| `service.version`        | Service version        |
| `deployment.environment` | Environment (dev/prod) |
| `http.method`            | HTTP method            |
| `http.url`               | Full URL (sanitized)   |
| `http.status_code`       | Response status code   |
| `db.system`              | Database type          |
| `db.operation`           | Database operation     |
| `messaging.system`       | Message broker type    |
| `messaging.operation`    | publish/consume        |

### 8.4 Health Checks

#### 8.4.1 Liveness Probe

```http
GET /health/live
```

**Response (200 OK):**

```json
{
  "status": "healthy",
  "service": "<!-- service-name -->",
  "timestamp": "2025-01-15T10:30:45.123Z"
}
```

**Purpose:** Indicates if the service process is running. Kubernetes restarts the pod if this fails.

#### 8.4.2 Readiness Probe

```http
GET /health/ready
```

**Response (200 OK):**

```json
{
  "status": "ready",
  "service": "<!-- service-name -->",
  "timestamp": "2025-01-15T10:30:45.123Z",
  "dependencies": {
    "database": {
      "status": "connected",
      "responseTime": "5ms"
    },
    "messagebroker": {
      "status": "connected",
      "responseTime": "3ms"
    },
    "cache": {
      "status": "connected",
      "responseTime": "1ms"
    }
  }
}
```

**Response (503 Service Unavailable):**

```json
{
  "status": "not-ready",
  "service": "<!-- service-name -->",
  "timestamp": "2025-01-15T10:30:45.123Z",
  "dependencies": {
    "database": {
      "status": "disconnected",
      "error": "Connection timeout after 5000ms"
    }
  },
  "reason": "Database connection unavailable"
}
```

**Purpose:** Indicates if the service can handle traffic. Kubernetes removes from load balancer if this fails.

### 8.5 Alerting Rules

| Alert                       | Condition                         | Severity | Action                                     |
| --------------------------- | --------------------------------- | -------- | ------------------------------------------ |
| HighErrorRate               | Error rate > 5% for 5 minutes     | Critical | Page on-call, investigate immediately      |
| HighLatency                 | P95 latency > 500ms for 5 minutes | Warning  | Investigate performance degradation        |
| ServiceDown                 | Health check fails for 2 minutes  | Critical | Auto-restart, page if persists             |
| DatabaseConnectionExhausted | Connection pool > 90%             | Warning  | Scale database or optimize queries         |
| EventProcessingLag          | Consumer lag > 1000 messages      | Warning  | Scale consumers or investigate bottleneck  |
| MemoryPressure              | Memory usage > 85%                | Warning  | Investigate memory leaks, consider scaling |
| DiskSpaceLow                | Disk usage > 80%                  | Warning  | Clean up logs, expand storage              |

### 8.6 Dashboard Requirements

Each service MUST have a Grafana dashboard displaying:

1. **Overview Panel:** Request rate, error rate, latency percentiles
2. **Dependencies Panel:** Database, cache, message broker health
3. **Business Metrics Panel:** Service-specific KPIs
4. **Resource Usage Panel:** CPU, memory, network I/O
5. **Events Panel:** Published/consumed events, processing lag

---

## 9. Error Handling

Comprehensive error handling strategy ensuring consistent behavior across all services.

### 9.1 Error Categories

| Category              | HTTP Range | Handling Strategy                   | User Impact                   |
| --------------------- | ---------- | ----------------------------------- | ----------------------------- |
| Validation Errors     | 400-422    | Return immediately with details     | Clear feedback, fix and retry |
| Authentication Errors | 401        | Clear auth state, redirect to login | Re-authenticate               |
| Authorization Errors  | 403        | Log, do not retry                   | Contact admin for permissions |
| Resource Not Found    | 404        | Return gracefully                   | Redirect or show not found    |
| Conflict Errors       | 409        | Return conflict details             | Resolve conflict, retry       |
| Rate Limit Errors     | 429        | Return retry-after header           | Wait and retry                |
| Server Errors         | 500-503    | Log, retry with backoff             | Show error, auto-retry        |
| Timeout Errors        | 504        | Retry with backoff                  | Show loading, auto-retry      |

### 9.2 Circuit Breaker Pattern

Implement circuit breakers for all external dependencies:

```yaml
# Circuit breaker configuration
circuitBreaker:
  <!-- dependency-name -->:
    failureThreshold: 5 # Failures before opening
    successThreshold: 3 # Successes before closing
    timeout: 30s # Time before half-open
    requestVolumeThreshold: 10 # Minimum requests before evaluation
```

**States:**

- **Closed:** Normal operation, failures counted
- **Open:** All requests fail fast, return cached/fallback response
- **Half-Open:** Limited requests allowed to test recovery

### 9.3 Retry Policies

| Operation Type | Max Retries | Initial Delay | Max Delay | Backoff                 |
| -------------- | ----------- | ------------- | --------- | ----------------------- |
| Database read  | 3           | 100ms         | 1s        | Exponential             |
| Database write | 2           | 200ms         | 2s        | Exponential             |
| External API   | 3           | 500ms         | 5s        | Exponential with jitter |
| Event publish  | 3           | 1s            | 10s       | Exponential             |
| Event consume  | 5           | 1s            | 30s       | Exponential             |

**Retry Formula:**

```
delay = min(initialDelay * (2 ^ attempt) + jitter, maxDelay)
```

### 9.4 Graceful Degradation

| Dependency       | Degradation Strategy       | Fallback Behavior              |
| ---------------- | -------------------------- | ------------------------------ |
| Database (read)  | Return cached data         | Serve stale data with warning  |
| Database (write) | Queue for retry            | Return accepted, process async |
| Cache            | Bypass cache               | Query database directly        |
| External API     | Return default/cached      | Use fallback data              |
| Message broker   | Write to dead letter queue | Log and alert                  |

### 9.5 Error Logging Requirements

All errors MUST be logged with:

```json
{
  "level": "ERROR",
  "correlationId": "<!-- correlation-id -->",
  "error": {
    "code": "<!-- error-code -->",
    "category": "<!-- validation/authentication/server/etc -->",
    "message": "<!-- error-message -->",
    "stack": "<!-- stack-trace -->",
    "context": {
      "operation": "<!-- what-was-being-done -->",
      "input": "<!-- sanitized-input -->",
      "userId": "<!-- user-id-if-available -->"
    }
  },
  "recovery": {
    "attempted": true,
    "strategy": "<!-- retry/circuit-breaker/fallback -->",
    "outcome": "<!-- success/failed -->"
  }
}
```

### 9.6 Dead Letter Queue (DLQ)

Failed events MUST be routed to DLQ:

```yaml
# Dapr DLQ configuration
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: <!-- service -->-dlq
spec:
  type: pubsub.rabbitmq
  metadata:
    - name: host
      value: 'amqp://rabbitmq:5672'
    - name: durable
      value: 'true'
    - name: deletedWhenUnused
      value: 'false'
```

**DLQ Processing Requirements:**

1. Monitor DLQ depth via metrics
2. Alert if DLQ grows beyond threshold
3. Provide admin UI for DLQ inspection
4. Support manual replay of failed events
5. Retain DLQ messages for 7 days minimum

---

## 10. Security

Security controls and practices implemented in the service.

### 10.1 Authentication

#### 10.1.1 JWT Validation

```yaml
# JWT configuration
jwt:
  issuer: '<!-- issuer-url -->'
  audience: '<!-- expected-audience -->'
  algorithm: 'RS256'
  publicKeyUrl: '<!-- jwks-url -->'
  clockSkew: 30s # Allow for clock drift
```

**Required JWT Claims:**
| Claim | Description | Validation |
|-------|-------------|------------|
| `sub` | Subject (user ID) | Required, non-empty |
| `iat` | Issued at | Required, not in future |
| `exp` | Expiration | Required, not expired |
| `aud` | Audience | Must match service audience |
| `iss` | Issuer | Must match configured issuer |
| `roles` | User roles | Optional, defaults to empty |

#### 10.1.2 Service-to-Service Authentication

```yaml
# Dapr mTLS (enabled by default)
# Services authenticate via Dapr sidecar certificates
```

### 10.2 Authorization (RBAC)

#### 10.2.1 Role Definitions

| Role      | Description                 | Typical Users            |
| --------- | --------------------------- | ------------------------ |
| `admin`   | Full system access          | Platform administrators  |
| `service` | Service-to-service calls    | Internal services        |
| `user`    | Standard authenticated user | Customers                |
| `guest`   | Limited read-only access    | Unauthenticated visitors |

#### 10.2.2 Permission Matrix

| Operation        | admin | service | user | guest |
| ---------------- | ----- | ------- | ---- | ----- |
| Read own data    | ✅    | ✅      | ✅   | ❌    |
| Read any data    | ✅    | ✅      | ❌   | ❌    |
| Create           | ✅    | ✅      | ✅   | ❌    |
| Update own       | ✅    | ✅      | ✅   | ❌    |
| Update any       | ✅    | ✅      | ❌   | ❌    |
| Delete own       | ✅    | ✅      | ✅   | ❌    |
| Delete any       | ✅    | ❌      | ❌   | ❌    |
| Admin operations | ✅    | ❌      | ❌   | ❌    |

### 10.3 Data Protection

#### 10.3.1 Data at Rest

| Data Type | Encryption          | Key Management            |
| --------- | ------------------- | ------------------------- |
| Database  | AES-256 (TDE)       | Azure Key Vault / AWS KMS |
| Secrets   | AES-256             | Dapr Secret Store         |
| Backups   | AES-256             | Managed encryption keys   |
| Logs      | Platform encryption | Managed by log aggregator |

#### 10.3.2 Data in Transit

| Channel            | Protection    | Configuration             |
| ------------------ | ------------- | ------------------------- |
| External HTTPS     | TLS 1.3       | Minimum TLS 1.2           |
| Service-to-Service | mTLS via Dapr | Auto-rotated certificates |
| Database           | TLS           | Require SSL connection    |
| Message Broker     | TLS           | Encrypted channels        |

#### 10.3.3 Sensitive Data Handling

| Data Category | Handling            | Storage                  | Logging           |
| ------------- | ------------------- | ------------------------ | ----------------- |
| Passwords     | Never stored plain  | Bcrypt hash (cost 12+)   | Never log         |
| API Keys      | Encrypted           | Secret store only        | Never log         |
| PII           | Minimize collection | Encrypted, access logged | Mask in logs      |
| Payment Data  | Not stored          | PCI-compliant provider   | Never log         |
| Tokens        | Short-lived         | Memory only              | Last 4 chars only |

### 10.4 Input Validation

#### 10.4.1 Validation Requirements

All inputs MUST be validated:

| Input Type | Validation                        | Example                           |
| ---------- | --------------------------------- | --------------------------------- |
| String     | Max length, allowed characters    | Name: 1-100 chars, alphanumeric   |
| Email      | RFC 5322 format                   | regex validation                  |
| URL        | Valid URL format, allowed schemes | https only for external           |
| Numeric    | Range, precision                  | Price: 0-999999.99                |
| Date       | ISO 8601 format, reasonable range | Not before 1900, not after 2100   |
| ID         | Format validation                 | MongoDB ObjectId, UUID v4         |
| Enum       | Allowed values only               | Status: active, inactive, deleted |

#### 10.4.2 Sanitization

```
1. Remove/escape HTML tags (prevent XSS)
2. Remove SQL injection patterns
3. Remove NoSQL injection patterns ($ operators)
4. Trim whitespace
5. Normalize Unicode
6. Reject null bytes
```

### 10.5 Rate Limiting

| Endpoint Category  | Limit         | Window   | Behavior          |
| ------------------ | ------------- | -------- | ----------------- |
| Authentication     | 5 requests    | 1 minute | 429 + lockout     |
| Public read        | 100 requests  | 1 minute | 429 + retry-after |
| Authenticated read | 1000 requests | 1 minute | 429 + retry-after |
| Write operations   | 100 requests  | 1 minute | 429 + retry-after |
| Admin operations   | 500 requests  | 1 minute | 429 + retry-after |

### 10.6 Security Headers

All responses MUST include:

```http
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
X-XSS-Protection: 1; mode=block
Content-Security-Policy: default-src 'self'
Cache-Control: no-store (for sensitive data)
```

### 10.7 Secrets Management

#### 10.7.1 Secret Sources

```yaml
# Priority order for secret resolution
1. Dapr Secret Store (Azure Key Vault / Kubernetes Secrets)
2. Environment variables (for local development only)

# Never:
- Hardcode secrets in source code
- Commit secrets to version control
- Log secret values
- Pass secrets in URLs
```

#### 10.7.2 Secret Rotation

| Secret Type            | Rotation Frequency | Automation                      |
| ---------------------- | ------------------ | ------------------------------- |
| Database credentials   | 90 days            | Automated via Key Vault         |
| API keys               | 90 days            | Manual with notification        |
| JWT signing keys       | 180 days           | Automated, support key rollover |
| Service account tokens | 30 days            | Automated via managed identity  |

---

## 11. Appendix

### A. Glossary

| Term        | Definition                                                      |
| ----------- | --------------------------------------------------------------- |
| BFF         | Backend for Frontend - API gateway pattern for specific clients |
| CloudEvents | Specification for describing event data in a common way         |
| Dapr        | Distributed Application Runtime - sidecar for microservices     |
| DLQ         | Dead Letter Queue - storage for failed messages                 |
| mTLS        | Mutual TLS - two-way certificate authentication                 |
| OTEL        | OpenTelemetry - observability framework                         |
| P95/P99     | 95th/99th percentile response time                              |
| RTO         | Recovery Time Objective - max acceptable downtime               |
| RPO         | Recovery Point Objective - max acceptable data loss             |

### B. Related Documents

| Document             | Description                   | Location                          |
| -------------------- | ----------------------------- | --------------------------------- |
| PRD                  | Product Requirements Document | `docs/PRD.md`                     |
| API Specification    | OpenAPI/Swagger documentation | `docs/API.md` or `/api/docs`      |
| Runbooks             | Operational procedures        | `docs/runbooks/`                  |
| Copilot Instructions | AI assistant guidelines       | `.github/copilot-instructions.md` |
| Infrastructure       | Deployment configurations     | `infrastructure/`                 |
