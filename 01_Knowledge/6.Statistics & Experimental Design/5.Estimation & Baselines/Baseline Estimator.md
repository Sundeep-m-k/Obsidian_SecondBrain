# Baseline Estimator (Analog / Backoff Estimation)

## What is it?

A **baseline estimator** answers a prediction question using the simplest defensible method available. An **analog** estimator groups new cases by shared characteristics and predicts using the historical outcome rate within that group:

$$\hat p(x) = \frac{1}{|\mathcal{G}(x)|}\sum_{i \in \mathcal{G}(x)} y_i$$

where $\mathcal{G}(x)$ is the set of historical examples sharing $x$'s group (e.g. same therapeutic area × NCT-count bucket).

---

## The Backoff Ladder, Formally

Define a sequence of increasingly coarse groupings $\mathcal{G}_1(x) \supset \mathcal{G}_2(x) \supset \cdots$ is wrong direction — rather, groupings ordered from most specific to least specific, $\mathcal{G}_1(x) \subseteq \mathcal{G}_2(x) \subseteq \cdots$ (each subsequent grouping matches *more* historical examples by relaxing the match criteria), and a minimum sample threshold $m_{\min}$:

$$k^*(x) = \min\{k : |\mathcal{G}_k(x)| \geq m_{\min}\}, \qquad \hat p(x) = \frac{1}{|\mathcal{G}_{k^*(x)}(x)|}\sum_{i \in \mathcal{G}_{k^*(x)}(x)} y_i$$

The estimator uses the *most specific* grouping that still clears the sample-size floor — e.g. try TA × NCT-bin first; if $|\mathcal{G}_1(x)| < m_{\min}=40$, back off to TA alone; if still too few, back off further. This prevents ever reporting a rate estimated from a handful of noisy historical cases.

## Worked Example

Target: a Psychiatry program with 3 prior NCTs. $\mathcal{G}_1$ = "Psychiatry × 3 NCTs" has 12 historical matches (below $m_{\min}=40$) → back off. $\mathcal{G}_2$ = "Psychiatry × 2-4 NCTs" has 58 matches (clears $m_{\min}$) → use this level. $\hat p(x) = \frac{1}{58}\sum y_i$, say $=0.34$. Note the variance of this estimate is roughly $\frac{\hat p(1-\hat p)}{58} \approx 0.0039$ (std. error $\approx 0.062$) — still fairly wide, which is exactly why $m_{\min}$ exists: at $|\mathcal{G}_1|=12$, the standard error would have been $\approx 0.13$, far too wide to trust.

---

## Why "ML Only Ships If It Honestly Beats the Baseline" Is the Right Default

Complex models have more ways to overfit, more hyperparameters that can be accidentally tuned on the test set, and more opacity about *why* a prediction came out as it did. A simple, transparent baseline sets the bar that justifies the added complexity and maintenance cost of a fancier model — complexity has to earn its place via [[Grouped Train-Test Split]] and [[Rolling-Origin Validation]], not be assumed superior.

## Common Mistakes

- Comparing an ML model against a baseline with no backoff ladder or sample threshold, so it looks artificially weak on sparse groups
- Assuming a fancier model wins without measuring it honestly using the *same* evaluation protocol as the baseline
- Treating "the baseline won globally" as a failure of the ML effort, rather than the evaluation protocol correctly preventing an unearned complexity upgrade

## Interview / discussion questions

- Derive the standard error of $\hat p(x)$ as a function of $|\mathcal{G}_{k^*}(x)|$ and explain why $m_{\min}$ matters.
- Why should a complex model be required to beat a simple baseline before shipping?
- If a fancy model wins on a single random split but the baseline wins under rolling-origin validation, which result should you trust?

## Prerequisites

[[Ensemble Learning]], basic probability

## Related concepts

[[Gradient Boosting]], [[HistGradientBoostingClassifier]], [[Grouped Train-Test Split]], [[Rolling-Origin Validation]], [[Brier Score]]

## Tags

#category/statistics #topic/estimation #math/probability

## One-line summary

> A backoff analog estimator picks the most specific historical grouping that still clears a minimum sample size $m_{\min}$, trading specificity for a bounded standard error — and any fancier model has to beat this honestly, under the same evaluation protocol, before it earns the right to ship.
