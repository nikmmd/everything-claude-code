# Operations Department

The operations department handles cross-cutting coordination, incident management, observability, institutional knowledge, and IT infrastructure routing. These agents work across all other departments — they're the connective tissue that keeps the organization running smoothly.

## When to Use Operations Agents

Use these agents when you need to:
- **Coordinate multiple agents** across departments for complex tasks
- **Handle distributed errors** with cascading failures across systems
- **Monitor and optimize** system and agent performance
- **Capture institutional knowledge** from workflows and decisions
- **Route IT operations** tasks to the right specialists

## Available Agents

### [**multi-agent-coordinator**](multi-agent-coordinator.md) - Chief of Staff
Advanced orchestration expert handling complex multi-department work. Decomposes tasks, matches agents to capabilities, manages parallel execution, tracks dependencies, and synthesizes results. Covers workflow automation, BPMN patterns, human-in-loop orchestration, and workload balancing.

**Use when:** A task spans multiple departments, you need parallel agent dispatch, or work requires dependency management across agents. This is the entry point for complex cross-cutting work.

### [**error-coordinator**](error-coordinator.md) - Incident Commander
Distributed error handling expert. Masters cascade prevention, failure correlation across components, recovery orchestration, and resilience patterns. Delegates single-component bugs to core `error-resolver`.

**Use when:** Failures span multiple components or services, cascading errors need containment, or you need coordinated recovery across a distributed system.

### [**performance-monitor**](performance-monitor.md) - Business Analytics
Performance and observability specialist. Tracks system metrics, detects anomalies, identifies bottlenecks, and recommends optimizations. Provides visibility into how the organization operates.

**Use when:** Monitoring system health, identifying performance bottlenecks, establishing observability infrastructure, or analyzing operational efficiency.

### [**knowledge-synthesizer**](knowledge-synthesizer.md) - Knowledge Manager
Institutional knowledge specialist. Extracts patterns from past work, captures lessons learned, builds decision logs, and surfaces best practices. Ensures the organization learns from experience.

**Use when:** Extracting patterns from completed work, building runbooks from incidents, capturing tribal knowledge, or synthesizing insights across multiple workflows.

### [**it-ops-orchestrator**](it-ops-orchestrator.md) - IT Operations Router
Meta-orchestrator for PowerShell/.NET/M365/infrastructure tasks. Routes ambiguous IT operations work to the right specialist agents.

**Use when:** A task spans multiple IT domains (AD, DNS/DHCP, Azure, M365, PowerShell modules) and needs routing to `m365-admin`, `powershell-module-architect`, or `security-reviewer`.

## Quick Selection Guide

| If you need to... | Use this agent |
|-------------------|----------------|
| Coordinate multi-department work | **multi-agent-coordinator** |
| Handle distributed failures | **error-coordinator** |
| Monitor and optimize performance | **performance-monitor** |
| Capture organizational knowledge | **knowledge-synthesizer** |
| Route IT ops tasks | **it-ops-orchestrator** |

## Common Operations Patterns

**Complex Cross-Department Project:**
- **multi-agent-coordinator** decomposes and routes work
- **knowledge-synthesizer** captures patterns for future projects
- **performance-monitor** tracks execution efficiency

**Incident Response:**
- **error-coordinator** for distributed failure correlation
- Core **error-resolver** for individual component fixes
- **knowledge-synthesizer** for post-incident runbook creation

**Organizational Learning:**
- **knowledge-synthesizer** extracts patterns from past work
- **performance-monitor** provides metrics and trends
- Core **doc-updater** integrates findings into documentation

**IT Infrastructure:**
- **it-ops-orchestrator** routes to specialists
- **powershell-module-architect** (Developer Experience) for PowerShell automation
- **m365-admin** (Specialized Engineering) for Microsoft 365 operations
