# Class Imbalance Handling

## What is it?

**Class imbalance** in training data: One class heavily dominates (e.g., 1% positive, 99% negative). Models learn to predict majority class; performance on minority class collapses.

**Handling techniques:** Resampling (SMOTE, oversample, undersample), class weights, threshold tuning, synthetic data generation.

See [[Class Imbalance Evaluation]] for evaluation strategies; this covers preprocessing solutions.

---

## Core Approaches

### 1. Class Weights

Model internally scales loss by class frequency. Positive examples weighted higher.

```python
from sklearn.linear_model import LogisticRegression

model = LogisticRegression(class_weight='balanced')
# Automatically: w_1 = n / (2 * n_1), w_0 = n / (2 * n_0)
# For 1% positive: w_1 = 50, w_0 ≈ 1
```

**Pros:** Simple, no data duplication, works with any model
**Cons:** Hyperparameter tuning still needed; can over-penalize

**When to use:** Default first approach; try before resampling

### 2. Oversampling (Random)

Duplicate minority examples until balanced.

```python
from imblearn.over_sampling import RandomOverSampler

oversampler = RandomOverSampler(random_state=42)
X_resampled, y_resampled = oversampler.fit_resample(X_train, y_train)
# Now 50/50 split
```

**Pros:** Simple; preserves all information
**Cons:** Exact duplication → overfitting; only works on train

**When to use:** Quick baseline; data already large

### 3. Undersampling

Remove majority examples until balanced.

```python
from imblearn.under_sampling import RandomUnderSampler

undersampler = RandomUnderSampler(random_state=42)
X_resampled, y_resampled = undersampler.fit_resample(X_train, y_train)
# Now 50/50 but smaller dataset
```

**Pros:** Smaller dataset (faster training)
**Cons:** Loses information; biased if majority has diverse patterns

**When to use:** Data is huge; can afford to discard

### 4. SMOTE (Synthetic Minority Over-sampling Technique)

Generate synthetic minority examples by interpolating between neighbors.

```python
from imblearn.over_sampling import SMOTE

smote = SMOTE(random_state=42, k_neighbors=5)
X_resampled, y_resampled = smote.fit_resample(X_train, y_train)
# Synthetics generated from k-nearest neighbors
```

**Pros:**
- No exact duplication; synthetics are plausible
- Preserves information
- Reduces overfitting vs. random oversample

**Cons:**
- Can generate out-of-distribution synthetics
- Slower than random oversample
- Requires tuning k_neighbors

**When to use:** Standard best practice; good balance of simplicity + performance

### 5. Combined: SMOTE + Undersampling

Balance both sides:
```python
from imblearn.pipeline import Pipeline
from imblearn.over_sampling import SMOTE
from imblearn.under_sampling import RandomUnderSampler

pipeline = Pipeline([
    ('smote', SMOTE(random_state=42)),
    ('undersample', RandomUnderSampler(random_state=42)),
    ('model', LogisticRegression())
])
```

**Pros:** Leverages benefits of both
**Cons:** Complex; more hyperparams to tune

---

## Critical: Prevent Leakage

### ✗ WRONG
```python
# Resample BEFORE split
X_resampled, y_resampled = SMOTE().fit_resample(X, y)  # On ALL data
X_train = X_resampled[:700]
X_test = X_resampled[700:]
# ← Synthetics from test set influenced training!
```

### ✓ CORRECT
```python
# Split FIRST
X_train, X_test, y_train, y_test = train_test_split(X, y)

# Resample TRAIN ONLY
X_train_resampled, y_train_resampled = SMOTE().fit_resample(X_train, y_train)

# Train on resampled, evaluate on original test
model.fit(X_train_resampled, y_train_resampled)
predictions = model.predict(X_test)  # Original, unmodified
```

Resampling must be inside CV loop or after train/test split.

---

## Comparison

| Method | Data Loss | Overfitting | Speed | Realism |
|--------|-----------|------------|-------|---------|
| Class Weights | No | Low | Fast | N/A (no data change) |
| Random Oversample | No | Medium | Fast | Duplication |
| Undersampling | Yes | Low | Fast | OK (discarding) |
| SMOTE | No | Low | Medium | High (interpolation) |
| SMOTE + Under | Yes | Low | Medium | High |

---

## Workflow

```
Class imbalance detected (e.g., 1% positive)
        ↓
Try class weights first (simplest)
        ↓
If still poor minority recall:
        ├─ Data is abundant? → SMOTE (standard)
        └─ Data is limited? → SMOTE + Undersample
        ↓
Apply inside CV loop or after train/test split (prevent leakage)
        ↓
Evaluate on original test set (unmodified)
```

---

## Interview Questions

**Q: Should you oversample before or after train/test split?**

**After split** (correct). Oversample X_train only; keep test original. If you oversample before splitting, synthetic minority examples end up in test. Test is no longer held-out; model is over-optimistic. Always split first, resample train. If using cross-validation, resample inside each CV fold. See [[Train Val Test Framework]].

---

**Q: When would you use SMOTE instead of simple random oversampling?**

Random oversample duplicates existing examples → overfitting (exact clones). SMOTE generates synthetics by interpolating between neighbors → more realistic, less overfitting. Use SMOTE when: (1) You want to avoid duplication. (2) Data allows interpolation (continuous features). Don't use if: features are discrete (genetic data, counts); need exact duplicates. SMOTE is typical best practice. See [[Class Imbalance Evaluation]].

---

**Q: If SMOTE generates synthetic data too close to test set examples, is that leakage?**

Potentially. SMOTE generates based on k-nearest neighbors in feature space. If synthetic happens to be similar to test example, it's not leakage (synthetics fit in train, test used separately). But practically, SMOTE can sometimes generate unrealistic combinations (especially high-dimensional data). Mitigate: (1) Don't set k too large. (2) Use domain validation (inspect synthetics). (3) Use SMOTE + Undersampling to reduce synthetic load. See [[Train Val Test Framework]].

---

## Connections

- [[Class Imbalance Evaluation]] — Evaluation metrics for imbalance
- [[Train Val Test Framework]] — Prevent leakage in resampling
- [[Preprocessing Pipelines]] — Resampling in pipeline

---

## One-line Summary

> Class imbalance handled via class weights (simplest), SMOTE (standard resampling), or combined SMOTE + undersampling — always resample inside CV loop or after train/test split to prevent leakage, and evaluate on original test set.
