---
tags: [category/ml-dl, index, moc]
---

# Feature Engineering & Data Preparation — Index

> **Position in vault**: `3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/`
> **Purpose**: Everything about turning raw data into what a model actually consumes — representation basics (what a feature vector is), classical feature engineering, and the practical data-prep discipline (missing values, encoding, outliers, scaling, imbalance, leakage, pipelines).
> **History**: This module merges what used to be two separately-built modules (an earlier "Features and Representation" pass, and a later "Data Preparation" pass) that covered overlapping ground — most notably feature scaling, which existed as two un-cross-linked notes before this merge. They're unified here as one module with one canonical note per concept.

## Section Map

**Representation basics:**

| Note | Covers |
|---|---|
| [[Data Representation]] | How raw data becomes a numeric input a model can consume |
| [[Feature Vector]] | One example, represented as a vector of feature values |
| [[Vectorization]] | Turning any raw input into a feature vector |
| [[Multiple Features]] | Working with more than one feature at once |
| [[Feature Engineering]] | Constructing new, more predictive features from raw ones |

**Preprocessing (the practical discipline):**

| Note | Covers |
|---|---|
| [[Missing Values Handling]] | Detection, MCAR/MAR/MNAR, imputation strategies |
| [[Categorical Encoding]] | Label, one-hot, target encoding; ordinal vs. nominal, high-cardinality handling |
| [[Outlier Detection and Treatment]] | IQR/Z-score/isolation forest detection; deletion, capping, transformation |
| [[Feature Scaling]] | Standardization, normalization, robust scaling, log transform — when each applies |
| [[Standardization]] | Z-score scaling in full detail |
| [[Normalization]] | Min-max scaling in full detail |
| [[Class Imbalance Handling]] | Class weights, SMOTE, oversampling/undersampling |
| [[Data Leakage Prevention]] | Practical checklist of leakage failure modes and fixes (see [[Data Leakage]] under Statistics for the formal theory) |
| [[Preprocessing Pipelines]] | sklearn `Pipeline`/`ColumnTransformer`, reproducibility, serialization |

## Reading Paths by Role

**Data Analyst** — priority: Missing Values, Outlier Detection, Feature Scaling. 1-day sprint: those three ≈ 80% readiness.

**ML Engineer / AI-ML roles** — priority: Missing Values, Categorical Encoding, Feature Scaling, Data Leakage Prevention, Preprocessing Pipelines (the full pipeline). 1-day sprint: those five ≈ 85% readiness.

**MLOps / FDE** — priority: Data Leakage Prevention, Preprocessing Pipelines (reproducibility and deployment matter most here).

## Key Topics by Interview Frequency

**Tier 1 (always asked):** handling missing values (delete vs. impute), why gradient-based models need scaled features, preventing data leakage (fit on train, apply to test), one-hot vs. label encoding.

**Tier 2 (often asked):** class imbalance solutions (class weights vs. SMOTE), outlier detection and treatment, why a pipeline matters.

**Tier 3 (sometimes asked):** Standardization vs. Normalization, MCAR/MAR/MNAR, target encoding for high cardinality.

## Common Interview Scenarios

**"How do you handle missing values?"** How much is missing (<5% → delete/mean, 5–20% → KNN/MICE, >20% → consider dropping the feature) → is it MCAR/MAR/MNAR (affects whether "just impute" is even valid) → fit the imputer on train only, apply to val/test → check whether imputation introduced bias. See [[Missing Values Handling]], [[Data Leakage Prevention]].

**"1000 categorical features, some with 10K unique values — how do you encode them?"** One-hot on 10K categories is a red flag (10K sparse columns). Prefer target encoding (mean target per category, one column), frequency encoding, embeddings (in a DL context), or grouping rare categories into "Other." See [[Categorical Encoding]].

**"Why scale for logistic regression but not a decision tree?"** Trees split on per-feature thresholds — "age > 30" behaves identically whether age is 0–100 or 0–100,000. Gradient-based models take a gradient step whose right size depends on scale, so an unscaled large-range feature makes the cost surface elongated and slows convergence. See [[Feature Scaling]].

**"You fit StandardScaler on all data, then split 70/30 — is that leakage?"** Yes — test statistics influenced the scaler's mean/std before the split even happened. Split first, fit on train, transform both. See [[Feature Scaling]], [[Data Leakage Prevention]].

**"95% negative, 5% positive training data — what do you do?"** Class weights first, SMOTE if still poor, report precision/recall/F1/AUC-PR instead of accuracy, use stratified CV, and resample only after the train/test split. See [[Class Imbalance Handling]], [[Class Imbalance Evaluation]].

## End-to-End Workflow

```
Raw data
  → Split into train / val / test
  → Handle missing values      (fit imputer on train only)
  → Detect & treat outliers    (fit on train only)
  → Encode categoricals        (fit encoder on train only)
  → Scale numericals           (fit scaler on train only)
  → Handle class imbalance     (resample train only)
  → Wrap steps above in a sklearn Pipeline
  → Cross-validate             (pipeline inside the CV loop)
  → Tune hyperparameters       (on validation set)
  → Final evaluation           (test set, once)
  → Save the whole pipeline and deploy
```

## Connections

- [[Model Evaluation Index]] — what happens after data is prepared
- [[Production ML and MLOps Index]] — monitoring data quality once deployed
- [[Data Leakage]] (Statistics) — the formal theory behind [[Data Leakage Prevention]]'s practical checklist
- [[Common Mistakes in ML]] — several of its entries are data-prep mistakes

## One-line Summary

> Data preparation turns raw data into a model-ready feature matrix through a fixed set of steps — impute, encode, treat outliers, scale, rebalance — every one of which must be fit on the training split only and wrapped in a pipeline, or the resulting evaluation numbers are quietly optimistic.
