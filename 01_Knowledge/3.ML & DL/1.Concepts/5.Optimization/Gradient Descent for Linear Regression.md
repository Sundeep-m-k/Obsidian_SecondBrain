# Gradient Descent for Linear Regression

## What is it?

This note works through the complete, concrete application of [[Gradient Descent]] to [[Linear Regression]] — deriving each formula from scratch so everything is self-contained.

---

## Setup

**Model:** $\hat{y}^{(i)} = \theta^T x^{(i)} = \theta_0 x_0^{(i)} + \theta_1 x_1^{(i)} + \cdots + \theta_d x_d^{(i)}$ (where $x_0^{(i)} = 1$ always)

**Cost function (MSE):**
$$J(\theta) = \frac{1}{2n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)})^2$$

**Goal:** Find $\hat{\theta} = \arg\min_\theta J(\theta)$

---

## Deriving the Gradient

Expand for one parameter $\theta_j$:

$$\frac{\partial J}{\partial \theta_j} = \frac{\partial}{\partial \theta_j}\left[\frac{1}{2n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)})^2\right]$$

Apply chain rule:
$$= \frac{1}{2n}\sum_{i=1}^{n} 2(\hat{y}^{(i)} - y^{(i)}) \cdot \frac{\partial \hat{y}^{(i)}}{\partial \theta_j}$$

Since $\hat{y}^{(i)} = \sum_k \theta_k x_k^{(i)}$, we have $\frac{\partial \hat{y}^{(i)}}{\partial \theta_j} = x_j^{(i)}$.

$$\boxed{\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}}$$

---

## The Update Rule

For each parameter $\theta_j$ simultaneously:

$$\theta_j \leftarrow \theta_j - \eta \cdot \frac{1}{n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}, \quad j = 0, 1, \ldots, d$$

**Critical:** All $\theta_j$ must be updated **simultaneously** — compute all gradients first with the current $\theta$, then update all $\theta_j$ at once. Do not update $\theta_0$ and then use the new $\theta_0$ to compute the gradient for $\theta_1$.

---

## Matrix Form

For $n$ examples and $d+1$ parameters:

$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y)$$

$$\theta \leftarrow \theta - \frac{\eta}{n}X^T(X\theta - y)$$

This is a single matrix operation — efficient with NumPy/PyTorch.

---

## Full Algorithm (Simple Linear Regression, $d=1$)

1. Initialise $\theta_0 = 0$, $\theta_1 = 0$
2. Repeat until convergence:
   - $\hat{y}^{(i)} = \theta_0 + \theta_1 x^{(i)}$ for all $i$
   - $\text{tmp}_0 = \frac{1}{n}\sum_{i=1}^n (\hat{y}^{(i)} - y^{(i)}) \cdot 1$ (gradient for $\theta_0$, since $x_0 = 1$)
   - $\text{tmp}_1 = \frac{1}{n}\sum_{i=1}^n (\hat{y}^{(i)} - y^{(i)}) \cdot x^{(i)}$ (gradient for $\theta_1$)
   - $\theta_0 \leftarrow \theta_0 - \eta \cdot \text{tmp}_0$
   - $\theta_1 \leftarrow \theta_1 - \eta \cdot \text{tmp}_1$

---

## Convergence Guarantee

Since $J(\theta)$ for linear regression with MSE is a **strictly convex quadratic function**, gradient descent is guaranteed to converge to the unique global minimum for any $\eta < \frac{2}{\lambda_{\max}(X^TX/n)}$.

The convergence rate is:
$$J(\theta_t) - J(\theta^*) \leq \left(1 - \frac{2\eta\lambda_{\min}\lambda_{\max}}{\lambda_{\min}+\lambda_{\max}}\right)^t (J(\theta_0) - J(\theta^*))$$

where $\lambda_{\min}, \lambda_{\max}$ are the smallest and largest eigenvalues of $\frac{1}{n}X^TX$.

---

## Gradient Descent vs. Normal Equations

| | Gradient Descent | Normal Equations |
|---|---|---|
| **Solution** | Iterative approximation | Exact |
| **Complexity per step** | $O(nd)$ | $O(d^3)$ overall |
| **When to use** | Large $d$ (e.g., $d > 10{,}000$) | Small-medium $d$ |
| **Needs learning rate** | Yes | No |
| **Works for other models** | Yes (universal) | Only for linear+MSE |

---

## Connections

- [[Linear Regression]] — the model
- [[Gradient Descent]] — the general algorithm
- [[Gradient]] — the mathematical object computed
- [[Mean Squared Error]] — the cost function differentiated
- [[Learning Rate]] — the $\eta$ parameter
- [[Convergence]] — guaranteed for this convex problem

---

## One-line Summary

> Gradient descent for linear regression repeatedly computes the prediction errors across all training examples, uses those errors to estimate how each parameter should change, and takes a small step in that direction — guaranteed to converge to the globally optimal parameters because the MSE cost surface is a perfect convex bowl.
