# K-Means

## What is it?

**K-Means** is the most widely used [[Clustering]] algorithm. It partitions $n$ data points into exactly $K$ clusters by minimising the total within-cluster variance (sum of squared distances to centroids).

---

## Objective Function

$$J = \sum_{k=1}^K\sum_{i: c^{(i)}=k}\|x^{(i)} - \mu_k\|^2$$

Where:
- $c^{(i)} \in \{1,\ldots,K\}$ = cluster assignment of point $i$
- $\mu_k \in \mathbb{R}^d$ = centroid of cluster $k$

K-Means minimises $J$ jointly over assignments $c$ and centroids $\mu$.

---

## The Algorithm (Lloyd's Algorithm)

**Input:** Data $\{x^{(1)},\ldots,x^{(n)}\}$, number of clusters $K$

1. **Initialise** $K$ centroids $\mu_1, \ldots, \mu_K$ (randomly or via k-means++)
2. **Repeat until convergence:**
   - **Assignment step:** Assign each point to its nearest centroid:
     $$c^{(i)} = \arg\min_k\|x^{(i)} - \mu_k\|^2$$
   - **Update step:** Recompute each centroid as the mean of its assigned points:
     $$\mu_k = \frac{1}{|C_k|}\sum_{i: c^{(i)}=k}x^{(i)}$$
3. **Converged** when assignments don't change between iterations.

---

## Convergence Properties

- **Always converges** in finite steps (finite number of partitions).
- **Not guaranteed to find global optimum** — may converge to a local minimum.
- The objective $J$ never increases between iterations (both steps decrease or maintain $J$).
- Solution depends on initialisation.

**Proof that steps decrease $J$:**
- Assignment step: assigns each point to its nearest centroid → $J$ decreases (or stays same).
- Update step: the mean minimises sum of squared distances within a cluster → $J$ decreases (or stays same).

---

## Initialisation: k-Means++

Random initialisation often gives poor local minima. **k-Means++** (Arthur & Vassilvitskii, 2007) initialises centroids to be spread out:

1. Choose first centroid $\mu_1$ uniformly at random from $\{x^{(1)},\ldots,x^{(n)}\}$.
2. For $k = 2, \ldots, K$:
   - For each point $x^{(i)}$, compute $D(x^{(i)}) = \min_{j<k}\|x^{(i)} - \mu_j\|^2$ (distance to nearest existing centroid).
   - Choose $\mu_k = x^{(i)}$ with probability $\propto D(x^{(i)})$.
   - Points farther from existing centroids are more likely to be chosen.

**Guarantee:** k-Means++ achieves an expected objective value within $O(\log K)$ of the optimal. In practice, dramatically better than random init.

---

## Choosing K: The Elbow Method

Plot the objective $J$ (inertia) as a function of $K$:

```
J
  │
  │╲
  │ ╲
  │  ╲  ← elbow here: K=3
  │   ╲──────────────
  └──────────────── K
    1  2  3  4  5
```

Choose $K$ at the "elbow" — where marginal reduction in $J$ starts diminishing.

**Limitation:** The elbow is often ambiguous. Supplement with Silhouette score.

---

## Assumptions and Limitations

| Assumption | Consequence when violated |
|---|---|
| Clusters are spherical | Elongated or irregular clusters split incorrectly |
| Clusters are similar size | Small clusters absorbed by large ones |
| $K$ is known | Wrong $K$ = poor clustering |
| Features are continuous | Works poorly with categorical features |

**For non-spherical clusters:** Use DBSCAN or spectral clustering.  
**For mixed data types:** Use k-Prototypes (extends k-Means to categorical features).

---

## Complexity

- Per iteration: $O(nKd)$ for assignments + $O(nd)$ for centroid updates.
- Number of iterations: typically $O(100)$ in practice.
- Total: $O(nKdI)$ where $I$ = number of iterations.

For large $n$: use **Mini-Batch K-Means** — update centroids using random mini-batches per iteration. Much faster, slightly worse solution.

---

## Connections

- [[Clustering]] — K-Means is the standard algorithm
- [[Centroid]] — the $\mu_k$ values
- [[Unsupervised Learning]] — no labels required
- [[Feature Scaling]] — K-Means is distance-based; always standardise first
- [[Dimensionality Reduction]] — apply PCA before K-Means in high dimensions

---

## One-line Summary

> K-Means partitions data into $K$ clusters by alternately assigning points to their nearest centroid and recomputing centroids as cluster means — simple, fast, and widely used, but sensitive to initialisation (use k-Means++) and limited to spherical clusters.
