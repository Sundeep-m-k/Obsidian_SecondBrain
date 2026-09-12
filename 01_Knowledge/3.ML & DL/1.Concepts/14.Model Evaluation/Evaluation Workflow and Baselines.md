# Evaluation Workflow and Baselines

## What is it?

An **evaluation workflow** is the structured process of assessing model performance end-to-end: from baseline setup through validation to final testing and deployment decision.

A **baseline** is a simple reference model or heuristic. Beating a baseline proves the model is learning; not beating it means the model is useless.

---

## Why Baselines Matter

### The Baseline Principle

> If your model doesn't beat the baseline, it's worse than the baseline. Always compute one.

**Without baseline:** Model gets 85% accuracy. Good? Unknown. Could be that random guessing achieves 80%.

**With baseline:** Model gets 85% accuracy. Baseline gets 60%. Now you know the model adds 25% relative improvement.

### Types of Baselines

| Baseline | Task | How to compute | Expected accuracy |
|----------|------|-----------------|-------------------|
| **Always predict mean** | Regression | $\hat{y} = \bar{y}$ for all $x$ | $R^2 = 0$ (by definition) |
| **Always predict majority** | Classification | $\hat{y} = 1[\text{class 0 > class 1}]$ | Majority class % |
| **Random forest (default)** | Any | Fit RF with default hyperparams | Usually 70-80% on medium data |
| **Logistic regression** | Classification | Fit LR with L2 regularization | Usually 70-85% on medium data |
| **Domain heuristic** | Domain-specific | E.g., "flag as fraud if amount > $1000" | Varies by domain |

---

## Regression Baseline

$$\hat{y} = \bar{y} = \frac{1}{n} \sum_{i=1}^n y_i$$

Always predict the mean training value.

**Performance:** $R^2 = 0$ (by definition).

**Why:** If model gets $R^2 < 0$, it's worse than guessing mean. Red flag.

**Example:** Predicting house prices, training mean = $500k.
- Baseline: Always predict $500k. RMSE = $150k.
- Model A: RMSE = $100k. ✓ Better than baseline.
- Model B: RMSE = $200k. ✗ Worse than baseline.

---

## Classification Baseline

$$\hat{y} = \arg\max_c n_c = \text{majority class}$$

Always predict the most common class.

**Performance:** Accuracy = $\frac{n_{\text{majority}}}{n}$.

**Why:** If model gets lower accuracy, it's worse than predicting majority class.

**Example:** 90% negative class.
- Baseline: Always predict negative. Accuracy = 90%.
- Model A: Accuracy = 92%. ✓ Better than baseline.
- Model B: Accuracy = 88%. ✗ Worse than baseline.
- Model C: Accuracy = 95%, Recall = 30%. Looks good until you check recall; catching few positives.

**Variant (imbalanced data):** Report both accuracy and recall/F1 so baseline weakness on minority class is visible.

---

## Complete Evaluation Workflow

### Phase 1: Before Model Building

#### 1.1 Define Success Metric

**Ask:**
- What is the business goal? (Precision? Recall? Cost-weighted?)
- What is the problem: Imbalanced? Regression? Multi-class?
- What are the costs of errors? (FP vs. FN)

**Decision:**
- Balanced data, equal error costs → Accuracy, F1
- Imbalanced data → Precision, Recall, F1, AUC-PR
- Cost asymmetry known → Cost-weighted metric
- Regression → RMSE, MAE, R²

#### 1.2 Compute Baseline

**For regression:** Fit model that always predicts $\bar{y}_{\text{train}}$.
```
Baseline RMSE = sqrt(mean((y_test - mean(y_train))^2))
```

**For classification:** Always predict majority class.
```
Baseline Accuracy = count(y_test == majority_class) / len(y_test)
```

**Record baseline metric** — this is your minimum bar.

### Phase 2: Model Development

#### 2.1 Train/Val/Test Split

```
Data → Split into train (70%), val (15%), test (15%)
```

Never train on val or test.

#### 2.2 Try Multiple Models

```
for model in [LinearRegression, RandomForest, GradientBoosting]:
    for hyperparams in [grid of options]:
        train model on train set
        evaluate on val set
        record val metric
```

#### 2.3 Select Best Model

```
best_model = argmax(val_metrics)
record best hyperparams
```

#### 2.4 Assess Overfitting

```
train_metric = evaluate(best_model, X_train, y_train)
val_metric = evaluate(best_model, X_val, y_val)

if train_metric >> val_metric:
    print("Overfitting detected. Regularize or get more data.")
else:
    print("Good generalization.")
```

### Phase 3: Final Evaluation

#### 3.1 Test Set Evaluation

```
test_metric = evaluate(best_model, X_test, y_test)
```

Look at test set **exactly once**. Never tune based on it.

#### 3.2 Report Comprehensive Metrics

**For classification:**
- Confusion matrix (TP, FP, FN, TN)
- Precision, Recall, F1
- AUC-PR (if imbalanced)
- AUC-ROC (for context)

**For regression:**
- RMSE, MAE, R²
- Residual analysis (are residuals Gaussian?)

#### 3.3 Compare to Baseline

```
Model performance vs. Baseline:
  Baseline: 0.60
  Model:    0.85
  Improvement: +25% relative
```

**Decision:**
- Model >> Baseline: ✓ Ship it
- Model > Baseline (small): ? Marginal; consider cost/complexity
- Model ≈ Baseline: ✗ Not worth it
- Model < Baseline: ✗ Useless; redesign

### Phase 4: Deployment Decision

#### 4.1 Sanity Checks

Before shipping:
- [ ] Does model beat baseline significantly?
- [ ] Is performance stable across CV folds? (Low std dev)
- [ ] Are residuals/errors interpretable? (No weird patterns)
- [ ] Have edge cases been tested? (Outliers, missing data)
- [ ] Is model inference speed acceptable?

#### 4.2 Monitoring Plan

Once deployed:
- [ ] Track predictions distribution over time
- [ ] Monitor performance on new data (manually label some)
- [ ] Set up alerts for performance drops
- [ ] Plan retraining schedule

See [[Model Monitoring in Production]] for details.

---

## Worked Example: Email Spam Classification

### Problem Setup

**Goal:** Classify emails as spam (1) or ham (0).
**Data:** 10,000 emails; 1,000 spam (10%), 9,000 ham (90%).
**Metric:** Precision (don't want to block legitimate emails).

### Workflow

**Phase 1: Baseline**
```
Baseline: Always predict "ham"
Accuracy = 9000/10000 = 90%
Precision = undefined (predicts 0 spams)
Recall = 0% (catches no spam)

Issues: High accuracy but useless (catches nothing).
```

Better metrics for this task: Precision, Recall, F1.

**Phase 2: Development**

```
Train/Val/Test: 7000 train, 1500 val, 1500 test

Model 1: Logistic Regression, no regularization
  Train Acc: 95%, Val Acc: 91%, Val Precision: 0.85, Val Recall: 0.70
  
Model 2: Logistic Regression, L2 regularization (λ=0.1)
  Train Acc: 93%, Val Acc: 92%, Val Precision: 0.88, Val Recall: 0.72
  
Model 3: Random Forest, depth=10
  Train Acc: 98%, Val Acc: 92%, Val Precision: 0.90, Val Recall: 0.75

Best on val: Model 3 (highest Recall)
```

**Phase 3: Final Evaluation**

```
Test Set Results:
  Accuracy: 91%
  Precision: 0.89 (of 300 predicted spams, 267 correct)
  Recall: 0.76 (of 150 actual spams in test, caught 114)
  F1: 0.82

Confusion Matrix:
             Predicted Spam  Predicted Ham
  Actual Spam       114              36
  Actual Ham         33            1317

Comparison to Baseline:
  Baseline (always ham): Precision undefined, Recall 0%
  Model: Precision 89%, Recall 76%
  
Decision: ✓ Ship it. Catches 76% of spam while blocking only ~2% of legitimate emails.
```

---

## When to Ship vs. When to Redesign

### Ship If

- [ ] Model beats baseline by significant margin (>20% relative improvement)
- [ ] Val and test metrics are similar (not overfitting)
- [ ] CV folds show consistent performance (low std dev)
- [ ] No systematic errors on important subsets
- [ ] Performance is acceptable for business requirements

### Redesign If

- [ ] Model barely beats baseline (< 10% improvement)
- [ ] Large gap between train and val (overfitting)
- [ ] Test performance lower than val (generalization failure)
- [ ] High variance across CV folds (unstable model)
- [ ] Systematic failures on specific subsets (e.g., one demographic)

### Examples

**Case A:** Baseline 60%, Model 85% → Ship (42% improvement)
**Case B:** Baseline 60%, Model 62% → Reconsider (3% improvement; is it worth complexity?)
**Case C:** Baseline 60%, Model 58% → Redesign (useless)

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| No baseline | Can't assess if model is useful | Always compute baseline first |
| Wrong baseline | Misleading comparison | Use majority class or mean; not a simple model |
| Report accuracy on imbalanced data | Misleading; hides poor minority-class performance | Report Precision, Recall, F1 |
| Only report val metric, hide test | Optimistic estimate | Always report test; difference reveals overfitting |
| Tweak hyperparams based on test results | Test set is biased | Tune only on val; test once, final |
| Ship without testing edge cases | Fails in production | Test on outliers, missing data, rare classes |

---

## Interview Questions

**Q: What is a baseline and why is it important?**

A baseline is a simple reference model or heuristic. It's important because it defines the minimum bar—if your model doesn't beat the baseline, it's worse than the baseline and useless. For regression, baseline = always predict mean ($R^2 = 0$). For classification, baseline = always predict majority class. Example: 85% accuracy sounds good but is useless if baseline is 80%. See [[Common Mistakes in ML#10 - Not Testing a Baseline]].

---

**Q: Walk me through your end-to-end evaluation workflow.**

(1) Define success metric and compute baseline. (2) Split data into train/val/test. (3) Try multiple models and hyperparams; evaluate on val set. (4) Select best model based on val metric. (5) Evaluate on test set exactly once (look once, report, done). (6) Report comprehensive metrics (precision, recall, F1 for classification; RMSE, R² for regression). (7) Compare model to baseline. (8) Ship if model beats baseline significantly, redesign if not. (9) Monitor in production. See [[Train Val Test Framework]].

---

**Q: How do you decide between two models that have similar test performance?**

Prefer: (1) Simpler model (fewer hyperparams, faster training). (2) Model with lower variance across CV folds (more stable). (3) Model with better interpretability (explainability matters). (4) Model with faster inference (for production). (5) Model with fewer dependencies (easier to maintain). If truly equivalent, simpler is better. Example: Logistic regression with 0.91 F1 vs. random forest with 0.92 F1. Logistic regression is preferable unless the 1% improvement is critical. See [[Cross Validation Strategy]].

---

**Q: When should you not ship a model even if it beats baseline?**

If: (1) Improvement is marginal (<10% over baseline) and model is complex. (2) Test performance is much lower than val (generalization failure; might fail in production). (3) High variance across CV folds (unstable; inconsistent in production). (4) Systematic errors on important subsets (e.g., fails on rare classes). (5) Inference latency is unacceptable. (6) Not enough data in production to monitor for drift. Prioritize stability and robustness over marginal gains. See [[Model Monitoring in Production]].

---

**Q: How do you handle the case where model beats baseline but not by much?**

Evaluate trade-offs: (1) Cost of wrong predictions (if FN costs $100 but model prevents 1 FN per month, value is $100/month, which might not justify maintenance overhead). (2) Complexity (is it worth maintaining a complex model for 5% improvement?). (3) Interpretability (simpler baseline is more trustworthy; complex model needs to prove worth). (4) Volume (if processing millions daily, small improvement at scale matters). Decision: Ship if value > maintenance cost, otherwise use baseline or simpler model. See [[Online vs Offline Evaluation]].

---

## Connections

- [[Classification Metrics]] — Metrics in evaluation workflow
- [[Train Val Test Framework]] — Train/val/test split
- [[Cross Validation Strategy]] — CV for evaluation
- [[Class Imbalance Evaluation]] — When baseline is weak on imbalanced data
- [[Model Monitoring in Production]] — Monitoring after deployment
- [[Common Mistakes in ML#10 - Not Testing a Baseline]] — Pitfall
- [[Common Mistakes in ML#6 - Evaluating on Training Data]] — Pitfall

---

## One-line Summary

> Evaluation workflow: (1) compute baseline, (2) split train/val/test, (3) tune on val, (4) evaluate on test once, (5) report comprehensive metrics, (6) compare to baseline, (7) ship if meaningful improvement and stable performance, (8) monitor in production.
