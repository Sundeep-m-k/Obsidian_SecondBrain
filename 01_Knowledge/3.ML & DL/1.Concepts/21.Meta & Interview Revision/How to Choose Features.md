# How to Choose Features

## The Core Principle

A feature is worth including if it carries **information about $y$ that is not already captured by other features** in your model.

**Two requirements:**
1. The feature is **correlated with $y$** (predictive signal)
2. The feature is **available at prediction time** (no data leakage)

---

## Step-by-Step Feature Selection Process

### Step 1: Domain Knowledge First

Before looking at data: what do human experts know drives $y$?

- Predicting house price → size, location, age are obvious
- Predicting heart disease → blood pressure, cholesterol, age, smoking are obvious
- Start with domain-obvious features; data-driven selection refines from there

### Step 2: Exploratory Analysis

Compute for each feature $x_j$:

**Correlation with target (continuous $y$):**
$$r_{x_j,y} = \frac{\text{Cov}(x_j,y)}{\sigma_{x_j}\sigma_y}$$
Keep features with $|r| > 0.1$ as a starting point.

**Mutual information (any $y$ type):**
$$I(x_j; y) = \sum_{x,y} p(x,y)\log\frac{p(x,y)}{p(x)p(y)}$$
Captures non-linear relationships. $I = 0$ means $x_j \perp y$.

**Visualisation:** Scatter plots (regression), box plots per class (classification).

### Step 3: Remove Clearly Useless Features

- **Near-zero variance:** $\text{Var}(x_j) < \epsilon$ → feature is almost constant → no signal
- **Too many missing values:** If > 50% missing, the feature may not be reliable
- **Perfect duplicates / near-duplicates:** Keep only one; others add no information

### Step 4: Handle Multicollinearity

Highly correlated features ($|r(x_j, x_k)| > 0.9$) are redundant — one can be dropped.

**Variance Inflation Factor:**
$$\text{VIF}_j = \frac{1}{1 - R_j^2}$$
Where $R_j^2$ = $R^2$ of regressing $x_j$ on all other features. $\text{VIF} > 10$ → drop or combine.

### Step 5: Model-Based Selection

**L1 Regularisation (Lasso):** Train with L1 penalty; features with $\hat{\theta}_j = 0$ are automatically excluded.

**Tree-based importance:** Train a Random Forest; rank features by mean decrease in impurity:
$$\text{Importance}(j) = \sum_{\text{nodes using }j} n_t \cdot \Delta \text{Impurity}_t$$

**Recursive Feature Elimination (RFE):** Train → remove least important feature → retrain → repeat.

### Step 6: Evaluate with Cross-Validation

**Never select features on the test set.** Feature selection must be inside the CV loop:

```
For each fold:
    1. Fit feature selector on training fold only
    2. Transform training + validation fold
    3. Train model → evaluate on validation fold
Average CV scores to select best feature set
```

Doing feature selection on the full dataset and then cross-validating is **data leakage** — it over-estimates performance.

---

## Feature Engineering Checklist

Before removing features, consider engineering them:

| Raw feature | Engineering | Rationale |
|---|---|---|
| Income ($10^5$ range) | Log transform | Compress scale |
| Date | Extract day_of_week, month, is_weekend | Capture periodicity |
| City | One-hot / target encode | Nominal categorical |
| Area, Population | $\text{density} = \text{area}/\text{pop}$ | Domain ratio |
| Two correlated features | PCA component | Capture joint variance |

---

## Red Flags: Signs of Bad Features

| Warning | Meaning |
|---|---|
| Feature perfectly predicts $y$ | Data leakage — this info won't be available at prediction time |
| Feature only available after $y$ is observed | Temporal leakage |
| Feature has huge importance but makes no domain sense | Check for leakage or spurious correlation |
| Feature improves train but not val accuracy | Adding noise, not signal |

---

## Connections

- [[Features]] — what you're selecting
- [[Feature Engineering]] — creating better features
- [[Regularization]] — L1 performs selection implicitly
- [[Overfitting]] — too many useless features causes overfitting
- [[Training Data]] — feature selection must be done only on training data

---

## One-line Summary

> Choose features that have genuine predictive signal about $y$ (measured by correlation, mutual information, or model importance), are available at prediction time, and are not redundant — always doing selection inside the cross-validation loop to avoid leakage.
