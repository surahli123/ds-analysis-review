# Smart Lists Drove a 12% Increase in 30-Day Retention

**Author:** Alex Chen, Product Analytics
**Date:** January 22, 2026
**Audience:** Product Leadership, Growth Team

---

## TL;DR

After launching Smart Lists on November 15, 2025, Acme's 30-day user retention improved from 38.1% to 42.7% — a **12.1% relative increase**. Smart Lists users show significantly higher engagement across all key metrics. We recommend expanding Smart Lists to all plan tiers and investing in discoverability to accelerate adoption.

---

## Background

Smart Lists is an AI-powered feature that automatically organizes a user's workspace items into contextual collections based on usage patterns and content similarity. The feature was developed in response to user research indicating that "finding things I've already created" was the #2 pain point (after onboarding complexity) in our Q3 2025 NPS verbatim analysis.

Smart Lists launched on November 15, 2025, initially available to Pro and Enterprise plan users. The rollout was announced via in-app banner, email campaign to eligible users, and a blog post. As of January 15, 2026, 67% of eligible users have activated Smart Lists at least once, and 41% are weekly active users of the feature.

---

## Retention Analysis

### Pre vs. Post Launch Comparison

We compared 30-day retention for users who signed up in the 8 weeks before the Smart Lists launch (September 20 – November 14, 2025) against users who signed up in the 8 weeks after (November 15, 2025 – January 9, 2026).

| Period | Cohort Size | 30-Day Retention | |
|--------|------------|-----------------|---|
| Pre-launch (Sep 20 – Nov 14) | 12,844 | 38.1% | |
| Post-launch (Nov 15 – Jan 9) | 13,201 | 42.7% | |
| **Difference** | | **+4.6 pp (+12.1% relative)** | p < 0.001 |

The improvement is statistically significant (chi-squared test, p < 0.001). The effect is consistent across weekly cohorts within each period — no single week is driving the result.

### Engagement Metrics

Smart Lists users demonstrate substantially higher engagement compared to non-users:

| Metric | Smart Lists Users | Non-Users | Difference |
|--------|------------------|-----------|------------|
| Sessions per week | 5.8 | 3.2 | +81% |
| Items created per week | 12.4 | 7.1 | +75% |
| Collaboration actions per week | 3.1 | 1.4 | +121% |
| Time in app per session (min) | 18.7 | 11.3 | +65% |

These engagement differences confirm that Smart Lists is making users more productive and more engaged with the platform, which naturally translates to higher retention.

### Retention by Plan Tier

Breaking down the retention improvement by plan tier:

| Plan Tier | Pre-Launch Retention | Post-Launch Retention | Change |
|-----------|---------------------|----------------------|--------|
| Pro | 51.3% | 56.8% | +5.5 pp |
| Enterprise | 68.2% | 71.4% | +3.2 pp |
| Starter | 34.7% | 37.9% | +3.2 pp |
| Free | 22.4% | 24.1% | +1.7 pp |

The largest absolute gain is among Pro users, which aligns with Smart Lists being available on Pro and Enterprise tiers. Notably, even Starter and Free users — who don't have access to Smart Lists — show modest retention improvements, likely due to broader platform improvements and seasonal effects.

### Time-to-Value Analysis

We also measured time-to-first-value-action (defined as creating a project or inviting a collaborator) for the pre and post periods:

| Period | Median Time to First Value (hours) |
|--------|-----------------------------------|
| Pre-launch | 26.4 |
| Post-launch | 19.8 |

The 25% reduction in time-to-value suggests that Smart Lists helps new users organize their work faster, getting them to productive use of the platform sooner.

---

## User Feedback

We collected qualitative feedback from 847 Smart Lists users via an in-app survey (response rate: 8.3%). Key themes:

- 72% rated Smart Lists as "very useful" or "extremely useful"
- "It saves me 10-15 minutes per day not having to manually organize things" — recurring sentiment
- Top feature request: ability to pin or customize Smart List categories (34% of respondents)
- 12% of respondents reported confusion about how items were categorized

The net promoter score for Smart Lists users is 47, compared to 31 for the overall platform — a 16-point lift attributable to the feature.

---

## Revenue Impact Estimate

Based on the retention improvement and current revenue per user metrics:

- Additional retained users per month (Pro + Enterprise): ~680
- Average monthly revenue per retained user: $94
- **Estimated monthly revenue impact: $63,920**
- **Annualized: ~$767,000**

This estimate is conservative as it does not account for expansion revenue from retained users upgrading their plans or adding seats over time.

---

## Recommendations

1. **Expand Smart Lists to all plan tiers.** The feature's impact on retention justifies making it broadly available. Even a partial version for Free/Starter users could meaningfully move retention in those tiers.

2. **Invest in discoverability.** Only 67% of eligible users have activated Smart Lists. Adding contextual prompts when users perform manual organization actions could increase adoption by an estimated 15-20%.

3. **Prioritize the "custom categories" feature request.** 34% of survey respondents asked for this. Giving users control over categorization would address the confusion reported by 12% of users while deepening engagement.

4. **Run a deeper causal study.** While the pre/post results are compelling, a randomized rollout to remaining tiers would allow us to isolate the Smart Lists effect with greater precision.

---

## Appendix: Methodology Notes

- Retention is defined as at least one logged session within 30 days of signup.
- Users are attributed to the period in which they signed up.
- The chi-squared test was used to compare retention proportions between the pre and post periods.
- Smart Lists engagement metrics are computed for the first 30 days of feature availability per user.
- Revenue estimates use the blended average revenue per user across Pro and Enterprise tiers as of January 2026.
