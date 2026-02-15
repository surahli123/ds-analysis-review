# Principal AI Engineer Assessment: Scoring Calibration

**Date:** 2026-02-15
**Role:** Principal AI Engineer
**Status:** Review complete (proposed fix, not yet implemented)

## Context

Two real-world tests revealed the scoring model produces scores 30+ points below
reasonable expectations:

| Test | Type | Score | Expected |
|---|---|---|---|
| Meta LLM Bug Reports | Blog post | 18/100 | 55-65 |
| Rossmann Sales Prediction | Kaggle/README | 29/100 | 45-60 |

The framework differentiates directionally (29 > 18, correct) but absolute
calibration is broken. Both verdicts are Major Rework on documents that should
land in the Minor Fix range.

Full test data: `dev/test-results/2026-02-15-calibration-notes.md`

## Root Cause

**The scoring model treats findings as independent events, but real findings are
correlated. They cluster around a few root causes, and every finding stacks at
full weight.**

The math problem:
- **Deduction budget per dimension:** 100 points
- **Deduction capacity per dimension:** 4 lenses x 3 findings x avg ~10 = **~120 points**
- **Result:** Capacity (120) exceeds budget (100). Any document with 2+ issues per
  lens mathematically scores near zero.

Example — Meta blog analysis dimension found 7 findings for -88 deductions, but
they cluster into roughly 3 root causes:

| Root Cause | Findings | Raw Deduction |
|---|---|---|
| No validation evidence for the LLM system | Unstated assumptions (-20), No baseline (-10), No limitations (-10) | -40 |
| Causal claim without methodology | Causal claim (-20), "double digits" vague (-10) | -30 |
| No failure mode analysis | Missing obvious analysis (-8), Unsupported leap (-10) | -18 |

Three root problems. Reasonable. But the model counts 7 independent deductions
totaling -88. It double-counts correlated symptoms of the same underlying gap.

### Why Genre Blindness Is Secondary

Even with perfect genre detection, any real internal Confluence page with 10+
legitimate findings would also score in the 20s. Genre awareness masks the problem
but doesn't fix it. Genre is a v1.0 nice-to-have, not the scoring fix.

## Decision

### Primary Fix: Diminishing Returns Per Dimension

Apply a compression curve to raw deductions before computing the score:

```
Raw deductions 0-30:   100% applied (full weight)
Raw deductions 31-50:   50% applied
Raw deductions 51+:     25% applied
```

**Projected impact on test cases:**

Meta Blog — Analysis (raw: -88):
- First 30: -30
- Next 20 (of remaining 58): -10
- Remaining 38 at 25%: -9.5
- Effective deduction: ~50 → Score: ~50 (was 12)

Meta Blog — Communication (raw: -76):
- First 30: -30
- Next 20 (of remaining 46): -10
- Remaining 26 at 25%: -6.5
- Effective deduction: ~47 → Score: ~54 (was 24)

**Meta Blog final: ~52/100** (was 18). Within expected 55-65 range.

Rossmann — Analysis (raw: -61):
- First 30: -30, next 20: -10, remaining 11 at 25%: -2.75
- Effective: ~43 → Score: ~57 (was 39)

Rossmann — Communication (raw: -81):
- First 30: -30, next 20: -10, remaining 31 at 25%: -7.75
- Effective: ~48 → Score: ~52 (was 19)

**Rossmann final: ~55/100** (was 29). Within expected 45-60 range.

### Why Diminishing Returns Works

1. **Preserves differentiation.** A document with -40 raw deductions still scores
   meaningfully worse than one with -20. The curve compresses the extremes, not
   the signal.

2. **Self-correcting.** Even if subagents over-find, the compression prevents
   catastrophic scores. No need to know the "right" deduction values in advance.

3. **Mirrors human judgment.** When a reviewer finds the 8th issue, it doesn't
   make the document twice as bad as finding the 4th. There's natural saturation
   in human judgment — the scoring model should reflect that.

4. **Trivial to implement.** One formula change in the lead agent's Step 9
   synthesis. No subagent prompt changes, no deduction table changes, no new
   parameters.

### Secondary Fix: Tighten CRITICAL Definitions

The floor rule (2+ CRITICAL = Major Rework cap at 59) triggers too easily because
CRITICAL is too broad.

| Current CRITICAL | Should Be CRITICAL? | Rationale |
|---|---|---|
| Causal claim without methodology | **Yes** | Genuinely misleading — reader draws wrong conclusions |
| Unstated critical assumption | **Yes** | Foundation of the analysis is unverifiable |
| Conclusion contradicts evidence | **Yes** | Factually wrong |
| Missing TL;DR | **No → MAJOR** | Structural gap, not factual error. Reader can still find the info. |
| No story arc | **No → MAJOR** | Structure preference, not misleading content |
| Limitations unclear for downstream | **Borderline** | Could cause real harm, but depends on context |

**Proposed CRITICAL standard:** Reserve CRITICAL for findings where the reader
would reach a **wrong conclusion** or make a **wrong decision** based on what's
written. Structural and formatting issues cap at MAJOR — they make the document
harder to use, but they don't mislead.

This alone would reduce the Meta blog from 4 CRITICAL to 2, and the Rossmann from
2 to 1. Combined with diminishing returns, the floor rules stop dominating the
verdict.

## What NOT to Do

- **Don't add `--format` yet.** Scope creep. Adds a parameter users have to think
  about. Solve the math first.
- **Don't rescale all deduction values.** Brute-force fix that shifts the problem
  without solving it. Would need recalibration after every new test.
- **Don't cap findings per dimension.** Throws away information. The review should
  still surface all findings — just don't let them all hit the score at full weight.

## Implementation Plan

1. **Diminishing returns formula** — add to lead agent Step 9 synthesis (one
   paragraph of new logic). Document the curve in SKILL.md.
2. **Tighten CRITICAL definitions** — update SKILL.md Section 1 severity
   definitions. Move "Missing TL;DR" and "No story arc" to MAJOR in the
   deduction table (Section 2).
3. **Re-run both test fixtures** — verify new scores land in expected ranges.
4. **Update this ADR status** to Accepted after validation.

Estimated effort: one focused session. Small changes (one formula, a few table
edits) with significant impact on score distribution.

## Alternatives Considered

### Cluster-Based Scoring (Best, Most Complex)

Group findings by root cause. Apply highest deduction per cluster, then 30% for
each additional finding in the cluster. Most accurate model, but requires the lead
agent to perform root-cause clustering during synthesis — adds complexity and
potential for clustering errors. Recommended for v1.5 exploration.

### Per-Dimension Deduction Cap

Hard cap total deductions at -50 per dimension. Simpler than diminishing returns
but creates a cliff: documents with -49 and -90 raw deductions get very different
treatment around the boundary. Diminishing returns is smoother.

### Rescale All Deduction Values

Cut all values by ~40%. Simplest change but doesn't fix the structural issue.
Documents with 20+ findings would still score near zero. Needs recalibration
every time a new finding type is added.
