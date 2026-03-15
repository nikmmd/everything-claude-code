# Agent Orchestration

## Architecture

```
Rules (always loaded)     → passive guidelines
Commands (/slash)         → user-invoked workflows, delegate to agents
  └── Agents              → specialized subprocesses that do the work
Skills (SKILL.md)         → auto-triggered context injection
Plugins                   → third-party extensions (may add skills/agents)
```

## Command → Agent Mapping

Each `/command` delegates to its matching agent:

| Command | Agent | Purpose |
|---------|-------|---------|
| `/plan` | planner | Implementation planning |
| `/tdd` | tdd-agent | Test-driven development |
| `/code-review` | code-reviewer | Quality & security review |
| `/bugfix` | error-resolver | Diagnose & fix errors |
| `/e2e` | e2e-runner | Playwright E2E testing |
| `/refactor-clean` | refactor-cleaner | Dead code cleanup |
| `/test-coverage` | tdd-agent | Coverage analysis |

## Standalone Agents (no command)

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| architect | System design | Architectural decisions, trade-off analysis |
| database-designer | DB schema & optimization | Schema design, query optimization, migrations |
| security-reviewer | Deep vulnerability analysis | Before commits, OWASP review |
| doc-updater | Documentation & codemaps | Updating docs and architecture maps |

## Auto-Triggered Skills

| Skill | Trigger | Related Agent |
|-------|---------|---------------|
| security-review | Auth, user input, secrets, API endpoints | security-reviewer |
| api-design | Creating/modifying API routes | — |
| docker-patterns | Dockerfiles, compose, container config | — |
| postgres-patterns | DB queries, schema changes, migrations | database-designer |
| deployment-patterns | CI/CD, deploy scripts, infra changes | — |

## Proactive Agent Usage

Use agents without being asked:
1. Complex feature requests → **planner** agent
2. Code just written/modified → **code-reviewer** agent
3. Bug fix or new feature → **tdd-agent** agent
4. Architectural decision → **architect** agent
5. Database schema/query work → **database-designer** agent
6. Build/runtime errors → **error-resolver** agent

## Parallel Execution

ALWAYS launch independent agents in parallel:

```
# GOOD: Parallel
Launch 3 agents simultaneously:
1. Security analysis of auth.ts
2. Performance review of cache system
3. Type checking of utils.ts

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```
