---
description: Restate requirements, assess risks, and create step-by-step implementation plan. WAIT for user CONFIRM before touching any code.
---

# Plan

Delegates to the **planner** agent for implementation planning.

## Workflow

1. **Restate requirements** — Clarify what needs to be built
2. **Break into phases** — Specific, actionable implementation steps
3. **Identify dependencies** — Between components and external services
4. **Assess risks** — Potential blockers with severity (HIGH/MEDIUM/LOW)
5. **Estimate complexity** — Per phase and overall
6. **WAIT for confirmation** — Do NOT proceed until user explicitly approves

## When to Use

- Starting a new feature
- Significant architectural changes
- Complex refactoring across multiple files
- Ambiguous or unclear requirements

## Rules

- **CRITICAL**: Never write code before user confirms the plan
- User can respond with "modify: [changes]" or "different approach: [alt]"
- Break large features into phases that can be implemented and tested independently

## Related

- `/tdd` — Implement with test-driven development after plan approval
- `/code-review` — Review completed implementation
- `/e2e` — Test critical user journeys
