# L2 Regularization

## What is it?

**L2 Regularization** (also called **Ridge regression** for linear models, or **weight decay** in neural networks) adds the sum of **squared parameter values** as a penalty to the cost function.

$$J_{\text{L2}}(\theta) = J(\theta) + \frac{\lambda}{2}\sum_{j=1}^{d}\theta_j^2 = J(\theta) + \frac{\lambda}{2}\|\theta\|_2^2$$

The $\frac{1}{2}$ is a convenience factor that cancels the 2 from differentiation. The bias term $\theta_0$ is conventionally **not** regularised.

---

## Effect on Parameters

L2 regularisation **shrinks all weights toward zero proportionally** — larger weights are penalised more heavily, but no weight is ever driven to exactly zero (unlike L1).

**After training:** Each weight satisfies the first-order condition:
$$\frac{\partial J}{\partial\theta_j} + \lambda\theta_j = 0$$

The regularisation term adds $\lambda\theta_j$ to the gradient, pulling the weight back toward zero at every update.

---

## Gradient and Update Rule

$$\frac{\partial J_{\text{L2}}}{\partial\theta_j} = \frac{\partial J}{\partial\theta_j} + \lambda\theta_j$$

**Gradient descent update with L2:**
$$\theta_j \leftarrow \theta_j - \eta\left(\frac{\partial J}{\partial\theta_j} + \lambda\theta_j\right) = \underbrace{(1 - \eta\lambda)}_{\text{weight decay factor}}\theta_j - \eta\frac{\partial J}{\partial\theta_j}$$

The factor $(1 - \eta\lambda)$ multiplies the weight at every step — this is called **weight decay**. For $\eta\lambda < 1$, the weight decays toward zero unless the gradient counteracts it.

---

## Closed-Form Solution for Ridge Regression

For linear regression with L2, the objective is:
$$J_{\text{Ridge}}(\theta) = \frac{1}{2n}\|X\theta - y\|^2 + \frac{\lambda}{2}\|\theta\|^2$$

Taking the gradient and setting to zero:
$$\nabla_\theta J_{\text{Ridge}} = \frac{1}{n}X^T(X\theta - y) + \lambda\theta = 0$$

$$\left(\frac{1}{n}X^TX + \lambda I\right)\theta = \frac{1}{n}X^Ty$$

$$\boxed{\hat{\theta}_{\text{Ridge}} = \left(X^TX + n\lambda I\right)^{-1}X^Ty}$$

**Key advantage:** $(X^TX + n\lambda I)$ is always invertible (positive definite) for $\lambda > 0$, even when $X^TX$ is singular (e.g., $d > n$ or multicollinearity). Ridge regression is always well-posed.

---

## Geometric Intuition

L2 constraint: $\|\theta\|_2^2 \leq r$ is a **sphere** (in 2D: a circle). The solution is where the cost function's ellipsoidal level set first touches the sphere.

Unlike the L1 diamond, the sphere has no corners — the touching point is generally not on an axis, so solutions are dense (no exact zeros).

```
  θ₂
   │    ╭──────╮
   │   ╱  (L2  ╲
   │  │  sphere) │
   │   ╲        ╱
───┼────╰────── ──── θ₁
   │
   (touches at a non-axis point → dense solution)
```

---

## Bayesian View

L2 regularisation = MAP estimation with a **Gaussian prior**:

$$p(\theta_j) = \mathcal{N}(0, 1/\lambda) = \frac{\sqrt{\lambda}}{\sqrt{2\pi}}\exp\!\left(-\frac{\lambda\theta_j^2}{2}\right)$$

More regularisation = tighter Gaussian prior = stronger belief that weights should be near zero.

---

## Effect on Eigenvalues (Solving Multicollinearity)

$X^TX$ has eigenvalue 0 when features are collinear. Adding $n\lambda I$ shifts all eigenvalues by $n\lambda$:
$$\text{eigenvalues}(X^TX + n\lambda I) = \text{eigenvalues}(X^TX) + n\lambda > 0$$

This stabilises the matrix inversion and makes coefficients stable even with correlated features.

**Condition number** of $X^TX + n\lambda I$:
$$\kappa = \frac{\lambda_{\max} + n\lambda}{\lambda_{\min} + n\lambda}$$

As $\lambda \to \infty$: $\kappa \to 1$ (perfect conditioning, but all weights → 0). Ridge finds the right balance.

---

## When to Use L2

✅ All features are potentially relevant (don't want to zero any out)  
✅ Features are correlated (Ridge averages correlated features; Lasso picks one)  
✅ Need a closed-form solution  
✅ Standard default for neural networks (as weight decay)  
❌ Want automatic feature selection → use L1  
❌ Want a sparse model → use L1  

---

## L2 vs L1

| Property | L2 (Ridge) | L1 (Lasso) |
|---|---|---|
| Penalty | $\frac{\lambda}{2}\|\theta\|^2$ | $\lambda\|\theta\|_1$ |
| Weights | Shrink toward 0, never = 0 | Drive some to exactly 0 |
| Feature selection | ❌ No | ✅ Yes |
| Correlated features | Averages them | Picks one |
| Differentiable | ✅ Everywhere | ❌ Not at 0 |
| Closed form (linear) | ✅ Yes | ❌ No |

---

## Connections

- [[Regularization]] — L2 is the most common type
- [[L1 Regularization]] — the sparse alternative
- [[Regularized Linear Regression]] — Ridge regression
- [[Regularized Logistic Regression]] — L2 on logistic model
- [[Penalty Term]] — the $\frac{\lambda}{2}\|\theta\|^2$ term
- [[Overfitting]] — what L2 prevents

---

## One-line Summary

> L2 regularisation adds the squared parameter norm as a penalty, shrinking all weights proportionally toward zero without inducing sparsity — it corresponds to a Gaussian prior on weights, always produces a unique closed-form solution for linear models, and is the standard choice when all features are potentially relevant.
