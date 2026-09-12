# Confidence Intervals and Bootstrap

## What is it?

A **confidence interval (CI)** is a range of values that likely contains the true parameter. Instead of reporting a single point estimate, report a range with uncertainty.

**Bootstrap** is a resampling technique to estimate CIs without distributional assumptions. Resample data with replacement, recompute statistic, build distribution of estimates.

---

## Why Confidence Intervals Matter

### Point Estimate vs. Interval

**Without CI:** "Model accuracy is 85%"
- Single number; no sense of uncertainty
- Could be 80-90% or 70-95%?

**With CI:** "Model accuracy is 85% (95% CI: [82%, 88%])"
- Range includes plausible values
- Smaller range = more confident (good)
- Larger range = more uncertain (bad)

### Practical Impact

**Example:** Two models
- Model A: 85% accuracy (95% CI: [84%, 86%])
- Model B: 87% accuracy (95% CI: [70%, 95%])

Model B has higher point estimate but larger uncertainty. Model A more reliable (narrower CI).

---

## Confidence Level Interpretation

**95% CI:** If we repeated the experiment 100 times, ~95 times the true parameter would fall in our reported interval.

**NOT:** "95% probability true value is in [82%, 88%]" (frequentist interpretation; true value is fixed, not random)

**Correct:** "If we repeated experiment 100 times, true value in interval ~95 times"

---

## Bootstrap Method

### Procedure

1. Start with sample of size $n$
2. Resample $n$ examples **with replacement**
3. Compute statistic (e.g., mean, accuracy)
4. Repeat steps 2-3 many times (e.g., 1000)
5. Distribution of resampled statistics ≈ true sampling distribution
6. CI = percentiles of bootstrap distribution (e.g., 2.5th to 97.5th for 95% CI)

### Example: Bootstrap for Mean

```
Original data: [10, 12, 11, 13, 9]  (mean = 11)

Bootstrap samples (with replacement):
Sample 1: [10, 10, 11, 13, 9]    → mean = 10.6
Sample 2: [12, 12, 11, 9, 9]     → mean = 10.6
Sample 3: [13, 11, 10, 12, 13]   → mean = 11.8
...
Sample 1000: [11, 12, 10, 11, 12] → mean = 11.2

Bootstrap means: [10.6, 10.6, 11.8, ..., 11.2]
Percentiles: 2.5th = 10.2, 97.5th = 11.8
95% CI: [10.2, 11.8]
```

### Python Implementation

```python
from scipy import stats
import numpy as np

# Data
X = np.array([10, 12, 11, 13, 9])

# Bootstrap
n_bootstrap = 1000
bootstrap_means = []

for _ in range(n_bootstrap):
    sample = np.random.choice(X, size=len(X), replace=True)
    bootstrap_means.append(sample.mean())

# Confidence interval
ci_lower = np.percentile(bootstrap_means, 2.5)
ci_upper = np.percentile(bootstrap_means, 97.5)
print(f"95% CI: [{ci_lower:.2f}, {ci_upper:.2f}]")

# Using scipy
def bootstrap_ci(X, n_bootstrap=1000, ci=95):
    bootstrap_stats = []
    for _ in range(n_bootstrap):
        sample = np.random.choice(X, size=len(X), replace=True)
        bootstrap_stats.append(sample.mean())
    alpha = (100 - ci) / 2
    return np.percentile(bootstrap_stats, alpha), np.percentile(bootstrap_stats, 100 - alpha)

ci = bootstrap_ci(X)
```

---

## Bootstrap for Model Metrics

### Accuracy CI

```python
from sklearn.metrics import accuracy_score

y_true = [1, 0, 1, 1, 0, 1, 0, 1]
y_pred = [1, 0, 1, 0, 0, 1, 0, 1]

# Point estimate
accuracy = accuracy_score(y_true, y_pred)  # 0.875

# Bootstrap
n_bootstrap = 1000
bootstrap_accuracies = []

n = len(y_true)
for _ in range(n_bootstrap):
    indices = np.random.choice(n, size=n, replace=True)
    y_true_boot = [y_true[i] for i in indices]
    y_pred_boot = [y_pred[i] for i in indices]
    bootstrap_accuracies.append(accuracy_score(y_true_boot, y_pred_boot))

# CI
ci_lower = np.percentile(bootstrap_accuracies, 2.5)
ci_upper = np.percentile(bootstrap_accuracies, 97.5)
print(f"Accuracy: {accuracy:.3f}, 95% CI: [{ci_lower:.3f}, {ci_upper:.3f}]")
```

### Precision/Recall CI

```python
from sklearn.metrics import precision_score, recall_score

# Bootstrap for precision
bootstrap_precisions = []
for _ in range(n_bootstrap):
    indices = np.random.choice(n, size=n, replace=True)
    y_true_boot = [y_true[i] for i in indices]
    y_pred_boot = [y_pred[i] for i in indices]
    bootstrap_precisions.append(precision_score(y_true_boot, y_pred_boot))

# CI
ci_precision = np.percentile(bootstrap_precisions, [2.5, 97.5])
```

---

## Analytical vs. Bootstrap

### Analytical (Parametric)

Assume data follows known distribution (e.g., normal). Use formula to compute CI.

**Standard Error (SE):** $SE = \frac{\sigma}{\sqrt{n}}$

**95% CI:** $\bar{x} \pm 1.96 \cdot SE$

**Pros:**
- Fast (formula, no resampling)
- Exact (if assumptions met)

**Cons:**
- Requires assumptions
- Fails for skewed data, non-normal distributions

**When to use:** Normal data, simple statistics

### Bootstrap (Non-parametric)

No distributional assumptions. Resample to estimate distribution.

**Pros:**
- No assumptions
- Works for any statistic (even complex ones)
- Robust to skewness, outliers

**Cons:**
- Slower (many resamples)
- Requires sufficient data (rule of thumb: n ≥ 30)

**When to use:** Non-normal data, complex statistics, model metrics

---

## Interpreting Bootstrap CIs

### Narrow CI (Good)

95% CI: [0.84, 0.86] (width 0.02)
- Estimate is precise
- Model performance stable

### Wide CI (Bad)

95% CI: [0.70, 0.95] (width 0.25)
- Estimate is uncertain
- Model unstable; performance varies a lot

**Implications:**
- Narrow CI: Trust the estimate; model is reliable
- Wide CI: Don't trust point estimate; need more data

---

## CI vs. Cross-Validation

**Cross-Validation:** Train $k$ times, get $k$ performance estimates. Report mean ± std dev.

**Bootstrap:** Resample from single dataset, estimate CI.

**Similarities:** Both estimate uncertainty in performance.

**Differences:**
- CV: Directly reflects training/testing variability
- Bootstrap: Estimates sampling variability

**Best practice:** Use both
```python
# CV gives stability estimate
cv_scores = cross_val_score(model, X, y, cv=5)
print(f"CV: {cv_scores.mean():.3f} ± {cv_scores.std():.3f}")

# Bootstrap gives CI on test set performance
bootstrap_ci = bootstrap_confidence_interval(y_test, y_pred, n_bootstrap=1000)
print(f"Test Accuracy CI: {bootstrap_ci}")
```

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Report point estimate without CI | No sense of uncertainty | Always include CI |
| Use too few bootstrap samples | CI estimates noisy (n_bootstrap < 100) | Use n_bootstrap ≥ 1000 |
| Assume CI = "95% probability true value in range" | Misinterpretation (frequentist vs. Bayesian) | Use correct frequentist interpretation |
| Ignore bootstrap for complex metrics | Think CI only for mean | Bootstrap works for any statistic |
| Use analytical CI for skewed data | Inaccurate; assumes normality | Use bootstrap instead |

---

## Interview Questions

**Q: What is a confidence interval and how do you interpret "95% CI: [0.82, 0.88]"?**

CI is a range of plausible values for a parameter. "95% CI: [0.82, 0.88]" means: If we repeated the experiment 100 times, the true parameter would fall in [0.82, 0.88] approximately 95 times. NOT "95% probability true value is in range" (that's Bayesian). Frequentist interpretation: repeated sampling interpretation. CI width reflects uncertainty; narrow = confident, wide = uncertain. See [[Statistical Significance Testing for Model Comparison]].

---

**Q: What is bootstrap and why would you use it instead of analytical formulas?**

Bootstrap resamples data with replacement to estimate uncertainty without assumptions. Instead of assuming normality, generate 1000 bootstrap samples, recompute statistic (mean, accuracy, precision), and use percentiles as CI. Useful because: (1) Works for any statistic (not just mean). (2) No distributional assumptions. (3) Handles skewed, non-normal data. Downside: Slower (needs many resamples). Use analytical for simple statistics on normal data; bootstrap for complex metrics or non-normal data. See [[Class Imbalance Evaluation]].

---

**Q: How many bootstrap samples do you need?**

Rule of thumb: $n_{bootstrap} \geq 1000$. For stable CI estimates, 1000 resamples is standard. If extreme tails matter (e.g., 99% CI), use more (5000+). Too few (<100) gives noisy CI; too many (>10,000) wastes computation. Python: `np.random.choice(X, size=len(X), replace=True)` repeated 1000 times. See [[Cross Validation Strategy]].

---

**Q: Should you use cross-validation or bootstrap to estimate model performance uncertainty?**

Both: CV reflects train/test split variability; bootstrap reflects sampling variability. Practical: Use nested CV for hyperparameter tuning (prevents overfitting). Use bootstrap on final test set to get CI on performance. Example: 5-fold CV gives 5 test scores (mean ± std); bootstrap those test predictions to get CI. Together they give complete picture: CV stability + bootstrap uncertainty. See [[Cross Validation Strategy]].

---

**Q: Can you use bootstrap on a test set to get a confidence interval for model accuracy?**

Yes. Resample test set with replacement, recompute accuracy for each resample, get CI from percentiles. Example: 100 test examples, 1000 bootstrap resamples → 1000 accuracies → CI. Caveat: This estimates uncertainty in accuracy estimate from this particular test set. Doesn't account for test set bias (might be easy/hard test set). Combine with cross-validation or separate hold-out validation for robustness. See [[Train Val Test Framework]].

---

## Prerequisites

[[Hypothesis Test]], [[Baseline Estimator]]

## Related concepts

[[Statistical Significance Testing for Model Comparison]], [[Cross Validation Strategy]], [[Class Imbalance Evaluation]]

## Tags

#category/statistics #topic/estimation #math/probability

## One-line summary

> Confidence intervals quantify uncertainty in estimates; bootstrap resamples data with replacement to estimate CI without distributional assumptions — use ≥1000 bootstrap samples, interpret as "true value in range ~95% of repeated experiments," and combine with cross-validation for complete uncertainty picture.
