# Brier Score

## What is it?

The **Brier score** is the mean squared error between predicted probability and actual (0/1) outcome:

$$\text{Brier} = \frac{1}{n}\sum_{i=1}^{n} (\hat{p}_i - y_i)^2$$

Lower is better. $\text{Brier}=0$ is perfect; always predicting $0.5$ gives $\text{Brier}=0.25$; a confidently *wrong* model can score worse than $0.25$.

---

## The Murphy Decomposition

$$\text{Brier} = \underbrace{\frac{1}{n}\sum_m |B_m|(\text{conf}(B_m)-\text{acc}(B_m))^2}_{\text{Reliability (calibration error)}} \;-\; \underbrace{\frac{1}{n}\sum_m |B_m|(\text{acc}(B_m)-\bar y)^2}_{\text{Resolution (discrimination)}} \;+\; \underbrace{\bar y (1-\bar y)}_{\text{Uncertainty (irreducible)}}$$

where bins $B_m$ are as in [[Calibration]], and $\bar y$ is the overall base rate. Reliability should be small (well-calibrated), resolution should be large (the model actually discriminates between cases), and uncertainty is a property of the outcome itself, not the model — a base rate near 50% makes even a *perfect* model's Brier score higher than one for a highly predictable ($\bar y$ near 0 or 1) outcome.

## Worked Example

Base rate $\bar y = 0.30$ (so uncertainty term $= 0.30 \times 0.70 = 0.21$). Suppose reliability (from the ECE-style table) $= 0.006$ and resolution $=0.045$:

$$\text{Brier} = 0.006 - 0.045 + 0.21 = 0.171$$

Compare this to the naive "always predict the base rate" model, whose Brier score is exactly the uncertainty term, $0.21$ — the model beats that trivial baseline by $0.21-0.171=0.039$, entirely attributable to its resolution outweighing its (small) reliability penalty.

---

## Why It's a Proper Scoring Rule

See [[Proper Scoring Rule]] — the Brier score is constructed so a forecaster's best strategy is to report their *true* believed probability. You cannot improve your expected score by hedging toward 50% or exaggerating toward 0/100%.

## Why It Matters

Comparing a candidate feature or model against a baseline should use the Brier score (or another proper scoring rule), because it penalizes a feature that improves ranking slightly while making probabilities themselves worse — a tradeoff AUC alone would miss entirely, since AUC can't see reliability at all. A feature that *increases* Brier score should not ship even if AUC ticks up.

## Common Mistakes

- Using accuracy or AUC as the sole metric, missing that predictions are badly calibrated
- Comparing raw Brier scores across datasets with different base rates without accounting for the uncertainty term — a noisier outcome has a higher *achievable* Brier score even for a perfect model
- Not decomposing a bad Brier score — reliability and resolution failures need different fixes (recalibrate vs. improve the model)

## Interview / discussion questions

- Derive the Murphy decomposition and explain what each term means practically.
- Why is $0.25$ a meaningful reference point for a binary Brier score?
- If AUC improves but Brier score worsens after adding a feature, which should decide whether to ship it, and why?

## Prerequisites

[[Calibration]], [[Mean Squared Error]]

## Related concepts

[[Proper Scoring Rule]], [[Baseline Estimator]]

## Tags

#category/statistics #topic/calibration-scoring #math/probability

## One-line summary

> The Brier score decomposes into reliability minus resolution plus irreducible uncertainty — a proper scoring rule that captures both calibration and ranking in one number, catching tradeoffs that AUC (which only sees rank order) is mathematically blind to.
