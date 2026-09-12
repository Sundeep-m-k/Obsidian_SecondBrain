# Rolling-Origin Validation

## What is it?

**Rolling-origin validation** (time-series cross-validation, a "time gate") tests whether a model generalizes *forward in time* by repeatedly asking: "if I had frozen this model using only data available up to year $Y$, how would it have performed on data from after $Y$?" — repeated across several historical cutoffs.

## Formal Framing

Let $\mathcal{D}_{\leq Y}$ be all data with outcome-observation date $\leq Y$, and $\mathcal{D}_{>Y}$ be data after $Y$. For each cutoff $Y \in \{Y_1, \ldots, Y_m\}$:

1. Fit (or re-run model selection for) $\hat f_Y$ using only $\mathcal{D}_{\leq Y}$.
2. Evaluate $\hat f_Y$ on $\mathcal{D}_{>Y}$, compute $\text{score}_Y(\hat f) - \text{score}_Y(\text{baseline})$.
3. Define a pass/fail indicator $w_Y = \mathbf{1}[\text{model beats baseline at cutoff } Y]$.

A model passes the gate if $\sum_{i=1}^m w_{Y_i} \geq \tau$ for some threshold $\tau$ (e.g. $\tau = 3$ of $m=4$ for a global decision; $\tau = m$, i.e. **every** cutoff, for a stricter segment-level decision).

---

## Worked Example

Four cutoffs, model vs. analog baseline Brier score (lower is better):

| Cutoff $Y$ | Model Brier | Baseline Brier | Model wins? |
|---|---|---|---|
| 2016 | 0.192 | 0.187 | No |
| 2018 | 0.181 | 0.189 | Yes |
| 2020 | 0.176 | 0.184 | Yes |
| 2022 | 0.179 | 0.183 | Yes |

$\sum w_Y = 3$ of $4$. This clears a global-decision bar of $\tau=3$ but would **fail** a segment-override bar requiring $\tau=4$ (all cutoffs) — exactly the asymmetry used when deciding "ship as the global default" vs. "ship as a narrow segment override."

---

## Why a Single Random Split Isn't Enough

A [[Grouped Train-Test Split]] prevents entity-level leakage but says nothing about *temporal* stability. Real-world data drifts — base rates change, populations shift, practices evolve. A single random split mixes old and new data freely into both train and test, which can hide a model that's quietly overfit to one era's patterns.

## Why Segment-Level Decisions Need $\tau = m$

A narrow segment (smaller sample, higher variance) has more opportunities to look good in any *one* period purely by chance — the same logic as the [[Multiple Comparisons Problem]], but across time instead of across segments. Requiring **every** cutoff to pass, not just most, is the temporal analog of demanding a low p-value: it filters out wins that are really just noise concentrated in one lucky period.

## Common Mistakes

- Evaluating with a single random split and treating it as evidence of long-term robustness
- Applying the lenient "wins most cutoffs" bar to a narrow, high-variance segment instead of the stricter "wins every cutoff" bar it warrants
- Re-tuning the model after seeing which cutoffs it failed — this reintroduces circularity, functionally similar to [[Data Leakage]] even though no row is duplicated

## Interview / discussion questions

- Why is passing several historical cutoffs a stronger robustness test than one random split?
- Why should a segment-level override require $\tau=m$ (all cutoffs) while a global decision only needs $\tau < m$?
- What kind of real-world drift would this catch that a random split would miss?

## Prerequisites

[[Grouped Train-Test Split]], [[Data Leakage]]

## Related concepts

[[Multiple Comparisons Problem]], [[Benjamini-Hochberg Procedure]], [[Permutation Test]]

## Tags

#category/statistics #topic/experimental-design

## One-line summary

> Rolling-origin validation repeatedly refits and re-tests a model at several historical cutoffs and requires it to beat the baseline at $\tau$ of $m$ of them — with $\tau=m$ reserved for narrow segment decisions, since they're more vulnerable to one-period luck.
