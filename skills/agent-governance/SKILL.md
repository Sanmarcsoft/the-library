---
name: agent-governance
description: Meta-governance framework for multi-agent operations — defines sub-agent spawn rules, MCP access tiers, mandatory workflows (TDD, GitHub issues, security pyramid), RCA process for failures, and ChromaDB knowledge persistence across all agents.
---

# Agent Governance Framework

Defines what sub-agents can spawn, what tools they inherit, and the mandatory professional standards every agent must follow.

## Iron Law

**EVERY AGENT OPERATES UNDER GOVERNANCE. NO EXCEPTIONS.**

Every sub-agent spawned in the SanMarcSoft ecosystem MUST:
1. Have access to mcp2cli + Moneypenny MCP (31 tools)
2. Follow GitHub issue → branch → TDD → PR workflow
3. Use the-library for skill routing before doing work
4. Store learnings in ChromaDB after completing work
5. Trigger RCA on any failure

## Agent Hierarchy & Spawn Rules

```
Prime Agent (meta-router)
├── Meta Agent (governance enforcer)
│   ├── Determines which sub-agents a task needs
│   ├── Assigns MCP access tiers
│   └── Convenes RCA sessions on failure
│
├── Tier 1: Autonomous Agents (can spawn sub-agents)
│   ├── devops-workflow-enforcer — spawns: scrum-master, github-project-manager
│   ├── planner — spawns: plan-reviewer, builder, scout
│   ├── builder — spawns: reviewer, red-team
│   └── software-architect — spawns: planner, builder, documenter
│
├── Tier 2: Specialist Agents (execute, don't spawn)
│   ├── scout — codebase exploration
│   ├── reviewer — code review
│   ├── red-team — security testing
│   ├── documenter — documentation
│   ├── scrum-master — GitHub project management
│   ├── bowser — browser automation
│   └── uv-agent — Python toolchain
│
└── Tier 3: Utility Skills (invoked, not spawned)
    ├── wolfram — computation (mcp2cli --toon)
    ├── mcp2cli-first — token-efficient MCP access
    ├── frontend-design — UI/UX
    ├── proofshot — visual verification
    └── ruff/ty/uv — Python quality
```

## MCP Access Matrix

Every agent gets these tools via environment:

| MCP Server | Access Method | What It Provides |
|---|---|---|
| **Moneypenny** | `mcp2cli --mcp http://moneypenny:3100/mcp` or `http://10.0.0.112:3101/mcp` | 31 tools: email, calendar, contacts, GitHub, portfolio, Wolfram, SMS, voice, Matrix |
| **Wolfram Alpha** | Via Moneypenny `wolfram_query`/`wolfram_short`/`wolfram_compute` | Computation, math, data lookup, Python/sympy fallback |
| **ChromaDB** | `mcp2cli --mcp chromadb` or raw MCP | Portfolio data, session memories, knowledge base |
| **SearxNG** | `mcp2cli --mcp searxng` | Web search |

### mcp2cli Rule
**ALL MCP access MUST use `mcp2cli --toon` first.** Raw MCP tool calls are fallback only. This saves 96-99% tokens per call.

## Mandatory Workflows

### 1. GitHub Issue → Branch → TDD → PR Pipeline

```
1. Create GitHub issue with acceptance criteria
2. Create feature branch: feature/<issue-number>-<description>
3. Write failing test (RED)
4. Implement minimal solution (GREEN)
5. Refactor (REFACTOR)
6. Commit with issue reference: "feat: description (#123)"
7. Push branch, create PR
8. Code review (reviewer agent)
9. Security review (red-team agent)
10. Merge to dev → testing → production (gated)
```

**Enforced by:** `agent:devops-workflow-enforcer` — auto-invoked on any `create branch`, `commit`, `deploy`, `merge` trigger.

### 2. Security Pyramid

```
Layer 4: Application Security (OWASP top 10, input validation)
Layer 3: Authentication & Authorization (Clerk JWT, session handling)
Layer 2: Infrastructure Security (Docker, networking, secrets management)
Layer 1: Data Security (encryption at rest, TLS, PKI)
Layer 0: Supply Chain (dependency auditing, Nix reproducibility)
```

**Every PR must pass:** red-team agent security scan before merge.

### 3. TDD Cycle

```
RED:    Write failing test that defines expected behavior
GREEN:  Write minimal code to pass the test
REFACTOR: Clean up while keeping tests green
COMMIT: Atomic commit with test + implementation
```

**Enforced by:** `superpowers:test-driven-development` skill — auto-invoked on any implementation task.

### 4. Specification Development

```
1. Brainstorm (superpowers:brainstorming) → design doc
2. Spec review (spec-document-reviewer) → validated spec
3. Plan (superpowers:writing-plans) → implementation plan
4. Plan review (plan-document-reviewer) → validated plan
5. Execute (superpowers:subagent-driven-development) → code
6. Review (superpowers:requesting-code-review) → approved code
7. Finish (superpowers:finishing-a-development-branch) → merged
```

## Root Cause Analysis (RCA) Protocol

### When to Trigger RCA

- Build failure after merge
- Test regression
- Production incident
- Agent task failure (BLOCKED status)
- Same error occurring twice
- Performance degradation

### RCA Multi-Agent Session

When a failure occurs, the meta-agent convenes:

```
1. SCOUT agent: Gather evidence (logs, diffs, timeline)
2. REVIEWER agent: Analyze code changes that may have caused failure
3. RED-TEAM agent: Check for security implications
4. PLANNER agent: Design the fix with root cause addressed
5. BUILDER agent: Implement the fix (TDD)
6. DOCUMENTER agent: Write the post-mortem to ChromaDB
```

### RCA Document Template (stored in ChromaDB)

```json
{
  "collection": "portfolio_rca",
  "id": "rca-YYYY-MM-DD-<slug>",
  "document": "## RCA: <title>\n### Timeline\n### Root Cause\n### Impact\n### Fix\n### Prevention\n### Lessons Learned",
  "metadata": {
    "type": "rca",
    "date": "YYYY-MM-DD",
    "severity": "critical|high|medium|low",
    "repo": "Sanmarcsoft/<repo>",
    "issue": "#<number>",
    "agents_involved": ["scout", "reviewer", "builder"],
    "status": "resolved|investigating|monitoring"
  }
}
```

## ChromaDB Knowledge Persistence

### Collections for Agent Knowledge

| Collection | Purpose | Who Writes | Who Reads |
|---|---|---|---|
| `portfolio_projects` | Project metadata | portfolio_sync | All agents |
| `portfolio_issues` | GitHub issues | portfolio_sync | All agents |
| `portfolio_infra` | Container state | portfolio_sync | All agents |
| `portfolio_email` | Email state | portfolio_sync | All agents |
| `portfolio_calendar` | Calendar events | portfolio_sync | All agents |
| `portfolio_rca` | RCA post-mortems | RCA sessions | All agents |
| `agent_sessions` | Session summaries | All agents at session end | All agents at session start |
| `agent_skills` | Skill improvement tracking | prime-agent | All agents |
| `agent_decisions` | Architecture decisions (ADRs) | software-architect | All agents |

### Session Memory Protocol

At session end, every agent SHOULD store a summary:

```bash
# Via Moneypenny MCP
mcp2cli --mcp moneypenny --toon portfolio-query '{"collection":"agent_sessions","where":{"session_id":"<id>"}}'

# Store new session
# (requires ChromaDB write access — use chroma_add_documents)
```

### Proactive Skill Creation

When an agent discovers a reusable pattern:
1. Check the-library for existing skill
2. If none exists, create a new skill following `/library` conventions
3. Push to the-library repo
4. Update triggers.yaml for auto-routing

## Sub-Agent Context Injection

When spawning a sub-agent, the parent MUST include:

```
## Environment

You have access to these MCP servers via mcp2cli:
- Moneypenny: `mcp2cli --mcp http://10.0.0.112:3101/mcp --toon <tool>`
  Tools: email, calendar, contacts, GitHub, portfolio, Wolfram, SMS, voice, Matrix
- ChromaDB: `mcp2cli --mcp chromadb --toon <tool>`
- SearxNG: `mcp2cli --mcp searxng --toon web_search`

## Mandatory Standards
- TDD: Write failing test first, then implement
- GitHub: All work requires an issue with acceptance criteria
- Branches: feature/<issue>-<description>, never commit to main directly
- Security: OWASP top 10 awareness, input validation, no secrets in code
- On failure: Report BLOCKED with full context for RCA

## Library
Check /library before starting work. Existing skills save time.
```
