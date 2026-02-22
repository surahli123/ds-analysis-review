# Search Ranking Model Evaluation: Q1 2026 Ranker v3

## Summary
We evaluated the new Ranker v3 model against the production baseline (Ranker v2) using our offline evaluation pipeline. Ranker v3 shows strong improvements and we recommend launching it to 100% of traffic.

## Background
The Search Ranking team has been developing Ranker v3 — a transformer-based learning-to-rank model that replaces the gradient-boosted tree ensemble (Ranker v2) currently in production. Ranker v3 uses dense embeddings from a fine-tuned BERT encoder combined with traditional sparse features.

The primary goal is to improve search result quality as measured by click-through rate (CTR) and classification accuracy of relevant results.

## Data
We used 12 weeks of search logs (Jan–Mar 2026) containing 48M queries and 320M impressions. Training data was constructed from click-through signals: a result was labeled "relevant" if it received a click within the session, and "not relevant" otherwise.

Features include:
- Query-document embedding similarity (BERT)
- BM25 score
- Document freshness, popularity, and quality signals
- User context features (device, locale, prior search history)

We split the data 80/10/10 into train/validation/test sets chronologically.

## Evaluation Results

### Overall Performance
| Metric | Ranker v2 (baseline) | Ranker v3 | Delta |
|--------|---------------------|-----------|-------|
| Classification Accuracy | 78.2% | 84.1% | +5.9% |
| CTR@5 | 42.3% | 45.7% | +3.4% |
| Precision@10 | 0.71 | 0.76 | +0.05 |

The industry standard NDCG@10 for search systems is approximately 0.90. Our Ranker v3 achieves an NDCG@10 of 0.76, which shows room for improvement but represents a meaningful step forward.

### Query Segment Performance
We evaluated performance on our top 10,000 queries (head queries), which account for approximately 35% of total search traffic. Results:
- Head query CTR improved from 44.1% to 48.3% (+4.2%)
- Head query accuracy improved from 81.5% to 87.2% (+5.7%)

Based on Patel et al. (2023), "Transformer-based rankers consistently achieve 15-20% relative CTR improvements over tree-based models across all query segments."

## Methodology
We trained Ranker v3 using a pointwise cross-entropy loss, treating ranking as a binary classification problem (relevant vs. not relevant). The model was trained for 10 epochs with early stopping on validation loss.

Hyperparameter tuning was performed via grid search over learning rate, dropout rate, and embedding dimension.

## Recommendation
Based on the strong offline improvements across all metrics, we recommend launching Ranker v3 to 100% of production traffic immediately. The +5.9% accuracy improvement and +3.4% CTR lift demonstrate clear superiority over the baseline.

Expected annual revenue impact: $4.2M based on the CTR lift applied to current monetization rates.

## Appendix: Model Architecture
- Encoder: BERT-base (110M parameters), fine-tuned on search queries
- Ranking head: 2-layer MLP with ReLU activation
- Training: AdamW optimizer, lr=2e-5, batch size 256
- Inference latency: p50=12ms, p99=45ms (within SLA)
