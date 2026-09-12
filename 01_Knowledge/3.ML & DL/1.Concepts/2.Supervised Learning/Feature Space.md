# Feature Space

## What is it?

The **feature space** (also called **input space**) is the mathematical space in which all data points live. Each data point is represented as a vector of $d$ features, so the feature space is $\mathbb{R}^d$ (or a subset of it).

$$x \in \mathcal{X} \subseteq \mathbb{R}^d$$

Every data point is a single **point** in this $d$-dimensional space.

---

## The Geometry

With $d$ features:
- $d = 1$: feature space is a line.
- $d = 2$: feature space is a 2D plane — easy to visualise.
- $d = 3$: 3D space.
- $d \gg 3$: high-dimensional space, impossible to visualise directly.

**Each axis** of the space corresponds to one feature. The position of a data point along axis $j$ is its value for feature $j$.

**Example:** For the iris dataset with features [sepal length, sepal width, petal length, petal width], the feature space is $\mathbb{R}^4$. Each flower is a point in 4D space.

---

## Why Feature Space Matters

1. **Model complexity:** The model learns a function over feature space. A [[Decision Boundary]] is a surface in this space.

2. **Distances:** Many algorithms (kNN, SVM, clustering) measure distances between points in feature space:
$$d(x, x') = \sqrt{\sum_{j=1}^{d}(x_j - x'_j)^2}$$

3. **Scale sensitivity:** If features have very different scales (e.g., age in years vs. salary in thousands), the distance metric is dominated by the larger-scaled feature. Always **scale features** before distance-based methods.

4. **Curse of dimensionality:** As $d$ increases, points become sparse. The volume of the space grows exponentially. See [[Features]] for the full treatment.

---

## Feature Space Transformations

You can **map** from the original feature space to a new space:
$$\phi: \mathbb{R}^d \to \mathbb{R}^{d'}$$

Reasons:
- Make non-linearly separable data linearly separable in the higher-dimensional space.
- Add polynomial features: $[x_1, x_2] \to [x_1, x_2, x_1^2, x_2^2, x_1 x_2]$
- Reduce dimensionality (PCA) for efficiency or visualisation.

**Kernel trick:** Computes inner products in the transformed space $\phi(x)^T\phi(x')$ without explicitly computing $\phi(x)$, making high-dimensional (even infinite-dimensional) transformations feasible.

---

## Connections

- [[Features]] — each dimension of the space
- [[Decision Boundary]] — a surface in this space
- [[Model]] — learns a function over this space
- [[Supervised Learning]] — maps feature space to output space

---

## One-line Summary

> Feature space is the $d$-dimensional mathematical space where every data point lives as a vector — the geometry of this space (distances, orientations, boundaries) is what ML algorithms operate on.
