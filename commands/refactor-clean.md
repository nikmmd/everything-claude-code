---
description: Safely identify and remove dead code, unused exports, and redundant dependencies with test verification.
---

# Refactor Clean

Delegates to the **refactor-cleaner** agent for dead code cleanup.

## Workflow

1. Run dead code analysis (knip, depcheck, ts-prune)
2. Categorize findings:
   - **SAFE**: Unused utilities, test files
   - **CAUTION**: API routes, components
   - **DANGER**: Config files, entry points
3. Propose safe deletions only
4. Before each deletion: run tests → apply → re-run tests → rollback if fail
5. Show cleanup summary

## Rules

- Never delete without running tests first
- Start with SAFE category, escalate with user approval
- Generate report in `.reports/dead-code-analysis.md`
