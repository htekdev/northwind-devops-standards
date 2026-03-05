# Testing Strategy

> Northwind Traders — Platform Engineering

This document outlines the testing strategy and requirements for all Northwind Traders projects. Consistent testing practices ensure code quality, reduce regressions, and enable confident deployments.

---

## 1. Testing Tiers

All projects must implement tests across the following tiers:

### Tier 1 — Unit Tests

- **Scope:** Individual functions, methods, and classes in isolation.
- **Dependencies:** All external dependencies (databases, APIs, file systems) must be mocked or stubbed.
- **Execution time:** Each unit test should complete in under 100 milliseconds.
- **Ownership:** Developers write unit tests alongside the code they produce.

### Tier 2 — Integration Tests

- **Scope:** Interactions between two or more modules, including database access, API calls, and message queues.
- **Dependencies:** Use real instances where practical (e.g., in-memory databases, Docker containers via test fixtures).
- **Execution time:** Individual integration tests should complete in under 5 seconds.
- **Ownership:** Developers write integration tests for module boundaries they own.

### Tier 3 — End-to-End Tests

- **Scope:** Full user workflows exercised through the application's public interfaces (UI, API).
- **Dependencies:** Run against a deployed (or locally composed) instance of the full application stack.
- **Execution time:** E2E test suites should complete in under 10 minutes total.
- **Ownership:** QA engineers and developers collaborate on E2E test suites.

> **Note:** Not all projects require Tier 3 tests. Libraries, CLI tools, and infrastructure-only repositories may omit E2E tests with documented justification.

---

## 2. Coverage Requirements

### Minimum Thresholds

| Metric | Minimum | Enforced By |
|--------|---------|-------------|
| Line coverage | 80% | CI workflow `coverage-threshold` input |
| Branch coverage | 75% | Project-level configuration |
| Function coverage | 80% | Project-level configuration |

### Enforcement

Coverage thresholds are enforced at the CI pipeline level:

- **.NET projects:** Coverage is collected via `XPlat Code Coverage` (Coverlet) and reported in Cobertura XML format. The `ci-dotnet.yml` workflow parses the `line-rate` attribute and fails if it falls below the threshold.
- **Node.js projects:** Coverage is collected via the test runner's built-in coverage support (Jest, Vitest, etc.) and reported in lcov and JSON summary formats. The `ci-node.yml` workflow reads `coverage-summary.json` and fails if line coverage falls below the threshold.

### How Coverage Is Measured Per Stack

#### .NET

```bash
dotnet test --collect:"XPlat Code Coverage" --results-directory ./test-results
```

The Cobertura XML file at `./test-results/**/coverage.cobertura.xml` contains the `line-rate` attribute (a decimal between 0 and 1). Multiply by 100 to get the percentage.

#### Node.js

```bash
npm run test -- --coverage --coverageDirectory=./coverage
```

The JSON summary at `./coverage/coverage-summary.json` contains per-metric percentages under `total.lines.pct`, `total.branches.pct`, and `total.functions.pct`.

---

## 3. Test Naming Conventions

Consistent naming makes tests self-documenting and makes failures easy to diagnose.

### Pattern

```
<MethodUnderTest>_<Scenario>_<ExpectedBehavior>
```

### Examples

| Language | Example |
|----------|---------|
| C# / .NET | `CalculateDiscount_WhenQuantityExceedsTen_ReturnsPercentageDiscount()` |
| JavaScript | `calculateDiscount - when quantity exceeds 10, returns percentage discount` |
| TypeScript | `calculateDiscount - given a premium customer, applies loyalty bonus` |

### Rules

- Test names must describe the **method under test**, the **scenario or input**, and the **expected outcome**.
- Avoid generic names like `Test1`, `ShouldWork`, or `HappyPath`.
- Group related tests by the class or module under test using `describe` blocks (JS/TS) or nested classes (C#).

---

## 4. Test Organization Patterns

### .NET Projects

```
src/
  MyApp/
    Services/
      OrderService.cs
tests/
  MyApp.Tests/
    Services/
      OrderServiceTests.cs
    MyApp.Tests.csproj
```

- Mirror the `src/` directory structure in `tests/`.
- One test class per production class.
- Test project names follow the pattern `<ProjectName>.Tests`.

### Node.js Projects

```
src/
  services/
    orderService.ts
  services/
    __tests__/
      orderService.test.ts
```

Or alternatively with a top-level test directory:

```
src/
  services/
    orderService.ts
tests/
  services/
    orderService.test.ts
```

- Co-located `__tests__/` directories are preferred for unit tests.
- Top-level `tests/` directories are acceptable for integration and E2E tests.
- Test files use the `.test.ts` or `.spec.ts` suffix.

---

## 5. Test Data and Fixtures

- **Avoid hard-coded magic values.** Use clearly named constants or factory functions for test data.
- **Use builders or factories** for complex object construction (e.g., `OrderBuilder.withItems(3).build()`).
- **Shared fixtures** (database seeds, mock data files) should live in a `fixtures/` or `testdata/` directory within the test project.
- **Never depend on test execution order.** Each test must be independent and idempotent.

---

## 6. Mocking Guidelines

- Mock only what you own. Do not mock third-party library internals.
- Prefer dependency injection to enable easy test doubles.
- Use the project's standard mocking library:
  - **.NET:** Moq or NSubstitute
  - **Node.js:** Jest built-in mocks or Sinon
- Verify mock interactions sparingly — assert on outcomes rather than implementation details.
