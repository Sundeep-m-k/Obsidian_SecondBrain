# Log Odds

## What is it?

The **log-odds** (also called the **logit**) is the natural logarithm of the odds ratio — the ratio of the probability of class 1 to the probability of class 0.

$$\text{logit}(\hat{p}) = \log\frac{\hat{p}}{1 - \hat{p}}$$

In logistic regression, the log-odds equals the linear combination of features:

$$\log\frac{P(Y=1\mid x)}{P(Y=0\mid x)} = \theta^T x = \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d$$

---

## The Three Linked Quantities

$$\underbrace{\theta^T x}_{\text{log-odds (logit)}} \xrightarrow{\exp} \underbrace{\frac{\hat{p}}{1-\hat{p}}}_{\text{odds}} \xrightarrow{\text{normalise}} \underbrace{\hat{p}}_{\text{probability}}$$

| Quantity | Formula | Range |
|---|---|---|
| Log-odds (logit) | $\log\frac{\hat{p}}{1-\hat{p}} = \theta^T x$ | $(-\infty, +\infty)$ |
| Odds | $\frac{\hat{p}}{1-\hat{p}} = e^{\theta^T x}$ | $(0, +\infty)$ |
| Probability | $\hat{p} = \sigma(\theta^T x)$ | $(0, 1)$ |

---

## Interpreting Logistic Regression Coefficients via Log-Odds

In linear regression, $\theta_j$ = change in $\hat{y}$ per unit increase in $x_j$.

In logistic regression, $\theta_j$ = change in **log-odds** per unit increase in $x_j$:

$$\log\frac{P(Y=1\mid x_j+1)}{P(Y=0\mid x_j+1)} - \log\frac{P(Y=1\mid x_j)}{P(Y=0\mid x_j)} = \theta_j$$

Equivalently, the **odds ratio** multiplies by $e^{\theta_j}$ per unit increase in $x_j$:

$$\text{OR} = e^{\theta_j}$$

**Examples:**
- $\theta_j = 0.5$: OR = 1.65 — each unit increase makes class 1 ~65% more likely (in odds).
- $\theta_j = -1.0$: OR = 0.37 — each unit increase makes class 1 ~63% less likely (in odds).
- $\theta_j = 0$: feature $j$ has no effect on the log-odds.

---

## Log-Odds Are Linear, Probability Is Not

The key property of logistic regression: **the log-odds is linear in features**, even though **probability is non-linear**.

This means the **decision boundary** (where log-odds = 0, probability = 0.5) is always a **linear hyperplane**, regardless of how complex the probability contours look.

---

## Connections

- [[Sigmoid Function]] — inverse of the logit: $\sigma = \text{logit}^{-1}$
- [[Logistic Regression]] — logit is the model's core
- [[Probability Output]] — final output after applying sigmoid to logit
- [[Decision Boundary in Logistic Regression]] — where logit = 0

---

## One-line Summary

> The log-odds (logit) is the linear part of logistic regression — by modelling log-odds as $\theta^T x$, the model keeps probability in $(0,1)$ while maintaining a linear decision boundary, and each coefficient $\theta_j$ tells us how much a unit increase in $x_j$ multiplies the odds by $e^{\theta_j}$.
