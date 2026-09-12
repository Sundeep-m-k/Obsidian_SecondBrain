# Standardization

## What is it?

**Standardization** (also called **Z-score normalization**) rescales each feature to have **zero mean** and **unit variance**.

$$x_j' = \frac{x_j - \mu_j}{\sigma_j}$$

Where:
- $\mu_j = \frac{1}{n}\sum_{i=1}^n x_j^{(i)}$ = mean of feature $j$ in the training set
- $\sigma_j = \sqrt{\frac{1}{n}\sum_{i=1}^n (x_j^{(i)} - \mu_j)^2}$ = standard deviation of feature $j$ in the training set

**After standardisation:** $\mathbb{E}[x_j'] = 0$, $\text{Var}(x_j') = 1$.

---

## Derivation: Why Zero Mean and Unit Variance?

**Zero mean** centres the data at the origin. For gradient descent, this prevents systematic bias in which direction parameters update — symmetric updates around zero converge faster.

**Unit variance** ensures all features contribute equally to distance computations and gradient magnitudes. Without this, a feature with $\sigma = 1000$ would dominate a feature with $\sigma = 0.01$.

---

## Example

Feature $x_1$ = salary: values $[40{,}000,\ 60{,}000,\ 50{,}000,\ 80{,}000]$

$$\mu_1 = 57{,}500, \quad \sigma_1 = 14{,}361$$

Standardised values:
$$x_1' = \frac{40000 - 57500}{14361} \approx -1.22, \quad \frac{60000 - 57500}{14361} \approx 0.17, \ldots$$

All values are now dimensionless numbers centred around 0.

---

## The Full Pipeline (Critical: Fit on Train Only)

```python
# Step 1: Compute statistics on TRAINING SET ONLY
mu = X_train.mean(axis=0)        # shape (d,)
sigma = X_train.std(axis=0)      # shape (d,)

# Step 2: Apply to ALL splits using training statistics
X_train_scaled = (X_train - mu) / sigma
X_val_scaled   = (X_val   - mu) / sigma   # NOT val mean/std
X_test_scaled  = (X_test  - mu) / sigma   # NOT test mean/std
```

**Why fit only on training?** If you compute $\mu$ and $\sigma$ on the full dataset (including val/test), you leak information about the distribution of test data into training — a subtle but serious form of data leakage.

---

## Standardization vs. Normalization

| Property | Standardization (Z-score) | Normalization (Min-Max) |
|---|---|---|
| Output range | $(-\infty, +\infty)$, centred at 0 | $[0, 1]$ |
| Preserves outliers | Yes (but less influential) | No (outliers dominate) |
| Assumes distribution | Works for any distribution | Works for any distribution |
| Works with GD | ✅ Excellent | ✅ Good |
| Sensitive to outliers | Less so | Very sensitive |
| Use when | General purpose, regression, most NNs | Bounded input required, image pixels |

---

## Effect on the Cost Surface

Without standardisation (features on different scales):
- Contours of $J(\theta)$ are elongated ellipses.
- Gradient descent zigzags slowly.
- Condition number $\kappa = \lambda_{\max}/\lambda_{\min}$ is large.

With standardisation:
- Contours are more circular.
- Gradient descent takes direct steps toward the minimum.
- Condition number approaches 1.

**Condition number improvement:** Standardisation reduces $\kappa$ from potentially $O((\text{scale ratio})^2)$ to $O(1)$. For features with ratio 1000:1, this is a $10^6\times$ improvement in the condition number.

---

## Robust Scaling (For Outlier-Heavy Data)

Uses quantiles instead of mean/std:

$$x_j' = \frac{x_j - Q_{50}(x_j)}{Q_{75}(x_j) - Q_{25}(x_j)}$$

Where $Q_{25}, Q_{50}, Q_{75}$ are the 25th, 50th (median), 75th percentiles.

Outliers don't affect the median or IQR as severely, making this more robust.

---

## Connections

- [[Feature Scaling]] — standardisation is one scaling method
- [[Normalization]] — min-max alternative
- [[Gradient Descent]] — dramatically faster after standardisation
- [[Multiple Linear Regression]] — scaling is essential before GD
- [[Feature Engineering]] — part of preprocessing

---

## One-line Summary

> Standardisation rescales features to zero mean and unit variance by subtracting the training-set mean and dividing by the training-set standard deviation — it is the most common and robust scaling method, making gradient descent converge dramatically faster by equalising the scale of all features.
