# Decision Boundary in Logistic Regression

## What is it?

The **decision boundary** in logistic regression is the set of input points where the predicted probability equals the threshold — the surface that separates predicted class 1 from predicted class 0.

With default threshold $\tau = 0.5$:
$$\text{Decision boundary} = \{x : \hat{p}(x) = 0.5\} = \{x : \sigma(\theta^T x) = 0.5\} = \{x : \theta^T x = 0\}$$

---

## It's Always a Linear Hyperplane

Since $\sigma(z) = 0.5$ iff $z = 0$:

$$\theta^T x = 0 \quad \Leftrightarrow \quad \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d = 0$$

This is a **linear equation** in $x$ — a hyperplane.

- In 2D ($d=2$): a **line** $\theta_1 x_1 + \theta_2 x_2 + \theta_0 = 0$
- In 3D ($d=3$): a **plane**
- In $d$ dimensions: a **hyperplane**

**The parameters $\theta$ define the orientation and position of this hyperplane:**
- $\theta_0$: shifts the boundary (bias term)
- $[\theta_1, \ldots, \theta_d]$: normal vector to the boundary

---

## Non-Linear Decision Boundaries via Feature Engineering

Logistic regression is a linear classifier in the original feature space, but adding polynomial features creates non-linear boundaries.

**Example:** With features $[1, x_1, x_2, x_1^2, x_2^2, x_1 x_2]$ and parameters $\theta$, the boundary:
$$\theta_0 + \theta_1 x_1 + \theta_2 x_2 + \theta_3 x_1^2 + \theta_4 x_2^2 + \theta_5 x_1 x_2 = 0$$

This is a **conic section** (circle, ellipse, parabola, or hyperbola) in the original $x_1, x_2$ space — but still a hyperplane in the 6-dimensional feature space.

---

## Probability Contours Around the Boundary

The probability varies smoothly:
- On the boundary: $\hat{p} = 0.5$ (indifferent)
- Moving perpendicular to boundary toward class 1 region: $\hat{p} \to 1$
- Moving toward class 0 region: $\hat{p} \to 0$

**Rate of change:** Controlled by $\|\theta\|$. Larger $\|\theta\|$ → steeper sigmoid → sharper boundary transition → more confident predictions.

---

## Connections

- [[Logistic Regression]] — the model that creates this boundary
- [[Decision Boundary]] — general concept
- [[Sigmoid Function]] — the function that equals 0.5 at the boundary
- [[Thresholding]] — the threshold determines where the boundary is
- [[Log Odds]] — boundary is where log-odds = 0

---

## One-line Summary

> The decision boundary in logistic regression is the hyperplane where the linear score $\theta^T x = 0$ — the model predicts class 1 on one side and class 0 on the other, with the boundary always being linear in the (possibly transformed) feature space.
