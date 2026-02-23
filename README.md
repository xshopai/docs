# xshopai Documentation

Central documentation for xshopai project

## Available Documentation

### Architecture & Workflows

- **[Platform Architecture](PLATFORM_ARCHITECTURE.md)** - Complete platform architecture overview with service catalog
- **[Workflow Documentation](workflows/)** - Visual sequence diagrams for key use cases
  - [User Registration Workflow](workflows/USER_REGISTRATION_WORKFLOW.md) - Complete registration flow with service interactions
- **[CloudEvents Standard](CLOUDEVENTS_STANDARD.md)** - Event payload specification for pub/sub messaging
- **[Messaging Architecture](MESSAGING_ARCHITECTURE.md)** - Dapr pub/sub and event-driven patterns
- **[Port Configuration](PORT_CONFIGURATION.md)** - Service port assignments and Dapr sidecar ports

### Development & Security

- **[Pre-commit Hooks Security Guide](PRE_COMMIT_HOOKS_SECURITY_GUIDE.md)** - Comprehensive guide for implementing automated security checks and code quality enforcement
- **[Logging Standard](LOGGING_STANDARD.md)** - Standards for logging across services
- **[Logging Implementation](LOGGING_IMPLEMENTATION.md)** - Implementation details for logging
- **[Correlation ID Guide](CORRELATION_ID_GUIDE.md)** - Guide for implementing correlation IDs for request tracking

### Project Management

- **[Contributing Guidelines](CONTRIBUTING.md)** - How to contribute to the project
- **[Code of Conduct](CODE_OF_CONDUCT.md)** - Community guidelines and expectations

## Quick Links

### Security Documentation

For security-related documentation, start with:

1. **[Pre-commit Hooks Security Guide](PRE_COMMIT_HOOKS_SECURITY_GUIDE.md)** - Essential reading for all developers to understand automated security measures

### GitHub Templates

Issue and pull request templates are available in the `.github/` directory:

- Bug report template
- Feature request template
- Custom issue template

## Getting Started

### For Developers

1. Review the [Platform Architecture](PLATFORM_ARCHITECTURE.md) to understand the system
2. Study [Workflow Documentation](workflows/) to see how services interact
3. Read the [Contributing Guidelines](CONTRIBUTING.md)
4. Set up [Pre-commit Hooks](PRE_COMMIT_HOOKS_SECURITY_GUIDE.md) for security
5. Follow the [Logging Standards](LOGGING_STANDARD.md) in your code
6. Implement [Correlation IDs](CORRELATION_ID_GUIDE.md) for request tracking

### Understanding Service Communication

- **Synchronous**: See HTTP endpoints in service-specific PRDs
- **Asynchronous**: Review [CloudEvents Standard](CLOUDEVENTS_STANDARD.md) and [Messaging Architecture](MESSAGING_ARCHITECTURE.md)
- **Visual Flows**: Check [Workflow diagrams](workflows/) for end-to-end sequences

For questions or additional documentation needs, please create an issue in the main repository.
