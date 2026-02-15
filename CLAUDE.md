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
