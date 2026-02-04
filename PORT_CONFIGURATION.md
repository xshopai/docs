# xshopai Platform - Port Configuration

## Local Development Configuration

When running **multiple services simultaneously** on the same machine, each service needs unique Dapr ports:

| #   | Service                 | App Port | Technology       | Dapr HTTP | Dapr gRPC |
| --- | ----------------------- | -------- | ---------------- | --------- | --------- |
| 1   | product-service         | 8001     | Python/FastAPI   | 3501      | 50001     |
| 2   | user-service            | 8002     | Node.js/Express  | 3502      | 50002     |
| 3   | admin-service           | 8003     | Node.js/Express  | 3503      | 50003     |
| 4   | auth-service            | 8004     | Node.js/Express  | 3504      | 50004     |
| 5   | inventory-service       | 8005     | Python/FastAPI   | 3505      | 50005     |
| 6   | order-service           | 8006     | .NET/C#          | 3506      | 50006     |
| 7   | order-processor-service | 8007     | Java/Spring Boot | 3507      | 50007     |
| 8   | cart-service            | 8008     | Java/Spring Boot | 3508      | 50008     |
| 9   | payment-service         | 8009     | .NET/C#          | 3509      | 50009     |
| 10  | review-service          | 8010     | Node.js/Express  | 3510      | 50010     |
| 11  | notification-service    | 8011     | Node.js/Express  | 3511      | 50011     |
| 12  | audit-service           | 8012     | Node.js/Express  | 3512      | 50012     |
| 13  | chat-service            | 8013     | Node.js/Express  | 3513      | 50013     |
| 14  | web-bff                 | 8014     | Node.js/Express  | 3514      | 50014     |
| 15  | customer-ui             | 3000     | React            | -         | -         |
| 16   | admin-ui                | 3001     | React            | -         | -         |

> **Pattern**: Dapr HTTP = 3500 + service number, Dapr gRPC = 50000 + service number
