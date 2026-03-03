# xshopai Platform — Quick Start

Get the full platform running in under 10 minutes using Docker Compose.

> For detailed setup options including Codespaces, single-service dev, and Dapr integration, see the [Installation Guide](INSTALLATION_GUIDE.md).

---

## Prerequisites

| Tool               | Version | Check              |
| :----------------- | :------ | :----------------- |
| **Git**            | 2.40+   | `git --version`    |
| **Docker Desktop** | 4.x+    | `docker --version` |
| **Node.js**        | 20 LTS  | `node --version`   |
| **Python**         | 3.11+   | `python --version` |
| **.NET SDK**       | 8.0+    | `dotnet --version` |
| **Java JDK**       | 17+     | `java --version`   |
| **Maven**          | 3.9+    | `mvn --version`    |

> **Docker Resources**: Allocate at least **8 GB RAM** and **4 CPUs** in Docker Desktop → Settings → Resources.

---

## 1. Clone the Repositories

```bash
mkdir xshopai && cd xshopai

# Clone all repositories (services + infrastructure)
for repo in dev docs infrastructure \
  admin-service admin-ui audit-service auth-service \
  cart-service chat-service customer-ui db-seeder \
  inventory-service notification-service \
  order-processor-service order-service \
  payment-service product-service review-service \
  user-service web-bff; do
  git clone https://github.com/xshopai/$repo.git
done
```

## 2. Start Infrastructure

```bash
cd dev
docker compose up -d
```

This starts all infrastructure containers:

| Container      | Ports               | Purpose                     |
| :------------- | :------------------ | :-------------------------- |
| MongoDB × 3    | 27018, 27019, 27020 | user, product, review DBs   |
| PostgreSQL × 2 | 5434, 5435          | audit, order-processor DBs  |
| SQL Server × 2 | 1433, 1434          | payment, order DBs          |
| MySQL          | 3306                | inventory DB                |
| Redis          | 6379                | cart state store            |
| RabbitMQ       | 5672 / 15672        | message broker / management |
| Zipkin         | 9411                | distributed tracing         |
| Mailpit        | 1025 / 8025         | SMTP / email viewer         |

Verify all containers are healthy:

```bash
docker compose ps
```

## 3. Seed the Databases

```bash
cd ../db-seeder
pip install -r requirements.txt
python seed.py
```

This creates demo users, products, and inventory data across all databases.

**Demo Credentials:**

| Account | Email             | Password |
| :------ | :---------------- | :------- |
| Admin   | admin@xshopai.com | admin    |
| Guest   | guest@xshopai.com | guest    |

## 4. Install Dependencies & Start Services

Open a terminal for each service group and install dependencies:

### Node.js Services

```bash
# In separate terminals, for each service:
cd admin-service   && npm install && npm run dev
cd auth-service    && npm install && npm run dev
cd user-service    && npm install && npm run dev
cd review-service  && npm install && npm run dev
cd web-bff         && npm install && npm run dev
```

### TypeScript Services

```bash
cd cart-service          && npm install && npm run dev
cd chat-service          && npm install && npm run dev
cd notification-service  && npm install && npm run dev
cd audit-service         && npm install && npm run dev
```

### Python Services

```bash
cd product-service   && pip install -r requirements.txt && python main.py
cd inventory-service && pip install -r requirements.txt && python main.py
```

### .NET Services

```bash
cd order-service   && dotnet run --project OrderService.Api
cd payment-service && dotnet run --project PaymentService
```

### Java Service

```bash
cd order-processor-service && mvn spring-boot:run
```

### Frontend Applications

```bash
cd customer-ui && npm install && npm start    # → http://localhost:3000
cd admin-ui    && npm install && npm start     # → http://localhost:3001
```

## 5. Verify

Open these URLs to confirm the platform is running:

| URL                                              | What You'll See                      |
| :----------------------------------------------- | :----------------------------------- |
| [http://localhost:3000](http://localhost:3000)   | Customer storefront                  |
| [http://localhost:3001](http://localhost:3001)   | Admin dashboard                      |
| [http://localhost:15672](http://localhost:15672) | RabbitMQ management (admin/admin123) |
| [http://localhost:9411](http://localhost:9411)   | Zipkin tracing                       |
| [http://localhost:8025](http://localhost:8025)   | Mailpit email viewer                 |

---

## Service Port Reference

| Service                 | Port | Technology           |
| :---------------------- | :--- | :------------------- |
| product-service         | 8001 | Python / FastAPI     |
| user-service            | 8002 | Node.js / Express    |
| admin-service           | 8003 | Node.js / Express    |
| auth-service            | 8004 | Node.js / Express    |
| inventory-service       | 8005 | Python / Flask       |
| order-service           | 8006 | C# / ASP.NET Core    |
| order-processor-service | 8007 | Java / Spring Boot   |
| cart-service            | 8008 | TypeScript / Express |
| payment-service         | 8009 | C# / ASP.NET Core    |
| review-service          | 8010 | Node.js / Express    |
| notification-service    | 8011 | TypeScript / Express |
| audit-service           | 8012 | TypeScript / Express |
| chat-service            | 8013 | TypeScript / Express |
| web-bff                 | 8014 | TypeScript / Express |
| customer-ui             | 3000 | React                |
| admin-ui                | 3001 | React / TypeScript   |

---

## Stopping Everything

```bash
# Stop all services (Ctrl+C in each terminal)

# Stop infrastructure
cd dev
docker compose down
```

To also remove database volumes (full reset):

```bash
docker compose down -v
```

---

## What's Next?

- **[Installation Guide](INSTALLATION_GUIDE.md)** — Detailed setup with Codespaces, Dapr, and single-service options
- **[Platform Architecture](PLATFORM_ARCHITECTURE.md)** — How the services fit together
- **[Messaging Architecture](MESSAGING_ARCHITECTURE.md)** — Event-driven communication patterns
- **[Contributing](CONTRIBUTING.md)** — How to contribute to the project
