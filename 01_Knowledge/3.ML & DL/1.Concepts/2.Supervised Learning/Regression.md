# Regression

## What is it?

**Regression** is a type of [[Supervised Learning]] task where the model learns to predict a **continuous numerical output** from input features.

$$\hat{f}: \mathbb{R}^d \rightarrow \mathbb{R}$$

The output $y \in \mathbb{R}$ is a real number, not a category.

---

## Formal Setup

Given training data:
$$\mathcal{D} = \{(x^{(1)}, y^{(1)}), \ldots, (x^{(n)}, y^{(n)})\}, \quad y^{(i)} \in \mathbb{R}$$

Find parameters $\theta$ such that:
$$\hat{y} = \hat{f}(x; \theta) \approx y \quad \text{for unseen } x$$

---

## When is it Regression?

The output is **continuous** — it can take any real value in some range.

| Problem | Output | Regression? |
|---|---|---|
| Predict house price | \$247,500 | ✅ Yes |
| Predict tomorrow's temperature | 22.3°C | ✅ Yes |
| Predict stock return | +2.4% | ✅ Yes |
| Predict patient age from scan | 54.2 years | ✅ Yes |
| Predict if email is spam | Yes/No | ❌ No (Classification) |
| Predict which digit (0–9) | 7 | ❌ No (Classification) |

---

## Standard Loss Functions for Regression

### Mean Squared Error (MSE)
$$\mathcal{L}_{\text{MSE}} = \frac{1}{n}\sum_{i=1}^{n}(y^{(i)} - \hat{y}^{(i)})^2$$

- Differentiable everywhere → easy to optimise with gradient descent.
- Penalises large errors **quadratically** — outliers have outsized influence.
- Minimising MSE is equivalent to Maximum Likelihood Estimation under the assumption that errors are Gaussian: $y = f(x) + \epsilon,\ \epsilon \sim \mathcal{N}(0, \sigma^2)$.

### Mean Absolute Error (MAE)
$$\mathcal{L}_{\text{MAE}} = \frac{1}{n}\sum_{i=1}^{n}|y^{(i)} - \hat{y}^{(i)}|$$

- Robust to outliers (linear not quadratic penalty).
- Not differentiable at 0 — requires subgradient methods.
- Equivalent to MLE under Laplace noise.

### Huber Loss
$$\mathcal{L}_{\delta}(r) = \begin{cases} \frac{1}{2}r^2 & \text{if } |r| \leq \delta \\ \delta|r| - \frac{1}{2}\delta^2 & \text{if } |r| > \delta \end{cases}$$

where $r = y - \hat{y}$. Combines quadratic (small errors) and linear (large errors) behaviour. Best of both worlds.

---

## Key Metrics

| Metric | Formula | Interpretation |
|---|---|---|
| **MSE** | $\frac{1}{n}\sum(y-\hat{y})^2$ | Average squared error |
| **RMSE** | $\sqrt{\text{MSE}}$ | Same units as $y$; interpretable |
| **MAE** | $\frac{1}{n}\sum|y-\hat{y}|$ | Average absolute error |
| **R² (R-squared)** | $1 - \frac{\sum(y-\hat{y})^2}{\sum(y-\bar{y})^2}$ | Proportion of variance explained |
| **MAPE** | $\frac{100}{n}\sum\left|\frac{y-\hat{y}}{y}\right|$ | Mean absolute percentage error |

**R² interpretation:**
- $R^2 = 1$: perfect predictions
- $R^2 = 0$: model is no better than predicting the mean $\bar{y}$
- $R^2 < 0$: model is worse than predicting the mean (possible!)

$$R^2 = 1 - \frac{SS_{\text{res}}}{SS_{\text{tot}}} \quad \text{where} \quad SS_{\text{res}} = \sum(y - \hat{y})^2,\ SS_{\text{tot}} = \sum(y - \bar{y})^2$$

---

## Types of Regression Models

| Model | Equation | Notes |
|---|---|---|
| **Simple Linear** | $\hat{y} = \theta_0 + \theta_1 x$ | One feature |
| **Multiple Linear** | $\hat{y} = \theta^T x$ | $d$ features |
| **Polynomial** | $\hat{y} = \theta_0 + \theta_1 x + \theta_2 x^2 + \ldots$ | Non-linear in $x$, linear in $\theta$ |
| **Ridge (L2)** | $\hat{y} = \theta^T x$, penalise $\|\theta\|^2$ | Prevents overfitting |
| **Lasso (L1)** | $\hat{y} = \theta^T x$, penalise $\|\theta\|_1$ | Sparse solutions |
| **Decision Tree** | Piecewise constant | Axis-aligned partitions |
| **Random Forest** | Ensemble of trees | Low variance |
| **Neural Network** | Arbitrary $\hat{f}(x;\theta)$, linear output | Universal approximation |

---

## The Assumptions of Linear Regression (OLS)

For the Ordinary Least Squares estimator to be optimal (BLUE — Best Linear Unbiased Estimator, per the **Gauss-Markov theorem**):

1. **Linearity:** $y = \theta^T x + \epsilon$
2. **Independence:** Errors $\epsilon^{(i)}$ are independent across examples
3. **Homoscedasticity:** Constant variance $\text{Var}(\epsilon^{(i)}) = \sigma^2$ for all $i$
4. **Zero mean errors:** $\mathbb{E}[\epsilon] = 0$
5. **No perfect multicollinearity:** No feature is a perfect linear combination of others (required for $(X^TX)^{-1}$ to exist)

When these hold, OLS is the most efficient unbiased estimator. When they're violated, use robust regression, WLS, or ML-based methods.

---

## Connections

- [[Linear Regression]] — the canonical regression algorithm
- [[Multiple Linear Regression]] — regression with $d > 1$ features
- [[Polynomial Regression]] — regression with polynomial features
- [[Loss Function]] — MSE is the standard regression loss
- [[Classification]] — the contrast: discrete outputs
- [[Supervised Learning]] — regression is a supervised task
- [[Generalization]] — do regression predictions work on new data?

---

## One-line Summary

> Regression is supervised learning for continuous outputs — the model learns to predict a real number, trained by minimising the difference between predictions and true values.
