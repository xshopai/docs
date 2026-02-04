# CloudEvents Standardization Across xshopai Services

## ✅ Standard Pattern (Use This)

### Publishing Events

**Send business data directly - let Dapr handle CloudEvents wrapping:**

```javascript
// Node.js/TypeScript
await client.pubsub.publish(pubsubName, topic, eventData);
// NO publishOptions, NO rawPayload metadata
```

```python
# Python
client.publish_event(
    pubsub_name=pubsub_name,
    topic_name=topic,
    data=json.dumps(event_data),
    data_content_type="application/json"
)
# NO publish_metadata
```

```csharp
// C# - Dapr SDK handles CloudEvents automatically
await daprClient.PublishEventAsync(pubsubName, topic, eventData);
```

### Consuming Events

**Dapr delivers CloudEvents format - extract data from `data` field:**

```javascript
// Node.js/TypeScript
const cloudEvent = req.body;
const eventData = cloudEvent.data || cloudEvent;
```

```python
# Python
event_data = await request.json()
data = event_data.get("data", {})
```

```java
// Java - Use CloudEvent<T> type
@PostMapping("/events/topic")
public ResponseEntity<Void> handleEvent(@RequestBody CloudEvent<MyEventType> cloudEvent) {
    MyEventType data = cloudEvent.getData();
}
```

## ❌ Anti-Patterns (Don't Do This)

### Don't Use rawPayload

```javascript
// ❌ WRONG - causes deserialization issues with Azure Service Bus
const publishOptions = {
  metadata: { rawPayload: 'true' }
};
await client.pubsub.publish(pubsubName, topic, data, publishOptions);
```

### Don't Manually Wrap in CloudEvents

```javascript
// ❌ WRONG - Dapr does this automatically
const cloudEvent = {
  specversion: '1.0',
  type: 'com.xshopai.event',
  data: eventData
};
await client.pubsub.publish(pubsubName, topic, cloudEvent);
```

## Why This Standard?

1. **Dapr Native Behavior**: Dapr automatically wraps/unwraps CloudEvents
2. **Azure Service Bus Compatibility**: Native handling works correctly with Service Bus Topics
3. **Cross-Platform Consistency**: Works across Node.js, Python, Java, C#
4. **No Double-Wrapping**: Prevents nested CloudEvents issues
5. **Simpler Code**: Less boilerplate, fewer bugs

## Service Status

| Service | Language | Status |
|---------|----------|--------|
| auth-service | Node.js | ✅ Standardized |
| notification-service | TypeScript | ✅ Standardized |
| admin-service | Node.js | ✅ Standardized |
| user-service | Node.js | ✅ Standardized |
| review-service | Node.js | ✅ Standardized |
| audit-service | TypeScript | ✅ Standardized |
| product-service | Python | ✅ Standardized |
| inventory-service | Python | ✅ Standardized |
| order-service | C# | ✅ Native (SDK) |
| payment-service | C# | ✅ Native (SDK) |
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
