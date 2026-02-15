# Experiment Report: Workspace Templates Feature — Randomized Controlled Trial

**Author:** Priya Mehta, Data Science
**Date:** January 28, 2026
**Experiment ID:** EXP-2025-Q4-031
**Status:** Complete

---

## Experiment Configuration

### Hypothesis

H0: The introduction of workspace templates does not affect the 14-day feature adoption rate (defined as the proportion of new users who create at least one project using a template within 14 days of signup).

H1: Workspace templates increase the 14-day feature adoption rate relative to the control experience.

### Design

This was a between-subjects, parallel-group, superiority RCT conducted on Acme Platform's web application. Randomization was performed at the user level using a deterministic hash of user_id with salt `exp-031-v1`, ensuring consistent assignment across sessions. Allocation ratio was 1:1. Neither participants nor the product team were informed of assignment status during the experiment (double-blind was not feasible as the feature is visually distinct, but analyst-blinding was maintained through coded group labels until final analysis).

The treatment group received the workspace templates feature: a modal presented during the project creation flow offering 12 pre-configured project templates across 4 categories (Marketing, Engineering, Design, General). The control group received the existing blank project creation flow with no changes.

### Sample Size Calculation

The pre-experiment baseline adoption rate (creating a project within 14 days) was 34.2% based on the preceding 90-day average. We targeted a minimum detectable effect (MDE) of 3 percentage points (absolute), representing an 8.8% relative lift. Using a two-sided proportion test with alpha = 0.05 and power = 0.80:

n = (Z_{alpha/2} + Z_{beta})^2 * (p1(1-p1) + p2(1-p2)) / (p1 - p2)^2
n = (1.96 + 0.84)^2 * (0.342 * 0.658 + 0.372 * 0.628) / (0.03)^2
n ≈ 3,842 per group

We targeted 4,500 per group to account for approximately 15% attrition due to incomplete tracking and users who never reached the project creation flow.

### Experiment Duration

The experiment ran from November 1 to December 15, 2025 (45 days), with the final 14-day observation window closing on December 29, 2025. All users who signed up during the enrollment window and were assigned to a treatment group were included in the ITT population.

---

## Randomization Diagnostics

### Balance Check

Covariate balance between treatment and control groups was assessed using standardized mean differences (SMD). An SMD < 0.1 is considered acceptable.

| Covariate | Control (n=4,612) | Treatment (n=4,587) | SMD |
|-----------|-------------------|---------------------|-----|
| Plan tier: Free | 53.8% | 54.1% | 0.006 |
| Plan tier: Starter | 28.9% | 28.4% | 0.011 |
| Plan tier: Pro | 12.7% | 13.1% | 0.012 |
| Plan tier: Enterprise | 4.6% | 4.4% | 0.010 |
| Acquisition: Organic | 32.8% | 33.4% | 0.013 |
| Acquisition: Paid | 38.1% | 37.4% | 0.015 |
| Acquisition: Referral | 14.7% | 15.0% | 0.008 |
| Acquisition: Direct | 14.4% | 14.2% | 0.006 |
| Company size: 1-10 | 41.2% | 40.8% | 0.008 |
| Company size: 11-50 | 29.4% | 30.1% | 0.015 |
| Company size: 51-200 | 17.8% | 17.3% | 0.013 |
| Company size: 200+ | 11.6% | 11.8% | 0.006 |
| Region: North America | 44.1% | 43.7% | 0.008 |
| Region: Europe | 28.3% | 28.9% | 0.013 |
| Region: APAC | 18.4% | 18.2% | 0.005 |
| Region: Other | 9.2% | 9.2% | 0.000 |

All SMDs are below 0.02, indicating excellent covariate balance. The Imbens-Rubin omnibus test yields B = 0.031, R = 1.003 (acceptable thresholds: B < 0.25, R in [0.5, 2.0]).

### SRM Check

Sample Ratio Mismatch test: chi-squared = 0.068, df = 1, p = 0.795. No evidence of systematic assignment imbalance.

### SUTVA Assessment

We checked for interference effects by examining whether the treatment effect varies by the proportion of treated users within the same company (for multi-user accounts). The interaction coefficient is not significant (p = 0.41), suggesting minimal spillover between treated and control users within organizations.

---

## Results

### Primary Endpoint

| Group | n | Adopted (14d) | Adoption Rate | SE |
|-------|---|---------------|---------------|----|
| Control | 4,612 | 1,586 | 34.4% | 0.70% |
| Treatment | 4,587 | 1,808 | 39.4% | 0.72% |

Absolute difference: +5.0 pp [95% CI: 3.0 pp, 7.0 pp]
Relative lift: +14.5% [95% CI: 8.7%, 20.3%]

Two-proportion z-test: z = 4.97, p < 0.001.

The result exceeds our MDE of 3 pp. The 95% confidence interval excludes zero and the lower bound (3.0 pp) equals our MDE, indicating the effect is both statistically and practically significant.

### CUPED-Adjusted Estimate

Using pre-experiment engagement (sessions in the 7 days prior to signup, where available for returning visitors) as a CUPED covariate to reduce variance:

CUPED-adjusted difference: +5.1 pp [95% CI: 3.4 pp, 6.8 pp]

The variance reduction from CUPED is approximately 18%, narrowing the confidence interval. The point estimate is essentially unchanged, indicating the result is robust to this adjustment.

### Subgroup Analysis

Treatment effects by pre-specified subgroups:

| Subgroup | Control Rate | Treatment Rate | Difference | 95% CI | Interaction p |
|----------|-------------|----------------|------------|--------|---------------|
| Free tier | 27.1% | 33.8% | +6.7 pp | [3.8, 9.6] | 0.042 |
| Starter tier | 38.4% | 42.1% | +3.7 pp | [0.1, 7.3] | — |
| Pro tier | 48.2% | 51.7% | +3.5 pp | [-1.4, 8.4] | — |
| Enterprise tier | 62.8% | 64.1% | +1.3 pp | [-6.2, 8.8] | — |
| Organic acquisition | 36.8% | 41.2% | +4.4 pp | [1.2, 7.6] | 0.71 |
| Paid acquisition | 31.4% | 37.1% | +5.7 pp | [2.5, 8.9] | — |
| Referral acquisition | 40.2% | 44.8% | +4.6 pp | [-0.1, 9.3] | — |
| Company size 1-10 | 30.1% | 36.4% | +6.3 pp | [3.2, 9.4] | 0.11 |
| Company size 11-50 | 35.7% | 39.8% | +4.1 pp | [0.5, 7.7] | — |
| Company size 51+ | 40.4% | 43.1% | +2.7 pp | [-0.9, 6.3] | — |

The plan tier x treatment interaction is significant (p = 0.042), suggesting the effect is moderated by plan tier, with Free tier users showing the largest benefit. Other interactions are not significant after multiplicity adjustment (Holm-Bonferroni).

Note: subgroup analyses are exploratory and should not be interpreted as confirmatory. The experiment was powered for the overall effect, not for subgroup differences.

### Secondary Endpoints

| Metric | Control | Treatment | Difference | 95% CI | p |
|--------|---------|-----------|------------|--------|---|
| Median time to first project (hours) | 31.2 | 18.4 | -12.8 | [-15.1, -10.5] | < 0.001 |
| Projects created (14d), mean | 1.8 | 2.6 | +0.8 | [0.6, 1.0] | < 0.001 |
| Sessions per week (first 2 weeks), mean | 3.4 | 4.1 | +0.7 | [0.4, 1.0] | < 0.001 |
| Template selection rate (treatment only) | — | 71.3% | — | — | — |
| 14-day retention | 52.1% | 57.8% | +5.7 pp | [3.6, 7.8] | < 0.001 |
| 28-day retention | 41.3% | 45.2% | +3.9 pp | [1.8, 6.0] | < 0.001 |

All secondary endpoints show improvement in the treatment group. The 14-day retention lift of 5.7 pp is notable and was not a pre-specified endpoint but warrants follow-up.

### Sequential Monitoring

The experiment used a group sequential design with O'Brien-Fleming alpha spending function (3 interim looks at 33%, 67%, and 100% of planned enrollment). The experiment crossed the efficacy boundary at the second interim look (67% enrollment, adjusted alpha = 0.019, observed p = 0.004). Per the pre-registered stopping rules, enrollment was continued to planned completion to enable subgroup analyses and secondary endpoint evaluation.

---

## Regression Analysis

To estimate the treatment effect while adjusting for observed covariates, we fit the following logistic regression:

logit(P(adoption)) = beta_0 + beta_1 * treatment + beta_2 * plan_tier + beta_3 * acquisition_channel + beta_4 * company_size + beta_5 * region

| Predictor | Coefficient | SE | OR | 95% CI | p |
|-----------|------------|----|----|--------|---|
| Treatment | 0.224 | 0.044 | 1.251 | [1.149, 1.362] | < 0.001 |
| Starter (vs. Free) | 0.508 | 0.052 | 1.662 | [1.501, 1.840] | < 0.001 |
| Pro (vs. Free) | 0.912 | 0.068 | 2.489 | [2.180, 2.841] | < 0.001 |
| Enterprise (vs. Free) | 1.524 | 0.098 | 4.590 | [3.785, 5.566] | < 0.001 |
| Paid (vs. Organic) | -0.187 | 0.048 | 0.829 | [0.755, 0.911] | < 0.001 |
| Referral (vs. Organic) | 0.174 | 0.061 | 1.190 | [1.056, 1.341] | 0.004 |
| Direct (vs. Organic) | 0.042 | 0.063 | 1.043 | [0.921, 1.181] | 0.508 |
| Company 11-50 (vs. 1-10) | 0.191 | 0.050 | 1.211 | [1.098, 1.335] | < 0.001 |
| Company 51+ (vs. 1-10) | 0.347 | 0.056 | 1.415 | [1.268, 1.579] | < 0.001 |

The covariate-adjusted OR for treatment is 1.251 [1.149, 1.362], consistent with the unadjusted analysis. Model discrimination: AUC = 0.638. Hosmer-Lemeshow goodness-of-fit: p = 0.287 (adequate fit).

---

## Bayesian Sensitivity Analysis

As a robustness check, we estimated the posterior distribution of the treatment effect using a Beta-Binomial model with weakly informative priors (Beta(1,1) for both groups):

- Posterior mean difference: +5.0 pp
- 95% credible interval: [3.1 pp, 6.9 pp]
- P(treatment effect > 0) = 0.9999
- P(treatment effect > 3 pp MDE) = 0.967

The Bayesian analysis is consistent with the frequentist results and provides an intuitive probability statement: there is a 96.7% probability that the true effect exceeds our minimum practically meaningful threshold.

---

## Threats to Validity

### Internal Validity
- **Selection bias:** Mitigated by random assignment; balance check confirms equivalence.
- **Attrition:** 3.2% of control and 2.8% of treatment users had incomplete tracking data. ITT analysis includes these users as non-adopters. Per-protocol analysis excluding them yields a slightly larger effect (+5.4 pp), suggesting the ITT estimate is conservative.
- **Instrumentation:** Feature adoption tracking was implemented 2 weeks before experiment launch and validated against server-side logs. No discrepancies found.
- **Novelty effect:** Possible. The 28-day retention lift (+3.9 pp) is smaller than the 14-day retention lift (+5.7 pp), which could indicate a novelty effect or simply differential timing. A 90-day follow-up is recommended.

### External Validity
- The experiment was conducted on web only. Mobile adoption patterns may differ.
- Holiday season (November-December) may influence baseline behavior.
- Templates were designed for current product categories; the effect may differ as the product evolves.

### Statistical Validity
- Multiple testing: Primary endpoint is a single pre-specified comparison. Subgroup and secondary analyses are exploratory and flagged as such.
- Sequential monitoring: Alpha spending function preserves overall type I error at 0.05.
- Effect heterogeneity: The interaction with plan tier is noted but does not invalidate the overall result.

---

## Template Usage Patterns

Within the treatment group, template usage breaks down as follows:

| Template Category | Selection Rate | Projects Created | Median Completion Time |
|-------------------|---------------|-----------------|----------------------|
| Marketing | 31.4% | 892 | 4.2 min |
| Engineering | 28.7% | 814 | 5.8 min |
| General | 24.1% | 684 | 3.1 min |
| Design | 15.8% | 448 | 6.4 min |
| (No template used) | 28.7% | 1,312 | 8.7 min |

Users who selected a template completed project setup 42% faster on average (4.7 min vs. 8.7 min). The "General" category had the fastest completion time, while "Design" templates took longest, likely due to additional configuration options in the Design template.

Template abandonment rate (started selecting a template but reverted to blank): 8.4%. Primary abandonment reason (from click-stream analysis): template category browsing exceeded 60 seconds, suggesting decision fatigue when too many options are presented.

---

## Guardrail Metrics

| Guardrail Metric | Control | Treatment | Difference | Threshold | Status |
|-----------------|---------|-----------|------------|-----------|--------|
| Page load time (p50) | 1.2s | 1.3s | +0.1s | < +0.5s | PASS |
| Page load time (p99) | 4.1s | 4.4s | +0.3s | < +1.0s | PASS |
| Error rate (project creation) | 2.1% | 2.3% | +0.2 pp | < +1.0 pp | PASS |
| Support ticket rate (14d) | 3.4% | 3.2% | -0.2 pp | < +0.5 pp | PASS |
| Blank project creation rate | 34.4% | 24.8% | -9.6 pp | > -20 pp | PASS |

All guardrail metrics are within pre-specified thresholds. The decrease in blank project creation is expected and mechanical (templates substitute for blank creation).

---

## Raw Data Summary

### Daily Enrollment Counts

| Date Range | Control Enrolled | Treatment Enrolled | Cumulative Control | Cumulative Treatment |
|------------|-----------------|-------------------|-------------------|---------------------|
| Nov 1-7 | 724 | 718 | 724 | 718 |
| Nov 8-14 | 691 | 703 | 1,415 | 1,421 |
| Nov 15-21 | 712 | 698 | 2,127 | 2,119 |
| Nov 22-28 | 643 | 651 | 2,770 | 2,770 |
| Nov 29-Dec 5 | 687 | 679 | 3,457 | 3,449 |
| Dec 6-12 | 724 | 711 | 4,181 | 4,160 |
| Dec 13-15 | 431 | 427 | 4,612 | 4,587 |

### Adoption Curves

Cumulative adoption over the 14-day window (treatment vs. control):

| Day | Control Cumulative Adoption | Treatment Cumulative Adoption |
|-----|---------------------------|------------------------------|
| 1 | 8.2% | 14.1% |
| 2 | 13.4% | 20.8% |
| 3 | 17.1% | 25.2% |
| 4 | 19.8% | 28.4% |
| 5 | 22.0% | 30.7% |
| 6 | 23.8% | 32.4% |
| 7 | 25.4% | 33.9% |
| 8 | 27.1% | 35.2% |
| 9 | 28.4% | 36.1% |
| 10 | 29.7% | 37.0% |
| 11 | 30.8% | 37.7% |
| 12 | 31.8% | 38.3% |
| 13 | 33.0% | 38.9% |
| 14 | 34.4% | 39.4% |

---

## Statistical Code Reference

Primary analysis conducted in Python 3.11 using:
- `statsmodels` 0.14.1 (proportion tests, logistic regression)
- `scipy` 1.12.0 (chi-squared tests, z-tests)
- `lifelines` 0.28.0 (survival analysis for time-to-event secondary endpoints)
- Custom CUPED implementation validated against internal experimentation platform

Reproducibility: Analysis notebook available at `analytics/experiments/exp-031/analysis_v2.ipynb`. Data snapshot frozen at `warehouse.experiments.exp_031_snapshot_20260115`.

---

## Appendix A: Pre-Registration

The experiment was pre-registered in Acme's internal experiment tracker (EXP-2025-Q4-031) on October 28, 2025. Pre-registration specified:
- Primary endpoint: 14-day feature adoption rate
- MDE: 3 pp absolute
- Alpha: 0.05 (two-sided)
- Power: 0.80
- Sequential monitoring: 3 interim looks with O'Brien-Fleming spending
- Subgroup analyses: plan tier, acquisition channel, company size (all exploratory)

No deviations from the pre-registered analysis plan occurred.

## Appendix B: Detailed Balance Tables

Full balance tables with all 47 covariates are available in the analysis notebook. The 16 covariates shown in the main report were selected as the most business-relevant. No covariate among the full 47 has SMD > 0.035.
