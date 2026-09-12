# Anomaly Detection

## What is it?

**Anomaly Detection** (also called **outlier detection**) identifies data points that are significantly different from the majority of the data — observations that don't conform to expected patterns.

It is primarily an [[Unsupervised Learning]] task since anomalies are rare and often unlabelled.

$$\text{anomaly score}(x) \text{ is high} \Leftrightarrow x \text{ is unusual}$$

---

## Formal Setup

Given dataset $\{x^{(1)},\ldots,x^{(n)}\}$ drawn mostly from a "normal" distribution $p_{\text{normal}}(x)$, flag point $x_{\text{new}}$ as an anomaly if:

$$p(x_{\text{new}}) < \epsilon$$

for some threshold $\epsilon$ (chosen to control the false positive rate).

---

## Density Estimation Approach (Gaussian Model)

Model each feature as independently Gaussian (Naive Bayes assumption):

$$p(x) = \prod_{j=1}^d p(x_j) = \prod_{j=1}^d \frac{1}{\sqrt{2\pi}\sigma_j}\exp\!\left(-\frac{(x_j - \mu_j)^2}{2\sigma_j^2}\right)$$

**Training (estimate parameters from normal data):**
$$\mu_j = \frac{1}{n}\sum_{i=1}^n x_j^{(i)}, \qquad \sigma_j^2 = \frac{1}{n}\sum_{i=1}^n(x_j^{(i)} - \mu_j)^2$$

**Detection:**
$$p(x_{\text{new}}) = \prod_{j=1}^d \frac{1}{\sqrt{2\pi}\sigma_j}\exp\!\left(-\frac{(x_{\text{new},j} - \mu_j)^2}{2\sigma_j^2}\right) < \epsilon \Rightarrow \text{anomaly}$$

**Threshold $\epsilon$ selection:** Use a small labelled validation set of known anomalies and normals; choose $\epsilon$ that maximises $F_1$ score.

---

## Multivariate Gaussian

Captures correlations between features:

$$p(x) = \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}\exp\!\left(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\right)$$

**Parameters:**
$$\mu = \frac{1}{n}\sum_{i=1}^n x^{(i)} \in \mathbb{R}^d$$
$$\Sigma = \frac{1}{n}\sum_{i=1}^n(x^{(i)}-\mu)(x^{(i)}-\mu)^T \in \mathbb{R}^{d\times d}$$

**Anomaly score:** Mahalanobis distance $(x-\mu)^T\Sigma^{-1}(x-\mu)$.

Advantage: captures unusual combinations of features, not just unusual individual features.  
Disadvantage: requires $n \gg d$ for $\Sigma$ to be invertible.

---

## Other Methods

| Method | Idea |
|---|---|
| **Isolation Forest** | Anomalies are isolated in fewer random splits |
| **One-Class SVM** | Fits a boundary around normal data |
| **Autoencoder** | High reconstruction error = anomaly |
| **DBSCAN** | Points not in any cluster = anomalies (noise points) |
| **LOF (Local Outlier Factor)** | Compares local density to neighbours |

**Isolation Forest:**
- Build random trees by repeatedly splitting on random features/thresholds.
- Anomalies reach leaf nodes in fewer splits (they're more isolated).
- Anomaly score = $2^{-\text{avg path length to leaf}}$.

---

## Anomaly Detection vs. Supervised Classification

| | Anomaly Detection | Supervised Classification |
|---|---|---|
| **Labels** | Mostly unlabelled | All labelled |
| **Anomaly frequency** | Very rare (0.01%–1%) | Any distribution |
| **When to use** | Very few known anomaly examples | Many examples of both classes |
| **Algorithm** | Density estimation, Isolation Forest | Logistic regression, SVM |

Use anomaly detection when you have abundant normal examples but very few (or no) labelled anomalies.

---

## Applications

| Domain | Example |
|---|---|
| Manufacturing | Defective parts detection |
| Finance | Fraudulent transactions |
| Cybersecurity | Intrusion/malware detection |
| Healthcare | Rare disease flagging |
| Infrastructure | Server failure prediction |

---

## Connections

- [[Density Estimation]] — anomaly detection via $p(x) < \epsilon$
- [[Clustering]] — DBSCAN identifies anomalies as noise points
- [[Unsupervised Learning]] — typically no labels available

---

## One-line Summary

> Anomaly detection identifies data points with unusually low probability density under the normal data distribution — using Gaussian models, Isolation Forests, or autoencoders — applied wherever rare, unlabelled deviations from normal behaviour must be flagged.
