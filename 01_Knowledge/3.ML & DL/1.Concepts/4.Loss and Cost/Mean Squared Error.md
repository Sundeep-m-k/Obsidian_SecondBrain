# Mean Squared Error

## What is it?

**Mean Squared Error (MSE)** is the standard [[Loss Function]] for [[Regression]] tasks. It measures the average of the squared differences between true values and predictions.

$$\text{MSE} = \frac{1}{n}\sum_{i=1}^{n}(y^{(i)} - \hat{y}^{(i)})^2$$

---

## As a Cost Function

The MSE cost function (with the $\frac{1}{2}$ convention for cleaner gradients):

$$J(\theta) = \frac{1}{2n}\sum_{i=1}^{n}(y^{(i)} - \hat{y}^{(i)})^2 = \frac{1}{2n}\|X\theta - y\|^2$$

**In matrix form:**
$$J(\theta) = \frac{1}{2n}(X\theta - y)^T(X\theta - y)$$

---

## Gradient of MSE

This is what gradient descent uses to update $\theta$:

$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)})\cdot x_j^{(i)}$$

In matrix form:
$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y)$$

The gradient points in the direction of steepest increase — gradient descent moves **opposite** to this.

---

## Statistical Derivation: Why MSE?

**Assume:** observations are generated as $y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}$ where $\epsilon^{(i)} \sim \mathcal{N}(0, \sigma^2)$.

**Likelihood of observing $y^{(i)}$ given $x^{(i)}$ and $\theta$:**
$$p(y^{(i)} \mid x^{(i)}; \theta) = \frac{1}{\sqrt{2\pi}\sigma}\exp\!\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)$$

**Log-likelihood of the full training set:**
$$\log \mathcal{L}(\theta) = \sum_{i=1}^n \log p(y^{(i)} \mid x^{(i)};\theta) = -\frac{n}{2}\log(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum_{i=1}^n(y^{(i)} - \theta^T x^{(i)})^2$$

**Maximising log-likelihood** is equivalent to **minimising** $\sum_i (y^{(i)} - \theta^T x^{(i)})^2$, which is the MSE.

**Conclusion:** MSE = MLE under Gaussian noise assumption. If your errors are Gaussian, MSE is theoretically optimal.

---

## MSE Properties

| Property | Details |
|---|---|
| **Non-negative** | $(y-\hat{y})^2 \geq 0$ always |
| **Differentiable everywhere** | Smooth — gradient always exists |
| **Convex** | As a function of $\theta$, $J(\theta)$ is a convex paraboloid |
| **Sensitive to outliers** | Error is squared — one large error dominates |
| **Units** | MSE is in squared units of $y$ |

---

## Related Metrics

| Metric | Formula | Units | Notes |
|---|---|---|---|
| **MSE** | $\frac{1}{n}\sum(y-\hat{y})^2$ | $[y]^2$ | Training objective |
| **RMSE** | $\sqrt{MSE}$ | $[y]$ | Interpretable, same units as $y$ |
| **MAE** | $\frac{1}{n}\sum|y-\hat{y}|$ | $[y]$ | Robust to outliers |
| **MAPE** | $\frac{100}{n}\sum\left|\frac{y-\hat{y}}{y}\right|$ | % | Relative error |
| **R²** | $1 - \frac{MSE}{\text{Var}(y)}$ | dimensionless | Proportion of variance explained |

---

## MSE vs. MAE: When to Use Which

| Situation | Use |
|---|---|
| Errors are approximately Gaussian (symmetric) | MSE (MLE-optimal) |
| Data has significant outliers | MAE (more robust) |
| You need a differentiable loss everywhere | MSE |
| You want to penalise large errors heavily | MSE |
| You want equal weighting of all errors | MAE |

---

## Visualising MSE

For $d=1$ (simple linear regression), $J(\theta_0, \theta_1)$ is a 2D paraboloid:

```
J(θ)
  │     ╭───╮
  │   ╱       ╲
  │ ╱     *     ╲     * = global minimum
  │╱               ╲
  └─────────────────── θ₁ (or θ₀)
```

Contour lines of $J$ in the $(\theta_0, \theta_1)$ plane are ellipses (circles if features are well-scaled). Gradient descent descends these ellipses toward the minimum.

---

## Connections

- [[Loss Function]] — MSE is a specific loss function
- [[Cost Function]] — $J(\theta) = \frac{1}{2n}\sum(y-\hat{y})^2$ is the MSE cost
- [[Linear Regression]] — MSE is the standard training objective
- [[Gradient Descent]] — uses $\nabla J$ to update parameters
- [[Cost Surface]] — the shape of $J(\theta)$
- [[Regression]] — the task MSE is designed for

---

## One-line Summary

> MSE is the average squared prediction error — it is the natural loss for regression under Gaussian noise, differentiable everywhere, convex in parameters for linear models, and forms the core training objective of linear regression.
