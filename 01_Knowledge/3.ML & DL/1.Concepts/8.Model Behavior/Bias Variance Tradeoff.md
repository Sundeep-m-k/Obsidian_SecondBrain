# Bias Variance Tradeoff

## What is it?

The **bias-variance tradeoff** is the fundamental tension in supervised learning: as you increase model complexity, bias decreases but variance increases. There is an optimal complexity that minimises total error.

$$\text{Expected MSE} = \text{Bias}^2 + \text{Variance} + \sigma_\epsilon^2$$

---

## Full Derivation

For a fixed point $x$, true label $y = f(x) + \epsilon$ where $\epsilon \sim \mathcal{N}(0, \sigma_\epsilon^2)$:

$$\mathbb{E}_{\mathcal{D},\epsilon}\left[(y - \hat{f}(x))^2\right]$$

$$= \mathbb{E}\left[(f(x) + \epsilon - \hat{f}(x))^2\right]$$

$$= \mathbb{E}\left[(f(x) - \hat{f}(x))^2\right] + 2\mathbb{E}[\epsilon]\mathbb{E}[f(x)-\hat{f}(x)] + \mathbb{E}[\epsilon^2]$$

Since $\mathbb{E}[\epsilon] = 0$ and $\mathbb{E}[\epsilon^2] = \sigma_\epsilon^2$:

$$= \mathbb{E}\left[(f(x) - \hat{f}(x))^2\right] + \sigma_\epsilon^2$$

Expanding the first term by adding and subtracting $\mathbb{E}[\hat{f}(x)]$:

$$= \underbrace{\left(f(x) - \mathbb{E}[\hat{f}(x)]\right)^2}_{\text{Bias}^2} + \underbrace{\mathbb{E}\left[(\hat{f}(x) - \mathbb{E}[\hat{f}(x)])^2\right]}_{\text{Variance}} + \underbrace{\sigma_\epsilon^2}_{\text{Irreducible Noise}}$$

---

## The Tradeoff Visualised

```
Test
Error
  │  
  │ ╲      Total Error
  │  ╲           ╱
  │   ╲         ╱
  │    ╲   *   ╱       * = optimal complexity
  │     ╲─────╱
  │ Bias²╲   ╱ Variance
  └──────────────────── Model Complexity
```

- **Low complexity:** High bias, low variance. Model misses the pattern.
- **High complexity:** Low bias, high variance. Model memorises noise.
- **Optimal:** Balances both.

---

## How Complexity Affects Each Component

| Model | Bias | Variance |
|---|---|---|
| Degree-1 polynomial | High (can't fit curves) | Low (stable) |
| Degree-3 polynomial | Medium | Medium |
| Degree-9 polynomial | Low (fits training perfectly) | High (unstable) |
| 1-NN (k=1) | Near-zero | Very high |
| n-NN (k=n, predict mean) | High | Near-zero |

---

## Double Descent: The Modern Complication

Classical theory predicts a U-shaped test error curve. But empirically, for neural networks:

```
Test Error
  │
  │      ╭──╮
  │     ╱    ╲               ╲
  │    ╱      ╲               ╲
  │───╱         ╲──────────────╲──
  └────────────────────────────── Model Size
     classical   interpolation   overparameterised
     regime      threshold       regime (GD inductive bias)
```

After the interpolation threshold (where the model can fit training data perfectly), test error can **decrease again** in the overparameterised regime. This is not explained by the classical bias-variance tradeoff and is an active research area.

---

## Practical Implications

1. **Choose model complexity via cross-validation** — plot val error vs. model complexity.
2. **Regularisation** shifts the tradeoff — moves the optimal point toward higher complexity.
3. **More data** reduces variance without increasing bias — shifts the tradeoff favorably.
4. **In practice**, most real-world ML suffers from either:
   - Underfitting: need more features or complex model
   - Overfitting: need regularisation or more data

---

## Connections

- [[Bias]] — decreases with complexity
- [[Variance]] — increases with complexity
- [[Underfitting]] — bias-dominated regime
- [[Overfitting]] — variance-dominated regime
- [[Regularization]] — controls the tradeoff
- [[Model Complexity]] — the x-axis of the tradeoff curve
- [[Generalization]] — the goal: minimise true error
- [[Bagging]] — a practical, quantified variance-reduction technique ($\rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$), directly attacking the variance term
- [[Boosting]] — a practical bias-reduction technique, directly attacking the bias term — bagging and boosting are the applied embodiment of this tradeoff's two sides
- [[Ensemble Learning]] — the general framework connecting both

---

## One-line Summary

> The bias-variance tradeoff states that model error decomposes into bias² + variance + noise — simpler models have high bias (underfit), complex models have high variance (overfit), and the art of ML is finding the complexity sweet spot where their sum is minimised.
