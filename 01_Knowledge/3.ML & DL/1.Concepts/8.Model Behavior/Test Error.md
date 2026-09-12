# Test Error

## What is it?

**Test error** (also called **generalisation error** or **out-of-sample error**) is the model's error on data it has **never seen during training or hyperparameter tuning** — a held-out test set that simulates real deployment.

$$\hat{R}_{\text{test}}(\theta) = \frac{1}{n_{\text{test}}}\sum_{i \in \text{test}}\ell(y^{(i)}, \hat{f}(x^{(i)};\hat{\theta}))$$

---

## Why It's the Only Error That Matters

Training error can be driven to zero by memorisation. Validation error can be inadvertently optimised through hyperparameter tuning (multiple rounds of tuning on validation set = implicit leakage). Only the test set — used **once and only once** — gives an unbiased estimate of real-world performance.

**The sacred rule:** Never use the test set until the model is fully finalised. No tuning after seeing test results.

---

## Train / Validation / Test: The Three Roles

| Split | Used for | Can look at it multiple times? |
|---|---|---|
| **Training** | Fit parameters $\theta$ | Yes — gradient updates |
| **Validation** | Tune hyperparameters, model selection | Yes — but each look slightly biases estimates |
| **Test** | Final unbiased evaluation | **No — one time only** |

---

## Test Error ≈ True Generalisation Error

By the Law of Large Numbers, for a large enough test set:
$$\hat{R}_{\text{test}} \approx R(\theta) = \mathbb{E}_{(x,y)\sim p_{\text{data}}}[\ell(y, \hat{f}(x))]$$

Confidence interval for test error with $n_{\text{test}}$ examples (by Central Limit Theorem):
$$\hat{R}_{\text{test}} \pm z_{0.975} \sqrt{\frac{\hat{R}_{\text{test}}(1-\hat{R}_{\text{test}})}{n_{\text{test}}}}$$

For 95% CI with 1,000 test examples and 10% error rate:
$$0.10 \pm 1.96\sqrt{\frac{0.10 \times 0.90}{1000}} \approx 0.10 \pm 0.019$$

Larger test set → tighter confidence interval.

---

## Reading Train vs. Val vs. Test Error

| Pattern | Diagnosis |
|---|---|
| Train error high, Val error high | Underfitting — increase complexity |
| Train error low, Val error high | Overfitting — regularise or get more data |
| Train ≈ Val (both low), Test high | Val set was used too much — test set is the honest number |
| Train ≈ Val ≈ Test (all low) | Model generalises well ✅ |

---

## Connections

- [[Training Error]] — what the model minimises
- [[Generalization]] — test error estimates true generalisation error
- [[Overfitting]] — large gap between train error and test error
- [[Underfitting]] — both train and test error are high
- [[Training Data]] — test set is separate from training data
- [[Grouped Train-Test Split]] — a prerequisite for the test set to be a valid, unbiased estimate at all when rows share real-world entities
- [[Rolling-Origin Validation]] — the temporal analog of the "test once" rule: test at several historical cutoffs instead of one static split
- [[Brier Score]] — for probabilistic predictions specifically, this is the proper-scoring-rule version of "test error"



---

## One-line Summary

> Test error is the model's performance on never-before-seen data — the single honest estimate of real-world performance — and protecting the test set from any influence during development is the most important procedural discipline in machine learning.
