# Polynomial Regression

## What is it?

**Polynomial Regression** fits a **polynomial curve** to data by adding polynomial terms of the features. It is still a **linear model** (linear in parameters $\theta$), just with non-linear features.

$$\hat{y} = \theta_0 + \theta_1 x + \theta_2 x^2 + \theta_3 x^3 + \cdots + \theta_p x^p$$

---

## Key Insight: Non-linear in $x$, Linear in $\theta$

This is the crucial point. If we define:
$$\phi(x) = [1,\ x,\ x^2,\ \ldots,\ x^p]^T$$

Then:
$$\hat{y} = \theta^T \phi(x)$$

This is just [[Multiple Linear Regression]] with features $\phi(x)$. **All the same math applies.** The same Normal Equations, same gradient descent, same loss function.

---

## Degree and Overfitting

| Degree $p$ | Model | Risk |
|---|---|---|
| $p = 1$ | Straight line | Underfitting if data is curved |
| $p = 2$ | Quadratic | Good for simple curves |
| $p = 3$ | Cubic | More flexible |
| $p \gg n$ | Very high-degree | Overfitting — interpolates noise |

**Classic example:** $n=10$ points, degree-9 polynomial will pass through all 10 points exactly but oscillate wildly between them (Runge's phenomenon).

The model that fits training data perfectly does not generalise. See [[Overfitting]].

---

## Multivariate Polynomial Features

For $d=2$ features $[x_1, x_2]$, a degree-2 polynomial expansion gives:
$$\phi(x) = [1,\ x_1,\ x_2,\ x_1^2,\ x_2^2,\ x_1 x_2]$$

5 new features + bias = 6 total features.

In general, a degree-$p$ polynomial of $d$ features creates:
$$\binom{d+p}{p} \text{ features}$$

This grows rapidly — for $d=10$, $p=3$: $\binom{13}{3} = 286$ features. For $d=100$, $p=2$: $\binom{102}{2} = 5151$ features.

**Beware the curse of dimensionality** as $p$ and $d$ increase.

---

## Choosing the Degree

Use **cross-validation**:
1. Try different values of $p$ (e.g., 1 through 10).
2. For each $p$, evaluate validation error.
3. Choose $p$ that minimises validation error.
4. Training error always decreases with $p$. Only validation error tells you the right $p$.

---

## Connections

- [[Linear Regression]] — polynomial regression is linear regression with transformed features
- [[Feature Engineering]] — polynomial features are engineered features
- [[Overfitting]] — high-degree polynomials overfit
- [[Regularization]] — essential for high-degree polynomials
- [[Generalization]] — the actual goal

---

## One-line Summary

> Polynomial regression captures non-linear patterns by expanding features into polynomial terms, then applying standard linear regression — it is linear in parameters but non-linear in the original input, enabling flexible curve fitting at the cost of increased overfitting risk.
