# CloudEvents Standardization Across xshopai Services

## Architecture Overview

xshopai uses a **messaging abstraction layer** that supports multiple providers:

- **Dapr** (with RabbitMQ, Azure Service Bus, Kafka, etc.)
- **RabbitMQ** (direct connection, no Dapr)
- **Azure Service Bus** (direct connection, no Dapr)

This requires a **provider-agnostic** approach to CloudEvents construction.

## ✅ Standard Pattern (Use This)

### Publishing Events

**Always manually construct CloudEvents 1.0 compliant payloads:**

```javascript
// Node.js/TypeScript - Example from review-service, user-service
const cloudEvent = {
  specversion: '1.0',
  type: 'com.xshopai.review.created',        // Namespaced event type
  source: 'review-service',                   // Service name
  id: `${Date.now()}-${Math.random()...}`,   // Unique event ID
  time: new Date().toISOString(),            // ISO 8601 timestamp
  datacontenttype: 'application/json',
  data: {                                     // Business payload
    reviewId: review._id.toString(),
    productId: review.productId,
    rating: review.rating,
    // ... more fields
  },
  traceparent: traceparent,                   // W3C Trace Context (optional)
  metadata: {                                 // Service-specific metadata
    userId: userId,
    causationId: orderReference
  }
};

// Pass to messaging abstraction layer
await messagingProvider.publishEvent(topic, cloudEvent, traceId);
```

```python
# Python - Example from inventory-service, product-service
cloud_event = {
    "specversion": "1.0",
    "type": "inventory.stock.updated",
    "source": "inventory-service",
    "id": str(uuid.uuid4()),
    "time": datetime.now(timezone.utc).isoformat(),
    "datacontenttype": "application/json",
    "data": {
        "sku": sku,
        "availableQuantity": available_qty,
        # ... more fields
    },
    "correlationId": correlation_id
}

# Pass to messaging abstraction layer
messaging_provider.publish_event(topic, cloud_event, correlation_id)
```

```csharp
// C# - Example from order-service, payment-service
var cloudEvent = new
{
    specversion = "1.0",
    type = "order.created",
    source = "order-service",
    id = Guid.NewGuid().ToString(),
    time = DateTime.UtcNow.ToString("o"),
    datacontenttype = "application/json",
    data = new
    {
        orderId = order.Id,
        customerId = order.CustomerId,
        // ... more fields
    },
    correlationId = correlationId
};

// Pass to messaging abstraction layer
await messagingProvider.PublishEventAsync(topic, cloudEvent, correlationId);
```

### Consuming Events

**Services receive CloudEvents - extract data from `data` field:**

```javascript
// Node.js/TypeScript
const cloudEvent = req.body;
const eventData = cloudEvent.data;
const traceId = cloudEvent.traceparent?.split('-')[1];
```

```python
# Python
event = await request.json()
data = event.get("data", {})
correlation_id = event.get("correlationId")
```

```csharp
// C#
var cloudEvent = await req.Content.ReadAsAsync<CloudEvent>();
var data = cloudEvent.Data;
var correlationId = cloudEvent.CorrelationId;
```

## ❌ Anti-Patterns (Don't Do This)

### Don't Send Raw Business Data

```javascript
// ❌ WRONG - Not CloudEvents compliant, won't work with all providers
await messagingProvider.publishEvent(topic, {
  orderId: '123',
  status: 'placed',
});
```

### Don't Rely on Provider Auto-Wrapping

```javascript
// ❌ WRONG - Breaks when switching between Dapr/RabbitMQ/Service Bus
// Only works with Dapr SDK, not provider-agnostic
await daprClient.pubsub.publish(pubsubName, topic, eventData);
```

### Don't Use Provider-Specific Metadata

```javascript
// ❌ WRONG - Dapr-specific, breaks with RabbitMQ direct connection
const publishOptions = {
  metadata: { rawPayload: 'true' },
};
```

## Why This Standard?

1. **Provider Agnostic**: Works with Dapr, RabbitMQ, Azure Service Bus, or any future provider
2. **Explicit Control**: Full control over CloudEvents structure and metadata
3. **W3C Trace Context**: Consistent distributed tracing across all providers
4. **No Provider Lock-In**: Easy to switch messaging backends without code changes
5. **CloudEvents 1.0 Compliant**: Follows industry standard specification
6. **Consistent Event Format**: Same structure regardless of transport mechanism

## Messaging Abstraction Layer

Each service uses a messaging factory that selects the provider based on `MESSAGING_PROVIDER` environment variable:

```javascript
// Node.js/TypeScript - src/messaging/index.js
export async function getMessagingProvider() {
  const provider = process.env.MESSAGING_PROVIDER || 'dapr';

  switch (provider) {
    case 'dapr':
      return new DaprProvider();
    case 'rabbitmq':
      return new RabbitMQProvider();
    case 'servicebus':
      return new ServiceBusProvider();
    default:
      throw new Error(`Unknown provider: ${provider}`);
  }
}
```

```python
# Python - src/messaging/__init__.py
def create_messaging_provider() -> MessagingProvider:
    provider_type = os.getenv('MESSAGING_PROVIDER', 'dapr')

    if provider_type == 'dapr':
        return DaprProvider()
    elif provider_type == 'rabbitmq':
        return RabbitMQProvider()
    elif provider_type == 'servicebus':
        return ServiceBusProvider()
    else:
        raise ValueError(f"Unknown provider: {provider_type}")
```

## CloudEvents Required Fields

All published events MUST include:

| Field             | Type   | Description              | Example                       |
| ----------------- | ------ | ------------------------ | ----------------------------- |
| `specversion`     | String | CloudEvents spec version | `"1.0"`                       |
| `type`            | String | Event type (namespaced)  | `"com.xshopai.order.created"` |
| `source`          | String | Service name             | `"order-service"`             |
| `id`              | String | Unique event ID          | `"evt-123-abc"`               |
| `time`            | String | ISO 8601 timestamp       | `"2026-02-10T10:30:00Z"`      |
| `datacontenttype` | String | Content type             | `"application/json"`          |
| `data`            | Object | Business event payload   | `{ orderId: "123", ... }`     |

Optional but recommended:

| Field           | Type   | Description               | Example                      |
| --------------- | ------ | ------------------------- | ---------------------------- |
| `traceparent`   | String | W3C Trace Context         | `"00-{traceId}-{spanId}-01"` |
| `correlationId` | String | Request correlation ID    | `"req-abc-123"`              |
| `metadata`      | Object | Service-specific metadata | `{ userId: "user-123" }`     |

## Service Status

| Service                 | Language   | Pattern            | Status       |
| ----------------------- | ---------- | ------------------ | ------------ |
| auth-service            | Node.js    | Manual CloudEvents | ✅ Compliant |
| notification-service    | TypeScript | Manual CloudEvents | ✅ Compliant |
| admin-service           | Node.js    | Manual CloudEvents | ✅ Compliant |
| user-service            | Node.js    | Manual CloudEvents | ✅ Compliant |
| review-service          | Node.js    | Manual CloudEvents | ✅ Compliant |
| audit-service           | TypeScript | Manual CloudEvents | ✅ Compliant |
| product-service         | Python     | Manual CloudEvents | ✅ Compliant |
| inventory-service       | Python     | Manual CloudEvents | ✅ Compliant |
| order-service           | C#         | Manual CloudEvents | ✅ Compliant |
| payment-service         | C#         | Manual CloudEvents | ✅ Compliant |
| cart-service            | Java       | Manual CloudEvents | ✅ Compliant |
| order-processor-service | Java       | Manual CloudEvents | ✅ Compliant |

## Provider Support Matrix

| Feature                | Dapr | RabbitMQ | Azure Service Bus |
| ---------------------- | ---- | -------- | ----------------- |
| CloudEvents 1.0        | ✅   | ✅       | ✅                |
| W3C Trace Context      | ✅   | ✅       | ✅                |
| Topic-based routing    | ✅   | ✅       | ✅                |
| Dead letter queue      | ✅   | ✅       | ✅                |
| Message retry          | ✅   | ✅       | ✅                |
| At-least-once delivery | ✅   | ✅       | ✅                |

## Testing CloudEvents Compliance

Use this checklist when implementing event publishers:

- [ ] Event has `specversion: "1.0"`
- [ ] Event has namespaced `type` (e.g., `com.xshopai.{service}.{event}`)
- [ ] Event has `source` matching service name
- [ ] Event has unique `id` (UUID or timestamp-based)
- [ ] Event has ISO 8601 `time` in UTC
- [ ] Event has `datacontenttype: "application/json"`
- [ ] Event has `data` object with business payload
- [ ] Event includes `traceparent` for distributed tracing (recommended)
- [ ] Event includes `correlationId` for request tracing (recommended)
- [ ] Publisher uses messaging abstraction layer (not direct Dapr/RabbitMQ/ServiceBus SDK)
      | order-processor | Java | ✅ Native (SDK) |

## Troubleshooting

### Empty `{}` Received by Consumer

**Symptom**: Consumer logs show `rawBody: "{}"`, `hasData: false`

**Cause**: Publisher used `rawPayload: 'true'` with manual CloudEvents wrapping

**Fix**: Remove `rawPayload` metadata, send business data directly

### Double-Wrapped CloudEvents

**Symptom**: Consumer sees `data: { specversion: "1.0", data: {...} }`

**Cause**: Manually wrapping in CloudEvents, then Dapr wraps again

**Fix**: Send business data only, let Dapr wrap

## Reference

- [Dapr Pub/Sub API](https://docs.dapr.io/reference/api/pubsub_api/)
- [CloudEvents Specification](https://cloudevents.io/)
- [Azure Service Bus Dapr Component](https://docs.dapr.io/reference/components-reference/supported-pubsub/setup-azure-servicebus-topics/)

---

**Last Updated**: February 4, 2026  
**Owner**: Platform Team
