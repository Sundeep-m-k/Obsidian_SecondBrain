# Data Leakage

## What is it?

**Data leakage** happens when information that wouldn't actually be available at prediction time enters the training data, making a model look far more accurate during evaluation than it will be in deployment.

## Formal Framing

Let $t$ be the decision time (e.g. "as of Phase 1") and $\mathcal{F}_t$ be the set of information genuinely knowable at time $t$. A feature $x_j$ is **leaky** if:

$$x_j = g(\mathcal{F}_{t'}), \quad t' > t$$

i.e. it's a function of information from *after* the decision point. This is a precise, checkable definition: for every feature, ask "what is the earliest time $t'$ at which this value is knowable?" If $t' > t$, it leaks.

---

## Two Main Flavors

**Feature leakage** — a feature is built from post-decision information. Classic example: lifetime total NCT count as a feature to predict Phase 1→2 advancement. A program that eventually reaches Phase 3 accumulates more trials over its *whole life* than one that dies at Phase 1 — so lifetime NCT count is $g(\mathcal{F}_{t'})$ for $t' \gg t$, and using it means the feature partially **encodes the label**. Fixed by restricting every feature to "from-phase-only" — computed using only $\mathcal{F}_t$.

**Label leakage** — information correlated with the label ends up directly in the features, often via pipeline bugs (a "final status" column populated only after the outcome is known, accidentally included as an input).

## A Quick Diagnostic: Leakage Inflates Apparent Signal

If $x_j$ is leaky, its correlation with $y$ in the training data will typically be far higher than any legitimately-available feature could produce — because it's partially a restatement of $y$ itself, not a genuine predictor of it. A single feature with implausibly high importance is the most common tell.

---

## How to Detect It

- The model performs *suspiciously* well — near-perfect accuracy on a genuinely hard, noisy real-world problem is a red flag, not a triumph
- One feature dominates importance by a wide margin
- Performance collapses on deployment despite strong validation metrics — because deployment necessarily only has access to $\mathcal{F}_t$, exposing any feature that secretly depended on $\mathcal{F}_{t'}$

## Why It Matters

Leakage is the single most common reason an ML project looks great in evaluation and fails in production. Evaluation protocols must be designed adversarially against yourself: treat every strong result as possibly leaky until the feature set is audited against the $t' > t$ test, and until a [[Grouped Train-Test Split]] rules out entity-level leakage too.

## Common Mistakes

- Using any feature that aggregates over an entity's *entire lifetime* to predict an earlier-stage outcome for that same entity
- Not asking, for every feature: "what's the earliest time this value is actually knowable?"
- Treating leakage as purely a coding bug — often it's a *design* bug: a feature is leaky even when the code computing it is perfectly correct

## Interview / discussion questions

- Give an example of a feature that looks harmless but is leaky under the $x_j = g(\mathcal{F}_{t'}), t' > t$ definition.
- Why does "the model has 99% accuracy" sometimes count as evidence *against* the model?
- How would you audit a feature set for leakage systematically before trusting a model's evaluation numbers?

## Prerequisites

[[Overfitting]], [[Generalization]], [[Feature Engineering]]

## Related concepts

[[Grouped Train-Test Split]], [[Rolling-Origin Validation]], [[Test Error]]

## Tags

#category/statistics #topic/experimental-design

## One-line summary

> A feature leaks if it's a function of information only knowable after the prediction's decision point ($t' > t$) — the fix is a per-feature audit of "when is this actually knowable," not just distrust of suspiciously good accuracy after the fact.
