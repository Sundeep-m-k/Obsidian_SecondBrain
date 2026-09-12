# Centroid

## What is it?

A **centroid** is the **mean position** of all data points assigned to a cluster. It is the representative or "centre of mass" of a cluster.

$$\mu_k = \frac{1}{|C_k|}\sum_{i: c^{(i)}=k}x^{(i)}$$

Where $|C_k|$ = number of points in cluster $k$.

---

## In K-Means

The centroid is the point that **minimises the sum of squared distances** to all cluster members:

$$\mu_k = \arg\min_{\mu}\sum_{i: c^{(i)}=k}\|x^{(i)} - \mu\|^2$$

**Proof:** Take the derivative with respect to $\mu$, set to zero:
$$\frac{\partial}{\partial\mu}\sum_{i: c^{(i)}=k}\|x^{(i)} - \mu\|^2 = -2\sum_{i: c^{(i)}=k}(x^{(i)} - \mu) = 0$$
$$\Rightarrow \mu_k = \frac{1}{|C_k|}\sum_{i: c^{(i)}=k}x^{(i)}$$

The mean is the unique optimal representative.

---

## Centroid vs. Medoid

| | Centroid | Medoid |
|---|---|---|
| **What** | Mean of cluster members | Actual data point closest to cluster centre |
| **Computation** | Always has closed form | Requires scanning all points in cluster |
| **Robust to outliers?** | No (mean is sensitive) | Yes (medoid is an actual data point) |
| **Algorithm** | K-Means | K-Medoids (PAM) |

---

## Connections

- [[K Means]] — centroids are the core component
- [[Clustering]] — centroids represent clusters

---

## One-line Summary

> A centroid is the mean position of all points in a cluster — the unique point minimising the sum of squared distances to those points, recomputed at every iteration of K-Means.
