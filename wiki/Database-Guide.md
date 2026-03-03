# Database Guide

xshopai uses **polyglot persistence** — each service chooses the database best suited for its data patterns. No databases are shared between services.

---

## Database Summary

| Database   | Version | Port  | Service                 | Purpose                               |
| :--------- | :------ | :---- | :---------------------- | :------------------------------------ |
| MongoDB    | 8.0     | 27018 | user-service            | User profiles, addresses, preferences |
| MongoDB    | 8.0     | 27019 | product-service         | Product catalog, categories           |
| MongoDB    | 8.0     | 27020 | review-service          | Reviews, ratings                      |
| PostgreSQL | 16      | 5434  | audit-service           | Immutable audit records               |
| PostgreSQL | 16      | 5435  | order-processor-service | Saga state tracking                   |
| SQL Server | 2022    | 1434  | order-service           | Orders, order items, returns          |
| SQL Server | 2022    | 1433  | payment-service         | Payments, transactions, refunds       |
| MySQL      | 8.0     | 3306  | inventory-service       | Stock levels, reservations            |
| Redis      | 7.0     | 6379  | cart-service            | Shopping cart state                   |

---

## Connection Details (Local Development)

### MongoDB Instances

```
# user-service
mongodb://admin:admin123@localhost:27018/user-service?authSource=admin

# product-service
mongodb://admin:admin123@localhost:27019/product-service?authSource=admin

# review-service
mongodb://admin:admin123@localhost:27020/review-service?authSource=admin
```

### PostgreSQL Instances

```
# audit-service
postgresql://postgres:postgres@localhost:5434/audit-service

# order-processor-service
postgresql://postgres:postgres@localhost:5435/order_processor_db
```

### SQL Server Instances

```
# payment-service
Server=localhost,1433;Database=PaymentServiceDb;User Id=sa;Password=Admin123!;TrustServerCertificate=True

# order-service
Server=localhost,1434;Database=OrderServiceDb;User Id=sa;Password=Admin123!;TrustServerCertificate=True
```

### MySQL

```
# inventory-service
mysql+pymysql://admin:admin123@localhost:3306/inventory_service_db
```

### Redis

```
# cart-service
Host=localhost;Port=6379;Password=redis_dev_pass_123
```

---

## Database Patterns by Technology

### MongoDB (user, product, review)

**Driver/ORM:**

- user-service, review-service: **Mongoose** ODM (Node.js)
- product-service: **Motor** async driver (Python)

**Key Patterns:**

- Document-oriented schema with embedded subdocuments (addresses, payment methods)
- Schema defined in code with built-in validation
- Mongoose timestamps enabled (`createdAt`, `updatedAt`)
- `lean()` for read-only queries (performance optimization)
- Compound indexes for common queries
- Aggregation pipelines for statistics (average ratings, search)

**Example — User Schema:**

```javascript
{
  email: String (unique, indexed),
  password: String (bcrypt hashed),
  profile: {
    firstName: String,
    lastName: String,
    phone: String
  },
  addresses: [AddressSubdocument],
  roles: [String],
  createdAt: Date,
  updatedAt: Date
}
```

**Example — Product Schema:**

```python
{
  "name": str,
  "description": str,
  "price": float,
  "category": str,
  "images": [str],
  "review_count": int,
  "average_rating": float,
  "created_at": datetime,
  "updated_at": datetime
}
```

---

### PostgreSQL (audit, order-processor)

**Driver/ORM:**

- audit-service: **Knex.js** query builder (TypeScript)
- order-processor-service: **Spring Data JPA** + **Flyway** migrations (Java)

**Key Patterns:**

- Relational schema with indexed columns
- JSONB columns for flexible event/order data storage
- Append-only pattern in audit-service (no UPDATE/DELETE)
- Flyway versioned migrations for order-processor

**Example — Audit Table:**

```sql
CREATE TABLE audit_records (
  id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_type    VARCHAR(255) NOT NULL,
  source        VARCHAR(255) NOT NULL,
  subject       VARCHAR(255),
  data          JSONB NOT NULL,
  correlation_id VARCHAR(255),
  created_at    TIMESTAMPTZ DEFAULT NOW()
);

CREATE INDEX idx_audit_event_type ON audit_records(event_type);
CREATE INDEX idx_audit_source ON audit_records(source);
CREATE INDEX idx_audit_created_at ON audit_records(created_at);
```

**Example — Saga Table:**

```sql
CREATE TABLE order_processing_sagas (
  id              UUID PRIMARY KEY,
  order_id        VARCHAR(255) NOT NULL,
  status          VARCHAR(50) NOT NULL,
  order_items     TEXT,  -- JSON serialized
  shipping_address TEXT, -- JSON serialized
  retry_count     INT DEFAULT 0,
  created_at      TIMESTAMP,
  updated_at      TIMESTAMP
);
```

---

### SQL Server (order, payment)

**Driver/ORM:** **Entity Framework Core 8** (C# .NET)

**Key Patterns:**

- Code-first migrations with EF Core
- Entity configurations via Fluent API
- Retry on transient failure (`EnableRetryOnFailure`)
- Event outbox table in order-service for reliable publishing
- `JsonStringEnumConverter` for enum serialization

**Example — Order Entity:**

```csharp
public class Order
{
    public Guid Id { get; set; }
    public string CustomerId { get; set; }
    public OrderStatus Status { get; set; }  // Enum
    public decimal TotalAmount { get; set; }
    public List<OrderItem> Items { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }
}

public class OrderItem
{
    public Guid Id { get; set; }
    public Guid OrderId { get; set; }
    public string ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}
```

**Order Statuses:** `Pending` → `Confirmed` → `Processing` → `Shipped` → `Delivered` / `Cancelled`

**Return Statuses:** `ReturnRequested` → `ReturnApproved` → `ReturnReceived` → `Refunded`

---

### MySQL (inventory)

**Driver/ORM:** **SQLAlchemy** ORM + **Flask-Migrate** (Alembic) (Python)

**Key Patterns:**

- ORM models with type hints
- Marshmallow schemas for request/response validation
- Alembic-managed migrations
- SKU as unique indexed column
- Reservation model for order-based stock reservations
- StockMovement model for audit trail

**Example — Inventory Models:**

```python
class InventoryItem(db.Model):
    id                 = Column(Integer, primary_key=True)
    sku                = Column(String(100), unique=True, index=True)
    quantity_available = Column(Integer, default=0)
    quantity_reserved  = Column(Integer, default=0)
    reorder_level      = Column(Integer, default=10)
    max_stock          = Column(Integer, default=1000)
    cost_per_unit      = Column(Numeric(10, 2))
    created_at         = Column(DateTime, default=datetime.utcnow)
    updated_at         = Column(DateTime, onupdate=datetime.utcnow)

class Reservation(db.Model):
    id          = Column(Integer, primary_key=True)
    sku         = Column(String(100), index=True)
    order_id    = Column(String(255))
    quantity    = Column(Integer)
    status      = Column(String(50))  # 'active', 'released', 'confirmed'
    created_at  = Column(DateTime, default=datetime.utcnow)
```

---

### Redis (cart)

**Driver:** **ioredis** (direct) or **Dapr State Store** (via Dapr SDK)

**Key Patterns:**

- `StorageFactory` selects driver based on `PLATFORM_MODE` (dapr vs direct)
- ETag-based optimistic concurrency for cart state
- TTL-based expiration: 30 days for user carts, 7 days for guest carts
- Cart stored as JSON blob keyed by user ID or guest UUID
- Max limits: configurable `maxItems` and `maxItemQuantity` per cart

**Cart Data Structure:**

```json
{
  "userId": "user-123",
  "items": [
    {
      "productId": "prod-456",
      "name": "Product Name",
      "price": 29.99,
      "quantity": 2,
      "image": "url"
    }
  ],
  "updatedAt": "2026-02-10T10:30:00Z"
}
```

---

## Migration Commands

| Service                 | Command                                                         | Tool                    |
| :---------------------- | :-------------------------------------------------------------- | :---------------------- |
| inventory-service       | `flask db migrate -m "description"` / `flask db upgrade`        | Flask-Migrate (Alembic) |
| order-service           | `dotnet ef migrations add <Name>` / `dotnet ef database update` | EF Core Migrations      |
| payment-service         | `dotnet ef migrations add <Name>` / `dotnet ef database update` | EF Core Migrations      |
| order-processor-service | Automatic on startup via Flyway                                 | Flyway                  |
| audit-service           | Via Knex.js migration files                                     | Knex.js                 |

---

## Database Seeding

Use `db-seeder` to populate demo data:

```bash
cd db-seeder
pip install -r requirements.txt

python seed.py                  # Seed all databases
python seed.py --users          # Seed users only
python seed.py --products       # Seed products only
python seed.py --inventory      # Seed inventory only
python seed.py --clear          # Clear ALL data (clean slate)
```

Demo credentials: `admin@xshopai.com` / `admin`, `guest@xshopai.com` / `guest`
