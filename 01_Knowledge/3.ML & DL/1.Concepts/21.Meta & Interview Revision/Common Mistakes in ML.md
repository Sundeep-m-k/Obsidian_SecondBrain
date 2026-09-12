# Common Mistakes in ML

A self-contained reference of the most frequent mistakes practitioners make — what they are, why they're wrong, and how to fix them.

---

## Data Mistakes

### 1. Data Leakage
**What:** Information from the future or from the test/val set influences the model during training.  
**Why it's bad:** Model looks great on paper but fails completely in production.  
**Common forms:**
- Fitting the scaler on the full dataset (including test) before splitting
- Including features that are consequences of the target (e.g., "treatment received" when predicting disease)
- Using future information in time-series (using next day's data to predict today)
- Feature selection on the full dataset before CV

**Fix:** Always split FIRST. Fit all preprocessing on training data only. In CV, preprocessing must be inside each fold.

---

### 2. Not Handling Class Imbalance
**What:** Treating a 99%/1% classification problem as if it were balanced.  
**Why it's bad:** Model achieves 99% accuracy by always predicting the majority class; recall on minority class = 0%.  
**Fix:** Use class weights, SMOTE, threshold tuning, and report F1/AUC-PR instead of accuracy.

---

### 3. Improper Train/Val/Test Split
**What:** Evaluating on the same data used for training or hyperparameter tuning.  
**Why it's bad:** Overoptimistic performance estimates; model won't generalise.  
**Fix:** Three-way split: train (fit), val (tune), test (evaluate once).

---

### 4. Not Scaling Features Before Gradient Descent
**What:** Training a linear/logistic model with raw features of vastly different scales.  
**Why it's bad:** Cost surface is highly elongated → gradient descent oscillates, converges extremely slowly or not at all.  
**Fix:** Standardise all features on the training set; apply same transform to val/test.

---

### 5. Ignoring Outliers
**What:** Feeding extreme values directly into MSE-based models.  
**Why it's bad:** A single outlier can dominate the MSE, distorting all parameter estimates.  
**Fix:** Inspect and handle outliers (cap, remove with justification, or use robust loss like Huber/MAE).

---

## Modelling Mistakes

### 6. Evaluating on Training Data
**What:** Reporting training accuracy as the model's performance.  
**Why it's bad:** A model that memorises the training set gets 100% — tells you nothing about generalisation.  
**Fix:** Always report validation/test performance. Watch the gap between train and val.

---

### 7. Using MSE for Classification
**What:** Using mean squared error as the loss for a classifier with sigmoid output.  
**Why it's bad:** Non-convex cost surface; vanishing gradients when model is confident and wrong.  
**Fix:** Use binary cross-entropy for binary classification; categorical cross-entropy for multi-class.

---

### 8. Forgetting to Regularise When $d$ is Large
**What:** Training an unregularised model with many features and few examples.  
**Why it's bad:** Model overfits heavily.  
**Fix:** Default to Ridge (L2) as a starting point. Tune $\lambda$ via CV.

---

### 9. Wrong Default Threshold
**What:** Using threshold $\tau=0.5$ for all binary classification problems.  
**Why it's bad:** 0.5 minimises misclassification rate assuming equal class prior and equal cost — rarely true in practice.  
**Fix:** Choose threshold based on cost of FP vs. FN. Tune on validation set using F1, Youden's J, or cost analysis.

---

### 10. Not Testing a Baseline
**What:** Jumping straight to complex models without a simple baseline.  
**Why it's bad:** No reference point — you don't know if your model is actually doing anything useful.  
**Fix:** Always compute:
- Regression baseline: predict $\bar{y}$ for all examples ($R^2 = 0$)
- Classification baseline: predict majority class (accuracy = class imbalance rate)

---

## Evaluation Mistakes

### 11. Not Separating Hyperparameter Tuning from Final Evaluation
**What:** Tuning hyperparameters on the test set (or looking at test set multiple times).  
**Why it's bad:** The test set becomes part of the development process — it's no longer a fair evaluation.  
**Fix:** Use val set for all tuning. Look at test set once, at the very end.

---

### 12. Reporting Accuracy on Imbalanced Data
**What:** Reporting 99% accuracy when class 0 has 99% of examples.  
**Why it's bad:** Misleading — a trivial model achieves this without learning anything.  
**Fix:** Report precision, recall, F1, and/or AUC-PR.

---

### 13. Feature Selection Before Cross-Validation
**What:** Selecting features on the whole training set and then running CV to evaluate.  
**Why it's bad:** Feature selection uses information from the validation folds → overfitting to validation data → inflated performance estimate.  
**Fix:** Feature selection must be inside each CV fold.

---

## Production Mistakes

### 14. Not Saving Preprocessing Statistics
**What:** Saving the trained model weights but forgetting to save the mean/std used for standardisation.  
**Why it's bad:** At inference time, you can't apply the same preprocessing → predictions are wrong.  
**Fix:** Save the full pipeline (scaler + model) as a unit (e.g., sklearn Pipeline, ONNX with preprocessing).

---

### 15. Ignoring Distribution Shift
**What:** Deploying a model and never monitoring whether input data distribution has drifted from training data.  
**Why it's bad:** Silent performance degradation — model quietly becomes wrong as the world changes.  
**Fix:** Monitor prediction distributions, feature distributions, and performance metrics over time. Set up drift detection alerts and periodic retraining.

---

## Connections

- [[Overfitting]] — mistakes 6, 7, 8
- [[Training Data]] — mistakes 1, 3, 4, 13
- [[Feature Engineering]] — mistake 4, 5
- [[Classification]] — mistakes 2, 9, 12
- [[Regularization]] — mistake 8
- [[Inference]] — mistake 14

---

## One-line Summary

> The most costly ML mistakes are data leakage (optimistic but wrong results), not scaling features (slow/broken training), evaluating on training data (deceptive performance), and ignoring class imbalance (misleading accuracy) — all preventable with disciplined train/val/test separation and proper preprocessing pipelines.
