# Topic 16: Data Preparation — Complete Index

## Overview

Data preparation transforms raw, messy data into clean, model-ready format. Seven detailed notes cover missing values, categorical encoding, outliers, scaling, imbalance handling, leakage prevention, and pipelines.

**Total Content:** 7 files, ~43-48 KB
**Interview Questions:** 45+ across all files
**Code Examples:** 30+ worked examples

---

## Files at a Glance

| File | Focus | Key Concepts |
|------|-------|--------------|
| [[16.1_Missing_Values_Handling]] | Detection, imputation strategies (mean, KNN, MICE) | MCAR, MAR, MNAR |
| [[16.2_Categorical_Encoding]] | Label, one-hot, target encoding, high-cardinality handling | Ordinal vs. nominal |
| [[16.3_Outlier_Detection_and_Treatment]] | IQR, Z-score, isolation forest; deletion, capping, transformation | Impact on models |
| [[16.4_Feature_Scaling_and_Normalization]] | Standardization, min-max, robust scaling, log transform | Prevent leakage |
| [[16.5_Class_Imbalance_Handling]] | Class weights, SMOTE, oversample, undersample | Resampling workflow |
| [[16.6_Data_Leakage_Prevention]] | Types of leakage, detection, prevention checklist | Critical for accuracy |
| [[16.7_Preprocessing_Pipelines]] | sklearn Pipeline, ColumnTransformer, serialization | Reproducibility |

---

## Reading Paths by Role

### Data Analyst

**Priority:** 16.1, 16.3, 16.4 (data quality + handling)
**Secondary:** 16.2, 16.5, 16.6, 16.7

**1-day sprint:** 16.1 + 16.3 + 16.4 = 80% readiness
**1-week:** Add 16.2, 16.5, 16.6

---

### ML Engineer

**Priority:** 16.1, 16.2, 16.4, 16.6, 16.7 (complete pipeline)
**Secondary:** 16.3, 16.5

**1-day sprint:** 16.1 + 16.4 + 16.6 + 16.7 = 85% readiness
**1-week:** Add 16.2, 16.3, 16.5

---

### ML Systems / MLOps

**Priority:** 16.6, 16.7 (reproducibility, deployment)
**Secondary:** 16.1, 16.2, 16.4, 16.5

**1-day sprint:** 16.6 + 16.7 = 75% readiness
**1-week:** Add 16.1, 16.4, 16.2

---

## Interview Preparation

### 1-Day Prep (80%)
Read: 16.1 + 16.4 + 16.6 (1.5 hours)
Focus: Missing values, scaling, leakage prevention
Practice: 15 Q&A

### 3-Day Prep (90%)
Day 1: 16.1 (Missing values)
Day 2: 16.4 (Scaling) + 16.6 (Leakage)
Day 3: 16.2 (Encoding) + 16.5 (Imbalance) + 16.7 (Pipelines)
Practice: 30 Q&A

### 1-Week Prep (95%)
Read all 7 files (3-4 hours)
Practice: All 45+ Q&A
Code: Implement preprocessing, SMOTE, pipeline
Interview scenario: "Walk me through data prep for an imbalanced classification problem"

---

## Key Topics by Interview Frequency

### Tier 1: Always Asked (100%)
- How to handle missing values (delete vs. impute)
- Why scale features for gradient-based algorithms
- Preventing data leakage (fit on train, apply to test)
- Handling categorical features (one-hot vs. label)

### Tier 2: Often Asked (70%)
- Class imbalance solutions (class weights, SMOTE)
- Outlier detection and treatment
- Preprocessing pipeline benefits

### Tier 3: Sometimes Asked (40%)
- Difference between standardization and min-max
- MCAR vs. MAR vs. MNAR
- Target encoding for high cardinality

### Tier 4: Rarely Asked (10%)
- Isolation forest for outliers
- Box-Cox transformation
- ColumnTransformer details

---

## Common Interview Scenarios

### Scenario 1: "How do you handle missing values?"

**Answer structure:**
1. How much is missing? (<5% → delete/mean; 5-20% → KNN; >20% → drop feature)
2. Is it MCAR, MAR, or MNAR? (Affects choice)
3. Which imputation method? (Simple = mean; better = KNN/MICE)
4. **Critical:** Fit imputer on train, apply to test (prevent leakage)
5. Evaluate impact (did imputation introduce bias?)

**References:** [[16.1_Missing_Values_Handling]], [[16.6_Data_Leakage_Prevention]]

---

### Scenario 2: "You have 1000 categorical features, some with 10K unique values. How would you encode them?"

**Red flag:** One-hot on 10K categories = 10K columns (sparse, inefficient)

**Solutions:**
1. **Target encoding:** Replace with mean target per category (single column)
2. **Frequency encoding:** Replace with category count
3. **Embedding (DL):** Learn 10-50 dimensional vectors
4. **Grouping:** Combine rare categories as "Other"; reduces to 100 categories

**Best for supervised:** Target encoding

**References:** [[16.2_Categorical_Encoding]], [[15.6_Class_Imbalance_Evaluation]]

---

### Scenario 3: "Why would you NOT scale features for a decision tree but WOULD for logistic regression?"

**Answer:**
Trees use splits (threshold comparisons), not distances. A split "age > 30" works the same whether age is 0-100 or 0-1000. Scaling doesn't help trees.

Linear models use gradients (distance-based). Gradient step size for age (scale 0-100) is tiny vs. income (scale 0-10M). Without scaling, gradient descent zigzags; converges slowly or not at all. Scaling fixes this.

**References:** [[16.4_Feature_Scaling_and_Normalization]]

---

### Scenario 4: "You fit StandardScaler on all 1000 examples, then split 70/30 train/test. Is this leakage?"

**Yes.** Scaler learned mean/std from test examples (implicitly). Test statistics influenced preprocessing.

**Correct:** Split first (700 train, 300 test). Fit StandardScaler on train only. Apply train's mean/std to test.

**References:** [[16.4_Feature_Scaling_and_Normalization]], [[16.6_Data_Leakage_Prevention]]

---

### Scenario 5: "Your training data is 95% negative, 5% positive. How do you handle this?"

**Multi-pronged approach:**
1. Use class weights in model (weight_1 = 19, weight_0 ≈ 1)
2. If still poor: SMOTE (generate synthetic positives)
3. For evaluation: Report precision, recall, F1, AUC-PR (not accuracy)
4. Use stratified CV (preserve class distribution per fold)
5. When using SMOTE: Resample after train/test split (prevent leakage)

**References:** [[16.5_Class_Imbalance_Handling]], [[15.6_Class_Imbalance_Evaluation]]

---

## Workflow: End-to-End Data Prep

```
Raw data arrives
        ↓
Step 1: SPLIT into train / val / test
        ↓
Step 2: Handle missing values (fit imputer on train only)
        ↓
Step 3: Detect & treat outliers (fit on train only)
        ↓
Step 4: Encode categoricals (fit encoder on train only)
        ↓
Step 5: Scale numericals (fit scaler on train only)
        ↓
Step 6: Handle class imbalance (resample train only)
        ↓
Step 7: Build sklearn Pipeline (automates steps 2-6)
        ↓
Step 8: Cross-validate (pipeline inside CV loop)
        ↓
Step 9: Hyperparameter tuning (on validation set)
        ↓
Step 10: Final evaluation on test set
        ↓
Step 11: Save entire pipeline + deploy
```

---

## Quick Reference: Decision Trees

```
Missing values?
  < 5% → Delete
  5-20% → Mean/median
  > 20% → Domain imputation or drop

Categorical feature?
  Ordinal (has order) → Label encoding
  Nominal (no order)
    Few categories → One-hot
    Many categories → Target encoding or embedding

Outliers?
  Errors → Delete or cap
  Real signal → Keep or transform (log)
  Model is tree → No treatment needed

Feature scaling?
  Linear/KNN/SVM → MUST scale (StandardScaler or MinMaxScaler)
  Tree models → No scaling needed

Class imbalance?
  Try class weights first
  If still poor → SMOTE
  Resample AFTER split (prevent leakage)

Data leakage?
  SPLIT first
  Fit ALL preprocessing on TRAIN only
  Apply to VAL/TEST using TRAIN's statistics
  Use sklearn Pipeline to automate
```

See [[16.1_Missing_Values_Handling]], [[16.2_Categorical_Encoding]], [[16.3_Outlier_Detection_and_Treatment]], [[16.4_Feature_Scaling_and_Normalization]], [[16.5_Class_Imbalance_Handling]], [[16.6_Data_Leakage_Prevention]], [[16.7_Preprocessing_Pipelines]]

---

## Links to Other Topics

**Related (before data prep):**
- [[15.5_Train_Val_Test_Framework]] — Train/val/test split

**Related (after data prep):**
- [[15.1_Classification_Metrics]] — Evaluate prepared data
- [[18.2_Model_Monitoring_in_Production]] — Monitor data quality

**Connections:**
- [[Common Mistakes in ML]] — References mistakes throughout

---

## Master Study Order

**Day 1:** 16.1 → 16.6 (see big picture)
**Day 2:** 16.4, 16.7 (scaling, pipelines)
**Day 3:** 16.2, 16.5 (encoding, imbalance)
**Day 4:** 16.3 (outliers, detail)
**Days 5-7:** Practice Q&A, code examples, scenarios

---

## One-line Summary per File

| File | Summary |
|------|---------|
| 16.1 | Missing < 5% delete; 5-20% mean/KNN; > 20% drop feature — fit on train, apply to test |
| 16.2 | Ordinal → label encode; nominal → one-hot (few) or target encode (many); prevent leakage |
| 16.3 | Outliers: detect via IQR/Z-score, treat via delete/cap/transform — trees robust, linear sensitive |
| 16.4 | Standardize (unbounded) or min-max (bounded); fit on train, apply to test; no scaling for trees |
| 16.5 | Class weights (simplest), SMOTE (standard), SMOTE+undersample (combined) — resample after split |
| 16.6 | Leakage: fit preprocessing on train only, apply to test; use pipelines to automate |
| 16.7 | sklearn Pipeline chains preprocessing + model; prevents leakage; pickle for serialization |

---

**Total Estimated Study Time:** 4-5 hours (skim), 8-10 hours (deep)
**Interview Readiness:** 80% after 3 files, 90% after 6 files, 95% after all 7
