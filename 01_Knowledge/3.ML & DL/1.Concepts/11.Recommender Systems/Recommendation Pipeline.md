# Recommendation Pipeline

## What is it?

The **recommendation pipeline** is the end-to-end system architecture for delivering recommendations at scale — from raw interaction data to ranked recommendations served to users in milliseconds.

---

## Industrial Pipeline: Two Stages

At scale (millions of users, millions of items), a single complex model can't score all items for a user in real time. The solution is a **two-stage pipeline**:

```
All Items (millions)
        ↓
┌──────────────────────┐
│  Stage 1: RETRIEVAL  │   Fast, recall-focused
│  (Candidate Generation)│  → keeps top ~1000 candidates
└──────────────────────┘
        ↓
┌──────────────────────┐
│  Stage 2: RANKING    │   Slow, precision-focused
│  (Re-ranking/Scoring) │  → ranks top 1000, show top 10
└──────────────────────┘
        ↓
    Top-K Recommendations shown to user
```

---

## Stage 1: Retrieval (Candidate Generation)

**Goal:** From millions of items, efficiently retrieve a few hundred candidates that are likely relevant.

**Methods:**
- **Matrix Factorization / Embedding lookup:** Embed users and items in $\mathbb{R}^k$. Retrieve nearest neighbours of the user embedding using Approximate Nearest Neighbour (ANN) search (FAISS, ScaNN).
- **Two-tower neural network:** Separate user tower and item tower, trained on click data. At inference, pre-compute item embeddings; use ANN to retrieve.
- **Popularity + rules:** Trending items, items from followed users.

**ANN Search:** Exact nearest neighbour is $O(nk)$ per query ($n$ items). ANN methods like HNSW or IVF-PQ give approximate results in $O(\log n)$ — fast enough for real-time.

---

## Stage 2: Ranking

**Goal:** Score the ~1000 candidates with a powerful model that considers many features; return top $K$.

Features for ranking:
- User-item interaction features (embedding dot product)
- User features (demographics, history statistics)
- Item features (popularity, recency, quality)
- Context features (time of day, device, session)
- Cross features (user × item interaction features)

**Models:** Gradient Boosting (GBDT), Deep Neural Networks, Wide & Deep (Google), DIN (Alibaba).

---

## Offline vs. Online

| | Offline (batch) | Online (real-time) |
|---|---|---|
| **Item embeddings** | Pre-computed daily | — |
| **User embeddings** | Updated periodically | Updated per session |
| **ANN index** | Rebuilt periodically | Queried in real-time |
| **Ranking model** | Trained offline | Inference in real-time |
| **Latency** | N/A | < 100ms typically |

---

## Evaluation Metrics for Ranking

| Metric | Formula | What it measures |
|---|---|---|
| **Precision@K** | $\frac{\text{relevant in top-}K}{K}$ | Fraction of top-K that are relevant |
| **Recall@K** | $\frac{\text{relevant in top-}K}{\text{total relevant}}$ | Fraction of relevant items in top-K |
| **NDCG@K** | $\frac{1}{Z}\sum_{k=1}^K\frac{\text{rel}_k}{\log_2(k+1)}$ | Relevance weighted by position |
| **MAP@K** | Mean average precision across users | Overall ranking quality |
| **MRR** | $\frac{1}{|U|}\sum_u\frac{1}{\text{rank of first relevant}}$ | How high first relevant item is |

**NDCG** (Normalised Discounted Cumulative Gain) is the gold standard:
- Items ranked higher contribute more.
- Normalised by ideal ranking (IDCG).
- Range $[0, 1]$; 1 = perfect ranking.

---

## The Cold Start Problem

| Situation | Problem | Solution |
|---|---|---|
| **New user** | No interaction history | Onboarding survey, popular items, content-based fallback |
| **New item** | No ratings | Content-based filtering using item features |
| **New user + new item** | No signal at all | Non-personalised popular recommendations |

---

## Connections

- [[Recommender Systems]] — the overall system
- [[Collaborative Filtering]] — the retrieval/ranking signal
- [[Matrix Factorization]] — produces item/user embeddings for retrieval
- [[Content Based Filtering]] — cold start fallback

---

## One-line Summary

> The recommendation pipeline retrieves a small candidate set from millions of items quickly via embedding-based ANN search, then re-ranks candidates with a powerful model using rich features — designed for sub-100ms latency at scale, evaluated by ranking metrics like NDCG@K.
