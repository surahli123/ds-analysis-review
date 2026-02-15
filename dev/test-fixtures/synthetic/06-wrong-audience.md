# Checkout Flow Redesign A/B Test — Final Results

**Author:** Sarah Chen, Senior Data Scientist
**Date:** September 22, 2025
**Test ID:** EXP-2025-0147
**Test Duration:** August 11 – September 8, 2025 (28 days)

---

## Experiment Design

### Hypothesis and Motivation

We hypothesized that simplifying the checkout flow from a 4-step process to a single-page layout would reduce friction and improve conversion rate among users who reach the cart page. The treatment variant consolidated shipping address, payment method, order review, and confirmation into a single scrollable page with inline validation and a persistent order summary sidebar.

### Randomization and Assignment

Traffic was randomized at the user-ID level using our internal experimentation platform (Acme Experiment Service v3.2) with a Murmur3 hash function. We used a 50/50 split with stratification on three dimensions: device type (mobile/desktop), customer tenure bucket (new / 1-6 months / 6+ months), and region (NA / EU / APAC). The stratification was implemented via blocked randomization within each of the 18 strata (3 device types collapsed to 2 for mobile/desktop × 3 tenure × 3 region). We verified balance post-assignment using chi-squared tests on each stratification dimension (all p > 0.40).

### Sample Size and Power Analysis

Pre-experiment power analysis assumed a baseline checkout conversion rate of 28.4% (trailing 90-day average for cart-reaching users), minimum detectable effect (MDE) of 1.5 percentage points (relative lift of ~5.3%), significance level alpha = 0.05 (two-tailed), and target power 1 - beta = 0.80. Using the formula for two-proportion z-test:

n = (Z_{alpha/2} + Z_beta)^2 × (p1(1-p1) + p2(1-p2)) / (p1 - p2)^2

This yielded a required sample size of approximately 4,800 users per arm for the primary metric. We targeted 6,000 per arm to provide additional power for subgroup analyses. Actual enrollment was 6,247 (control) and 6,231 (treatment) after excluding users who triggered the experiment but never loaded the checkout page (intent-to-treat violation filter).

### Metrics Definition

**Primary metric:** Checkout conversion rate — defined as the proportion of users who reached the cart page and completed a purchase within the same session or within a 24-hour attribution window.

**Secondary metrics:**
- Time-to-completion: median and p90 elapsed time from first checkout page load to order confirmation, measured in seconds
- Cart abandonment rate: proportion of cart-reaching users who did not complete purchase within the attribution window
- Error rate: proportion of checkout sessions with at least one form validation error
- Revenue per cart-reaching user: total revenue divided by number of cart-reaching users, to capture both conversion and AOV effects

**Guardrail metrics:**
- Refund rate within 7 days of purchase
- Customer support ticket rate within 48 hours of purchase
- Payment failure rate

---

## Statistical Methodology

### Primary Analysis

We computed the difference in proportions for the primary metric using a two-sided z-test with unpooled standard errors. The test statistic is:

Z = (p_treatment - p_control) / sqrt(SE_treatment^2 + SE_control^2)

where SE = sqrt(p(1-p)/n) for each arm.

For continuous secondary metrics (time-to-completion, revenue per user), we used Welch's t-test due to unequal variances between arms (Levene's test p = 0.023 for time-to-completion).

### Multiple Testing Correction

We applied the Benjamini-Hochberg procedure to control the false discovery rate (FDR) at q = 0.05 across the 5 secondary metrics. The primary metric was tested at alpha = 0.05 without correction, consistent with our pre-registered analysis plan.

### Bootstrap Confidence Intervals

To validate the parametric confidence intervals and account for potential non-normality in the revenue-per-user metric (which has a right-skewed distribution with a mass at zero), we computed bootstrap confidence intervals using the bias-corrected and accelerated (BCa) method with B = 10,000 bootstrap resamples. The bootstrap was stratified by the same dimensions used in randomization to preserve the covariate structure.

The BCa intervals for checkout conversion rate were [1.42pp, 3.18pp], compared to the Wald interval of [1.38pp, 3.22pp]. The close agreement confirms that the normal approximation is adequate for the primary metric.

For revenue per user, the BCa interval was [$0.82, $3.41] versus the Wald interval of [$0.64, $3.58], showing modest asymmetry consistent with the right-skewed distribution. We report the BCa intervals for this metric.

### Sequential Testing Note

We used a group-sequential design with two interim analyses at 33% and 66% information fraction, using O'Brien-Fleming spending function boundaries. Neither interim analysis crossed the futility or efficacy boundary, and the final analysis uses the full-information critical value of Z = 2.024 (slightly more conservative than 1.96 due to alpha-spending).

---

## Results

### Primary Metric

| Metric | Control | Treatment | Difference | 95% CI | p-value |
|--------|---------|-----------|------------|--------|---------|
| Checkout Conversion Rate | 28.2% | 30.5% | +2.3pp | [1.42, 3.18] | 0.0004 |

The observed effect size corresponds to Cohen's h = 0.051, which falls in the "small" range by conventional benchmarks. However, given the large baseline volume of checkout-reaching users (~220K/month), even a small effect translates to meaningful business volume.

The Bayesian posterior probability that the treatment effect is positive is P(delta > 0 | data) = 0.9998, computed using a Beta(1,1) prior on each arm's conversion rate. The posterior median effect is +2.28pp with a 95% highest density interval of [1.39, 3.17].

### Secondary Metrics

| Metric | Control | Treatment | Difference | 95% CI | p-value | BH-adjusted p |
|--------|---------|-----------|------------|--------|---------|----------------|
| Median Time-to-Completion | 187s | 142s | -45s | [-52, -38] | <0.0001 | <0.0001 |
| P90 Time-to-Completion | 384s | 298s | -86s | [-101, -71] | <0.0001 | <0.0001 |
| Cart Abandonment Rate | 71.8% | 69.5% | -2.3pp | [-3.18, -1.42] | 0.0004 | 0.0005 |
| Error Rate | 12.4% | 8.1% | -4.3pp | [-5.42, -3.18] | <0.0001 | <0.0001 |
| Revenue / Cart-Reaching User | $49.12 | $51.24 | +$2.12 | [$0.82, $3.41]* | 0.0014 | 0.0018 |

*BCa bootstrap confidence interval reported for this metric.

All secondary metrics are statistically significant after BH correction and directionally consistent with the hypothesis.

### Guardrail Metrics

| Metric | Control | Treatment | Difference | 95% CI | p-value |
|--------|---------|-----------|------------|--------|---------|
| 7-day Refund Rate | 3.2% | 3.1% | -0.1pp | [-0.8, 0.6] | 0.78 |
| Support Ticket Rate | 2.8% | 2.4% | -0.4pp | [-1.0, 0.2] | 0.19 |
| Payment Failure Rate | 1.9% | 1.7% | -0.2pp | [-0.7, 0.3] | 0.43 |

No guardrail metrics showed statistically significant movement, suggesting the new flow does not introduce downstream quality issues.

### Subgroup Analysis

We conducted exploratory subgroup analyses across our stratification dimensions. Due to the exploratory nature, we present these without multiple testing correction and flag them as hypothesis-generating only.

| Subgroup | Control CVR | Treatment CVR | Lift | p-value | Interaction p |
|----------|-------------|---------------|------|---------|---------------|
| Mobile | 22.1% | 25.4% | +3.3pp | <0.001 | 0.018 |
| Desktop | 38.7% | 39.4% | +0.7pp | 0.34 | — |
| New Customers | 24.8% | 28.1% | +3.3pp | <0.001 | 0.024 |
| Tenure 1-6mo | 29.4% | 30.8% | +1.4pp | 0.12 | — |
| Tenure 6mo+ | 32.1% | 33.2% | +1.1pp | 0.21 | — |
| NA | 28.9% | 31.4% | +2.5pp | 0.001 | 0.72 |
| EU | 27.8% | 30.0% | +2.2pp | 0.007 | — |
| APAC | 26.4% | 28.6% | +2.2pp | 0.041 | — |

The interaction tests suggest meaningfully larger effects on mobile (interaction p = 0.018) and for new customers (interaction p = 0.024). We recommend the single-page checkout be prioritized for mobile rollout if a staged deployment is preferred.

### Novelty and Primacy Effects

We examined weekly conversion rates to assess whether the treatment effect attenuated over time (novelty) or grew (learning). The week-over-week treatment effects were: +2.1pp (W1), +2.5pp (W2), +2.4pp (W3), +2.2pp (W4). A regression of treatment effect on time showed no significant trend (beta = -0.02, p = 0.89), suggesting the effect is stable and not driven by novelty.

---

## Sensitivity Analyses

### CUPED Variance Reduction

We applied CUPED (Controlled-experiment Using Pre-Experiment Data) using 30-day pre-experiment checkout conversion rate as the covariate. The CUPED-adjusted treatment effect was +2.31pp (95% CI: [1.58, 3.04]), representing a 19% reduction in confidence interval width compared to the unadjusted analysis. The point estimate is consistent with the unadjusted result.

### Winsorization

For the revenue-per-user metric, we applied 99th percentile winsorization to address potential outlier sensitivity. The winsorized treatment effect was +$1.98 (95% CI: [$0.71, $3.25]), compared to the unwinsorized +$2.12. The result is robust to outlier treatment.

### Exclusion of Bot Traffic

Excluding sessions flagged by our bot detection system (1.2% of traffic) did not materially change results. Primary metric effect: +2.3pp (unchanged to one decimal place).

---

## Limitations

1. The experiment ran during a period without major promotions or seasonal events, so generalizability to peak shopping periods (Black Friday, holiday season) is uncertain.
2. The single-page checkout was not tested with guest checkout users (account-only in treatment). Guest checkout compatibility is a separate engineering workstream.
3. Subgroup analyses are exploratory and should not be used for differential rollout decisions without confirmatory testing.
4. Long-term effects on repurchase rate and LTV were not measured in this 28-day window.

---

## Appendix: Diagnostic Plots

- QQ plot of per-user revenue residuals (treatment arm): moderate right-skew, supporting BCa interval choice
- Assignment balance verification: chi-squared test results across all 18 strata
- Sequential monitoring: cumulative Z-statistic trajectory with O'Brien-Fleming boundaries
- SRM check: observed split 50.06% / 49.94%, chi-squared p = 0.87 (no sample ratio mismatch)
