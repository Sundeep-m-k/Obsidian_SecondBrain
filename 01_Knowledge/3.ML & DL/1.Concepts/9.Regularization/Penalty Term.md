# Penalty Term

## What is it?

The **penalty term** (also called the **regularisation term**) is the component added to the [[Cost Function]] to penalise model complexity and prevent [[Overfitting]].

$$J_{\text{reg}}(\theta) = \underbrace{J(\theta)}_{\text{data fit}} + \underbrace{\lambda\,\Omega(\theta)}_{\text{penalty term}}$$

---

## Common Penalty Terms

| Name | Formula | Effect |
|---|---|---|
| **L2 (Ridge)** | $\frac{\lambda}{2}\|\theta\|_2^2 = \frac{\lambda}{2}\sum_j\theta_j^2$ | Shrinks all weights toward 0 |
| **L1 (Lasso)** | $\lambda\|\theta\|_1 = \lambda\sum_j|\theta_j|$ | Drives some weights to exactly 0 |
| **Elastic Net** | $\lambda_1\|\theta\|_1 + \frac{\lambda_2}{2}\|\theta\|_2^2$ | Combines both |
| **Nuclear norm** | $\lambda\|W\|_*$ (sum of singular values) | Low-rank matrix regularisation |
| **Total variation** | $\lambda\sum_j|\theta_j - \theta_{j-1}|$ | Smooth/piecewise constant solutions |

---

## The Role of $\lambda$

$\lambda$ controls the **tradeoff between data fit and penalty**:

$$\min_\theta \underbrace{J(\theta)}_{\lambda\to 0: \text{only this matters}} + \underbrace{\lambda\Omega(\theta)}_{\lambda\to\infty: \text{only this matters}}$$

- $\lambda = 0$: ordinary training (no constraint)
- $\lambda \to \infty$: all parameters driven to 0

Choose $\lambda$ via cross-validation over a log-spaced grid.

---

## What Is Not Penalised

By convention, the **bias term $\theta_0$** is never included in the penalty. Regularising the bias would shift the overall level of predictions, not reduce complexity. The bias term captures the mean of $y$ — penalising it would force predictions toward zero regardless of the data's mean.

---

## Connections

- [[Regularization]] — penalty term is the mechanism
- [[L1 Regularization]] — L1 penalty
- [[L2 Regularization]] — L2 penalty
- [[Objective Function]] — cost + penalty = objective

---

## One-line Summary

> The penalty term is the complexity-penalising component of the regularised objective — by adding a cost for large weights, it prevents the optimiser from fitting noise and forces it to find simpler, more generalisable solutions.
