# Agent Roster

This is the organizational directory for all available agents. It serves two purposes:

1. **Direct invocation** — You know what you need. Find the agent, call it. No ceremony required.
2. **Routed delegation** — You have a fuzzy or cross-cutting task. Use the department structure and lead agents to triage and delegate.

Both modes are always available. The org chart is a routing aid, not a chain of command.

---

## How to Use This Roster

**Quick lookup:** Search for a keyword (e.g., "payment", "testing", "kubernetes") — each agent has a one-line capability summary.

**Routing a complex task:** Identify the department, start with the department lead. Leads know their team's capabilities and will delegate or collaborate with peer departments.

**Cross-department work:** When a task spans multiple departments (e.g., "build an ML feature with a database schema and API docs"), either:
- Invoke each specialist directly in parallel, or
- Start with `multi-agent-coordinator` (Operations) to decompose and route the work

---

## Engineering Department

Core engineering agents. These run the development lifecycle: plan → design → implement → test → review → ship.

| Agent | Role | Capability |
|-------|------|-----------|
| **architect** | VP Engineering | System design, trade-off analysis, scalability planning. Delegates domain work to specialists. |
| **planner** | Staff Engineer | Implementation plans, risk assessment, step breakdown. Recommends agents per phase. |
| **tdd-agent** | QA Lead | Test-first methodology, 80%+ coverage. Delegates E2E to e2e-runner. |
| **code-reviewer** | Code Quality Lead | Quality, security, and maintainability review. Escalates to security-reviewer. |
| **e2e-runner** | QA Automation | Playwright E2E tests, flaky test quarantine, artifact management. |
| **error-resolver** | Incident Response | Single-component bugs, build errors, runtime failures. Minimal diffs. |
| **error-coordinator** | Incident Commander | Distributed failures, cascade prevention, cross-system recovery. |
| **database-designer** | Data Architect | Schema design, query optimization, migration planning, index strategy. |
| **security-reviewer** | Security Lead | OWASP Top 10, secrets detection, dependency CVEs, threat modeling. |
| **refactor-cleaner** | Tech Debt Lead | Dead code removal via knip/depcheck/ts-prune. Safe cleanup. |
| **dependency-manager** | Supply Chain | Package updates, version conflicts, CVE remediation, bundle optimization. |

**Lead:** `architect` — consult for system-level decisions, delegates domain work downward.

---

## 05 — Data & AI Department

Machine learning, data pipelines, and AI systems from research to production.

| Agent | Role | Capability |
|-------|------|-----------|
| **ai-engineer** | AI Lead | End-to-end AI systems, model deployment, inference optimization. |
| **llm-architect** | LLM Specialist | RAG, fine-tuning, prompt strategies, LLM serving infrastructure. |
| **ml-engineer** | ML Systems | Training pipelines, model optimization, serving, automated retraining. |
| **mlops-engineer** | ML Platform | CI/CD for ML, model registry, experiment tracking, GPU orchestration. |
| **data-engineer** | Data Infrastructure | ETL/ELT, streaming, data warehousing, Spark/Kafka/Airflow. |
| **data-analyst** | Analytics | Statistical analysis, dashboards, business intelligence, reporting. |
| **data-scientist** | Research | Predictive modeling, experimentation, advanced analytics. |
| **nlp-engineer** | NLP Specialist | Text processing, language models, NER, sentiment, translation. |
| **prompt-engineer** | Prompt Design | Prompt optimization, evaluation, testing for production LLM systems. |
| **reinforcement-learning-engineer** | RL Specialist | RL environments, reward shaping, policy optimization, sim-to-real. |

**Lead:** `ai-engineer` — consult for AI system architecture, routes to ML/NLP/data specialists.

---

## 06 — Developer Experience Department

Developer productivity, tooling, modernization, and build systems.

| Agent | Role | Capability |
|-------|------|-----------|
| **dx-optimizer** | DX Lead | Workflow analysis, friction identification, productivity measurement. |
| **build-engineer** | Build Systems | Compilation optimization, caching, monorepo builds, CI/CD pipelines. |
| **tooling-engineer** | Dev Tools | CLIs, code generators, IDE extensions, developer platforms. |
| **legacy-modernizer** | Migration Lead | Incremental modernization, framework migration, tech debt strategy. |
| **git-workflow-manager** | VCS Lead | Branching strategies, merge workflows, Git automation. |
| **mcp-developer** | MCP Specialist | Model Context Protocol servers/clients, AI-tool integrations. |
| **readme-generator** | README Specialist | Repository READMEs from code scanning, zero-hallucination. |
| **slack-expert** | Slack Platform | @slack/bolt, Block Kit, OAuth, Events API, Slack integrations. |
| **powershell-module-architect** | PowerShell Modules | Module design, profile optimization, cross-version PS compatibility. |
| **powershell-ui-architect** | PowerShell UIs | WinForms, WPF, Metro dashboards, TUIs for PowerShell tools. |

**Lead:** `dx-optimizer` — consult for developer workflow improvement, routes to build/tooling specialists.

---

## 07 — Specialized Engineering

Domain-specific engineering requiring deep vertical expertise.

| Agent | Role | Capability |
|-------|------|-----------|
| **fintech-engineer** | Fintech Lead | Banking, compliance, KYC/AML, trading platforms. |
| **payment-integration** | Payments | Gateway integration, PCI DSS, checkout flows, subscriptions. |
| **blockchain-developer** | Web3 | Solidity, smart contracts, DeFi, NFTs, on-chain architecture. |
| **mobile-app-developer** | Mobile | iOS/Android native and cross-platform, app store deployment. |
| **game-developer** | Gaming | Game engines, real-time networking, multiplayer, game physics. |
| **embedded-systems** | Firmware | Microcontrollers, RTOS, hardware interfaces, power optimization. |
| **iot-engineer** | IoT | Device protocols, edge computing, fleet management, sensor data. |
| **m365-admin** | Microsoft 365 | Exchange, Teams, SharePoint, Graph API, license management. |
| **api-documenter** | API Docs | OpenAPI specs, interactive portals, SDK documentation. |
| **quant-analyst** | Quantitative | Trading algorithms, risk models, backtesting, derivatives pricing. |
| **risk-manager** | Risk | Enterprise risk assessment, compliance, control frameworks. |
| **seo-specialist** | SEO | Technical SEO, structured data, content strategy, search rankings. |

**No single lead** — these are independent vertical specialists invoked by domain need.

---

## 08 — Business & Product Department

Product strategy, project management, content, legal, and customer-facing functions.

| Agent | Role | Capability |
|-------|------|-----------|
| **product-manager** | Product Lead | Product vision, roadmap, feature prioritization, market fit. |
| **project-manager** | Program Manager | Timelines, resource planning, stakeholder coordination, risk tracking. |
| **scrum-master** | Agile Coach | Sprint facilitation, velocity improvement, impediment removal. |
| **business-analyst** | Requirements | Business process analysis, user stories, requirement translation. |
| **technical-writer** | Documentation | User guides, tutorials, knowledge bases, developer onboarding. |
| **content-marketer** | Content | Blog posts, content strategy, SEO writing, audience engagement. |
| **sales-engineer** | Pre-Sales | Technical demos, POCs, objection handling, solution architecture. |
| **customer-success-manager** | Customer Success | Onboarding, retention, health scoring, upsell identification. |
| **ux-researcher** | User Research | Usability testing, user interviews, persona development, surveys. |
| **legal-advisor** | Legal | Contracts, privacy/GDPR, IP strategy, regulatory compliance. |
| **license-engineer** | Licensing | OSS license selection, dependency compliance, dual-licensing. |
| **wordpress-master** | WordPress | Theme/plugin dev, WooCommerce, performance, security hardening. |

**Lead:** `product-manager` — consult for product direction, routes to project/content/research specialists.

---

## 09 — Operations Department

Cross-cutting coordination, incident management, observability, and institutional knowledge.

| Agent | Role | Capability |
|-------|------|-----------|
| **multi-agent-coordinator** | Chief of Staff | Task decomposition, parallel agent dispatch, workflow orchestration. |
| **error-coordinator** | Incident Commander | Distributed error correlation, cascade prevention, recovery orchestration. |
| **performance-monitor** | Business Analytics | System metrics, anomaly detection, bottleneck analysis, optimization. |
| **knowledge-synthesizer** | Knowledge Manager | Pattern extraction, lessons learned, institutional memory, best practices. |
| **it-ops-orchestrator** | IT Operations | Routes PowerShell/.NET/M365/infra tasks to the right specialist. |

**Lead:** `multi-agent-coordinator` — consult when a task spans multiple departments or agents.

---

## 10 — Research Department

Market intelligence, competitive analysis, trend forecasting, and deep research.

| Agent | Role | Capability |
|-------|------|-----------|
| **research-analyst** | Research Lead | Multi-source synthesis, comprehensive reports, trend identification. |
| **competitive-analyst** | Competitive Intel | Competitor strategy, benchmarking, SWOT, positioning analysis. |
| **market-researcher** | Market Analysis | Market sizing, segmentation, consumer behavior, entry strategy. |
| **trend-analyst** | Forecasting | Emerging patterns, weak signal detection, future scenario planning. |
| **search-specialist** | Information Retrieval | Advanced search, query optimization, precision fact-finding. |
| **project-idea-validator** | Idea Validator | Brutally honest go/no-go assessment, competitor teardown, MVP scoping. |
| **scientific-literature-researcher** | Literature Review | PubMed/Scholar search, evidence synthesis, structured data extraction. |

**Lead:** `research-analyst` — consult for research strategy, routes to competitive/market/trend specialists.

---

## Documentation (cross-cutting)

Documentation spans departments. Use the specialist that matches the output type:

| Agent | What it produces |
|-------|-----------------|
| **doc-updater** | Codemaps from AST analysis, README sync from code |
| **readme-generator** | Maintainer-ready READMEs from repo scanning |
| **api-documenter** | OpenAPI specs, interactive API portals, SDK docs |
| **technical-writer** | User guides, tutorials, knowledge bases |

---

## Routing Quick Reference

| Task shape | Start with |
|-----------|-----------|
| "Build a feature" | `planner` → recommended agents per phase |
| "Fix a bug" | `error-resolver` (single) or `error-coordinator` (distributed) |
| "Review this code" | `code-reviewer` → escalates as needed |
| "Design the architecture" | `architect` → delegates domain work |
| "I need ML/AI work" | `ai-engineer` → routes within Data & AI |
| "Improve our dev workflow" | `dx-optimizer` → routes within DX |
| "Product decision needed" | `product-manager` → routes within Business |
| "Research this topic" | `research-analyst` → routes within Research |
| "Complex multi-department task" | `multi-agent-coordinator` → decomposes and routes |
| "What did we learn from X?" | `knowledge-synthesizer` → extracts patterns |
| "Security review" | `security-reviewer` → escalates to domain specialists |
| "I know exactly what I need" | Call the agent directly. No routing needed. |
