# Data Leakage Prevention

## What is it?

**Data leakage** occurs when information from validation/test sets influences training. Model appears to perform well but fails in production because it "saw" future/held-out data during development.

See [[Common Mistakes in ML#1 - Data Leakage]] for detailed pitfalls.

---

## Types of Leakage

### 1. Preprocessing Leakage

Fit preprocessing on full data (including test) before splitting.

**Example:** StandardScaler
```python
# ✗ WRONG
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)  # Fit on all data
X_train = X_scaled[:700]
X_test = X_scaled[700:]
# ← Test mean/std influenced scaler
```

**Fix:** Split first, fit on train
```python
# ✓ CORRECT
X_train, X_test = X[:700], X[700:]
scaler = StandardScaler()
scaler.fit(X_train)
X_train_scaled = scaler.transform(X_train)
X_test_scaled = scaler.transform(X_test)
```

### 2. Feature Selection Leakage

Select features on full dataset before splitting.

```python
# ✗ WRONG
top_features = select_top_k_features(X, y, k=10)  # On ALL data
X_train = X[:700][top_features]
X_test = X[700:][top_features]
# ← Test examples influenced feature selection
```

**Fix:** Select on train only
```python
# ✓ CORRECT
X_train, X_test = X[:700], X[700:]
top_features = select_top_k_features(X_train, y_train, k=10)
X_train = X_train[top_features]
X_test = X_test[top_features]
```

### 3. Hyperparameter Tuning on Test Set

Trying different hyperparams and selecting based on test results.

```python
# ✗ WRONG
best_score = 0
for lambda in [0.01, 0.1, 1.0]:
    model = train(X_train, y_train, lambda)
    test_score = evaluate(model, X_test, y_test)
    if test_score > best_score:
        best_lambda = lambda
        best_score = test_score
# ← Tuned on test set!
```

**Fix:** Use validation set
```python
# ✓ CORRECT
best_score = 0
for lambda in [0.01, 0.1, 1.0]:
    model = train(X_train, y_train, lambda)
    val_score = evaluate(model, X_val, y_val)  # Tune on VAL
    if val_score > best_score:
        best_lambda = lambda
final_model = train(X_train, y_train, best_lambda)
test_score = evaluate(final_model, X_test, y_test)  # Evaluate on TEST
```

### 4. Temporal Leakage (Time-Series)

Using future data to train model that predicts the past.

```python
# ✗ WRONG
data = shuffle(data)  # Destroys temporal order
X_train = data[:700]
X_test = data[700:]
# ← Training includes future examples relative to test
```

**Fix:** Preserve temporal order
```python
# ✓ CORRECT
X_train = data[:700]  # Past
X_test = data[700:]   # Future
# No shuffling; train on past, test on future
```

### 5. Resampling Leakage

Apply SMOTE/oversampling before splitting.

```python
# ✗ WRONG
X_resampled, y_resampled = SMOTE().fit_resample(X, y)  # On ALL
X_train = X_resampled[:700]
X_test = X_resampled[700:]
# ← Test examples influenced synthetic generation
```

**Fix:** Resample after split
```python
# ✓ CORRECT
X_train, X_test = train_test_split(X, y)
X_train_res, y_train_res = SMOTE().fit_resample(X_train, y_train)
# Train and evaluate normally
```

### 6. Target Information in Features

Including features that are consequences of target.

```python
# ✗ WRONG
# Predicting: Did patient recover?
# Features include: "Treatment received" (determined by doctor based on diagnosis)
# ← Treatment is consequence of diagnosis, not predictor
```

**Fix:** Use only features available at prediction time
```python
# ✓ CORRECT
# Features: Initial symptoms, lab results
# Don't include: Treatment, post-treatment outcomes
```

---

## Detection Checklist

Before training, ask:

1. **Preprocessing:** Did I fit scaling/imputation on full data? → Fix
2. **Feature Selection:** Did I select features on full dataset? → Fix
3. **Hyperparameter Tuning:** Did I select hyperparams based on test results? → Fix
4. **Temporal Data:** Did I shuffle time-series? → Fix
5. **Resampling:** Did I resample before splitting? → Fix
6. **Target Leakage:** Are features available at prediction time? → Fix
7. **Cross-Validation:** Is preprocessing inside each CV fold? → Fix

**Red flags:**
- Val/test performance much better than expected
- Model works great in development, fails in production
- Train performance >> test performance (overfitting)

---

## Prevention Strategy

```
Step 1: Split data into train / val / test
Step 2: Fit ALL preprocessing on train ONLY
Step 3: Apply train's preprocessing to val/test
Step 4: Tune hyperparams using val results
Step 5: Evaluate on test (look ONCE)
Step 6: Deploy and monitor
```

**sklearn Pipeline automates this:**
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LogisticRegression

# Pipeline fits scaler on X_train, applies to X_test automatically
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', LogisticRegression())
])

pipeline.fit(X_train, y_train)
predictions = pipeline.predict(X_test)  # Scaler fit only on X_train
```

---

## Interview Questions

**Q: What is data leakage and why is it critical to prevent?**

Data leakage: Test/val set statistics or information influences training. Model appears great but fails in production (test wasn't truly held-out). Example: Fitting StandardScaler on all data before splitting; test mean/std influenced preprocessing. Prevention: Split FIRST, fit preprocessing on train only, apply to test. Critical because: (1) Invalidates performance estimates. (2) Causes production failures. (3) Wastes time debugging why model works in dev but not live. See [[Train Val Test Framework]].

---

**Q: Give an example of data leakage in time-series data.**

Shuffle data before splitting, destroying temporal order. If training data includes tomorrow's price to predict today's, model will perform great in development (sees future). But in production, future data doesn't exist; model fails. Prevention: Preserve temporal order. Train on past, test on future. Use forward-chaining cross-validation: Fold 1: train on [t=1], test on [t=2]. Fold 2: train on [t=1-2], test on [t=3]. See [[Cross Validation Strategy]].

---

**Q: You scale features using StandardScaler, then split into train/test. Is this leakage?**

Yes. The scaler learned mean/std from test examples (implicitly; test data influenced the statistics). Test should be unseen. Correct: Split first, fit StandardScaler on train only, apply train's mean/std to test. Python: `scaler.fit(X_train)` then `scaler.transform(X_test)`. Using train's statistics on test is OK; it's unseen and evaluated fairly. See [[Feature Scaling]].

---

## Connections

- [[Common Mistakes in ML#1 - Data Leakage]] — Pitfall reference
- [[Train Val Test Framework]] — Train/val/test framework
- [[Cross Validation Strategy]] — CV prevents leakage
- [[Preprocessing Pipelines]] — Pipelines prevent leakage

---

## One-line Summary

> Data leakage occurs when test/val info influences training (preprocessing, feature selection, tuning, resampling) — prevent by splitting first, fitting all preprocessing on train only, using validation for tuning, and using sklearn pipelines to automate correctly.
