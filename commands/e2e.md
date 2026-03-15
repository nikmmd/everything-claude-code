---
description: Generate and run end-to-end tests with Playwright. Creates test journeys, captures artifacts on failure.
---

# E2E Tests

Delegates to the **e2e-runner** agent for Playwright end-to-end testing.

## Workflow

1. Analyze user flow and identify test scenarios
2. Generate Playwright tests using Page Object Model pattern
3. Run tests across browsers (Chrome, Firefox, Safari)
4. Capture failures with screenshots, videos, and traces
5. Identify flaky tests and recommend fixes
6. Generate results report

## When to Use

- Testing critical user journeys (login, payments, core flows)
- Verifying multi-step workflows end-to-end
- Validating frontend-backend integration
- Pre-deployment verification

## Artifacts

**Always:** HTML report, JUnit XML
**On failure:** Screenshots, video recording, trace file, network/console logs

## Quick Commands

```bash
npx playwright test                          # Run all
npx playwright test tests/e2e/file.spec.ts   # Run specific
npx playwright test --headed                 # See browser
npx playwright test --debug                  # Debug mode
npx playwright show-report                   # View report
```

## Rules

- Use `data-testid` attributes for selectors, not CSS classes
- Wait for API responses, not arbitrary timeouts
- Never run financial tests against production
- Use Page Object Model for maintainability

## Related

- `/tdd` — Unit and integration tests
- `/test-coverage` — Coverage analysis
