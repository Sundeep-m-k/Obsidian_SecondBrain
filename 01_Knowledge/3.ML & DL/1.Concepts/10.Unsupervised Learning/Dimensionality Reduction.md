# Dimensionality Reduction

## What is it?

**Dimensionality Reduction** transforms data from a high-dimensional space $\mathbb{R}^d$ to a lower-dimensional space $\mathbb{R}^k$ (where $k \ll d$), preserving as much useful information as possible.

$$x \in \mathbb{R}^d \xrightarrow{\text{dim. reduction}} z \in \mathbb{R}^k, \quad k \ll d$$

---

## Why Reduce Dimensions?

| Reason | Detail |
|---|---|
| **Visualisation** | Project to 2D/3D for human interpretation |
| **Compression** | Fewer dimensions = less storage and computation |
| **Noise reduction** | Remove dimensions that carry only noise |
| **Curse of dimensionality** | Many algorithms degrade in high dimensions |
| **Feature learning** | Discover latent structure not obvious in raw features |

---

## Linear Methods

### PCA (Principal Component Analysis)
See [[PCA]] for full treatment.
- Finds orthogonal directions of maximum variance.
- Best linear compression in terms of reconstruction error.
- Assumes the important structure is captured by variance.

### Linear Discriminant Analysis (LDA)
- Supervised: finds directions that maximise class separation.
- Maximises: $\frac{w^TS_B w}{w^TS_W w}$ where $S_B$ = between-class scatter, $S_W$ = within-class scatter.
- At most $K-1$ dimensions for $K$ classes.

### Factor Analysis
- Assumes $x = \mu + Wz + \epsilon$ where $z \in \mathbb{R}^k$ is a latent factor and $\epsilon$ is noise.
- Similar to PCA but explicitly models noise.

---

## Non-Linear Methods

### t-SNE (t-Distributed Stochastic Neighbour Embedding)
- Maps to 2D/3D preserving local neighbourhood structure.
- Uses heavy-tailed Student-t kernel in low-dimensional space to avoid crowding.
- Great for visualisation, not for downstream tasks.
- Stochastic — different runs give different results.

### UMAP (Uniform Manifold Approximation and Projection)
- Preserves both local and global structure better than t-SNE.
- Faster than t-SNE for large datasets.
- Deterministic (with fixed random seed).
- Can be used for dimensionality reduction as a preprocessing step.

### Autoencoders
See [[Unsupervised Learning]].
- Neural network: encoder compresses to latent $z$, decoder reconstructs $x$.
- Non-linear compression. Very flexible.
- VAE adds a probabilistic structure to the latent space.

---

## Evaluation

| Method | Metric |
|---|---|
| PCA | Explained variance ratio (EVR) |
| Any | Reconstruction error $\|x - \hat{x}\|^2$ |
| Any | Downstream task performance with reduced features |
| Visualisation methods | Subjective visual inspection |

---

## Connections

- [[PCA]] — the canonical linear dimensionality reduction
- [[Clustering]] — clustering is often applied after dimensionality reduction
- [[Feature Engineering]] — dimensionality reduction is a form of feature transformation
- [[Unsupervised Learning]] — dimensionality reduction is unsupervised

---

## One-line Summary

> Dimensionality reduction compresses high-dimensional data into a lower-dimensional representation while preserving structure — used for visualisation, denoising, and making downstream ML algorithms faster and more effective.
