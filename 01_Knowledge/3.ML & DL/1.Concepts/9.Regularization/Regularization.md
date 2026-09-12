# Regularization

## What is it?

**Regularization** is any technique that reduces a model's [[Variance]] (tendency to overfit) by adding constraints or penalties that discourage overly complex solutions.

The general form adds a penalty term to the [[Objective Function]]:

$$J_{\text{reg}}(\theta) = \underbrace{J(\theta)}_{\text{data fit}} + \underbrace{\lambda\,\Omega(\theta)}_{\text{complexity penalty}}$$

Where:
- $\lambda > 0$ = **regularisation strength** (hyperparameter — you tune this)
- $\Omega(\theta)$ = penalty function measuring model complexity
- Minimising $J_{\text{reg}}$ forces the model to balance fitting the data and staying simple

---

## Why Regularization Works

Without regularisation, the training objective only cares about fitting training data — this can lead to arbitrarily large weights that memorise noise.

Regularisation **constrains the parameter space** effectively to a smaller region, reducing the model's effective capacity.

**Geometric view:** L2 regularisation constrains $\theta$ to lie within a ball of radius $r$ around the origin. L1 constrains to a diamond shape (which intersects the solution at sparse corners).

---

## The Regularisation Hyperparameter $\lambda$

| $\lambda$ | Effect |
|---|---|
| $\lambda = 0$ | No regularisation; standard ERM |
| $\lambda$ small | Mild penalty; model mostly fits data |
| $\lambda$ large | Strong penalty; weights shrink toward 0; high bias |
| $\lambda \to \infty$ | All weights → 0; model predicts constant |

Choose $\lambda$ via cross-validation.

---

## Bayesian Interpretation

Regularisation corresponds to placing a **prior** on parameters and doing MAP estimation:

$$\hat{\theta}_{\text{MAP}} = \arg\max_\theta \left[\log p(\mathcal{D}\mid\theta) + \log p(\theta)\right] = \arg\min_\theta\left[-\log p(\mathcal{D}\mid\theta) - \log p(\theta)\right]$$

| Regulariser | Prior on $\theta_j$ |
|---|---|
| L2 (Ridge) | $\theta_j \sim \mathcal{N}(0, 1/\lambda)$ — Gaussian |
| L1 (Lasso) | $\theta_j \sim \text{Laplace}(0, 1/\lambda)$ — double exponential |

More regularisation = stronger prior that pulls $\theta$ toward 0.

---

## Other Regularisation Methods

| Method | Applies to | Mechanism |
|---|---|---|
| **L2 (Ridge)** | Any linear/NN | Penalise $\sum\theta_j^2$ |
| **L1 (Lasso)** | Any linear/NN | Penalise $\sum|\theta_j|$; induces sparsity |
| **Dropout** | Neural networks | Randomly zero neurons during training |
| **Early stopping** | Any iterative | Stop when val loss stops improving |
| **Data augmentation** | Images, text | Expand training set with transforms |
| **Batch normalisation** | Neural networks | Normalise activations per batch |
| **Weight decay** | Neural networks | Same as L2 on weights (not biases) |
| **Max-norm constraint** | Neural networks | Clip $\|w\|_2 \leq c$ for each neuron |

---

## Connections

- [[L1 Regularization]] — Lasso penalty
- [[L2 Regularization]] — Ridge penalty
- [[Regularized Linear Regression]] — Ridge/Lasso on linear model
- [[Regularized Logistic Regression]] — Ridge/Lasso on logistic model
- [[Penalty Term]] — the $\lambda\Omega(\theta)$ component
- [[Overfitting]] — what regularisation prevents
- [[Bias Variance Tradeoff]] — regularisation increases bias, decreases variance
- [[Gradient Boosting Libraries (XGBoost & LightGBM)]] — a concrete non-linear-model example of the same idea: XGBoost's $\Omega(h)=\gamma T + \frac12\lambda\sum w_j^2$ penalizes tree leaf weights exactly the way L2 penalizes linear-model weights here

---

## One-line Summary

> Regularisation prevents overfitting by adding a penalty to the training objective that discourages large weights — it reduces model variance at the cost of some bias, with the strength controlled by the hyperparameter $\lambda$ tuned via cross-validation.
