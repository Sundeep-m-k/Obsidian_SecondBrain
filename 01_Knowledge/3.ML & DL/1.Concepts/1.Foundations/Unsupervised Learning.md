## What is it?

**Unsupervised Learning** is a type of [[Machine Learning]] where the algorithm learns patterns from data that has **no labels** — there is no "correct answer" provided. The system must discover structure on its own.

> Contrast with [[Supervised Learning]]: there, you have $(x, y)$ pairs. Here, you only have $\{x^{(1)}, x^{(2)}, \ldots, x^{(n)}\}$.

**Why it matters:** Most data in the world is unlabelled. Labelling is expensive, slow, and requires domain expertise. Unsupervised methods unlock this vast resource.

---

## Formal Setup

**Dataset:**
$$\mathcal{D} = \{x^{(1)}, x^{(2)}, \ldots, x^{(n)}\} \quad x^{(i)} \in \mathbb{R}^d$$

**Goal:** Discover hidden structure — groups, dimensions, densities, or generative factors — in this data.

There is no explicit loss function with ground truth. Objectives are defined differently for each task.

---

## The Main Tasks

| Task | Goal | Output |
|---|---|---|
| **Clustering** | Group similar examples | Cluster assignment per example |
| **Dimensionality Reduction** | Compress features, preserve structure | Lower-dimensional representation |
| **Density Estimation** | Learn the distribution $p(x)$ | Probability model |
| **Anomaly Detection** | Find unusual examples | Anomaly score per example |
| **Representation Learning** | Learn useful features | Embeddings/latent vectors |
| **Generative Modelling** | Learn to produce new samples | New synthetic data |

---

## Clustering

### k-Means Clustering

**Goal:** Partition $n$ points into $k$ clusters, minimising within-cluster variance.

**Objective (minimise):**
$$J = \sum_{j=1}^{k} \sum_{x \in C_j} \|x - \mu_j\|^2$$

where $\mu_j$ is the **centroid** (mean) of cluster $C_j$.

**Algorithm (Lloyd's algorithm):**
1. Initialise $k$ centroids $\mu_1, \ldots, \mu_k$ (randomly or with k-means++)
2. **Assignment step:** assign each point to its nearest centroid
$$c^{(i)} = \arg\min_j \|x^{(i)} - \mu_j\|^2$$
3. **Update step:** recompute centroids
$$\mu_j = \frac{1}{|C_j|} \sum_{i: c^{(i)}=j} x^{(i)}$$
4. Repeat steps 2–3 until convergence (assignments don't change)

**Guarantees:** Always converges, but may converge to a local minimum (not global). Run multiple times with different initialisations.

**How to choose $k$:**
- **Elbow method:** Plot $J$ vs. $k$. Look for the "elbow" where adding another cluster gives diminishing returns.
- **Silhouette score:** Measures cohesion vs. separation; ranges from $-1$ to $1$, higher is better.

$$s(i) = \frac{b(i) - a(i)}{\max(a(i), b(i))}$$

where $a(i)$ = mean distance to points in same cluster, $b(i)$ = mean distance to nearest other cluster.

**Weaknesses:** Assumes spherical, equal-size clusters. Sensitive to scale (always standardise features first).

---

### Hierarchical Clustering

Builds a **dendrogram** — a tree showing how clusters merge (agglomerative) or split (divisive).

**Agglomerative (bottom-up):**
1. Start: each point is its own cluster.
2. Merge the two closest clusters.
3. Repeat until all points are in one cluster.

**Linkage criteria** (how to measure distance between clusters):

| Linkage | Distance between clusters A, B |
|---|---|
| Single | $\min_{a \in A, b \in B} d(a,b)$ |
| Complete | $\max_{a \in A, b \in B} d(a,b)$ |
| Average | $\frac{1}{|A||B|}\sum_{a \in A}\sum_{b \in B} d(a,b)$ |
| Ward | Minimises increase in total within-cluster variance |

Cut the dendrogram at a desired level to get any number of clusters.

---

### DBSCAN (Density-Based Spatial Clustering)

Finds clusters as **dense regions** separated by sparse regions.

**Parameters:** $\epsilon$ (neighbourhood radius), $\text{minPts}$ (minimum points to form a dense region).

**Point types:**
- **Core point:** has $\geq \text{minPts}$ points within radius $\epsilon$
- **Border point:** within $\epsilon$ of a core point but not itself core
- **Noise point (outlier):** neither core nor border

**Strengths:** Finds arbitrarily shaped clusters, automatically identifies outliers, no need to specify $k$.  
**Weaknesses:** Struggles with varying density, sensitive to $\epsilon$.

---

## Dimensionality Reduction

### Principal Component Analysis (PCA)

**Goal:** Find a lower-dimensional linear subspace that preserves maximum variance.

**Intuition:** Project high-dimensional data onto the directions of greatest spread.

**Steps:**
1. Standardise features (zero mean, unit variance)
2. Compute covariance matrix: $\Sigma = \frac{1}{n} X^T X \in \mathbb{R}^{d \times d}$
3. Compute eigendecomposition: $\Sigma = V \Lambda V^T$
   - Columns of $V$ = eigenvectors (principal components)
   - Diagonal of $\Lambda$ = eigenvalues $\lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_d$
4. Select top $k$ eigenvectors: $V_k \in \mathbb{R}^{d \times k}$
5. Project: $Z = XV_k \in \mathbb{R}^{n \times k}$

**Explained Variance Ratio:**
$$\text{EVR}_j = \frac{\lambda_j}{\sum_{i=1}^{d} \lambda_i}$$

Choose $k$ such that the cumulative EVR $\geq$ 90–95%.

**What PCA does NOT do:**
- It does not consider labels (it's unsupervised)
- It finds directions of variance, not necessarily directions of class separation (that's LDA)
- It's linear — cannot capture non-linear manifolds

**Reconstruction:**
$$\hat{x} = Z V_k^T$$

**Reconstruction error:**
$$\|x - \hat{x}\|^2 = \sum_{j=k+1}^{d} \lambda_j$$

(Sum of discarded eigenvalues)

---

### t-SNE (t-Distributed Stochastic Neighbour Embedding)

**Goal:** Visualisation — non-linear dimensionality reduction to 2D or 3D while preserving **local structure**.

**How it works:**
1. Compute pairwise similarities in high-dimensional space using a Gaussian:
$$p_{j|i} = \frac{\exp(-\|x_i - x_j\|^2 / 2\sigma_i^2)}{\sum_{k \neq i}\exp(-\|x_i - x_k\|^2 / 2\sigma_i^2)}$$
2. Compute pairwise similarities in low-dimensional space using a Student-t distribution (heavier tails):
$$q_{ij} = \frac{(1 + \|z_i - z_j\|^2)^{-1}}{\sum_{k \neq l}(1 + \|z_k - z_l\|^2)^{-1}}$$
3. Minimise KL divergence: $KL(P \| Q) = \sum_{i \neq j} p_{ij} \log \frac{p_{ij}}{q_{ij}}$

**Key parameter:** `perplexity` ≈ effective number of neighbours (typically 5–50).

**Important caveat:** t-SNE is for **visualisation only**. Distances between clusters in the 2D plot are not meaningful. Do not use t-SNE representations as features.

---

### Autoencoders

A neural network trained to **reconstruct its own input** through a bottleneck.

```
Input x → [Encoder] → Latent z → [Decoder] → Reconstructed x̂
```

$$z = f_\theta(x) \quad \text{(encoder)}$$
$$\hat{x} = g_\phi(z) \quad \text{(decoder)}$$

**Loss:**
$$\mathcal{L} = \|x - \hat{x}\|^2 = \|x - g_\phi(f_\theta(x))\|^2$$

The bottleneck forces the encoder to learn a **compact representation** $z$ capturing the most important structure.

**Variants:**
- **Sparse autoencoder:** penalises activations in $z$, learns sparse codes
- **Denoising autoencoder:** reconstructs clean $x$ from corrupted input — more robust
- **Variational Autoencoder (VAE):** encodes to a distribution $q(z|x) = \mathcal{N}(\mu, \sigma^2)$; allows generation of new samples

**VAE loss (ELBO):**
$$\mathcal{L}_{\text{VAE}} = \underbrace{\mathbb{E}_{q(z|x)}[\log p(x|z)]}_{\text{reconstruction}} - \underbrace{KL(q(z|x) \| p(z))}_{\text{regularisation}}$$

---

## Density Estimation

### Gaussian Mixture Models (GMM)

Assume data is generated from $K$ Gaussian distributions:

$$p(x) = \sum_{k=1}^{K} \pi_k \mathcal{N}(x \mid \mu_k, \Sigma_k)$$

where $\pi_k$ = mixing weight ($\sum_k \pi_k = 1$), $\mu_k$ = mean, $\Sigma_k$ = covariance.

**Trained with the Expectation-Maximisation (EM) algorithm:**

- **E-step:** Compute "soft assignments" — probability that example $i$ belongs to cluster $k$:
$$r_{ik} = \frac{\pi_k \mathcal{N}(x^{(i)} \mid \mu_k, \Sigma_k)}{\sum_j \pi_j \mathcal{N}(x^{(i)} \mid \mu_j, \Sigma_j)}$$

- **M-step:** Update parameters using the soft assignments:
$$\pi_k = \frac{1}{n}\sum_i r_{ik}, \quad \mu_k = \frac{\sum_i r_{ik} x^{(i)}}{\sum_i r_{ik}}, \quad \Sigma_k = \frac{\sum_i r_{ik}(x^{(i)}-\mu_k)(x^{(i)}-\mu_k)^T}{\sum_i r_{ik}}$$

k-Means is a special case of GMM with hard assignments and spherical equal-variance clusters.

---

## Self-Supervised Learning

A powerful modern approach: **create labels automatically from the data itself**.

Examples:
- **Language models:** predict the next word. Input = preceding context, label = next word. No human annotation needed.
- **Masked modelling (BERT):** mask 15% of tokens, predict them.
- **Contrastive learning (SimCLR, MoCo):** two augmented views of the same image should have similar representations; views of different images should differ.

**Contrastive loss (InfoNCE):**
$$\mathcal{L} = -\log \frac{\exp(\text{sim}(z_i, z_j)/\tau)}{\sum_{k=1}^{2N} \mathbf{1}_{k \neq i} \exp(\text{sim}(z_i, z_k)/\tau)}$$

where $\text{sim}(u,v) = \frac{u^T v}{\|u\|\|v\|}$ is cosine similarity, $\tau$ is a temperature parameter.

Self-supervised learning powers modern LLMs, CLIP, and other foundation models.

---

## Evaluation (The Hard Part)

Evaluating unsupervised learning is fundamentally difficult because there's no ground truth label to compare against.

| Method | What it measures | Notes |
|---|---|---|
| Silhouette score | Cohesion and separation of clusters | Higher is better; no labels needed |
| Davies-Bouldin index | Average ratio of within-cluster to between-cluster distances | Lower is better |
| Reconstruction error | How well an autoencoder recovers input | Proxy for representation quality |
| Downstream task performance | Use learned representations as features for supervised task | Gold standard for representation learning |
| Visual inspection (t-SNE/UMAP) | Qualitative structure | Useful but subjective |

---

## Connections

- [[Machine Learning]] — parent framework
- [[Supervised Learning]] — the labelled contrast
- [[Features]] — what unsupervised methods operate on
- [[Training Data]] — unlabelled data used
- [[Model]] — the structure learned (cluster assignments, embeddings, density)
- [[Generalization]] — do the discovered structures hold on new data?

---

## One-line Summary

> Unsupervised learning finds hidden structure — groups, dimensions, distributions — in data that comes with no labels, letting the patterns in the data speak for themselves.
