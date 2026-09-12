# Topic 15: Model Evaluation & Validation — Complete Index

## Overview

This topic covers comprehensive evaluation of classification and regression models: metrics, curves, validation strategies, and workflows. Seven detailed notes build from basic metrics to production-ready evaluation.

**Total Content:** 7 files, ~45-50 KB
**Interview Questions:** 50+ across all files
**Code Examples:** 20+ worked examples and scenarios

---

## Files at a Glance

| File | Focus | Size | Key Concepts |
|------|-------|------|--------------|
| [[Classification Metrics]] | Confusion matrix, accuracy, precision, recall, F1 | 4.5 KB | Core metrics, trade-offs |
| [[ROC and PR Curves]] | AUC-ROC, AUC-PR, threshold selection, curve interpretation | 5 KB | Ranking ability, imbalance handling |
| [[Calibration and Probability Evaluation]] | Calibration, Brier score, reliability diagrams, recalibration | 5.5 KB | Probability accuracy, proper scoring rules |
| [[Cross Validation Strategy]] | k-fold, stratified CV, nested CV, time-series CV | 5.5 KB | Stable estimates, hyperparameter tuning |
| [[Train Val Test Framework]] | 3-way split, data leakage prevention, split sizes | 5 KB | Train/val/test separation, workflow |
| [[Class Imbalance Evaluation]] | Precision/recall for imbalance, class weights, SMOTE, AUC-PR | 5.5 KB | Imbalanced data, cost-sensitive learning |
| [[Evaluation Workflow and Baselines]] | End-to-end workflow, baseline design, deployment decision | 5 KB | From baseline to shipping |

---

## Reading Paths by Role

### Data Analyst

**Priority files:** 15.1, 15.7, 15.5 (fundamentals)
**Secondary:** 15.6, 15.2, 15.3, 15.4 (depth)

**Rationale:** Analysts focus on reporting metrics and understanding data quality. Calibration and nested CV less critical.

**1-day sprint:** Read 15.1 (metrics) + 15.7 (workflow) = 80% readiness
**1-week deep dive:** Add 15.5 (train/val/test), 15.6 (imbalance)

---

### Machine Learning Engineer

**Priority files:** 15.1, 15.2, 15.4, 15.5 (core)
**Secondary:** 15.3, 15.6, 15.7 (domain-specific)

**Rationale:** ML engineers need CV, curves, thresholding, and workflow. Calibration for probabilistic models.

**1-day sprint:** Read 15.1 + 15.2 + 15.4 + 15.5 = 85% readiness
**1-week deep dive:** Add 15.3 (calibration), 15.6 (imbalance), 15.7 (decisions)

---

### Data Scientist / ML Researcher

**Priority files:** 15.1, 15.2, 15.3, 15.4, 15.6 (comprehensive)
**Secondary:** 15.5, 15.7 (applied)

**Rationale:** Researchers need deep understanding of metrics, curves, and statistical validity. Calibration important for generalization.

**1-day sprint:** Read 15.1 + 15.2 + 15.3 + 15.4 = 90% readiness
**1-week deep dive:** Add 15.6 (imbalance deep dive), 15.5, 15.7

---

### ML Systems Engineer / MLOps

**Priority files:** 15.4, 15.5, 15.7 (operational)
**Secondary:** 15.1, 15.2, 15.6 (metrics context)

**Rationale:** Systems engineers focus on workflows, CV pipelines, and deployment. Need to understand metric selection and stability.

**1-day sprint:** Read 15.5 + 15.7 = 75% readiness (focus: train/val/test, decision criteria)
**1-week deep dive:** Add 15.4 (CV pipelines), 15.1 (metrics for monitoring), 15.6 (imbalance)

---

## Interview Preparation Strategy

### 1-Day Interview Prep (80% readiness)

**Read:** 15.1 + 15.2 + 15.7 (1.5 hours)
**Focus:** Confusion matrix, precision/recall, AUC, baselines, when to ship
**Practice:** 10 Q&A from 15.1 and 15.2

---

### 3-Day Interview Prep (90% readiness)

**Day 1:** 15.1 (Classification Metrics) — 1 hour
**Day 2:** 15.2 (ROC/PR Curves) + 15.5 (Train/Val/Test) — 1.5 hours
**Day 3:** 15.4 (Cross-Validation) + 15.6 (Imbalance) + 15.7 (Workflow) — 1.5 hours
**Practice:** 25 Q&A total; focus on explaining trade-offs

---

### 1-Week Interview Prep (95% readiness)

**Read all 7 files:** 3-4 hours total
**Practice:** All 50+ Q&A
**Worked examples:** Code up confusion matrix, plot ROC, implement CV
**System design:** How would you build an evaluation system for a real product?

---

## Key Topics by Interview Frequency

### Tier 1: Always Asked (100% probability)
- Confusion matrix, precision, recall, F1 ([[Classification Metrics]])
- Accuracy on imbalanced data ([[Class Imbalance Evaluation]])
- Train/val/test split ([[Train Val Test Framework]])
- What metrics to use for your problem ([[Classification Metrics]], [[Class Imbalance Evaluation]])

### Tier 2: Often Asked (70% probability)
- AUC-ROC vs. AUC-PR ([[ROC and PR Curves]])
- Cross-validation ([[Cross Validation Strategy]])
- Data leakage examples ([[Train Val Test Framework]])
- Baseline design ([[Evaluation Workflow and Baselines]])

### Tier 3: Sometimes Asked (40% probability)
- Calibration ([[Calibration and Probability Evaluation]])
- Nested CV ([[Cross Validation Strategy]])
- Stratified CV ([[Cross Validation Strategy]])
- Class weights vs. threshold tuning ([[Class Imbalance Evaluation]])

### Tier 4: Rarely Asked (10% probability)
- Brier score ([[Calibration and Probability Evaluation]])
- Temperature scaling ([[Calibration and Probability Evaluation]])
- MCC (Matthews Correlation Coefficient) ([[Class Imbalance Evaluation]])

---

## Common Interview Scenarios

### Scenario 1: "How do you evaluate a binary classifier?"

**Answer structure:**
1. Start with confusion matrix (TP, FP, FN, TN)
2. Compute precision, recall, F1
3. Check: Is data balanced? If not, mention AUC-PR
4. Use threshold tuning for cost-sensitive scenarios
5. Report baseline for context
6. Mention CV for stable estimate

**References:** [[Classification Metrics]], [[Class Imbalance Evaluation]], [[Evaluation Workflow and Baselines]]

---

### Scenario 2: "Your model gets 95% accuracy. Is this good?"

**Red flag questions:**
1. What's the class distribution? (If 95% baseline, not impressive)
2. What's your baseline? (Need baseline to compare)
3. What metric did you optimize for? (Accuracy misleading for imbalance)

**Answer:** Depends on baseline and class distribution. Always report precision, recall, F1, not just accuracy. If baseline is 95%, model is useless. See [[Class Imbalance Evaluation]], [[Classification Metrics]]

---

### Scenario 3: "Train accuracy 95%, test accuracy 80%. What's happening?"

**Diagnosis:** Overfitting (large gap).

**Solutions:**
1. More regularization (L1/L2)
2. More data
3. Simpler model
4. Feature selection

**Prevention:** Use CV with consistent folds; watch train/val gap during development.

**References:** [[Cross Validation Strategy]], [[Train Val Test Framework]]

---

### Scenario 4: "How would you evaluate a model on imbalanced data?"

**Answer:**
1. Never use accuracy alone
2. Use precision/recall; report both
3. F1 or AUC-PR as summary metric
4. Use stratified CV to preserve class distribution
5. Consider class weights or SMOTE
6. Threshold tuning for asymmetric costs

**References:** [[Class Imbalance Evaluation]], [[ROC and PR Curves]]

---

### Scenario 5: "You're selecting between Model A (85% accuracy, stable) and Model B (87% accuracy, high variance). Which do you pick?"

**Answer:** Model A.

**Reasoning:**
- Higher variance = unstable; different results on different data
- 2% improvement doesn't justify instability
- In production, Model B is unpredictable
- Stable > Slightly better but unstable

**Decision criteria:** Prioritize stability, then interpretability, then performance.

**References:** [[Evaluation Workflow and Baselines]], [[Cross Validation Strategy]]

---

## Quick Reference: When to Use Each Metric

```
Balanced data? → Accuracy, F1
Imbalanced data? → Precision, Recall, F1, AUC-PR
Need single number? → F1 or AUC
Probabilistic output? → Calibration, Brier score
Cost-sensitive? → Cost-weighted metric, threshold tuning
Ranking ability? → AUC-ROC
Ranking + imbalance? → AUC-PR
Regression? → RMSE, MAE, R²
```

See [[Classification Metrics]], [[ROC and PR Curves]], [[Class Imbalance Evaluation]]

---

## Links to Other Topics

**Foundational (read before this topic):**
- [[Classical ML Algorithms Index]] — Models being evaluated
- [[Logistic Regression Index]] — Binary classification model

**Related (read alongside or after):**
- [[Feature Engineering and Data Preparation Index]] — Quality of data affects evaluation stability
- [[Production ML and MLOps Index]] — Monitoring and retraining

**Prerequisite knowledge:**
- [[Foundations Index]] — Bias-variance tradeoff
- [[Model Behavior Index]] — Overfitting/underfitting
- [[Regularization Index]] — How regularization affects metrics

---

## One-Page Cheat Sheet

**Confusion Matrix → Metrics:**
- Accuracy = (TP+TN)/n (don't use if imbalanced)
- Precision = TP/(TP+FP) (use if FP costly)
- Recall = TP/(TP+FN) (use if FN costly)
- F1 = 2·P·R/(P+R) (balanced metric for imbalance)

**Curves:**
- ROC: TPR vs. FPR; AUC = ranking ability (0.5 = random, 1.0 = perfect)
- PR: Precision vs. Recall; better for imbalanced data

**Validation:**
- Train/Val/Test: Learn / Tune / Evaluate (strictly separate)
- CV: k-fold for stable estimate; stratified for imbalance
- Nested CV: Outer loop tests, inner loop tunes

**Data Leakage:** Always split FIRST, then fit preprocessing on train only.

**Baselines:** Regression = mean; Classification = majority class.

---

## Master Study Order

**Day 1:** 15.1 → 15.7 (see big picture)
**Day 2:** 15.2, 15.5 (curves, train/val/test)
**Day 3:** 15.4, 15.6 (CV, imbalance)
**Day 4:** 15.3 (calibration, depth)
**Days 5-7:** Practice Q&A, code examples, system design

---

## One-line Summary per File

| File | Summary |
|------|---------|
| 15.1 | Confusion matrix → Precision, Recall, F1; they're in tension via threshold |
| 15.2 | ROC curves measure ranking (AUC); PR curves better for imbalanced data |
| 15.3 | Calibration = predicted probs match observed frequencies; independent of ranking |
| 15.4 | k-fold CV gives stable estimate; nested CV for tuning; stratified for imbalance |
| 15.5 | Train/Val/Test strict separation prevents data leakage and bias |
| 15.6 | Imbalanced data requires Precision/Recall/F1/AUC-PR, not Accuracy; use class weights |
| 15.7 | Evaluation workflow: Baseline → Train/Val/Test → Metrics → Compare → Ship decision |

---

**Total Estimated Study Time:** 4-5 hours (skim), 8-10 hours (deep)
**Interview Readiness:** 80% after 1 file, 90% after 4 files, 95% after all 7
