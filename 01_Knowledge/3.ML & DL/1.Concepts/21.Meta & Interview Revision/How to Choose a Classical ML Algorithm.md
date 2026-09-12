# How to Choose a Classical ML Algorithm

## The Decision

With [[Decision Tree]]/[[Random Forest]]/[[Gradient Boosting]], [[Support Vector Machines]], [[Naive Bayes]], and [[K-Nearest Neighbors]] all being viable classical classifiers, the practical choice usually comes down to a small number of concrete questions — not "which is most accurate in the abstract," since that depends entirely on the specific dataset.

| Question | Answer favors |
|---|---|
| Is interpretability/auditability required? | [[Decision Tree]] (single tree, not an ensemble) |
| Tabular data, need strong out-of-the-box accuracy, don't need interpretability? | [[Gradient Boosting]] (XGBoost/LightGBM) — usually the strongest default for structured/tabular data |
| High-dimensional sparse data (text, bag-of-words)? | [[Naive Bayes]] (fast, handles sparsity well) or [[Support Vector Machines]] with a linear kernel |
| Small dataset, non-linear boundary, some tolerance for tuning kernel/C? | [[Support Vector Machines]] with RBF kernel |
| Need a quick baseline, small-to-medium data, no training time budget? | [[K-Nearest Neighbors]] (no training phase at all) |
| Features have very different scales and you can't/won't scale them? | [[Decision Tree]] / [[Random Forest]] / [[Gradient Boosting]] — the only options here that don't need [[Feature Scaling]] |
| Correlated/redundant features expected? | Tree-based methods handle this natively; [[K-Nearest Neighbors]] and [[Support Vector Machines]] don't correct for it |
| Very large dataset (millions of rows)? | [[Gradient Boosting]] or [[Random Forest]] — [[Support Vector Machines]] and [[K-Nearest Neighbors]] both scale poorly here |
| Need calibrated probabilities out of the box? | None of them, fully — [[Naive Bayes]] and [[Support Vector Machines]] are both known to be poorly calibrated; recalibrate with [[Calibration and Probability Evaluation]] regardless of choice |

## The One Rule That Matters Most

**Start with a simple baseline** ([[Logistic Regression]] or a single [[Decision Tree]]) before reaching for anything more complex — see [[Evaluation Workflow and Baselines]]. Every algorithm above should be justified by beating that baseline by a meaningful margin, not chosen first and evaluated in isolation.

## Interview Framing

When asked "which model would you use for X," the strongest answer names the actual constraints that decide it (data size, dimensionality, interpretability need, scaling feasibility) rather than a single named algorithm — see [[Framing Ambiguous Business Problems]] for the same "clarify constraints before naming a technique" discipline applied to full case questions, not just algorithm choice.

## Connections

- [[Classical ML Algorithms Index]] — full detail on each algorithm this table compares
- [[When to Use Linear Regression]], [[When to Use Logistic Regression]] — the same decision-guide pattern, one level simpler
- [[Bias Variance Tradeoff]] — the underlying tradeoff every row in this table is ultimately about

## One-line Summary

> Choose among classical algorithms by constraint (data size, dimensionality, interpretability, scaling feasibility), not by assumed accuracy — and always benchmark against a simple baseline before trusting the added complexity was worth it.
