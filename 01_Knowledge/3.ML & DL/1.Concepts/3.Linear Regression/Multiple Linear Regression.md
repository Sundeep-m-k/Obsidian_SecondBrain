# Multiple Linear Regression

## What is it?

**Multiple Linear Regression** extends [[Linear Regression]] from one input feature to $d > 1$ features. The model is still linear in the parameters, but now each feature contributes to the prediction.

$$\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \cdots + \theta_d x_d$$

---

## Vector Form

With augmented feature vector (bias absorbed as $x_0 = 1$):

$$x = \begin{bmatrix}1\\x_1\\x_2\\\vdots\\x_d\end{bmatrix} \in \mathbb{R}^{d+1}, \quad \theta = \begin{bmatrix}\theta_0\\\theta_1\\\theta_2\\\vdots\\\theta_d\end{bmatrix} \in \mathbb{R}^{d+1}$$

$$\hat{y} = \theta^T x = \sum_{j=0}^{d} \theta_j x_j$$

---

## Matrix Form (All Training Examples)

$$\hat{y} = X\theta$$

$$X = \begin{bmatrix}1 & x_1^{(1)} & \cdots & x_d^{(1)} \\ 1 & x_1^{(2)} & \cdots & x_d^{(2)} \\ \vdots & & & \vdots \\ 1 & x_1^{(n)} & \cdots & x_d^{(n)}\end{bmatrix} \in \mathbb{R}^{n \times (d+1)}, \quad y = \begin{bmatrix}y^{(1)}\\\vdots\\y^{(n)}\end{bmatrix} \in \mathbb{R}^n$$

**Cost function:**
$$J(\theta) = \frac{1}{2n}\|X\theta - y\|^2 = \frac{1}{2n}(X\theta - y)^T(X\theta-y)$$

**Normal Equations solution:**
$$\hat{\theta} = (X^T X)^{-1} X^T y$$

**Gradient:**
$$\nabla_\theta J = \frac{1}{n} X^T(X\theta - y)$$

**Gradient descent update:**
$$\theta \leftarrow \theta - \frac{\eta}{n} X^T(X\theta - y)$$

---

## Interpretation of Coefficients

$\theta_j$ = the expected change in $\hat{y}$ for a one-unit increase in $x_j$, **holding all other features constant**.

This is the **partial regression coefficient** or **marginal effect** of feature $j$.

**Important:** This interpretation holds only when the features are truly independent. When features are correlated (**multicollinearity**), individual $\theta_j$ values become unstable and hard to interpret.

---

## Multicollinearity

When two or more features are highly correlated, $(X^TX)$ becomes ill-conditioned (near-singular), making $(X^TX)^{-1}$ unstable or undefined.

**Signs of multicollinearity:**
- Small changes in data cause large changes in $\hat{\theta}$
- High standard errors on coefficients
- VIF (Variance Inflation Factor) > 10: $VIF_j = \frac{1}{1 - R_j^2}$, where $R_j^2$ is the R² of regressing $x_j$ on all other features

**Fixes:** Remove one of the correlated features, use PCA to decorrelate, apply Ridge regression.

---

## Feature Scaling Is Essential

With multiple features on different scales, gradient descent is inefficient. Without scaling:
- The cost surface is an elongated ellipse
- Gradient descent oscillates and converges slowly

With standardised features ($\mu=0$, $\sigma=1$), the cost surface is more circular and gradient descent converges faster.

**Always standardise features before training multiple linear regression with gradient descent.**

---

## Connections

- [[Linear Regression]] — the single-feature special case
- [[Hypothesis Function]] — $h_\theta(x) = \theta^T x$
- [[Parameters]] — $\theta \in \mathbb{R}^{d+1}$
- [[Feature Scaling]] — required for efficient training
- [[Gradient Descent]] — optimises $\theta$
- [[Polynomial Regression]] — extends with non-linear features

---

## One-line Summary

> Multiple linear regression extends linear regression to $d$ features by fitting a hyperplane in $(d+1)$-dimensional space, where each coefficient represents the marginal effect of one feature on the output, solved exactly via normal equations or iteratively via gradient descent.
