# Model Representation

## What is it?

**Model representation** refers to how we mathematically write down the structure of a [[Model]] — the functional form that maps inputs to outputs. It specifies what family of functions the model belongs to.

For [[Linear Regression]]:
$$\hat{y} = h_\theta(x) = \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d$$

This is the **hypothesis function** — a linear combination of the input features.

---

## General Form

Any ML model can be written as:
$$\hat{y} = h_\theta(x)$$

Where:
- $h$ = the **hypothesis function** (the structural form — e.g., linear, polynomial, neural net)
- $\theta$ = the **parameters** (the adjustable values)
- $x$ = the **input features**
- $\hat{y}$ = the **prediction**

The model representation separates the **structure** (architecture, which is fixed before training) from the **parameters** (which are learned from data).

---

## Compact Vector Form

With bias absorbed into the feature vector by appending $x_0 = 1$:

$$x = \begin{bmatrix}1 \\ x_1 \\ x_2 \\ \vdots \\ x_d\end{bmatrix} \in \mathbb{R}^{d+1}, \quad \theta = \begin{bmatrix}\theta_0 \\ \theta_1 \\ \theta_2 \\ \vdots \\ \theta_d\end{bmatrix} \in \mathbb{R}^{d+1}$$

$$\hat{y} = \theta^T x = \sum_{j=0}^{d} \theta_j x_j$$

This dot-product form is compact and efficient for computation.

---

## Design Matrix Form (All Training Examples)

For $n$ examples simultaneously:
$$\hat{y} = X\theta$$

Where $X \in \mathbb{R}^{n \times (d+1)}$ is the **design matrix**:
$$X = \begin{bmatrix}1 & x_1^{(1)} & x_2^{(1)} & \cdots & x_d^{(1)} \\ 1 & x_1^{(2)} & x_2^{(2)} & \cdots & x_d^{(2)} \\ \vdots & & & & \vdots \\ 1 & x_1^{(n)} & x_2^{(n)} & \cdots & x_d^{(n)}\end{bmatrix}$$

Row $i$ is the feature vector of the $i$-th training example (including the bias 1).

---

## Connections

- [[Hypothesis Function]] — the specific functional form
- [[Parameters]] — the $\theta$ values to be learned
- [[Linear Regression]] — uses a linear model representation
- [[Multiple Linear Regression]] — extends to $d$ features

---

## One-line Summary

> Model representation is the mathematical skeleton of a model — the functional form $h_\theta(x)$ that specifies what family of functions the model belongs to, before any parameters are learned.
