# xshopai Microservice Review Checklist

## Configuration Files

### .env.local (Local without Dapr)

- [ ] SERVICE_NAME/NAME, VERSION, PORT, ENVIRONMENT/FLASK_ENV
- [ ] DB_NAME or MONGODB_DB_NAME
- [ ] MESSAGING_PROVIDER=rabbitmq
- [ ] RABBITMQ_URL=amqp://admin:admin123@localhost:5672/
- [ ] LOG_LEVEL, LOG_FORMAT, LOG_TO_CONSOLE, LOG_TO_FILE, LOG_FILE_PATH
- [ ] Database connection string (direct or via hyphenated key)
- [ ] All `xshopai-*` secrets (hyphenated naming)

### .env.dapr (Local with Dapr)

- [ ] SERVICE_NAME/NAME, VERSION, PORT, ENVIRONMENT/FLASK_ENV
- [ ] DB_NAME or MONGODB_DB_NAME
- [ ] MESSAGING_PROVIDER=dapr
- [ ] DAPR_HTTP_PORT=3500, DAPR_GRPC_PORT=50001
- [ ] LOG_LEVEL, LOG_FORMAT, LOG_TO_CONSOLE, LOG_TO_FILE=false
- [ ] NO secrets in this file

### .dapr/secrets.json

- [ ] Valid JSON with key-value pairs only
- [ ] `xshopai-{db}-{level}-connection` (database)
- [ ] `xshopai-jwt-secret`
- [ ] `xshopai-flask-secret` (Flask only)
- [ ] `xshopai-appinsights-connection` (empty for local)
- [ ] `xshopai-svc-{service}-token` for each dependency

### aca.sh (Azure Deployment)

- [ ] ENVIRONMENT=production / FLASK_ENV=production
- [ ] NAME/SERVICE_NAME, PORT, DB_NAME
- [ ] MESSAGING_PROVIDER=dapr
- [ ] DAPR_HTTP_PORT=3500, DAPR_GRPC_PORT=50001
- [ ] OTEL_SERVICE_NAME, OTEL_RESOURCE_ATTRIBUTES
- [ ] APPLICATIONINSIGHTS_CONNECTION_STRING (from Key Vault)
- [ ] Database connection (from Key Vault, if needed at startup)
- [ ] LOG_LEVEL=info

---

## Naming Conventions

### Secrets (lowercase, hyphens, xshopai- prefix)

- [ ] Database: `xshopai-{db}-{level}-connection` (e.g. `xshopai-mysql-server-connection`)
- [ ] JWT: `xshopai-jwt-secret`
- [ ] Flask: `xshopai-flask-secret`
- [ ] App Insights: `xshopai-appinsights-connection`
- [ ] Service Token: `xshopai-svc-{service}-token` (e.g. `xshopai-svc-order-token`)

### Config Variables (UPPER_CASE, underscores)

- [ ] SERVICE_NAME, PORT, ENVIRONMENT, LOG_LEVEL, etc.

---

## Validation Checklist

### Secrets & Configuration

- [ ] Same secrets exist in: `.env.local`, `.dapr/secrets.json`, Key Vault
- [ ] Secret names match exactly across all environments
- [ ] Service tokens consistent across all microservices
- [ ] No secrets in `.env.dapr`

### Messaging

- [ ] Local: MESSAGING_PROVIDER=rabbitmq + RABBITMQ_URL
- [ ] Dapr: MESSAGING_PROVIDER=dapr, no RabbitMQ config
- [ ] Subscription topics match publisher topics exactly

### Observability

- [ ] Azure Monitor SDK in requirements.txt/package.json
- [ ] `configure_azure_monitor()` called at startup
- [ ] `xshopai-appinsights-connection` in secrets.json (empty for local)
- [ ] OTEL_SERVICE_NAME set in aca.sh

### Dapr Components

- [ ] All required components configured
- [ ] No unused/duplicate component files
- [ ] Subscription topics match exactly

### Deployment

- [ ] Database connection: env var if needed at startup, else Dapr secretstore
- [ ] App Insights connection: always env var (Dapr race condition)
- [ ] Health/readiness endpoints accessible
- [ ] Event pub/sub working

---

## Output Format

### Discrepancies

- ❌ File path, variable name, expected vs actual

### Risks

- ⚠️ Inconsistencies or potential issues

### Compliant

- ✅ Verified and working
