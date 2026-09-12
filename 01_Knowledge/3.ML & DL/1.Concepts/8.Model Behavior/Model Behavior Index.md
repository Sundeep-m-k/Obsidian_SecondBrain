---
tags: [category/ml-dl, index, moc]
---

# Model Behavior — Index

> **Position in vault**: `3.ML & DL/1.Concepts/8.Model Behavior/`
> **Purpose**: Why a model that fits the training data perfectly can still be a bad model — the bias-variance tradeoff, and the vocabulary (over/underfitting, training/test error) used to diagnose it in every later module.
> **Prerequisite**: [[Loss and Cost Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Training Error]] | Error on the data the model was fit to |
| [[Test Error]] | Error on unseen data — the number that actually matters |
| [[Overfitting]] | Low training error, high test error |
| [[Underfitting]] | High error on both |
| [[Model Complexity]] | The knob that trades off over- and under-fitting |
| [[Bias]] | Systematic error from a model too simple to capture the pattern |
| [[Variance]] | Error from sensitivity to the specific training sample |
| [[Bias Variance Tradeoff]] | Why bias and variance move in opposite directions as complexity changes |

## Connections

- [[Regularization Index]] — the primary tool for controlling this tradeoff
- [[Cross Validation Strategy]] — how test error is actually estimated in practice
- [[How to Diagnose Overfitting]] (Meta & Interview Revision) — the decision-guide version of this module

## One-line Summary

> Every model selection decision in this vault reduces to managing the bias-variance tradeoff — this module is the diagnostic vocabulary, module 9 is the primary lever for fixing it.
