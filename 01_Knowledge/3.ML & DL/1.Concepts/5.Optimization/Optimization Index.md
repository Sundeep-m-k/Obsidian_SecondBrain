---
tags: [category/ml-dl, index, moc]
---

# Optimization — Index

> **Position in vault**: `3.ML & DL/1.Concepts/5.Optimization/`
> **Purpose**: How a cost function actually gets minimized — gradient descent and its mechanics. This is the classical, single-model foundation; the deep-learning-specific optimizers (Adam, RMSProp, learning-rate schedules) live in `18.Deep Learning/5.Training Deep Networks/` and build directly on the concepts here.
> **Prerequisite**: [[Loss and Cost Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Gradient]] | The direction of steepest increase of a function |
| [[Gradient Descent]] | Iteratively stepping against the gradient to minimize cost |
| [[Batch Gradient Descent]] | Using the full training set for every update step |
| [[Gradient Descent for Linear Regression]] | The concrete update rule for the linear regression cost |
| [[Learning Rate]] | The step-size hyperparameter, and why it's the first thing to tune |
| [[Convergence]] | Knowing when to stop |
| [[Local Minimum]] | Where gradient descent can get stuck for non-convex objectives |
| [[Global Minimum]] | The actual target, and when it's guaranteed reachable |

## One-line Summary

> Gradient descent is the one optimization algorithm every model in this vault — from linear regression to deep networks — is trained with; everything downstream is a variant on "follow the negative gradient with some step size."
