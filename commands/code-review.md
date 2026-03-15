---
description: Review uncommitted code changes for security vulnerabilities, code quality, and best practices.
---

# Code Review

Delegates to the **code-reviewer** agent for comprehensive review of uncommitted changes.

## Workflow

1. Get changed files: `git diff --name-only HEAD`
2. Read and analyze each changed file
3. Generate findings report with severity and suggested fixes
4. Block commit if CRITICAL or HIGH issues found

## What Gets Checked

**Security (CRITICAL):**
- Hardcoded credentials, API keys, tokens
- SQL injection, XSS vulnerabilities
- Missing input validation
- Path traversal risks

**Code Quality (HIGH):**
- Functions > 50 lines, files > 800 lines
- Nesting depth > 4 levels
- Missing error handling
- `console.log` statements left in code

**Best Practices (MEDIUM):**
- Mutation patterns (should use immutable)
- Missing tests for new code
- Accessibility issues

## Output Format

Each finding includes:
- **Severity**: CRITICAL / HIGH / MEDIUM / LOW
- **File**: Location and line numbers
- **Issue**: Description
- **Fix**: Suggested resolution

## Related

- `/security-review` — Deep security analysis (auto-triggered)
- `/test-coverage` — Verify test coverage
- `/tdd` — Write tests for flagged code
