# Troubleshooting

Common issues and their solutions when developing or deploying xshopai.

---

## Docker & Infrastructure

### Docker Compose fails to start

**Symptom**: `docker compose up -d` errors or containers exit immediately.

**Fixes**:

- Allocate at least **8 GB memory** to Docker Desktop (Settings → Resources).
- On Windows, ensure WSL 2 backend is enabled.
- SQL Server containers require `ACCEPT_EULA=Y` — this is set in the compose file.
- If a container keeps restarting, check logs: `docker compose logs <service-name>`.

### Port already in use

**Symptom**: `Error: listen EADDRINUSE: address already in use :::8001`

**Fixes**:

```bash
# Find the process using the port (Windows)
netstat -ano | findstr :8001
taskkill /PID <pid> /F

# Find the process using the port (macOS/Linux)
lsof -i :8001
kill -9 <pid>
```

Full port map: see [Port Configuration](../PORT_CONFIGURATION.md).

### SQL Server container won't start on Apple Silicon

**Symptom**: SQL Server image fails on ARM64 (M1/M2 Macs).

**Fix**: Use the Azure SQL Edge image instead:

```yaml
image: mcr.microsoft.com/azure-sql-edge:latest
```

The `dev/docker-compose.yml` already handles this — ensure you're using the latest version.

### Docker running out of disk space

```bash
# Remove unused images, containers, and volumes
docker system prune -a --volumes
```

---

## Database Connectivity

### MongoDB connection refused

**Symptom**: `MongoNetworkError: connect ECONNREFUSED 127.0.0.1:27018`

**Fixes**:

1. Verify the container is running: `docker ps | grep mongo`
2. Check the correct port — each MongoDB instance uses a different port:
   - user-service: **27018**
   - product-service: **27019**
   - review-service: **27020**
3. Ensure the connection string includes `?authSource=admin`
4. Default credentials: `admin` / `admin123`

### MySQL access denied

**Symptom**: `Access denied for user 'admin'@'localhost'`

**Fixes**:

1. Verify MySQL container is healthy: `docker compose logs mysql`
2. Default credentials: `admin` / `admin123`
3. Database name: `inventory_service_db`
4. MySQL may need 10-15 seconds after container start for initialization.

### SQL Server login failed

**Symptom**: `Login failed for user 'sa'`

**Fixes**:

1. Default credentials: `sa` / `Admin123!`
2. Verify the correct port:
   - payment-service: **1433**
   - order-service: **1434**
3. Ensure `TrustServerCertificate=True` in connection string
4. SQL Server needs 15-30 seconds to initialize after container start.

### PostgreSQL connection refused

**Symptom**: `ECONNREFUSED` or `Connection refused` on PostgreSQL port

**Fixes**:

1. Default credentials: `postgres` / `postgres`
2. Port mapping:
   - audit-service: **5434**
   - order-processor-service: **5435**
3. Check container: `docker compose logs postgres-audit` or `postgres-order-processor`

### Database migrations fail

**Fixes by technology**:

```bash
# Flask-Migrate (inventory-service)
cd inventory-service
flask db upgrade

# EF Core (.NET services)
cd order-service
dotnet ef database update --project OrderService.Api

# Flyway (order-processor-service — auto-runs on startup)
cd order-processor-service
mvn spring-boot:run   # Flyway runs automatically

# Knex.js (audit-service)
cd audit-service
npx knex migrate:latest
```

---

## Dapr Issues

### Dapr sidecar not starting

**Symptom**: `dapr run` hangs or fails.

**Fixes**:

1. Verify Dapr is installed: `dapr --version`
2. Initialize Dapr: `dapr init`
3. Check Docker is running (Dapr init requires Docker).
4. Verify component files exist: `ls .dapr/components/`

### `ERR_PUBSUB_NOT_FOUND`

**Symptom**: Publishing events fails with component not found.

**Fixes**:

1. Verify `--resources-path` points to a directory containing `pubsub.yaml`.
2. Check the component name matches: `xshopai-pubsub`.
3. Ensure RabbitMQ is running: `docker ps | grep rabbitmq`.
4. Verify the Dapr `--config` flag points to a valid `config.yaml`.

### Events not being received by subscribers

**Fixes**:

1. Verify the subscriber exposes `GET /dapr/subscribe` endpoint.
2. Ensure the subscriber's Dapr sidecar is running.
3. Check RabbitMQ management UI at http://localhost:15672 (admin/admin123).
4. Look for unacked messages in the queue.
5. Verify the topic name matches exactly between publisher and subscriber.

### Dapr sidecar port conflicts

**Symptom**: `Error: listen tcp 127.0.0.1:3500: bind: address already in use`

**Fix**: When running multiple services, use unique Dapr ports:

```bash
dapr run --app-id product-service --dapr-http-port 3501 --dapr-grpc-port 50001 ...
dapr run --app-id user-service --dapr-http-port 3502 --dapr-grpc-port 50002 ...
```

See the [Dapr Integration Guide](Dapr-Integration-Guide.md) for the full port table.

---

## Authentication & JWT

### `401 Unauthorized` on all requests

**Fixes**:

1. Ensure `JWT_SECRET` environment variable is set and matches across all services.
2. Default dev secret is defined in each service's config — verify it's consistent.
3. Check token expiry: access tokens default to 1 hour.
4. For web-bff: ensure cookies are being passed (check `withCredentials: true` in Axios).

### `403 Forbidden` on admin endpoints

**Fixes**:

1. The JWT must contain `"roles": ["admin"]` in the payload.
2. Use the admin seed account: `admin@xshopai.com` / `admin` (after running db-seeder).
3. Verify the admin role check middleware is using the correct claim path.

### JWT token expired

**Fixes**:

1. Log in again to get a fresh token.
2. Check refresh token flow — `POST /api/v1/auth/refresh`.
3. Default expiry: access = 1h, refresh = 7d.

---

## Service-Specific Issues

### web-bff returning 502 / connection refused

**Symptom**: API calls through the BFF fail even though services are running.

**Fixes**:

1. Check `PLATFORM_MODE`:
   - `direct` — services must be running on their native ports
   - `dapr` — Dapr sidecars must be running for each service
2. Verify `ALLOWED_ORIGINS` includes the frontend URL.
3. Check individual service health: `curl http://localhost:8001/health`

### order-processor-service saga stuck

**Symptom**: Orders stay in `PENDING_PAYMENT_CONFIRMATION` state.

**Fix**: This is by design — the Amazon admin portal pattern requires manual payment confirmation:

1. Log into admin-ui (port 3001).
2. Navigate to Order Processing.
3. Confirm the payment manually.

### cart-service state not persisting

**Fixes**:

1. Check `PLATFORM_MODE`:
   - `dapr` → requires Dapr statestore component and Redis
   - `direct` → requires direct Redis connection
2. Verify Redis is running: `docker ps | grep redis`
3. Check Redis password: `redis_dev_pass_123`

### notification-service emails not sending

**Fixes**:

1. Verify Mailpit is running for local dev: http://localhost:8025
2. Check SMTP config: host=`localhost`, port=`1025`
3. Ensure the notification-service is subscribed to events (check Dapr subscription).

### inventory-service SSL errors (Azure)

**Symptom**: `SSL: CERTIFICATE_REQUIRED` when connecting to Azure MySQL.

**Fix**: The service strips `ssl_mode` from the URL and uses `connect_args` instead. Check the SQLAlchemy configuration in `src/__init__.py`.

---

## Frontend Issues

### customer-ui / admin-ui not loading

**Fixes**:

1. Verify `REACT_APP_API_URL` points to the BFF: `http://localhost:8014`
2. Check that the BFF is running and healthy.
3. Clear browser cache and cookies.
4. Check browser console for CORS errors.

### CORS errors in browser

**Symptom**: `Access-Control-Allow-Origin` errors in the browser console.

**Fixes**:

1. Set `ALLOWED_ORIGINS` in web-bff to include the frontend URLs:
   ```
   ALLOWED_ORIGINS=http://localhost:3000,http://localhost:3001
   ```
2. Ensure the BFF's CORS middleware is loaded before routes.
3. For credentials (cookies), the origin must be explicit (not `*`).

---

## Build Issues

### TypeScript compilation errors

```bash
# Check types without emitting
npm run type-check

# Clean and rebuild
rm -rf dist/ && npm run build
```

### Python dependency conflicts

```bash
# Create a fresh virtual environment
python -m venv venv
source venv/bin/activate       # macOS/Linux
venv\Scripts\activate          # Windows
pip install -r requirements.txt
```

### .NET restore failures

```bash
# Clear NuGet cache
dotnet nuget locals all --clear
dotnet restore
```

### Java/Maven build failures

```bash
# Force dependency refresh
mvn clean install -U

# Skip tests if they're blocking the build
mvn clean package -DskipTests
```

---

## Seeding Issues

### db-seeder fails partially

**Symptom**: `seed.py` succeeds for some databases but fails for others.

**Fixes**:

1. Ensure all database containers are healthy (wait 30 seconds after `docker compose up`).
2. If a specific DB driver is missing, install it:
   ```bash
   pip install pymongo pymysql psycopg2-binary pymssql bcrypt
   ```
3. Run the seeder for specific data only:
   ```bash
   python seed.py --users      # Users only
   python seed.py --products   # Products only
   python seed.py --inventory  # Inventory only
   ```
4. For a clean slate: `python seed.py --clear` then `python seed.py`.

---

## Logging & Debugging

### Enable verbose logging

| Service Type | Setting                                             |
| :----------- | :-------------------------------------------------- |
| Node.js / TS | `LOG_LEVEL=debug` env var                           |
| Python       | `LOG_LEVEL=DEBUG` env var                           |
| .NET         | `Serilog:MinimumLevel:Default=Debug` in appsettings |
| Java         | `logging.level.root=DEBUG` in application.yml       |
| Dapr sidecar | `--log-level debug` flag                            |

### View distributed traces

1. Ensure Zipkin is running: http://localhost:9411
2. Each service propagates W3C Trace Context via `traceparent` header.
3. Search by trace ID or service name in Zipkin UI.

### View RabbitMQ queues

1. Open RabbitMQ Management: http://localhost:15672
2. Login: `admin` / `admin123`
3. Check Queues tab for message backlogs or unacked messages.

### View emails (local dev)

1. Open Mailpit: http://localhost:8025
2. All emails sent by notification-service appear here.
