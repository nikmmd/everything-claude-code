# Testing Requirements

## Minimum Test Coverage: 80%

Test Types (ALL required):
1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations
3. **E2E Tests** - Critical user flows (Playwright)

## Test-Driven Development

MANDATORY workflow:
1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)

## Troubleshooting Test Failures

1. Use **tdd-agent** agent
2. Check test isolation
3. Verify mocks are correct
4. Fix implementation, not tests (unless tests are wrong)

## Agent Support

- **tdd-agent** - Use PROACTIVELY for new features, enforces write-tests-first
- **e2e-runner** - Playwright E2E testing specialist

## Language-Specific Testing Tools

| Language | Framework | Coverage | E2E | Run Command |
|----------|-----------|----------|-----|-------------|
| TypeScript | Jest/Vitest | c8/istanbul | Playwright | `npm test` |
| Python | pytest | pytest-cov | pytest-playwright | `uv run pytest` or `poetry run pytest` |
| Go | testing | go test -cover | testcontainers | `go test ./...` |
| Terraform | tflint | - | Terratest | `terraform validate` |

### Python Testing

```bash
# Run tests
uv run pytest tests/ -v

# With coverage (80%+ required)
uv run pytest tests/ --cov=src --cov-report=html

# Run specific test
uv run pytest tests/test_markets.py -v

# Run async tests
uv run pytest tests/ -v --asyncio-mode=auto

# Watch mode
uv run ptw tests/
```

**Key packages**: pytest, pytest-asyncio, pytest-cov, pytest-mock, httpx

### Go Testing

```bash
# Run all tests
go test ./...

# With coverage (80%+ required)
go test ./... -cover

# Coverage report
go test ./... -coverprofile=coverage.out
go tool cover -html=coverage.out

# Run with race detector
go test ./... -race

# Run specific test
go test ./handlers -run TestGetMarket -v

# Benchmarks
go test ./... -bench=. -benchmem
```

**Key patterns**: Table-driven tests, httptest, testify/mock

### Terraform Testing

```bash
# Syntax validation
terraform validate

# Lint
tflint --recursive

# Security scan
tfsec .
checkov -d .

# Plan (dry run)
terraform plan
```

**For infrastructure testing**: Use Terratest (Go-based) for integration tests
