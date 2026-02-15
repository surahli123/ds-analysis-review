# Session: Real-World Testing & Calibration Discovery

**Date:** 2026-02-15
**Previous session:** `2026-02-15-v03-implementation.md`

## What Happened

1. Ran the full 10-step ds-review-lead pipeline against the Meta LLM Bug Reports
   fixture (`dev/test-fixtures/real/meta-llm-bug-reports.md`) with parameters:
   full / exec / proactive. Both subagents dispatched in parallel via Task tool.

2. Result: **18/100 — Major Rework**. Product owner flagged this as far too harsh
   for a published blog post with 109 likes and real engagement.

3. Saved full review output to `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md`.

4. A second test was run on the Rossmann Sales Prediction fixture (full / mixed /
   proactive). Result: **29/100 — Major Rework**. Also too harsh.

5. Created calibration notes capturing both tests and root cause observations at
   `dev/test-results/2026-02-15-calibration-notes.md`. Includes UX/output format
   decisions approved by owner (emoji indicators, compressed per-lens sections,
   rewrite highlighting).

6. Principal AI Engineer assessment identified the real root cause: **the scoring
   model treats correlated findings as independent deductions, causing extreme
   score compression**. Proposed fix: diminishing returns formula.

7. Created `dev/test-results/2026-02-15-principal-ai-engineer-assessment.md` with
   full analysis, projected score impacts, implementation plan, and alternatives.

8. Updated `dev/backlog.md` — scoring calibration is now P0 for v0.4.

## Key Decisions

- **ADR-003 (Proposed):** Diminishing returns scoring curve (0-30: 100%, 31-50: 50%, 51+: 25%)
- **CRITICAL tightening:** Move "Missing TL;DR" and "No story arc" from CRITICAL to MAJOR
- **Output UX (owner approved):** Emoji indicators in lens dashboard, compressed per-lens sections, rewrite highlighting in Top 3 fixes

## Files Modified

- `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md` (created)
- `dev/test-results/2026-02-15-calibration-notes.md` (created, then updated with Test #2 + UX decisions)
- `dev/test-results/2026-02-15-principal-ai-engineer-assessment.md` (created)
- `dev/backlog.md` (updated v0.4 section)

## What's Next

The next session should NOT implement yet. We're collecting perspectives from
multiple roles before deciding how to fix. So far we have:
- Principal AI Engineer assessment (dev/test-results/2026-02-15-principal-ai-engineer-assessment.md) — done
- DS Lead assessment — next session
- PM Lead assessment — future session (or same session)

## Pickup Prompt

```
I'm picking up the DS Analysis Review Agent project for a calibration review.

Read these files in order:
1. dev/backlog.md — current priorities
2. dev/sessions/2026-02-15-real-world-testing-calibration.md — where we left off
3. dev/test-results/2026-02-15-calibration-notes.md — two real-world test results, observations, and approved UX decisions
4. dev/test-results/2026-02-15-meta-llm-bug-reports-review.md — full review output from test #1
5. dev/test-results/2026-02-15-principal-ai-engineer-assessment.md — Principal AI Engineer's root cause analysis and proposed fix

Also read the current framework and agent prompts to understand the scoring system:
6. plugin/skills/ds-review-framework/SKILL.md — the rubrics, deduction tables, floor rules
7. plugin/agents/ds-review-lead.md — the orchestrator pipeline
8. plugin/agents/analysis-reviewer.md — analysis subagent
9. plugin/agents/communication-reviewer.md — communication subagent

Context: We ran the review agent on two real-world documents and the scores came back
far too low (18/100 and 29/100 on documents that should score 50-65). A Principal AI
Engineer already reviewed the problem and proposed a fix (diminishing returns
on deduction stacking).

Your task: Play the role of a **Senior DS Lead** — someone who manages a team of data
scientists and regularly reviews their analyses before they go to stakeholders. Read
through everything above, then provide your independent assessment of:

1. **The review agent's output quality** — Read the full Meta blog post review
   (test-results/2026-02-15-meta-llm-bug-reports-review.md). As a DS lead, are the
   individual findings legitimate? Which ones would you agree with, which would you
   push back on? Is the agent catching the right things?

2. **The scoring calibration problem** — Do you agree the scores are too low? What
   would YOU score this document if you were reviewing it for your team? What's your
   gut-check score and verdict?

3. **The Principal AI Engineer's diagnosis** — Do you agree that deduction
   stacking is the primary root cause? Or do you see it differently from a DS practice
   perspective? Does the diminishing returns fix make sense, or would you approach it
   differently?

4. **What the agent is missing from a DS perspective** — Are there things a good DS
   lead would flag that the agent doesn't? Are there things the agent flags that a
   DS lead wouldn't care about?

Save your assessment to dev/test-results/2026-02-15-ds-lead-assessment.md.

Do NOT implement any fixes yet. This is a review-only session. We're collecting
perspectives from multiple roles (Principal AI Engineer done, DS Lead is this session,
PM Lead next) before deciding how to fix.
```
