# Decision Tree

## What is it?

A **Decision Tree** is a supervised model that predicts an output via a sequence of yes/no or threshold questions about the input features, arranged as a tree. Each internal node is a feature test, each branch an outcome, each leaf a prediction.

## How It's Built

1. Start with all training examples at the root.
2. Pick the feature + threshold that best **splits** the data into purer subsets.
3. Recurse on each child.
4. Stop at a purity floor, depth limit, or minimum leaf size.

## Measuring Split Quality

$$\text{Gini}(S) = 1 - \sum_{k=1}^{K} p_k^2 \qquad \text{Entropy}(S) = -\sum_{k=1}^{K} p_k \log_2 p_k$$

where $p_k$ is the fraction of class $k$ in node $S$. A split on feature $f$ at threshold $\tau$ producing children $S_L, S_R$ is scored by its **impurity reduction**:

$$\Delta = I(S) - \left(\frac{|S_L|}{|S|}I(S_L) + \frac{|S_R|}{|S|}I(S_R)\right)$$

The tree greedily picks the split maximizing $\Delta$ at each node.

## Worked Example: Gini at a Node

Node $S$ has 100 examples: 70 successes, 30 failures → $p_1=0.7, p_2=0.3$, $\text{Gini}(S) = 1-(0.7^2+0.3^2) = 1-0.58=0.42$. Splitting on "sponsor=industry?" gives $S_L$ (industry, 80 examples: 62 success/18 fail, $\text{Gini}=1-(0.775^2+0.225^2)=0.349$) and $S_R$ (non-industry, 20 examples: 8 success/12 fail, $\text{Gini}=1-(0.4^2+0.6^2)=0.48$):

$$\Delta = 0.42 - \left(\frac{80}{100}(0.349) + \frac{20}{100}(0.48)\right) = 0.42 - (0.279+0.096) = 0.045$$

This $\Delta=0.045$ is compared against every other candidate split (other features, other thresholds); the tree picks whichever maximizes $\Delta$.

## Why It Matters

Decision trees are the building block behind [[Random Forest]] (bagged trees), [[Gradient Boosting]] and [[HistGradientBoostingClassifier]] (boosted trees) — understanding a single tree's split mechanics is a prerequisite for all of them.

## Advantages

- No feature scaling needed (threshold-based, not distance-based)
- Captures non-linear relationships and feature interactions automatically
- Interpretable — the decision path is directly readable
- Handles mixed numeric/categorical features

## Limitations

- A deep tree **overfits** easily — can memorize training data to single examples ([[Overfitting]], [[Model Complexity]])
- High **variance**: small training-data changes can produce a very different tree ([[Bias Variance Tradeoff]], [[Variance]])
- Greedy splitting isn't globally optimal
- Predictions are step-like — poor at extrapolating smooth numeric trends

## Common Mistakes

- Growing trees to full depth with no limit or pruning, then being surprised by overfitting
- Trusting feature importance from a single tree — unstable; ensembles ([[Random Forest]], [[Gradient Boosting]]) give more reliable importance
- Forgetting that a leaky feature (see [[Data Leakage]]) will be exploited greedily and immediately, since the tree only cares about impurity reduction, not whether the split "makes sense" causally

## Interview / discussion questions

- Compute Gini impurity and impurity reduction for a small worked split by hand.
- Why does a single decision tree tend to overfit, and what are the standard fixes?
- Why don't decision trees need feature scaling, unlike linear/logistic regression?
- How does Gini impurity differ from entropy in practice, and do they usually pick different splits?

## Prerequisites

[[Classification]], [[Regression]], [[Overfitting]]

## Related concepts

[[Ensemble Learning]], [[Bagging]], [[Random Forest]], [[Boosting]], [[Gradient Boosting]], [[HistGradientBoostingClassifier]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/information-theory

## One-line summary

> A decision tree predicts by recursively picking the split that maximizes impurity reduction $\Delta = I(S) - \sum \frac{|S_c|}{|S|}I(S_c)$ — simple and interpretable alone, but high-variance, which is exactly what ensembles like Random Forest and Gradient Boosting are built to fix.
