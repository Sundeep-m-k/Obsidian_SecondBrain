# Clustering

## What is it?

**Clustering** is an [[Unsupervised Learning]] task that groups data points into **clusters** — subsets where points within a cluster are more similar to each other than to points in other clusters.

No labels are provided. The algorithm discovers the grouping structure entirely from the data's geometry.

$$\mathcal{D} = \{x^{(1)}, \ldots, x^{(n)}\} \xrightarrow{\text{clustering}} \{C_1, C_2, \ldots, C_K\}$$

---

## Types of Clustering

| Type | Description | Example algorithms |
|---|---|---|
| **Partitioning** | Divide into $K$ non-overlapping clusters | k-Means, k-Medoids |
| **Hierarchical** | Build a tree of nested clusters | Agglomerative, DIANA |
| **Density-based** | Find dense regions separated by sparse regions | DBSCAN, HDBSCAN |
| **Probabilistic** | Soft assignments via probability model | GMM (EM algorithm) |
| **Spectral** | Use eigenvectors of the similarity graph | Spectral Clustering |

---

## Distance / Similarity Measures

Clustering algorithms require a notion of how similar two points are.

**Euclidean distance** (most common):
$$d(x, x') = \|x - x'\|_2 = \sqrt{\sum_{j=1}^d(x_j - x'_j)^2}$$

**Manhattan distance (L1):**
$$d(x, x') = \|x - x'\|_1 = \sum_{j=1}^d|x_j - x'_j|$$

**Cosine similarity** (for text/high-dimensional):
$$\text{sim}(x, x') = \frac{x \cdot x'}{\|x\|\|x'\|} = \cos\theta \in [-1, 1]$$

**Mahalanobis distance** (accounts for feature correlations):
$$d(x, x') = \sqrt{(x-x')^T\Sigma^{-1}(x-x')}$$

---

## Cluster Quality Metrics (No Labels Needed)

### Silhouette Score
For data point $i$:
$$s(i) = \frac{b(i) - a(i)}{\max(a(i), b(i))}$$

Where:
- $a(i)$ = mean distance from $i$ to all other points in its cluster (intra-cluster)
- $b(i)$ = mean distance from $i$ to all points in the nearest other cluster (inter-cluster)

Range: $[-1, 1]$.  
$s(i) \approx 1$: well-clustered.  
$s(i) \approx 0$: on boundary.  
$s(i) < 0$: possibly in wrong cluster.

**Overall silhouette score:** $\bar{s} = \frac{1}{n}\sum_i s(i)$.

### Davies-Bouldin Index
$$DB = \frac{1}{K}\sum_{k=1}^K \max_{j\neq k}\frac{\sigma_k + \sigma_j}{d(\mu_k, \mu_j)}$$

Where $\sigma_k$ = mean distance of cluster $k$'s points to centroid $\mu_k$, $d(\mu_k,\mu_j)$ = distance between centroids.

Lower = better. Measures compact and well-separated clusters.

### Calinski-Harabasz Index (Variance Ratio)
$$CH = \frac{\text{Between-cluster variance}}{\text{Within-cluster variance}} \times \frac{n-K}{K-1}$$

Higher = better.

---

## Cluster Quality Metrics (With Ground Truth Labels)

When true labels are available (evaluation only, not training):

**Adjusted Rand Index (ARI):**
$$ARI = \frac{\text{RI} - \mathbb{E}[\text{RI}]}{\max(\text{RI}) - \mathbb{E}[\text{RI}]}$$

Range $[-1, 1]$. 1 = perfect match. 0 = random. Adjusted for chance.

**Normalised Mutual Information (NMI):**
$$NMI = \frac{2 \cdot I(Y; \hat{Y})}{H(Y) + H(\hat{Y})}$$

Where $I$ is mutual information and $H$ is entropy. Range $[0, 1]$.

---

## Applications of Clustering

| Domain | Use case |
|---|---|
| Marketing | Customer segmentation |
| Biology | Gene expression grouping, species discovery |
| Computer Vision | Image compression (colour quantisation) |
| NLP | Topic modelling, document grouping |
| Anomaly Detection | Points that don't fit any cluster = outliers |
| Recommendation | Group similar users/items |

---

## Connections

- [[K Means]] — the most common partitioning algorithm
- [[Centroid]] — the representative point of each cluster
- [[Density Estimation]] — GMM models cluster as densities
- [[Dimensionality Reduction]] — often applied before clustering to reduce noise
- [[Unsupervised Learning]] — clustering is an unsupervised task

---

## One-line Summary

> Clustering discovers natural groupings in unlabelled data by assigning similar points to the same cluster, measured by distance or density — the number and shape of clusters are learned from data, not specified by labels.
