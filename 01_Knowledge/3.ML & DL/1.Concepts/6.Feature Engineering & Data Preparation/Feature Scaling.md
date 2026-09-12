# Feature Scaling

## What is it?

**Feature scaling** is the process of transforming features to similar numerical ranges so that no single feature dominates due to its scale rather than its actual predictive importance. Raw features often span wildly different ranges — age (0–100), income ($0–$10,000,000), height (50–250 cm) — and several classes of model treat that raw scale as if it were signal unless it's corrected.

---

## Why It's Necessary

**For gradient-based algorithms:** when features have vastly different scales, the cost surface becomes elongated (high condition number). Gradient descent oscillates in the steep direction and barely moves in the shallow one, so convergence is slow or unstable.

$$x_1 = \text{house size (500–5000)}, \quad x_2 = \text{bedrooms (1–6)}$$

Without scaling, $J(\theta)$ changes rapidly along $\theta_1$ and slowly along $\theta_2$; gradient descent must take tiny steps to avoid overshooting $\theta_1$, so $\theta_2$ converges extremely slowly. With both features scaled to $[0,1]$ or $(-1,1)$, the contours become roughly circular and gradient descent moves directly toward the minimum.

**For distance-based algorithms** (KNN, K-Means, SVM): Euclidean distance is dominated by whichever feature has the largest raw range.

$$d = \sqrt{(30-32)^2 + (100{,}000-500{,}000)^2} \approx 400{,}000$$

A 2-year age gap becomes invisible next to a $400k income gap purely because of units, not relevance. After scaling both to $[0,1]$, both differences contribute comparably.

**Does NOT matter for:** decision trees, random forests, gradient-boosted trees — they split on thresholds per feature independently, never comparing magnitudes across features or computing distances.

| Algorithm | Scale? | Reason |
|---|---|---|
| Linear / Logistic Regression | Yes | Gradient descent is scale-sensitive |
| Neural Networks | Yes | Activation functions assume typical input ranges |
| SVM, KNN | Yes | Distance/kernel computations dominated by large-scale features |
| Decision Trees, Random Forest, Gradient Boosting | No | Splits are per-feature thresholds, scale-invariant |

---

## Scaling Methods

The two dominant methods each have their own atomic note with full derivation — this note covers *when to use which* rather than re-deriving them:

- [[Standardization]] — $x' = \frac{x-\mu}{\sigma}$ (z-score). Unbounded, preserves distribution shape, the usual default; sensitive to outliers.
- [[Normalization]] — $x' = \frac{x-\min(x)}{\max(x)-\min(x)}$ (min-max). Bounded to $[0,1]$, interpretable, good when a bounded input range matters (e.g. many neural-net activations); a single outlier compresses everything else.

Two less-common but genuinely distinct methods:

**Robust Scaling** — $x' = \frac{x - \text{median}}{\text{IQR}}$. Uses median/IQR instead of mean/std, so it isn't distorted by outliers the way Standardization is. Worth reaching for specifically when the data has heavy outliers you aren't otherwise treating.

```python
from sklearn.preprocessing import RobustScaler
X_scaled = RobustScaler().fit_transform(X)
```

**Log Transformation** — $x' = \log(x)$, for right-skewed data (income, durations, counts). Reduces skew and compresses large values, but doesn't work on zero/negative values, loses direct interpretability, and requires inverse-transforming predictions back if the target itself was log-transformed.

---

## Rule: Always Fit Scaling on the Training Set Only

$$\mu_j = \frac{1}{n_{\text{train}}}\sum_{i \in \text{train}} x_j^{(i)}, \quad \sigma_j = \sqrt{\frac{1}{n_{\text{train}}}\sum_{i \in \text{train}}(x_j^{(i)} - \mu_j)^2}$$

Apply to validation and test using **training-set statistics only**:
$$x_j^{(i)'} = \frac{x_j^{(i)} - \mu_j^{\text{train}}}{\sigma_j^{\text{train}}}$$

Fitting the scaler on the full dataset before splitting leaks test-set distributional information into training — the test set is no longer a fair estimate of generalization. This is a specific instance of [[Data Leakage]].

```python
# WRONG — scaler sees test data before the split does
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
X_train, X_test = X_scaled[:70], X_scaled[70:]

# CORRECT — split first, fit only on train, transform both
X_train, X_test = X[:70], X[70:]
scaler = StandardScaler().fit(X_train)
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

In practice, encode this with a pipeline so it can't be gotten wrong:

```python
from sklearn.pipeline import Pipeline
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("model", LogisticRegression()),
])
pipeline.fit(X_train, y_train)          # scaler fits on X_train only, internally
predictions = pipeline.predict(X_test)  # scaler transforms X_test using train stats
```

See [[Preprocessing Pipelines]] for the general pattern this belongs to.

---

## What Scaling Does *Not* Fix

Scaling does not remove multicollinearity. Age and year-of-birth stay perfectly correlated after both are scaled — the model still gets unstable coefficients from the redundancy. That's a [[Regularization]] problem, not a scaling problem.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| Fit scaler on test/full data | Test statistics leak into preprocessing | Split first, fit on train only |
| Skip scaling for a linear/NN model | Slow or non-converging gradient descent | Always scale for gradient-based algorithms |
| Use StandardScaler with heavy outliers | Mean/std get distorted, scaling is thrown off | Use RobustScaler, or treat outliers first ([[Outlier Detection and Treatment]]) |
| Scale a tree-based model | Wasted computation, no benefit | Skip it — trees are scale-invariant |

---

## Interview Questions

**Why do gradient-based algorithms need scaled features?** Without scaling, the cost surface is elongated (elliptical) because features live on different numeric ranges; a step size that's right for a small-range feature is far too small for a large-range one, so convergence is slow or zig-zags. Scaling makes the surface roughly circular, so gradient descent converges directly and quickly.

**StandardScaler vs. MinMaxScaler — when does each apply?** StandardScaler is unbounded and preserves distribution shape but is sensitive to outliers — use it for roughly-normal, unbounded data (age, temperature). MinMaxScaler bounds everything to $[0,1]$ and is easy to interpret but a single outlier compresses the rest of the data — use it when a bounded range genuinely matters (e.g. many NN activation functions) or the data has no extreme outliers.

**Do you scale one-hot-encoded categorical features?** Usually not — one-hot columns are already 0/1, so scaling just relabels rather than fixing a real scale problem. Trees never need it; for linear models it's rarely worth doing. See [[Categorical Encoding]].

**If you apply a log transform and then StandardScaler, which comes first?** Log transform first, to fix skew, then StandardScaler on the now-more-symmetric distribution — `Pipeline([("log", LogTransformer()), ("scaler", StandardScaler()), ("model", Model())])`.

---

## Connections

- [[Standardization]] — the primary scaling method, full derivation
- [[Normalization]] — min-max scaling, full derivation
- [[Feature Engineering]] — scaling is one step in the broader preprocessing pipeline
- [[Preprocessing Pipelines]] — where scaling sits relative to encoding, imputation, etc.
- [[Data Leakage]] — fitting a scaler on test data is a specific leakage pattern
- [[Gradient Descent]] — the mechanism scaling is protecting the convergence of
- [[Outlier Detection and Treatment]] — outliers should usually be handled before choosing a scaler

---

## One-line Summary

> Feature scaling transforms features to comparable numerical ranges so gradient-based and distance-based models treat them fairly instead of by raw magnitude — always fit the scaler on the training set only, choose Standardization/Normalization/Robust based on outlier sensitivity, and skip it entirely for tree-based models.
