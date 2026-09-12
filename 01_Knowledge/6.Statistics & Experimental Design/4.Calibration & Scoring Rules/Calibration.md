# Calibration

## What is it?

A model's predicted probabilities are **calibrated** if, among all cases where it predicts $\hat p$, the true outcome rate really is $\hat p$:

$$P(Y=1 \mid \hat p(X) = p) = p \quad \text{for all } p \in [0,1]$$

Calibration is distinct from ranking ability: a model can rank cases correctly (higher scores for cases more likely to succeed) while being badly calibrated (e.g. everything it calls "70%" actually succeeds 40% of the time).

---

## Measuring It: Expected Calibration Error

Bucket predictions into $M$ bins $B_1, \ldots, B_M$ by predicted probability (e.g. deciles). Within each bin, compare the average predicted probability to the observed event rate:

$$\text{ECE} = \sum_{m=1}^{M} \frac{|B_m|}{n} \left| \text{acc}(B_m) - \text{conf}(B_m) \right|$$

where $\text{conf}(B_m)$ = mean predicted probability in bin $m$, $\text{acc}(B_m)$ = observed event rate in bin $m$. A perfectly calibrated model has $\text{acc}(B_m) = \text{conf}(B_m)$ for every bin, so $\text{ECE}=0$.

## Worked Example

| Bin (predicted range) | Mean predicted $\hat p$ | $n$ in bin | Observed rate | $|\text{acc}-\text{conf}|$ |
|---|---|---|---|---|
| 0.0–0.2 | 0.11 | 500 | 0.09 | 0.02 |
| 0.2–0.4 | 0.31 | 400 | 0.38 | 0.07 |
| 0.4–0.6 | 0.49 | 300 | 0.44 | 0.05 |
| 0.6–0.8 | 0.71 | 200 | 0.63 | 0.08 |
| 0.8–1.0 | 0.89 | 100 | 0.90 | 0.01 |

$\text{ECE} = \frac{500(0.02)+400(0.07)+300(0.05)+200(0.08)+100(0.01)}{1500} \approx 0.047$ — on average, predicted probabilities are off by about 4.7 percentage points from the true rate within each bucket. The 0.2–0.4 and 0.6–0.8 bins are the worst offenders here and would be the first place to investigate (often systematic — e.g. overconfidence in a particular segment).

---

## Why It's a Separate Concern From Accuracy or AUC

Accuracy and ROC-AUC only care whether predictions are correctly *ordered* relative to a threshold — they're invariant to any monotonic transformation of the score. If you want to say "this program has a 62% chance of advancing" as a usable, quotable number — not just "more likely than that one" — you need calibration specifically.

## How to Fix Miscalibration

- **Platt scaling:** fit a logistic regression $P(Y=1) = \sigma(a \cdot z + b)$ on top of the model's raw score $z$
- **Isotonic regression:** fit a non-decreasing step function mapping raw scores to calibrated probabilities, more flexible but needs more data

Both are fit on a **held-out** calibration set, never on the same data used to fit the underlying model or to report final metrics.

## Why It Matters

Any system reporting a probability as a decision input implicitly promises calibration. Fixing it is separate from improving ranking ability ([[Brier Score]] captures both in one number; ECE isolates calibration alone).

## Common Mistakes

- Reporting raw model output as "the probability" without checking a calibration curve or computing ECE
- Assuming high AUC implies good calibration — AUC is invariant to monotonic rescaling, so it can't detect calibration error at all
- Fitting a feature that improves AUC while silently worsening calibration, and only checking AUC before shipping

## Interview / discussion questions

- Derive why ROC-AUC cannot detect miscalibration.
- Walk through computing ECE on a small worked example.
- What's the difference between Platt scaling and isotonic regression for recalibration, and when would you pick one over the other?

## Prerequisites

[[Classification]], [[Probability Output]]

## Related concepts

[[Brier Score]], [[Proper Scoring Rule]]

## Tags

#category/statistics #topic/calibration-scoring #math/probability

## One-line summary

> Calibration means $P(Y=1\mid \hat p = p) = p$ — measured via ECE by bucketing predictions and comparing observed rate to predicted rate per bucket — and it's a property AUC is mathematically blind to, since AUC only sees rank order.
