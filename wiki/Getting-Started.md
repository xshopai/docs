# Getting Started

Step-by-step guide to running the xshopai platform locally.

> **Fastest path?** See the [Quick Start](../QUICKSTART.md) for a condensed version.
> **All setup options?** See the [Installation Guide](../INSTALLATION_GUIDE.md) for Codespaces, single-service, and Dapr options.

---

## Prerequisites

### Required Software

| Tool           | Version | Verify             | Install                                                       |
| :------------- | :------ | :----------------- | :------------------------------------------------------------ |
| Git            | 2.40+   | `git --version`    | [git-scm.com](https://git-scm.com/)                           |
| Docker Desktop | 4.x+    | `docker --version` | [docker.com](https://docs.docker.com/get-docker/)             |
| Node.js        | 20 LTS  | `node --version`   | [nodejs.org](https://nodejs.org/)                             |
| Python         | 3.11+   | `python --version` | [python.org](https://www.python.org/)                         |
| .NET SDK       | 8.0+    | `dotnet --version` | [dotnet.microsoft.com](https://dotnet.microsoft.com/download) |
| Java JDK       | 17+     | `java --version`   | [adoptium.net](https://adoptium.net/)                         |
| Maven          | 3.9+    | `mvn --version`    | [maven.apache.org](https://maven.apache.org/)                 |

### Docker Resources

Allocate at least **8 GB RAM** and **4 CPUs** to Docker Desktop (Settings → Resources). The full stack runs 13 infrastructure containers including SQL Server instances that need significant memory.

### VS Code Workspace

Open `dev/xshopai.code-workspace` for the multi-root workspace with all services.

---

## Step 1: Clone All Repositories

```bash
mkdir xshopai && cd xshopai

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

## Step 2: Start Infrastructure

```bash
cd dev
docker compose up -d
```

Wait for all containers to reach `healthy` state:

```bash
docker compose ps
```

### Infrastructure Containers

| Container                | Image                 | Ports        | Credentials         |
| :----------------------- | :-------------------- | :----------- | :------------------ |
| user-mongodb             | mongo:latest          | 27018        | admin / admin123    |
| product-mongodb          | mongo:latest          | 27019        | admin / admin123    |
| review-mongodb           | mongo:latest          | 27020        | admin / admin123    |
| audit-postgres           | postgres:16           | 5434         | postgres / postgres |
| order-processor-postgres | postgres:16           | 5435         | postgres / postgres |
| order-sqlserver          | mssql/server:2022     | 1434         | sa / Admin123!      |
| payment-sqlserver        | mssql/server:2022     | 1433         | sa / Admin123!      |
| inventory-mysql          | mysql:latest          | 3306         | admin / admin123    |
| redis                    | redis:7-alpine        | 6379         | redis_dev_pass_123  |
| rabbitmq                 | rabbitmq:3-management | 5672 / 15672 | admin / admin123    |
| zipkin                   | openzipkin/zipkin     | 9411         | —                   |
| mailpit                  | axllent/mailpit       | 1025 / 8025  | —                   |
| consul                   | hashicorp/consul:1.17 | 8500         | —                   |

## Step 3: Seed Demo Data

```bash
cd ../db-seeder
pip install -r requirements.txt
python seed.py
```

This creates users, products, and inventory across all databases.

| Account | Email             | Password |
| :------ | :---------------- | :------- |
| Admin   | admin@xshopai.com | admin    |
| Guest   | guest@xshopai.com | guest    |

## Step 4: Start Services

Each service runs in its own terminal. Install dependencies and start:

### Python Services

```bash
# Product Service (port 8001)
cd product-service
pip install -r requirements.txt
python main.py

# Inventory Service (port 8005)
cd inventory-service
pip install -r requirements.txt
python main.py
```

### Node.js Services

```bash
# User Service (port 8002)
cd user-service && npm install && npm run dev

# Admin Service (port 8003)
cd admin-service && npm install && npm run dev

# Auth Service (port 8004)
cd auth-service && npm install && npm run dev

# Review Service (port 8010)
cd review-service && npm install && npm run dev
```

### TypeScript Services

```bash
# Cart Service (port 8008)
cd cart-service && npm install && npm run dev

# Notification Service (port 8011)
cd notification-service && npm install && npm run dev

# Audit Service (port 8012)
cd audit-service && npm install && npm run dev

# Chat Service (port 8013)
cd chat-service && npm install && npm run dev

# Web BFF (port 8014)
cd web-bff && npm install && npm run dev
```

### .NET Services

```bash
# Order Service (port 8006)
cd order-service
dotnet run --project OrderService.Api

# Payment Service (port 8009)
cd payment-service
dotnet run --project PaymentService
```

### Java Service

```bash
# Order Processor Service (port 8007)
cd order-processor-service
mvn spring-boot:run
```

### Frontend Applications

```bash
# Customer UI (port 3000)
cd customer-ui && npm install && npm start

# Admin UI (port 3001)
cd admin-ui && npm install && npm start
```

## Step 5: Verify

| URL                    | Description              |
| :--------------------- | :----------------------- |
| http://localhost:3000  | Customer storefront      |
| http://localhost:3001  | Admin dashboard          |
| http://localhost:15672 | RabbitMQ management      |
| http://localhost:9411  | Zipkin tracing           |
| http://localhost:8025  | Mailpit email viewer     |
| http://localhost:8500  | Consul service discovery |

---

## Stopping

```bash
# Ctrl+C in each service terminal, then:
cd dev
docker compose down

# To also wipe database volumes:
docker compose down -v
```

---

## Next Steps

- [Service Catalog](Service-Catalog.md) — Detailed info on each service
- [Dapr Integration Guide](Dapr-Integration-Guide.md) — Running with Dapr sidecars
- [Testing Guide](Testing-Guide.md) — How to run tests across all services
