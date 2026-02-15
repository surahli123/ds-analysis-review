# Senior DS Lead Assessment: Scoring Calibration

**Reviewer:** Senior DS Lead (manages 5-8 data scientists, regularly reviews analyses before stakeholder delivery)
**Date:** 2026-02-15
**Scope:** Full Meta blog review output, Vanguard A/B test review output, scoring framework (SKILL.md), all agent prompts, Principal AI Engineer assessment, PM Lead review, calibration notes, both source documents
**Status:** Review complete. Assessment only — no implementation.

---

## Part 1: Finding-by-Finding Audit — Meta Blog Review

I'm going through every finding in `dev/test-results/2026-02-15-meta-llm-bug-reports-review.md` and evaluating each one as if I were the DS lead whose team member wrote this blog post and someone handed me this review.

### Analysis Dimension Findings

**Finding 1: Causal claims without supporting methodology (CRITICAL, -20)**

**I agree this is a real finding, but I'd calibrate it differently.**

The sentence "ultimately reducing topline bug reports by double digits over the last few months" IS a causal claim ("our approach reduced") without supporting evidence. If one of my team members wrote this in an internal analysis, I'd circle it immediately. Causal language without a counterfactual is one of the most common mistakes I correct.

However, I would word this differently. The review treats it as a binary: "no A/B test, no quasi-experimental design, no counterfactual" — as if the only options are formal causal inference or nothing. In practice, I'd tell my DS: "You can say 'we observed a double-digit decline in bug reports during the period the system was active' without claiming causation. Or better yet, show a time-series chart with the intervention point marked and let the reader draw their own conclusions." The fix is a reframe, not a full experimental redesign.

For a blog post, I'd make this **MAJOR** (-10), not CRITICAL. For an internal analysis driving investment, CRITICAL is right. The blog isn't asking anyone to fund a $10M initiative based on this claim — it's sharing methodology.

**Verdict: Agree with finding, disagree with severity. MAJOR for a blog, CRITICAL for internal analysis.**

---

**Finding 2: Unstated critical assumptions about LLM classification accuracy (CRITICAL, -20)**

**I agree this is a real finding, and it's actually the most important one the agent caught.**

This is the one I'd push hardest on in a real review. The entire value proposition rests on "LLM classifies bug reports accurately enough to drive product decisions," and there's zero evidence presented. No precision/recall, no comparison to human labels, no error analysis. If the classification is 60% accurate, every dashboard and trend signal is noise.

That said, in a public blog post, there are legitimate reasons not to disclose internal accuracy numbers (competitive advantage, NDA constraints). I'd still flag it — you can say "we achieved X% agreement with human reviewers on a held-out set" without revealing proprietary details.

For a blog, I'd make this **MAJOR** (-10). The blog reader can't adopt this system anyway — they're reading for inspiration, not for a production runbook. For an internal analysis, CRITICAL is correct.

**Verdict: Agree strongly with finding. Severity depends entirely on context. MAJOR for blog, CRITICAL for internal.**

---

**Finding 3: No limitations acknowledged anywhere (MAJOR, -10)**

**Partially agree — the review missed a limitation that IS stated.**

The blog says: "achieving reliable and actionable results requires upfront investment in tuning and iterations." That IS a limitation acknowledgment. It's saying: this doesn't work out of the box, you need domain expertise and iteration. The review should have caught this and given partial credit.

That said, the blog IS missing the limitations I'd want to see: selection bias in who files bug reports (power users over-represented), LLM hallucination risk in root-cause analysis, coverage gaps for non-text feedback, token limit issues for long reports. These are legitimate gaps.

For a blog post, I'd make this **MINOR** (-5). Blog posts rarely have formal limitations sections — but the best ones weave caveats into the narrative. For an internal analysis, MAJOR is right.

**Verdict: Partially agree. Agent missed an existing limitation statement. Severity too high for genre.**

---

**Finding 4: Unsupported logical leap from detection to impact (MAJOR, -10)**

**I agree with the finding but the review's framing is too academic.**

The logic chain is: LLM detects patterns → teams fix bugs → bug reports decrease. The missing link is "teams fix bugs BECAUSE of LLM insights specifically (not because they would have found them anyway)." The outage detection example actually weakens the case — that outage was already visible through other channels.

The stronger evidence is the "less visible bugs" claim. If the LLM surfaces bugs that would otherwise go undetected, THAT'S the value prop. But the blog doesn't provide examples of these less-visible bugs or quantify how many were surfaced.

In a real review, I'd say: "Your best story is the bugs nobody else would have found. Can you give me 2-3 specific examples? The outage story is good for color but doesn't prove unique value."

**Verdict: Agree with finding. MAJOR is right. Would reword to be more constructive.**

---

**Finding 5: Missing obvious analysis: no evaluation of failure modes (MAJOR, -8)**

**I disagree for a blog post.**

In a real internal analysis, yes, I'd ask "where does this fail?" But in a published blog post? Failure mode analysis is not a standard blog component. I've never reviewed a team member's blog draft and said "you need a failure mode section before this goes on Medium."

If this were an internal analysis recommending company-wide adoption, absolutely flag it. For thought leadership content, this is noise.

**Verdict: Disagree for blog context. Would not flag this. For an internal analysis, agree with MAJOR.**

---

**Finding 6: No baseline or benchmark for LLM performance claims (MAJOR, -10)**

**I agree — this one bugs me even for a blog.**

"LLMs excel" compared to what? "Traditional ML models fall short" — at what, specifically? These are unsupported superiority claims. Even a single sentence — "In our testing, LLM classification achieved X% accuracy versus Y% for our previous rule-based system" — would transform the credibility of the piece.

I've reviewed dozens of "we tried ML and it was great" blog posts. The ones that include even a rough comparison are 10x more credible. The ones that say "it's better" without any numbers read like marketing copy.

**Verdict: Agree. MAJOR is right. This applies regardless of genre.**

---

**Finding 7: "Double digits" metric without specificity (MAJOR, -10)**

**I strongly agree — this is actually the finding that should hit hardest.**

"Double digits" is not a metric. Is it 10% or 99%? "Over the last few months" — is that 2 months or 8? What's the baseline volume? If you had 100 bug reports and now you have 85, that's a very different story than going from 10,000 to 7,000.

This is the kind of vagueness I red-pen every single time. Even in a blog post. If you're going to make a quantitative claim, make it specific. If you can't disclose the exact number, say "more than 20%" or "roughly a third." "Double digits" is evasive.

I'd actually elevate this to **MAJOR** and make it the #1 finding, ahead of the causal claim. The causal framing is a methodology issue; the vague metric is a credibility issue that affects every reader of the blog.

**Verdict: Strongly agree. Would make this the #1 finding. MAJOR is right; arguably should be the highest-priority fix.**

---

### Communication Dimension Findings

**Finding 1: Missing effective TL;DR (CRITICAL, -15)**

**I agree it's a finding, but CRITICAL is wrong for a blog post.**

The blog opens with context-setting, which is a standard blog convention. Blog readers expect a narrative hook, not a TL;DR. That said, the blog's opening IS weaker than it could be — it leads with philosophy rather than the interesting thing ("we used LLMs to classify 10M+ bug reports and reduced our backlog by X%").

For an internal Confluence page to an exec, CRITICAL is right — bury the lede in a stakeholder analysis and you've failed. For a Medium post, this is MAJOR at most.

**Verdict: Agree with finding. Severity should be MAJOR (-10), not CRITICAL (-15).**

---

**Finding 2: No clear story arc — flat "here's what we did" structure (CRITICAL, -12)**

**I disagree with this finding.**

The blog DOES have a story arc: Problem (unstructured feedback is hard to analyze) → Solution (LLMs for classification) → Evidence (case studies showing value) → Playbook (how to replicate) → Scale (production deployment). That is a coherent arc. It's not the "What → So What → Now What" framework the rubric demands for exec/proactive, but it IS a standard blog structure.

Applying the exec/proactive communication template to a Medium blog post is like grading a journal article against slide deck conventions. The structure serves a different audience and purpose.

If I were reviewing this for my team, I'd say: "The structure works for a blog. If you ever adapt this into an internal recommendation, restructure around What → So What → Now What." I would NOT give this CRITICAL.

**Verdict: Disagree. Would not flag this, or MINOR at most. The blog has a logical arc; it's just not the exec-analysis arc.**

---

**Finding 3: Generic headings that don't telegraph findings (MINOR, -3)**

**I agree — fair, low-impact feedback.**

"The benefit of LLM" is a weak heading. "LLMs catch bug patterns human reviewers miss" would be better. This is polish feedback, and MINOR is the right severity.

**Verdict: Agree. MINOR is right.**

---

**Finding 4: Terminology section inappropriate for exec audience (MAJOR, -10)**

**I agree with the substance but the context is wrong.**

Defining "dashboard" for any audience is patronizing. That said, this finding is amplified by the `--audience exec` parameter. On Medium, the audience IS general/mixed, and a terminology section is debatable but not absurd. The finding is half genuine (the section is unnecessary) and half parameter-artifact (it's MAJOR because we told the agent the audience is execs).

I'd flag it as MINOR: "Consider whether your audience needs these definitions. Most readers of this blog will already know what a dashboard is."

**Verdict: Agree the section is unnecessary. But MAJOR is too harsh — this is MINOR for a blog audience.**

---

**Finding 5: Limitations and scope boundaries absent for downstream consumers (MAJOR, -12)**

**This is a cross-cutting double of Analysis Finding 3.**

The review notes the cross-cutting nature but still applies a full -12 deduction. Combined with the analysis-side -10 for the same root cause, that's -22 total for "no limitations." This is exactly the double-counting the PM Lead flagged in Critique 4.

Also: the SKILL.md deduction table has this as CRITICAL (-12), but the review classified it as MAJOR. That's either an error in the review or the agent independently downgraded — either way, it's inconsistent.

**Verdict: Valid root cause, but this is double-counted and the severity is inconsistent with the deduction table.**

---

**Finding 6: Playbook section is descriptive, not actionable (MAJOR, -8)**

**I disagree for a blog post.**

The playbook section is actually one of the best parts of the blog. It gives a replicable four-step framework: Classification → Trend Monitoring → Root Cause Analysis → Inform Fixes. For a blog reader, that IS actionable — it tells you the sequence and what each step does.

Expecting effort estimates, per-step outcomes, and prioritization signals in a public blog is unreasonable. That's internal planning detail. The playbook as written is appropriate for its genre.

**Verdict: Disagree. Would not flag this for a blog post. For an internal analysis, agree with MAJOR.**

---

**Finding 7: No specific recommendations with owners or next steps (CRITICAL in finding, -8 in deduction)**

**I disagree, and there's a scoring inconsistency.**

First, the substance: expecting named owners and timelines from a public blog post is absurd. Blog posts don't have "owners" for recommendations. The conclusion saying "teams across any product area can gain scalable, quantitative insights" IS the recommendation for a blog: "you can do this too."

Second, the inconsistency: the finding is labeled CRITICAL in the body, but the deduction logged is -8, which matches the MAJOR deduction for "Vague recommendation or answer" in the deduction table. The severity and the deduction don't match. This is a bug.

**Verdict: Disagree for blog context. Also a severity/deduction mismatch (bug). For internal analysis, agree with MAJOR.**

---

**Finding 8: "So what?" missing for the primary quantitative claim (MAJOR, -8)**

**I agree — this is valid regardless of genre.**

"Double digits reduction" without business translation (engineering hours saved, user satisfaction impact, cost reduction) is a missed opportunity. Even a blog reader wants to know "is this a big deal?" Saying "equivalent to saving X engineer-weeks per quarter" would make the piece significantly more compelling.

**Verdict: Agree. MAJOR is right. Valid across genres.**

---

### Finding Audit Summary

| # | Finding | Agent Severity | My Severity (Blog) | Agree? |
|---|---|---|---|---|
| A1 | Causal claim without methodology | CRITICAL (-20) | MAJOR (-10) | Finding: Yes, Severity: No |
| A2 | LLM accuracy unstated | CRITICAL (-20) | MAJOR (-10) | Finding: Yes, Severity: No |
| A3 | No limitations | MAJOR (-10) | MINOR (-5) | Partial — missed existing limitation |
| A4 | Detection-to-impact leap | MAJOR (-10) | MAJOR (-10) | Yes |
| A5 | No failure mode analysis | MAJOR (-8) | Would not flag | No — genre-inappropriate |
| A6 | No baseline for LLM claims | MAJOR (-10) | MAJOR (-10) | Yes |
| A7 | "Double digits" vague | MAJOR (-10) | MAJOR (-10) | Strongly yes |
| C1 | Missing TL;DR | CRITICAL (-15) | MAJOR (-10) | Finding: Yes, Severity: No |
| C2 | No story arc | CRITICAL (-12) | Would not flag | No — blog has a valid arc |
| C3 | Generic headings | MINOR (-3) | MINOR (-3) | Yes |
| C4 | Terminology section | MAJOR (-10) | MINOR (-5) | Finding: Yes, Severity: No |
| C5 | Limitations absent (comms) | MAJOR (-12) | Double-count of A3 | Cross-cutting overlap |
| C6 | Playbook not actionable | MAJOR (-8) | Would not flag | No — genre-inappropriate |
| C7 | No recommendations/owners | CRITICAL (-8) | Would not flag | No — genre-inappropriate + bug |
| C8 | Missing "so what" | MAJOR (-8) | MAJOR (-8) | Yes |

**Bottom line: Of 15 findings, I fully agree with 5, partially agree with 4, and disagree with 6.** The 6 disagreements are all genre-related — they're valid for internal analyses but wrong for a blog post. The agent is applying one rubric to all genres, and that's the fundamental problem at the finding level.

---

## Part 2: What's Missing from a DS Practice Lens

These are the things I'd flag in a real review that the agent completely missed.

### 1. The Agent Never Asks "What Is This Document Trying to Do?"

This is the single biggest gap. Before I review anything from my team, I ask: "What's the purpose? Who's the audience? What decision does this support?" The answers determine my review lens entirely.

- A blog post? I'm reviewing for clarity, credibility, and engagement.
- An internal analysis for an exec? I'm reviewing for decision-readiness.
- A notebook for the DS team? I'm reviewing for reproducibility and rigor.

The agent applies the same rubric to all three. That's like grading a tweet, a press release, and a research paper with the same rubric. The findings might overlap, but the severity and relevance change completely.

This is distinct from genre awareness. It's purpose awareness. A blog post on Medium has a fundamentally different purpose than a Confluence page titled "Q4 Churn Analysis — Decision Needed by Friday."

### 2. The Agent Doesn't Evaluate Whether the Analysis Succeeded at Its Purpose

The Meta blog got 109 likes on Medium and substantive positive comments. By the measure that matters for a blog (audience engagement), it succeeded. The agent evaluates it against "would an exec make a good decision from this?" when the real question is "does this teach the reader something useful about using LLMs for bug report analysis?"

In my practice, I evaluate analyses against their stated purpose:
- Did the blog teach the methodology clearly? **Mostly yes.**
- Did the blog provide enough evidence to be credible? **Weakly — the vague metrics hurt.**
- Did the blog inspire adoption? **Yes — the playbook section is good for this.**
- Did the blog overstate its claims? **Yes — the causal language is a real problem.**

### 3. The Agent Under-Weights the Honest Difficulty Acknowledgment

The blog says: "achieving reliable and actionable results requires upfront investment in tuning and iterations." In the sea of "AI solves everything" corporate blog posts, this honesty is rare and valuable. It's a strength the agent completely missed.

When I review blog posts from my team, one of the first things I look for is whether they've been honest about what was hard. Readers — especially technical readers — trust authors who acknowledge difficulty. This sentence is doing important credibility work.

### 4. The Agent Misses the Strength of the Outage Example

The outage detection case study is the best evidence in the entire piece. It's a concrete, real-world demonstration: during an actual incident, the LLM system detected specific complaint patterns ("Feed Not Loading," "Can't post"). The agent mentions this as a positive but underweights it.

In a real review, I'd say: "This outage example should be your opening paragraph. Start with the story, then zoom out to the methodology. The reader will remember the outage long after they forget the playbook steps."

### 5. No Feedback on the Strongest Improvement Opportunity

The biggest improvement this blog could make — and something I always coach my team on — is to show one complete evidence chain. Pick the best "less visible bug" example and trace it: LLM classified the report → analyst noticed the pattern → engineer found the root cause → fix deployed → bug reports in that category dropped by X%.

The agent identifies the gap ("no specific LLM-surfaced insight is traced to a specific fix to a measured outcome") but frames it purely as a criticism. A good DS review would frame it as: "You're 80% of the way there. Add one detailed case study and this piece becomes significantly more credible."

### 6. The Review Is Pure Criticism — No Coaching

When I review my team's work, roughly 40% of my feedback is "here's what to change" and 60% is "here's how to think about it differently next time." The agent is 100% criticism. There's no coaching — no "in the future, when you make causal claims, here's the test to run on your own draft." A review that only tells you what's wrong teaches you nothing about how to be better.

---

## Part 3: Scoring Calibration — My Scores

### Meta Blog Post: My Score — 52/100, Minor Fix

**How I'd break it down:**

**Analysis dimension: 45/100**

Real, legitimate gaps:
- Causal claim without evidence: -15 (MAJOR — serious but not CRITICAL for a blog)
- "Double digits" vagueness: -10 (MAJOR — this hurts credibility regardless of genre)
- No validation evidence for LLM accuracy: -10 (MAJOR — even a blog should give some evidence)
- No baseline comparison to prior system: -10 (MAJOR — "it's better" without data is weak)
- Detection-to-impact logic gap: -10 (MAJOR — the evidence chain has a real hole)

Total deductions: -55, but I'd give partial credit for the honest difficulty acknowledgment and the concrete outage example: roughly +5 to +8.

**Effective analysis score: ~45-50**

**Communication dimension: 55-60/100**

Real gaps:
- Opens with philosophy instead of the key result: -10 (MAJOR — could lead with impact)
- "So what?" missing for the headline metric: -8 (MAJOR — needs business translation)
- Terminology section is unnecessary: -5 (MINOR — not harmful, just wasted space)
- Generic headings: -3 (MINOR — standard blog headings but could be better)

Strengths that offset some deductions:
- Clear problem-solution-evidence-playbook structure
- Concrete outage example that works for any audience
- Honest about implementation difficulty
- End-to-end pipeline thinking signals operational maturity

Total deductions: -26 raw, with structural strengths providing some offset.

**Effective communication score: ~55-60**

**Final: (45 + 58) / 2 ≈ 52/100 — Minor Fix**

This feels right. It says: "Solid blog post with real credibility gaps in its core claims. Fix the vague metrics and reframe the causal language, and this is a strong piece." That's the review I'd give my team member.

### Vanguard A/B Test: My Score — 48/100, Minor Fix

**How I'd break it down:**

**Analysis dimension: 42/100**

The Vanguard doc does several things right that the scoring completely ignores:
- Pre-specified hypotheses with named KPIs: +8
- Real experimental design with test/control: +8
- Pre-specified 5% practical significance threshold: +5
- Covariate balance check across 7 variables: +5
- Reports actual quantitative results: +3

These are non-trivial analytical strengths. Total credit: +29 (before any cap).

Real gaps:
- No statistical tests (p-values, CIs): -15 (CRITICAL-INCOMPLETE — the experiment exists, the data exists, the stats just aren't reported. This is fundamentally different from never running an experiment.)
- H2 contradiction unacknowledged: -10 (MAJOR — test group is slower but this is never addressed)
- H3 direction never clearly stated: -8 (MAJOR — up or down? The doc is ambiguous)
- No sample sizes reported: -8 (MAJOR — essential for any experiment)
- Completion rate definition ambiguous: -5 (MINOR — the concept is clear even if the precise denominator isn't)
- No subgroup analysis: -3 (MINOR — nice to have, not essential for the primary question)
- Balance check not rigorous enough: -5 (MINOR — a check WAS done; calling it CRITICAL ignores the effort)

Total deductions: -54. Analytical credits: +29 (capped at +25 per the proposed system). Net: -29.

**Effective analysis score: ~42-48** (100 - 54 + 25 = 71 is too generous; with the strength of the gap in statistical reporting, I'd land around 42-48)

Wait — let me recalibrate. The missing statistical tests IS the big one. An A/B test without stats is like a recipe without cooking times. The experiment is set up right, but the critical output is missing. That should be -15 minimum but not the -20 CRITICAL the system assigned, because the work to fix it is an afternoon of running t-tests and z-tests on data that already exists.

**Revised analysis: ~42/100** — reflecting "strong experimental bones, significant reporting gaps."

**Communication dimension: 45-52/100**

Strengths:
- Hypothesis-driven structure provides a clear narrative thread
- Results are organized by hypothesis (logical for a tech audience)
- Includes practical significance threshold upfront

Real gaps:
- No TL;DR — answer buried at ~60% through: -10 (MAJOR — for a reactive tech doc, lead with the answer)
- Limitations section completely absent: -8 (MAJOR — tech audience needs scope boundaries)
- Vague recommendations in conclusion: -8 (MAJOR — "users might need some time" is not a recommendation)
- Formatting issues (typos, broken links, phantom figure refs): -5 (MINOR)
- Generic headings: -3 (MINOR)

What I would NOT flag:
- "No clear story arc" — DISAGREE. The document follows Hypotheses → Design Check → Results by Hypothesis → Conclusion. That IS a story arc. It's a hypothesis-driven structure, which is appropriate for a technical audience evaluating an experiment. Calling this "no story arc" because it doesn't follow "answer-first" reactive format is applying the wrong template.
- "Insufficient technical rigor for tech audience" — This is a CROSS-CUTTING DOUBLE of the analysis finding about missing stats. The agent flagged it in both dimensions.
- "Over-interpretation boundary unclear" — Also derivative of the missing stats finding.

Total deductions: -34. Strengths offset: roughly +5 for the hypothesis-driven structure.

**Effective communication score: ~48-52**

**Final: (42 + 50) / 2 ≈ 46/100 — Minor Fix (low end)**

This also feels right. It says: "Well-designed experiment with clear structure, but you need to add the statistical inference before this goes to stakeholders. An afternoon of work to fix." That's what I'd tell my DS.

### Score Comparison

| Document | Agent Score | My Score | Gap | My Verdict |
|---|---|---|---|---|
| Meta Blog | 18/100 | 52/100 | +34 | Minor Fix |
| Vanguard A/B | 16/100 | 46/100 | +30 | Minor Fix |
| **Vanguard - Meta gap** | **-2** (Vanguard lower!) | **-6** (Vanguard slightly lower) | — | — |

Interesting: my gut puts Vanguard slightly BELOW Meta overall, even though Vanguard has much stronger analytical foundations. Why? Because Meta is better at communication — it's a polished blog post written by experienced communicators. Vanguard is a course project with formatting issues and buried results.

BUT — and this is critical — on the analysis dimension, Vanguard should score meaningfully higher than Meta. My analysis scores: Vanguard 42, Meta 45... actually that's close. Let me re-examine.

On reflection, Meta's analysis score should be lower than Vanguard's because Meta never did the analysis at all. Meta claimed results without methodology. Vanguard ran the experiment but didn't report the stats. Those are fundamentally different levels of analytical work.

**Revised calibration:**
- Meta analysis: 35-40 (claimed results without doing the work)
- Vanguard analysis: 42-48 (did the work, didn't report it fully)
- Meta communication: 55-60 (polished blog writing)
- Vanguard communication: 40-48 (rough but structured)

| Document | Analysis | Communication | Final | Verdict |
|---|---|---|---|---|
| Meta Blog | ~38 | ~58 | ~48 | Minor Fix (low) |
| Vanguard A/B | ~45 | ~45 | ~45 | Minor Fix (low) |

This is closer to right. Meta gets carried by communication quality; Vanguard gets carried by analytical quality. Both land in Minor Fix, but for different reasons. The gap between them is small (~3 points) which feels correct — they're in the same tier of "has real value, needs specific fixes before it's stakeholder-ready."

The calibration notes suggest Meta should be 20-35 and Vanguard 40-55. I think Meta at 20-35 is too harsh — it's a published, well-received blog post. 45-55 is more reasonable. Vanguard at 40-55 I agree with.

---

## Part 4: Reactions to the Two Existing Assessments

### Principal AI Engineer Assessment

**Where I agree:**
- The root cause identification is correct. 7 findings from 3 root causes getting -88 deductions is indeed the math problem. The deduction capacity (120) exceeding the budget (100) is a clean diagnosis.
- The diminishing returns formula is mechanically sound and easy to implement.
- "Don't add --format yet" is the right call.

**Where I disagree:**
- **Diminishing returns alone doesn't solve the differentiation problem.** The engineer's projected scores: Meta ~52, Rossmann ~55. But what about Vanguard? Vanguard also had -88 analysis deductions, so the diminishing returns curve would give it roughly the same score as Meta (~50). That's the same differentiation failure — a real experiment and a blog post with no methodology score identically, just at a higher baseline.
- **The engineer never questions whether individual findings are correct.** The entire analysis assumes the findings are right and only the scoring math is wrong. But as I showed in Part 1, several findings are genre-inappropriate. Fixing the math without fixing the finding quality means the system produces reasonable scores for the wrong reasons.
- **Genre is dismissed too quickly.** "Even with perfect genre detection, any real internal Confluence page with 10+ legitimate findings would also score in the 20s" is true — but the deeper issue is that a blog post shouldn't HAVE 15 findings in the first place. Half of them are genre-inappropriate. Genre awareness doesn't just change deduction weights; it changes which findings are generated.
- **The CRITICAL tightening is good but incomplete.** Moving "Missing TL;DR" and "No story arc" from CRITICAL to MAJOR is correct, but the engineer doesn't address the other over-assigned CRITICALs (e.g., "No specific recommendations with owners" for a blog post).

**What's missing from a DS perspective:**
The engineer treats this as a pure systems problem — inputs (findings) and outputs (scores). The DS perspective is: are we measuring the right thing? The findings are the measurement instrument. If the instrument is miscalibrated (measuring genre-inappropriate attributes), no amount of downstream math will fix it.

### PM Lead Review

**Where I agree:**
- **Critique 2 (CRITICAL over-assignment) is the most important insight across all reviews.** The table showing which CRITICALs are genuinely misleading vs. structural preferences is exactly right. "Missing TL;DR" is not in the same severity class as "causal claim without methodology." This is the #1 thing to fix.
- **Critique 3 (no strength credits) identifies the differentiation root cause.** Vanguard does 6+ things right that Meta doesn't, and all of them are worth 0 points. This IS the primary driver of the differentiation failure.
- **Critique 5 (teardown-style output) is sharp.** The review is nearly 2x the length of the document being reviewed. "A communication review tool with a communication problem" — that's the right framing.
- **Fix 4 (output restructure — strengths before findings)** is exactly right from a coaching perspective.

**Where I disagree:**
- **Critique 1's framing of "structurally broken" is too strong.** The math isn't broken in the sense that the formula is wrong — it's working as designed. The problem is that the design assumed 3-5 findings per document (where additive deductions work fine) but the agent consistently generates 15 findings. The fix should target finding volume AND deduction math, not just deduction math.
- **Fix 3 (strength credits at +3 to +5, capped at +15) is too modest.** +15 per dimension would give Vanguard roughly 12 extra points. That helps but doesn't close the gap. The calibration notes' proposed credit system (+5 to +8 per strength, capped at +25) is closer to right. In my scoring above, Vanguard's analytical strengths are worth +25-29 points. You need meaningful credit to differentiate "did the work" from "didn't do the work."
- **The PM Lead's expected scores still cluster.** Projected: Meta 50, Rossmann 54, Vanguard 50. That's 4-point spread across three very different documents. We need 15-25 point spread between tiers.

**What's missing from a DS perspective:**
- **Neither assessment addresses the finding quality.** Both assume the 15 findings are correct and focus on how findings map to scores. But as I've shown, 6 of the 15 Meta findings are genre-inappropriate. No amount of score formula changes will fix "the agent flags things that shouldn't be flagged for this document type."
- **Neither addresses finding caps.** The agent reports up to 3 findings per lens × 4 lenses × 2 dimensions = 24 possible findings. That's too many for a useful review. In practice, I give my team 3-5 things to fix. The most important ones. Not every possible criticism I could generate.
- **Neither considers the interaction between audience/workflow parameters and document genre.** The Meta blog was reviewed as `exec/proactive` — the harshest combination. But a blog post on Medium is NEVER exec/proactive in practice. Either the user made a parameter mistake, or the system should warn: "This appears to be a public blog post. The exec/proactive parameters may not be appropriate."

---

## Part 5: My Proposed Fixes

Prioritized by "what would make this tool actually useful for my DS team."

### Fix 1 (P0): Strength Credits — The Differentiation Fix

**This is the single most important change.** Without it, "did the work but has gaps" and "never did the work" score identically.

I'd go further than either existing proposal:

| Analytical Strength | Credit |
|---|---|
| Pre-specified, testable hypotheses with KPIs | +5 |
| Real experimental/quasi-experimental design | +8 |
| Pre-specified practical significance threshold | +5 |
| Validation against ground truth (human labels, holdout set) | +5 |
| Reports specific quantitative results (not vague claims) | +3 |
| Covariate balance / confound check performed | +5 |
| Limitations acknowledged | +3 |
| Sensitivity / robustness analysis included | +5 |

| Communication Strength | Credit |
|---|---|
| Clear TL;DR or executive summary present | +5 |
| Argument follows logical arc appropriate for audience | +5 |
| Visualizations directly support narrative | +3 |
| Recommendations are specific and actionable | +5 |
| Appropriate detail level for stated audience | +3 |

**Cap: +25 per dimension.** This is significant enough to create real differentiation. Vanguard would get +20-25 on analysis; Meta would get +3-5. That's a 15-22 point gap from strengths alone.

**Why this matters more than diminishing returns:** Diminishing returns compresses the low end of the scale (prevents catastrophic scores). Strength credits expand the middle (creates differentiation). Both are needed, but differentiation is the more important user outcome.

### Fix 2 (P0): Reclassify CRITICALs — Agree with PM Lead

I agree completely with the PM Lead's reclassification table. Move these from CRITICAL to MAJOR:
- Missing TL;DR: CRITICAL (-15) → MAJOR (-10)
- No clear story arc: CRITICAL (-12) → MAJOR (-8)
- Limitations/scope absent: CRITICAL (-12) → MAJOR (-10)

**Additional reclassification I'd add:**
- "No specific recommendations with owners": should NEVER be CRITICAL. It's currently hitting CRITICAL via the "Vague recommendation or answer" pathway with the proactive workflow multiplier. Even in a real internal analysis, vague recommendations are MAJOR, not CRITICAL. CRITICAL should be reserved for "the reader will reach a wrong conclusion," not "the reader won't know what to do next."

**Proposed CRITICAL standard (agree with Principal AI Engineer):** CRITICAL = the reader would reach a wrong conclusion or make a harmful decision based on what's written. Structural, formatting, and completeness gaps max out at MAJOR.

### Fix 3 (P0): Diminishing Returns — Agree with Both Assessments

I agree with the diminishing returns formula as a safety net:
- 0-30 deductions: 100%
- 31-50 deductions: 50%
- 51+ deductions: 25%

But I want to be clear: **this is the safety net, not the primary fix.** Diminishing returns prevents catastrophic scores but doesn't create differentiation. Strength credits create differentiation. You need both.

### Fix 4 (P1): Reduce Finding Volume

Neither assessment proposes this, but I think it's important. **The agent should report fewer, more impactful findings.**

Current cap: 3 findings per lens × 4 lenses = 12 per dimension, 24 total.
My proposed cap: **8 total findings max** (across both dimensions). Top 5 by severity/impact, plus up to 3 MINOR polish items.

Why: When I review a team member's analysis, I give them 3-5 things to fix. Not 15. If I find 15 issues, I prioritize the 5 that matter most and save the rest for a follow-up conversation. A 15-finding review is not more helpful than a 5-finding review — it's overwhelming and demoralizing.

The practical benefit: fewer findings = fewer deductions = the scoring math problem partially resolves itself. A document with 8 findings at ~-10 average = -80 raw deductions. With diminishing returns, that's about -50 effective. That's a reasonable range.

**Implementation:** The lead agent synthesis (Step 9) already selects Top 3 priority fixes. Extend this concept: the per-lens detail sections should only include findings that are independently actionable and not subsumed by a higher-priority finding. If "no statistical tests" is the #1 finding, don't also separately flag "no confidence intervals" and "no p-values" — those are components of the same fix.

### Fix 5 (P1): Output Restructure — Agree with PM Lead's Fix 4

Move "What You Did Well" before findings. Compress per-lens detail. Add emoji indicators to dashboard. Use blockquote format for suggested rewrites.

I'd add one thing: **a one-sentence "Review Summary" right after the score.** Something like: "Well-structured experiment with strong analytical foundations — primary gap is missing statistical inference." This gives the reader the narrative arc of the review in one line before they dive into details.

### Fix 6 (P2): Purpose/Genre Awareness — Defer but Design

I agree with both assessments that `--format` should be deferred. But I disagree that it's unnecessary.

Diminishing returns + strength credits + CRITICAL reclassification will fix the SCORES. But they won't fix the FINDINGS. The agent will still flag "no failure mode analysis" on a blog post and "no named owners for recommendations" on a course project. Those findings are noise that erodes trust.

**For v0.5:** Add lightweight genre detection to the lead agent's pre-processing. Not a parameter — auto-detection. Signals:
- Medium/blog URL → blog genre
- `.ipynb` or Kaggle reference → notebook genre
- Confluence URL → internal analysis genre
- Contains "course project" or "assignment" → academic genre

Genre affects which findings are generated (suppress genre-inappropriate findings), not just how they're scored.

### What I Would NOT Do

1. **Don't rescale all deduction values.** Agree with both assessments.
2. **Don't add per-dimension hard caps.** Diminishing returns makes this unnecessary.
3. **Don't implement cluster-based scoring yet.** Too complex for v0.4. Revisit in v1.5.
4. **Don't add CRITICAL-INCOMPLETE / CRITICAL-ABSENT gradation yet.** The calibration notes propose this. It's interesting but adds complexity. Start with reclassifying structural CRITICALs to MAJOR and see if that's sufficient. If the remaining CRITICALs still need gradation after testing, add it in v0.5.

---

## Summary: Where All Three Assessments Align and Diverge

| Topic | Principal AI Engineer | PM Lead | DS Lead (Me) |
|---|---|---|---|
| Root cause | Deduction stacking | Broken scoring math + CRITICAL over-assignment + no strength credits | All of the above + genre-inappropriate findings + too many findings |
| Primary fix | Diminishing returns formula | Diminishing returns + CRITICAL reclassify + strength credits | Strength credits + CRITICAL reclassify + diminishing returns (different priority order) |
| Strength credits | Not proposed | +3 to +5, cap +15/dimension | +3 to +8, cap +25/dimension |
| Finding quality | Not addressed | Not addressed | 6 of 15 Meta findings are genre-inappropriate |
| Finding volume | Not addressed | Mentioned (output too long) | Cap at 8 total findings |
| Genre awareness | Defer | Defer | Defer scoring, but auto-detect for finding suppression in v0.5 |
| CRITICAL reclassification | TL;DR + story arc → MAJOR | TL;DR + story arc + limitations → MAJOR | All of those + recommendations/owners → MAJOR |
| Output restructure | Not addressed | Strengths before findings, compress detail | Agree + add one-sentence review summary |

**The key insight the other assessments miss:** The scoring problem has two layers. Layer 1 is the math (how findings map to scores) — both assessments address this well. Layer 2 is the measurement (which findings are generated) — neither assessment addresses this. You can fix the math perfectly and still get a review that flags 15 things on a blog post, 6 of which are irrelevant. The user will read those 6 irrelevant findings and lose trust in the tool, regardless of what score it produces.

---

## Appendix: Specific Bugs Found During Audit

1. **Meta review Finding C7 (Actionability):** Severity labeled CRITICAL in finding body but deduction logged as -8, which matches MAJOR in the deduction table. Severity and deduction are inconsistent.

2. **Meta review Finding C5 (Audience Fit):** Severity labeled MAJOR but deduction logged as -12, which matches CRITICAL in the deduction table for "Limitations/scope unclear for downstream." Either the severity is wrong or the deduction amount is wrong.

3. **Cross-cutting double-counting:** "No limitations" appears as Analysis Finding A3 (-10) AND Communication Finding C5 (-12) = -22 total for one root cause. The routing table (SKILL.md Section 5) assigns "Missing limitation" to communication-reviewer. Analysis-reviewer should not have independently flagged it.
