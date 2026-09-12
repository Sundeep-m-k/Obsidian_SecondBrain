# Underfitting

## What is it?

**Underfitting** occurs when a model is **too simple** to capture the underlying patterns in the data. It performs poorly on both the training set and new data.

$$\text{Training error is high} \quad \text{AND} \quad \text{Validation error is high}$$

---

## Formal Characterisation

Underfitting = **high bias** in the bias-variance decomposition:

$$\text{True error} = \underbrace{\text{Bias}^2}_{\text{HIGH}} + \text{Variance} + \text{Noise}$$

The model's average predictions are systematically wrong — it consistently misses the true pattern.

---

## Diagnosing Underfitting

**Learning curve signature:**

```
Error
  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  validation error
  │──────────────────────  training error
  │   (both converge to a high value)
  └───────────────────────────── Training size n
```

Both training and validation error converge to a high value. Adding more data doesn't help — the model can't use it.

**Symptom:** Train accuracy 60%, Val accuracy 61% — close but both bad.

---

## Causes of Underfitting

| Cause | Example |
|---|---|
| Model too simple | Linear model for a quadratic relationship |
| Too few features | Missing key predictive features |
| Too much regularisation | $\lambda$ too large → weights all forced toward 0 |
| Not enough training iterations | Gradient descent stopped too early |

---

## Fixes

| Fix | Why it helps |
|---|---|
| Use a more complex model | Increase hypothesis class capacity |
| Add more/better features | Give model more signal to learn from |
| Add polynomial features | Let linear model capture non-linear patterns |
| Reduce regularisation strength $\lambda$ | Allow model to fit data more closely |
| Train longer (more epochs) | Allow GD to reach lower training loss |

---

## Connections

- [[Bias]] — underfitting is high bias
- [[Bias Variance Tradeoff]] — underfitting is one end of the tradeoff
- [[Model Complexity]] — underfitting means insufficient complexity
- [[Training Error]] — high training error is the signature
- [[Overfitting]] — the opposite problem

---

## One-line Summary

> Underfitting is when the model is too simple to learn the true pattern — both training and validation errors are high, the model has high bias, and the fix is to increase model complexity or add better features.
