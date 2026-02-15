# Session: v0.4 Calibration Review — 2026-02-15

## What Happened

Picked up after owner completed 3 real-world tests in a separate session (Meta blog 18/100, Rossmann 29/100, Vanguard 16/100). Read all test outputs and calibration notes. Produced a PM Lead review identifying 6 critiques and 4 proposed fixes.

## Key Artifacts Read

| File | Purpose |
|---|---|
| `dev/test-results/2026-02-15-calibration-notes.md` | Root cause analysis across tests #1 and #2 + UX decisions |
| `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md` | Full review output for test #1 (18/100) |
| `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md` | Full review output for test #3 (16/100) |
| `plugin/skills/ds-review-framework/SKILL.md` | Current rubric and deduction tables |
| `plugin/agents/ds-review-lead.md` | Lead agent pipeline |
| `plugin/agents/analysis-reviewer.md` | Analysis subagent prompt |

## Key Artifact Created

| File | Purpose |
|---|---|
| `dev/reviews/2026-02-15-pm-lead-calibration-review.md` | PM Lead review — 6 critiques, 4 fixes, validation plan |

## PM Lead Review Summary

**6 Critiques:**
1. Scoring math structurally broken — additive deductions saturate at ~15 findings
2. CRITICAL over-assigned — structural issues treated same as analytical validity threats
3. No strength credits — positives have zero score impact
4. Cross-cutting issues double-count through different lenses
5. Output reads like a teardown, not a review
6. No genre or document maturity awareness

**4 Proposed Fixes (from PM review):**
1. Diminishing returns on deductions (50% after -30, 25% after -50)
2. Reclassify structural CRITICALs (missing TL;DR, no story arc, absent limitations) as MAJOR
3. Add strength credits (+3 to +5 per positive, capped +15/dimension)
4. Restructure output (strengths before findings, compress per-lens detail)

## Owner Revised Calibration Approach

Owner reviewed the PM Lead proposal and revised the fix priorities based on calibration
notes "Proposed Solution." Key changes from the PM review:

- **Positive Credit System promoted to P1** — +5 to +8 per good analytical practice, cap +25/dimension (PM review proposed +3 to +5, cap +15). Owner sees this as the primary differentiation driver.
- **CRITICAL Gradation replaces CRITICAL reclassification** — instead of demoting all structural CRITICALs to MAJOR, split into CRITICAL-ABSENT (-20, floor rule applies) vs CRITICAL-INCOMPLETE (-12, no floor rule). More nuanced than the PM proposal.
- **Per-dimension deduction cap at -70 (P2)** — simple safety net instead of diminishing returns curve.
- **Diminishing returns deferred** — positive credits + deduction cap may be sufficient.
- **Target scores revised** — Meta stays low (20-35) because it has real analytical flaws. Vanguard 40-55, Rossmann 45-60. Acceptance test: clear tier separation with 15-25 point gap.

## Owner UX Decisions (All Approved)

- Compress per-lens detail by default (1-2 sentences, no `--detail` flag)
- Emoji indicators on lens dashboard (SOUND -> check, MINOR -> warning, MAJOR/CRITICAL -> x)
- Rewrite examples: `> **pencil Suggested rewrite:**` blockquote format (one format, no alternatives)

## Files Modified

- `CHANGELOG.md` — added v0.4.0 entry
- `dev/backlog.md` — v0.4 promoted to current sprint with detailed task list
- `dev/reviews/2026-02-15-pm-lead-calibration-review.md` (new)
- `dev/sessions/2026-02-15-v04-calibration-review.md` (new, this file)

## What's Next

Per backlog (owner-revised priorities):
1. P1: Implement Positive Credit System (STRENGTH LOG in subagents, credit table in SKILL.md)
2. P1: Implement CRITICAL Gradation (CRITICAL-ABSENT vs CRITICAL-INCOMPLETE)
3. P2: Per-dimension deduction cap at -70
4. P3: UX sprint (emoji dashboard, compressed output, blockquote rewrites)
5. ADR-003 for calibration approach
6. Rerun 3 real-world tests — target: tier separation with 15-25 point gaps
7. Rerun synthetic fixtures for regression check

## Pickup Instructions

1. Read `dev/backlog.md` — v0.4 is current sprint with owner-revised task list
2. Read `dev/test-results/2026-02-15-calibration-notes.md` — contains the "Proposed Solution" with credit table that backlog references
3. Read `dev/reviews/2026-02-15-pm-lead-calibration-review.md` for PM perspective (owner diverged on some points — backlog is authoritative)
4. No commits made — all changes are unstaged
