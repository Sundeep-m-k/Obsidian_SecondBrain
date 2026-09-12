# How to Choose Learning Rate

## Why It Matters

The learning rate $\eta$ is the single most important hyperparameter in gradient-based training. Too small → painfully slow. Too large → diverges. Getting it right is essential.

---

## Visual Diagnostic: Loss Curves

Always plot training loss vs. epoch/iteration:

| Loss curve shape | Diagnosis | Action |
|---|---|---|
| Smoothly decreasing to low value | $\eta$ is good ✅ | Keep it |
| Decreasing very slowly | $\eta$ too small | Increase $\eta$ by 10× |
| Oscillating but overall decreasing | $\eta$ slightly too high | Decrease by 2–5× |
| Immediately diverging (loss → ∞ or NaN) | $\eta$ way too large | Decrease by 100× |
| Loss flat from the start | $\eta$ too small OR wrong gradient | Check implementation, try higher $\eta$ |
| Loss decreases then plateaus | Might need decay schedule | Add LR schedule or reduce LR |

---

## Method 1: Learning Rate Range Test (Most Reliable)

(Smith, 2017 — "Cyclical Learning Rates")

**Procedure:**
1. Start with tiny $\eta_{\min} = 10^{-7}$.
2. Increase $\eta$ exponentially over one epoch: $\eta_t = \eta_{\min} \cdot (\eta_{\max}/\eta_{\min})^{t/T}$.
3. Plot loss vs. $\log(\eta)$.
4. Choose $\eta$ slightly before the loss reaches its minimum — the **steepest descent region**.

```
Loss
  │────╮
  │    ╲         ╭────────
  │     ╲       ╱
  │      ╲─────╱
  └────────────────── log(η)
    1e-7  1e-5 1e-3  1e-1
              ↑
         Choose here (steepest slope)
```

This method is used by fast.ai's one-cycle policy and is highly reliable.

---

## Method 2: Grid Search on Log Scale

Try a set of log-spaced values:
$$\eta \in \{10^{-5},\ 10^{-4},\ 10^{-3},\ 10^{-2},\ 10^{-1}\}$$

Train for a few epochs each; pick the one with lowest validation loss. Quick and sufficient for most problems.

---

## Method 3: Good Defaults by Algorithm

| Algorithm | Default $\eta$ | Notes |
|---|---|---|
| Linear/Logistic Regression (GD) | 0.01–0.1 | Scale with batch size |
| SGD (deep learning) | 0.01–0.1 | Needs careful tuning; use momentum |
| **Adam** | **0.001** | Works for most deep learning problems |
| AdamW | 0.001 | Standard for Transformers |
| RMSProp | 0.001 | Default for RNNs |

**Start with Adam at $\eta = 10^{-3}$** for neural networks — works surprisingly often without further tuning.

---

## Learning Rate Schedules

After finding a good initial $\eta$, decay it during training:

**Cosine annealing:**
$$\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})\left(1 + \cos\frac{\pi t}{T}\right)$$

**Warmup + cosine decay (standard for Transformers):**
- Linear warmup for $W$ steps: $\eta_t = \eta_{\max} \cdot t/W$
- Cosine decay from $\eta_{\max}$ to 0 for remaining steps

**ReduceLROnPlateau:**
- Monitor val loss; reduce $\eta$ by factor 0.5 if no improvement for $P=5$ epochs
- Hands-off, works well for traditional ML

---

## The Linear Scaling Rule

If you multiply batch size by $k$, multiply $\eta$ by $k$:
$$\eta_{\text{new}} = k \cdot \eta_{\text{base}}$$

Intuition: larger batch = less noisy gradient = can take proportionally larger steps.

**Works for:** Linear scaling up to batch sizes ~8000. With very large batches, use linear warmup to stability.

---

## Quick Checklist

```
□ Is loss decreasing at all?  → If no, try 10× larger η
□ Is loss oscillating?         → Try 3× smaller η
□ Is loss decreasing but slow? → Try 5× larger η
□ Is it a new architecture?    → Run LR range test
□ Using Adam?                  → Start at 1e-3, rarely need to change
□ Training for many epochs?    → Add a decay schedule
□ Large batch size?            → Scale η linearly with batch size
```

---

## Connections

- [[Learning Rate]] — the concept
- [[Gradient Descent]] — the algorithm $\eta$ controls
- [[Convergence]] — right $\eta$ ensures convergence
- [[Cost Surface]] — $\eta$ is the step size on this surface

---

## One-line Summary

> Choose the learning rate by running a range test (increase $\eta$ exponentially, pick the value at steepest loss decrease), default to Adam at 1e-3 for neural networks, and always add a decay schedule for long training runs — diagnosing via the loss curve shape.
