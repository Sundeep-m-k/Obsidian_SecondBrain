---
tags: [category/ml-dl, index, moc]
---

# Model Interpretability — Index

> **Position in vault**: `3.ML & DL/1.Concepts/16.Model Interpretability/`
> **Purpose**: Answering "why did the model predict this" — both in aggregate (which features matter overall) and for a single prediction (why this specific output). Increasingly asked in Data Scientist and Applied AI Engineer interviews as "explain this model to a stakeholder" style questions.
> **Prerequisite**: [[Model Behavior Index]], [[Classical ML Algorithms Index]]

## Section Map

| Note | Scope |
|---|---|
| [[Feature Importance]] | Global — which features matter overall (impurity-based, permutation, coefficient magnitude) |
| [[SHAP]] | Local — game-theoretically grounded, per-prediction attribution with formal guarantees |
| [[LIME]] | Local — faster, model-agnostic per-prediction attribution without formal guarantees |

## Decision Guide

- Need a quick, cheap sense of what the model relies on overall → [[Feature Importance]] (permutation variant, to avoid impurity bias)
- Need to explain *one specific* prediction, and the explanation must be defensible/consistent → [[SHAP]]
- Need to explain *one specific* prediction, fast, model-agnostic, stakes are lower → [[LIME]]

## Interview Questions

1. What's the difference between global and local interpretability, and when does each matter?
2. Why is impurity-based feature importance from a random forest sometimes misleading?
3. What formal guarantee does SHAP provide that LIME doesn't?
4. A stakeholder says "the model is a black box, I don't trust it" — what's your response and what would you show them?

## Connections

- [[Random Forest]], [[Gradient Boosting]] — where TreeSHAP and impurity-based importance are cheap/free
- [[Logistic Regression]] — where scaled coefficients are already a form of built-in interpretability
- [[Case Thinking & Product Sense]] (module 20) — communicating a model's behavior to a non-technical stakeholder is as much a communication skill as a technical one

## One-line Summary

> Start with permutation-based feature importance for a global view, then reach for SHAP (defensible, consistent) or LIME (fast, model-agnostic) when a specific prediction needs explaining.
