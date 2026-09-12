# Gradient Descent

## What is it?

**Gradient Descent** is an iterative optimisation algorithm that minimises the [[Cost Function]] $J(\theta)$ by repeatedly updating parameters in the direction of the **negative gradient** — the direction of steepest descent.

$$\theta \leftarrow \theta - \eta \nabla_\theta J(\theta)$$

Where:
- $\eta > 0$ = **learning rate** (step size)
- $\nabla_\theta J(\theta)$ = gradient vector (direction of steepest increase)
- The **negative** gradient = direction of steepest decrease

---

## The Algorithm

```
Initialise θ (randomly or to zeros)
Repeat until convergence:
    Compute gradient: g = ∇_θ J(θ)
    Update parameters: θ ← θ - η · g
```

**Convergence criterion:** $\|g\| < \epsilon$ (gradient is small enough), or validation loss stops improving.

---

## The Gradient: What Is It?

The gradient $\nabla_\theta J \in \mathbb{R}^{d+1}$ is a vector where component $j$ is:

$$(\nabla_\theta J)_j = \frac{\partial J}{\partial \theta_j}$$

It points in the direction that most steeply increases $J$. Moving **opposite** to it decreases $J$.

**For MSE cost in linear regression:**
$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y) = \frac{1}{n}\sum_{i=1}^n (\hat{y}^{(i)} - y^{(i)})x^{(i)}$$

**Per-parameter form:**
$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^n (\hat{y}^{(i)} - y^{(i)}) x_j^{(i)}$$

**For logistic loss:**
$$\nabla_\theta J = \frac{1}{n}X^T(\hat{p} - y) = \frac{1}{n}\sum_{i=1}^n (\hat{p}^{(i)} - y^{(i)})x^{(i)}$$

Both have the same elegant form: $\text{(prediction error)} \times \text{(input feature)}$.

---

## Three Variants

### Batch Gradient Descent
Uses the **full training set** to compute each gradient update.

$$\theta \leftarrow \theta - \frac{\eta}{n}\sum_{i=1}^n \nabla_\theta \ell_i(\theta)$$

- Exact gradient → smooth, stable convergence
- Very slow per update for large $n$ (scans all $n$ examples per step)
- Deterministic: same path every run
- Guaranteed to converge for convex $J$

### Stochastic Gradient Descent (SGD)
Uses **one randomly chosen example** per update.

$$\theta \leftarrow \theta - \eta \nabla_\theta \ell^{(i)}(\theta) \quad \text{(random } i\text{)}$$

- Very fast per update
- Noisy gradient → erratic path, but noise can **escape local minima**
- High variance: may not converge to exact minimum, but oscillates near it
- Effective for large datasets

### Mini-Batch Gradient Descent
Uses a **small batch** of $b$ examples per update (typically $b = 32, 64, 128, 256$).

$$\theta \leftarrow \theta - \frac{\eta}{b}\sum_{i \in \mathcal{B}} \nabla_\theta \ell^{(i)}(\theta)$$

- Best of both worlds: less noisy than SGD, faster than batch
- **Standard in practice** for deep learning
- Batch size $b$ is a hyperparameter

**Comparing variance:** $\text{Var}(\text{mini-batch gradient}) = \frac{1}{b}\text{Var}(\text{single gradient})$. Larger batch → lower variance → more stable update.

---

## Learning Rate $\eta$: The Most Important Hyperparameter

$$\theta \leftarrow \theta - \eta \nabla_\theta J$$

| $\eta$ too small | $\eta$ too large |
|---|---|
| Slow convergence | Divergence or oscillation |
| Very many iterations | May overshoot minimum |
| Safe | Unstable training |

**Optimal $\eta$:** For quadratic cost, the optimal learning rate is $\eta^* = \frac{2}{\lambda_{\min} + \lambda_{\max}}$ where $\lambda$ are eigenvalues of the Hessian.

See [[Learning Rate]] for full treatment.

---

## Advanced Optimisers

### Momentum
Accumulate a velocity vector in the direction of persistent gradient:

$$v \leftarrow \beta v + (1-\beta)\nabla_\theta J$$
$$\theta \leftarrow \theta - \eta v$$

$\beta \approx 0.9$. Dampens oscillations, accelerates in consistent gradient directions.

### RMSProp
Adapts learning rate per parameter by dividing by a running average of squared gradients:

$$s \leftarrow \rho s + (1-\rho)(\nabla_\theta J)^2$$
$$\theta \leftarrow \theta - \frac{\eta}{\sqrt{s + \epsilon}} \nabla_\theta J$$

$\rho \approx 0.9$, $\epsilon \approx 10^{-8}$ (prevent division by zero). Handles non-stationary objectives.

### Adam (Adaptive Moment Estimation)
Combines momentum (first moment) and RMSProp (second moment). Standard for deep learning.

$$m_t \leftarrow \beta_1 m_{t-1} + (1-\beta_1)g_t \quad \text{(first moment / momentum)}$$
$$v_t \leftarrow \beta_2 v_{t-1} + (1-\beta_2)g_t^2 \quad \text{(second moment)}$$

**Bias correction** (needed because $m_0 = v_0 = 0$):
$$\hat{m}_t = \frac{m_t}{1-\beta_1^t}, \quad \hat{v}_t = \frac{v_t}{1-\beta_2^t}$$

**Update:**
$$\theta_t \leftarrow \theta_{t-1} - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon}\hat{m}_t$$

Default hyperparameters: $\beta_1 = 0.9$, $\beta_2 = 0.999$, $\epsilon = 10^{-8}$, $\eta = 0.001$.

**Why Adam works:** Parameters with large gradients (noisy dimensions) get smaller step sizes; parameters with small gradients get larger step sizes. Adapts to each dimension individually.

### AdamW
Adam + **decoupled weight decay** — applies L2 regularisation directly to weights, not through gradients. Often better than Adam for training transformers:

$$\theta_t \leftarrow \theta_{t-1} - \frac{\eta}{\sqrt{\hat{v}_t}+\epsilon}\hat{m}_t - \eta\lambda\theta_{t-1}$$

---

## Learning Rate Schedules

Instead of a fixed $\eta$, decrease it over time:

| Schedule | Formula | Notes |
|---|---|---|
| Step decay | $\eta \leftarrow \eta \cdot \gamma$ every $k$ epochs | Simple, widely used |
| Exponential decay | $\eta_t = \eta_0 e^{-kt}$ | Smooth decrease |
| Cosine annealing | $\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})(1+\cos(\pi t/T))$ | Smooth, popular for NNs |
| Warmup + decay | Linear increase then decrease | Standard for Transformers |
| ReduceLROnPlateau | Halve $\eta$ when val loss stops improving | Adaptive, model-agnostic |

**Warmup:** Start with very small $\eta$, increase linearly for the first few epochs. Prevents large updates when parameters are random.

---

## Why Gradient Descent Works: Intuition

Taylor expansion of $J$ around current $\theta$:
$$J(\theta + \Delta\theta) \approx J(\theta) + \nabla_\theta J^T \Delta\theta + \frac{1}{2}\Delta\theta^T H \Delta\theta$$

Setting $\Delta\theta = -\eta \nabla_\theta J$:
$$J(\theta - \eta\nabla J) \approx J(\theta) - \eta\|\nabla J\|^2 + \frac{\eta^2}{2}\nabla J^T H \nabla J$$

For small enough $\eta$: the $-\eta\|\nabla J\|^2$ term dominates and $J$ decreases. This guarantees descent for sufficiently small step sizes.

---

## Connections

- [[Gradient]] — the mathematical object used for updates
- [[Learning Rate]] — controls step size
- [[Convergence]] — when and why training stops
- [[Batch Gradient Descent]] — the full-data variant
- [[Cost Function]] — what is being minimised
- [[Cost Surface]] — the landscape being navigated
- [[Global Minimum]] — the goal
- [[Local Minimum]] — potential trap for non-convex surfaces

---

## One-line Summary

> Gradient descent minimises the cost function by repeatedly computing the gradient and taking a step opposite to it — the size of each step is controlled by the learning rate, and the variant (batch, SGD, mini-batch) determines the tradeoff between gradient accuracy and computational speed.
