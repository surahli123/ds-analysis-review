# DS Analysis Review Agent — Repo Structure (v4)

## Overview

Two layers:
1. **Plugin** — Claude Code plugin with a lead orchestrator agent, two reviewer subagents, one entry command, shared skill
2. **Dev process** — session logs, decisions, changelog, backlog for build continuity

Same prompt content can be copy-pasted into Agent Studio for web distribution.

## Agent Architecture

```
User provides DS analysis (Confluence page or local markdown)
                    │
                    ▼
        ┌───────────────────────┐
        │   ds-review-lead      │  ← Orchestrator agent
        │                       │  - Understands the full review process
        │   1. Assess input     │  - Invoked via /ds-review:review or natural language
        │   2. Route to subs    │  - Asks user preferences (dimensions, audience)
        │   3. Synthesize       │  - Dispatches subagents
        │   4. Top-level assess │  - Synthesizes outputs into unified review
        └──────┬──────┬─────────┘  - Adds its own top-level assessment
               │      │
        ┌──────┘      └──────┐
        ▼                    ▼
┌──────────────┐  ┌───────────────────┐
│  analysis-   │  │  communication-   │
│  reviewer    │  │  reviewer         │
│              │  │                   │
│ methodology  │  │ structure & TL;DR │
│ logic        │  │ audience fit      │
│ completeness │  │ conciseness &     │
│ metrics      │  │   prioritization  │
│              │  │ actionability     │
└──────────────┘  └───────────────────┘
```

## Folder Structure

```
ds-analysis-review-agent/
│
├── CLAUDE.md                          # Project context (loaded every session)
├── CLAUDE.local.md                    # Personal overrides, gitignored
├── README.md                          # Project overview, install, roadmap
├── CHANGELOG.md                       # Updated every meaningful session
├── LICENSE
├── .gitignore
│
├── plugin/                            # ← Claude Code plugin
│   ├── .claude-plugin/
│   │   └── plugin.json               # Plugin manifest
│   │
│   ├── commands/
│   │   └── review.md                 # /ds-review:review — explicit entry point
│   │
│   ├── agents/
│   │   ├── ds-review-lead.md         # Orchestrator: routes, synthesizes, assesses
│   │   ├── analysis-reviewer.md      # Subagent: methodology, logic, completeness, metrics
│   │   └── communication-reviewer.md # Subagent: narrative, audience, visualization, actionability
│   │
│   ├── skills/
│   │   └── ds-review-framework/
│   │       └── SKILL.md              # Review framework, rubrics, audience personas
│   │                                  # Auto-activates when DS analysis detected
│   │
│   └── README.md                     # Plugin install & usage docs
│
├── dev/                               # ← Build process tracking (not distributed)
│   ├── backlog.md                     # Todo / In Progress / Done
│   ├── decisions/                     # Architecture Decision Records
│   │   └── 001-two-dimension-split.md
│   ├── sessions/                      # One log per Claude Code session
│   │   └── YYYY-MM-DD-description.md
│   └── specs/
│       └── v1-review-plugin.md        # v1 scope spec
│
└── .claude/                           # Project-level Claude Code config
    ├── settings.json
    └── rules/
        └── plugin-conventions.md
```

## How the Three Agents Work Together

### ds-review-lead.md (orchestrator)
The brain of the system. Responsibilities:
- **Explicit invocation only:** User runs `/ds-review:review` or uses natural language (no auto-detection in v1)
- **Intake:** Asks user which dimensions (analysis, communication, or both) and target audience
- **Dispatch:** Invokes the appropriate subagent(s) with the input and preferences
- **Synthesize:** Combines subagent outputs into a unified review
- **Top-level assessment:** Adds its own overall verdict — overall quality score, biggest strengths, highest-priority improvements, and a "ready to share?" recommendation

### analysis-reviewer.md (subagent)
Called by the lead agent. Reviews the analysis dimension:
- Statistical methodology rigor
- Logical reasoning soundness
- Completeness of insights
- Metric selection appropriateness

### communication-reviewer.md (subagent)
Called by the lead agent. Reviews the communication dimension:
- Structure & TL;DR
- Audience fit (tailored to exec / tech lead / peer DS based on user selection)
- Conciseness & prioritization
- Actionability of recommendations

## What Goes Where

| Content | Location | Why |
|---|---|---|
| Orchestration logic | `agents/ds-review-lead.md` | Agent decides routing and synthesis |
| Analysis review prompt | `agents/analysis-reviewer.md` | Standalone subagent, also copy-pasteable to Agent Studio |
| Communication review prompt | `agents/communication-reviewer.md` | Standalone subagent, also copy-pasteable to Agent Studio |
| Rubrics + personas | `skills/ds-review-framework/SKILL.md` | Shared knowledge, auto-loaded |
| User-facing entry point | `commands/review.md` | Explicit invocation path |

## CLAUDE.md Template

```markdown
# DS Analysis Review Agent

## What This Is
Claude Code plugin that reviews DS analyses across two dimensions:
- Analysis: methodology, logic, completeness, metrics
- Communication: narrative, audience fit, visualization, actionability

Uses a lead orchestrator agent (ds-review-lead) that dispatches two
reviewer subagents, synthesizes their outputs, and adds a top-level assessment.

Long-term: prompts are also deployed via Agent Studio for non-technical users.

## Agent Architecture
- ds-review-lead.md — orchestrator (invoked via /ds-review:review command or natural language, explicit only)
- analysis-reviewer.md — subagent for analysis dimension
- communication-reviewer.md — subagent for communication dimension
- ds-review-framework SKILL — shared rubrics and personas (auto-loaded)

## Current State
See dev/backlog.md for priorities.
See latest file in dev/sessions/ for where we left off.

## Pickup Instructions
Every session start:
1. Read dev/backlog.md for what to work on
2. Read the latest dev/sessions/ log for context
3. Check dev/decisions/ before making architectural choices

## Domain Context: ACTIVE
Search relevance and DS expertise applies. Use it.

## Session End Protocol
Before ending every session:
1. Update dev/backlog.md
2. Create dev/sessions/YYYY-MM-DD-description.md
3. Update CHANGELOG.md if anything shipped
4. Create dev/decisions/ADR if a design choice was made
```
