# Cost Function

## What is it?

The **cost function** $J(\theta)$ is the **average loss over the entire training set**. It is the single scalar value that training tries to minimise by adjusting the model's [[Parameters]] $\theta$.

$$J(\theta) = \frac{1}{n}\sum_{i=1}^{n} \ell\!\left(y^{(i)},\ \hat{f}(x^{(i)};\theta)\right)$$

Where:
- $n$ = number of training examples
- $\ell$ = the [[Loss Function]] for one example
- $\hat{f}(x^{(i)};\theta)$ = model prediction for example $i$

---

## Loss vs. Cost vs. Objective

These three terms are often used interchangeably but have a precise hierarchy:

| Term | Scope | Formula |
|---|---|---|
| **Loss** $\ell$ | One example | $\ell(y^{(i)}, \hat{y}^{(i)})$ |
| **Cost** $J$ | Full training set | $J(\theta) = \frac{1}{n}\sum_i \ell_i$ |
| **Objective** | What we minimise | May include regularisation: $J(\theta) + \lambda\Omega(\theta)$ |

---

## Cost Function for Linear Regression (MSE)

$$J(\theta) = \frac{1}{2n}\sum_{i=1}^{n}\left(\hat{y}^{(i)} - y^{(i)}\right)^2 = \frac{1}{2n}\|X\theta - y\|^2$$

The $\frac{1}{2}$ factor is a convention: it cancels the 2 that appears when differentiating, giving a clean gradient:

$$\nabla_\theta J(\theta) = \frac{1}{n}X^T(X\theta - y)$$

**Shape of the cost surface:** For linear regression, $J(\theta)$ is a **convex paraboloid** (bowl shape) in parameter space. There is exactly **one global minimum** — no local minima. Gradient descent is guaranteed to find it.

---

## Cost Function for Logistic Regression (Log Loss)

$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log \hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

Where $\hat{p}^{(i)} = \sigma(\theta^T x^{(i)})$.

This is also convex in $\theta$ (no local minima) — proven by the fact that the negative log-likelihood of a logistic model is convex.

**Why not use MSE for classification?**  
If you use $J(\theta) = \frac{1}{n}\sum(\hat{p}^{(i)} - y^{(i)})^2$ with sigmoid output, the cost surface becomes **non-convex** — full of local minima that make training unreliable. Cross-entropy avoids this.

---

## Cost Surface Geometry

The cost surface is a plot of $J(\theta)$ over all possible parameter values.

| Model | Cost surface | Implication |
|---|---|---|
| Linear regression (MSE) | Convex bowl | One global minimum, GD always converges |
| Logistic regression (cross-entropy) | Convex | One global minimum |
| Neural network | Non-convex | Many local minima, saddle points |

**For neural networks**, the cost surface is extremely high-dimensional and non-convex. However, empirically:
- Local minima in high dimensions tend to have similar loss values to the global minimum.
- Saddle points (gradient = 0, not a minimum) are more problematic than local minima.
- SGD with noise helps escape saddle points.

---

## Regularised Cost Function

Adding a regularisation term to prevent overfitting:

$$J_{\text{reg}}(\theta) = \underbrace{\frac{1}{n}\sum_{i=1}^{n}\ell(y^{(i)}, \hat{f}(x^{(i)};\theta))}_{\text{data fit term}} + \underbrace{\lambda \Omega(\theta)}_{\text{regularisation term}}$$

- **L2 (Ridge):** $\Omega(\theta) = \frac{1}{2}\sum_{j=1}^{d}\theta_j^2 = \frac{1}{2}\|\theta\|^2$ (don't penalise $\theta_0$)
- **L1 (Lasso):** $\Omega(\theta) = \sum_{j=1}^{d}|\theta_j|$
- $\lambda > 0$ controls the regularisation strength (hyperparameter)

---

## Decomposing Cost Into Expected Error

$$J(\theta) = \underbrace{\text{Bias}^2}_{\text{model too simple}} + \underbrace{\text{Variance}}_{\text{model too complex}} + \underbrace{\sigma^2_\epsilon}_{\text{irreducible noise}}$$

The cost on unseen data (generalisation error) decomposes this way. The training cost only reflects the data fit — it doesn't tell you about bias or variance directly.

---

## Connections

- [[Loss Function]] — loss is the per-example version
- [[Objective Function]] — cost + regularisation
- [[Mean Squared Error]] — specific cost for regression
- [[Logistic Loss]] — specific cost for classification
- [[Gradient Descent]] — minimises the cost
- [[Cost Surface]] — visual geometry of $J(\theta)$
- [[Regularization]] — adds a term to cost to prevent overfitting

---

## One-line Summary

> The cost function is the average loss across all training examples — it is the single number that training minimises, and its geometry (convex vs. non-convex) determines whether optimisation is straightforward or challenging.
