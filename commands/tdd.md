---
description: Enforce test-driven development. Write tests FIRST, then implement minimal code to pass. Ensure 80%+ coverage.
---

# TDD

Delegates to the **tdd-agent** agent for test-driven development.

## Workflow

1. **Scaffold** — Define types/interfaces for inputs and outputs
2. **RED** — Write failing tests (tests must fail because code doesn't exist)
3. **GREEN** — Write minimal implementation to make tests pass
4. **REFACTOR** — Improve code while keeping tests green
5. **COVERAGE** — Verify 80%+ coverage (100% for critical/financial code)

## When to Use

- Implementing new features or functions
- Fixing bugs (write test that reproduces bug first)
- Refactoring existing code
- Building critical business logic

## Test Types

| Type | Tool | Scope |
|------|------|-------|
| Unit | Jest/Vitest/pytest | Functions, utilities, components |
| Integration | Same + httptest | API endpoints, DB operations |
| E2E | Playwright (use `/e2e`) | Full user journeys |

## Rules

- Tests MUST be written BEFORE implementation
- Run tests after each step to verify RED → GREEN transition
- One assert per test when possible
- Test behavior, not implementation details
- Use semantic selectors (`data-testid`, roles), not CSS classes

## Related

- `/plan` — Understand what to build before TDD
- `/e2e` — End-to-end tests for user journeys
- `/code-review` — Review completed implementation
- `/test-coverage` — Verify coverage gaps
