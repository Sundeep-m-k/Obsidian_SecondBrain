# Convergence

## What is it?

**Convergence** in optimisation means that the training algorithm has reached (or is very close to) a minimum of the [[Cost Function]] — further iterations produce negligible improvement.

Formally, gradient descent has converged when:
$$\|\nabla_\theta J(\theta)\| < \epsilon \quad \text{(gradient is near zero)}$$
or
$$|J(\theta_{t+1}) - J(\theta_t)| < \epsilon \quad \text{(cost barely changes)}$$

---

## Convergence for Convex vs. Non-Convex Problems

### Convex Problems (Linear Regression, Logistic Regression)
- **Guaranteed to converge** to the global minimum with appropriate $\eta$.
- Convergence rate for gradient descent: $O(1/t)$ (sublinear).
- With strong convexity: **linear convergence** — error decreases by a constant factor each step: $J(\theta_t) - J^* \leq (1-\mu\eta)^t (J(\theta_0) - J^*)$

### Non-Convex Problems (Neural Networks)
- No guarantee of global minimum — may converge to a local minimum or saddle point.
- In practice, for large neural networks, local minima found by SGD are usually near-globally optimal (Dauphin et al., 2014).
- Saddle points are more problematic than local minima in high dimensions.

---

## Convergence Diagnostics

**Plot training loss vs. epoch/iteration:**

| Pattern | Meaning |
|---|---|
| Smoothly decreasing | Good convergence |
| Oscillating but decreasing | Learning rate slightly too high, but OK |
| Flat (no decrease) | Too small LR, stuck on plateau, or wrong gradient |
| Increasing | LR too large; diverging |
| Decreasing then increasing (val loss) | Overfitting |

---

## Epoch vs. Iteration

- **Iteration:** One gradient update (one mini-batch processed).
- **Epoch:** One full pass through the entire training set.
- Iterations per epoch = $\lceil n/b \rceil$ where $b$ = batch size.

---

## Connections

- [[Gradient Descent]] — the algorithm whose convergence we study
- [[Learning Rate]] — main factor controlling convergence
- [[Global Minimum]] — where we want to converge
- [[Local Minimum]] — where we might get stuck
- [[Cost Surface]] — the landscape being traversed

---

## One-line Summary

> Convergence means the optimisation has found a minimum where further updates produce no meaningful improvement — guaranteed for convex objectives, probabilistic for non-convex ones like neural networks.
