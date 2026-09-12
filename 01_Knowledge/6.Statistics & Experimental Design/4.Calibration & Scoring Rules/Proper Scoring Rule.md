# Proper Scoring Rule

## What is it?

A **proper scoring rule** $S(\hat p, y)$ is an evaluation metric for probabilistic predictions constructed so that the forecaster's expected score is optimized by reporting their *true* believed probability $p^*$:

$$\mathbb{E}_{y \sim p^*}[S(\hat p, y)] \text{ is optimized at } \hat p = p^*$$

You cannot get a better expected score by hedging, exaggerating, or otherwise misreporting your actual belief. "Strictly proper" means $p^*$ is the *unique* optimum (not tied with any other report).

---

## Why This Property Matters

Without it, a metric can be gamed: a forecaster might learn that reporting 50% for every uncertain case, or rounding toward 0/100%, scores better than honesty — defeating the point of asking for a probability at all. A proper scoring rule guarantees the mathematically optimal strategy is also the honest one.

## Proof Sketch for the Brier Score

For outcome $Y \sim \text{Bernoulli}(p^*)$ and a report $\hat p$, the expected [[Brier Score]] is:

$$\mathbb{E}[(\hat p - Y)^2] = (\hat p - p^*)^2 + p^*(1-p^*)$$

(This follows from the bias-variance-style identity $\mathbb{E}[(\hat p - Y)^2] = \mathbb{E}[(\hat p - p^*)^2] + \text{Var}(Y)$, since $\hat p$ is a constant given the forecaster's choice.) The first term is minimized — driven to exactly $0$ — only at $\hat p = p^*$; the second term is irreducible. This is a direct proof that reporting your true belief is the *unique* optimal strategy under the Brier score.

---

## Examples

| Scoring rule | Formula ($y\in\{0,1\}$, prediction $\hat p$) | Proper? |
|---|---|---|
| [[Brier Score]] | $(\hat p - y)^2$ | Yes (strictly) |
| Log loss / cross-entropy (see [[Loss Function]]) | $-[y\log\hat p + (1-y)\log(1-\hat p)]$ | Yes (strictly) |
| Accuracy at a fixed threshold | Correct/incorrect count | **No** — rewards pushing $\hat p$ away from $p^*$ toward whichever side of the threshold is "safer" |

## Why It Matters

Any time a system reports a probability that a person or process acts on directly (not just for ranking), the metric used to build and validate it should be proper — otherwise the model may have been implicitly tuned to game a metric that doesn't reward honest probability estimates.

## Common Mistakes

- Using classification accuracy as the primary metric when the system's whole point is producing a usable probability
- Assuming any "reasonable-looking" metric is automatically proper — properness is a specific, provable mathematical property, not a vibe

## Interview / discussion questions

- Prove that the Brier score is (strictly) proper using the bias-variance-style decomposition above.
- Why is plain classification accuracy not a proper scoring rule?
- Give a concrete scenario where a forecaster could game a non-proper scoring rule and quantify the benefit of doing so.

## Prerequisites

[[Calibration]], [[Loss Function]]

## Related concepts

[[Brier Score]]

## Tags

#category/statistics #topic/calibration-scoring #math/probability

## One-line summary

> A proper scoring rule is built so $\mathbb{E}[S(\hat p, Y)]$ is uniquely optimized at $\hat p = p^*$ — provably true for the Brier score via a bias-variance decomposition — which is what makes it safe to use as an honest evaluation metric.
