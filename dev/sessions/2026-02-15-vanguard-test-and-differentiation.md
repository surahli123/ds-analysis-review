# Session: Vanguard Test Run & Differentiation Failure Analysis

**Date:** 2026-02-15
**Duration:** ~30 min
**Context:** v0.4 calibration sprint, third real-world test

## What Happened

1. Ran the full 10-step ds-review-lead pipeline against `dev/test-fixtures/real/vanguard-ab-test.md`
   - Parameters: full mode / tech audience / reactive workflow
   - Both subagents dispatched in parallel via Task tool, both returned successfully
   - Pipeline mechanics confirmed working (dispatch, synthesis, floor rules, deduction adherence)

2. **Result: 16/100 — Major Rework** (Analysis: 12, Communication: 19, 5 CRITICALs)
   - Saved to `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md`

3. Identified the differentiation failure: Vanguard (16) scored LOWER than Meta (18)
   despite being a fundamentally better analysis (real experiment, pre-specified hypotheses,
   balance checks, specific numbers)

4. Rewrote `dev/test-results/2026-02-15-calibration-notes.md`:
   - Added new top section "THE CENTRAL PROBLEM" framing the differentiation failure
   - Added full Test #3 (Vanguard) section with calibration assessment
   - Rewrote root cause analysis — "No credit for what's present" is now RC1 (primary),
     genre blindness demoted from primary to compounding factor
   - Updated test matrix with target scores and acceptance criteria
   - Added proposed solution section (Positive Credits + CRITICAL Gradation + Deduction Cap)

5. Updated `dev/backlog.md` — Vanguard test line now highlights it scored lower than Meta.
   Owner further updated the backlog To Do section to align with calibration notes proposals.

## Key Insight

The scoring system treats "forgot to report the p-value" the same as "never ran the
experiment." This is the core differentiation failure. Root cause: purely subtractive
scoring with zero credit for good practices, plus a binary CRITICAL threshold that
can't distinguish incomplete reporting from absent methodology.

## Files Modified
- `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md` (NEW — full review output)
- `dev/test-results/2026-02-15-calibration-notes.md` (MAJOR UPDATE — reframed around differentiation)
- `dev/backlog.md` (minor update — Vanguard line + owner updated To Do section)

## What's Next
- P1: Implement positive credit system + CRITICAL gradation (see calibration notes)
- P2: Per-dimension deduction cap as safety net
- P3: UX sprint (emoji dashboard, compressed output, rewrite formatting)
- Then: rerun all 3 tests to validate calibration targets
- ADR-003 pending once owner confirms approach
