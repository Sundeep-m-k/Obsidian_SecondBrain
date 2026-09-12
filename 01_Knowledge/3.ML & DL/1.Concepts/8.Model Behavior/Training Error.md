# Training Error

## What is it?

**Training error** is the value of the cost/loss function evaluated on the **training set** — the data the model was trained on.

$$\hat{R}_{\text{train}}(\theta) = \frac{1}{n_{\text{train}}}\sum_{i \in \text{train}}\ell(y^{(i)}, \hat{f}(x^{(i)};\theta))$$

---

## What Training Error Tells You

Training error tells you **how well the model fits the training data**. It is what gradient descent directly minimises.

**Low training error alone means nothing.** A model that memorises all training examples achieves zero training error but may be useless on new data.

**High training error** means the model is underfitting — it can't even fit the data it was trained on.

---

## Relationship to Generalisation

The gap between training error and true error (generalisation error) is the **generalisation gap**:

$$\underbrace{R(\theta)}_{\text{true error}} = \underbrace{\hat{R}_{\text{train}}(\theta)}_{\text{training error}} + \underbrace{[R(\theta) - \hat{R}_{\text{train}}(\theta)]}_{\text{generalisation gap}}$$

VC theory bounds this gap:
$$R(\theta) \leq \hat{R}_{\text{train}}(\theta) + O\!\left(\sqrt{\frac{d_{VC}}{n}}\right)$$

The bound tightens as $n$ grows (more data → training error is a better proxy for true error) and loosens as $d_{VC}$ grows (more complex models have larger potential gaps).

---

## Training Error vs. Epoch

Training error should decrease as training progresses:
```
Train Error
  │
  │───╮
  │   ╲
  │    ╲──────────────────
  └──────────────────── Epochs
```
If it's not decreasing: learning rate too small, wrong architecture, data issue.

---

## Connections

- [[Test Error]] — what we actually care about
- [[Overfitting]] — low training error + high test error
- [[Underfitting]] — high training error
- [[Cost Function]] — training error is the average cost on training data

---

## One-line Summary

> Training error measures how well the model fits its training data — it is what gradient descent minimises, but low training error alone does not mean the model is good; what matters is the gap between training and test error.
