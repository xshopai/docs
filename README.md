# xshopai Documentation

Central documentation hub for the xshopai e-commerce platform — 16 microservices, 5 languages, event-driven architecture.

## Quick Start

| Guide                                           | Description                                                                         |
| :---------------------------------------------- | :---------------------------------------------------------------------------------- |
| **[Quickstart](QUICKSTART.md)**                 | Clone, start infrastructure, seed data, and run services in under 10 minutes        |
| **[Installation Guide](INSTALLATION_GUIDE.md)** | Comprehensive setup with 4 deployment options (Docker Compose, Dapr, hybrid, cloud) |

## Wiki

In-depth guides covering every aspect of the platform:

| Page                                                     | Description                                                      |
| :------------------------------------------------------- | :--------------------------------------------------------------- |
| [Wiki Home](wiki/Home.md)                                | Navigation hub and documentation map                             |
| [Architecture Overview](wiki/Architecture-Overview.md)   | System design, communication patterns, saga flow, security       |
| [Getting Started](wiki/Getting-Started.md)               | Prerequisites, setup, and first-run instructions                 |
| [Service Catalog](wiki/Service-Catalog.md)               | All 16 services — ports, languages, databases, events            |
| [API Reference](wiki/API-Reference.md)                   | REST endpoints for BFF gateway and direct service APIs           |
| [Event Catalog](wiki/Event-Catalog.md)                   | CloudEvents reference — all topics, publishers, consumers        |
| [Database Guide](wiki/Database-Guide.md)                 | Polyglot persistence — connections, schemas, migrations, seeding |
| [Dapr Integration Guide](wiki/Dapr-Integration-Guide.md) | Pub/sub, state store, service invocation, component config       |
| [Testing Guide](wiki/Testing-Guide.md)                   | Jest, pytest, xUnit, JUnit — per-language patterns and commands  |
| [CI/CD Pipelines](wiki/CICD-Pipelines.md)                | GitHub Actions workflows, OIDC auth, multi-stage Dockerfiles     |
| [Deployment Guide](wiki/Deployment-Guide.md)             | Local Docker, Azure App Service (Bicep), Azure Container Apps    |
| [Troubleshooting](wiki/Troubleshooting.md)               | Common issues — Docker, databases, Dapr, auth, CORS              |

## Reference Documents

### Architecture & Workflows

- **[Platform Architecture](PLATFORM_ARCHITECTURE.md)** — Complete platform architecture overview with service catalog
- **[Workflow Documentation](workflows/)** — Visual sequence diagrams for key use cases
  - [User Registration Workflow](workflows/USER_REGISTRATION_WORKFLOW.md) — Complete registration flow with service interactions
- **[CloudEvents Standard](CLOUDEVENTS_STANDARD.md)** — Event payload specification for pub/sub messaging
- **[Messaging Architecture](MESSAGING_ARCHITECTURE.md)** — Dapr pub/sub and event-driven patterns
- **[Port Configuration](PORT_CONFIGURATION.md)** — Service port assignments and Dapr sidecar ports
- **[Monitoring Architecture](MONITORING_ARCHITECTURE.md)** — Azure Monitor, Application Insights, KQL queries

### Project Management

- **[Contributing Guidelines](CONTRIBUTING.md)** — How to contribute to the project
- **[Code of Conduct](CODE_OF_CONDUCT.md)** — Community guidelines and expectations
- **[Review Checklist](REVIEW_CHECKLIST.md)** — Pull request review standards
- **[Service Architecture Template](SERVICE_ARCHITECTURE_TEMPLATE.md)** — Template for new service design docs
- **[PRD Template](PRD_TEMPLATE.md)** — Product requirements document template

## Getting Started

### For New Developers

1. Run through the [Quickstart](QUICKSTART.md) to get the platform running
2. Review the [Architecture Overview](wiki/Architecture-Overview.md) to understand the system
3. Browse the [Service Catalog](wiki/Service-Catalog.md) to find the service you'll work on
4. Read the [Contributing Guidelines](CONTRIBUTING.md) before your first PR

### Understanding Service Communication

- **Synchronous**: See the [API Reference](wiki/API-Reference.md) for REST endpoints
- **Asynchronous**: Review the [Event Catalog](wiki/Event-Catalog.md) and [Messaging Architecture](MESSAGING_ARCHITECTURE.md)
- **Visual Flows**: Check [Workflow diagrams](workflows/) for end-to-end sequences

For questions or additional documentation needs, please create an issue in the docs repository.
