# Search Ranking Model v2 — Impact Analysis

## Background
- Current model (v1) launched March 2025
- Main complaint: long-tail queries return irrelevant results
- v2 adds semantic embeddings alongside BM25 signals
- Need to quantify impact before requesting eng resources for prod rollout

## Offline Evaluation
- NDCG@10 improved from 0.61 to 0.68 on held-out test set
- Biggest gains on tail queries (volume < 100/week): +14% NDCG
- Head queries roughly flat (+1.2% NDCG, not significant)
- TODO: re-run on fresh judgment set — current labels are 4 months stale

## Online A/B Test Plan
- 10% traffic holdout, 2-week minimum run
- Primary metric: search conversion rate
- Secondary: CTR@1, null result rate, time-to-first-click
- Need to align with ML team on feature serving latency budget
- TODO: confirm with infra team that embedding index can handle 10% traffic

## Expected Business Impact
- Rough estimate: if search CVR lifts 5-8%, that's ~$X00K/quarter incremental revenue
- TODO: get exact search revenue baseline from finance team
- Need to factor in serving cost increase for embeddings

## Retention & Engagement
- Hypothesis: better search → higher repeat usage
- TODO: add retention data from v1 launch to show precedent
- Need to verify with ML team whether engagement metrics moved last time

## Risks
- Latency: embedding lookup adds ~30ms p50, ~80ms p95
- Cost: estimated $12K/month additional serving infrastructure
- TODO: validate cost estimate with infra

## Next Steps
- [ ] Finalize offline eval with fresh judgments
- [ ] Get sign-off from ML lead on A/B test design
- [ ] Coordinate with infra on capacity
- [ ] Draft experiment brief for product review
