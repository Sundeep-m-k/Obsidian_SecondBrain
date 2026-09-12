# How to Diagnose Overfitting

## The One Diagnostic That Matters

**Compare training error to validation error.**

| Pattern | Diagnosis | Action |
|---|---|---|
| Train error high, Val error high | **Underfitting** (high bias) | More complex model, more features |
| Train error low, Val error high (large gap) | **Overfitting** (high variance) | Regularise, more data, simpler model |
| Train error ≈ Val error (both low) | **Good fit** ✅ | Ship it |
| Train error ≈ Val error (both high) | **Underfitting** | More complex model |

---

## Diagnostic Tools

### 1. Learning Curves (Error vs. Training Size)

Plot train and val error as you increase the number of training examples $n$:

**Underfitting (high bias):**
```
Error
  │─ ─ ─ ─ ─ ─ ─ val error (high plateau)
  │─────────────  train error (converges to same high value)
  └──────────────────── n
```
Both converge to a high value. **More data won't help** — model is fundamentally too simple.

**Overfitting (high variance):**
```
Error
  │─ ─ ─ ─ ─ ─ ─ ─ ─  val error (still high)
  │
  │               ─────  train error (low)
  └──────────────────── n
```
Large gap. **More data WILL help** — the gap narrows as $n$ increases.

### 2. Training Curves (Error vs. Epoch)

Plot train and val loss over training epochs:

```
Loss
  │
  │─ ─ ─ ─ ─ val loss
  │         ╱
  │────────╱────  val loss starts increasing = overfitting starts here
  │                ─────  train loss (continues decreasing)
  └──────────────────── Epochs
         ↑ Stop here (early stopping)
```

### 3. Residual Plots (Regression)

Plot predicted $\hat{y}$ vs. residuals $(y - \hat{y})$:
- Random scatter → good fit ✅
- Curved pattern → non-linear relationship missed (underfit) ❌
- Funnel pattern → heteroscedasticity ❌
- Individual extreme points → outliers ❌

### 4. Confusion Matrix / ROC (Classification)

- Perfect training confusion matrix but poor val matrix → overfitting
- Check if val AUC-ROC is substantially lower than train AUC-ROC

---

## Quantitative Overfitting Threshold

A practical rule: if

$$\frac{\text{Val error} - \text{Train error}}{\text{Train error}} > 0.2 \quad \text{(20\% relative gap)}$$

you likely have meaningful overfitting. Investigate further.

---

## Fixing Overfitting (Checklist)

In order of what to try first:

1. **Get more training data** — most reliable fix
2. **Add regularisation** — L2 (Ridge) or L1 (Lasso); tune $\lambda$
3. **Reduce model complexity** — fewer layers/neurons, smaller polynomial degree
4. **Early stopping** — monitor val loss, stop when it starts rising
5. **Dropout** (neural networks) — randomly zero neurons during training
6. **Data augmentation** — if images/text, augment the training set
7. **Feature selection** — remove noisy/irrelevant features

---

## Connections

- [[Overfitting]] — what you're diagnosing
- [[Underfitting]] — the other possibility
- [[Bias Variance Tradeoff]] — conceptual framework
- [[Regularization]] — the fix
- [[Training Error]] — one half of the comparison
- [[Test Error]] — true performance

---

## One-line Summary

> Diagnose overfitting by comparing training and validation errors — a large gap (low train, high val) means overfitting; fix it with regularisation, more data, or a simpler model; a high value on both means underfitting — fix it with a more complex model or better features.
