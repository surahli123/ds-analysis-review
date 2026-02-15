# Scoring Calibration Fix Plan — A3 Problem Analysis

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Owner:** Product Owner (surahli)
**Date:** 2026-02-15
**Status:** AWAITING OWNER APPROVAL — do not implement until approved
**Synthesized from:** 3 role reviews (PM Lead, Principal AI Engineer, DS Lead)

---

## 1. BACKGROUND (Why This Matters)

The DS Analysis Review Agent (v0.3) passed all 59 smoke test checks on synthetic
fixtures but fails on real-world documents. Three real-world tests produced scores
of 16, 18, and 29 on published, functional analyses. This makes the tool unusable:

- **No user trust.** A review tool that scores a polished blog post 18/100 will not
  get a second use. The PM Lead's analogy: "a spell checker that marks every paragraph
  as unintelligible."
- **No differentiation.** Vanguard (real experiment, pre-specified hypotheses, balance
  checks) scored LOWER than Meta (blog post, no experiment, vague claims). The system
  cannot tell good analytical work from bad analytical work.
- **Shipping blocker.** Calibration is the sole v0.4 objective. Until scores are
  reasonable and differentiated, the tool cannot be shared with anyone.

Three independent reviewers (Principal AI Engineer, PM Lead, Senior DS Lead) have
completed assessments. All three agree the system is broken. They disagree on
priorities and approach. This document synthesizes their feedback into a single
fix plan.

---

## 2. CURRENT CONDITION (What's Happening — Facts and Data)

### Scoring Data

| # | Document | Genre | Audience | Workflow | Score | CRITICALs | Findings |
|---|---|---|---|---|---|---|---|
| 1 | Meta LLM Bug Reports | Blog | exec | proactive | **18/100** | 4 | 15 |
| 2 | Rossmann Sales Prediction | Kaggle/README | mixed | proactive | **29/100** | 2 | 15 |
| 3 | Vanguard A/B Test | Course project | tech | reactive | **16/100** | 5 | 16 |

### The Differentiation Failure (The Core Problem)

Vanguard scored **2 points below** Meta (16 vs. 18). Any senior DS would rank
Vanguard significantly above Meta — it has a real experiment, pre-specified
hypotheses, balance checks, and quantitative results. Meta has none of these.

The scoring is blind to this because:
- Vanguard's 6+ analytical strengths earn **+0 points** (purely subtractive model)
- Both hit the same -88 deduction wall on analysis (ceiling effect)
- Both trigger 2+ CRITICAL floor rule → both capped at Major Rework

### The Math Behind the Ceiling Effect

| Metric | Value |
|---|---|
| Deduction budget per dimension | 100 points |
| Deduction capacity per dimension | 4 lenses × 3 findings × ~10 avg = **~120 points** |
| Result | Capacity exceeds budget. Any document with 2+ issues per lens mathematically scores near zero. |

Deduction totals across tests: -142 to -169. Remarkably consistent across three
very different documents. The scoring saturates — it cannot distinguish quality
levels because everything hits the same floor.

### Finding Quality Issue (DS Lead Unique Insight)

Of Meta's 15 findings, the DS Lead audited each one finding-by-finding:
- **5 fully agree** — real findings regardless of genre
- **4 partially agree** — valid finding, wrong severity
- **6 disagree** — genre-inappropriate (valid for internal analysis, wrong for blog)

The 6 disagreements: no failure mode analysis, playbook not actionable enough, no
story arc, no recommendations with owners, terminology section is MAJOR not MINOR,
limitations are cross-cutting double-counted. All are genre artifacts.

**Key insight the other reviews miss:** Fixing the scoring math without fixing
finding quality means reasonable scores for the wrong reasons. The user reads 6
irrelevant findings and loses trust regardless of the score.

### Bugs Found in Review Output (DS Lead Audit)

1. **Meta C7 (Actionability):** Severity labeled CRITICAL but deduction is -8 (MAJOR value). Mismatch.
2. **Meta C5 (Audience Fit):** Severity labeled MAJOR but deduction is -12 (CRITICAL value). Mismatch.
3. **Meta A3 + C5:** "No limitations" flagged by both subagents (-10 + -12 = -22). Routing table assigns this to communication-reviewer only.

---

## 3. GOAL/TARGET (What Success Looks Like)

### Primary Goal: Quality Differentiation

After calibration, the system must clearly separate "fixable analyses" from
"fundamentally flawed work." A senior DS should look at the scores and agree
with the tier placement.

### Target Scores

> **DECISION NEEDED — see Section 7.** The three reviewers and calibration notes
> disagree on Meta's target. Owner must decide before implementation.

**Option A: Calibration Notes Target (preserve tier separation)**

| Document | Current | Target | Verdict | Rationale |
|---|---|---|---|---|
| Vanguard | 16 | 40-55 | Minor Fix | Real experiment, fixable gaps |
| Rossmann | 29 | 45-60 | Minor Fix | End-to-end ML project, reporting gaps |
| Meta | 18 | 20-35 | Major Rework | No methodology, vague claims |

Gap between tiers: 15-25 points. Clear separation.

**Option B: DS Lead Target (all Minor Fix, within-tier differentiation)**

| Document | Current | Target | Verdict | Rationale |
|---|---|---|---|---|
| Vanguard | 16 | 42-48 | Minor Fix | Strong foundations, missing stats |
| Rossmann | 29 | 50-60 | Minor Fix | Most complete of the three |
| Meta | 18 | 45-55 | Minor Fix | Published blog with real communication quality |

Gap between documents: 5-15 points. Same tier, different positions within it.

**Acceptance test (regardless of option):**
1. Vanguard scores ABOVE Meta (currently below)
2. Clear numeric separation between strongest and weakest document (15+ points)
3. Vanguard and Rossmann in Minor Fix range (40-60)
4. No document scores below 20 unless genuinely fabricated/incoherent

---

## 4. ROOT CAUSE ANALYSIS (Why the Problem Exists)

### Where All Three Reviewers Agree (High-Confidence Root Causes)

**RC1: No credit for analytical strengths (ALL THREE AGREE — PRIMARY)**

The scoring model is purely subtractive: 100 minus deductions. Positive findings
appear in narrative output but have zero score impact. This is the direct cause of
the differentiation failure — Vanguard's 6+ strengths are worth exactly 0 points.

- Engineer: Does not propose strength credits but acknowledges the math problem
- PM Lead: "A review that finds 3 genuine strengths and then delivers 16/100 feels
  dishonest." Proposes +3 to +5, cap +15/dimension.
- DS Lead: "This is the single most important change." Proposes +3 to +8, cap
  +25/dimension. Shows Vanguard's strengths are worth +25-29 in their scoring.

**RC2: Deduction stacking without diminishing returns (ALL THREE AGREE)**

7 findings from 3 root causes generate -88 deductions per dimension. Correlated
findings are treated as independent events. The math saturates at ~15 findings.

- Engineer: Identified 3 root cause clusters in Meta's 7 analysis findings.
  Reasonable deduction for 3 problems = ~-40. Actual: -88.
- PM Lead: Capacity (120) exceeds budget (100). Predictable ceiling effect.
- DS Lead: Agrees as safety net. "Diminishing returns prevents catastrophic scores
  but doesn't create differentiation."

**RC3: CRITICAL over-assigned (ALL THREE AGREE)**

The CRITICAL definition ("fundamental flaw that would cause the analysis to fail
its purpose") is too broad. Structural gaps (missing TL;DR, no story arc) get
CRITICAL alongside genuinely dangerous flaws (causal claim without methodology).

All three agree on reclassifying:
- Missing TL;DR: CRITICAL (-15) → MAJOR (-10)
- No clear story arc: CRITICAL (-12) → MAJOR (-8)

PM and DS Lead also agree on:
- Limitations/scope absent: CRITICAL (-12) → MAJOR (-10)

DS Lead uniquely adds:
- "No recommendations with owners" should never be CRITICAL

**RC4: Output reads like a teardown (PM + DS LEAD AGREE)**

The review leads with a devastating score, then an all-red dashboard, then
CRITICAL findings. Positives are buried at the end. The Vanguard review is
2,000 words of findings on a 1,125-word document. The review is nearly 2x the
length of the document being reviewed.

### Where Reviewers Disagree (Tradeoffs to Resolve)

**D1: Primary fix priority**

| Reviewer | What they say is primary |
|---|---|
| Engineer | Diminishing returns (math fix first, easiest to implement) |
| PM Lead | Diminishing returns + CRITICAL reclassification together |
| DS Lead | Strength credits (differentiation fix first — math fix alone doesn't differentiate) |

**Tradeoff:** Diminishing returns is simpler (one formula, no subagent changes) but
produces similar scores for Meta and Vanguard (both ~50). Strength credits require
changes to SKILL.md, both subagent prompts, and the lead agent, but create the
15-20 point gap that makes the tool useful. DS Lead's argument is stronger on
user outcomes — differentiation is the core product problem.

**D2: Strength credit cap**

| Reviewer | Cap |
|---|---|
| PM Lead | +15 per dimension |
| DS Lead | +25 per dimension |
| Calibration Notes | +25 per dimension |

**Tradeoff:** +15 gives Vanguard ~12 extra points (not enough to clearly separate
tiers). +25 gives Vanguard ~20-25 extra points (creates clear gap from Meta). DS
Lead's math shows +15 is too modest for differentiation. PM Lead's concern is that
high credits could let strengths overwhelm legitimate gaps.

**D3: Finding volume cap**

| Reviewer | Position |
|---|---|
| Engineer | No cap — let diminishing returns handle it |
| PM Lead | Notes output is too long, doesn't propose explicit cap |
| DS Lead | Cap at 8 total findings (5 by severity + up to 3 MINOR) |

**Tradeoff:** A cap reduces deduction pressure and output length, but throws away
information. Diminishing returns lets the review surface all findings without
score devastation. The DS Lead's coaching perspective: "I give my team 3-5 things
to fix, not 15." The Engineer's systems perspective: "Don't throw away information."

**D4: Meta's target score (the biggest disagreement)**

| Source | Meta Target | Rationale |
|---|---|---|
| Calibration Notes | 20-35 (Major Rework) | Blog with no methodology; fundamentally flawed analysis |
| PM Lead | 50-65 (Minor Fix) | After diminishing returns, all documents cluster around 50 |
| DS Lead | 45-55 (Minor Fix) | Published, well-received blog; communication quality is high |

**Tradeoff:** If Meta stays in Major Rework, the tool is saying "this published
blog post needs a complete redo." If Meta is Minor Fix, the tool is saying "solid
blog, fix a few things." The DS Lead's frame: a blog post evaluated as a blog
shouldn't score Major Rework. The calibration notes' frame: the analytical claims
are genuinely dangerous regardless of genre.

**This is the most important decision in this plan.** It determines the
calibration target and therefore which fix combinations are necessary.

**D5: CRITICAL-ABSENT vs. CRITICAL-INCOMPLETE gradation**

| Source | Position |
|---|---|
| Calibration Notes | Add it — distinguishes "methodology absent" from "methodology incomplete" |
| DS Lead | Don't add yet — try reclassification first, add gradation if needed |
| Engineer | Not discussed |
| PM Lead | Not discussed |

**Tradeoff:** Gradation is conceptually elegant (Vanguard's "stats not reported"
≠ Meta's "no experiment at all") but adds complexity. Reclassification + strength
credits may be sufficient. Try the simpler approach first.

---

## 5. COUNTERMEASURES (Ordered Fix Plan)

### Phase 1: P0 — Ship Together (Interactions Between Fixes)

These three fixes interact and should be implemented + tested as a unit.

#### Fix 1: Add Strength Credits to Scoring

**Why first:** This is the only fix that creates differentiation between "did the
work" and "didn't do the work." Without it, diminishing returns just lifts all
scores equally — Meta and Vanguard both rise ~30 points but stay 2 points apart.

**Files to modify:**

| File | What Changes |
|---|---|
| `plugin/skills/ds-review-framework/SKILL.md` | New Section 2b: Strength Credit Table (analysis + communication strengths with credit values) |
| `plugin/agents/analysis-reviewer.md` | Add STRENGTH LOG to output format (after POSITIVE FINDINGS) |
| `plugin/agents/communication-reviewer.md` | Add STRENGTH LOG to output format (after POSITIVE FINDINGS) |
| `plugin/agents/ds-review-lead.md` | Step 9: collect strength credits, apply cap, add to dimension score before averaging |

**Specific changes:**

Add to SKILL.md — new Section 2b "Strength Credit Table":

Analysis strengths:
| Strength | Credit |
|---|---|
| Pre-specified, testable hypotheses with KPIs | +5 |
| Real experimental/quasi-experimental design | +8 |
| Pre-specified practical significance threshold | +5 |
| Validation against ground truth | +5 |
| Reports specific quantitative results (not vague claims) | +3 |
| Covariate balance / confound check performed | +5 |
| Limitations acknowledged | +3 |
| Sensitivity / robustness analysis included | +5 |

Communication strengths:
| Strength | Credit |
|---|---|
| Clear TL;DR or executive summary present | +5 |
| Argument follows logical arc appropriate for audience | +5 |
| Visualizations directly support narrative | +3 |
| Recommendations specific and actionable | +5 |
| Appropriate detail level for stated audience | +3 |

Cap: **DECISION NEEDED** — +15 or +25 per dimension (see Section 7).

Add to subagent output formats (both agents):
```
STRENGTH LOG:
- [Strength from SKILL.md table] → [credit amount]
Total credits: [sum, before cap]
```

Update ds-review-lead Step 9:
- Collect STRENGTH LOG from each subagent
- Apply per-dimension cap
- New formula: dimension_score = 100 - effective_deductions + capped_credits
- Score ceiling is 100 (credits cannot exceed deductions if document scores well)

#### Fix 2: Reclassify Structural CRITICALs

**Why:** Reduces false CRITICAL triggers that fire the floor rule on nearly every
document. Combined with Fix 1, the floor rule only fires for genuinely dangerous
analytical flaws.

**Files to modify:**

| File | What Changes |
|---|---|
| `plugin/skills/ds-review-framework/SKILL.md` | Section 1: tighten CRITICAL definition. Section 2: change severity/deduction for 3-4 entries. |

**Specific changes to SKILL.md Section 1:**

Update CRITICAL definition:
> **CRITICAL:** A fundamental flaw where the reader would reach a **wrong conclusion**
> or make a **harmful decision** based on what is written. The analysis presents
> incorrect or dangerously unsupported claims as established fact. Structural,
> formatting, and completeness gaps — even significant ones — cap at MAJOR.

**Specific changes to SKILL.md Section 2 (Communication Deduction Table):**

| Entry | Current | New |
|---|---|---|
| Missing or ineffective TL;DR | CRITICAL, -15 | **MAJOR, -10** |
| No clear story arc or structure | CRITICAL, -12 | **MAJOR, -8** |
| Limitations/scope unclear for downstream | CRITICAL, -12 | **MAJOR, -10** |

**DECISION NEEDED:** Also reclassify "No named owner or next step" from the
Actionability section? DS Lead argues it should never trigger CRITICAL via the
proactive workflow path. Currently MINOR (-5) in the table, but the agent
sometimes elevates it via proactive workflow context. (See Section 7.)

#### Fix 3: Diminishing Returns Formula

**Why:** Safety net that prevents catastrophic scores even if findings are
numerous. All three reviewers agree on the formula.

**Files to modify:**

| File | What Changes |
|---|---|
| `plugin/agents/ds-review-lead.md` | Step 9: add diminishing returns computation before score calculation |
| `plugin/skills/ds-review-framework/SKILL.md` | New Section 3b: document the diminishing returns curve |

**Formula (all three reviewers agree on this exact formulation):**

```
Per-dimension effective deductions:
  Raw deductions 0-30:    100% applied
  Raw deductions 31-50:   50% applied
  Raw deductions 51+:     25% applied
```

**Example computation (Meta analysis, raw -88):**
- First 30: -30
- Next 20 (of remaining 58): -10
- Remaining 38 at 25%: -9.5
- Effective deduction: -49.5 → score before credits: ~50

**Update to ds-review-lead Step 9 synthesis logic:**

After collecting deduction logs from subagents, before computing dimension score:
1. Sum raw deductions per dimension
2. Apply diminishing returns curve to get effective deductions
3. Apply strength credits (from Fix 1)
4. Dimension score = 100 - effective_deductions + capped_credits (floor 0, ceiling 100)
5. Final score = average of both dimension scores
6. Apply floor rules on verdict (unchanged)

Document the curve in SKILL.md new Section 3b with the formula and a worked example.

---

### Phase 1 Projected Impact

With all three P0 fixes applied, projected scores:

**Assumption: +25 cap on strength credits**

| Document | Analysis (old → new) | Comms (old → new) | Final (old → new) | Verdict |
|---|---|---|---|---|
| Vanguard | 12 → ~48-55 | 19 → ~42-48 | 16 → ~45-52 | Minor Fix |
| Rossmann | 39 → ~55-62 | 19 → ~42-48 | 29 → ~48-55 | Minor Fix |
| Meta | 12 → ~38-45 | 24 → ~45-50 | 18 → ~42-48 | Minor Fix (low) |

**Key check:** Vanguard now scores ABOVE Meta on analysis (~50 vs ~40) due to
strength credits. Meta still leads on communication (~48 vs ~45). Overall
differentiation: ~5-10 points, same tier. This matches DS Lead's Option B targets.

**If we want Option A** (Meta in Major Rework, 15-25 point gap), we would also
need the finding volume cap (Fix 4) and/or the CRITICAL-INCOMPLETE gradation.
Diminishing returns + strength credits alone are unlikely to keep Meta below 40.

---

### Phase 2: P1 — After Phase 1 Validation

Implement these after running tests with Phase 1 fixes and confirming direction.

#### Fix 4: Reduce Finding Volume (DECISION NEEDED)

**Proposed by DS Lead only.** Cap at 8 total findings across both dimensions.

**Files to modify:**

| File | What Changes |
|---|---|
| `plugin/agents/ds-review-lead.md` | Step 9: after collecting all findings, select top 8 by severity/deduction. Subsume derivative findings under parent root cause. |

**Specific logic for Step 9:**
1. Collect all findings from both subagents
2. Rank by severity (CRITICAL > MAJOR > MINOR), then by deduction size
3. Select top 5 highest-severity findings
4. Add up to 3 MINOR polish items
5. For scoring: only count the 8 selected findings' deductions
6. For output: mention subsumed findings in a "Also noted" one-liner list

**Why this helps:** Fewer findings = fewer deductions = less need for diminishing
returns to rescue the score. Also makes the output shorter and more actionable.

**Why it's controversial:** Engineer says "Don't throw away information." The review
should still surface all findings. Finding cap changes the review's personality from
"exhaustive audit" to "coaching review."

**Owner decision:** Include this or not? If yes, implement alongside Phase 1.

#### Fix 5: Output Restructure

**Agreed by PM Lead and DS Lead.** Restructure the lead agent output template.

**Files to modify:**

| File | What Changes |
|---|---|
| `plugin/agents/ds-review-lead.md` | Step 10: reorder output sections, compress per-lens detail, add emoji dashboard, add review summary |

**New Full Mode output order (Step 10):**

1. `# DS Analysis Review: [Document Title]`
2. `**Score: [X]/100 — [Verdict]**` + floor rule if applied
3. **NEW:** One-sentence review summary (DS Lead addition)
   Example: *"Well-structured experiment with strong analytical foundations — primary gap is missing statistical inference."*
4. Metadata line (mode, audience, workflow, tier, word count)
5. Lens Dashboard with emoji indicators (already approved):
   - SOUND → ✅ | MINOR ISSUES → ⚠️ | MAJOR ISSUES → ❌ | CRITICAL → ❌
6. **`## What You Did Well`** (MOVED — before findings, was after)
7. `## Top 3 Priority Fixes` — full detail with blockquote rewrite format:
   ```
   > **✏️ Suggested rewrite:**
   > [rewrite text here]
   ```
8. `## Analysis Dimension (Score: [X]/100)` — **compressed**: 1-2 sentences per
   finding (title + core issue only). No location, no multi-sentence issue, no
   suggested fix outside the Top 3.
9. `## Communication Dimension (Score: [X]/100)` — same compressed format

**Effect:** Cuts output roughly in half. Front-loads trust-building content. The
user reads strengths before criticism.

#### Fix 6: Fix 3 Bugs Found by DS Lead

**Files to modify:**

| File | What Changes |
|---|---|
| Root cause is in agent behavior, not file content | Investigate whether bugs are in SKILL.md definitions or agent interpretation |

**Bug 1:** C7 (Actionability) labeled CRITICAL but deducted -8 (MAJOR value).
- **Likely fix:** The deduction table has "Vague recommendation or answer" at
  MAJOR (-8) and "No named owner or next step" at MINOR (-5). The agent is using
  the -8 deduction (correct for MAJOR) but labeling it CRITICAL (wrong). This is
  an agent interpretation error. Fix by adding a clarifying instruction to the
  communication-reviewer prompt: "Use the severity from the deduction table entry
  you are applying. Do not elevate severity beyond what the table specifies."

**Bug 2:** C5 (Audience Fit) labeled MAJOR but deducted -12 (CRITICAL value).
- **Likely fix:** "Limitations/scope unclear for downstream" is CRITICAL (-12) in
  the table, but the agent labeled it MAJOR. This one will be fixed by Fix 2
  (reclassifying this to MAJOR -10). After reclassification, the label and
  deduction will match.

**Bug 3:** A3 + C5 cross-cutting double-count on "no limitations."
- **Fix:** The routing table (Section 5) assigns "Missing limitation" to
  communication-reviewer. Add explicit instruction to analysis-reviewer prompt:
  "Per Section 5 routing table, 'Missing limitation' is owned by communication-
  reviewer. Do not flag it independently. You may note the analytical IMPACT of
  missing limitations (e.g., 'conclusions should be qualified') but assign the
  finding to Methodology & Assumptions as a scope issue, not Completeness."

---

### Deferred to v0.5

| Item | Why Defer |
|---|---|
| Genre/format auto-detection | All 3 reviewers agree: fix scores first. DS Lead recommends designing for v0.5 (affects finding generation, not just scoring) |
| CRITICAL-ABSENT vs. CRITICAL-INCOMPLETE | Try reclassification first. Add gradation only if Vanguard's "no stats" and Meta's "no experiment" still need differentiation after strength credits |
| Per-dimension deduction hard cap | Diminishing returns makes this redundant (Engineer, PM Lead agree) |
| Cluster-based scoring | Engineer recommends for v1.5. Most accurate model but requires root-cause clustering in synthesis — too complex for v0.4 |

---

## 6. IMPLEMENTATION PLAN (Dependency Order)

### Phase 1 Implementation Sequence

Fixes 1-3 interact and should be validated together. Recommended implementation order:

```
Fix 2 (SKILL.md severity reclassification)
  ↓  no dependency — can run in parallel with Fix 3
Fix 3 (SKILL.md + lead agent diminishing returns)
  ↓  depends on: nothing
Fix 1 (Strength credits — SKILL.md + both subagents + lead agent)
  ↓  depends on: Fix 2 done (severity definitions settled before credits)
Validation: Rerun 3 real-world tests
  ↓  depends on: Fixes 1-3 all done
```

**Step-by-step:**

**Step 1:** Update SKILL.md Section 1 — tighten CRITICAL definition *(Fix 2)*
**Step 2:** Update SKILL.md Section 2 — reclassify 3 communication CRITICALs to MAJOR *(Fix 2)*
**Step 3:** Add SKILL.md Section 2b — strength credit table *(Fix 1)*
**Step 4:** Add SKILL.md Section 3b — diminishing returns documentation *(Fix 3)*
**Step 5:** Update analysis-reviewer.md — add STRENGTH LOG to output format *(Fix 1)*
**Step 6:** Update communication-reviewer.md — add STRENGTH LOG to output format *(Fix 1)*
**Step 7:** Update ds-review-lead.md Step 9 — new synthesis logic:
  - Collect strength logs from subagents
  - Apply diminishing returns to raw deductions
  - Apply capped strength credits
  - Compute dimension scores with new formula
  *(Fixes 1 + 3)*
**Step 8:** Rerun Vanguard test → check analysis score rises, overall > Meta
**Step 9:** Rerun Meta test → check score lands in target range
**Step 10:** Rerun Rossmann test → check score lands in target range
**Step 11:** Compare tier separation — does the acceptance test pass?

### Phase 2 Implementation (After Phase 1 Validation)

**Step 12:** Update ds-review-lead.md Step 10 — output restructure *(Fix 5)*
**Step 13:** Fix bug 1 in communication-reviewer.md — severity consistency note *(Fix 6)*
**Step 14:** Fix bug 3 in analysis-reviewer.md — limitations routing note *(Fix 6)*
**Step 15:** *(If owner approves)* Add finding volume cap to ds-review-lead.md Step 9 *(Fix 4)*
**Step 16:** Rerun all 3 tests + 2-3 synthetic fixtures to verify floor rules still work
**Step 17:** Run 2-3 untested real-world fixtures for consistency
**Step 18:** Run cross-consistency test (same doc 3x, scores within ±10)

---

## 7. DECISIONS NEEDED (Owner Input Required)

### Decision 1: Strength Credit Cap — +15 or +25 per dimension?

| Option | Who Proposes | Effect on Vanguard | Tradeoff |
|---|---|---|---|
| +15 | PM Lead | ~+12 points analysis | Modest differentiation. Meta and Vanguard stay ~8 points apart. Safer — credits can't overwhelm gaps. |
| +25 (Recommended) | DS Lead + Calibration Notes | ~+20-25 points analysis | Strong differentiation. Creates 15-20 point gap. Risk: very strong documents might score too high. Mitigated by: credits only fire for things the document actually does. |

**My recommendation:** Start with +25. It's easier to reduce than to increase later,
and the primary user problem (no differentiation) requires meaningful credit values.
At +15, the PM Lead's own projections show all three documents clustering at ~50-55
with 4-point spread. That's not enough differentiation.

### Decision 2: Meta's Target Score

| Option | Target | Verdict | Implication |
|---|---|---|---|
| A: Tier Separation | 20-35 | Major Rework | Tool says "this needs fundamental work." Blog author would disagree — it's published and well-received. |
| B: Within-Tier (Recommended) | 42-50 | Minor Fix (low) | Tool says "solid piece, fix the vague claims." Blog author would find this reasonable. Meta's real gaps (causal claim, vague metrics) are flagged as the top fixes. |

**My recommendation:** Option B. A published, well-received blog post scoring Major
Rework breaks user trust. The DS Lead's reframe is persuasive: "The question is
whether this teaches the reader something useful, and it does." The causal claim
and vague metrics are real findings — they'll be the #1 and #2 priority fixes
regardless of the score. Major Rework vs. Minor Fix changes the headline, not the
actionable feedback.

**If you choose Option A:** We would need the CRITICAL-INCOMPLETE gradation AND the
finding volume cap to push Meta below 40 while keeping Vanguard above 40. This
adds complexity to Phase 1.

### Decision 3: Finding Volume Cap — Include or Defer?

| Option | Effect | Tradeoff |
|---|---|---|
| Include in Phase 1 | 8 findings max. Shorter output, less deduction pressure. | Throws away information. Changes the review personality from "exhaustive" to "coaching." |
| Defer to Phase 2 (Recommended) | Test Phase 1 first. Add cap only if output is still too long or scores are still too compressed. | Keeps all findings visible. Let diminishing returns + strength credits do the work first. |

**My recommendation:** Defer. Implement Phase 1, validate scores, THEN decide.
If scores differentiate well with just the math fixes, the cap is unnecessary.
If the output is still too long, add the cap in Phase 2.

### Decision 4: "No Recommendations with Owners" — Reclassify?

The DS Lead argues this finding should never reach CRITICAL. It's currently
MINOR (-5) in the deduction table, but the agent sometimes elevates it via
proactive workflow context. The Vanguard and Meta reviews both show this
elevation happening.

| Option | Effect |
|---|---|
| Add explicit instruction to not elevate beyond table severity | Prevents agent from using workflow context to escalate MINOR to CRITICAL |
| Leave as-is | Risk of continued CRITICAL over-assignment on blog/portfolio documents |

**Recommended:** Add the instruction. This is a bug, not a calibration question.

### Decision 5: Diminishing Returns — Include or Not?

All three reviewers agree on including it. The DS Lead calls it a "safety net,
not the primary fix." Including it has no downside — it prevents catastrophic
scores without affecting well-scoring documents.

**Recommended:** Include. No decision needed unless you disagree.

---

## 8. FOLLOW-UP (Verification & Prevention)

### Verification Plan

**After Phase 1 (Fixes 1-3):**

| Test | Current | Target | Pass Criteria |
|---|---|---|---|
| Vanguard | 16 | 40-55 | Score > Meta by 3+ points on analysis dimension |
| Rossmann | 29 | 45-60 | Score in Minor Fix range |
| Meta | 18 | 42-50 (Option B) or 20-35 (Option A) | Verdict matches owner's target tier |
| **Tier separation** | 2-point gap | 15+ point gap | Widest gap between any two documents is 15+ points on the analysis dimension |
| **CRITICAL count** | 4-5 per test | 1-2 per test | Only genuinely misleading findings get CRITICAL |

**After Phase 2 (Fixes 4-6):**

| Test | Pass Criteria |
|---|---|
| Rerun 3 real-world fixtures | Scores stable (within ±5 of Phase 1 results) |
| Rerun 2-3 synthetic fixtures | Floor rules still fire correctly on designed-to-fail fixtures |
| Cross-consistency (same doc 3x) | Scores within ±10 |
| 2-3 untested real-world fixtures | Scores feel reasonable to product owner gut-check |

### Prevention

- **Create ADR-003** documenting the calibration approach after owner approves
- **Add calibration test suite** to the backlog (automated checks that Vanguard > Meta, all in target ranges)
- **Session end protocol:** Update dev/backlog.md and CHANGELOG.md after implementation

---

## APPENDIX: Reviewer Agreement Matrix

Quick reference for which reviewer supports which fix.

| Fix | Engineer | PM Lead | DS Lead | Consensus? |
|---|---|---|---|---|
| Diminishing returns | ✅ Primary | ✅ Fix 1 | ✅ Safety net | **Yes — all agree on inclusion and formula** |
| CRITICAL reclassification | ✅ TL;DR + arc | ✅ + limitations | ✅ + recommendations | **Yes — agree on core 2, differ on scope** |
| Strength credits | Not proposed | ✅ Cap +15 | ✅ Cap +25 | **PM + DS agree on concept; differ on cap** |
| Finding volume cap | ❌ Don't cap | Partial (output too long) | ✅ Cap at 8 | **No consensus — DS Lead only** |
| Output restructure | Not addressed | ✅ Strengths before findings | ✅ + review summary | **PM + DS agree** |
| Genre/format detection | Defer | Defer | Defer (design for v0.5) | **Yes — all defer** |
| CRITICAL gradation (ABSENT/INCOMPLETE) | Not discussed | Not discussed | Defer | **Not needed yet — try reclassification first** |
