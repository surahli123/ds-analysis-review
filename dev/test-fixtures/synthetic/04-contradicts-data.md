# Q3 2025 Engagement Metrics Review

**Author:** Morgan Liu, Product Analytics
**Date:** October 14, 2025
**Audience:** Product Leadership, Growth Team

---

## TL;DR

Platform engagement showed mixed signals in Q3 2025. While user acquisition and weekly active users grew, core engagement depth metrics declined, driven primarily by a shift in user mix toward lower-intent segments. Action is needed to re-engage mid-funnel users before the engagement decline compounds into retention losses in Q4.

---

## Background

This quarterly review covers Acme Platform engagement metrics for Q3 2025 (July 1 – September 30). The review is part of the standing quarterly cadence requested by the VP of Product to track platform health. All metrics use consistent definitions from the Acme Metrics Glossary (v3.2, updated June 2025).

Key context for Q3: the Growth team launched an expanded paid social campaign in August targeting small business owners, and the Product team shipped a redesigned dashboard in September. Both may influence the metrics discussed below.

---

## User Acquisition & Top-of-Funnel

New signups in Q3 totaled 41,208 — up 18.3% from Q2's 34,831. The acceleration is largely attributable to the paid social expansion, which drove 12,440 signups (30.2% of total), compared to 4,210 from paid social in Q2.

| Month | New Signups | QoQ Growth |
|-------|------------|------------|
| July | 12,104 | +8.2% vs. April |
| August | 14,387 | +22.1% vs. May |
| September | 14,717 | +24.7% vs. June |

Weekly active users (WAU) averaged 89,400 across Q3, up from 82,100 in Q2 (+8.9%). The WAU trend is positive throughout the quarter, ending September at 93,200.

---

## Core Engagement Metrics

### Daily Active Sessions

Daily active sessions (defined as unique user-days with at least one session exceeding 30 seconds) averaged 52,140 per day in Q3, compared to 56,670 per day in Q2.

| Month | Avg Daily Active Sessions | Change vs. Prior Quarter Avg |
|-------|--------------------------|------------------------------|
| July | 54,820 | -3.3% |
| August | 52,410 | -7.5% |
| September | 49,190 | -13.2% |

This represents an **8.0% decline** in average daily active sessions from Q2 to Q3. The decline accelerated through the quarter, with September's average 13.2% below the Q2 baseline.

The per-user session frequency also declined: average sessions per WAU per week dropped from 3.8 in Q2 to 3.3 in Q3, a 13.2% decrease. This indicates the decline is not simply a denominator effect from WAU growth — users are individually engaging less frequently.

### Session Duration

Average session duration decreased from 14.2 minutes in Q2 to 12.8 minutes in Q3 (-9.9%). The decline is concentrated in the 0-5 minute session bucket, which grew from 28% to 36% of all sessions, suggesting an increase in "bounce" sessions where users log in but don't reach meaningful engagement.

| Session Duration Bucket | Q2 Share | Q3 Share | Change |
|------------------------|----------|----------|--------|
| 0-5 minutes | 28% | 36% | +8 pp |
| 5-15 minutes | 34% | 31% | -3 pp |
| 15-30 minutes | 24% | 21% | -3 pp |
| 30+ minutes | 14% | 12% | -2 pp |

### Feature Adoption Depth

We track "feature depth" as the number of distinct feature categories used per user per week (out of 8 possible categories: Projects, Collaboration, Search, Templates, Integrations, Analytics, Settings, Help).

Average feature depth declined from 3.4 categories in Q2 to 3.0 categories in Q3 (-11.8%). The decline is most pronounced among users acquired through paid social channels (average depth: 2.1 categories vs. 3.6 for organic users).

---

## Segment-Level Analysis

### Engagement by Acquisition Channel

| Channel | WAU (Q3 Avg) | Sessions/WAU/Week | Avg Session Duration |
|---------|-------------|-------------------|---------------------|
| Organic Search | 34,200 | 4.1 | 16.2 min |
| Paid Search | 18,700 | 3.4 | 13.1 min |
| Referral | 12,300 | 4.4 | 17.8 min |
| Paid Social | 15,800 | 2.1 | 8.4 min |
| Direct | 8,400 | 3.7 | 14.7 min |

Paid social users — the fastest-growing segment — engage at roughly half the frequency and half the session duration of organic or referral users. Because paid social grew from 12% to 18% of the active user base during Q3, this compositional shift explains a meaningful portion of the aggregate decline.

A Blinder-Oaxaca decomposition estimates that 61% of the aggregate daily active session decline is attributable to compositional shifts (more low-engagement users in the mix), while 39% reflects genuine behavioral decline within existing segments. The within-segment decline is concentrated among Starter-tier users, whose session frequency dropped 9.2% even after controlling for channel mix.

### Engagement by Plan Tier

| Plan Tier | Sessions/WAU/Week Q2 | Sessions/WAU/Week Q3 | Change |
|-----------|----------------------|----------------------|--------|
| Free | 2.8 | 2.4 | -14.3% |
| Starter | 4.1 | 3.7 | -9.8% |
| Pro | 5.2 | 5.0 | -3.8% |
| Enterprise | 6.1 | 6.0 | -1.6% |

The engagement decline is inversely correlated with plan tier — free users show the steepest decline, while Enterprise users are essentially flat. This pattern is consistent with the hypothesis that the new paid social users (predominantly Free tier) are diluting engagement metrics.

---

## Dashboard Redesign Impact

The dashboard redesign shipped September 8 and affected all users. Early signals from the final 3 weeks of Q3:

- Average time-on-dashboard increased from 2.1 to 3.4 minutes (+62%), suggesting users are spending more time on the new surface.
- Click-through rate from dashboard to feature areas declined from 48% to 39% (-19%), indicating the new dashboard may be less effective at driving users deeper into the product.
- The net effect on overall session quality is ambiguous at this point and warrants a dedicated follow-up analysis after a full month of data.

---

## Conclusion

Q3 2025 was a strong quarter for user growth — signups grew 18.3% and WAU grew 8.9%, both driven by the paid social expansion. Daily active sessions improved over Q3, indicating positive engagement trends that complement the top-of-funnel growth.

However, the engagement depth story is more nuanced. Feature adoption depth declined 11.8%, and session durations shortened, particularly among paid social users. The Starter tier within-segment decline (-9.8% session frequency) is a leading indicator worth monitoring, as historically a 5%+ within-segment engagement decline has preceded retention deterioration within 1-2 quarters.

### Recommendations

1. **Audit the paid social funnel.** Paid social users engage at roughly half the rate of organic users. The Growth team should evaluate whether targeting criteria or landing page expectations are attracting users with low product-market fit, or whether the onboarding flow is failing to convert these users into engaged platform users.

2. **Investigate the Starter tier engagement decline.** The 9.8% within-segment decline is not explained by channel mix shifts and may indicate a product issue specific to this tier. Recommend a targeted user research study (n=15-20 qualitative interviews) to identify friction points.

3. **Conduct a dedicated dashboard redesign analysis.** The September 8 redesign shows promising surface engagement but concerning click-through declines. A full analysis with 30+ days of data should evaluate whether the redesign is helping or hurting downstream engagement.

4. **Set a Q4 guardrail metric.** Propose a "minimum engagement floor" for paid social cohorts: if 30-day session frequency for paid social users does not reach 2.5 sessions/week by end of Q4, the channel expansion should be paused for review.
