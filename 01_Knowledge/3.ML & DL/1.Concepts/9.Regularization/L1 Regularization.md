# L1 Regularization

## What is it?

**L1 Regularization** (also called **Lasso** — Least Absolute Shrinkage and Selection Operator) adds the sum of the **absolute values** of the parameters as a penalty to the cost function.

$$J_{\text{L1}}(\theta) = J(\theta) + \lambda\sum_{j=1}^{d}|\theta_j|= J(\theta) + \lambda\|\theta\|_1$$

The bias term $\theta_0$ is conventionally **not** regularised.

---

## Key Property: Sparsity

L1 regularisation drives many weights to **exactly zero** — it performs automatic feature selection.

**Why exact zeros?** The L1 penalty $|\theta_j|$ has a non-smooth corner at $\theta_j = 0$. The subgradient condition allows $\theta_j = 0$ to be a stable solution even when the data gradient is nonzero. L2 (smooth penalty) only drives weights near zero, never exactly zero.

**Geometric intuition:**
The L1 constraint region $\|\theta\|_1 \leq r$ is a **diamond** (in 2D) / cross-polytope (in high dimensions). Its corners lie on the axes. The cost function's level sets (ellipses for quadratic cost) tend to touch the constraint region at a corner — which is exactly where one parameter is zero.

```
θ₂
 │   ╱╲
 │  ╱  ╲
 │ ╱    ╲        ← L1 diamond (constraint region)
 │╱      ╲
─●────────●── θ₁
 │╲      ╱
 │ ╲    ╱
 │  ╲  ╱
 │   ╲╱
      ↑ corner touches axis → sparse solution
```

---

## Gradient (or Subgradient)

Since $|\theta_j|$ is not differentiable at $\theta_j = 0$, we use the subgradient:

$$\frac{\partial}{\partial\theta_j}|\theta_j| = \begin{cases}+1 & \theta_j > 0 \\ -1 & \theta_j < 0 \\ [-1,+1] & \theta_j = 0 \end{cases} = \text{sign}(\theta_j)$$

**Gradient of L1 regularised cost (for $\theta_j \neq 0$):**
$$\frac{\partial J_{\text{L1}}}{\partial\theta_j} = \frac{\partial J}{\partial\theta_j} + \lambda\,\text{sign}(\theta_j)$$

**Soft-thresholding update** (coordinate descent solution for Lasso):
$$\theta_j \leftarrow \text{sign}(\rho_j)\max(|\rho_j| - \lambda, 0)$$

Where $\rho_j = \frac{1}{n}\sum_i x_j^{(i)}(y^{(i)} - \hat{y}_{-j}^{(i)})$ is the partial residual.

This is called the **soft-threshold operator** $S(\rho_j, \lambda)$: if $|\rho_j| < \lambda$, set $\theta_j = 0$; otherwise shrink toward zero by $\lambda$.

---

## Bayesian View

L1 regularisation = MAP estimation with a **Laplace prior**:

$$p(\theta_j) = \frac{\lambda}{2}\exp(-\lambda|\theta_j|)$$

The Laplace distribution has heavy tails and a sharp peak at zero — it strongly encourages zeros while allowing occasional large values (unlike Gaussian which gently penalises all values equally).

---

## Lasso for Linear Regression

$$\hat{\theta}_{\text{Lasso}} = \arg\min_\theta \frac{1}{2n}\|X\theta - y\|^2 + \lambda\|\theta\|_1$$

No closed-form solution (unlike Ridge). Solved via:
- **Coordinate Descent** (efficient, standard)
- **LARS** (Least Angle Regression)
- **Proximal Gradient Descent**

---

## When to Use L1

✅ When you suspect many features are irrelevant — L1 will zero them out automatically  
✅ When interpretability is important — sparse models are easier to understand  
✅ When $d \gg n$ — high-dimensional settings where feature selection is necessary  
❌ When all features are genuinely relevant — L1 will arbitrarily drop some  
❌ When features are highly correlated — L1 arbitrarily picks one (L2 or Elastic Net is better)  

---

## L1 vs L2 Comparison

| Property | L1 (Lasso) | L2 (Ridge) |
|---|---|---|
| Penalty | $\lambda\sum|\theta_j|$ | $\frac{\lambda}{2}\sum\theta_j^2$ |
| Solution | Sparse (some weights = 0) | Dense (all weights shrink toward 0) |
| Feature selection | ✅ Automatic | ❌ No |
| Differentiable | ❌ Not at 0 | ✅ Everywhere |
| Closed form | ❌ No | ✅ Yes (Ridge) |
| Correlated features | Picks one arbitrarily | Averages them |
| Prior | Laplace | Gaussian |

---

## Elastic Net: Best of Both

Combines L1 and L2:
$$J_{\text{EN}}(\theta) = J(\theta) + \lambda_1\|\theta\|_1 + \lambda_2\|\theta\|_2^2$$

- Gets sparsity (from L1) + stability with correlated features (from L2).
- Hyperparameters: $\lambda_1, \lambda_2$ (or equivalently, total strength $\lambda$ and mixing ratio $\alpha$).

---

## Connections

- [[Regularization]] — L1 is one type
- [[L2 Regularization]] — the alternative
- [[Regularized Linear Regression]] — Lasso applied to linear regression
- [[Penalty Term]] — the $\lambda\|\theta\|_1$ term
- [[Features]] — L1 performs feature selection by zeroing weights

---

## One-line Summary

> L1 regularisation adds the sum of absolute parameter values as a penalty, producing sparse solutions where many weights are driven to exactly zero — making it a powerful method for simultaneous regularisation and automatic feature selection.
