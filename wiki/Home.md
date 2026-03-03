# xshopai Documentation Wiki

Welcome to the **xshopai** platform documentation — a polyglot, AI-powered, open-source e-commerce platform built with microservices architecture.

---

## Quick Links

| Resource                                       | Description                                                      |
| :--------------------------------------------- | :--------------------------------------------------------------- |
| [Quick Start](../QUICKSTART.md)                | Get the full platform running in under 10 minutes                |
| [Installation Guide](../INSTALLATION_GUIDE.md) | Detailed setup with Codespaces, Dapr, and single-service options |

---

## Wiki Pages

### Architecture & Design

| Page                                                | Description                                                          |
| :-------------------------------------------------- | :------------------------------------------------------------------- |
| [Architecture Overview](Architecture-Overview.md)   | Platform architecture, service interactions, and design principles   |
| [Service Catalog](Service-Catalog.md)               | All 16 services — tech stack, ports, databases, and responsibilities |
| [Event Catalog](Event-Catalog.md)                   | Complete Dapr pub/sub event reference across all services            |
| [Database Guide](Database-Guide.md)                 | Polyglot persistence — MongoDB, PostgreSQL, SQL Server, MySQL, Redis |
| [Dapr Integration Guide](Dapr-Integration-Guide.md) | Dapr setup, components, pub/sub, state store, and service invocation |

### Development

| Page                                  | Description                                                  |
| :------------------------------------ | :----------------------------------------------------------- |
| [Getting Started](Getting-Started.md) | Prerequisites, clone, docker compose, seed, run              |
| [API Reference](API-Reference.md)     | REST endpoints for every service                             |
| [Testing Guide](Testing-Guide.md)     | Testing strategies per language — Jest, pytest, xUnit, JUnit |

### Operations

| Page                                    | Description                                                    |
| :-------------------------------------- | :------------------------------------------------------------- |
| [CI/CD Pipelines](CICD-Pipelines.md)    | GitHub Actions workflows, OIDC auth, and deployment automation |
| [Deployment Guide](Deployment-Guide.md) | Local Docker, Azure App Service, Azure Container Apps          |
| [Troubleshooting](Troubleshooting.md)   | Common issues and solutions across all services                |

---

## Platform at a Glance

| Metric         | Value                                                     |
| :------------- | :-------------------------------------------------------- |
| **Services**   | 14 backend + 2 frontend = 16 total                        |
| **Languages**  | C#, Java, Python, TypeScript, JavaScript                  |
| **Frameworks** | Express, FastAPI, Flask, Spring Boot, ASP.NET Core, React |
| **Databases**  | MongoDB × 3, PostgreSQL × 2, SQL Server × 2, MySQL, Redis |
| **Messaging**  | Dapr pub/sub with RabbitMQ backend                        |
| **Tracing**    | OpenTelemetry + Zipkin                                    |
| **CI/CD**      | GitHub Actions with OIDC Azure auth                       |
| **IaC**        | Bicep (Azure App Service), Bash (Azure Container Apps)    |
| **License**    | MIT                                                       |

---

## Additional Resources

- [Platform Architecture](../PLATFORM_ARCHITECTURE.md) — Detailed architecture document
- [Messaging Architecture](../MESSAGING_ARCHITECTURE.md) — Event-driven patterns deep dive
- [CloudEvents Standard](../CLOUDEVENTS_STANDARD.md) — Event payload specification
- [Port Configuration](../PORT_CONFIGURATION.md) — All service and Dapr port assignments
- [Monitoring Architecture](../MONITORING_ARCHITECTURE.md) — Azure Monitor, App Insights, KQL queries
- [Contributing](../CONTRIBUTING.md) — How to contribute
- [Code of Conduct](../CODE_OF_CONDUCT.md) — Community guidelines
