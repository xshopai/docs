# Deployment Guide

xshopai supports three deployment targets: **local Docker**, **Azure App Service**, and **Azure Container Apps**. This guide covers each approach.

---

## Deployment Targets

| Target                     | Tool                   | IaC                                        | Directory                            |
| :------------------------- | :--------------------- | :----------------------------------------- | :----------------------------------- |
| Local Docker               | Docker Compose         | `dev/docker-compose.yml`                   | `dev/`                               |
| Azure App Service          | Bicep + GitHub Actions | `infrastructure/app-service/bicep/`        | `infrastructure/app-service/`        |
| Azure App Service (Docker) | Bicep + GitHub Actions | `infrastructure/app-service-docker/bicep/` | `infrastructure/app-service-docker/` |
| Azure Container Apps       | Azure CLI bash scripts | `infrastructure/aca/bash/`                 | `infrastructure/aca/`                |

---

## Local Docker Deployment

### Prerequisites

- Docker Desktop with 8 GB+ memory allocated
- Docker Compose v2

### Start All Infrastructure

```bash
cd dev
docker compose up -d
```

This starts **15 containers**:

| Container                    | Image                   | Port(s)     |
| :--------------------------- | :---------------------- | :---------- |
| RabbitMQ                     | rabbitmq:4-management   | 5672, 15672 |
| Redis                        | redis:7                 | 6379        |
| MongoDB (user)               | mongo:8                 | 27018       |
| MongoDB (product)            | mongo:8                 | 27019       |
| MongoDB (review)             | mongo:8                 | 27020       |
| MySQL (inventory)            | mysql:8                 | 3306        |
| PostgreSQL (audit)           | postgres:16             | 5434        |
| PostgreSQL (order-processor) | postgres:16             | 5435        |
| SQL Server (order)           | mcr.microsoft.com/mssql | 1434        |
| SQL Server (payment)         | mcr.microsoft.com/mssql | 1433        |
| Zipkin                       | openzipkin/zipkin       | 9411        |
| Mailpit                      | axllent/mailpit         | 1025, 8025  |
| Consul                       | hashicorp/consul        | 8500        |
| mongo-express (product)      | mongo-express           | 8081        |
| mongo-express (user)         | mongo-express           | 8082        |

### Seed Databases

```bash
cd db-seeder
pip install -r requirements.txt
python seed.py
```

### Start Services Individually

Each service runs on its native port (8001–8014) during local development. See [Getting Started](Getting-Started.md) for per-language startup commands.

### Stop Everything

```bash
cd dev
docker compose down        # Stop containers
docker compose down -v     # Stop + remove volumes (clean slate)
```

---

## Azure App Service Deployment

### Architecture

```
                    ┌─────────────────────────────────┐
                    │   Azure Resource Group           │
                    │                                  │
                    │  ┌─────────────┐                 │
                    │  │ App Service │ × 16 services   │
                    │  │   Plans     │   + 2 UIs       │
                    │  └─────────────┘                 │
                    │  ┌─────────────┐                 │
                    │  │    ACR      │ Container images │
                    │  └─────────────┘                 │
                    │  ┌─────────────┐                 │
                    │  │  Key Vault  │ Secrets          │
                    │  └─────────────┘                 │
                    │  ┌─────────────┐                 │
                    │  │  App        │ Telemetry        │
                    │  │  Insights   │                  │
                    │  └─────────────┘                 │
                    │  ┌─────────────┐                 │
                    │  │ Databases   │ Cosmos, SQL,     │
                    │  │             │ MySQL, Postgres  │
                    │  └─────────────┘                 │
                    └─────────────────────────────────┘
```

### Infrastructure as Code (Bicep)

```
infrastructure/app-service/bicep/
├── main.bicep                    # Root orchestration
├── parameters/
│   ├── dev.bicepparam            # Dev environment
│   ├── staging.bicepparam        # Staging
│   └── prod.bicepparam           # Production
└── modules/
    ├── appServicePlan.bicep      # App Service Plans
    ├── webApp.bicep              # Web App per service
    ├── containerRegistry.bicep   # ACR
    ├── keyVault.bicep            # Key Vault
    ├── appInsights.bicep         # Application Insights
    ├── logAnalytics.bicep        # Log Analytics Workspace
    ├── cosmosDb.bicep            # Cosmos DB for MongoDB API
    ├── sqlServer.bicep           # Azure SQL
    ├── mysql.bicep               # Azure MySQL Flexible
    ├── postgresql.bicep          # Azure PostgreSQL Flexible
    ├── redis.bicep               # Azure Cache for Redis
    ├── serviceBus.bicep          # Azure Service Bus
    └── ...
```

### Manual Deployment

```bash
# Login to Azure
az login

# Deploy infrastructure
az deployment group create \
  --resource-group rg-xshopai-dev \
  --template-file infrastructure/app-service/bicep/main.bicep \
  --parameters infrastructure/app-service/bicep/parameters/dev.bicepparam
```

### CI/CD Deployment

Push to `main` in the `infrastructure` repo triggers the deployment pipeline:

1. **Validate** — Bicep linting and template validation
2. **What-If** — Preview resource changes
3. **Manual Approval** — Gate via GitHub Environments
4. **Deploy** — Execute `az deployment group create`

Service repos trigger their own pipelines on push to `main`:

1. Build & test
2. Docker build
3. Push image to ACR
4. Deploy to App Service

---

## Azure Container Apps Deployment

### Architecture

Azure Container Apps provides a Dapr-native container hosting platform:

```
┌─────────────────────────────────────┐
│   Container Apps Environment        │
│                                     │
│  ┌──────────┐  ┌──────────┐        │
│  │ Service  │  │ Service  │  ×16   │
│  │ (Dapr)   │  │ (Dapr)   │        │
│  └──────────┘  └──────────┘        │
│                                     │
│  ┌──────────────────────┐           │
│  │  Dapr Components     │           │
│  │  (pub/sub, state)    │           │
│  └──────────────────────┘           │
└─────────────────────────────────────┘
```

### Deployment Scripts

```
infrastructure/aca/bash/
├── deploy.sh              # Main deployment orchestrator
├── deploy-services.sh     # Deploy all service containers
├── deploy-infra.sh        # Deploy supporting infrastructure
└── config/
    └── env.sh             # Environment variables
```

### Running ACA Deployment

```bash
cd infrastructure/aca/bash
chmod +x deploy.sh
./deploy.sh
```

---

## Environment Configuration

### Production Environment Variables

Each service requires environment variables configured in the deployment target:

| Variable         | Source (App Service) | Source (Container Apps)  |
| :--------------- | :------------------- | :----------------------- |
| `PORT`           | App Setting (8080)   | Container env var (8080) |
| `NODE_ENV`       | App Setting          | Container env var        |
| `JWT_SECRET`     | Key Vault reference  | Dapr secrets store       |
| `DATABASE_URL`   | Key Vault reference  | Dapr secrets store       |
| `DAPR_HTTP_PORT` | N/A (no Dapr)        | Auto-configured (3500)   |
| `PLATFORM_MODE`  | `direct`             | `dapr`                   |

### Key Vault Integration

App Service uses Key Vault references in app settings:

```
@Microsoft.KeyVault(SecretUri=https://kv-xshopai.vault.azure.net/secrets/jwt-secret/)
```

Container Apps uses Dapr secrets store component pointing to Key Vault.

### Azure Managed Databases

| Local               | Azure Equivalent                              |
| :------------------ | :-------------------------------------------- |
| MongoDB (Docker)    | Azure Cosmos DB for MongoDB API               |
| SQL Server (Docker) | Azure SQL Database                            |
| PostgreSQL (Docker) | Azure Database for PostgreSQL Flexible Server |
| MySQL (Docker)      | Azure Database for MySQL Flexible Server      |
| Redis (Docker)      | Azure Cache for Redis                         |
| RabbitMQ (Docker)   | Azure Service Bus                             |

---

## Health Checks

All services expose health endpoints used by deployment targets:

| Endpoint            | Purpose         | Azure Config                   |
| :------------------ | :-------------- | :----------------------------- |
| `GET /health`       | Liveness probe  | App Service Health Check path  |
| `GET /health/ready` | Readiness probe | Container Apps readiness probe |

### App Service Health Check

Configured in Bicep:

```bicep
resource webApp 'Microsoft.Web/sites@2023-01-01' = {
  properties: {
    siteConfig: {
      healthCheckPath: '/health'
    }
  }
}
```

### Container Apps Health Probes

```bicep
resource containerApp 'Microsoft.App/containerApps@2023-05-01' = {
  properties: {
    template: {
      containers: [{
        probes: [
          { type: 'Liveness', httpGet: { path: '/health', port: 8080 } }
          { type: 'Readiness', httpGet: { path: '/health/ready', port: 8080 } }
        ]
      }]
    }
  }
}
```

---

## Tagging & Resource Management

All Azure resources include standard tags:

```bicep
tags: {
  environment: 'dev'       // dev | staging | production
  project: 'xshopai'
  managedBy: 'bicep'       // bicep | cli
}
```

---

## Rollback

### App Service

```bash
# List recent deployments
az webapp deployment list --resource-group rg-xshopai --name app-product-service-xshopai

# Revert to previous image
az webapp config container set \
  --resource-group rg-xshopai \
  --name app-product-service-xshopai \
  --docker-custom-image-name acr-xshopai.azurecr.io/product-service:<previous-sha>
```

### Container Apps

```bash
# List revisions
az containerapp revision list --name product-service --resource-group rg-xshopai

# Activate previous revision
az containerapp revision activate --name product-service --resource-group rg-xshopai --revision <revision-name>
```

---

## Pre-Deployment Checklist

- [ ] All tests pass (`npm test` / `pytest` / `dotnet test` / `mvn test`)
- [ ] Docker image builds successfully locally
- [ ] Environment variables documented and configured
- [ ] Database migrations applied (if applicable)
- [ ] Health check endpoint responds 200
- [ ] Key Vault secrets created for sensitive values
- [ ] OIDC federated credentials configured for GitHub Actions
- [ ] Resource group and App Service Plan exist
- [ ] ACR login server configured in pipeline secrets
