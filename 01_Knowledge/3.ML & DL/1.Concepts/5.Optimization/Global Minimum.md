# Global Minimum

## What is it?

The **global minimum** of the [[Cost Function]] $J(\theta)$ is the parameter setting $\theta^*$ that achieves the **lowest possible cost value** over all possible $\theta$.

$$\theta^* = \arg\min_{\theta} J(\theta) \quad \text{such that } J(\theta^*) \leq J(\theta) \quad \forall \theta$$

---

## Existence and Uniqueness

- For **convex** cost functions (linear regression with MSE, logistic regression with cross-entropy): the global minimum **always exists and is unique** (or a convex set of equivalent minima).
- For **non-convex** functions (neural networks): global minimum exists in principle but may be hard to find. Multiple global minima may exist.

**First-order condition:** At any minimum: $\nabla_\theta J(\theta^*) = 0$.
**Second-order condition:** At a global minimum: $\nabla^2_\theta J(\theta^*) \succeq 0$ (positive semi-definite Hessian).

---

## For Linear Regression

The global minimum is given by the Normal Equations:
$$\theta^* = (X^TX)^{-1}X^Ty$$

At this point, $\nabla_\theta J = \frac{1}{n}X^T(X\theta^* - y) = 0$.

The minimum cost value is:
$$J(\theta^*) = \frac{1}{2n}\|y - X\theta^*\|^2 = \frac{1}{2n}\|y - \hat{y}\|^2$$

This is zero only if the data is perfectly linearly separable (which almost never happens with real data — there's always some irreducible noise).

---

## Connections

- [[Cost Function]] — the function whose minimum we seek
- [[Gradient Descent]] — attempts to reach the global minimum
- [[Local Minimum]] — a sub-optimal stopping point
- [[Convergence]] — reaching (near) the global minimum

---

## One-line Summary

> The global minimum is the parameter setting that achieves the lowest possible cost — gradient descent is guaranteed to find it for convex objectives, but only approximates it for non-convex ones like neural networks.
