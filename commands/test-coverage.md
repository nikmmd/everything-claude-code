---
description: Analyze test coverage, identify gaps below 80%, and generate missing tests.
---

# Test Coverage

Delegates to the **tdd-agent** agent for coverage analysis and test generation.

## Workflow

1. Run tests with coverage (`npm test --coverage` / `pytest --cov` / `go test -cover`)
2. Parse coverage report and identify files below 80% threshold
3. For each under-covered file:
   - Analyze untested code paths
   - Generate unit tests (happy path, errors, edge cases, boundaries)
   - Generate integration tests for APIs
4. Verify new tests pass
5. Show before/after coverage metrics

## Coverage Targets

- **80% minimum** for all code
- **100% required** for financial calculations, auth logic, security-critical code

## Related

- `/tdd` — Full TDD workflow for new features
- `/e2e` — End-to-end tests for user journeys
