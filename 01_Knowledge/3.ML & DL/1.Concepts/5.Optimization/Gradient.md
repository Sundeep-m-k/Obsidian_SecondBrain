# Gradient

## What is it?

The **gradient** $\nabla_\theta J(\theta)$ is a vector of partial derivatives of the [[Cost Function]] with respect to each parameter. It points in the direction of **steepest increase** of $J$ at the current $\theta$.

$$\nabla_\theta J = \begin{bmatrix}\frac{\partial J}{\partial \theta_0} \\ \frac{\partial J}{\partial \theta_1} \\ \vdots \\ \frac{\partial J}{\partial \theta_d}\end{bmatrix} \in \mathbb{R}^{d+1}$$

---

## Geometric Meaning

- The gradient vector at point $\theta$ points in the direction that **most steeply increases** $J$.
- Its magnitude $\|\nabla_\theta J\|$ is the rate of increase in that direction.
- The **negative gradient** $-\nabla_\theta J$ points toward steepest decrease — this is the direction [[Gradient Descent]] moves.
- The gradient is **perpendicular** to contour lines of $J$.

---

## Key Gradients to Know

**MSE cost (linear regression):**
$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^n (\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}$$
$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y)$$

**Logistic loss (logistic regression):**
$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^n (\hat{p}^{(i)} - y^{(i)}) x_j^{(i)}$$
$$\nabla_\theta J = \frac{1}{n}X^T(\hat{p} - y)$$

**L2 regularisation term:**
$$\frac{\partial}{\partial \theta_j}\left(\frac{\lambda}{2}\sum_k \theta_k^2\right) = \lambda\theta_j$$

So regularised gradient = unregularised gradient + $\lambda\theta_j$.

---

## The Chain Rule and Backpropagation

For composed functions (neural networks), the gradient is computed via the **chain rule**:
$$\frac{\partial J}{\partial \theta} = \frac{\partial J}{\partial \hat{y}} \cdot \frac{\partial \hat{y}}{\partial \theta}$$

For a network with layers $a^{(1)}, \ldots, a^{(L)}$:
$$\frac{\partial J}{\partial W^{(l)}} = \frac{\partial J}{\partial a^{(L)}} \cdot \frac{\partial a^{(L)}}{\partial a^{(L-1)}} \cdots \frac{\partial a^{(l+1)}}{\partial a^{(l)}} \cdot \frac{\partial a^{(l)}}{\partial W^{(l)}}$$

This chain of multiplications is **backpropagation** — computed efficiently via dynamic programming (compute once, reuse).

---

## Connections

- [[Gradient Descent]] — uses the gradient to update parameters
- [[Cost Function]] — the function being differentiated
- [[Parameters]] — what the gradient is computed with respect to
- [[Cost Surface]] — the gradient is the slope of this surface

---

## One-line Summary

> The gradient is the vector of partial derivatives of the cost with respect to all parameters — it points toward the steepest increase in cost, so gradient descent moves in the opposite direction to reduce cost.
