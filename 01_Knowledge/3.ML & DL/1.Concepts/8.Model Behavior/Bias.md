# Bias

## What is it?

**Bias** measures the systematic error of a model — how far the model's average prediction is from the true value, averaged over all possible training sets.

$$\text{Bias}^2(\hat{f}(x)) = \left(\mathbb{E}_{\mathcal{D}}[\hat{f}(x)] - f(x)\right)^2$$

Where:
- $f(x)$ = the true (unknown) function
- $\mathbb{E}_{\mathcal{D}}[\hat{f}(x)]$ = the expected prediction across all possible training sets of size $n$
- High bias = model is consistently wrong in the same direction

---

## Intuition

**High bias** = the model makes the same type of mistake regardless of which training data it sees.

Example: fitting a straight line to data that is truly quadratic. No matter how much data you give it, the line will always be too flat in the curved region — it's structurally incapable of fitting the true curve.

---

## Sources of Bias

- Model hypothesis class is too restricted (linear model for non-linear truth)
- Missing important features
- Excessive regularisation forcing weights toward zero

---

## Bias in the Error Decomposition

$$\mathbb{E}\left[(y - \hat{f}(x))^2\right] = \text{Bias}^2(\hat{f}(x)) + \text{Var}(\hat{f}(x)) + \sigma_\epsilon^2$$

Bias² is one of three additive components. It is irreducible without changing the model family.

---

## Connections

- [[Variance]] — the other main component of error
- [[Bias Variance Tradeoff]] — bias and variance trade off
- [[Underfitting]] — high bias is the cause
- [[Regularization]] — too much regularisation increases bias

---

## One-line Summary

> Bias is the systematic error from a model that is too simple — it measures how much the model's average prediction consistently deviates from the truth, independent of which training data was used.
