# xshopai Platform - Port Configuration

This document defines the standardized port configuration for all services in the xshopai platform.

## Service Port Configuration

> **Note:** Dapr HTTP port is **3500** and gRPC port is **50001** for **ALL services**, both locally and in Azure Container Apps. This ensures consistent configuration across environments.

| #   | Service                 | App Port | Technology       | Dapr HTTP | Dapr gRPC |
| --- | ----------------------- | -------- | ---------------- | --------- | --------- |
| 1   | product-service         | 8001     | Python/FastAPI   | 3500      | 50001     |
| 2   | user-service            | 8002     | Node.js/Express  | 3500      | 50001     |
| 3   | admin-service           | 8003     | Node.js/Express  | 3500      | 50001     |
| 4   | auth-service            | 8004     | Node.js/Express  | 3500      | 50001     |
| 5   | inventory-service       | 8005     | Python/FastAPI   | 3500      | 50001     |
| 6   | order-service           | 8006     | .NET/C#          | 3500      | 50001     |
| 7   | order-processor-service | 8007     | Java/Spring Boot | 3500      | 50001     |
| 8   | cart-service            | 8008     | Java/Spring Boot | 3500      | 50001     |
| 9   | payment-service         | 8009     | .NET/C#          | 3500      | 50001     |
| 10  | review-service          | 8010     | Node.js/Express  | 3500      | 50001     |
| 11  | notification-service    | 8011     | Node.js/Express  | 3500      | 50001     |
| 12  | audit-service           | 8012     | Node.js/Express  | 3500      | 50001     |
| 13  | chat-service            | 8013     | Node.js/Express  | 3500      | 50001     |
| 14  | web-bff                 | 8014     | Node.js/Express  | 3500      | 50001     |

## Frontend Applications (No Dapr)

| Service     | Port | Technology |
| ----------- | ---- | ---------- |
| customer-ui | 3000 | React      |
| admin-ui    | 3001 | React      |

## Infrastructure Services

| Service             | Port  | Purpose             |
| ------------------- | ----- | ------------------- |
| MongoDB             | 27017 | Database            |
| RabbitMQ            | 5672  | Message Broker      |
| RabbitMQ Management | 15672 | Management UI       |
| Redis               | 6379  | Caching             |
| Zipkin              | 9411  | Distributed Tracing |

## Environment Variables

Each service should use these environment variables for configuration:

```bash
# Application
PORT=800X                    # Service-specific port (8001-8014)

# Dapr (same for ALL services)
DAPR_HTTP_PORT=3500          # Dapr HTTP sidecar port
DAPR_GRPC_PORT=50001         # Dapr gRPC sidecar port
DAPR_APP_ID=<service-name>   # Unique Dapr application ID
```

## Local Development

### Running a Single Service

When running ONE service locally with Dapr:

```bash
dapr run \
  --app-id <service-name> \
  --app-port 800X \
  --dapr-http-port 3500 \
  --dapr-grpc-port 50001 \
  --resources-path .dapr/components \
  --config .dapr/config.yaml \
  --log-level warn
```

### Running Multiple Services Simultaneously

When running **multiple services on the same machine**, use **Docker Compose** to isolate each service in its own container. This way each service gets its own Dapr sidecar at `localhost:3500` without port conflicts.

```bash
# From scripts/docker-compose directory
docker-compose -f docker-compose.services.yml up
```

> **Why Docker Compose?** Each container is network-isolated, so all services can use port 3500 simultaneously, just like in Azure Container Apps.

## Azure Container Apps Deployment

In Azure Container Apps, each service runs in its own isolated container with Dapr sidecar at:

- **HTTP**: `localhost:3500`
- **gRPC**: `localhost:50001`

This matches the local Docker Compose setup, ensuring zero configuration changes between environments.

## Quick Reference

```
Service                    App Port    Dapr HTTP   Dapr gRPC
───────────────────────────────────────────────────────────
product-service           8001         3500        50001
user-service              8002         3500        50001
admin-service             8003         3500        50001
auth-service              8004         3500        50001
inventory-service         8005         3500        50001
order-service             8006         3500        50001
order-processor-service   8007         3500        50001
cart-service              8008         3500        50001
payment-service           8009         3500        50001
review-service            8010         3500        50001
notification-service      8011         3500        50001
audit-service             8012         3500        50001
chat-service              8013         3500        50001
web-bff                   8014         3500        50001
───────────────────────────────────────────────────────────
customer-ui               3000          -            -
admin-ui                  3001          -            -
```
