# Multiple Features

## What is it?

**Multiple features** (also called **multivariate input**) means that each training example is described by more than one measurement. With $d$ features, the input is a vector in $\mathbb{R}^d$.

$$x = [x_1, x_2, \ldots, x_d]^T \in \mathbb{R}^d$$

This is the standard setting in virtually all real ML problems.

---

## Notation

| Symbol | Meaning |
|---|---|
| $d$ | Number of features (dimensionality) |
| $x^{(i)}$ | Feature vector of the $i$-th training example |
| $x_j^{(i)}$ | Value of feature $j$ for example $i$ |
| $X$ | Design matrix $\in \mathbb{R}^{n \times d}$ (row = example, column = feature) |

**Design matrix:**
$$X = \begin{bmatrix} x_1^{(1)} & x_2^{(1)} & \cdots & x_d^{(1)} \\ x_1^{(2)} & x_2^{(2)} & \cdots & x_d^{(2)} \\ \vdots & & & \vdots \\ x_1^{(n)} & x_2^{(n)} & \cdots & x_d^{(n)} \end{bmatrix}$$

With the bias column prepended (for linear models):
$$X \leftarrow \begin{bmatrix} 1 & x_1^{(1)} & \cdots & x_d^{(1)} \\ \vdots & & & \vdots \end{bmatrix} \in \mathbb{R}^{n \times (d+1)}$$

---

## Example

Predicting house price from multiple features:

| Example | Size ($x_1$) | Bedrooms ($x_2$) | Age ($x_3$) | Price ($y$) |
|---|---|---|---|---|
| 1 | 1500 | 3 | 20 | 320,000 |
| 2 | 2400 | 4 | 5 | 510,000 |
| 3 | 900 | 2 | 40 | 180,000 |

Model: $\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \theta_3 x_3$

With $\theta = [−100{,}000,\ 200,\ 15{,}000,\ −1{,}000]^T$, the prediction for example 1 is:
$$\hat{y}^{(1)} = -100{,}000 + 200(1500) + 15{,}000(3) + (-1{,}000)(20) = 325{,}000$$

---

## Implications of Multiple Features

1. **Model expressiveness:** More features allow the model to capture more complex relationships.
2. **Feature scaling becomes critical:** Features on different scales cause gradient descent to oscillate.
3. **Curse of dimensionality:** Too many features with too few examples → overfitting.
4. **Multicollinearity:** Correlated features make coefficients unstable.

---

## Connections

- [[Feature Vector]] — the compact representation of multiple features
- [[Multiple Linear Regression]] — linear model with multiple features
- [[Feature Scaling]] — essential for gradient-based methods with multiple features
- [[Feature Engineering]] — how to create and transform features

---

## One-line Summary

> Multiple features means each data point is described by a vector of measurements — the standard setting in ML, where the design matrix organises all examples and features into a compact matrix form that enables efficient vectorised computation.
