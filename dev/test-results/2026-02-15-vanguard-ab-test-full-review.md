# Test Run: Vanguard A/B Test — Full Review (2026-02-15)

## Test Parameters

- **Fixture:** `dev/test-fixtures/real/vanguard-ab-test.md`
- **Mode:** Full
- **Audience:** tech
- **Workflow:** reactive
- **Model:** claude-opus-4-6
- **Pipeline:** Full 10-step (ds-review-lead orchestrator with parallel subagent dispatch via Task tool)
- **Tier:** 1 (1,125 words, ~5 min read)

## Result

**Score: 16/100 — Major Rework**
Floor rule applied: 5 CRITICAL findings (2 Analysis + 3 Communication) → 2+ CRITICAL → Major Rework (max 59). Computed score of 16 already within band.

---

# DS Analysis Review: Vanguard Project

**Score: 16/100 — Major Rework**
*Floor rule applied: 5 CRITICAL findings across both dimensions (2+ CRITICAL → verdict capped at Major Rework, max 59). Computed score of 16 already falls within Major Rework band.*

Mode: Full | Audience: Technical Lead | Workflow: Reactive | Tier 1 | 1,125 words | ~5 min read

### Lens Dashboard

| Dimension | Lens | Rating |
|---|---|---|
| Analysis | Methodology & Assumptions | CRITICAL |
| Analysis | Logic & Traceability | MAJOR ISSUES |
| Analysis | Completeness & Source Fidelity | MAJOR ISSUES |
| Analysis | Metrics | MAJOR ISSUES |
| Communication | Structure & TL;DR | CRITICAL |
| Communication | Audience Fit | CRITICAL |
| Communication | Conciseness & Prioritization | MAJOR ISSUES |
| Communication | Actionability | MAJOR ISSUES |

---

## Top 3 Priority Fixes

**1. No statistical testing reported for the A/B experiment (CRITICAL — Analysis)**
- **Location:** Hypothesis 1 results — *"The result of the analysis shows that the completion rate of the test group is approximately 10% higher than the control group."*
- **Issue:** This is an A/B experiment, yet the entire document contains zero hypothesis tests, no p-values, no confidence intervals, and no measure of statistical significance for any KPI. The headline 10% completion rate lift is reported as a bare number. For a technical audience evaluating an experiment, the absence of statistical inference means the reader cannot distinguish a real effect from random noise. This is the single most fundamental gap — it undermines the analytical validity *and* makes the communication non-actionable for a tech reader (cross-cutting impact: without stats, the communication cannot provide the confidence intervals and decision context that the Actionability lens requires in reactive workflow).
- **Suggested fix:** For each hypothesis, report the statistical test used (e.g., two-proportion z-test for completion rate, two-sample t-test for time), the p-value, a 95% confidence interval for the treatment effect, and whether the result meets a pre-specified alpha level.

**2. Causal claims rest on unverified group comparability (CRITICAL — Analysis)**
- **Location:** Experiment Evaluation, Design Effectiveness — *"the maximum bias is 4% which appears in the gender category. In general, the bias is acceptable for the experiment."*
- **Issue:** The analysis makes causal claims throughout ("The new feature would encourage more clients...", "The reduction of time spent on 'step_1' led to a significant increase..."), but group comparability is only loosely assessed. "Bias" is undefined (absolute difference? standardized mean difference?), the 4% threshold has no justification, and no formal balance check (e.g., SMD < 0.1) is presented. Without establishing comparability rigorously, every causal claim in the analysis is built on an unverified assumption. This compounds with the missing limitations section (cross-cutting: the communication dimension flagged that zero limitations are stated anywhere, meaning downstream readers have no guardrails against over-interpreting these causal claims).
- **Suggested fix:** Define the balance metric used (e.g., standardized mean difference), report it for each covariate in a balance table, state the acceptability threshold with a citation (e.g., SMD < 0.1 per Imbens & Rubin), and frame causal claims as conditional on verified balance.

**3. Missing TL;DR — no direct answer to the experiment question upfront (CRITICAL — Communication)**
- **Location:** Document opening — the reader must traverse Project Overview, Data, Metadata (14 field definitions), and Experiment Evaluation before encountering any result.
- **Issue:** In a reactive workflow, the TL;DR should lead with the direct answer to the question asked: *"Would these changes encourage more clients to complete the process?"* The answer does not appear until approximately 60% through the document. The opening is background and methodology setup ("The digital world is evolving, and so are Vanguard's clients..."). A technical reader consuming a reactive experiment report expects to see the answer immediately, with evidence below. Instead, they get marketing copy and metadata before any findings.
- **Suggested fix:** Add a TL;DR immediately after the title: *"The new UI increased completion rate by ~10% (exceeding the 5% cost-justification threshold), improved visitor retention by 9.1pp, but introduced temporarily elevated error rates during user adaptation. Statistical evidence and methodology below."* Then restructure to put results before detailed methodology.

---

## What You Did Well

1. **Hypothesis-driven analytical framework (Analysis):** The analysis is organized around three pre-specified, testable hypotheses, each with a named KPI (completion rate, time per step, error rate). This is a strong analytical structure — it makes the intent of each section unambiguous and creates a testable framework. Many analyses lack this discipline.

2. **Practical significance threshold pre-specified (Analysis):** The 5% minimum completion rate increase is stated *before* results are presented, preventing post-hoc rationalization. This is good experimental practice — the bar for success was defined upfront, not reverse-engineered to match the outcome.

3. **Experiment validity assessment included (Communication):** The document addresses group balance and experiment duration before presenting results. This is the right instinct for a technical audience: establish that the experiment is trustworthy before presenting what it found.

---

## Analysis Dimension (Score: 12/100)

### Methodology & Assumptions — CRITICAL

1. **No statistical testing reported for primary hypothesis**
   - Severity: CRITICAL | Location: Hypothesis 1 results
   - The A/B experiment reports no hypothesis test, p-value, confidence interval, or significance measure for any KPI. The 10% lift is a bare number — indistinguishable from noise without inferential statistics.
   - *Fix: Report test, p-value, 95% CI, and pre-specified alpha for each hypothesis.*

2. **Unstated assumption of group comparability despite causal claims**
   - Severity: CRITICAL | Location: Experiment Evaluation (Design Effectiveness) and Conclusion
   - "Bias" is undefined, the 4% threshold is unjustified, and no formal balance check is presented. Causal claims throughout rest on this unverified foundation.
   - *Fix: Define balance metric, report per-covariate, justify threshold, condition causal claims on verified balance.*

### Logic & Traceability — MAJOR ISSUES

3. **Contradictory evidence for Hypothesis 2 presented without resolution**
   - Severity: MAJOR | Location: Hypothesis 2 — *"The total time spent for control and test group is 309.68 s and 315.32 s, respectively."*
   - The hypothesis predicts time reduction; the data shows the test group is *slower*. The analysis never acknowledges this and pivots to retention — a different metric — without connecting the reasoning.
   - *Fix: Explicitly state H2 is unsupported for time-per-step. Frame retention as a separate emergent finding.*

4. **Error rate hypothesis unsupported — direction of effect never stated**
   - Severity: MAJOR | Location: Hypothesis 3 — *"significant deviation in error rates"*
   - The document never states whether errors went up or down for the test group. The Conclusion implies errors increased ("they will make more errors") but H3 is never formally rejected.
   - *Fix: Report error rates for both groups with CIs. Explicitly state H3 is rejected. Quantify the adaptation period.*

### Completeness & Source Fidelity — MAJOR ISSUES

5. **No segmentation by demographics despite rich data**
   - Severity: MAJOR | Location: Data section lists demographics used only in balance check
   - The dataset includes age, tenure, balance, accounts, calls, and logons — yet no heterogeneous treatment effect analysis is performed. Does the new UI help younger users more? Do high-balance clients respond differently?
   - *Fix: Add subgroup analysis or acknowledge as a limitation/future work item.*

### Metrics — MAJOR ISSUES

6. **No sample sizes or power analysis reported**
   - Severity: MAJOR | Location: Throughout — no N appears anywhere
   - The reader cannot assess reliability of any metric without knowing how many clients/visits were in each group or whether the experiment was powered for the 5% threshold.
   - *Fix: Report N for test and control (clients and visits). Include power analysis.*

7. **Completion rate metric definition is ambiguous**
   - Severity: MAJOR | Location: Hypothesis 1 — *"The completion rate indicates the number of visits which reach 'confirm' step."*
   - A rate requires a denominator. Is it visits reaching confirm / all visits? / all visitors? / visitors reaching step 1? The aggregation method (daily average vs. total) is also unspecified.
   - *Fix: Define precisely: numerator, denominator, and aggregation method.*

**Analysis Deduction Log:**

| Issue | Deduction |
|---|---|
| Flawed statistical methodology (no tests) | -20 |
| Unstated critical assumption (group comparability) | -20 |
| Unsupported logical leap (H2 contradiction) | -10 |
| Unsupported logical leap (H3 direction unstated) | -10 |
| Missing obvious analysis (no segmentation) | -8 |
| Missing baseline/benchmark (no sample sizes) | -10 |
| Missing baseline/benchmark (metric definition ambiguous) | -10 |
| **Total** | **-88** |

---

## Communication Dimension (Score: 19/100)

### Structure & TL;DR — CRITICAL

1. **Missing TL;DR — no direct answer upfront**
   - Severity: CRITICAL | Location: Document opening (Project Overview)
   - Opens with company background and experiment setup. The answer to the question asked doesn't appear until ~60% through the document. In reactive workflow for a tech audience, lead with the direct answer.
   - *Fix: Add TL;DR after title with key finding + evidence summary + technical caveats.*

2. **No clear story arc — document reads as a methodology log**
   - Severity: CRITICAL | Location: Overall structure (Overview → Data → Metadata → Evaluation → Results → Conclusion)
   - Follows chronological "how we did it" order. The 14-field Metadata section sits between context and results. A tech reader expects deductive structure: evidence → conclusion.
   - *Fix: Restructure to TL;DR → Brief design → Results by hypothesis → Limitations → Recommendations. Move Metadata to appendix.*

3. **Generic headings do not telegraph findings**
   - Severity: MINOR | Location: All major section headings
   - "Experiment Evaluation," "Data," "Conclusion" are labels, not signposts. A reader skimming headings learns nothing about findings.
   - *Fix: Rewrite headings to state the takeaway (e.g., "New UI Drives 10% Completion Lift").*

### Audience Fit — CRITICAL

4. **Limitations and scope boundaries completely absent**
   - Severity: CRITICAL | Location: Entire document — no limitations section exists
   - Zero limitations, caveats, or scope boundaries stated. A tech reader forwarding this analysis downstream has no guardrails against over-interpretation. Key gaps: sensitivity to 55-day window, tax-season confound impact, segment robustness, novelty effects.
   - *Fix: Add "Limitations & Scope" section covering window sensitivity, confounders, generalizability, and unanswered questions.*

5. **Insufficient technical rigor for tech audience**
   - Severity: MAJOR | Location: All three hypothesis result sections
   - No confidence intervals, p-values, sample sizes, test descriptions, or effect sizes reported. "Approximately 10% higher" and "significant deviation" are bare assertions — not sufficient for a technical lead to evaluate.
   - *Fix: For each hypothesis: sample sizes, test used, p-value/CI, effect size.*

### Conciseness & Prioritization — MAJOR ISSUES

6. **Data without narrative context — Hypothesis 2 metrics contradictory and unexplained**
   - Severity: MAJOR | Location: Hypothesis 2 section
   - Time data contradicts the hypothesis (test group slower: 315.32s vs 309.68s) but this is never acknowledged. Pivot to retention metrics lacks transition. "The figure below" references point to non-existent figures.
   - *Fix: State H2 unsupported for time. Introduce retention as separate finding. Remove phantom figure references.*

7. **Sloppy formatting and inconsistent polish**
   - Severity: MINOR | Location: Throughout
   - "categeory" typo, inconsistent capitalization, phantom figure references, broken PowerBI link, marketing opening sentence.
   - *Fix: Spelling/grammar pass. Remove/embed figure references. Remove marketing language.*

### Actionability — MAJOR ISSUES

8. **Vague recommendations — no specific actions, owners, or timelines**
   - Severity: MAJOR | Location: Conclusion section
   - Conclusion offers observations, not recommendations. "The users might need some time..." is descriptive. The most critical question — should the new UI be rolled out? — is never explicitly answered.
   - *Fix: Add "Recommendations" section: (1) Roll out new UI, (2) Investigate step_1/step_2 UX, (3) Monitor error rates post-rollout.*

9. **Over-interpretation boundary unclear**
   - Severity: MAJOR | Location: Hypothesis 1 results and Conclusion
   - The 10% lift is presented as definitive with no uncertainty, no segment consistency check, no novelty-effect caveat, and no causal-language qualification.
   - *Fix: Add boundary statements: "This supports X but cannot confirm Y." Provide CIs. Note what remains unanswered.*

**Communication Deduction Log:**

| Issue | Deduction |
|---|---|
| Missing or ineffective TL;DR | -15 |
| No clear story arc or structure | -12 |
| Limitations/scope unclear for downstream | -12 |
| Audience mismatch (insufficient rigor for tech) | -10 |
| Data without narrative context (H2 section) | -8 |
| Vague recommendation or answer | -8 |
| Over-interpretation boundary unclear | -8 |
| Generic/non-actionable headings | -3 |
| Sloppy formatting / inconsistent polish | -5 |
| **Total** | **-81** |

---

## Score Computation

| Component | Score |
|---|---|
| Analysis Dimension | 12/100 |
| Communication Dimension | 19/100 |
| **Final Score** | **(12 + 19) / 2 = 16/100** |
| Floor Rule | 5 CRITICALs → Major Rework (max 59). Score already in band. |
| **Verdict** | **Major Rework** |

---

## Pipeline Observations

- **Subagent dispatch:** Both subagents (analysis-reviewer, communication-reviewer) dispatched in parallel via Task tool. Both returned successfully with properly formatted output.
- **Output format compliance:** Both subagents produced PER-LENS RATINGS, FINDINGS, POSITIVE FINDINGS, DEDUCTION LOG, and SUBAGENT SCORE as specified.
- **Deduction table adherence:** All deductions match SKILL.md Section 2 values exactly.
- **Dimension boundary respect:** No duplicate findings across dimensions. Gray-zone items (e.g., missing statistical rigor) correctly routed — analysis-reviewer owned the methodology gap, communication-reviewer owned the audience-fit presentation gap.
- **Floor rules correctly applied:** 5 CRITICALs triggered the 2+ CRITICAL rule (max 59). Score of 16 already below cap.
- **Cross-cutting issues noted:** Missing stats impacts both analysis validity and communication actionability — noted in Top 3 narrative without creating duplicate findings.
