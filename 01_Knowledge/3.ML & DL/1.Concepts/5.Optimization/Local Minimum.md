# Local Minimum

## What is it?

A **local minimum** is a point $\theta^*$ where the [[Cost Function]] is lower than at all nearby points, but not necessarily lower than all other points globally.

$$\exists\ \epsilon > 0: J(\theta^*) \leq J(\theta) \quad \forall \theta \text{ with } \|\theta - \theta^*\| < \epsilon$$

At a local minimum: $\nabla_\theta J(\theta^*) = 0$ and $\nabla^2_\theta J(\theta^*) \succeq 0$, but $J(\theta^*) > J(\theta^*_{\text{global}})$.

---

## Why Local Minima Matter (and Don't)

### For Convex Problems (Linear/Logistic Regression)
**No local minima exist** — any local minimum is the global minimum. Gradient descent always finds the best solution.

### For Non-Convex Problems (Neural Networks)
Local minima exist in principle. However:

**Key empirical finding (Goodfellow et al., 2015; Dauphin et al., 2014):**
In high-dimensional loss landscapes, true local minima (where the loss is significantly higher than the global minimum) are rare. Most critical points in high dimensions are **saddle points**, not local minima.

**Why saddle points matter more than local minima:**
A local minimum in $d$ dimensions requires the Hessian to be positive definite — all $d$ eigenvalues must be positive. For a random critical point, each eigenvalue is independently positive with probability $p < 1$, so the probability of a true local minimum is $\sim p^d \to 0$ exponentially. Most critical points are saddle points (some positive, some negative eigenvalues).

---

## Saddle Points

At a saddle point: $\nabla J = 0$ but the Hessian has **both positive and negative eigenvalues**.

Moving along negative-eigenvalue directions decreases $J$. SGD's noise helps escape saddle points by adding perturbations in these directions.

---

## Practical Impact

For modern deep learning:
- Local minima found are typically near-globally optimal (similar loss value).
- The **generalisation quality** of different local minima may differ (flat minima generalise better).
- **Flat minima hypothesis:** Minima with low curvature (small Hessian eigenvalues) generalise better than sharp minima. SGD's noise bias toward flatter minima.

---

## Connections

- [[Global Minimum]] — what we want to find
- [[Cost Surface]] — the landscape containing local minima
- [[Gradient Descent]] — can get stuck at local minima
- [[Convergence]] — may converge to a local minimum

---

## One-line Summary

> A local minimum is a point lower than its neighbours but not globally lowest — while problematic in theory for neural networks, empirical evidence shows that SGD rarely gets stuck in bad local minima because high-dimensional landscapes have exponentially few true local minima relative to saddle points.
