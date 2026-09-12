# Bagging (Bootstrap Aggregating)

## What is it?

**Bagging** trains many copies of the same model on different **bootstrapped** samples (drawn with replacement) and averages (regression) or votes (classification).

## Algorithm

1. Draw $M$ bootstrap samples of size $n$ from the training set (each: $n$ draws with replacement, ~63.2% unique examples per sample).
2. Train one model per sample.
3. Combine: $\hat f_{\text{bag}}(x) = \frac{1}{M}\sum_{m=1}^{M} \hat f_m(x)$.

## Why 63.2%?

The probability a given example is *not* selected in one draw is $1-\frac1n$; over $n$ draws, $P(\text{never selected}) = (1-\frac1n)^n \to e^{-1} \approx 0.368$ as $n$ grows. So each bootstrap sample contains roughly $1-e^{-1} \approx 63.2\%$ unique examples, leaving the remaining ~36.8% as **out-of-bag (OOB)** — a free validation set for that particular model.

## Why It Reduces Variance

For $M$ base models with individual variance $\sigma^2$ and pairwise correlation $\rho$, the variance of the average is:

$$\text{Var}(\hat f_{\text{bag}}) = \rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$$

As $M \to \infty$, this converges to $\rho\sigma^2$, not zero — bagging can only drive variance down to the *correlated* component; the more independent the base models (lower $\rho$), the more averaging helps. This formula is the direct mathematical reason [[Random Forest]] adds extra feature-subsampling randomness on top of plain bagging: it further lowers $\rho$.

## Worked Example

Suppose a single deep tree has $\sigma^2 = 0.09$ (variance of its prediction across resamples) and bagged trees have pairwise correlation $\rho=0.4$. With $M=100$ trees:

$$\text{Var}(\hat f_{\text{bag}}) = 0.4(0.09) + \frac{0.6}{100}(0.09) = 0.036 + 0.00054 \approx 0.0365$$

Variance drops from $0.09$ to $\approx 0.0365$ — a 59% reduction — but is bounded below by $\rho\sigma^2 = 0.036$ no matter how large $M$ gets. Lowering $\rho$ (e.g. via Random Forest's feature subsampling) is the only way to push past that floor.

## Why It Matters

Bagging is the mechanism behind [[Random Forest]]. It's the standard fix for a model that's accurate on average but unstable — small data changes swing predictions a lot.

## Common Mistakes

- Bagging a low-variance, high-bias model (linear regression) — little payoff since $\sigma^2$ is already small; the benefit is largest for high-variance base learners like deep trees
- Confusing bagging (parallel, variance-focused) with boosting (sequential, bias-focused) — see [[Boosting]]
- Forgetting the variance floor $\rho\sigma^2$ — assuming more trees always helps, when correlation is the real bottleneck past a certain $M$

## Interview / discussion questions

- Derive $\text{Var}(\hat f_{\text{bag}})$ for $M$ correlated base models and explain the role of $\rho$.
- Why does each bootstrap sample contain roughly 63.2% unique examples? Derive this using the limit $(1-1/n)^n \to e^{-1}$.
- Why does bagging help unstable, high-variance models but not stable, high-bias ones?

## Prerequisites

[[Decision Tree]], [[Bias Variance Tradeoff]], [[Variance]]

## Related concepts

[[Ensemble Learning]], [[Random Forest]], [[Boosting]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/probability

## One-line summary

> Bagging averages $M$ models trained on bootstrap resamples (each ~63.2% unique data via $(1-1/n)^n \to e^{-1}$), reducing variance to $\rho\sigma^2 + \frac{1-\rho}{M}\sigma^2$ — bounded below by the correlation floor $\rho\sigma^2$, which is exactly why decorrelating the base models matters as much as adding more of them.
