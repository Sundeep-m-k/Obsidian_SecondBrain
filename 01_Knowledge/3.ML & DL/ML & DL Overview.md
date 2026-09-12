---
tags: [category/ml-dl, index, moc]
---

# ML & DL — Overview

> **Position in vault**: `01_Knowledge/3.ML & DL/`
> **Purpose**: Classical machine learning and deep learning as a discipline — the algorithms, theory, evaluation practice, and (as of the 2026-09 restructure) deep learning and applied-AI/LLM systems, built for AI/ML Engineer, Data Scientist, Data Analyst, Forward Deployed Engineer, and Applied AI Engineer interview preparation.
> **History**: This module previously had no top-level overview note despite being one of the two largest and most active subjects in the vault — a real gap fixed in the 2026-09 restructure alongside consolidating two incompatible module-numbering conventions (modules 1–14 vs. the bare-numbered 15–18) into one.

## Section Map

| Module | Covers | Status |
|---|---|---|
| [[Foundations Index\|1. Foundations]] | Core vocabulary — model, features, labels, generalization | ✅ Built |
| [[Supervised Learning Index\|2. Supervised Learning]] | Classification/regression split, feature space, decision boundary | ✅ Built |
| [[Linear Regression Index\|3. Linear Regression]] | The first concrete model — the hypothesis/cost/optimize template | ✅ Built |
| [[Loss and Cost Index\|4. Loss and Cost]] | Making "wrong" precise | ✅ Built |
| [[Optimization Index\|5. Optimization]] | Gradient descent and its mechanics | ✅ Built |
| [[Feature Engineering and Data Preparation Index\|6. Feature Engineering & Data Preparation]] | Representation basics through the full practical preprocessing discipline | ✅ Built (merged from two earlier modules) |
| [[Logistic Regression Index\|7. Logistic Regression]] | The default classification baseline | ✅ Built |
| [[Model Behavior Index\|8. Model Behavior]] | Bias-variance tradeoff, over/underfitting | ✅ Built |
| [[Regularization Index\|9. Regularization]] | L1/L2 — the direct lever on model complexity | ✅ Built |
| [[Unsupervised Learning Index\|10. Unsupervised Learning]] | Clustering, dimensionality reduction, anomaly detection | ✅ Built |
| [[Recommender Systems Index\|11. Recommender Systems]] | Collaborative and content-based filtering | ✅ Built |
| [[Reinforcement Learning Index\|12. Reinforcement Learning]] | Agent/environment/reward/policy | ✅ Built (thin — low priority for target roles) |
| [[Classical ML Algorithms Index\|13. Classical ML Algorithms]] | Trees, ensembles, SVM, Naive Bayes, KNN | ✅ Built |
| [[Model Evaluation Index\|14. Model Evaluation]] | Metrics, ROC/PR, calibration, CV, train/val/test, class imbalance | ✅ Built |
| [[Hyperparameter Tuning and Model Selection Index\|15. Hyperparameter Tuning & Model Selection]] | Grid/Random/Bayesian search, nested CV | ✅ Built |
| [[Model Interpretability Index\|16. Model Interpretability]] | Feature importance, SHAP, LIME | ✅ Built |
| [[Production ML and MLOps Index\|17. Production ML & MLOps]] | Drift, monitoring, versioning, deployment, retraining | ✅ Built |
| [[Deep Learning Index\|18. Deep Learning]] | Neural net foundations, sequence models, CNNs, attention/Transformers, training deep networks, applied DL | ✅ Built |
| 19. Applied AI & LLM Systems | LLM fundamentals, RAG, agents, production AI | ⬜ Not started |
| 20. Data Science, Analytics & Product Thinking | A/B testing, time series, case thinking | ⬜ Not started |
| [[ML Cheatsheet\|21. Meta & Interview Revision]] | Cross-module cheatsheet, common mistakes, role-based interview maps | 🟡 Needs re-scoping to cover modules 13–20 |

## Why This Structure

Modules 1–9 are the classical foundations, built as one continuous template (define a hypothesis function → define a cost → minimize it) that every later model reuses. Modules 10–13 are the other classical model families (unsupervised, recommenders, RL, trees/ensembles/margin/probabilistic/instance-based methods). Modules 14–17 are the practitioner discipline layer — how to evaluate, tune, interpret, and operate a model once it exists, not just fit one. Module 18 is Deep Learning as its own first-class citizen (previously entirely absent despite the folder's name). Module 19 extends outward into applied AI/LLM systems, the area this vault's target roles — especially Forward Deployed Engineer and Applied AI Engineer — were previously weakest in. Module 20 covers the statistics-adjacent and product-thinking material Data Analyst/Data Scientist interviews specifically probe, cross-linking to `6.Statistics & Experimental Design` rather than duplicating it. Module 21 is navigation and revision, not new content.

## Key Cross-Links to Other Subjects

| ML & DL Concept | Links to |
|---|---|
| Tokenization, embeddings (static & contextual) | [[NLP Index]] — canonical home, linked not duplicated |
| Sequence model architecture (RNN/LSTM/GRU) | Relocated *from* NLP into `18. Deep Learning` — see [[Sequence Models Index]] for the full rationale |
| Hypothesis testing, confidence intervals, multiple comparisons | `6.Statistics & Experimental Design` — canonical theory home; `14. Model Evaluation`'s [[Statistical Significance Testing for Model Comparison]] applies it to model comparison specifically |
| Linear algebra, calculus underlying gradient descent | [[Mathematics Overview]] |

## Common Exam / Interview Questions

1. Walk through the bias-variance tradeoff and name one technique that primarily fixes each side of it.
2. When would you reach for L1 vs. L2 regularization?
3. Bagging vs. boosting — which reduces variance and which reduces bias, and why?
4. Precision vs. recall — when does optimizing one over the other actually matter?
5. Why does an unscaled feature set slow gradient descent but not affect a decision tree?
6. What's the seq2seq bottleneck problem, and what architecture was invented specifically to solve it?

## One-line Summary

> This subject runs from classical ML foundations (modules 1–13) through the practitioner discipline of evaluating, tuning, interpreting, and operating models (14–17) into Deep Learning and Applied AI/LLM systems (18–19) and the statistics/product-thinking layer Data Analyst and Data Scientist interviews specifically probe (20) — module 21 is the cross-cutting revision layer tying all of it together for interview prep.
