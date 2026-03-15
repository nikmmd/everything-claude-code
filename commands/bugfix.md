---
description: Diagnose and fix build errors, functional bugs, and performance issues with minimal diffs.
---

# Bugfix

Delegates to the **error-resolver** agent for error diagnosis and minimal fixes.

## Error Categories

- **Build**: Compilation, type errors, imports, config
- **Functional**: Logic errors, runtime exceptions, edge cases
- **Non-Functional**: Performance, memory leaks, race conditions

## Workflow

1. **Diagnose** — Identify error category and collect evidence
2. **Root cause** — Trace execution to failure point
3. **Minimal fix** — Change only what's necessary
4. **Verify** — Error resolved, tests pass, no regressions

## Commands by Language

| Language | Build | Test | Lint |
|----------|-------|------|------|
| TypeScript | `npm run build` | `npm test` | `npx eslint .` |
| Python | `python -m py_compile` | `pytest` | `ruff check .` |
| Go | `go build ./...` | `go test ./...` | `golangci-lint run` |

## Rules

- Fix one error at a time
- Stop if fix introduces new errors
- Stop if same error persists after 3 attempts
- Minimal diffs only — no refactoring unrelated code
