---
tags: [category/ml-dl, index, moc]
---

# Hyperparameter Tuning & Model Selection — Index

> **Position in vault**: `3.ML & DL/1.Concepts/15.Hyperparameter Tuning & Model Selection/`
> **Purpose**: How to choose good hyperparameters and, more broadly, how to pick a final model among candidates once each is tuned.
> **Prerequisite**: [[Cross Validation Strategy]], [[Model Evaluation Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Hyperparameter Search Methods]] | Grid search, random search, Bayesian optimization; when to use each; nested CV |

## Connections

- [[Evaluation Workflow and Baselines]] — where hyperparameter tuning fits in the overall model-selection workflow (tune on val, evaluate once on test)
- [[Model Interpretability Index]] — a tuned model still needs to be explained, not just optimized

## One-line Summary

> Hyperparameter search picks good configurations by evaluating candidates on validation data only — random search is the default for most problems, grid search for a small exhaustive sweep, Bayesian optimization when trials are expensive.
