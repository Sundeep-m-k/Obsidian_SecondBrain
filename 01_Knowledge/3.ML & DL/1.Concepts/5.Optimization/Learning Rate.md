# Learning Rate

## What is it?

The **learning rate** $\eta$ (eta) is a **hyperparameter** that controls the size of each step [[Gradient Descent]] takes when updating parameters.

$$\theta \leftarrow \theta - \eta \nabla_\theta J(\theta)$$

It is arguably the single most important hyperparameter in training a machine learning model.

---

## Effect of Learning Rate

| $\eta$ | Effect |
|---|---|
| **Too small** | Convergence is very slow; many epochs needed; may not reach minimum before compute budget runs out |
| **Just right** | Smooth, efficient convergence to the minimum |
| **Too large** | Oscillates around or diverges away from minimum; loss may increase |
| **Way too large** | Loss explodes (goes to $\infty$ or NaN) |

**Mathematical condition:** For gradient descent to guarantee decrease, $\eta$ must satisfy:
$$\eta < \frac{2}{\lambda_{\max}}$$

where $\lambda_{\max}$ is the largest eigenvalue of the Hessian $H = \nabla^2_\theta J$. Larger curvature → smaller $\eta$ needed.

---

## Optimal Learning Rate

For a quadratic cost surface (the case for linear regression), the theoretically optimal learning rate is:
$$\eta^* = \frac{2}{\lambda_{\min}(H) + \lambda_{\max}(H)}$$

This minimises the number of steps to convergence. In practice, estimated via learning rate range tests.

---

## Learning Rate Schedules

Using a fixed $\eta$ throughout training is suboptimal. Schedules adapt $\eta$ over time:

**Step decay:**
$$\eta_t = \eta_0 \cdot \gamma^{\lfloor t/k \rfloor}$$
Drop $\eta$ by factor $\gamma$ (e.g., 0.1) every $k$ epochs.

**Exponential decay:**
$$\eta_t = \eta_0 \cdot e^{-\lambda t}$$

**Cosine annealing:**
$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\frac{\pi t}{T}\right)$$
Smooth decrease from $\eta_{\max}$ to $\eta_{\min}$ over $T$ steps. Popular for neural networks.

**Warmup:**
Linearly increase $\eta$ from 0 to $\eta_{\max}$ over the first $W$ steps, then decay.
$$\eta_t = \eta_{\max} \cdot \min\left(\frac{t}{W},\ \text{decay}(t)\right)$$
Essential for training Transformers. Prevents large destructive updates early when parameters are random.

**ReduceLROnPlateau:**
Monitor validation loss. If it hasn't improved for $P$ epochs (patience), multiply $\eta$ by $\gamma$ (e.g., 0.5). Fully adaptive.

---

## How to Choose the Learning Rate

### Learning Rate Range Test (Smith, 2017)
1. Start with a tiny $\eta$ (e.g., $10^{-7}$).
2. Increase $\eta$ exponentially over a short run (e.g., one epoch).
3. Plot loss vs. $\eta$.
4. Choose $\eta$ slightly before the loss starts increasing — the steepest descent region.

```
Loss
  │
  │────╮
  │     ╲         ╭────
  │      ╲       ╱
  │       ╲─────╱
  └────────────────── log(η)
            ↑
       Choose here (steepest descent)
```

### Grid Search / Random Search
Try a logarithmically-spaced set of values: $\{10^{-5}, 10^{-4}, 10^{-3}, 10^{-2}, 10^{-1}\}$. Evaluate each on validation set. Choose the best.

### Typical Defaults
- Adam: $\eta = 10^{-3}$ (default in most frameworks)
- SGD: $\eta = 10^{-2}$ to $10^{-1}$ (with momentum)
- Transformers: $\eta = 10^{-4}$ with warmup

---

## Learning Rate vs. Batch Size

Empirically, if you multiply batch size by $k$, you should also multiply $\eta$ by $k$ (linear scaling rule) to maintain similar training dynamics. Intuition: larger batch = less noisy gradient = can take larger steps.

---

## Connections

- [[Gradient Descent]] — learning rate is its core hyperparameter
- [[Convergence]] — learning rate determines whether and how fast GD converges
- [[Cost Surface]] — learning rate determines step size on this surface
- [[Global Minimum]] — right learning rate finds it efficiently

---

## One-line Summary

> The learning rate controls how large each gradient descent step is — too small means painfully slow training, too large means divergence, and the right value (often found via range test or schedule) is the most impactful single tuning decision in training any ML model.
