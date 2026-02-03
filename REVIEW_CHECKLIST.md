# xshopai Microservice Review Checklist

> **Updated**: Based on inventory-service reference implementation  
> **Applies to**: Python (Flask), Node.js (Express), Java (Spring Boot), .NET (ASP.NET Core)

## Configuration Files

### .env.local (Local without Dapr)

- [ ] SERVICE_NAME, VERSION, PORT
- [ ] ENVIRONMENT=development (Python) or NODE_ENV=development (Node.js)
- [ ] DB_NAME or MONGODB_DB_NAME
- [ ] MESSAGING_PROVIDER=rabbitmq
- [ ] RABBITMQ_URL=amqp://admin:admin123@localhost:5672/
- [ ] LOG_LEVEL, LOG_FORMAT, LOG_TO_CONSOLE, LOG_TO_FILE, LOG_FILE_PATH
- [ ] Database connection string (direct connection)
- [ ] JWT_SECRET (for auth validation)
- [ ] ❌ No framework-specific secrets (FLASK_SECRET, etc.) - not needed for stateless REST APIs

### .env.dapr (Local with Dapr)

- [ ] SERVICE_NAME, VERSION, PORT
- [ ] ENVIRONMENT=development (Python) or NODE_ENV=development (Node.js)
- [ ] DB_NAME or MONGODB_DB_NAME
- [ ] MESSAGING_PROVIDER=dapr
- [ ] DAPR_HTTP_PORT=3500, DAPR_GRPC_PORT=50001
- [ ] LOG_LEVEL, LOG_FORMAT, LOG_TO_CONSOLE, LOG_TO_FILE=false
- [ ] ❌ NO secrets in this file (read from .dapr/secrets.json)

### .dapr/secrets.json (Local Dapr secrets)

- [ ] Valid JSON with key-value pairs only (flat structure)
- [ ] `{db}-{level}-connection` (e.g., `mysql-server-connection`)
- [ ] `jwt-secret`
- [ ] `appinsights-connection` (empty string for local)
- [ ] `service-{service}-token` for each dependency (e.g., `service-order-token`)
- [ ] ❌ No `xshopai-` prefix (use simple names)

### .dapr/components/ (Dapr component files)

- [ ] **event-bus.yaml**: Minimal config - connectionString + `exchangeKind: topic`
- [ ] **secret-store.yaml**: Minimal config - secretsFile path only
- [ ] **subscriptions.yaml**: pubsubname, topic, route per subscription
- [ ] ❌ No `config.yaml` (optional, delete if not needed)
- [ ] ❌ No `scopes` in subscriptions (handled by Dapr sidecar)
- [ ] ❌ No `deadLetterTopic` (adds complexity, skip for now)

### scripts/aca.sh (Azure Deployment)

- [ ] Header comment references `deploy.sh` (not deploy-infra.sh)
- [ ] Environment mapping: dev→development, prod→production
- [ ] ENVIRONMENT (Python) or NODE_ENV (Node.js)
- [ ] SERVICE_NAME, APP_PORT, DB_NAME
- [ ] MESSAGING_PROVIDER=dapr
- [ ] DAPR_HTTP_PORT=3500, DAPR_GRPC_PORT=50001
- [ ] OTEL_SERVICE_NAME, OTEL_RESOURCE_ATTRIBUTES
- [ ] APPLICATIONINSIGHTS_CONNECTION_STRING (from Key Vault)
- [ ] Database credentials as env vars (from Key Vault)
- [ ] LOG_LEVEL=info (prod) or LOG_LEVEL=debug (dev)

---

## Language-Specific Notes

### Python (Flask/FastAPI)

| Item            | Convention                                                                  |
| --------------- | --------------------------------------------------------------------------- |
| Config file     | `config.py`                                                                 |
| Environment var | Use `ENVIRONMENT`, not `FLASK_ENV` (deprecated in Flask 2.3+)               |
| Secret key      | Not needed for stateless APIs - set `SECRET_KEY = 'not-used-stateless-api'` |
| Dependencies    | `requirements.txt` or `pyproject.toml`                                      |
| Azure Monitor   | `azure-monitor-opentelemetry` package                                       |

### Node.js (Express)

| Item            | Convention                             |
| --------------- | -------------------------------------- |
| Config file     | `config.js` or `config/index.js`       |
| Environment var | Use `ENVIRONMENT` or `NODE_ENV`        |
| Dependencies    | `package.json`                         |
| Azure Monitor   | `@azure/monitor-opentelemetry` package |

### Java (Spring Boot)

| Item            | Convention                                                           |
| --------------- | -------------------------------------------------------------------- |
| Config file     | `application.yml` or `application.properties`                        |
| Environment var | Use `ENVIRONMENT` (Spring reads `SPRING_PROFILES_ACTIVE` separately) |
| Dependencies    | `pom.xml` (Maven) or `build.gradle` (Gradle)                         |
| Azure Monitor   | `applicationinsights-agent` JAR                                      |

### .NET (ASP.NET Core)

| Item            | Convention                                                      |
| --------------- | --------------------------------------------------------------- |
| Config file     | `appsettings.json`, `appsettings.{env}.json`                    |
| Environment var | Use `ENVIRONMENT` (maps to `ASPNETCORE_ENVIRONMENT` in startup) |
| Dependencies    | `.csproj` file                                                  |
| Azure Monitor   | `Azure.Monitor.OpenTelemetry.AspNetCore` package                |

---

## Naming Conventions

### Key Vault Secrets (lowercase, hyphens, NO xshopai- prefix)

| Secret Type   | Format                    | Example                   |
| ------------- | ------------------------- | ------------------------- |
| Database      | `{db}-{level}-connection` | `mysql-server-connection` |
| JWT           | `jwt-secret`              | `jwt-secret`              |
| App Insights  | `appinsights-connection`  | `appinsights-connection`  |
| Service Token | `service-{service}-token` | `service-order-token`     |

### Environment Variables (UPPER_SNAKE_CASE)

- [ ] NAME, PORT, ENVIRONMENT, LOG_LEVEL
- [ ] DB_NAME, DB_HOST, DB_PORT (if separate)
- [ ] MESSAGING_PROVIDER, DAPR_HTTP_PORT, DAPR_GRPC_PORT
- [ ] JWT_SECRET (injected from Key Vault)

### Variable Mappings

| Environment | APP_CONFIG  | ENVIRONMENT |
| ----------- | ----------- | ----------- |
| dev         | development | development |
| prod        | production  | production  |

---

## Validation Checklist

### Secrets & Configuration

- [ ] Same secrets exist in: `.dapr/secrets.json` and Key Vault
- [ ] Secret names match exactly (no `xshopai-` prefix anywhere)
- [ ] Service tokens consistent across all microservices
- [ ] No secrets in `.env.dapr`
- [ ] No framework-specific secrets (FLASK_SECRET, etc.)

### Secret Access Pattern (CRITICAL)

**Priority order in code must be: Environment variables FIRST, Dapr fallback for local dev only**

| Mode               | How It Works                                                                       |
| ------------------ | ---------------------------------------------------------------------------------- |
| Azure ACA          | `aca.sh` reads Key Vault at deploy time → injects as env vars → app reads env vars |
| Local with Dapr    | Env vars not set → falls back to Dapr → Dapr reads `.dapr/secrets.json`            |
| Local without Dapr | Developer sets env vars manually → app reads env vars                              |

- [ ] Code checks `System.getenv()` / `os.environ` / `process.env` FIRST
- [ ] Dapr secretstore is FALLBACK only (for local dev with Dapr sidecar)
- [ ] ❌ NO runtime Key Vault queries from application code
- [ ] `aca.sh` retrieves ALL secrets from Key Vault at deployment time
- [ ] Secrets injected as container env vars (not Dapr secretstore in Azure)

### Environment Variables

- [ ] ENVIRONMENT used consistently across all languages
- [ ] Framework-specific vars (FLASK_ENV, ASPNETCORE_ENVIRONMENT) derived from ENVIRONMENT if needed
- [ ] DB credentials passed as env vars in Azure (not Dapr secretstore)
- [ ] App Insights connection as env var (avoids Dapr race condition)

### Messaging

- [ ] Local without Dapr: MESSAGING_PROVIDER=rabbitmq + RABBITMQ_URL
- [ ] Local with Dapr: MESSAGING_PROVIDER=dapr, event-bus.yaml has `exchangeKind: topic`
- [ ] Azure: MESSAGING_PROVIDER=dapr (uses Service Bus via Dapr)
- [ ] Subscription topics match publisher topics exactly

### Observability

- [ ] Azure Monitor SDK in dependencies (see language-specific section)
- [ ] OpenTelemetry configured at startup
- [ ] OTEL_SERVICE_NAME, OTEL_RESOURCE_ATTRIBUTES set in aca.sh
- [ ] `appinsights-connection` in secrets.json (empty for local)

### Dapr Components (Minimal Config)

- [ ] event-bus.yaml: Only connectionString + exchangeKind
- [ ] secret-store.yaml: Only secretsFile path
- [ ] subscriptions.yaml: No scopes, no deadLetterTopic
- [ ] config.yaml: Delete if not needed (tracing config optional)

### Deployment Script (aca.sh)

- [ ] Prerequisite comment: `deploy.sh` (not deploy-infra.sh)
- [ ] Uses APP_CONFIG variable for environment mapping
- [ ] All secrets retrieved from Key Vault with new naming
- [ ] ENV_VARS array has ENVIRONMENT
- [ ] Health check verification at end
