# Cost Surface

## What is it?

The **cost surface** (also called the **loss landscape** or **error surface**) is the function $J(\theta)$ visualised as a surface over the parameter space. Each point on the surface represents a specific parameter setting and its corresponding cost value.

For a model with 2 parameters $(\theta_0, \theta_1)$, the cost surface is a 3D surface:
$$(\theta_0, \theta_1) \mapsto J(\theta_0, \theta_1)$$

For more parameters, it's a hypersurface in $(d+2)$-dimensional space — impossible to visualise directly, but the same geometric concepts apply.

---

## Geometry of Cost Surfaces

### Convex Surface (Linear Regression with MSE)

```
J(θ)
  │
  │        ╭───╮
  │      ╱       ╲
  │    ╱           ╲
  │  ╱       *       ╲   * = global minimum
  │──────────────────────── θ
```

**Properties:**
- Exactly one global minimum
- No local minima
- Any starting point → gradient descent converges to global minimum
- Contour lines (level sets) are ellipses

**Condition for convexity:** $J(\theta)$ is convex if and only if $\nabla^2_\theta J \succeq 0$ (the Hessian is positive semi-definite).

For MSE cost of linear regression:
$$\nabla^2_\theta J = \frac{1}{n}X^TX$$
This is always positive semi-definite (PSD), confirming convexity.

### Non-Convex Surface (Neural Networks)

```
J(θ)
  │   ╭─╮       ╭─╮
  │  ╱   ╲     ╱   ╲
  │ ╱  ○  ╲   ╱     ╲
  │╱        ╲╱   *    ╲   ○ = local min, * = global min
  └──────────────────── θ
```

Neural network cost surfaces are highly non-convex with:
- **Local minima:** gradient = 0, but not the lowest point
- **Saddle points:** gradient = 0, but it's a minimum in some directions and a maximum in others
- **Flat regions (plateaus):** gradient ≈ 0, but far from minimum — training slows down
- **Steep cliffs:** gradient suddenly very large — causes gradient explosion

---

## Contour Plots

A **contour plot** shows level sets of $J(\theta)$ — curves where $J$ takes a constant value. Gradient descent steps are always **perpendicular to contour lines** (in the direction of steepest descent).

**Well-scaled features:** Contours are roughly circular → gradient points directly toward minimum → fast convergence.

**Poorly-scaled features:** Contours are elongated ellipses → gradient points toward the edge of the valley, not the bottom → oscillation and slow convergence.

```
Well-scaled (circular):     Poorly-scaled (elliptical):
  θ₁                          θ₁
   │   ○○○                      │────────────
   │  ○   ○                     │──────────
   │ ○  *  ○     * = min        │──────  * ←   
   │  ○   ○                     │──────────
   │   ○○○                      │────────────
   └────── θ₀                   └────── θ₀

 GD: straight path to min    GD: zigzags slowly
```

---

## Key Points and Their Definitions

**Global minimum:** $\theta^*$ such that $J(\theta^*) \leq J(\theta)$ for all $\theta$.

**Local minimum:** $\theta^*$ such that $J(\theta^*) \leq J(\theta)$ for all $\theta$ in a neighbourhood of $\theta^*$.

**Saddle point:** $\nabla_\theta J = 0$ but is neither a local min nor max. Gradient is zero, but moving in different directions increases or decreases $J$.

**Plateau:** Extended region where $\|\nabla_\theta J\| \approx 0$. Gradient descent moves very slowly through plateaus.

---

## The Hessian Matrix

The **Hessian** $H = \nabla^2_\theta J$ (matrix of second derivatives) characterises the local curvature of the cost surface:

$$H_{jk} = \frac{\partial^2 J}{\partial \theta_j \partial \theta_k}$$

At a critical point ($\nabla J = 0$):
- $H \succ 0$ (all eigenvalues positive): **local minimum**
- $H \prec 0$ (all eigenvalues negative): **local maximum**
- $H$ has both positive and negative eigenvalues: **saddle point**

The eigenvalues of $H$ also determine how fast gradient descent converges:
- The ratio $\kappa = \lambda_{\max}/\lambda_{\min}$ (condition number) determines elongation of contours
- High $\kappa$ → very elongated → slow convergence
- Well-scaled features → $\kappa$ closer to 1 → fast convergence

---

## Practical Implications

| Observation during training | What it means | Fix |
|---|---|---|
| Loss decreasing steadily | Gradient descent working correctly | — |
| Loss oscillating wildly | Learning rate too large | Reduce $\eta$ |
| Loss barely moving | On a plateau, or learning rate too small | Adjust $\eta$, use momentum |
| Loss decreases then levels off | Stuck in local minimum or saddle | Random restarts, use Adam |
| Loss explodes (→ NaN) | Gradient explosion | Gradient clipping, reduce $\eta$ |

---

## Connections

- [[Cost Function]] — the function being visualised
- [[Gradient Descent]] — navigates the cost surface
- [[Learning Rate]] — controls step size on the surface
- [[Global Minimum]] — the best point on the surface
- [[Local Minimum]] — sub-optimal stopping points
- [[Convergence]] — reaching a minimum on the surface

---

## One-line Summary

> The cost surface is the landscape of all possible costs over parameter space — its geometry (convex bowl for linear models, rugged terrain for neural nets) determines how easily gradient descent can find the minimum and how many restarts may be needed.
