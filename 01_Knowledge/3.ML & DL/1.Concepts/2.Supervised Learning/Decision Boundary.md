# Decision Boundary

## What is it?

A **decision boundary** is the surface in [[Feature Space]] that separates regions predicted as different classes. It is the set of all input points $x$ where the model is exactly indifferent between two (or more) classes.

For a binary classifier with threshold $\tau = 0.5$:
$$\text{Decision boundary} = \{x : \hat{p}(Y=1 \mid x) = 0.5\}$$

---

## Formal Definition

For a binary classifier $\hat{f}: \mathbb{R}^d \to \mathbb{R}$ (outputting a score):

$$\text{Decision boundary} = \{x \in \mathbb{R}^d : \hat{f}(x) = 0\}$$

Points with $\hat{f}(x) > 0$: predict class 1.  
Points with $\hat{f}(x) < 0$: predict class 0.

For multi-class, the boundary between class $j$ and $k$ is:
$$\{x : P(Y=j \mid x) = P(Y=k \mid x)\}$$

---

## Shape of Decision Boundaries by Model

### Linear Models (Logistic Regression, Linear SVM)

$$\hat{f}(x) = \theta^T x + b = 0$$

The boundary is a **hyperplane** — a flat, $(d-1)$-dimensional surface.

- In 2D ($d=2$): a **line** $\theta_1 x_1 + \theta_2 x_2 + b = 0$
- In 3D ($d=3$): a **plane**
- In $d$ dimensions: a **hyperplane**

**Normal vector:** $\theta$ is perpendicular to the decision boundary.  
**Distance** from a point $x$ to the boundary:
$$\text{distance} = \frac{|\theta^T x + b|}{\|\theta\|}$$

This is exactly what the SVM maximises — the **margin** is $\frac{2}{\|\theta\|}$.

### Polynomial / Kernel Models

With a degree-$p$ polynomial expansion of features, the boundary is a degree-$p$ curve in the original space but a hyperplane in the expanded feature space.

Example: quadratic features in 2D give ellipses, parabolas, or hyperbolas as boundaries.

### Decision Trees

The boundary is a set of **axis-aligned hyperplanes** — splits are always of the form $x_j \leq c$ for some feature $j$ and threshold $c$.

This creates rectangular decision regions, regardless of the shape of the true boundary.

### Neural Networks

With non-linear activations and enough capacity, the boundary can be **arbitrarily shaped** — curves, disconnected regions, intricate manifolds.

### k-Nearest Neighbours

The boundary is a **Voronoi diagram** — piecewise linear with edges equidistant between training examples of different classes. The boundary is highly non-linear for small $k$.

---

## Linear Separability

A dataset is **linearly separable** if there exists a hyperplane that perfectly separates all examples of one class from all examples of the other class.

Formally: $\exists\ \theta, b$ such that:
$$\theta^T x^{(i)} + b > 0 \quad \forall i: y^{(i)} = 1$$
$$\theta^T x^{(i)} + b < 0 \quad \forall i: y^{(i)} = 0$$

Most real datasets are **not** linearly separable in the original feature space, which is why:
1. Non-linear models (trees, NNs) are often used.
2. Features are transformed (polynomial, kernel) to make data linearly separable in a higher-dimensional space.

**XOR is the classic non-linearly separable example:**
```
  x₂
  1 │ ○  ×         ○ = class 0
    │               × = class 1
  0 │ ×  ○
    └─────── x₁
      0   1
```
No single line can separate ○ from ×.

---

## Margin

For a linear classifier with decision boundary $\theta^T x + b = 0$, the **margin** is the distance from the boundary to the nearest training example.

$$\text{Margin} = \min_i \frac{y^{(i)}(\theta^T x^{(i)} + b)}{\|\theta\|}$$

**SVM** maximises this margin:
- Boundary: $\theta^T x + b = 0$
- Support vectors (positive): $\theta^T x + b = +1$
- Support vectors (negative): $\theta^T x + b = -1$
- Margin width: $\frac{2}{\|\theta\|}$

A larger margin → better [[Generalization]] (the model is less sensitive to small changes in data).

---

## Visualising Decision Boundaries

For 2D feature spaces, you can visualise by:
1. Create a fine grid over the feature space.
2. Evaluate the model at every grid point.
3. Colour each point by its predicted class.
4. Plot training examples on top.

This reveals the shape of the boundary directly.

For high-dimensional spaces, use dimensionality reduction (PCA, t-SNE) to project to 2D, then visualise — though this may distort the boundary.

---

## Connections

- [[Feature Space]] — the space in which the boundary lives
- [[Logistic Regression]] — linear boundary
- [[Binary Classification]] — boundary separates 2 classes
- [[Multiclass Classification]] — multiple pairwise boundaries
- [[Model]] — the model defines the boundary shape
- [[Supervised Learning]] — learning places the boundary to separate classes in training data

---

## One-line Summary

> A decision boundary is the surface in feature space where the model transitions from predicting one class to another — its shape (line, curve, or arbitrary manifold) is determined by the model type and encodes everything the model has learned about where the classes differ.
