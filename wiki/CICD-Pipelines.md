# CI/CD Pipelines

xshopai uses **GitHub Actions** for continuous integration and deployment. Every service has its own workflow that builds, tests, containerizes, and deploys to Azure App Service.

---

## Pipeline Inventory

### Service Pipelines (16)

Every microservice and UI has a `ci-app-service.yml` workflow:

| Service                 | Workflow                               | Language    | Build Tool         |
| :---------------------- | :------------------------------------- | :---------- | :----------------- |
| product-service         | `.github/workflows/ci-app-service.yml` | Python      | pip + Docker       |
| user-service            | `.github/workflows/ci-app-service.yml` | Node.js ESM | npm + Docker       |
| admin-service           | `.github/workflows/ci-app-service.yml` | Node.js ESM | npm + Docker       |
| auth-service            | `.github/workflows/ci-app-service.yml` | Node.js ESM | npm + Docker       |
| review-service          | `.github/workflows/ci-app-service.yml` | Node.js ESM | npm + Docker       |
| cart-service            | `.github/workflows/ci-app-service.yml` | TypeScript  | npm + tsc + Docker |
| web-bff                 | `.github/workflows/ci-app-service.yml` | TypeScript  | npm + tsc + Docker |
| notification-service    | `.github/workflows/ci-app-service.yml` | TypeScript  | npm + tsc + Docker |
| audit-service           | `.github/workflows/ci-app-service.yml` | TypeScript  | npm + tsc + Docker |
| chat-service            | `.github/workflows/ci-app-service.yml` | TypeScript  | npm + tsc + Docker |
| order-service           | `.github/workflows/ci-app-service.yml` | C# / .NET 8 | dotnet + Docker    |
| payment-service         | `.github/workflows/ci-app-service.yml` | C# / .NET 8 | dotnet + Docker    |
| inventory-service       | `.github/workflows/ci-app-service.yml` | Python      | pip + Docker       |
| order-processor-service | `.github/workflows/ci-app-service.yml` | Java 17     | Maven + Docker     |
| customer-ui             | `.github/workflows/ci-app-service.yml` | React (JSX) | npm + Docker       |
| admin-ui                | `.github/workflows/ci-app-service.yml` | React (TSX) | npm + Docker       |

### Infrastructure Pipelines (2)

Located in the `infrastructure` repo:

| Workflow                 | Purpose                                     | Tool                         |
| :----------------------- | :------------------------------------------ | :--------------------------- |
| `deploy-app-service.yml` | Deploy Bicep templates to Azure App Service | `az deployment group create` |
| `deploy-aca.yml`         | Deploy to Azure Container Apps              | Azure CLI scripts            |

---

## Service Pipeline Structure

Each service's `ci-app-service.yml` follows a **4-job pattern**:

```
┌──────────┐    ┌──────────┐    ┌──────────┐    ┌──────────┐
│  Build &  │───▶│  Docker  │───▶│ Push to  │───▶│ Deploy   │
│   Test    │    │  Build   │    │   ACR    │    │ App Svc  │
└──────────┘    └──────────┘    └──────────┘    └──────────┘
```

### Job Details

#### 1. Build & Test

Installs dependencies, compiles (if needed), and runs tests.

**Node.js / TypeScript:**

```yaml
- uses: actions/setup-node@v4
  with:
    node-version: '20'
    cache: 'npm'
- run: npm ci
- run: npm run build # TypeScript only
- run: npm test
```

**Python:**

```yaml
- uses: actions/setup-python@v5
  with:
    python-version: '3.11'
- run: pip install -r requirements.txt
- run: pytest tests/ -v
```

**.NET:**

```yaml
- uses: actions/setup-dotnet@v4
  with:
    dotnet-version: '8.0.x'
- run: dotnet restore
- run: dotnet build --no-restore
- run: dotnet test --no-build
```

**Java:**

```yaml
- uses: actions/setup-java@v4
  with:
    java-version: '17'
    distribution: 'temurin'
    cache: 'maven'
- run: mvn clean verify
```

#### 2. Docker Build

Multi-stage Dockerfile builds optimized per language:

```yaml
- uses: docker/build-push-action@v5
  with:
    context: .
    file: Dockerfile
    push: false
    tags: ${{ env.ACR_NAME }}.azurecr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

#### 3. Push to ACR

Authenticates to Azure Container Registry and pushes the image:

```yaml
- uses: azure/docker-login@v1
  with:
    login-server: ${{ env.ACR_NAME }}.azurecr.io
    username: ${{ secrets.ACR_USERNAME }} # Or OIDC
    password: ${{ secrets.ACR_PASSWORD }}
- run: docker push ${{ env.ACR_NAME }}.azurecr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

#### 4. Deploy to App Service

Updates the Azure App Service with the new image:

```yaml
- uses: azure/webapps-deploy@v2
  with:
    app-name: ${{ env.APP_NAME }}
    images: ${{ env.ACR_NAME }}.azurecr.io/${{ env.IMAGE_NAME }}:${{ github.sha }}
```

---

## Authentication — OIDC

Pipelines use **OpenID Connect (OIDC)** for passwordless Azure authentication — no stored secrets needed:

```yaml
permissions:
  id-token: write
  contents: read

- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
```

> OIDC is configured via an Azure AD App Registration with federated credentials for each GitHub repo.

---

## Azure Resource Naming

Resources follow the naming convention:

```
app-{service}-xshopai-{suffix}
```

Examples:

- `app-product-service-xshopai-eastus`
- `app-web-bff-xshopai-eastus`
- `acr-xshopai-eastus`

---

## Triggers

### Service Pipelines

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'Dockerfile'
      - 'package.json' # or requirements.txt, *.csproj, pom.xml
      - '.github/workflows/**'
  pull_request:
    branches: [main]
```

- **Push to main**: Full pipeline (build → test → docker → deploy)
- **Pull request**: Build and test only (no deploy)

### Infrastructure Pipelines

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'app-service/bicep/**'
  workflow_dispatch: # Manual trigger
```

---

## Multi-Stage Dockerfiles

All services use optimized multi-stage Docker builds:

### Node.js / TypeScript Pattern

```dockerfile
# Stage 1: Base
FROM node:20-alpine AS base
WORKDIR /app

# Stage 2: Dependencies
FROM base AS deps
COPY package*.json ./
RUN npm ci --only=production

# Stage 3: Build (TypeScript only)
FROM base AS build
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 4: Production
FROM base AS production
RUN addgroup -g 1001 -S nodejs && adduser -S appuser -u 1001
COPY --from=deps /app/node_modules ./node_modules
COPY --from=build /app/dist ./dist
USER appuser
EXPOSE 8080
HEALTHCHECK CMD wget -qO- http://localhost:8080/health || exit 1
CMD ["node", "dist/server.js"]
```

### Python Pattern

```dockerfile
FROM python:3.11-slim AS base
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
RUN useradd -r appuser && chown -R appuser /app
USER appuser
EXPOSE 8080
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
CMD ["python", "main.py"]
```

### .NET Pattern

```dockerfile
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY *.csproj ./
RUN dotnet restore
COPY . .
RUN dotnet publish -c Release -o /app/publish

FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS production
WORKDIR /app
COPY --from=build /app/publish .
RUN useradd -r appuser
USER appuser
EXPOSE 8080
HEALTHCHECK CMD curl -f http://localhost:8080/health || exit 1
ENTRYPOINT ["dotnet", "ServiceName.dll"]
```

### Java Pattern

```dockerfile
FROM maven:3.9-eclipse-temurin-17 AS build
WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

FROM eclipse-temurin:17-jre-alpine AS production
WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
RUN addgroup -S appgroup && adduser -S appuser -G appgroup
USER appuser
EXPOSE 8080
HEALTHCHECK CMD wget -qO- http://localhost:8080/actuator/health || exit 1
ENTRYPOINT ["java", "-jar", "app.jar"]
```

> **Note**: Production containers use port **8080** (Azure App Service convention). Local development uses the service-specific ports (8001–8014).

---

## Infrastructure Pipeline

The Bicep deployment pipeline (`deploy-app-service.yml`) has a 4-job structure:

```
┌────────────┐    ┌────────────┐    ┌────────────┐    ┌────────────┐
│  Validate  │───▶│  What-If   │───▶│  Approve   │───▶│   Deploy   │
│   Bicep    │    │  Preview   │    │  (manual)  │    │  (az cli)  │
└────────────┘    └────────────┘    └────────────┘    └────────────┘
```

1. **Validate** — `az deployment group validate` with parameter file
2. **What-If** — Preview changes without deploying
3. **Approve** — Manual approval gate (GitHub Environments)
4. **Deploy** — `az deployment group create` using OIDC auth

---

## Secrets & Variables

### Repository Secrets (per service repo)

| Secret                  | Purpose                         |
| :---------------------- | :------------------------------ |
| `AZURE_CLIENT_ID`       | OIDC app registration client ID |
| `AZURE_TENANT_ID`       | Azure AD tenant ID              |
| `AZURE_SUBSCRIPTION_ID` | Target subscription             |
| `ACR_LOGIN_SERVER`      | Container registry URL          |

### Organization-Level Variables

| Variable          | Purpose                       |
| :---------------- | :---------------------------- |
| `ACR_NAME`        | Azure Container Registry name |
| `RESOURCE_GROUP`  | Target resource group         |
| `APP_NAME_SUFFIX` | Environment-specific suffix   |

---

## Running Workflows Locally

Use [act](https://github.com/nektos/act) to test workflows locally:

```bash
# Install act
brew install act          # macOS
choco install act-cli     # Windows

# Run a workflow
cd product-service
act push -W .github/workflows/ci-app-service.yml
```
