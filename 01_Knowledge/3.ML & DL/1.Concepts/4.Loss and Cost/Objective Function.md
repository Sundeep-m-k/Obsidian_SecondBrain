# Objective Function

## What is it?

The **objective function** is the mathematical expression that training **optimises** (minimises or maximises). In supervised ML, it is the quantity that defines what "good parameters" means.

$$\hat{\theta} = \arg\min_{\theta} \mathcal{O}(\theta)$$

The objective function typically has two components:

$$\mathcal{O}(\theta) = \underbrace{J(\theta)}_{\text{data fit (cost function)}} + \underbrace{\lambda \Omega(\theta)}_{\text{regularisation penalty}}$$

---

## Objective = Cost + Regularisation

| Component | Purpose | Example |
|---|---|---|
| Cost $J(\theta)$ | Fit the training data | MSE, cross-entropy |
| Penalty $\lambda\Omega(\theta)$ | Prevent overfitting | L1, L2 norm of weights |

Without regularisation, $\mathcal{O}(\theta) = J(\theta)$.

With L2 regularisation (Ridge):
$$\mathcal{O}(\theta) = \frac{1}{n}\sum_{i=1}^n \ell(y^{(i)}, \hat{f}(x^{(i)};\theta)) + \frac{\lambda}{2}\sum_{j=1}^d \theta_j^2$$

With L1 regularisation (Lasso):
$$\mathcal{O}(\theta) = \frac{1}{n}\sum_{i=1}^n \ell(y^{(i)}, \hat{f}(x^{(i)};\theta)) + \lambda\sum_{j=1}^d |\theta_j|$$

**Note:** The bias term $\theta_0$ is conventionally **not** regularised (it doesn't contribute to model complexity).

---

## Probabilistic View: MAP Estimation

Regularisation has a Bayesian interpretation:

$$\hat{\theta}_{\text{MAP}} = \arg\max_\theta \underbrace{p(\mathcal{D} \mid \theta)}_{\text{likelihood}} \cdot \underbrace{p(\theta)}_{\text{prior}}$$

Taking the negative log:
$$= \arg\min_\theta \left[-\log p(\mathcal{D} \mid \theta) - \log p(\theta)\right]$$

| Prior on $\theta$ | Regulariser |
|---|---|
| $\theta_j \sim \mathcal{N}(0, 1/\lambda)$ | L2: $\lambda\|\theta\|^2$ |
| $\theta_j \sim \text{Laplace}(0, 1/\lambda)$ | L1: $\lambda\|\theta\|_1$ |

**L2 regularisation = MAP with Gaussian prior. L1 = MAP with Laplace prior.**

---

## Objective Functions for Common Models

| Model | Objective |
|---|---|
| Linear Regression | $\min_\theta \frac{1}{2n}\|X\theta-y\|^2$ |
| Ridge Regression | $\min_\theta \frac{1}{2n}\|X\theta-y\|^2 + \frac{\lambda}{2}\|\theta\|^2$ |
| Lasso | $\min_\theta \frac{1}{2n}\|X\theta-y\|^2 + \lambda\|\theta\|_1$ |
| Logistic Regression | $\min_\theta -\frac{1}{n}\sum[y\log\hat{p}+(1-y)\log(1-\hat{p})]$ |
| SVM | $\min_{\theta,b} \frac{1}{2}\|\theta\|^2 + C\sum_i \max(0,1-y^{(i)}(\theta^Tx^{(i)}+b))$ |
| Neural Network | $\min_\theta \frac{1}{n}\sum \ell(y^{(i)}, \hat{f}(x^{(i)};\theta)) + \lambda\|\theta\|^2$ |

---

## Connections

- [[Cost Function]] — the data-fit component of the objective
- [[Regularization]] — the penalty component
- [[Loss Function]] — per-example loss that sums into the cost
- [[Gradient Descent]] — minimises the objective

---

## One-line Summary

> The objective function is the complete mathematical expression that training minimises — typically the sum of a data-fit cost and a regularisation penalty, and its form encodes all assumptions about how predictions should be made and how complex the model should be.
