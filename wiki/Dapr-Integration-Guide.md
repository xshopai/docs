# Dapr Integration Guide

[Dapr (Distributed Application Runtime)](https://dapr.io) provides the building blocks for microservice communication in xshopai — **pub/sub messaging**, **state management**, **service invocation**, and **distributed tracing**.

---

## Dapr Components Used

| Building Block    | Component            | Backend  | Purpose                                 |
| :---------------- | :------------------- | :------- | :-------------------------------------- |
| **Pub/Sub**       | `xshopai-pubsub`     | RabbitMQ | Event-driven messaging between services |
| **State Store**   | `xshopai-statestore` | Redis    | Shopping cart state (cart-service)      |
| **Configuration** | `xshopai-config`     | —        | Tracing configuration (Zipkin)          |

---

## Port Configuration

### Isolated Mode (single service with Dapr)

When running one service at a time, all services use the same Dapr ports:

| Setting        | Value |
| :------------- | :---- |
| Dapr HTTP Port | 3500  |
| Dapr gRPC Port | 50001 |

### Multi-Service Mode (all services running simultaneously)

When running multiple services on the same machine, each needs unique Dapr ports:

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

## Running a Service with Dapr

### Prerequisites

1. Install the Dapr CLI: [docs.dapr.io/getting-started](https://docs.dapr.io/getting-started/install-dapr-cli/)
2. Initialize Dapr: `dapr init`
3. Start infrastructure: `cd dev && docker compose up -d`

### Start a Service with Dapr Sidecar

```bash
cd product-service

dapr run \
  --app-id product-service \
  --app-port 8001 \
  --dapr-http-port 3500 \
  --dapr-grpc-port 50001 \
  --resources-path .dapr/components \
  --config .dapr/config.yaml \
  --log-level warn
```

Then start the service in a separate terminal:

```bash
python main.py
```

Or use each service's VS Code task (`.vscode/tasks.json`):

- "Start Dapr Sidecar" / "dapr-start" tasks are preconfigured per service.

---

## Component Configuration

Each service has its own `.dapr/` directory with component definitions.

### Pub/Sub Component

```yaml
# .dapr/components/pubsub.yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: xshopai-pubsub
spec:
  type: pubsub.rabbitmq
  version: v1
  metadata:
    - name: connectionString
      value: 'amqp://admin:admin123@127.0.0.1:5672'
    - name: exchangeName
      value: 'xshopai.events'
    - name: exchangeKind
      value: 'topic'
    - name: durable
      value: 'true'
    - name: autoAck
      value: 'false'
    - name: deliveryMode
      value: '2'
    - name: requeueInFailure
      value: 'true'
    - name: prefetchCount
      value: '10'
```

### State Store Component (cart-service)

```yaml
# .dapr/components/statestore.yaml
apiVersion: dapr.io/v1alpha1
kind: Component
metadata:
  name: xshopai-statestore
spec:
  type: state.redis
  version: v1
  metadata:
    - name: redisHost
      value: 'localhost:6379'
    - name: redisPassword
      value: 'redis_dev_pass_123'
    - name: actorStateStore
      value: 'false'
```

### Tracing Configuration

```yaml
# .dapr/config.yaml
apiVersion: dapr.io/v1alpha1
kind: Configuration
metadata:
  name: xshopai-config
spec:
  tracing:
    samplingRate: '1'
    zipkin:
      endpointAddress: 'http://localhost:9411/api/v2/spans'
```

---

## Pub/Sub Patterns

### Publishing Events

All services manually construct CloudEvents 1.0 payloads:

**Node.js / TypeScript:**

```javascript
const cloudEvent = {
  specversion: '1.0',
  type: 'com.xshopai.user.created',
  source: 'user-service',
  id: `${Date.now()}-${Math.random().toString(36).substr(2, 9)}`,
  time: new Date().toISOString(),
  datacontenttype: 'application/json',
  data: { userId, email, roles },
  traceparent: traceparent,
  correlationId: correlationId,
};
await messagingProvider.publishEvent('user.created', cloudEvent);
```

**Python:**

```python
cloud_event = {
    "specversion": "1.0",
    "type": "inventory.stock.updated",
    "source": "inventory-service",
    "id": str(uuid.uuid4()),
    "time": datetime.now(timezone.utc).isoformat(),
    "datacontenttype": "application/json",
    "data": {"sku": sku, "quantity": quantity}
}
messaging_provider.publish_event("inventory.stock.updated", cloud_event)
```

**C# / .NET:**

```csharp
var cloudEvent = new {
    specversion = "1.0",
    type = "order.created",
    source = "order-service",
    id = Guid.NewGuid().ToString(),
    time = DateTime.UtcNow.ToString("o"),
    datacontenttype = "application/json",
    data = new { orderId = order.Id, customerId = order.CustomerId }
};
await messagingProvider.PublishEventAsync("order.created", cloudEvent);
```

### Subscribing to Events

**Node.js / TypeScript (Express route):**

```javascript
router.post('/dapr/events/order.created', async (req, res) => {
  const cloudEvent = req.body;
  const data = cloudEvent.data;
  // Process event...
  res.sendStatus(200);
});
```

**Python (Flask route):**

```python
@app.route('/dapr/events/order.created', methods=['POST'])
def handle_order_created():
    event = request.get_json()
    data = event.get('data', {})
    # Process event...
    return '', 200
```

**Java (Spring Boot controller):**

```java
@PostMapping("/dapr/events/order-created")
public ResponseEntity<?> handleOrderCreated(@RequestBody CloudEvent<OrderCreatedEvent> event) {
    OrderCreatedEvent data = event.getData();
    // Process event...
    return ResponseEntity.ok().build();
}
```

---

## Service Invocation

Some services call others via Dapr service invocation instead of direct HTTP:

| Caller        | Target          | Usage                              |
| :------------ | :-------------- | :--------------------------------- |
| auth-service  | user-service    | Validate credentials, create users |
| admin-service | user-service    | Administrative user management     |
| chat-service  | product-service | AI product search                  |
| chat-service  | order-service   | AI order tracking                  |

**Dapr Service Invocation URL Pattern:**

```
http://localhost:{dapr-port}/v1.0/invoke/{app-id}/method/{endpoint}
```

**Example:**

```
http://localhost:3500/v1.0/invoke/user-service/method/api/v1/users/lookup
```

---

## Messaging Abstraction

Services use a messaging factory pattern that supports both Dapr and direct RabbitMQ:

```
PLATFORM_MODE=dapr     → Uses Dapr SDK for pub/sub
PLATFORM_MODE=direct   → Uses direct RabbitMQ connection (AMQP)
```

This allows the same codebase to run:

- Locally without Dapr (direct mode)
- With Dapr sidecar (dapr mode)
- On Azure Container Apps with native Dapr integration

---

## Troubleshooting

| Issue                        | Cause                                | Fix                                                                             |
| :--------------------------- | :----------------------------------- | :------------------------------------------------------------------------------ |
| `ERR_PUBSUB_NOT_FOUND`       | Pub/sub component not loaded         | Check `.dapr/components/pubsub.yaml` exists and `--resources-path` points to it |
| Events not received          | Subscription endpoint not registered | Verify `/dapr/subscribe` returns subscriptions (GET request)                    |
| `Connection refused` on 3500 | Dapr sidecar not running             | Start Dapr with `dapr run` before the service                                   |
| Empty `{}` in consumer       | Publisher using `rawPayload`         | Remove `rawPayload` metadata, send full CloudEvent envelope                     |
| Double-wrapped events        | Manual CloudEvent + Dapr auto-wrap   | Send business data only OR disable Dapr auto-wrapping                           |
