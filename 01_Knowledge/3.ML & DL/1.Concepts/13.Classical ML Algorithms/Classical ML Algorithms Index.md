---
tags: [category/ml-dl, index, moc]
---

# Classical ML Algorithms — Index

> **Position in vault**: `3.ML & DL/1.Concepts/13.Classical ML Algorithms/`
> **Purpose**: The standard non-linear, non-neural model families. Renamed from "Tree-Based & Ensemble Methods" (2026-09) because that name no longer describes the module's intended scope — SVM, Naive Bayes, and KNN are planned additions here (see status below), not just trees and ensembles.
> **Prerequisite**: [[Supervised Learning Index]], [[Model Behavior Index]]

## Section Map

**Trees & Ensembles (built):**

| Note | Covers |
|---|---|
| [[Decision Tree]] | Splitting feature space with axis-aligned thresholds |
| [[Ensemble Learning]] | Combining multiple models for better performance |
| [[Bagging]] | Training on bootstrap samples and averaging — variance reduction |
| [[Random Forest]] | Bagged decision trees with feature-subsampling |
| [[Boosting]] | Sequentially correcting prior models' errors — bias reduction |
| [[Gradient Boosting]] | Boosting via gradient descent on the loss |
| [[Gradient Boosting Libraries (XGBoost & LightGBM)]] | The production implementations |
| [[HistGradientBoostingClassifier]] | sklearn's histogram-based gradient boosting |

**Distance/probability-based (⬜ not yet built — high interview frequency, planned next):**

- Support Vector Machines
- Naive Bayes
- K-Nearest Neighbors (as a classifier — [[K Means]] under Unsupervised Learning is a different algorithm despite the similar name)

## Connections

- Model Interpretability (module 16, not yet built) — feature importance is native to tree-based models
- [[Feature Scaling]] — required for SVM/KNN, irrelevant for trees/ensembles
- [[Bias Variance Tradeoff]] — bagging reduces variance, boosting reduces bias

## One-line Summary

> Trees split feature space directly and ensembles combine many of them (bagging for variance, boosting for bias) — the planned SVM/Naive Bayes/KNN additions cover the other classical paradigms (margin-based, probabilistic, instance-based) an interview expects alongside them.
