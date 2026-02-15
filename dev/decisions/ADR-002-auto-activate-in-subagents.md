# ADR-002: auto_activate Skill Availability in Subagents

**Date:** 2026-02-14
**Status:** Accepted

## Context
Subagents are dispatched via Task tool. SKILL.md has `auto_activate: true`.
All three agent prompts reference SKILL.md sections for deduction tables, severity
definitions, routing rules, and audience personas.

## Test Result
Subagents dispatched via Task tool CANNOT see auto_activate skills. The skill is NOT
injected into the subagent's context automatically. However, subagents CAN read the
SKILL.md file directly using the Read tool.

## Decision
The lead agent's dispatch payload (Step 7) always instructs subagents to read SKILL.md
from disk before beginning their review. This is the default behavior, not a fallback.

**Why Read tool over embedding:**
- SKILL.md is ~209 lines — embedding it in every dispatch doubles payload size
- The Read tool is reliable and the file path is stable
- This keeps the dispatch payload focused on the review request
- If the file read fails, the subagent can report the error and the lead agent can
  fall back to re-dispatching with embedded content

## Consequences
- Lead agent Step 7 dispatch payload updated to include Read instruction
- No changes needed to subagent prompts (they already reference SKILL.md by section)
- auto_activate: true is kept in SKILL.md frontmatter for potential future plugin
  system support, but is not relied upon for subagent contexts
