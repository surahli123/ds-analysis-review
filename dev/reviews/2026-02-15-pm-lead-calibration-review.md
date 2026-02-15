# PM Lead Review: Calibration Failure Assessment

**Reviewer:** Senior PM Lead (product lens)
**Date:** 2026-02-15
**Scope:** Real-world test results #1-3, calibration notes, SKILL.md rubric, all agent prompts
**Status:** Review complete. Pending owner decision on which fixes to pursue.

---

## Context

Three real-world tests completed. All scored far below expert intuition:

| # | Fixture | Genre | Audience | Workflow | Score | Expected |
|---|---|---|---|---|---|---|
| 1 | Meta LLM Bug Reports | Blog post | exec | proactive | 18/100 | 55-65 |
| 2 | Rossmann Sales Prediction | Kaggle/README | mixed | proactive | 29/100 | 45-60 |
| 3 | Vanguard A/B Test | Course project | tech | reactive | 16/100 | 40-55 |

The pipeline works correctly (subagent dispatch, dimension separation, deduction table adherence, floor rules). The findings are often genuinely insightful. The problem is that the scoring translates those findings into verdicts no user will trust.

---

## The Product Problem

This tool promises to help DS practitioners improve their analyses. After 3 real-world tests, it produces scores of 16, 18, and 29 on published, functional documents. That's the equivalent of a spell checker that marks every paragraph as unintelligible. No user will trust this tool after one use, and no user will come back for a second.

The irony is sharp: a communication review tool with a communication problem. The review output tells the user their document buries the lede — while the review itself buries its best insights under a demoralizing 16/100 headline.

---

## Critique 1: The Scoring Math Is Structurally Broken

This is not a calibration issue. It is a design flaw.

The deduction table has 8 analysis rows (-8 to -20) and 18 communication rows (-3 to -15). Each reviewer can report up to 3 findings per lens (4 lenses = max 12 findings per dimension). Each CRITICAL costs -12 to -20.

In practice, the agent consistently finds ~15 findings across both dimensions. At an average of ~-10.5 per deduction, that is -158 in total deductions against a 200-point budget (100 per dimension). The result is predictable:

| Test | Analysis Deductions | Comm Deductions | Total | Final Score |
|---|---|---|---|---|
| Meta blog | -88 | -76 | -164 | 18 |
| Rossmann | -61 | -81 | -142 | 29 |
| Vanguard | -88 | -81 | -169 | 16 |

The deduction totals are remarkably consistent (-142 to -169) across three very different documents. That is a ceiling effect — the scoring saturates. It cannot distinguish "needs significant work" from "fundamentally broken" because both produce the same score band.

The fix is not reducing individual deduction values. The problem is that deductions are additive with no diminishing returns, across too many lenses, with values calibrated for a world where a document has 3-5 findings — not 15.

---

## Critique 2: CRITICAL Is Over-Assigned and Under-Defined

The severity definition says CRITICAL means "a fundamental flaw that would cause the analysis to fail its purpose — wrong conclusions reached, key audience misled, or decisions made on faulty foundation."

In practice, roughly half the CRITICALs are legitimate analytical validity threats. The other half are structural/presentation issues that the rubric treats identically:

| CRITICAL Finding | Test | Genuinely Misleading? |
|---|---|---|
| Missing TL;DR | All 3 | No — structural preference |
| No clear story arc | Meta, Vanguard | No — both have logical progression |
| No named owners for recommendations | Meta | No — it is a blog post |
| Limitations absent | Vanguard | Debatable — weakens but does not mislead |
| No statistical testing | Vanguard | **Yes** — genuinely CRITICAL |
| Causal claim without methodology | Meta | **Yes** — genuinely CRITICAL |
| Unstated assumption of comparability | Vanguard | **Yes** — genuinely CRITICAL |
| LLM accuracy not validated | Meta | **Yes** — genuinely CRITICAL |

"Missing TL;DR" (-15, CRITICAL) is in the same severity class as "causal claim without methodology" (-20, CRITICAL). A missing TL;DR makes a document annoying. A false causal claim makes it dangerous.

The floor rules amplify this: 2+ CRITICAL triggers Major Rework (max 59). Since "missing TL;DR" is CRITICAL in the communication dimension and almost every real document will trigger it, the floor rule fires on virtually everything. The Vanguard test had 5 CRITICALs — more than most production incident reports.

---

## Critique 3: The Agent Does Not Credit What Is Working

The scoring model is purely subtractive: start at 100, lose points. Strengths are reported narratively in "What You Did Well" (always 2-3 items) but have zero impact on the score.

The Vanguard review's positives:
- Hypothesis-driven analytical framework (pre-specified, testable hypotheses)
- Practical significance threshold pre-specified (5% minimum lift — strong experimental practice)
- Experiment validity assessment included (balance check before results)

These are non-trivial analytical strengths. But the analysis score is 12/100 — the same score you would get from a document with zero methodology at all. The scoring cannot distinguish "good foundations with specific gaps" from "no foundations whatsoever."

A review that finds 3 genuine strengths and then delivers 16/100 feels dishonest. The user reads the positives, feels seen, then reads the score and feels lied to.

---

## Critique 4: Cross-Cutting Issues Double-Count in Practice

The routing table (SKILL.md Section 5) says findings should not be duplicated across dimensions. In theory, the system respects this. In practice, the same root cause generates separate deductions in each dimension through different lenses.

Vanguard example — missing statistical tests:
- Analysis: "No statistical testing" -> CRITICAL, -20 (Methodology)
- Analysis: "No sample sizes or power analysis" -> MAJOR, -10 (Metrics)
- Communication: "Insufficient technical rigor for tech audience" -> MAJOR, -10 (Audience Fit)
- Communication: "Over-interpretation boundary unclear" -> MAJOR, -8 (Actionability)

One root cause generating -48 in total deductions across 4 separate line items. The routing table prevents the exact same finding from appearing twice, but it does not prevent the same root cause from spawning derivatives in every lens it touches.

---

## Critique 5: The Output Reads Like a Teardown, Not a Review

The output structure leads with the score (devastating), then the lens dashboard (all red), then the top 3 fixes (all CRITICAL), and only then gets to "What You Did Well." By the time the user reaches the positives, they have already mentally checked out.

The full Vanguard review has 9 numbered findings, each with title, severity, location, issue (2-3 sentences), and suggested fix. That is roughly 2,000 words of findings for a 1,125-word input document. The review is nearly 2x the length of the document being reviewed.

The output format treats every finding with equal weight and detail, when the user really needs to know the 2-3 things that matter most.

---

## Critique 6: No Genre or Document Maturity Awareness

The calibration notes call this "genre blindness." I would reframe it: the issue is not just genre — it is document maturity and purpose.

- A blog post is a finished artifact optimized for engagement. It intentionally omits limitations sections and named owners.
- A Kaggle notebook is a portfolio piece demonstrating technical skill. It is not trying to drive a business decision.
- A course project is a learning exercise.
- An internal analysis for a CFO is the only genre where the current rubric's expectations are fully reasonable.

Even if a `--format` flag is added, most users will not know which format to pick. The tool should help them, not require them to self-classify. The lead agent already computes word count and detects partial input. It should also form a lightweight hypothesis about document type and adjust expectations accordingly.

---

## Proposed Fixes (Priority Ordered)

### Fix 1: Diminishing Returns on Deductions (Addresses Critique 1)

After total deductions exceed -30 per dimension, subsequent deductions apply at 50%. After -50, they apply at 25%. This creates a soft floor:

| Raw Deductions | Effective Deductions | Dimension Score |
|---|---|---|
| -20 | -20 | 80 |
| -40 | -35 | 65 |
| -60 | -42.5 | 57 |
| -88 | -52 | 48 |

Recomputed scores with this model:

| Test | Analysis (raw -> eff -> score) | Comm (raw -> eff -> score) | New Score | Old Score |
|---|---|---|---|---|
| Meta | -88 -> ~52 -> 48 | -76 -> ~47 -> 53 | ~50 | 18 |
| Rossmann | -61 -> ~43 -> 57 | -81 -> ~49 -> 51 | ~54 | 29 |
| Vanguard | -88 -> ~52 -> 48 | -81 -> ~49 -> 51 | ~50 | 16 |

Scores land in the 50-55 range. Still "Major Rework" or low "Minor Fix" but no longer absurd.

### Fix 2: Reclassify Structural CRITICALs as MAJOR (Addresses Critique 2)

Reserve CRITICAL exclusively for findings that would cause the reader to reach wrong conclusions or take harmful action based on faulty evidence:

| Current CRITICAL | Proposed | Rationale |
|---|---|---|
| Missing TL;DR | MAJOR (-10) | Annoying, not misleading |
| No clear story arc | MAJOR (-8) | Structural weakness, not analytical failure |
| Limitations/scope absent | MAJOR (-10) | Real gap but does not fabricate conclusions |
| Unstated critical assumption | Keep CRITICAL (-20) | Reader misled about analytical foundation |
| Flawed statistical methodology | Keep CRITICAL (-20) | Conclusions may be wrong |
| Conclusion doesn't trace to evidence | Keep CRITICAL (-15) | Reader misled about support for claims |

The Meta blog drops from 4 CRITICALs to 2 (both legitimate). Vanguard drops from 5 to 2. Floor rules still fire for genuinely dangerous analytical flaws, not because a document lacks a formatted TL;DR.

### Fix 3: Add Strength Credits to Scoring (Addresses Critique 3)

Each "What You Did Well" item adds +3 to +5 to the relevant dimension (capped at +15 total per dimension). This is modest — it will not turn a bad score into a good one — but it prevents the absurd outcome where strong foundations have zero score impact.

The Vanguard analysis positives (hypothesis-driven framework, pre-specified practical significance threshold) would add roughly +6 to +10. Combined with diminishing returns, the analysis score moves from 12 to ~55-58. That feels right for "methodologically disciplined experiment with real statistical gaps."

### Fix 4: Compress Output and Lead with Strengths (Addresses Critique 5)

Restructure the output order:

1. Score + verdict (keep up top — now reasonable after Fixes 1-3)
2. Lens dashboard with emoji indicators (already approved)
3. **What You Did Well** (move BEFORE findings)
4. **Top 3 Priority Fixes** (only these get full detail: location, issue, suggested fix with highlighted rewrite)
5. Per-lens sections: 1-2 sentences per finding (finding title + core issue only). No location, no multi-sentence explanation, no suggested fix — those are in the Top 3 if the finding is important enough.

This cuts the output roughly in half and front-loads the trust-building content.

---

## What I Would Defer

**Genre/format detection:** If Fixes 1-3 are implemented correctly, the scoring should produce reasonable results across genres without needing the user to classify their document. If scores are still off after recalibration, revisit genre handling as a v0.5 feature. Adding a new parameter increases cognitive load and creates a classification problem you have to maintain.

**Per-dimension deduction cap:** With diminishing returns (Fix 1), a hard cap is redundant. The soft floor naturally prevents extreme scores. If testing shows documents still hitting unreasonable lows, add a hard floor of 20/100 per dimension. Try without it first.

**Cross-cutting deduction deduplication (Critique 4):** Real but hard to fix without making agent prompts significantly more complex. The diminishing returns model partially addresses it by reducing the marginal impact of each additional deduction from the same root cause. Defer unless recalibrated scores still feel wrong.

---

## Validation Plan

After implementing Fixes 1-4, rerun all 3 real-world tests. Expected outcomes:

| Test | Old Score | Target Range | Expected Verdict |
|---|---|---|---|
| Meta blog | 18 | 50-65 | Minor Fix |
| Rossmann | 29 | 50-65 | Minor Fix |
| Vanguard | 16 | 40-55 | Major Rework or low Minor Fix |

The Vanguard doc should score lowest because it has the most legitimate analytical gaps (an A/B test with no statistical inference). But it should not score 35+ points lower than the others.

After real-world tests pass, rerun synthetic fixtures to verify floor rules and dimension separation still behave correctly.
