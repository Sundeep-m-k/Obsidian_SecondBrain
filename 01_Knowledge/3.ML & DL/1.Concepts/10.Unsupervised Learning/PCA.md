# PCA (Principal Component Analysis)

## What is it?

**PCA** is the most widely used linear [[Dimensionality Reduction]] technique. It finds the orthogonal directions of **maximum variance** in the data and projects data onto the top-$k$ of these directions.

**Goal:** Find a $k$-dimensional subspace that captures as much of the data's variance as possible, minimising information loss.

---

## Intuition

Imagine a cloud of points shaped like a flat ellipse. The long axis of the ellipse (direction of most spread) is the first principal component. The short axis (second most spread, perpendicular to first) is the second. PCA rotates the coordinate system to align with these axes of spread.

---

## The Full Algorithm

### Step 1: Centre the data

$$\bar{x} = \frac{1}{n}\sum_{i=1}^n x^{(i)}$$

$$\tilde{x}^{(i)} = x^{(i)} - \bar{x}$$

**Always centre before PCA.** Without centring, the first PC is dominated by the mean direction, not the direction of spread.

### Step 2: Compute the covariance matrix

$$\Sigma = \frac{1}{n}\tilde{X}^T\tilde{X} = \frac{1}{n}\sum_{i=1}^n \tilde{x}^{(i)}(\tilde{x}^{(i)})^T \in \mathbb{R}^{d\times d}$$

The $(j,k)$ entry: $\Sigma_{jk} = \frac{1}{n}\sum_i\tilde{x}_j^{(i)}\tilde{x}_k^{(i)}$ = covariance between features $j$ and $k$.

### Step 3: Eigendecomposition of $\Sigma$

$$\Sigma = V\Lambda V^T$$

Where:
- $V = [v_1, v_2, \ldots, v_d] \in \mathbb{R}^{d\times d}$ — columns are orthonormal eigenvectors (principal components)
- $\Lambda = \text{diag}(\lambda_1, \lambda_2, \ldots, \lambda_d)$ — eigenvalues, sorted $\lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_d \geq 0$
- $v_k$ = the $k$-th principal component direction
- $\lambda_k$ = variance of data projected onto $v_k$

### Step 4: Select top $k$ components

$$V_k = [v_1, v_2, \ldots, v_k] \in \mathbb{R}^{d\times k}$$

### Step 5: Project data

$$Z = \tilde{X}V_k \in \mathbb{R}^{n\times k}$$

Row $i$ of $Z$: $z^{(i)} = V_k^T \tilde{x}^{(i)} \in \mathbb{R}^k$ — the low-dimensional representation of example $i$.

---

## SVD Connection

For large $d$, computing $\Sigma = \frac{1}{n}\tilde{X}^T\tilde{X}$ and its eigendecomposition is expensive ($O(d^3)$). Instead use the **SVD** of $\tilde{X}$:

$$\tilde{X} = U S V^T$$

Where $U \in \mathbb{R}^{n\times n}$, $S \in \mathbb{R}^{n\times d}$ (diagonal), $V \in \mathbb{R}^{d\times d}$.

The right singular vectors $V$ = eigenvectors of $\tilde{X}^T\tilde{X}$. The singular values $\sigma_k = \sqrt{n\lambda_k}$.

SVD is numerically more stable and works for $n < d$ (more features than examples).

---

## Explained Variance Ratio

**Variance explained by component $k$:**
$$\text{EVR}_k = \frac{\lambda_k}{\sum_{j=1}^d \lambda_j}$$

**Cumulative variance explained by top $k$ components:**
$$\text{Cumulative EVR}(k) = \frac{\sum_{j=1}^k \lambda_j}{\sum_{j=1}^d \lambda_j}$$

**Choose $k$** such that cumulative EVR $\geq$ 90–95%.

**Scree plot:** Plot $\lambda_k$ vs. $k$. The "elbow" where eigenvalues flatten is a natural choice for $k$.

---

## Reconstruction

Project back from $k$-dimensional space to original:
$$\hat{x}^{(i)} = V_k z^{(i)} + \bar{x} = V_k V_k^T \tilde{x}^{(i)} + \bar{x}$$

**Reconstruction error:**
$$\|x^{(i)} - \hat{x}^{(i)}\|^2 = \|\tilde{x}^{(i)} - V_k V_k^T \tilde{x}^{(i)}\|^2 = \sum_{j=k+1}^d (v_j^T \tilde{x}^{(i)})^2$$

Total reconstruction error over all points = $\sum_{j=k+1}^d \lambda_j$ (sum of discarded eigenvalues).

**PCA minimises reconstruction error** among all linear projections to $k$ dimensions. This is the Eckart-Young theorem.

---

## Whitening (PCA Whitening)

After projecting, normalise each component to unit variance:
$$\tilde{z}_k = \frac{z_k}{\sqrt{\lambda_k}}$$

Whitened data has identity covariance matrix: $\text{Cov}(\tilde{z}) = I$.

Useful preprocessing for algorithms that assume uncorrelated, unit-variance features.

---

## When to Apply PCA

✅ High-dimensional data ($d$ is large), want to reduce computational cost  
✅ Visualisation (reduce to 2D or 3D)  
✅ Remove noise (discard low-variance components that carry mainly noise)  
✅ Remove multicollinearity (PCA components are uncorrelated by construction)  
✅ Preprocessing for clustering or classification  

❌ When interpretability is important (PCA components are linear combinations of all original features — hard to interpret)  
❌ When important variation is not captured by variance (e.g., rare but important features)  
❌ Non-linear structure (use UMAP, t-SNE, autoencoders instead)  

---

## PCA Is Not Feature Selection

PCA creates **new features** (linear combinations of originals). It does not select a subset of original features. If you need original features, use feature selection (L1 regularisation, mutual information, etc.).

---

## Connections

- [[Dimensionality Reduction]] — PCA is the canonical method
- [[Feature Engineering]] — PCA produces new features
- [[Clustering]] — apply PCA before K-Means in high dimensions
- [[Unsupervised Learning]] — PCA requires no labels
- [[Variance]] — PCA maximises explained variance

---

## One-line Summary

> PCA finds the directions of maximum variance in centred data via eigendecomposition of the covariance matrix, projects data onto the top-$k$ eigenvectors, and is the optimal linear compression method in terms of minimising reconstruction error — chosen $k$ based on the cumulative explained variance ratio.
