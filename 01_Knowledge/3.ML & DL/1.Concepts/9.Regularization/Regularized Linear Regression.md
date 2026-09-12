# Regularized Linear Regression

## What is it?

**Regularized Linear Regression** applies [[Regularization]] to [[Linear Regression]] to prevent overfitting by adding a penalty on the parameter magnitude to the [[Mean Squared Error]] cost.

---

## Ridge Regression (L2)

$$J_{\text{Ridge}}(\theta) = \frac{1}{2n}\sum_{i=1}^n(\hat{y}^{(i)} - y^{(i)})^2 + \frac{\lambda}{2}\sum_{j=1}^d\theta_j^2$$

**Closed-form solution:**
$$\hat{\theta}_{\text{Ridge}} = (X^TX + n\lambda I)^{-1}X^Ty$$

**Gradient descent update:**
$$\theta_j \leftarrow (1-\eta\lambda)\theta_j - \frac{\eta}{n}\sum_{i=1}^n(\hat{y}^{(i)} - y^{(i)})x_j^{(i)}, \quad j=1,\ldots,d$$
$$\theta_0 \leftarrow \theta_0 - \frac{\eta}{n}\sum_{i=1}^n(\hat{y}^{(i)} - y^{(i)}) \quad \text{(no regularisation on bias)}$$

**Effect:** Shrinks all weights toward zero. Stabilises solution when $X^TX$ is ill-conditioned.

---

## Lasso Regression (L1)

$$J_{\text{Lasso}}(\theta) = \frac{1}{2n}\sum_{i=1}^n(\hat{y}^{(i)} - y^{(i)})^2 + \lambda\sum_{j=1}^d|\theta_j|$$

**No closed-form solution.** Solved via coordinate descent or proximal gradient.

**Soft-thresholding update per coordinate:**
$$\theta_j \leftarrow S\!\left(\theta_j - \frac{\eta}{n}\sum_i(\hat{y}^{(i)}-y^{(i)})x_j^{(i)},\ \eta\lambda\right)$$

Where $S(\rho, \lambda) = \text{sign}(\rho)\max(|\rho|-\lambda, 0)$ (soft threshold operator).

**Effect:** Drives some weights exactly to zero — automatic feature selection.

---

## Elastic Net

Combines both:
$$J_{\text{EN}}(\theta) = \frac{1}{2n}\|X\theta-y\|^2 + \lambda_1\|\theta\|_1 + \frac{\lambda_2}{2}\|\theta\|^2$$

Best of both: sparsity (L1) + stability with correlated features (L2).

---

## Bias Reduction Path

As $\lambda$ increases from 0 to $\infty$:
- $\lambda = 0$: OLS solution (no regularisation)
- $\lambda$ small: slight shrinkage
- $\lambda$ large: all weights → 0

The **regularisation path** shows how each $\theta_j$ changes with $\lambda$:

```
θⱼ
  │ θ₁────────╮
  │ θ₂──────────╮
  │ θ₃─────────────╮
  │                  ╲
  │                   ──────── (all approach 0)
  └──────────────────── log(λ)
    small λ              large λ
```

---

## How to Choose $\lambda$

Use **cross-validation**:
1. Try a log-spaced grid of $\lambda$ values (e.g., $10^{-4}$ to $10^4$).
2. For each $\lambda$, compute $k$-fold CV error.
3. Choose $\lambda$ with minimum CV error (or one-standard-error rule: choose the simplest model within 1 SE of the minimum).

---

## Connections

- [[Linear Regression]] — base model
- [[L1 Regularization]] — Lasso penalty
- [[L2 Regularization]] — Ridge penalty
- [[Regularization]] — the general concept
- [[Overfitting]] — what regularisation prevents
- [[Bias Variance Tradeoff]] — regularisation trades bias for variance

---

## One-line Summary

> Regularized linear regression adds L1 or L2 penalties to MSE — Ridge produces stable dense solutions via a modified normal equation, Lasso produces sparse solutions via soft-thresholding, and both require $\lambda$ tuning via cross-validation.
