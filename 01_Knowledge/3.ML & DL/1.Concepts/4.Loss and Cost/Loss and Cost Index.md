---
tags: [category/ml-dl, index, moc]
---

# Loss and Cost — Index

> **Position in vault**: `3.ML & DL/1.Concepts/4.Loss and Cost/`
> **Purpose**: What "wrong" means, made precise — the function that turns a prediction error into a single number a model can be optimized against.
> **Prerequisite**: [[Linear Regression Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Loss Function]] | Error on a single example |
| [[Cost Function]] | Average loss over the whole training set — what's actually minimized |
| [[Objective Function]] | The general term covering cost functions plus any added penalty terms |
| [[Mean Squared Error]] | The standard regression loss |
| [[Logistic Loss]] | The standard classification loss (cross-entropy) |
| [[Cost Surface]] | The geometric shape being descended during optimization |
| [[Error Minimization]] | Why minimizing cost is the training objective in the first place |

## One-line Summary

> Loss measures one example's error, cost averages it across the dataset, and the objective function is cost plus whatever regularization is added on top — module 5 is entirely about how that objective actually gets minimized.
