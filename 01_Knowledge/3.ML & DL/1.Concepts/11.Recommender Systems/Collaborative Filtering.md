# Collaborative Filtering

## What is it?

**Collaborative Filtering (CF)** recommends items based on the past behaviour of **many users** — the idea is that users with similar preferences in the past will agree in the future.

> "People who liked what you liked also liked..."

---

## Two Flavours

### User-Based CF
Find users similar to the target user; recommend what they liked.

$$\hat{r}_{ui} = \bar{r}_u + \frac{\sum_{v \in \mathcal{N}(u)}\text{sim}(u,v)(r_{vi} - \bar{r}_v)}{\sum_{v \in \mathcal{N}(u)}|\text{sim}(u,v)|}$$

Where $\mathcal{N}(u)$ = top-$k$ users most similar to $u$, $\bar{r}_u$ = mean rating of user $u$.

**Cosine similarity between users:**
$$\text{sim}(u,v) = \frac{\sum_{i \in I_{uv}} r_{ui} r_{vi}}{\sqrt{\sum_{i \in I_u} r_{ui}^2} \sqrt{\sum_{i \in I_v} r_{vi}^2}}$$

Where $I_{uv}$ = items rated by both $u$ and $v$.

**Pearson correlation (better, accounts for rating bias):**
$$\text{sim}(u,v) = \frac{\sum_{i \in I_{uv}} (r_{ui}-\bar{r}_u)(r_{vi}-\bar{r}_v)}{\sqrt{\sum_{i \in I_{uv}} (r_{ui}-\bar{r}_u)^2} \sqrt{\sum_{i \in I_{uv}} (r_{vi}-\bar{r}_v)^2}}$$

### Item-Based CF
Find items similar to items the user liked; recommend those.

$$\hat{r}_{ui} = \frac{\sum_{j \in \mathcal{N}(i)}\text{sim}(i,j) \cdot r_{uj}}{\sum_{j \in \mathcal{N}(i)}|\text{sim}(i,j)|}$$

Where $\mathcal{N}(i)$ = items most similar to item $i$, $r_{uj}$ = user $u$'s rating of item $j$.

Item similarities are more stable than user similarities (items don't change their "taste").

---

## Limitations of Memory-Based CF

- **Scalability:** Computing pairwise similarities is $O(m^2)$ or $O(n^2)$.
- **Cold start:** New users/items have no ratings history.
- **Sparsity:** Similarity estimates are noisy when few ratings overlap.

Solution: **Matrix Factorization** (model-based CF). See [[Matrix Factorization]].

---

## Connections

- [[Recommender Systems]] — CF is the main approach
- [[Matrix Factorization]] — model-based CF
- [[Content Based Filtering]] — the alternative

---

## One-line Summary

> Collaborative filtering recommends by finding similar users or items in the rating matrix — memory-based methods compute explicit similarities, while model-based methods (matrix factorisation) learn latent factors more scalably.
