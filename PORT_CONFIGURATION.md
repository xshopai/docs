# xshopai Platform - Port Configuration

This document defines the standardized port configuration for all services in the xshopai platform.

## Port Allocation Pattern

| Port Type        | Pattern | Example                 |
| ---------------- | ------- | ----------------------- |
| Application Port | `800X`  | 8001, 8002, ... 8014    |
| Dapr HTTP Port   | `350X`  | 3501, 3502, ... 3514    |
| Dapr gRPC Port   | `5000X` | 50001, 50002, ... 50014 |

## Service Port Configuration

| #   | Service                 | App Port | Dapr HTTP | Dapr gRPC | Technology       |
| --- | ----------------------- | -------- | --------- | --------- | ---------------- |
| 1   | product-service         | 8001     | 3501      | 50001     | Python/FastAPI   |
| 2   | user-service            | 8002     | 3502      | 50002     | Node.js/Express  |
| 3   | admin-service           | 8003     | 3503      | 50003     | Node.js/Express  |
| 4   | auth-service            | 8004     | 3504      | 50004     | Node.js/Express  |
| 5   | inventory-service       | 8005     | 3505      | 50005     | Python/FastAPI   |
| 6   | order-service           | 8006     | 3506      | 50006     | .NET/C#          |
| 7   | order-processor-service | 8007     | 3507      | 50007     | Java/Spring Boot |
| 8   | cart-service            | 8008     | 3508      | 50008     | Java/Spring Boot |
| 9   | payment-service         | 8009     | 3509      | 50009     | .NET/C#          |
| 10  | review-service          | 8010     | 3510      | 50010     | Node.js/Express  |
| 11  | notification-service    | 8011     | 3511      | 50011     | Node.js/Express  |
| 12  | audit-service           | 8012     | 3512      | 50012     | Node.js/Express  |
| 13  | chat-service            | 8013     | 3513      | 50013     | Node.js/Express  |
| 14  | web-bff                 | 8014     | 3514      | 50014     | Node.js/Express  |

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
PORT=800X                    # Service-specific port

# Dapr
DAPR_HTTP_PORT=350X          # Dapr HTTP sidecar port
DAPR_GRPC_PORT=5000X         # Dapr gRPC sidecar port
DAPR_APP_ID=<service-name>   # Unique Dapr application ID
```

## Local Development

When running services locally with Dapr, use the following command pattern:

```bash
dapr run \
  --app-id <service-name> \
  --app-port 800X \
  --dapr-http-port 350X \
  --dapr-grpc-port 5000X \
  --resources-path .dapr/components \
  --config .dapr/config.yaml \
  --log-level warn
```

## Azure Container Apps Deployment

> **Important:** In Azure Container Apps, the Dapr sidecar **always** runs on port **3500** (HTTP) and **50001** (gRPC), regardless of the service. This is different from local development where each service has unique ports.

| Environment          | Dapr HTTP Port            | Dapr gRPC Port       | Reason                                                         |
| -------------------- | ------------------------- | -------------------- | -------------------------------------------------------------- |
| Local Dev            | 350X (unique per service) | 5000X (unique)       | All services on same machine, need unique ports                |
| Azure Container Apps | 3500 (same for all)       | 50001 (same for all) | Each app is isolated, Dapr sidecar is always at localhost:3500 |

When deploying to ACA, set:

```bash
DAPR_HTTP_PORT=3500
DAPR_GRPC_PORT=50001
```

## Quick Reference

```
Service                    App     Dapr HTTP   Dapr gRPC
─────────────────────────────────────────────────────────
product-service           8001      3501        50001
user-service              8002      3502        50002
admin-service             8003      3503        50003
auth-service              8004      3504        50004
inventory-service         8005      3505        50005
order-service             8006      3506        50006
order-processor-service   8007      3507        50007
cart-service              8008      3508        50008
payment-service           8009      3509        50009
review-service            8010      3510        50010
notification-service      8011      3511        50011
audit-service             8012      3512        50012
chat-service              8013      3513        50013
web-bff                   8014      3514        50014
─────────────────────────────────────────────────────────
customer-ui               3000       -            -
admin-ui                  3001       -            -
```

---

_Last updated: January 27, 2026_
