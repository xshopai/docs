# Testing Guide

xshopai uses **five testing frameworks** across its polyglot stack. Each service maintains its own test suite following language-idiomatic patterns.

---

## Framework Summary

| Language         | Framework         | Runner             | Coverage                | Services                                 |
| :--------------- | :---------------- | :----------------- | :---------------------- | :--------------------------------------- |
| JavaScript (ESM) | Jest + Babel      | `npm test`         | `npm run test:coverage` | user, auth, admin, review                |
| TypeScript       | Jest + ts-jest    | `npm test`         | `npm run test:coverage` | web-bff, cart, notification, audit, chat |
| Python           | pytest            | `pytest tests/ -v` | `pytest --cov=src`      | product, inventory                       |
| C# / .NET 8      | xUnit + Moq       | `dotnet test`      | via coverlet            | order, payment                           |
| Java 17          | JUnit 5 + Mockito | `mvn test`         | via JaCoCo              | order-processor                          |

**Frontend:**

| App               | Framework                    | Runner             |
| :---------------- | :--------------------------- | :----------------- |
| customer-ui       | React Testing Library + Jest | `npm test`         |
| admin-ui          | React Testing Library + Jest | `npm test`         |
| customer-ui (E2E) | Playwright                   | `npm run test:e2e` |

---

## Running Tests

### All Tests (per service)

```bash
# Node.js / TypeScript services
cd user-service && npm test
cd cart-service && npm test

# Python services
cd product-service && pytest tests/ -v
cd inventory-service && pytest tests/ -v

# .NET services
cd order-service && dotnet test
cd payment-service && dotnet test

# Java service
cd order-processor-service && mvn test

# Frontend
cd customer-ui && npm test
cd admin-ui && npm test
```

### By Test Type

```bash
# Unit tests only
cd user-service && npm run test:unit
cd product-service && pytest tests/unit/ -v

# Integration tests
cd user-service && npm run test:integration
cd inventory-service && pytest tests/integration/ -v

# End-to-end tests
cd auth-service && npm run test:e2e
cd customer-ui && npm run test:e2e
```

### Coverage Reports

```bash
# Node.js / TypeScript
cd user-service && npm run test:coverage

# Python
cd product-service && pytest --cov=src --cov-report=html

# .NET
cd order-service && dotnet test --collect:"XPlat Code Coverage"

# Java
cd order-processor-service && mvn verify    # JaCoCo report
```

---

## JavaScript / TypeScript — Jest

### Configuration

Services use either `jest.config.js` or the `jest` key in `package.json`.

**ESM services** (user, auth, admin, review) require Babel transform:

```javascript
// jest.config.js
export default {
  transform: { '^.+\\.js$': 'babel-jest' },
  testEnvironment: 'node',
  testMatch: ['**/tests/**/*.test.js'],
  coverageDirectory: 'coverage',
  coverageThreshold: { global: { branches: 80, functions: 80, lines: 80 } },
};
```

```json
// babel.config.json
{
  "env": {
    "test": {
      "presets": [["@babel/preset-env", { "targets": { "node": "current" } }]]
    }
  }
}
```

**TypeScript services** (web-bff, cart, notification, audit, chat):

```javascript
// jest.config.js
export default {
  preset: 'ts-jest',
  testEnvironment: 'node',
  extensionsToTreatAsEsm: ['.ts'],
  moduleNameMapper: { '^(\\.{1,2}/.*)\\.js$': '$1' },
  transform: { '^.+\\.tsx?$': ['ts-jest', { useESM: true }] },
};
```

### Mocking Patterns

```javascript
// Mock a module
jest.mock('../src/clients/userServiceClient.js');

// Mock Dapr client
jest.mock('@dapr/dapr', () => ({
  DaprClient: jest.fn().mockImplementation(() => ({
    pubsub: { publish: jest.fn() },
    state: { save: jest.fn(), get: jest.fn() },
  })),
}));

// Mock Express request/response
const req = { body: { email: 'test@test.com' }, headers: {} };
const res = { status: jest.fn().mockReturnThis(), json: jest.fn() };
```

---

## Python — pytest

### Configuration

```ini
# pytest.ini
[pytest]
testpaths = tests
python_files = test_*.py
python_functions = test_*
addopts = -v --tb=short
```

### Test Structure

```
product-service/
└── tests/
    ├── unit/
    │   ├── test_product_service.py
    │   └── test_product_repository.py
    └── integration/
        └── test_product_routes.py
```

### Testing Patterns

**FastAPI (product-service) — httpx AsyncClient:**

```python
import pytest
from httpx import AsyncClient, ASGITransport
from src.main import app

@pytest.fixture
async def client():
    transport = ASGITransport(app=app)
    async with AsyncClient(transport=transport, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_get_products(client):
    response = await client.get("/api/v1/products")
    assert response.status_code == 200
    assert isinstance(response.json(), list)
```

**Flask (inventory-service) — pytest fixtures:**

```python
import pytest
from src import create_app

@pytest.fixture
def app():
    app = create_app('testing')
    with app.app_context():
        yield app

@pytest.fixture
def client(app):
    return app.test_client()

def test_get_inventory(client):
    response = client.get('/api/v1/inventory')
    assert response.status_code == 200
```

---

## C# / .NET — xUnit

### Test Structure

```
order-service/
└── OrderService.Tests/
    ├── OrderService.Tests.csproj
    ├── Services/
    │   └── OrderServiceTests.cs
    ├── Controllers/
    │   └── OrdersControllerTests.cs
    └── Validators/
        └── CreateOrderValidatorTests.cs
```

### Testing Patterns

```csharp
using Xunit;
using Moq;
using FluentAssertions;

public class OrderServiceTests
{
    private readonly Mock<IOrderRepository> _repoMock;
    private readonly Mock<IMessagingProvider> _messagingMock;
    private readonly OrderService _service;

    public OrderServiceTests()
    {
        _repoMock = new Mock<IOrderRepository>();
        _messagingMock = new Mock<IMessagingProvider>();
        _service = new OrderService(_repoMock.Object, _messagingMock.Object);
    }

    [Fact]
    public async Task GetOrder_ReturnsOrder_WhenExists()
    {
        // Arrange
        var orderId = Guid.NewGuid();
        var order = new Order { Id = orderId, Status = OrderStatus.Pending };
        _repoMock.Setup(r => r.GetByIdAsync(orderId)).ReturnsAsync(order);

        // Act
        var result = await _service.GetOrderAsync(orderId);

        // Assert
        result.Should().NotBeNull();
        result.Id.Should().Be(orderId);
    }
}
```

### Running .NET Tests

```bash
# All tests
dotnet test

# Specific project
dotnet test OrderService.Tests/

# With verbosity
dotnet test --verbosity normal

# Filter by test name
dotnet test --filter "ClassName=OrderServiceTests"
```

---

## Java — JUnit 5

### Test Structure

```
order-processor-service/
└── src/test/java/com/xshopai/orderprocessor/
    ├── service/
    │   └── SagaOrchestratorServiceTest.java
    ├── controller/
    │   └── AdminControllerTest.java
    └── events/consumer/
        └── OrderCreatedConsumerTest.java
```

### Testing Patterns

```java
@ExtendWith(MockitoExtension.class)
class SagaOrchestratorServiceTest {

    @Mock private OrderProcessingSagaRepository sagaRepository;
    @Mock private DaprEventPublisher eventPublisher;
    @InjectMocks private SagaOrchestratorService service;

    @Test
    void shouldCreateSagaOnOrderCreated() {
        // Arrange
        var event = OrderCreatedEvent.builder()
            .orderId("order-123")
            .customerId("user-456")
            .build();

        // Act
        service.handleOrderCreated(event);

        // Assert
        verify(sagaRepository).save(any(OrderProcessingSaga.class));
    }
}
```

### Running Java Tests

```bash
# All tests
mvn test

# Integration tests
mvn verify

# Specific test class
mvn test -Dtest=SagaOrchestratorServiceTest

# Skip tests during build
mvn clean package -DskipTests
```

---

## Frontend — React Testing Library

### Testing Patterns

```javascript
import { render, screen, fireEvent } from '@testing-library/react';
import { Provider } from 'react-redux';
import { BrowserRouter } from 'react-router-dom';
import ProductList from '../components/ProductList';
import { store } from '../store';

const renderWithProviders = (ui) => {
  return render(
    <Provider store={store}>
      <BrowserRouter>{ui}</BrowserRouter>
    </Provider>,
  );
};

test('renders product list', async () => {
  renderWithProviders(<ProductList />);
  expect(await screen.findByText(/products/i)).toBeInTheDocument();
});
```

### E2E — Playwright (customer-ui)

```bash
# Install Playwright browsers
npx playwright install

# Run all E2E tests
npm run test:e2e

# Run with UI mode
npx playwright test --ui

# Run specific test file
npx playwright test tests/e2e/checkout.spec.js
```

---

## Mocking External Dependencies

| Dependency              | Mock Strategy                                                 |
| :---------------------- | :------------------------------------------------------------ |
| MongoDB                 | `mongodb-memory-server` (Node.js), `mongomock-motor` (Python) |
| Redis                   | `ioredis-mock` or test against Docker container               |
| SQL Server / PostgreSQL | In-memory database or Docker test container                   |
| Dapr SDK                | Jest mocks / Mockito mocks of SDK clients                     |
| HTTP services           | `nock` (Node.js), `responses` (Python), `WireMock` (Java)     |
| Azure OpenAI            | Mock responses in unit tests                                  |

---

## CI Integration

All tests run automatically in GitHub Actions CI pipelines:

```yaml
# Typical test step in ci-app-service.yml
- name: Run tests
  run: npm test # Node.js / TypeScript
  # run: pytest tests/   # Python
  # run: dotnet test     # .NET
  # run: mvn test        # Java
```

Tests must pass before Docker image build and deployment proceed, as set via the `needs` dependency chain in the pipeline.
