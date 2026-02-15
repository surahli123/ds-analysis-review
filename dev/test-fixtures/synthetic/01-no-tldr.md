# User Churn Analysis: Q4 2025 Cohort Deep-Dive

**Author:** Jamie Park, Data Science
**Date:** January 15, 2026
**Stakeholders:** Product (Growth), Marketing, Customer Success

---

## Methodology

### Data Sources and Definitions

We define churn as the absence of any logged session within a rolling 28-day window, consistent with the definition adopted by the Growth team in Q2 2025. This analysis covers all users who created an Acme Platform account between October 1 and December 31, 2025 (the "Q4 cohort"), giving us a minimum of 45 days of post-signup behavioral data as of the analysis cutoff date of February 13, 2026.

The dataset was constructed by joining three internal tables:

- `analytics.user_sessions` — timestamped session-level activity, including session duration, feature interactions, and page views.
- `growth.user_attributes` — signup metadata including acquisition channel, plan tier, company size, and geographic region.
- `product.onboarding_events` — granular event stream from the onboarding flow, covering each step from account creation through first value action (defined as creating a project or inviting a team member).

After deduplication and removal of internal test accounts (identified via the `@acme.com` email domain and the internal QA flag), the working dataset contains 34,217 unique users.

### Cohort Construction

Users were segmented into weekly signup cohorts (13 cohorts total across Q4). Each cohort was tracked from signup date through the analysis cutoff. We applied survival analysis using Kaplan-Meier estimation to model time-to-churn, with right-censoring for users still active at the cutoff. Cox proportional hazards regression was then used to identify factors associated with elevated churn risk, controlling for plan tier, acquisition channel, company size, and region.

We verified the proportional hazards assumption using Schoenfeld residuals. The global test yielded p = 0.34, indicating no significant violation.

### Statistical Approach

All hypothesis tests use a significance threshold of alpha = 0.05. Confidence intervals are reported at the 95% level. Where multiple comparisons are made (e.g., across acquisition channels), we apply Bonferroni correction. Effect sizes are reported as hazard ratios with 95% CIs.

---

## Cohort-Level Churn Rates

The overall 28-day churn rate for the Q4 cohort is 23.4% (8,007 of 34,217 users), compared to 19.1% for the Q3 cohort (6,492 of 33,990 users). This represents a 4.3 percentage point increase quarter-over-quarter.

Week-over-week trends show churn rates climbing from 20.8% in the October 1-7 cohort to 27.1% in the December 24-31 cohort. The increase is not monotonic — there is a notable dip in the November 18-24 cohort (18.9%), which coincides with the Black Friday promotional period that attracted a different user mix. Excluding the Black Friday cohort, the upward trend is consistent (Spearman rho = 0.81, p = 0.003).

Churn rates by plan tier:

| Plan Tier | Users | Churned | Churn Rate | 95% CI |
|-----------|-------|---------|------------|--------|
| Free | 18,432 | 5,714 | 31.0% | [30.3%, 31.7%] |
| Starter ($29/mo) | 9,876 | 1,580 | 16.0% | [15.3%, 16.7%] |
| Pro ($79/mo) | 4,311 | 539 | 12.5% | [11.5%, 13.5%] |
| Enterprise | 1,598 | 174 | 10.9% | [9.4%, 12.5%] |

The churn rate gradient across tiers is expected and consistent with historical patterns. Free tier churn is the primary driver of the overall increase — free tier churn rose from 25.2% (Q3) to 31.0% (Q4), accounting for 78% of the total increase in churned users.

---

## Acquisition Channel Analysis

Churn rates vary substantially by acquisition channel, even after controlling for plan tier:

| Channel | Users | Adj. Churn Rate | Hazard Ratio | 95% CI |
|---------|-------|-----------------|--------------|--------|
| Organic Search | 11,240 | 20.1% | 1.00 (ref) | — |
| Paid Search | 8,115 | 22.8% | 1.14 | [1.06, 1.23] |
| Social (Organic) | 5,432 | 25.7% | 1.31 | [1.20, 1.43] |
| Social (Paid) | 4,877 | 29.3% | 1.52 | [1.39, 1.66] |
| Referral | 2,981 | 15.2% | 0.72 | [0.64, 0.81] |
| Direct | 1,572 | 18.4% | 0.90 | [0.79, 1.02] |

Paid social users churn at 1.52x the rate of organic search users (p < 0.001), the highest hazard ratio of any channel. Referral users have the lowest churn risk (HR = 0.72, p < 0.001). These patterns hold after Bonferroni correction for the 5 pairwise comparisons against the reference channel.

---

## Feature Engagement and Behavioral Signals

We examined first-week feature engagement patterns as predictors of subsequent churn. Using the Cox model, we identified the following behavioral variables with the strongest association with churn risk:

| Behavioral Signal | Hazard Ratio | 95% CI | p-value |
|-------------------|-------------|--------|---------|
| Completed onboarding flow | 0.41 | [0.37, 0.45] | < 0.001 |
| Created first project within 48h | 0.53 | [0.48, 0.58] | < 0.001 |
| Invited a team member within 7d | 0.38 | [0.33, 0.44] | < 0.001 |
| Used search feature in first session | 0.67 | [0.61, 0.74] | < 0.001 |
| Viewed help docs in first 3 days | 1.24 | [1.14, 1.35] | < 0.001 |

Completing the onboarding flow is the single strongest protective factor against churn (HR = 0.41), meaning users who finish onboarding have 59% lower churn risk than those who do not, all else equal. However, only 41.3% of Q4 free-tier users completed the full onboarding flow, down from 52.7% in Q3.

The decline in onboarding completion maps directly to a product change shipped on September 28, 2025: the onboarding flow was expanded from 4 steps to 7 steps to incorporate new workspace configuration options. Funnel analysis of the expanded flow shows a steep drop-off between steps 4 and 5 — the newly added "Configure Integrations" step, where 34.2% of users who reached step 4 abandon the flow entirely. The pre-expansion flow had no single step with greater than 12% drop-off.

When we restrict the analysis to users who completed at least 4 onboarding steps (matching the old flow length), the Q4 churn rate drops to 20.3% — within 1.2 percentage points of the Q3 baseline. This strongly suggests that the onboarding expansion is the primary driver of the Q4 churn increase, mediated through reduced onboarding completion rates.

Users who abandon at step 5 exhibit a distinctive behavioral pattern: 62% never return to the onboarding flow, 23% return within 48 hours but abandon again at the same step, and 15% eventually complete the flow after 3+ days. The "never return" segment has a 44.8% churn rate — nearly double the cohort average.

Additionally, the "Viewed help docs" signal being associated with higher churn (HR = 1.24) is counterintuitive at first glance. Deeper investigation reveals this is a confusion signal: users who hit help docs in the first three days are disproportionately those stuck in the new onboarding steps. 71% of early help doc viewers were on step 5 or 6 of the expanded flow when they accessed documentation. This is not a case where documentation causes churn — it is a marker of friction in the flow.

---

## Sensitivity Analysis

To test robustness:

1. **Alternative churn window:** Using a 14-day window instead of 28-day yields qualitatively identical results (onboarding completion remains the dominant predictor, HR = 0.44). The 42-day window shows slightly weaker effects (HR = 0.49) as some users return, but the overall pattern is unchanged.

2. **Excluding Black Friday cohort:** Removing the November 18-24 cohort (which had anomalous channel mix) does not materially change the Cox model coefficients. The onboarding completion hazard ratio shifts from 0.41 to 0.42.

3. **Interaction effects:** We tested a plan_tier x onboarding_completion interaction. The interaction is significant (p = 0.008), indicating that onboarding completion is slightly more protective for free-tier users (HR = 0.38) than for paid users (HR = 0.48). This aligns with the intuition that paid users have higher baseline intent.

---

## Limitations

- This analysis is observational. While the temporal alignment between the onboarding flow change and the churn increase is suggestive, we cannot rule out confounding from seasonality (Q4 holiday period behavioral differences) or simultaneous changes to the marketing mix.
- The 28-day churn definition means some users classified as churned may return. Historical data suggests a 6-8% resurrection rate at 90 days, which would not change the directional findings.
- Company size data is self-reported at signup and may be inaccurate for 12-15% of users who skip this field (imputed as "unknown" in the model).
- We did not analyze in-app messaging or email re-engagement campaigns, which may differentially affect churn across cohorts if campaign cadence changed.

---

## Recommendations

1. **Revert or simplify the onboarding flow.** The data strongly suggests that the expanded onboarding is the primary driver of increased churn. At minimum, make steps 5-7 optional or defer them to post-first-value. The "Configure Integrations" step is the critical bottleneck.

2. **Implement progressive onboarding.** Rather than a linear 7-step flow, allow users to reach their first value action (project creation or team invite) within 3-4 steps, then surface additional configuration contextually.

3. **Run an A/B test on onboarding length.** Before a full revert, validate with a controlled experiment: 4-step (legacy) vs. 7-step (current) vs. 4-step-with-deferred-setup (proposed). Power analysis suggests we need approximately 5,000 users per arm for 80% power to detect a 3 percentage point difference in 28-day churn at alpha = 0.05.

4. **Prioritize paid social acquisition review.** Paid social users churn at 1.52x the baseline rate. This warrants a joint review with Marketing to evaluate whether the audience targeting or landing page experience is setting incorrect expectations.

5. **Build a churn early-warning model.** The behavioral signals identified here (onboarding completion, first project creation, team invite) could feed a real-time scoring model that triggers intervention campaigns within the first 72 hours.
