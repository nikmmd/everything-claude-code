# Bugfix

Diagnose and fix errors with minimal diffs using the **error-resolver** agent.

## Error Categories

- **Build Errors**: Compilation, type errors, imports, config
- **Functional Bugs**: Logic errors, runtime exceptions, edge cases
- **Non-Functional Issues**: Performance, memory leaks, race conditions

## Workflow

1. **Diagnose** the error category:
   - Build error → Run build/compile commands
   - Functional bug → Reproduce with test or manually
   - Non-functional → Profile or measure

2. **Collect evidence**:
   - Error messages and stack traces
   - Logs around failure point
   - Input that triggers issue

3. **Root cause analysis**:
   - Trace execution to failure point
   - Identify specific line/function
   - Determine minimal fix

4. **Apply minimal fix**:
   - Change only what's necessary
   - Preserve existing behavior
   - Don't "improve" unrelated code

5. **Verify**:
   - Error no longer occurs
   - Tests pass
   - No new errors introduced
   - Build succeeds

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
- Minimal diffs only - no refactoring
