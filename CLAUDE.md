# DS Productivity Agents

## What This Is
A suite of Claude Code agents for DS productivity in Search Relevance:
- **DS Analysis Review Agent:** Reviews DS analyses across methodology, logic, communication, and domain expertise
- **SQL Review Agent:** (Q2 2026) Reviews SQL queries for syntax and domain-specific patterns
- **Search Metric Analysis Agent:** (Q2 2026) Analyzes search metrics and generates insights

All agents share domain knowledge infrastructure for consistent Search Relevance expertise.

## Agent Architecture

### DS Analysis Review
- ds-review-lead.md — orchestrator (invoked via /ds-review command)
- analysis-reviewer.md — subagent for analysis dimension
- communication-reviewer.md — subagent for communication dimension
- domain-expert-reviewer.md — subagent for domain dimension (v0.5+)

### Shared Skills
- ds-review-framework — shared rubrics and personas (auto-loaded)
- search-domain-knowledge — Search Relevance domain expertise (v0.5+)

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
