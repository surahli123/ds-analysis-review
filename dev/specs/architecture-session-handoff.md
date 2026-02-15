# Architecture Session Handoff: DS Analysis Review Agent

## Context

You are starting an architecture design session for a DS Analysis Review Agent — a Claude Code plugin that reviews data science analyses across two dimensions (analysis quality and communication effectiveness) using a 3-agent system.

Read these two documents before starting:

1. **PRD:** `PRD-DS-Analysis-Review-Agent.md` — the full product requirements document. This is the source of truth for what to build.
2. **Repo Structure:** `DS Analysis Review Agent — Repo Structure (v4).md` — the file/folder layout for the plugin.

## What This Session Should Produce

By the end of this session, you should have:

1. **Agent prompt files** — `.md` files for all 3 agents (ds-review-lead, analysis-reviewer, communication-reviewer) written to the repo structure locations. Each prompt should be under ~120 lines (see PRD Section 9.8 on the ~150 instruction constraint).

2. **Shared skill file** — `plugin/skills/ds-review-framework/SKILL.md` containing rubrics, deduction table, severity definitions, audience persona definitions, dimension boundary routing table, and common anti-patterns. This is the detailed content that keeps agent prompts slim.

3. **Architecture decisions** on the open questions listed below.

## Open Questions to Resolve

These are the highest-priority decisions for this session (from PRD Section 17):

### 1. Long Document Chunking (Highest Priority)
Early testing confirmed that long analyses (10+ pages) produce erroneous/incomplete output. How should the agent handle documents that exceed context limits? Options to evaluate:
- Lead agent pre-processes/summarizes before dispatch
- Chunking strategy with overlap
- Subagents receive full document but with focused instructions on which sections to prioritize
- Context window management per parallel subagent

### 2. Rubric Content Per Lens
What specific criteria score high vs low for each of the 8 lenses (4 analysis, 4 communication)? The PRD provides the lens definitions and deduction table — the architecture session needs to turn these into concrete checklists in the shared skill file. Start from Pedro Sant'Anna's domain-reviewer lens structure as a reference: https://github.com/pedrohcgs/claude-code-my-workflow

### 3. Quick Mode Architecture
How to hit 1-2 min latency while maintaining quality? Key tension: the plan-first step requires a user round trip. Quick mode may need to skip or streamline this. Also: does Quick mode run both subagents or just one? Does it use a lighter checklist?

### 4. Skill File vs Embedded Rubrics
Determine whether the target CLI agent (opencode or similar) supports auto-loaded skill files. If not, rubrics must be embedded in agent prompt files (see PRD Section 9.1.1 fallback). This decision affects how you write the prompts.

### 5. Partial/Minimal Input Handling
How should the agent handle cases where the user provides just a few bullet points or an incomplete draft? Minimum viable input threshold?

## Key Design Constraints

- **~150 instruction limit per agent** — Claude reliably follows 100-150 custom instructions. Agent prompts must be concise; rubric detail goes in shared skill file.
- **Parallel subagent execution** — analysis-reviewer and communication-reviewer run simultaneously (PRD Section 8.5).
- **Floor rules in scoring** — any CRITICAL finding caps verdict at Minor Fix; 2+ CRITICALs cap at Major Rework (PRD Section 10.2).
- **Workflow context awareness** — agent must handle both proactive (DS recommends) and reactive (DS measures/validates) modes. TL;DR and Actionability always heavily scored in both modes (PRD Section 6).
- **Report-only constraint** — agents review, they don't edit or rewrite.

## Reference Material

- Pedro Sant'Anna's workflow: https://psantanna.com/claude-code-my-workflow/
- Pedro's repo (agent examples): https://github.com/pedrohcgs/claude-code-my-workflow
- Full workflow guide: https://psantanna.com/claude-code-my-workflow/workflow-guide.html

## How to Start

1. Read both docs (PRD + repo structure)
2. Decide on skill file vs embedded rubrics (question #4) — this gates how you write everything else
3. Draft the shared skill file first (rubrics, deduction table, routing table)
4. Draft the 3 agent prompts
5. Address long document chunking strategy
6. Address Quick mode architecture
