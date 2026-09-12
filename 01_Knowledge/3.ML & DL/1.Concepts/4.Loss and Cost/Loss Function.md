# Loss Function

## What is it?

A **loss function** $\ell(y, \hat{y})$ measures the **cost of a single prediction** — how wrong the model was on one training example.

$$\ell: \mathcal{Y} \times \mathcal{Y} \rightarrow \mathbb{R}_{\geq 0}$$

- $y$ = true label
- $\hat{y}$ = predicted value
- Output: a non-negative scalar (higher = worse prediction)

The loss is zero (or minimal) when prediction is perfect.

---

## Loss vs. Cost Function

| Term | Scope | Formula |
|---|---|---|
| **Loss** $\ell$ | Single example | $\ell(y^{(i)}, \hat{y}^{(i)})$ |
| **Cost** $J$ | Entire training set | $J(\theta) = \frac{1}{n}\sum_{i=1}^n \ell(y^{(i)}, \hat{y}^{(i)})$ |

The cost function is the average loss over all training examples. Training = minimising $J(\theta)$.

---

## Common Loss Functions

### Regression Losses

**Mean Squared Error (MSE) / Squared Loss:**
$$\ell(y, \hat{y}) = (y - \hat{y})^2$$

- Differentiable everywhere
- Penalises large errors quadratically
- Sensitive to outliers

**Mean Absolute Error (MAE) / L1 Loss:**
$$\ell(y, \hat{y}) = |y - \hat{y}|$$

- Robust to outliers
- Not differentiable at 0

**Huber Loss:**
$$\ell_\delta(y, \hat{y}) = \begin{cases} \frac{1}{2}(y-\hat{y})^2 & |y-\hat{y}| \leq \delta \\ \delta|y-\hat{y}| - \frac{\delta^2}{2} & |y-\hat{y}| > \delta \end{cases}$$

Best of both: quadratic near zero, linear for large errors.

**Log-Cosh Loss:**
$$\ell(y, \hat{y}) = \log(\cosh(y - \hat{y}))$$

Smooth approximation to Huber. Twice differentiable everywhere.

### Classification Losses

**Binary Cross-Entropy (Log Loss):**
$$\ell(y, \hat{p}) = -[y\log\hat{p} + (1-y)\log(1-\hat{p})]$$

**Categorical Cross-Entropy:**
$$\ell(y, \hat{p}) = -\sum_{k=1}^{K} y_k \log \hat{p}_k = -\log \hat{p}_{c} \quad \text{(where } c \text{ is the true class)}$$

**Hinge Loss (SVM):**
$$\ell(y, \hat{f}) = \max(0, 1 - y\hat{f}(x)), \quad y \in \{-1, +1\}$$

**Squared Hinge Loss:**
$$\ell(y, \hat{f}) = \max(0, 1 - y\hat{f}(x))^2$$

---

## Properties of Good Loss Functions

1. **Differentiable** (or subgradient exists) — needed for gradient descent
2. **Convex** — guarantees no local minima (when used with linear models)
3. **Consistent / proper** — minimising expected loss gives the true conditional distribution
4. **Computationally efficient** — fast to evaluate for millions of examples

---

## Statistical Interpretation

Loss functions derive from probabilistic models via **Maximum Likelihood Estimation (MLE)**:

- Assuming Gaussian noise → MSE
- Assuming Laplace noise → MAE
- Assuming Bernoulli outputs → Binary Cross-Entropy
- Assuming Categorical outputs → Categorical Cross-Entropy

**Negative Log-Likelihood:**
$$\ell(y, \hat{p}) = -\log p(y \mid x; \theta)$$

Minimising the loss = maximising the likelihood of the training data under the model.

---

## Connections

- [[Cost Function]] — average of loss over all training examples
- [[Mean Squared Error]] — standard regression loss
- [[Logistic Loss]] — standard classification loss
- [[Objective Function]] — loss/cost is the objective being minimised
- [[Gradient Descent]] — minimises the cost by following loss gradients
- [[Proper Scoring Rule]] — formalizes exactly what "consistent/proper" means in the properties list above, with a worked proof for the Brier score
- [[Brier Score]] — the proper scoring rule used for probabilistic binary predictions, with a full bias-variance-style decomposition

---

## One-line Summary

> A loss function measures how wrong a single prediction is — it is the elementary building block from which the training objective is constructed, and its choice encodes the statistical assumptions about how errors are distributed.
