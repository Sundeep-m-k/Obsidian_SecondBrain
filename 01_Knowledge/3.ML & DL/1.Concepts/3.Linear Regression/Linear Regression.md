# Linear Regression

## What is it?

**Linear Regression** is the foundational [[Supervised Learning]] algorithm for predicting a **continuous** output. It models the relationship between input features and output as a **linear function**.

$$\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \cdots + \theta_d x_d = \theta^T x$$

---

## The Model

With features $x \in \mathbb{R}^d$ and an appended bias term $x_0 = 1$:

$$\hat{y} = \theta^T x = \sum_{j=0}^{d} \theta_j x_j$$

Where $\theta = [\theta_0, \theta_1, \ldots, \theta_d]^T \in \mathbb{R}^{d+1}$ are the parameters (weights).

**Geometric interpretation:**
- $d=1$: a **line** in 2D
- $d=2$: a **plane** in 3D
- $d>2$: a **hyperplane** in $(d+1)$-dimensional space

---

## Training: Minimise MSE

**Cost function (MSE over $n$ training examples):**
$$J(\theta) = \frac{1}{2n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)})^2 = \frac{1}{2n}\|X\theta - y\|^2$$

(The $\frac{1}{2}$ is a convention to simplify the gradient formula.)

**Matrix form:**

$$J(\theta) = \frac{1}{2n}(X\theta - y)^T(X\theta - y)$$

Where:
- $X \in \mathbb{R}^{n \times (d+1)}$ = design matrix (each row is one training example)
- $y \in \mathbb{R}^n$ = vector of labels
- $\theta \in \mathbb{R}^{d+1}$ = parameter vector

---

## Solution 1: Normal Equations (Closed Form)

Set the gradient to zero and solve:

$$\nabla_\theta J = \frac{1}{n} X^T(X\theta - y) = 0$$

$$X^T X \theta = X^T y$$

$$\boxed{\hat{\theta} = (X^T X)^{-1} X^T y}$$

This is the **Normal Equation** — a direct, exact solution.

**$(X^T X)^{-1} X^T$** is called the **Moore-Penrose pseudoinverse** of $X$, denoted $X^+$.

**When is $(X^T X)$ invertible?**
- Requires $n \geq d+1$ (more examples than features)
- Requires no perfect multicollinearity (no feature is a linear combination of others)

**Computational cost:** $O(d^3)$ to invert a $(d+1) \times (d+1)$ matrix — prohibitively expensive for large $d$. Use gradient descent for large $d$.

**Pros:** Exact solution in one step. No hyperparameter (learning rate).  
**Cons:** $O(d^3)$ — doesn't scale to many features.

---

## Solution 2: Gradient Descent

Iteratively update parameters in the direction of the negative gradient:

$$\theta \leftarrow \theta - \eta \nabla_\theta J(\theta)$$

**Gradient computation:**
$$\nabla_\theta J(\theta) = \frac{1}{n} X^T(X\theta - y) = \frac{1}{n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)}) x^{(i)}$$

**Per-parameter update:**
$$\theta_j \leftarrow \theta_j - \eta \cdot \frac{1}{n}\sum_{i=1}^{n}(\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}, \quad j = 0, 1, \ldots, d$$

The cost surface $J(\theta)$ for linear regression is **convex** (a bowl shape) — gradient descent is guaranteed to find the **global minimum**.

---

## Probabilistic View

Linear regression assumes:
$$y^{(i)} = \theta^T x^{(i)} + \epsilon^{(i)}, \quad \epsilon^{(i)} \sim \mathcal{N}(0, \sigma^2)$$

**Likelihood of the data:**
$$p(y \mid X, \theta) = \prod_{i=1}^n \frac{1}{\sqrt{2\pi}\sigma}\exp\left(-\frac{(y^{(i)} - \theta^T x^{(i)})^2}{2\sigma^2}\right)$$

**Log-likelihood:**
$$\log p(y \mid X, \theta) = -\frac{n}{2}\log(2\pi\sigma^2) - \frac{1}{2\sigma^2}\sum_{i=1}^n (y^{(i)} - \theta^T x^{(i)})^2$$

**Maximising log-likelihood = minimising MSE.** This is why MSE is the natural loss for regression when errors are Gaussian.

---

## Key Metrics

| Metric | Formula | Interpretation |
|---|---|---|
| MSE | $\frac{1}{n}\sum(y-\hat{y})^2$ | Average squared error |
| RMSE | $\sqrt{MSE}$ | In same units as $y$ |
| MAE | $\frac{1}{n}\sum|y-\hat{y}|$ | Average absolute error |
| R² | $1 - \frac{SS_{res}}{SS_{tot}}$ | 1 = perfect, 0 = predicts mean |

---

## Assumptions (Gauss-Markov)

For OLS to be BLUE (Best Linear Unbiased Estimator):
1. $y = \theta^T x + \epsilon$ (linearity)
2. $\mathbb{E}[\epsilon] = 0$ (zero-mean errors)
3. $\text{Var}(\epsilon^{(i)}) = \sigma^2$ for all $i$ (homoscedasticity)
4. $\text{Cov}(\epsilon^{(i)}, \epsilon^{(j)}) = 0$ for $i \neq j$ (independent errors)
5. No perfect multicollinearity in $X$

---

## When to Use Linear Regression

✅ Output is continuous  
✅ Relationship between features and output is approximately linear  
✅ Interpretability is important (each $\theta_j$ = marginal effect of feature $j$)  
✅ Data volume is small to medium  
❌ Non-linear relationship → use polynomial regression or non-linear model  
❌ Output is categorical → use logistic regression  

---

## Connections

- [[Model Representation]] — how the linear model is written
- [[Hypothesis Function]] — $\hat{y} = \theta^T x$
- [[Parameters]] — the $\theta$ vector
- [[Multiple Linear Regression]] — with $d > 1$ features
- [[Polynomial Regression]] — non-linear extension
- [[Mean Squared Error]] — the loss minimised during training
- [[Gradient Descent]] — the iterative optimisation method
- [[Regression]] — the general task

---

## One-line Summary

> Linear regression finds the hyperplane that best fits the training data by minimising the mean squared error, either via a closed-form solution (Normal Equations) or iterative gradient descent, under the assumption that the true relationship is linear plus Gaussian noise.
