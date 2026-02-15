# Scoring Calibration Notes

**Last updated:** 2026-02-15 (added Test #3 Vanguard, reframed around differentiation failure)
**Status:** OPEN — Blocking v0.4 completion

---

## THE CENTRAL PROBLEM: The System Cannot Differentiate Quality

Three real-world tests now show a scoring system that **cannot tell good work
from bad work**:

| # | Document | Type | Score | Analysis | Comms | Verdict | CRITICALs |
|---|---|---|---|---|---|---|---|
| 1 | Meta LLM Bug Reports | Blog (exec/proactive) | **18** | 12 | 24 | Major Rework | 4 |
| 2 | Rossmann Sales Prediction | Kaggle (mixed/proactive) | **29** | 39 | 19 | Major Rework | 2 |
| 3 | Vanguard A/B Test | A/B test (tech/reactive) | **16** | 12 | 19 | Major Rework | 5 |

**Vanguard (16) scored LOWER than Meta (18).** The system thinks a polished blog
post with zero methodology is better than an actual experiment with real data,
pre-specified hypotheses, and balance checks. This is backwards.

### Why This Is the #1 Priority

The differentiation failure isn't a cosmetic issue — it makes the tool unusable.
If a user reviews two analyses and both get ~18/100 Major Rework, they have no
signal about which one needs more work. The review becomes noise, not signal.

**The human reviewer test:** Ask any senior DS: "Which of these two is closer to
being ready for stakeholders?" 100% pick Vanguard. Our system picks Meta.

### What Makes Vanguard Better Than Meta (That the Scoring Ignores)

| What Vanguard Has | What Meta Has | Score Impact |
|---|---|---|
| Real experimental design (test vs. control) | No experiment at all | +0 for Vanguard |
| 3 pre-specified, testable hypotheses with KPIs | Vague "double digits" claim | +0 for Vanguard |
| Pre-specified 5% practical significance threshold | No success criteria | +0 for Vanguard |
| Covariate balance check across 7 variables | No balance check | +0 for Vanguard |
| Reports actual numbers (309.68s, 51.72%, etc.) | "Double digits over last few months" | +0 for Vanguard |
| Evaluates experiment duration for validity | No validity assessment | +0 for Vanguard |

Every row is something Vanguard does right that Meta doesn't. Every row is worth
**exactly zero points** in the current scoring model. This is the root cause.

### The Fix-Distance Gap

**Vanguard** — a senior DS could fix this in an afternoon:
- Add 3 statistical tests with p-values and CIs
- Add sample sizes to each result section
- Define the completion rate metric precisely
- Add a TL;DR paragraph and a limitations section
- The data exists, the experiment was run, the structure is there

**Meta** — a senior DS cannot fix this:
- There's no experiment to add statistics to
- There's no validation data to report accuracy from
- The causal claims need entirely new studies to substantiate
- The "double digits" needs data that may not exist in the blog context
- You'd need to go back and do work that was never done

The current system treats these as equally broken. They are not.

### What "Right" Looks Like After Calibration

| Document | Current Score | Target Score | Target Verdict |
|---|---|---|---|
| Vanguard A/B Test | 16/100 | 40-55 | Minor Fix |
| Rossmann Sales Prediction | 29/100 | 45-60 | Minor Fix |
| Meta LLM Bug Reports | 18/100 | 20-35 | Major Rework |

The key: **clear separation between tiers.** Fixable analyses in the 40-55 range.
Fundamentally flawed work in the 20-35 range. These shouldn't overlap.

---

## Root Cause Analysis (Updated After All 3 Tests)

### RC1: No Credit for What's Present (PRIMARY — drives differentiation failure)

The scoring model is purely subtractive: 100 minus deductions. Positive findings
appear in the output but have **zero impact on the numeric score.**

This is the #1 root cause because it directly explains why Vanguard and Meta score
the same. Vanguard does 6+ things right that Meta doesn't. In the current model,
those 6 things are worth 0 points.

**Perverse incentive:** The more detailed your analysis, the more surface area you
expose for the reviewer to find gaps. A vague blog post that says nothing specific
is harder to penalize than a detailed analysis that shows its work but has gaps.
Analyses that try harder get punished more.

### RC2: Deduction Stacking Without Diminishing Returns

Both Vanguard and Meta hit -88 on the Analysis dimension (score: 12). But the
underlying gaps are categorically different:

**Vanguard's -88 = reporting gaps (all fixable):**
- No p-values reported → -20 (but the experiment exists, data exists)
- Balance check not rigorous enough → -20 (but a check was done)
- H2 contradiction unresolved → -10 (narrative gap, data is there)
- H3 direction unstated → -10 (narrative gap, data is there)
- No subgroup analysis → -8 (scope gap, data available)
- No sample sizes → -10 (reporting gap, data exists)
- Metric definition ambiguous → -10 (clarity gap, easy fix)

**Meta's -88 = work-never-done (mostly unfixable):**
- Causal claim without methodology → -20 (no experiment was ever run)
- LLM accuracy unvalidated → -20 (validation was never done)
- Detection-to-impact logical leap → -10 (evidence chain doesn't exist)
- No baseline comparison → -10 (comparison was never run)
- No failure mode analysis → -8 (analysis was never done)
- No limitations acknowledged → -10 (fundamental omission)
- "Double digits" without specifics → -10 (data not provided)

Same total. Completely different severity. The model is blind to this because it
only counts what's missing, not whether what's missing could plausibly be added.

### RC3: CRITICAL Threshold Is Binary — No Gradation

"No statistical tests reported" (Vanguard) and "causal claim with zero
methodology" (Meta) both get CRITICAL/-20. But:
- Vanguard ran the experiment, collected data, and forgot to report stats
- Meta never ran an experiment and claimed causation anyway

These are categorically different. The CRITICAL severity has no way to express
"methodology present but under-reported" vs. "methodology absent entirely."

### RC4: Genre Blindness (Compounds RC1-RC3, Not Primary)

After Test #1, genre blindness was identified as the primary issue. With Test #3,
it's clear genre blindness is a **compounding factor**, not the root cause. Even
comparing analysis-like documents (Vanguard vs. a hypothetical internal analysis),
the scoring would still fail to credit good analytical practices.

Genre awareness would help Meta (blog posts evaluated less strictly) but would NOT
help differentiate Vanguard from a weaker analysis in the same genre.

### RC5: More Detailed Analyses Get Punished (Consequence of RC1+RC2)

Vanguard exposes surface area by reporting numbers and stating hypotheses. Each
number without a CI is a finding. Each hypothesis without a formal test is a
finding. Meta's vagueness gives the reviewer less to grab. Result: the analysis
that tries harder scores lower.

---

# Individual Test Results

## Test #1: Meta LLM Bug Reports

**Date:** 2026-02-15
**Fixture:** `dev/test-fixtures/real/meta-llm-bug-reports.md`
**Full review output:** `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md`
**Parameters:** full / exec / proactive / Tier 1

### Problem

The Meta blog post "How Facebook Leverages LLMs to Understand User Bug Reports" scored
**18/100 — Major Rework**. This is a published article on the Analytics at Meta Medium blog
with 109 likes and 2 substantive positive comments. An 18/100 verdict is clearly too harsh.

The scoring penalizes the article as if it were an internal DS analysis meant to drive
an executive decision. It's not — it's a public thought-leadership blog post sharing
a methodology. The agent doesn't know the difference.

## Root Cause Analysis

### 1. Genre Blindness (Primary)

The framework assumes every input is an **internal DS analysis** meant to drive decisions.
It has no concept of document genre/format. A blog post, an internal Confluence page, a
Kaggle notebook, and an exec deck all get evaluated against the same rubric.

A blog post doesn't need:
- Named owners for recommendations (Actionability CRITICAL: -8)
- Explicit limitations section (Audience Fit MAJOR: -12)
- Confidence intervals or statistical validation (Metrics MAJOR: -10)
- Executive TL;DR with "recommended action" (Structure CRITICAL: -15)

These are legitimate gaps in an internal analysis. In a blog post, they're normal.

**Potential fix options:**
- A. Add a `--format` parameter (blog, internal-analysis, notebook, deck) that adjusts
  severity thresholds and lens weights per format
- B. Add format-awareness to the lead agent's pre-processing step — detect genre from
  content signals and adjust expectations (lighter touch, more risk of wrong detection)
- C. Keep the current framework but add a "calibration note" to the output when the
  document appears to be a public/external piece (softest change, least useful)

### 2. Deduction Stacking Is Too Aggressive

With 8 lenses and up to 3 findings per lens at -8 to -20 each, the math allows
reaching 0/100 easily. Current test result:
- Analysis: 7 findings totaling -88 → score 12
- Communication: 8 findings totaling -76 → score 24

That's 15 findings. Even if every finding is legitimate, a 15-finding review that
produces 18/100 feels more like a teardown than a review. Most real-world analyses will
have 5-10 legitimate findings — the question is whether they should all stack to zero.

**Potential fix options:**
- A. Diminishing returns: after total deductions exceed -40, subsequent deductions apply
  at 50% (soft floor that prevents extreme low scores)
- B. Cap total deductions per lens (e.g., max -25 per lens, max -50 per dimension)
- C. Cap total findings counted toward score (e.g., top 5 deductions per dimension)
- D. Adjust base deduction values downward across the board (simplest but bluntest)

### 3. CRITICAL Threshold Too Sensitive

"Missing TL;DR" (-15) and "No story arc" (-12) are both CRITICAL. Together they
trigger the 2+ CRITICAL floor rule (verdict max 59). The Meta blog post DOES have
structure — it has titled sections, a logical progression, and a conclusion. It just
doesn't match the inductive exec-facing pattern the rubric demands.

Similarly, "no specific recommendations with owners" is rated CRITICAL under Actionability
in proactive workflow. For an internal analysis, that's fair. For a blog post, it's absurd.

**Potential fix options:**
- A. Tighten CRITICAL definition: reserve for things that would genuinely mislead the
  reader (wrong conclusions, fabricated data), not structural preferences
- B. Genre-gate CRITICALs: some issue types can only be CRITICAL for internal analyses
- C. Raise the floor rule threshold: require 3+ CRITICAL instead of 2+ for Major Rework

### 4. Proactive + Exec Is the Harshest Lens

The test used `--audience exec --workflow proactive`, which is the maximum-strictness
combination. This sets the highest bar for TL;DR, actionability, and recommendations.
Fair for an internal "here's what we should invest in" memo — too strict for a blog post.

This compounds with root cause #1 (genre blindness). If we fix genre awareness, this
resolves itself — a blog post would never be evaluated as proactive/exec regardless
of what the user passes.

### 5. Some Findings Are Genuinely Valid

Not everything is miscalibrated. These findings are fair regardless of genre:
- The "double digits" claim IS vague — even a blog should be more specific
- The causal framing IS misleading — "our approach reduced bug reports" without
  evidence of causation is a real analytical flaw
- The article IS light on evidence for its core claims

The problem isn't that the agent found nothing real. It's that real findings get
stacked with genre-inappropriate findings, and the combined score is devastating.

## Recommended Tuning Priority

**For the next session (v0.4 calibration sprint):**

1. **Run 2-3 more real-world tests** before changing anything. Use different fixture
   types (Kaggle notebooks, internal-style analyses) to see if the over-penalization
   is consistent or genre-specific.

2. **If genre-specific** (blog posts score much lower than internal analyses), the
   primary fix is adding `--format` or auto-detection. This is a design decision
   that warrants an ADR.

3. **If consistent** (everything scores too low), the fix is deduction table
   recalibration — probably diminishing returns + tighter CRITICAL definitions.

4. **Either way**, consider a per-dimension deduction cap to prevent extreme scores.
   No real-world document should score 12/100 on analysis unless it's genuinely
   fabricated or incoherent.

## Test Scores for Reference

| Metric | Score |
|---|---|
| Analysis dimension | 12/100 |
| Communication dimension | 24/100 |
| Final (average) | 18/100 |
| CRITICAL findings | 4 (2 analysis, 2 communication) |
| Total findings | 15 |
| Verdict | Major Rework |
| Floor rule | Applied (4 CRITICAL → max 59) |

## What "Right" Might Look Like for This Article

Gut-check estimate for a well-calibrated score on this blog post:
- Analysis: 50-65 (real gaps in evidence and causal claims, but functional for its purpose)
- Communication: 55-70 (decent structure for a blog, but could lead with impact better)
- Final: 55-65, likely **Minor Fix** verdict
- 1-2 CRITICAL at most (causal claim is the strongest candidate)
- The review should acknowledge this is a blog post and calibrate accordingly

This would produce a review that says "solid blog post with some real analytical gaps
to address" rather than "this analysis has failed its purpose."

---

# Calibration Notes — Real-World Test #2

**Date:** 2026-02-15
**Fixture:** `dev/test-fixtures/real/rossmann-sales-prediction.md`
**Full review output:** Produced in-session (not saved to file)
**Parameters:** full / mixed / proactive / Tier 3

## Score Summary

| Metric | Score |
|---|---|
| Analysis dimension | 39/100 |
| Communication dimension | 19/100 |
| Final (average) | 29/100 |
| CRITICAL findings | 2 (1 analysis, 1 communication) |
| Total findings | 15 (6 analysis, 9 communication) |
| Verdict | Major Rework |
| Floor rule | Applied (2 CRITICAL → max 59) |

## Calibration Assessment

### Marginally better differentiation — but still too harsh

The Rossmann notebook scored 29/100 vs the Meta blog post's 18/100. This is
directionally correct — the Rossmann analysis IS a stronger piece of analytical
work (proper time-series CV, hypothesis-driven EDA, data leakage awareness,
end-to-end deployment). The 11-point gap confirms the framework CAN distinguish
quality levels.

**But 29/100 is still far too punitive.** This is a complete, deployed Kaggle
project with real methodological rigor. It has real gaps (no TL;DR, no stated
assumptions, vague recommendations), but a reasonable gut-check score would be
45–60 (Minor Fix range). The scoring math punishes too aggressively —
especially stacking 9 communication findings on a document that is fundamentally
a GitHub portfolio README, not an exec-ready analysis.

### Confirms: deduction stacking is the primary calibration problem

Both tests now show the same pattern:
- Meta blog: 15 findings, 18/100
- Rossmann notebook: 15 findings, 29/100

15 findings producing scores below 30 is a math problem, not a content problem.
The fix priority from Test #1 holds: diminishing returns or per-dimension
deduction cap.

### Genre blindness partially confirmed (different angle)

The Rossmann doc is a GitHub README / Kaggle portfolio piece. It's not an internal
Confluence analysis for a CFO, even though the hypothetical scenario frames it
that way. The framework evaluates it as if the CFO is actually waiting for this
deliverable, which inflates communication findings.

That said, the proactive/mixed parameters were intentionally set for this test.
The Rossmann doc DOES set up a CFO stakeholder scenario, so evaluating against it
isn't unfair — just very strict.

## UX / Output Format Issues Observed

### 1. Output is too long — compress per-lens sections by default

The full review output is massive — multiple screenfuls. For a CLI tool, this
is a wall of text that loses the reader (ironic for a communication review agent).

**Decision (owner approved):** Compress per-lens detail sections directly — no
new flags or modes. The default output should be concise. Approach:
- Per-lens findings: 1-2 sentences max (finding title + core issue)
- Full detail (location, multi-sentence issue, suggested fix) only in Top 3
- No `--detail compact` flag — just make the default output shorter

### 2. Lens Dashboard needs visual indicators ✅ APPROVED

**Decision (owner approved):** Add emoji indicators to lens dashboard ratings.

Mapping:
- SOUND → ✅
- MINOR ISSUES → ⚠️
- MAJOR ISSUES → ❌
- CRITICAL → ❌

Target format:
```
| Analysis | Methodology & Assumptions | ❌ CRITICAL |
| Analysis | Logic & Traceability       | ❌ MAJOR    |
| Analysis | Metrics                    | ❌ MAJOR    |
| Communication | Structure & TL;DR    | ❌ CRITICAL |
| Communication | Audience Fit         | ⚠️ MINOR    |  ← example
```

Instantly scannable — reader gets the "shape" of the review at a glance.

### 3. Top 3 Priority Fixes: rewrite examples need color highlighting

The suggested fixes include rewrite examples (e.g., a sample TL;DR), but they
blend into the surrounding text. The user wants these to visually pop.

**Decision (owner approved):** Use blockquote with emoji prefix — one format,
highest confidence. Don't list multiple options in the output; the reader is
already processing a lot of information.

Format:
```
> **✏️ Suggested rewrite:**
> TL;DR: Our XGBoost model forecasts R$286.7M in total 6-week sales...
> Average prediction error is 10% MAPE, with most stores between 5–20%.
```

This works in plain markdown, renders in CLI and GitHub, and visually
separates "here's what good looks like" from the issue description.

---

## Test #3: Vanguard A/B Test

**Date:** 2026-02-15
**Fixture:** `dev/test-fixtures/real/vanguard-ab-test.md`
**Full review output:** `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md`
**Parameters:** full / tech / reactive / Tier 1

### Score Summary

| Metric | Score |
|---|---|
| Analysis dimension | 12/100 |
| Communication dimension | 19/100 |
| Final (average) | 16/100 |
| CRITICAL findings | 5 (2 analysis, 3 communication) |
| Total findings | 16 (7 analysis, 9 communication) |
| Verdict | Major Rework |
| Floor rule | Applied (5 CRITICAL → max 59) |

### Calibration Assessment

#### This score is the clearest evidence of the differentiation failure

Vanguard scored 16/100 — 2 points LOWER than Meta's 18/100. This result
triggered the reframing of this entire document around the differentiation
problem (see "THE CENTRAL PROBLEM" section above).

The Vanguard analysis has genuine gaps: no statistical tests reported, no sample
sizes, no limitations section, no TL;DR. These are legitimate findings. But the
analysis also has real strengths that the scoring completely ignores:

- Pre-specified hypotheses with named KPIs
- Real experimental design with test/control groups
- Pre-specified practical significance threshold (5%)
- Covariate balance check across 7 variables
- Actual quantitative results (not vague claims)

None of these earn credit. The system only sees what's missing, never what's present.

#### Identical analysis scores for opposite problems

Both Vanguard and Meta scored **12/100 on the Analysis dimension** with the same
total deductions (-88). But the nature of the gaps is fundamentally different:

- **Vanguard:** 7 findings, all reporting/narrative gaps. The experiment was run,
  the data exists, the statistical tests could be added in an afternoon.
- **Meta:** 7 findings, mostly work-never-done gaps. No experiment was designed,
  no validation was run, no baseline was measured.

12/100 for both is absurd. Vanguard should be 35-50 on Analysis (fixable gaps in
an otherwise sound experiment). Meta should stay around 12-20 (no analysis to fix).

#### Communication score is also too harsh

19/100 on communication for a document that has clear section headings, a logical
hypothesis-driven progression, and specific results is too punitive. The missing
TL;DR and absent limitations section are real issues, but 3 CRITICALs (missing
TL;DR, no story arc, no limitations) triggers the 2+ CRITICAL floor rule
aggressively. The story arc issue in particular is debatable — the document DOES
follow a hypothesis → method → result → conclusion arc, just not the specific
"answer-first" pattern the rubric demands for reactive workflow.

### What "Right" Might Look Like

- Analysis: 35-50 (real experiment with reporting gaps — fixable in an afternoon)
- Communication: 35-45 (structured but needs TL;DR and audience calibration)
- Final: 40-55, **Minor Fix** verdict
- 1-2 CRITICAL at most (missing stats is the strongest candidate for analysis;
  missing TL;DR for communication)
- The gap from Meta should be 20-30 points, not 2

---

## Updated Test Matrix

| # | Fixture | Type | Audience | Workflow | Score | Target |
|---|---|---|---|---|---|---|
| 1 | Meta LLM Bug Reports | Blog post | exec | proactive | 18/100 | 20-35 |
| 2 | Rossmann Sales Prediction | Kaggle/README | mixed | proactive | 29/100 | 45-60 |
| 3 | Vanguard A/B Test | A/B test writeup | tech | reactive | 16/100 | 40-55 |

**Current differentiation:**
- Vanguard vs Meta: **2 points** (16 vs 18). Should be **20-30 points.**
- Rossmann vs Meta: 11 points (29 vs 18). Should be ~20 points. Directionally OK.
- Vanguard vs Rossmann: 13 points below (16 vs 29). Should be similar or Vanguard
  slightly higher (both are legitimate analyses with different gap profiles).

**The acceptance test for any calibration fix:** Vanguard and Rossmann both land
in Minor Fix range (40-55). Meta stays in Major Rework range (20-35). Clear tier
separation with a 15-25 point gap between tiers.

---

## Proposed Solution (For Next Session)

### Priority 1: Positive Credit System + CRITICAL Gradation

These two changes together address the differentiation failure directly:

**A. Positive Credit System** — award +5 to +8 for good analytical practices:

| Indicator | Credit |
|---|---|
| Pre-specified hypotheses with named KPIs | +5 to +8 |
| Experimental design (test vs. control) | +5 to +8 |
| Pre-specified significance threshold | +3 to +5 |
| Covariate balance check | +3 to +5 |
| Reports specific numbers (not vague claims) | +3 to +5 |
| External validation or benchmarking | +3 to +5 |

Cap at +25 per dimension to prevent positives from overwhelming negatives.

Effect: Vanguard gets +20 to +25 analysis credit. Meta gets +0 to +5. Gap widens
from 2 points to 20-25 points.

**B. CRITICAL Severity Gradation:**
- **CRITICAL-ABSENT:** Methodology/evidence doesn't exist. Full -20 deduction.
  Floor rule applies.
- **CRITICAL-INCOMPLETE:** Methodology/evidence exists but insufficiently reported.
  Reduced to -12 deduction. Floor rule does NOT apply.

Effect: Vanguard's "no stats reported" becomes CRITICAL-INCOMPLETE (-12 instead of
-20). Meta's "causal claim without methodology" stays CRITICAL-ABSENT (-20).

**Combined effect:** Vanguard moves from 16 to ~40-55 (Minor Fix). Meta stays
around 18-25 (Major Rework). That's real differentiation.

### Priority 2: Per-Dimension Deduction Cap (Safety Net)

Cap deductions at -70 per dimension. Prevents any real analysis from scoring below
30 on a dimension unless it's genuinely fabricated. This is a safety net, not the
primary fix.

### Priority 3: UX Sprint (Already Approved)

- ✅ Add emoji indicators to lens dashboard (approved)
- ✅ Use `> **✏️ Suggested rewrite:**` blockquote format for Top 3 fix
  examples (approved — one format, don't list alternatives)
- ✅ Compress per-lens detail sections to 1-2 sentences per finding by
  default (approved — no new flags, just make it shorter)

### What NOT to Do

- Don't just raise all scores uniformly (deduction cap alone). Makes everyone feel
  better but doesn't differentiate. Vanguard and Meta would both rise by ~15 points
  and still land near each other.
- Don't add `--format` as the primary fix. Genre awareness helps Meta specifically
  but doesn't solve differentiation within the same genre.
- Don't change the rubric lenses or finding definitions. The findings are mostly
  correct — the problem is how findings map to scores.

### Decision Needed

Product owner should choose which options to prototype. Recommendation is
**Positive Credits + CRITICAL Gradation + Deduction Cap**. An ADR (ADR-003) should
be created once decided.

---

## Files Referenced

- `dev/test-results/2026-02-15-vanguard-ab-test-full-review.md` — Test #3 full review
- `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md` — Test #1 full review
- `plugin/skills/ds-review-framework/SKILL.md` — current deduction table
- `dev/backlog.md` — v0.4 calibration sprint
- `dev/decisions/` — ADR-003 pending for calibration approach
