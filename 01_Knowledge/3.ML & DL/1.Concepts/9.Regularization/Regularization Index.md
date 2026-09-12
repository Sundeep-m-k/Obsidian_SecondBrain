---
tags: [category/ml-dl, index, moc]
---

# Regularization — Index

> **Position in vault**: `3.ML & DL/1.Concepts/9.Regularization/`
> **Purpose**: How to fight overfitting by penalizing model complexity directly in the objective function. This module covers the classical L1/L2 form; Deep-Learning-specific regularization (dropout, batch norm's regularizing side-effect) lives in `18.Deep Learning/5.Training Deep Networks/` and links back here for the underlying idea.
> **Prerequisite**: [[Model Behavior Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Regularization]] | The general idea — penalize complexity in the objective function |
| [[Penalty Term]] | The added term itself |
| [[L1 Regularization]] | Lasso — drives weights to exactly zero, performs feature selection |
| [[L2 Regularization]] | Ridge — shrinks weights smoothly, handles correlated features |
| [[Regularized Linear Regression]] | L1/L2 applied to linear regression specifically |
| [[Regularized Logistic Regression]] | L1/L2 applied to logistic regression specifically |

## One-line Summary

> Regularization is the direct lever on the bias-variance tradeoff from module 8 — L1 for sparsity/feature selection, L2 for stability with correlated features, both by adding a complexity penalty straight into the cost function.
