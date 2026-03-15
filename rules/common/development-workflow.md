# Development Workflow

> Extends [git-workflow.md](./git-workflow.md) with the full feature development process before git operations.

## Feature Implementation Workflow

0. **Research & Reuse** _(mandatory before any new implementation)_
   - **GitHub code search first:** Run `gh search repos` and `gh search code` to find existing implementations, templates, and patterns before writing anything new.
   - **Library docs second:** Use Context7 MCP or primary vendor docs to confirm API behavior, package usage, and version-specific details.
   - **Check package registries:** Search npm, PyPI, Go modules, Maven Central before writing utility code. Prefer battle-tested libraries over hand-rolled solutions.
   - **Search for adaptable implementations:** Look for open-source projects that solve 80%+ of the problem and can be forked, ported, or wrapped.
   - Prefer adopting or porting a proven approach over writing net-new code.

1. **Plan First**
   - Use **planner** agent (`/plan`) for implementation plan
   - Identify dependencies and risks
   - Break down into phases
   - WAIT for user confirmation before coding

2. **TDD Approach**
   - Use **tdd-agent** (`/tdd`)
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage

3. **Code Review**
   - Use **code-reviewer** agent (`/code-review`) immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Follow conventional commits format
   - See [git-workflow.md](./git-workflow.md) for commit message format and PR process
