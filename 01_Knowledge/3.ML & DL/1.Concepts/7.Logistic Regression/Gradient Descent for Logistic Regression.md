# Gradient Descent for Logistic Regression

## What is it?

This note derives and works through the full application of [[Gradient Descent]] to [[Logistic Regression]] — from the model equations to the parameter update rule.

---

## Setup

**Model:** $\hat{p}^{(i)} = \sigma(\theta^T x^{(i)}) = \frac{1}{1+e^{-\theta^T x^{(i)}}}$

**Cost:** $J(\theta) = -\frac{1}{n}\sum_{i=1}^n\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$

**Goal:** $\hat{\theta} = \arg\min_\theta J(\theta)$

---

## Deriving the Gradient

For one parameter $\theta_j$, applying the chain rule:

$$\frac{\partial J}{\partial \theta_j} = -\frac{1}{n}\sum_{i=1}^n\left[\frac{y^{(i)}}{\hat{p}^{(i)}} - \frac{1-y^{(i)}}{1-\hat{p}^{(i)}}\right]\frac{\partial \hat{p}^{(i)}}{\partial \theta_j}$$

Using the sigmoid derivative $\frac{\partial \sigma(z)}{\partial z} = \sigma(z)(1-\sigma(z))$:

$$\frac{\partial \hat{p}^{(i)}}{\partial \theta_j} = \hat{p}^{(i)}(1-\hat{p}^{(i)}) \cdot x_j^{(i)}$$

Substituting and simplifying:

$$\frac{\partial J}{\partial \theta_j} = -\frac{1}{n}\sum_{i=1}^n\left[\frac{y^{(i)}}{\hat{p}^{(i)}} - \frac{1-y^{(i)}}{1-\hat{p}^{(i)}}\right]\hat{p}^{(i)}(1-\hat{p}^{(i)})x_j^{(i)}$$

$$= -\frac{1}{n}\sum_{i=1}^n\left[y^{(i)}(1-\hat{p}^{(i)}) - (1-y^{(i)})\hat{p}^{(i)}\right]x_j^{(i)}$$

$$= -\frac{1}{n}\sum_{i=1}^n\left[y^{(i)} - \hat{p}^{(i)}\right]x_j^{(i)}$$

$$\boxed{\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^n(\hat{p}^{(i)} - y^{(i)})x_j^{(i)}}$$

---

## The Update Rule

$$\theta_j \leftarrow \theta_j - \eta \cdot \frac{1}{n}\sum_{i=1}^n(\hat{p}^{(i)} - y^{(i)})x_j^{(i)}, \quad j = 0, 1, \ldots, d$$

In matrix form:
$$\theta \leftarrow \theta - \frac{\eta}{n}X^T(\hat{p} - y)$$

Where $\hat{p} = \sigma(X\theta) \in \mathbb{R}^n$.

All $\theta_j$ must be updated **simultaneously**.

---

## Comparison with Linear Regression Update

| | Linear Regression | Logistic Regression |
|---|---|---|
| Update rule | $\theta \leftarrow \theta - \frac{\eta}{n}X^T(\hat{y}-y)$ | $\theta \leftarrow \theta - \frac{\eta}{n}X^T(\hat{p}-y)$ |
| Prediction | $\hat{y} = \theta^T x$ | $\hat{p} = \sigma(\theta^T x)$ |
| Same? | ✅ Identical structure | ✅ Identical structure |

The only difference is the prediction formula — the update structure is identical. This elegance comes from using MLE with the natural exponential family.

---

## Connections

- [[Logistic Regression]] — the model
- [[Gradient Descent]] — the general algorithm
- [[Cost Function for Logistic Regression]] — what is differentiated
- [[Sigmoid Function]] — used in the gradient derivation
- [[Learning Rate]] — the $\eta$ hyperparameter

---

## One-line Summary

> Gradient descent for logistic regression uses the identical update structure as linear regression — (prediction error) × (input) — with the only difference being that predictions are sigmoid-squashed rather than raw linear combinations.
