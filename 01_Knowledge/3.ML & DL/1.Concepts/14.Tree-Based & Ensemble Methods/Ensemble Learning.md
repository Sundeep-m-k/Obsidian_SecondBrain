# Ensemble Learning

## What is it?

**Ensemble learning** combines predictions from multiple base learners into one stronger prediction:

$$\hat{f}_{\text{ensemble}}(x) = \sum_{m=1}^{M} w_m \cdot \hat{f}_m(x)$$

The bet: if base learners' errors are somewhat independent, combining them cancels noise and reduces overall error below any single model's.

## The Two Main Families

| Family | Strategy | Effect | Example |
|---|---|---|---|
| **Bagging** | Train $M$ models in parallel on bootstrapped data, average | Reduces **variance** (see [[Bagging]]'s $\rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$) | [[Random Forest]] |
| **Boosting** | Train sequentially, each correcting prior errors | Reduces **bias**, often variance too | [[Boosting]], [[Gradient Boosting]] |

## Why Independence Matters: the Variance Formula

For $M$ base learners with individual variance $\sigma^2$ and average pairwise correlation $\rho$, simple averaging gives:

$$\text{Var}\left(\frac1M\sum_m \hat f_m\right) = \rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$$

At $\rho=0$ (fully independent), variance shrinks to $\sigma^2/M \to 0$ as $M\to\infty$. At $\rho=1$ (fully correlated — every base learner makes the exact same errors), variance stays at $\sigma^2$ regardless of $M$ — ensembling a bunch of identical models buys nothing. Real ensembles sit somewhere in between, which is exactly why techniques that actively reduce $\rho$ (Random Forest's feature subsampling, boosting's sequential error-targeting) matter more than simply adding more models.

## Common Mistakes

- Assuming any combination of models helps — if base learners are highly correlated ($\rho$ near 1), ensembling barely helps, per the formula above
- Confusing bagging's variance-reduction goal with boosting's bias-reduction goal — different mechanisms, different failure modes
- Over-trusting ensemble confidence without checking [[Calibration]]

## Interview / discussion questions

- Derive the ensemble variance formula and use it to explain why correlated base learners limit the benefit of ensembling.
- When would you prefer bagging over boosting, and vice versa?
- What's the difference between an ensemble of diverse model *types* vs. an ensemble of the same model type trained differently?

## Prerequisites

[[Bias Variance Tradeoff]], [[Decision Tree]]

## Related concepts

[[Bagging]], [[Random Forest]], [[Boosting]], [[Gradient Boosting]], [[HistGradientBoostingClassifier]], [[Baseline Estimator]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/probability

## One-line summary

> Ensembling's benefit is governed by $\text{Var} = \rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$ — bagging attacks this by decorrelating parallel models (lowering $\rho$), boosting attacks bias directly by sequentially targeting errors; adding more models alone only helps as much as $\rho$ allows.
