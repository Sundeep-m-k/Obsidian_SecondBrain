# Overfitting

## What is it?

**Overfitting** occurs when a model learns the training data **too well** — including its noise and random fluctuations — so it fails to generalise to new, unseen data.

$$\text{Training error is low} \quad \text{BUT} \quad \text{Validation error is high}$$

---

## Formal Characterisation

Overfitting = **high variance** in the bias-variance decomposition:

$$\text{True error} = \text{Bias}^2 + \underbrace{\text{Variance}}_{\text{HIGH}} + \text{Noise}$$

The model fits the training set almost perfectly (memorises it), but the learned function is too sensitive to the specific training examples — it changes drastically with different training sets.

---

## Intuition

An overfit model has learned the signal **plus** the noise. On new data, the signal is similar but the noise is different — so the model fails.

**Analogy:** A student who memorises every past exam question but doesn't understand the underlying concepts. They ace the past papers but fail on new questions.

---

## Diagnosing Overfitting

**Learning curve signature:**

```
Error
  │                    ─ ─ ─ ─  validation error (high, diverging)
  │
  │
  │                    ──────── training error (very low)
  └───────────────────────────────── Epochs / Training size
              ↑ large gap here = overfitting
```

**Symptom:** Train accuracy 99%, Val accuracy 70% — large gap.

---

## Why Overfitting Happens

| Cause | Mechanism |
|---|---|
| Model too complex | High-capacity model can memorise $n$ training points |
| Too few training examples | Not enough data to constrain the model |
| Too many features (high $d$, low $n$) | Model fits noise in the high-dimensional space |
| Training too long | Model continues to fit noise after signal is learned |
| No regularisation | Nothing prevents weights from growing arbitrarily large |

---

## The Classic Example: Polynomial Degree

For $n=10$ data points:
- Degree 1 (line): underfits the curved data
- Degree 3: good fit
- Degree 9: perfectly passes through all 10 points, but oscillates wildly between them

The degree-9 polynomial has zero training error but huge test error.

---

## Fixes

| Method | How it prevents overfitting |
|---|---|
| **More training data** | More signal, less relative noise |
| **Simpler model** | Reduce hypothesis class capacity |
| **L2 regularisation** | Penalises large weights; forces smoother function |
| **L1 regularisation** | Sets irrelevant feature weights to zero |
| **Dropout** (NNs) | Randomly disables neurons; forces redundant representations |
| **Early stopping** | Stop training when validation loss starts increasing |
| **Data augmentation** | Artificially expand training set |
| **Cross-validation** | Better estimate of true generalisation |
| **Ensemble methods** | Average multiple models; variance cancels |

---

## Early Stopping: When to Stop

```
Loss
  │
  │           ─ ─ ─ ─ val loss (starts increasing = overfit)
  │         ╱
  │──────╱            ─────── train loss (continues decreasing)
  └──────↑─────────────────── Epochs
      Stop here (validation loss minimum)
```

Keep the checkpoint with the lowest validation loss, even if training loss can still decrease.

---

## Connections

- [[Variance]] — overfitting is high variance
- [[Bias Variance Tradeoff]] — overfitting is one end of the tradeoff
- [[Regularization]] — the main technical fix
- [[Training Error]] — low training error alone doesn't mean good model
- [[Test Error]] — what we actually care about
- [[Generalization]] — what overfitting destroys
- [[Underfitting]] — the opposite problem
- [[Data Leakage]] — a model that looks suspiciously good isn't always overfitting in the classical sense; check for leakage first, since it produces the same symptom (great training/val metrics, poor real-world performance) via a different mechanism
- [[Ensemble Learning]] — bagging's variance-reduction ($\rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$) is a direct, quantified fix for the variance half of overfitting

---

## One-line Summary

> Overfitting is when the model memorises training noise and loses the ability to generalise — evidenced by a large gap between low training error and high validation error, fixed by regularisation, more data, or reduced model complexity.
