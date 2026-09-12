# Classification Pipeline

## What is it?

The **classification pipeline** is the end-to-end workflow for building a classification system — from raw data to a deployed prediction service.

---

## The Full Pipeline

```
1. Problem Definition
        ↓
2. Data Collection & Labelling
        ↓
3. Exploratory Data Analysis (EDA)
        ↓
4. Data Preprocessing
        ↓
5. Feature Engineering
        ↓
6. Model Selection
        ↓
7. Training
        ↓
8. Evaluation (Train / Val)
        ↓
9. Threshold Selection
        ↓
10. Hyperparameter Tuning
        ↓
11. Final Test Set Evaluation
        ↓
12. Deployment
        ↓
13. Monitoring
```

---

## Step-by-Step Details

### 1. Problem Definition
- Is output discrete? → Classification ✅
- Binary ($K=2$) or multi-class ($K>2$)?
- What is the class imbalance? (Check $\frac{n_{\text{positive}}}{n}$)
- What is the cost of FP vs. FN? (Determines which metric to optimise)

### 2. Data Collection & Labelling
- Gather raw data (images, text, tabular).
- Obtain labels: human annotation, programmatic labelling, distant supervision.
- Check: inter-annotator agreement $\kappa$, label noise.

### 3. Exploratory Data Analysis
- Class distribution: are classes balanced?
- Feature distributions per class: do features differ between classes?
- Missing values, outliers.
- Pair plots, correlation matrices.

### 4. Data Preprocessing
- **Train/Val/Test split first** (e.g., 70/15/15 or 80/10/10).
- Handle missing values.
- Encode categoricals.
- **Standardise** numerical features (fit on train, apply to all).

### 5. Feature Engineering
- Create domain-specific features.
- Add polynomial/interaction terms if needed.
- For text: TF-IDF, embeddings.

### 6. Model Selection
Start simple, increase complexity only when needed:

| Model | Use when |
|---|---|
| Logistic Regression | Linear boundary, interpretability needed, baseline |
| Decision Tree | Non-linear, need explainability |
| Random Forest | Robust non-linear baseline |
| Gradient Boosting | Best performance on tabular data |
| Neural Network | Complex patterns, images, text, large data |

### 7. Training
- Fit on training set.
- For logistic regression: gradient descent on cross-entropy.
- For imbalanced data: set `class_weight='balanced'` or use SMOTE.

### 8. Evaluation
Report on the **validation set**:

$$\text{Accuracy} = \frac{TP+TN}{n}, \quad \text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}$$
$$F_1 = \frac{2 \cdot P \cdot R}{P+R}, \quad \text{AUC-ROC}, \quad \text{AUC-PR (for imbalanced)}$$

**Compare** train vs. val metrics:
- Train ≈ Val (both bad): underfitting → more complex model or more features
- Train good, Val bad: overfitting → regularise, more data, simpler model

### 9. Threshold Selection
Default threshold is 0.5 but may not be optimal. Use:
- ROC curve to find threshold maximising Youden's J = TPR − FPR.
- PR curve to find threshold maximising $F_1$.
- Cost analysis: $\tau^* = \frac{c_{FP}}{c_{FP} + c_{FN}}$.

**Tune threshold on validation set.** Never on test set.

### 10. Hyperparameter Tuning
- Regularisation strength $\lambda$ (C in sklearn = 1/λ).
- Model-specific params (max depth, n_estimators, etc.).
- Use grid search or random search with cross-validation.

### 11. Final Test Set Evaluation
- Evaluate once on the test set.
- Report all metrics including confusion matrix.
- Never tune further after seeing test results.

### 12. Deployment
- Serialise model + preprocessing pipeline together.
- Expose as API endpoint.
- Preprocessing at inference must match training exactly.

### 13. Monitoring
- Track predicted class distribution over time.
- Alert on distribution shift.
- Monitor precision/recall on labelled production data.
- Retrain periodically.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| Scaling after splitting | Data leakage | Split first, fit scaler on train only |
| Accuracy with imbalanced data | 99% accuracy, 0% recall | Use F1, AUC-PR |
| Default threshold 0.5 | Suboptimal for asymmetric costs | Tune on validation set |
| Evaluate on training data | Overfitting goes undetected | Always use held-out val/test |
| Feature selection on all data | Leakage | Feature selection inside CV loop |

---

## Connections

- [[Classification]] — the task
- [[Logistic Regression]] — common classifier
- [[Feature Engineering]] — step 5
- [[Feature Scaling]] — step 4
- [[Thresholding]] — step 9
- [[Overfitting]] — what step 8 diagnoses
- [[Regularization]] — the fix for overfitting

---

## One-line Summary

> The classification pipeline is the systematic workflow from raw labelled data to a deployed classifier — its critical checkpoints are the train/val/test split, class imbalance handling, metric selection, and threshold tuning, all before a single evaluation on the sacred test set.
