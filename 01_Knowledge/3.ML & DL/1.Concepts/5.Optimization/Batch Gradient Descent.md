# Batch Gradient Descent

## What is it?

**Batch Gradient Descent** (also called **Full-Batch Gradient Descent**) is the variant of [[Gradient Descent]] that uses the **entire training set** to compute the gradient before each parameter update.

$$\theta \leftarrow \theta - \frac{\eta}{n}\sum_{i=1}^{n}\nabla_\theta \ell(y^{(i)}, \hat{f}(x^{(i)};\theta))$$

In matrix form:
$$\theta \leftarrow \theta - \eta \nabla_\theta J(\theta) = \theta - \frac{\eta}{n}X^T(X\theta - y)$$

---

## Comparison of GD Variants

| Variant | Batch Size | Gradient Noise | Speed per Update | Total Speed |
|---|---|---|---|---|
| **Batch GD** | $n$ (all data) | None (exact) | Slow | Slow for large $n$ |
| **Stochastic GD (SGD)** | 1 | Very high | Fast | Fast, noisy |
| **Mini-batch GD** | $b$ (e.g., 32–256) | Moderate | Moderate | **Best in practice** |

---

## When to Use Batch GD

✅ Small datasets ($n$ fits in memory)  
✅ Convex problems where exact gradient is valuable  
✅ When a smooth, deterministic training trajectory is needed  
❌ Large datasets — impractical  
❌ Neural networks — mini-batch is always preferred  

---

## One Epoch of Batch GD

```
For one epoch:
  Compute ŷ = Xθ for all n examples (one forward pass)
  Compute gradient = (1/n) Xᵀ(Xθ - y)
  Update θ ← θ - η × gradient
  (one gradient step per epoch)
```

One epoch = one gradient step for batch GD.  
One epoch = $\lceil n/b \rceil$ gradient steps for mini-batch GD.

---

## Connections

- [[Gradient Descent]] — batch GD is one variant
- [[Learning Rate]] — controls step size
- [[Cost Function]] — full dataset gradient of this
- [[Convergence]] — batch GD has smooth, guaranteed convergence for convex problems

---

## One-line Summary

> Batch gradient descent computes the exact gradient over the full training set before each update — producing stable, smooth convergence but becoming impractically slow for large datasets, which is why mini-batch variants are used in practice.
