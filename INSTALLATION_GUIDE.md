# xshopai Platform — Installation Guide

Complete guide to setting up the xshopai e-commerce platform for development. Choose the setup path that best fits your workflow.

| Setup Option                                                        | Best For                        | Time    | Prerequisites                  |
| :------------------------------------------------------------------ | :------------------------------ | :------ | :----------------------------- |
| [**A. GitHub Codespaces**](#option-a-github-codespaces-recommended) | Fastest start, no local install | ~5 min  | GitHub account                 |
| [**B. Full Local Stack**](#option-b-local-development-full-stack)   | Full platform development       | ~30 min | Docker, all runtimes           |
| [**C. Single Service**](#option-c-single-service-development)       | Contributing to one service     | ~10 min | Docker, one runtime            |
| [**D. Local with Dapr**](#option-d-local-with-dapr)                 | Production-like local setup     | ~30 min | Docker, Dapr CLI, all runtimes |

---

## Prerequisites

### Required for All Local Options (B, C, D)

| Tool               | Minimum Version | Install                                                                |
| :----------------- | :-------------- | :--------------------------------------------------------------------- |
| **Git**            | 2.40+           | `winget install Git.Git` · `brew install git` · `sudo apt install git` |
| **Docker Desktop** | 4.x+            | [docker.com/get-docker](https://docs.docker.com/get-docker/)           |
| **VS Code**        | Latest          | [code.visualstudio.com](https://code.visualstudio.com/)                |

### Language Runtimes (Full Stack)

Install all runtimes below for Options B and D. For Option C, install only the runtime for the service you're working on.

| Runtime      | Version | Services                                                                                   | Install                                                                                                                           |
| :----------- | :------ | :----------------------------------------------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| **Node.js**  | 20 LTS  | admin, auth, user, review, cart, chat, notification, audit, web-bff, admin-ui, customer-ui | `winget install OpenJS.NodeJS.LTS` · `brew install node@20` · [nodejs.org](https://nodejs.org/)                                   |
| **Python**   | 3.11+   | product, inventory, db-seeder                                                              | `winget install Python.Python.3.11` · `brew install python@3.11` · [python.org](https://www.python.org/downloads/)                |
| **.NET SDK** | 8.0+    | order, payment                                                                             | `winget install Microsoft.DotNet.SDK.8` · `brew install dotnet@8` · [dotnet.microsoft.com](https://dotnet.microsoft.com/download) |
| **Java JDK** | 17+     | order-processor                                                                            | `winget install EclipseAdoptium.Temurin.17.JDK` · `brew install temurin@17` · [adoptium.net](https://adoptium.net/)               |
| **Maven**    | 3.9+    | order-processor                                                                            | `winget install Apache.Maven` · `brew install maven`                                                                              |

### Optional Tools

| Tool          | Purpose                 | Install                                                                                |
| :------------ | :---------------------- | :------------------------------------------------------------------------------------- |
| **Dapr CLI**  | Service mesh (Option D) | [docs.dapr.io/getting-started](https://docs.dapr.io/getting-started/install-dapr-cli/) |
| **Azure CLI** | Cloud deployment        | `winget install Microsoft.AzureCLI` · `brew install azure-cli`                         |

### Docker Resource Requirements

Allocate at least **8 GB RAM** and **4 CPUs** to Docker Desktop (Settings → Resources). The full infra stack runs 13 containers including SQL Server, which needs significant memory.

### Recommended VS Code Extensions

Open `dev/xshopai.code-workspace` and install all recommended extensions, or install manually:

| Category              | Extension                                              |
| :-------------------- | :----------------------------------------------------- |
| JavaScript/TypeScript | ESLint, Prettier, npm Intellisense                     |
| Python                | Python, Pylint, Black Formatter                        |
| Java                  | Java Extension Pack                                    |
| .NET/C#               | C# Dev Kit                                             |
| Docker                | Docker                                                 |
| Databases             | MongoDB for VS Code, SQLTools + PG/MSSQL/MySQL drivers |
| API Testing           | REST Client                                            |
| GitHub                | GitHub Pull Requests, GitHub Copilot                   |

---

## Option A: GitHub Codespaces (Recommended)

The fastest way to get started. Everything is pre-configured — infrastructure, services, and tooling.

### Requirements

- GitHub account with Codespaces access
- Machine type: **≥ 8 cores / 32 GB RAM** (required for 13 infra containers + 16 services)

### Setup Steps

1. **Open the dev repo**: Go to [github.com/xshopai/dev](https://github.com/xshopai/dev)

2. **Create a Codespace**: Click **Code → Codespaces → Create codespace on main**. Select an **8-core** machine.

3. **Configure secrets** (first time only): Go to [github.com/settings/codespaces](https://github.com/settings/codespaces) and add:

   | Secret                                  | Required | Description                                       |
   | :-------------------------------------- | :------- | :------------------------------------------------ |
   | `JWT_SECRET`                            | **Yes**  | Shared JWT signing key (any strong random string) |
   | `AZURE_OPENAI_API_KEY`                  | Optional | For chat-service AI features                      |
   | `AZURE_OPENAI_ENDPOINT`                 | Optional | For chat-service AI features                      |
   | `AZURE_COMMUNICATION_CONNECTION_STRING` | Optional | For notification-service email via Azure          |

4. **Wait for setup** (~5 minutes): The `postCreateCommand` automatically clones all repos, installs dependencies, starts infrastructure, and seeds databases.

5. **Start all services**:

   ```bash
   .devcontainer/dev.sh
   ```

6. **Verify**: Open the **Ports** tab in VS Code — all 16 services should be forwarded. Open Customer UI at the forwarded port 3000.

### Codespace Notes

- Connection strings use Docker service hostnames (e.g., `user-mongodb:27017`) not `localhost`
- Environment variables are pre-set in `devcontainer.json` → `remoteEnv`
- The multi-root workspace file opens automatically on first attach
- Forwarded ports are public by default for easy sharing

---

## Option B: Local Development (Full Stack)

Run the entire platform on your local machine. All 16 services communicate via HTTP (no Dapr sidecar).

### Setup Steps

```bash
# 1. Clone the dev repo
git clone https://github.com/xshopai/dev.git
cd dev/local

# 2. Run the automated setup (clones repos, starts infra, builds, seeds)
./setup.sh --seed

# 3. Start all 16 services
./dev.sh
```

The setup script runs 6 phases automatically:

| #   | Phase          | Script                        | What it does                                         |
| :-- | :------------- | :---------------------------- | :--------------------------------------------------- |
| 0   | Prerequisites  | `scripts/00-prerequisites.sh` | Checks Docker, Node, Python, Java, .NET, Git         |
| 1   | Clone          | `scripts/01-clone.sh`         | Clones all 17 repos in parallel (skips existing)     |
| 2   | Infrastructure | `scripts/04-infra.sh`         | Starts Docker Compose + waits for health checks      |
| 3   | Environment    | `scripts/03-env.sh`           | Copies `.env.example` → `.env` for each service      |
| 4   | Build          | `scripts/02-build.sh`         | Builds all services (wave-based parallel build)      |
| 5   | Seed           | `scripts/05-seed.sh`          | Seeds databases with demo users, products, inventory |

### Run Individual Steps

Each phase can be run independently for debugging:

```bash
bash scripts/00-prerequisites.sh   # Check tools are installed
bash scripts/01-clone.sh           # Clone repos (skips existing)
bash scripts/04-infra.sh           # Start Docker + health-check ports
bash scripts/03-env.sh             # Copy .env templates
bash scripts/02-build.sh           # Build all services (wave-based)
bash scripts/05-seed.sh            # Seed databases
```

### Manage Services

```bash
./dev.sh              # Start all 14 backend services + 2 UIs
./dev.sh --stop       # Stop all running services
```

Service logs are written to `dev/logs/<service-name>.log`:

```bash
tail -f ../logs/web-bff.log
tail -f ../logs/product-service.log
```

### Build Individual Services

```bash
./build.sh --all                     # Build everything (parallel)
./build.sh --all --sequential        # Build sequentially
./build.sh user-service              # Build a single service
./build.sh --all --clean             # Clean + rebuild
./build.sh --all --test              # Build + run tests
```

---

## Option C: Single Service Development

For contributors working on just one service. Only requires Docker and the relevant language runtime.

### Setup Steps

```bash
# 1. Clone the specific service
git clone https://github.com/xshopai/<service-name>.git
cd <service-name>

# 2. Start its database (if it has one)
docker-compose -f docker-compose.db.yml up -d

# 3. Set up environment variables
cp .env.example .env
# Edit .env if needed

# 4. Install dependencies and run (see table below)
```

### Language-Specific Commands

| Runtime        | Install                           | Run                                             | Test          |
| :------------- | :-------------------------------- | :---------------------------------------------- | :------------ |
| **Node.js**    | `npm install`                     | `npm start` or `npm run dev`                    | `npm test`    |
| **TypeScript** | `npm install`                     | `npm run dev`                                   | `npm test`    |
| **Python**     | `pip install -r requirements.txt` | `python main.py` or `uvicorn main:app --reload` | `pytest`      |
| **.NET**       | `dotnet restore`                  | `dotnet run`                                    | `dotnet test` |
| **Java**       | `mvn install -DskipTests`         | `mvn spring-boot:run`                           | `mvn test`    |
| **React**      | `npm install`                     | `npm start`                                     | `npm test`    |

### Service → Database Mapping

| Service                 | Database                           | Docker Compose                                   | Local Port |
| :---------------------- | :--------------------------------- | :----------------------------------------------- | :--------- |
| product-service         | MongoDB                            | `docker-compose.db.yml`                          | 27019      |
| user-service            | MongoDB                            | `docker-compose.db.yml`                          | 27018      |
| review-service          | MongoDB                            | `docker-compose.db.yml`                          | 27020      |
| auth-service            | MongoDB (shared with user-service) | Use user-service's DB                            | 27018      |
| admin-service           | MongoDB (shared with user-service) | Use user-service's DB                            | 27018      |
| inventory-service       | MySQL                              | `docker-compose.db.yml`                          | 3306       |
| order-service           | SQL Server                         | `docker-compose.db.yml`                          | 1434       |
| payment-service         | SQL Server                         | `docker-compose.db.yml`                          | 1433       |
| audit-service           | PostgreSQL                         | `docker-compose.db.yml`                          | 5434       |
| order-processor-service | PostgreSQL                         | `docker-compose.db.yml`                          | 5435       |
| cart-service            | Redis                              | Use `dev/docker-compose.yml` or standalone Redis | 6379       |
| notification-service    | — (email only)                     | —                                                | —          |
| chat-service            | — (OpenAI API)                     | —                                                | —          |
| web-bff                 | — (aggregator)                     | —                                                | —          |
| customer-ui             | — (frontend)                       | —                                                | —          |
| admin-ui                | — (frontend)                       | —                                                | —          |

### Health Check

Every backend service exposes a health endpoint:

```bash
curl http://localhost:<port>/health
```

---

## Option D: Local with Dapr

Run the platform locally with Dapr sidecars for pub/sub messaging, service invocation, and state management. This is the closest to the production deployment on Azure Container Apps.

### Prerequisites

1. Install **Dapr CLI**: [docs.dapr.io/getting-started](https://docs.dapr.io/getting-started/install-dapr-cli/)
2. Initialize Dapr (installs the sidecar runtime):
   ```bash
   dapr init
   ```

### How Dapr Works Locally

Each service runs with a Dapr sidecar that:

- **Pub/Sub**: Routes events through RabbitMQ (configured in `.dapr/components/event-bus.yaml`)
- **State Store**: Provides state management via Redis (for cart-service, order-processor)
- **Config**: Enables distributed tracing via Zipkin (`.dapr/config.yaml`)

```
┌─────────────┐    Dapr sidecar    ┌──────────────┐
│  Service A   │◄──────────────────►│  RabbitMQ    │
│  (port 8001) │   (port 3501)     │  (port 5672) │
└─────────────┘                    └──────────────┘
```

### Setup Steps

```bash
# 1. Start infrastructure (same as Option B)
cd dev
docker compose up -d

# 2. For each service, use run.sh/run.ps1 or VS Code tasks
cd ../product-service
./run.sh        # Linux/macOS
.\run.ps1       # Windows
```

### VS Code Tasks

Each service has pre-configured VS Code tasks for Dapr:

| Task                                | Description                          |
| :---------------------------------- | :----------------------------------- |
| `dapr-start` / `Start Dapr Sidecar` | Starts the service with Dapr sidecar |
| `dapr-stop` / `Stop Dapr Sidecar`   | Stops the Dapr sidecar process       |

Run via: **Terminal → Run Task** or `Ctrl+Shift+P` → "Tasks: Run Task"

### Dapr Component Files

Each service has a `.dapr/` directory with:

```
.dapr/
├── components/
│   ├── event-bus.yaml        # Pub/sub: RabbitMQ
│   ├── state-store.yaml      # State: Redis (if applicable)
│   └── subscriptions.yaml    # Event subscriptions (if applicable)
└── config.yaml               # Tracing, resiliency policies
```

### Running Multiple Services with Dapr

When running multiple services simultaneously, each needs unique Dapr ports:

| #   | Service                 | App Port | Dapr HTTP | Dapr gRPC |
| :-- | :---------------------- | :------- | :-------- | :-------- |
| 1   | product-service         | 8001     | 3501      | 50001     |
| 2   | user-service            | 8002     | 3502      | 50002     |
| 3   | admin-service           | 8003     | 3503      | 50003     |
| 4   | auth-service            | 8004     | 3504      | 50004     |
| 5   | inventory-service       | 8005     | 3505      | 50005     |
| 6   | order-service           | 8006     | 3506      | 50006     |
| 7   | order-processor-service | 8007     | 3507      | 50007     |
| 8   | cart-service            | 8008     | 3508      | 50008     |
| 9   | payment-service         | 8009     | 3509      | 50009     |
| 10  | review-service          | 8010     | 3510      | 50010     |
| 11  | notification-service    | 8011     | 3511      | 50011     |
| 12  | audit-service           | 8012     | 3512      | 50012     |
| 13  | chat-service            | 8013     | 3513      | 50013     |
| 14  | web-bff                 | 8014     | 3514      | 50014     |

> **Pattern**: Dapr HTTP = 3500 + service number, Dapr gRPC = 50000 + service number

---

## Infrastructure Services

The platform requires 13 infrastructure containers, all defined in `dev/docker-compose.yml`.

### Start/Stop Infrastructure

```bash
cd dev
docker compose up -d                  # Start all
docker compose ps                     # Check status
docker compose logs -f rabbitmq       # View specific logs
docker compose restart user-mongodb   # Restart one container
docker compose down                   # Stop (preserve data)
docker compose down --volumes         # Stop + DELETE all data ⚠️
```

### Container Reference

#### Service Discovery & Messaging

| Service  | Container      | Port(s)                | Credentials                    | Management UI                             |
| :------- | :------------- | :--------------------- | :----------------------------- | :---------------------------------------- |
| Consul   | `dev-consul`   | 8500, 8600/udp         | —                              | [localhost:8500](http://localhost:8500)   |
| RabbitMQ | `dev-rabbitmq` | 5672, 15672            | `admin` / `admin123`           | [localhost:15672](http://localhost:15672) |
| Zipkin   | `dev-zipkin`   | 9411                   | —                              | [localhost:9411](http://localhost:9411)   |
| Mailpit  | `dev-mailpit`  | 1025 (SMTP), 8025 (UI) | —                              | [localhost:8025](http://localhost:8025)   |
| Redis    | `dev-redis`    | 6379                   | Password: `redis_dev_pass_123` | —                                         |

#### Databases

| Database           | Container                      | Port  | User       | Password    | Database Name          |
| :----------------- | :----------------------------- | :---- | :--------- | :---------- | :--------------------- |
| User MongoDB       | `dev-user-mongodb`             | 27018 | `admin`    | `admin123`  | `user_service_db`      |
| Product MongoDB    | `dev-product-mongodb`          | 27019 | `admin`    | `admin123`  | `product_service_db`   |
| Review MongoDB     | `dev-review-mongodb`           | 27020 | `admin`    | `admin123`  | `review_service_db`    |
| Audit PostgreSQL   | `dev-audit-postgres`           | 5434  | `admin`    | `admin123`  | `audit_service_db`     |
| Order Processor PG | `dev-order-processor-postgres` | 5435  | `postgres` | `postgres`  | `order_processor_db`   |
| Order SQL Server   | `dev-order-sqlserver`          | 1434  | `sa`       | `Admin123!` | `order_service_db`     |
| Payment SQL Server | `dev-payment-sqlserver`        | 1433  | `sa`       | `Admin123!` | `payment_service_db`   |
| Inventory MySQL    | `dev-inventory-mysql`          | 3306  | `admin`    | `admin123`  | `inventory_service_db` |

### Connection Strings

For use in `.env` files or database clients:

```bash
# MongoDB
mongodb://admin:admin123@localhost:27018/user_service_db?authSource=admin
mongodb://admin:admin123@localhost:27019/product_service_db?authSource=admin
mongodb://admin:admin123@localhost:27020/review_service_db?authSource=admin

# PostgreSQL
postgresql://admin:admin123@localhost:5434/audit_service_db
postgresql://postgres:postgres@localhost:5435/order_processor_db

# SQL Server
Server=localhost,1434;Database=order_service_db;User Id=sa;Password=Admin123!;TrustServerCertificate=True
Server=localhost,1433;Database=payment_service_db;User Id=sa;Password=Admin123!;TrustServerCertificate=True

# MySQL
mysql://admin:admin123@localhost:3306/inventory_service_db

# Redis
redis://:redis_dev_pass_123@localhost:6379

# RabbitMQ
amqp://admin:admin123@localhost:5672
```

---

## Database Seeding

The `db-seeder` tool populates all databases with demo data for development and testing.

### Demo Credentials

After seeding:

| Role         | Email               | Password |
| :----------- | :------------------ | :------- |
| **Admin**    | `admin@xshopai.com` | `admin`  |
| **Customer** | `guest@xshopai.com` | `guest`  |

### Seed Data

| Data File             | Contents                                |
| :-------------------- | :-------------------------------------- |
| `data/users.json`     | Demo users (guest customer, admin)      |
| `data/products.json`  | 25 products across all UI categories    |
| `data/inventory.json` | Inventory records matching product SKUs |

### Run the Seeder

```bash
cd db-seeder

# Install dependencies
pip install -r requirements.txt

# Seed all data
python seed.py

# Or seed selectively
python seed.py --users        # Users only
python seed.py --products     # Products only
python seed.py --inventory    # Inventory only

# Clear and reseed
python seed.py --clear
```

> **Note:** When using the full-stack setup (Option B), the setup script runs the seeder automatically with `--seed`.

---

## Verification Checklist

After setup, verify the platform is running correctly.

### 1. Infrastructure Health

```bash
# RabbitMQ
curl -s http://localhost:15672/api/healthchecks/node -u admin:admin123 | head -1

# Zipkin
curl -s http://localhost:9411/health

# Mailpit
curl -s http://localhost:8025/api/v1/info

# Redis
docker exec dev-redis redis-cli -a redis_dev_pass_123 ping
```

### 2. Service Health Checks

```bash
# Backend services
curl -s http://localhost:8001/health    # product-service
curl -s http://localhost:8002/health    # user-service
curl -s http://localhost:8003/health    # admin-service
curl -s http://localhost:8004/health    # auth-service
curl -s http://localhost:8005/health    # inventory-service
curl -s http://localhost:8006/health    # order-service
curl -s http://localhost:8007/health    # order-processor-service
curl -s http://localhost:8008/health    # cart-service
curl -s http://localhost:8009/health    # payment-service
curl -s http://localhost:8010/health    # review-service
curl -s http://localhost:8011/health    # notification-service
curl -s http://localhost:8012/health    # audit-service
curl -s http://localhost:8013/health    # chat-service
curl -s http://localhost:8014/health    # web-bff
```

### 3. Frontend Verification

| Application | URL                                            | Expected                            |
| :---------- | :--------------------------------------------- | :---------------------------------- |
| Customer UI | [http://localhost:3000](http://localhost:3000) | Landing page with trending products |
| Admin UI    | [http://localhost:3001](http://localhost:3001) | Login page                          |

### 4. Smoke Test (End-to-End)

Walk through the complete workflow:

1. **Browse products**: Open [http://localhost:3000](http://localhost:3000) → see product listings
2. **Register a user**: Click "Sign Up" → fill form → submit
3. **Check email**: Open [http://localhost:8025](http://localhost:8025) (Mailpit) → find verification email → click link
4. **Log in**: Use the registered credentials
5. **Add to cart**: Browse a product → click "Add to Cart"
6. **Checkout**: Go to cart → proceed to checkout → complete order
7. **Admin access**: Open [http://localhost:3001](http://localhost:3001) → log in with `admin@xshopai.com` / `admin`

### 5. API Documentation

FastAPI services provide auto-generated interactive docs:

| Service           | Swagger UI                                        | ReDoc                                               |
| :---------------- | :------------------------------------------------ | :-------------------------------------------------- |
| product-service   | [localhost:8001/docs](http://localhost:8001/docs) | [localhost:8001/redoc](http://localhost:8001/redoc) |
| inventory-service | [localhost:8005/docs](http://localhost:8005/docs) | [localhost:8005/redoc](http://localhost:8005/redoc) |

---

## VS Code Workspace Setup

The platform uses a multi-root workspace to manage all 20 repositories in a single VS Code window.

### Open the Workspace

```bash
code dev/xshopai.code-workspace
```

Or from within VS Code: **File → Open Workspace from File** → select `dev/xshopai.code-workspace`.

### Workspace Structure

The workspace includes all service repos as top-level folders:

```
xshopai (Workspace)
├── admin-service/
├── admin-ui/
├── audit-service/
├── auth-service/
├── cart-service/
├── chat-service/
├── customer-ui/
├── db-seeder/
├── dev/
├── docs/
├── infrastructure/
├── inventory-service/
├── notification-service/
├── order-processor-service/
├── order-service/
├── payment-service/
├── product-service/
├── review-service/
├── user-service/
└── web-bff/
```

### Pre-configured Settings

| Setting           | Value                                                       | Effect                     |
| :---------------- | :---------------------------------------------------------- | :------------------------- |
| Format on Save    | Enabled                                                     | Auto-formats files on save |
| Default Formatter | Prettier                                                    | For JS/TS/JSON/HTML/CSS    |
| Python Formatter  | Black                                                       | PEP 8 compliant formatting |
| C# Formatter      | C# Dev Kit                                                  | .NET conventions           |
| Excluded Files    | `node_modules`, `__pycache__`, `target`, `bin/Debug`, `obj` | Reduces clutter            |

---

## Troubleshooting

### Port Already in Use

**Symptom:** Service fails to start with `EADDRINUSE` or "Address already in use"

**Fix:**

```bash
# Find what's using the port
# Windows
netstat -ano | findstr :<port>
taskkill /PID <pid> /F

# macOS/Linux
lsof -i :<port>
kill -9 <pid>
```

### Docker Out of Memory

**Symptom:** Containers crash or restart repeatedly, especially SQL Server

**Fix:** Increase Docker Desktop memory allocation to at least **8 GB** (Settings → Resources → Memory). The full stack uses ~6 GB.

### SQL Server Won't Start (Apple Silicon / ARM64)

**Symptom:** `dev-order-sqlserver` or `dev-payment-sqlserver` fails on Apple M1/M2/M3

**Fix:** SQL Server 2022 has native ARM64 support. Ensure you're using image `mcr.microsoft.com/mssql/server:2022-latest`. If issues persist, enable Rosetta emulation in Docker Desktop (Settings → General → "Use Rosetta for x86_64/amd64 emulation").

### MongoDB Authentication Failed

**Symptom:** `MongoServerError: Authentication failed`

**Fix:** The MongoDB instances use `admin` / `admin123` with `authSource=admin`. Ensure your connection string includes `?authSource=admin`:

```
mongodb://admin:admin123@localhost:27018/user_service_db?authSource=admin
```

### Dapr Init Fails

**Symptom:** `dapr init` fails with Docker errors

**Fix:**

1. Ensure Docker Desktop is running
2. Remove existing Dapr containers: `docker rm -f dapr_placement dapr_zipkin dapr_redis`
3. Retry: `dapr init`

### Missing Environment Variables

**Symptom:** Service starts but crashes with "missing required environment variable"

**Fix:**

```bash
# Ensure .env file exists
cp .env.example .env

# For .NET services, check appsettings.Development.json exists
# For Java services, check application-dev.properties exists
```

### Services Can't Reach Databases

**Symptom:** Connection refused errors on database ports

**Fix:**

1. Check containers are running: `docker compose ps` (from `dev/` directory)
2. SQL Server and MySQL can take 30–60 seconds to initialize on first start
3. Re-run health checks: `bash scripts/04-infra.sh`

### Build Failures

**Symptom:** `build.sh` fails for a specific service

**Fix:**

```bash
# Clean and rebuild a single service
cd dev/local
./build.sh <service-name> --clean

# Check the build output
cat ../logs/build.log

# For .NET services, ensure correct SDK version
dotnet --list-sdks

# For Java services, ensure JAVA_HOME is set
echo $JAVA_HOME
```

### Reset Everything

When all else fails, start fresh:

```bash
# Stop all services
cd dev/local
./dev.sh --stop

# Tear down infrastructure + delete all data
cd ..
docker compose down --volumes

# Re-run full setup
cd local
./setup.sh --seed
```

---

## Platform Architecture Overview

The xshopai platform consists of 16 services built with 5 technology stacks:

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│   ┌──────────────┐                    ┌──────────────┐      │
│   │ Customer UI  │                    │  Admin UI    │      │
│   │ React :3000  │                    │ React :3001  │      │
│   └──────┬───────┘                    └──────┬───────┘      │
│          │                                   │              │
│          └─────────────┬─────────────────────┘              │
│                        ▼                                    │
│               ┌──────────────┐                              │
│               │   Web BFF    │                              │
│               │  TS/Express  │                              │
│               │   :8014      │                              │
│               └──────┬───────┘                              │
│                      │                                      │
├──────────────────────┼──────────────────────────────────────┤
│                Backend Services (via HTTP / Dapr Pub/Sub)    │
│                      │                                      │
│  ┌─────────┐  ┌──────────┐  ┌─────────┐  ┌──────────────┐ │
│  │ Product  │  │  User    │  │  Auth   │  │    Admin     │ │
│  │ Python   │  │ Node.js  │  │ Node.js │  │   Node.js    │ │
│  │ :8001    │  │ :8002    │  │ :8004   │  │   :8003      │ │
│  └────┬─────┘  └────┬─────┘  └────┬────┘  └──────┬───────┘ │
│       │              │             │              │         │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌───────────┐   │
│  │Inventory│  │  Order   │  │ Payment  │  │Order Proc. │   │
│  │ Python  │  │  .NET    │  │  .NET    │  │   Java     │   │
│  │ :8005   │  │  :8006   │  │  :8009   │  │   :8007    │   │
│  └────┬─────┘  └────┬─────┘  └────┬────┘  └──────┬──────┘  │
│       │              │             │              │         │
│  ┌─────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐    │
│  │  Cart   │  │ Review   │  │  Notify  │  │  Audit   │    │
│  │   TS    │  │ Node.js  │  │    TS    │  │    TS    │    │
│  │  :8008  │  │  :8010   │  │  :8011   │  │  :8012   │    │
│  └─────────┘  └──────────┘  └──────────┘  └──────────┘    │
│                                                             │
│  ┌─────────┐                                               │
│  │  Chat   │                                               │
│  │   TS    │                                               │
│  │  :8013  │                                               │
│  └─────────┘                                               │
├─────────────────────────────────────────────────────────────┤
│                    Data Layer                                │
│  MongoDB(×3) · PostgreSQL(×2) · SQL Server(×2) · MySQL · Redis │
│                                                             │
│                  Infrastructure                             │
│  RabbitMQ · Zipkin · Mailpit · Consul                       │
└─────────────────────────────────────────────────────────────┘
```

For the full architecture, see [PLATFORM_ARCHITECTURE.md](PLATFORM_ARCHITECTURE.md).

---

## Next Steps

- **Explore the architecture**: [PLATFORM_ARCHITECTURE.md](PLATFORM_ARCHITECTURE.md)
- **Understand messaging**: [MESSAGING_ARCHITECTURE.md](MESSAGING_ARCHITECTURE.md)
- **Contribute**: [CONTRIBUTING.md](CONTRIBUTING.md)
- **Review events**: [CLOUDEVENTS_STANDARD.md](CLOUDEVENTS_STANDARD.md)
- **Monitor services**: [MONITORING_ARCHITECTURE.md](MONITORING_ARCHITECTURE.md)

---

> **⚠️ Security Note:** All credentials in this guide are for **local development only**. Never use these in production. Production deployments use Azure Key Vault, managed identities, and OIDC authentication.
