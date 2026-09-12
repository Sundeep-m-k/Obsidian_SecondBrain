# Random Forest

## What is it?

**Random Forest** is [[Bagging]] applied to [[Decision Tree]]s, plus one extra trick: at each split, only a random subset of features is considered, not all of them. This further decorrelates the trees beyond what bootstrap resampling alone achieves.

## Why the Extra Feature Randomness

Recall [[Bagging]]'s variance formula $\text{Var}(\hat f_{\text{bag}}) = \rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$, bounded below by $\rho\sigma^2$. If one feature is very strong, plain bagged trees would all pick it as the top split, keeping $\rho$ high. Restricting each split to a random subset of $m$ features (typically $m=\sqrt{p}$ for classification, $m=p/3$ for regression, out of $p$ total) forces trees to diversify their early splits, lowering $\rho$ and thus lowering the variance floor itself.

## Algorithm

1. For $m=1$ to $M$: draw a bootstrap sample, grow a tree where each split considers only a random $\sqrt p$ (or $p/3$) feature subset, grow deep (usually unpruned).
2. Predict by averaging (regression) or majority vote (classification).

## Worked Example: Choosing $m$

With $p=25$ features, classification default $m=\sqrt{25}=5$ features considered per split. If $\rho$ drops from $0.4$ (plain bagging) to $0.25$ (Random Forest's extra randomness) while $\sigma^2$ stays at $0.09$: bagging floor $=0.4(0.09)=0.036$; Random Forest floor $=0.25(0.09)=0.0225$ — a lower achievable variance simply from decorrelating the trees, independent of how many trees $M$ are grown.

## Feature Importance

Mean impurity reduction $\Delta$ (see [[Decision Tree]]) attributable to a feature, averaged across all trees and all splits using it. More stable than a single tree's importance, though still biased toward high-cardinality features (more possible split points → more chances to look "important" by chance).

## Advantages

- Handles non-linear relationships and interactions automatically
- Robust to outliers, no feature scaling needed
- Parallelizable (trees are independent) — fast to train
- Free OOB error estimate and feature importances

## Limitations

- Usually beaten by well-tuned [[Gradient Boosting]] / [[HistGradientBoostingClassifier]] on tabular tasks
- Can still overfit with very deep trees on noisy data, though far less than a single tree
- Averaging many trees still can't extrapolate outside the training data's feature range

## Common Mistakes

- Assuming Random Forest can't overfit — more robust than one tree, but depth and tree count still matter
- Interpreting feature importance as causal — it reflects predictive/split usefulness, not causation
- Using $m=p$ (no feature subsampling) — this degenerates back to plain bagging, losing the decorrelation benefit

## Interview / discussion questions

- What's the difference between Random Forest and plain Bagging of decision trees, in terms of the variance formula?
- Why does restricting the feature subset at each split help, given it makes each individual tree "worse"?
- How would you choose $m$ (features per split), and what happens at the extremes $m=1$ and $m=p$?

## Prerequisites

[[Decision Tree]], [[Bagging]]

## Related concepts

[[Ensemble Learning]], [[Boosting]], [[Gradient Boosting]], [[HistGradientBoostingClassifier]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/probability

## One-line summary

> Random Forest bags decision trees while also randomizing the feature subset ($m=\sqrt p$ typical) considered at each split, lowering the tree-to-tree correlation $\rho$ and therefore the variance floor $\rho\sigma^2$ that plain bagging alone can't get past.
