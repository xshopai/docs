# Event Catalog

Complete reference for all Dapr pub/sub events in the xshopai platform. All events follow the [CloudEvents 1.0 specification](../CLOUDEVENTS_STANDARD.md).

> For detailed messaging patterns and architecture diagrams, see [Messaging Architecture](../MESSAGING_ARCHITECTURE.md).

---

## Event Format

Every event follows the CloudEvents 1.0 envelope:

```json
{
  "specversion": "1.0",
  "type": "com.xshopai.order.created",
  "source": "order-service",
  "id": "evt-unique-id",
  "time": "2026-02-10T10:30:00Z",
  "datacontenttype": "application/json",
  "data": {
    /* business payload */
  },
  "traceparent": "00-{traceId}-{spanId}-01",
  "correlationId": "req-abc-123"
}
```

---

## Service Role Matrix

| Service                 | Role          | Published | Consumed |
| :---------------------- | :------------ | :-------- | :------- |
| auth-service            | Publisher     | 6         | 0        |
| user-service            | Publisher     | 7         | 0        |
| order-service           | Publisher     | 7         | 0        |
| review-service          | Publisher     | 3         | 0        |
| admin-service           | Publisher     | 3         | 0        |
| cart-service            | Publisher     | 4         | 0        |
| payment-service         | Hybrid        | 4         | 2        |
| inventory-service       | Hybrid        | 6         | 8        |
| product-service         | Hybrid        | 6         | 5        |
| order-processor-service | Hybrid (Saga) | 7         | 7        |
| notification-service    | Hybrid        | 2         | 15+      |
| audit-service           | Consumer      | 0         | 40+      |
| web-bff                 | None          | 0         | 0        |
| chat-service            | None          | 0         | 0        |

---

## Events by Domain

### Auth Events

Published by **auth-service**.

| Event                              | Topic                              | Description                 | Consumers           |
| :--------------------------------- | :--------------------------------- | :-------------------------- | :------------------ |
| `auth.login`                       | `auth.login`                       | User successfully logged in | audit, notification |
| `auth.register`                    | `auth.register`                    | New user registered         | audit, notification |
| `auth.email.verification.required` | `auth.email.verification.required` | Email verification needed   | audit, notification |
| `auth.password.reset.requested`    | `auth.password.reset.requested`    | Password reset initiated    | audit, notification |
| `auth.password.reset.completed`    | `auth.password.reset.completed`    | Password successfully reset | audit, notification |
| `auth.account.reactivated`         | `auth.account.reactivated`         | Account reactivated         | audit, notification |

### User Events

Published by **user-service**.

| Event                 | Topic                 | Description              | Consumers           |
| :-------------------- | :-------------------- | :----------------------- | :------------------ |
| `user.created`        | `user.created`        | New user profile created | audit, notification |
| `user.updated`        | `user.updated`        | User profile updated     | audit, notification |
| `user.deleted`        | `user.deleted`        | User account deleted     | audit, notification |
| `user.logged_in`      | `user.logged_in`      | User login recorded      | audit               |
| `user.logged_out`     | `user.logged_out`     | User logout recorded     | audit               |
| `user.deactivated`    | `user.deactivated`    | Account deactivated      | audit, notification |
| `user.email_verified` | `user.email_verified` | Email verified           | audit, notification |

### Order Events

Published by **order-service**.

| Event             | Topic             | Description      | Consumers                                                |
| :---------------- | :---------------- | :--------------- | :------------------------------------------------------- |
| `order.created`   | `order.created`   | New order placed | order-processor, payment, inventory, audit, notification |
| `order.confirmed` | `order.confirmed` | Order confirmed  | audit, notification                                      |
| `order.shipped`   | `order.shipped`   | Order shipped    | audit, notification, inventory                           |
| `order.delivered` | `order.delivered` | Order delivered  | audit, notification, review                              |
| `order.completed` | `order.completed` | Order fulfilled  | inventory, audit, notification                           |
| `order.cancelled` | `order.cancelled` | Order cancelled  | payment, inventory, audit, notification                  |
| `order.refunded`  | `order.refunded`  | Order refunded   | inventory, audit, notification                           |

### Payment Events

Published by **payment-service**.

| Event                | Topic                | Description             | Consumers                                       |
| :------------------- | :------------------- | :---------------------- | :---------------------------------------------- |
| `payment.processing` | `payment.processing` | Payment being processed | audit                                           |
| `payment.received`   | `payment.received`   | Payment successful      | order-processor, inventory, audit, notification |
| `payment.failed`     | `payment.failed`     | Payment failed          | order-processor, audit, notification            |
| `payment.refund`     | `payment.refund`     | Refund processed        | audit, notification                             |

**Consumed by payment-service:**

| Event             | Source        | Action                   |
| :---------------- | :------------ | :----------------------- |
| `order.created`   | order-service | Initiate payment process |
| `order.cancelled` | order-service | Cancel/refund payment    |

### Inventory Events

Published by **inventory-service**.

| Event                     | Topic                     | Description              | Consumers                    |
| :------------------------ | :------------------------ | :----------------------- | :--------------------------- |
| `inventory.stock.updated` | `inventory.stock.updated` | Stock quantity changed   | product, audit               |
| `inventory.reserved`      | `inventory.reserved`      | Stock reserved for order | order-processor, audit       |
| `inventory.released`      | `inventory.released`      | Reserved stock released  | order-processor, audit       |
| `inventory.low.stock`     | `inventory.low.stock`     | Low stock alert          | notification, audit          |
| `inventory.out.of.stock`  | `inventory.out.of.stock`  | Out of stock             | product, notification, audit |
| `inventory.created`       | `inventory.created`       | New inventory record     | audit                        |

**Consumed by inventory-service:**

| Event              | Source          | Action                          |
| :----------------- | :-------------- | :------------------------------ |
| `order.created`    | order-service   | Reserve stock for order items   |
| `order.cancelled`  | order-service   | Release reserved stock          |
| `order.completed`  | order-service   | Confirm stock sold              |
| `order.refunded`   | payment-service | Return stock to available       |
| `payment.received` | payment-service | Confirm stock reservation       |
| `payment.failed`   | payment-service | Release reserved stock          |
| `product.created`  | product-service | Create initial inventory record |
| `product.deleted`  | product-service | Mark inventory as discontinued  |

### Product Events

Published by **product-service**.

| Event                    | Topic                    | Description       | Consumers           |
| :----------------------- | :----------------------- | :---------------- | :------------------ |
| `product.created`        | `product.created`        | New product added | inventory, audit    |
| `product.updated`        | `product.updated`        | Product modified  | inventory, audit    |
| `product.deleted`        | `product.deleted`        | Product removed   | inventory, audit    |
| `product.price.changed`  | `product.price.changed`  | Price updated     | audit, notification |
| `product.badge.assigned` | `product.badge.assigned` | Badge added       | audit               |
| `product.badge.removed`  | `product.badge.removed`  | Badge removed     | audit               |

**Consumed by product-service:**

| Event                     | Source            | Action                        |
| :------------------------ | :---------------- | :---------------------------- |
| `review.created`          | review-service    | Update review aggregates      |
| `review.updated`          | review-service    | Recalculate average rating    |
| `review.deleted`          | review-service    | Recalculate review aggregates |
| `inventory.stock.updated` | inventory-service | Update availability status    |
| `inventory.out.of.stock`  | inventory-service | Mark product as out of stock  |

### Review Events

Published by **review-service**.

| Event            | Topic            | Description          | Consumers      |
| :--------------- | :--------------- | :------------------- | :------------- |
| `review.created` | `review.created` | New review submitted | product, audit |
| `review.updated` | `review.updated` | Review modified      | product, audit |
| `review.deleted` | `review.deleted` | Review removed       | product, audit |

### Cart Events

Published by **cart-service**.

| Event               | Topic               | Description            | Consumers           |
| :------------------ | :------------------ | :--------------------- | :------------------ |
| `cart.item.added`   | `cart.item.added`   | Item added to cart     | audit               |
| `cart.item.removed` | `cart.item.removed` | Item removed from cart | audit               |
| `cart.cleared`      | `cart.cleared`      | Cart emptied           | audit               |
| `cart.abandoned`    | `cart.abandoned`    | Cart abandoned         | notification, audit |

### Admin Events

Published by **admin-service**.

| Event                    | Topic                    | Description            | Consumers           |
| :----------------------- | :----------------------- | :--------------------- | :------------------ |
| `admin.action.performed` | `admin.action.performed` | Admin action logged    | audit               |
| `admin.user.created`     | `admin.user.created`     | Admin created user     | audit, notification |
| `admin.config.changed`   | `admin.config.changed`   | Configuration modified | audit               |

### Notification Events

Published by **notification-service**.

| Event                 | Topic                 | Description            | Consumers |
| :-------------------- | :-------------------- | :--------------------- | :-------- |
| `notification.sent`   | `notification.sent`   | Notification delivered | audit     |
| `notification.failed` | `notification.failed` | Delivery failed        | audit     |

### Order Processor Events (Saga)

Published by **order-processor-service** during saga orchestration.

| Event                  | Topic                  | Description                  | Consumers           |
| :--------------------- | :--------------------- | :--------------------------- | :------------------ |
| `order.processing`     | `order.processing`     | Order processing started     | audit, notification |
| `order.completed`      | `order.completed`      | Order successfully completed | audit, notification |
| `order.failed`         | `order.failed`         | Order processing failed      | audit, notification |
| `payment.processing`   | `payment.processing`   | Payment step initiated       | payment, audit      |
| `inventory.reserve`    | `inventory.reserve`    | Stock reservation request    | inventory, audit    |
| `inventory.release`    | `inventory.release`    | Stock release request        | inventory, audit    |
| `shipping.preparation` | `shipping.preparation` | Shipping preparation start   | audit               |

**Consumed by order-processor-service:**

| Event                | Source            | Action                             |
| :------------------- | :---------------- | :--------------------------------- |
| `order.created`      | order-service     | Start order processing saga        |
| `payment.received`   | payment-service   | Proceed to inventory reservation   |
| `payment.failed`     | payment-service   | Cancel order, trigger compensation |
| `inventory.reserved` | inventory-service | Proceed to shipping                |
| `inventory.released` | inventory-service | Handle reservation failure         |
| `shipping.prepared`  | (future)          | Complete order                     |
| `shipping.failed`    | (future)          | Trigger compensation               |

---

## Saga Event Flow

The order fulfillment saga follows this sequence:

```
order-service         order-processor      payment-service     inventory-service
    │                       │                     │                    │
    │── order.created ─────▶│                     │                    │
    │                       │── payment.processing ──▶│                │
    │                       │                     │                    │
    │                       │◀── payment.received ───│                │
    │                       │                     │                    │
    │                       │── inventory.reserve ────────────────────▶│
    │                       │                     │                    │
    │                       │◀── inventory.reserved ──────────────────│
    │                       │                     │                    │
    │                       │── order.completed   │                    │
    │                       │                     │                    │
```

**Compensation Flow (on failure):**

```
    │                       │◀── payment.failed ─────│                │
    │                       │                     │                    │
    │                       │── inventory.release ────────────────────▶│
    │                       │                     │                    │
    │                       │── order.failed      │                    │
```

---

## Pub/Sub Configuration

All services use the same Dapr pub/sub component:

```yaml
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
```
