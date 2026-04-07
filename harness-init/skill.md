---
name: harness-init
description: Claude Code agent infrastructure setup — generates rules, hooks, memory structure, and agent routing from an interview. Turns 6 weeks of harness trial-and-error into a 5-minute conversation.
user_invocable: true
---

# Harness Init — Claude Code Agent Infrastructure Setup

## Purpose
Set up the full Claude Code harness layer for a project — rules, hooks, memory,
agent routing, and initial skills. This is NOT project scaffolding (use `/project-init` for that).
This is the AI orchestration layer that makes Claude Code sessions productive from day one.

Extracted from 6 weeks of iterative harness engineering on a production system
(98K LOC, 12 agents, 1275 tests).

---

## Phase 0: Prerequisites

Check if `CLAUDE.md` exists in the project root.
- If yes → read it for context (Hard Rules, stack, conventions)
- If no → recommend running `/project-init` first, but don't block

Check if `~/.claude/` global structure exists.
- If yes → read existing rules to avoid conflicts
- If no → this will be the first setup

---

## Phase 1: Interview (one question at a time)

### Q1 — Agent Complexity
```
How complex is your AI agent setup?

- Minimal: Claude Code as a smart assistant — just rules + memory
- Moderate: Multiple review agents (code review, security, testing)
- Orchestrated: Multi-agent system with routing, sub-agents, parallel execution
- Custom: something else (describe it)

This determines how many agent definitions and routing rules to generate.
```

### Q2 — Review Gates
```
What quality gates do you want before code ships?

- None (move fast)
- Basic: code review only
- Standard: code review + test validation
- Strict: code review + security review + verification checklist + build validation
- Custom: pick your own

Principle: Every gate you add slows you down but catches real bugs.
Start with fewer gates, add more after your first production incident.

Available gates:
  code-reviewer — finds issues, never fixes them directly
  security-reviewer — secrets, injection, OWASP Top 10
  verification — mandatory checklist before declaring "done"
  build-error-resolver — fixes build/type errors only
  quick-validator — fast pytest + lint pass
  database-reviewer — SQL, schema, migration review
```

### Q3 — Memory Strategy
```
How should context persist between sessions?

- Session-only: no persistence, start fresh every time
- Manual: MEMORY.md + session-handoff file (human-maintained)
- Structured: auto-checkpoint with handoff file + compact hooks
- External: MCP memory server (e.g., mempalace, custom)

Principle: Start with "Structured" — it's the sweet spot.
Manual gets stale. External adds infrastructure overhead.

Structured gives you:
  memory/MEMORY.md — project knowledge base
  memory/session-handoff-LATEST.md — inter-session continuity
  checkpoint-compact skill — save state before /compact
```

### Q4 — Lifecycle Hooks
```
What should happen automatically at session boundaries?

- SessionStart: auto-load handoff file? show project status?
- PreCompact: remind to save state? auto-checkpoint?
- Stop: remind to checkpoint? export session log?
- Custom: file watchers, test runners, deploy triggers?

Principle: Hooks run in the harness (not in Claude).
They execute shell commands — keep them fast and idempotent.
A slow SessionStart hook delays every conversation start.
```

### Q5 — Rules Hierarchy
```
What rules should Claude always follow in this project?

Tiers (higher = more important, cannot be overridden):

Tier 0 — Immutable: hard rules that never bend
  Examples: "never execute real trades", "reject if data missing"

Tier 1 — Mandatory workflow: always runs
  Examples: "verification after every code change"

Tier 2 — Process: development workflow order
  Examples: "brainstorming before implementation"

Tier 3 — Quality: review standards
  Examples: "code review, security review"

Tier 4 — Style: output formatting
  Examples: "concise responses", "Korean conversation, English code"

Which tiers do you need? What goes in each?
```

### Q6 — Initial Skills
```
Which skills should be pre-installed?

Core (recommended for all projects):
  □ checkpoint-compact — session state preservation

Development:
  □ project-init — new project scaffolding
  □ pre-push — security + quality pipeline before git push

Specialized:
  □ write-post — content generation with evaluation loop
  □ deep-research — multi-source research pipeline
  □ Custom skill (describe what you need)

Or: "none, I'll add them as needed"
```

---

## Phase 2: Harness Summary

Present the decided configuration:

```
Harness Configuration:
- Complexity: [minimal / moderate / orchestrated]
- Review gates: [list]
- Memory: [strategy]
- Hooks: [list]
- Rule tiers: [which ones, key rules]
- Skills: [list]

Scope:
- [global (~/.claude/) / project-level (.claude/) / both]

Estimated files to generate: [count]
```

Confirm before generating.

---

## Phase 3: File Generation

### 3-1. Rules (rules/*.md or .claude/rules/)

Decide scope: global (`~/.claude/rules/`) vs project-level (`.claude/rules/`).
Global = applies to ALL projects. Project-level = this project only.

**ai-constitution.md** (Tier 0 — always generate):
```markdown
# AI Constitution — [Project Name]

## Core Identity
[From Q5 Tier 0 answers]

## Truth & Clarity
1. Unverifiable information → must state "unknown"
2. No fabrication of data or sources
3. Confidence proportional to evidence strength

## Execution Discipline
1. Answer first, reasoning second
2. No unrequested features
3. If unsure, say so — never guess confidently
```

**agents.md** (if complexity >= moderate):
```markdown
# Agent Orchestration

## Available Agents
[Based on Q2 gate selections]

## Routing Rules
[Based on Q1 complexity level]

## Tier Priorities
[From Q5]
```

**output-style.md** (Tier 4):
```markdown
# Output Style
- No sycophantic openers or closing fluff
- Be concise. Short answer when short is enough.
- If unsure, say so. Never guess.
```

**development-workflow.md** (if Q2 != none):
```markdown
# Development Workflow
[Review gate pipeline based on Q2 answers]
```

### 3-2. Hooks (settings.json injection)

Generate hooks config based on Q4 answers.
**IMPORTANT**: Read existing `~/.claude/settings.json` first. Merge — never overwrite.

```json
{
  "hooks": {
    "SessionStart": [{"hooks": [{"type": "command", "command": "[from Q4]"}]}],
    "PreCompact": [{"hooks": [{"type": "command", "command": "[from Q4]"}]}],
    "Stop": [{"hooks": [{"type": "command", "command": "[from Q4]"}]}]
  }
}
```

### 3-3. Memory Structure

If Q3 = structured or manual:

```
memory/
├── MEMORY.md                      # project knowledge base
└── session-handoff-LATEST.md      # inter-session state
```

**MEMORY.md template:**
```markdown
# [Project Name] — Project Memory

## Current Status
- [version, language, location, test count]

## Key Files & Architecture
- Entry: [main entrypoint]
- [key modules]

## Workflow & Conventions
- [from CLAUDE.md Dev Conventions]

## Learnings
- [empty — populated during development]
```

**session-handoff-LATEST.md template:**
```markdown
# Session Handoff — [Project Name]

## Next Actions (priority order)
1. [empty — populated at session end]

## Open Decisions
- [empty]

## Remaining Issues
- [empty]

## System Understanding
- [empty — key discoveries go here]
```

### 3-4. Agent Definitions (if complexity = orchestrated)

Generate agent .md files in `~/.claude/agents/` or `.claude/agents/`:

Each agent file follows:
```markdown
---
name: [agent-name]
description: [one line]
---

# [Agent Name]

## Role
[What this agent does]

## Scope
[What it does / does NOT do]

## Inputs
[What it needs to start]

## Outputs
[What it produces]

## Handoff
[When to delegate to another agent]
```

---

## Phase 4: Verification

After generating all files:

```
□ Rules don't conflict with existing global rules
□ Hooks config merged (not overwritten) into settings.json
□ Memory structure created
□ CLAUDE.md references are consistent with rules
□ No duplicate agent definitions
□ Skills copied to correct location
```

Any failure → fix and re-verify.

---

## Phase 5: Refinement Loop

```
Harness generated. Review the structure.

Adjustable:
- Add/remove review gates
- Change memory strategy
- Modify hook triggers
- Add rules to any tier
- Add/remove skills

Approve → files confirmed
[change request] → apply and regenerate
```

---

## Scope Decision Guide

| Situation | Scope | Why |
|-----------|-------|-----|
| Personal conventions (style, commit format) | Global (~/.claude/) | Applies everywhere |
| Project-specific rules (domain invariants) | Project (.claude/) | Only this codebase |
| Review agents | Global | Reusable across projects |
| Domain agents (trading, medical, etc.) | Project | Context-specific |
| Memory | Project | Different state per project |
| Hooks | Global | Session lifecycle is universal |

---

## Principles Embedded in This Skill

- **Harness before code** — the orchestration layer determines how productive every session will be
- **Tier 0 rules are immutable** — no agent, skill, or prompt can override them
- **Hooks must be fast** — slow SessionStart = slow every conversation
- **Merge, never overwrite** — settings.json has other configs; destroying them is catastrophic
- **Start with fewer gates** — add review steps after the first bug that would have been caught
- **Memory is project-scoped** — global memory pollutes unrelated projects
- **Agent scope boundaries** — agents that do everything end up doing nothing well
