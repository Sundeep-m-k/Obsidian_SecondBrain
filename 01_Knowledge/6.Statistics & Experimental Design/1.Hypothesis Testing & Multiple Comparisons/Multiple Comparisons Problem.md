# Multiple Comparisons Problem

## What is it?

The **multiple comparisons problem** (multiple testing problem) is the fact that running many [[Hypothesis Test]]s at once inflates the overall chance of at least one false positive, even if every individual test uses a "safe" threshold like $\alpha=0.05$.

---

## The Math

If $k$ independent tests are run, each with false-positive rate $\alpha$ under $H_0$, the probability that **none** are false positives is $(1-\alpha)^k$, so:

$$P(\text{at least one false positive}) = 1-(1-\alpha)^k$$

| $k$ | $P(\text{at least one false positive})$ at $\alpha=0.05$ |
|---|---|
| 1 | 5.0% |
| 5 | 22.6% |
| 20 | 64.2% |
| 60 | 95.4% |
| 100 | 99.4% |

Testing 60 segments (e.g. therapeutic area × trial-size bucket) at $\alpha=0.05$ each means a >95% chance that *at least one* looks like a real win purely by chance, even if nothing real is happening anywhere.

---

## The Fix

| Approach | Controls | Threshold per test | Strictness |
|---|---|---|---|
| Bonferroni | Family-wise error rate: $P(\text{any false positive})$ | $\alpha/k$ | Very strict, conservative |
| [[Benjamini-Hochberg Procedure]] (FDR) | Expected *proportion* of false positives among rejections | rank-dependent, $\leq \frac{i}{k}q$ | Less strict, practical at scale |

Bonferroni guarantees $P(\text{any false positive}) \leq \alpha$ by making each individual test much harder to pass — at $k=60$, the per-test bar becomes $0.05/60 \approx 0.00083$, which real, modest effects often can't clear. FDR control instead accepts a *rate* of false positives among the discoveries you make, trading some false-positive risk for much more power to detect real effects.

---

## Why It Matters Here Specifically

Validating whether a model beats a baseline across many segments and shipping overrides based on uncorrected per-segment [[Permutation Test]] results would mean shipping noise dressed up as signal — the table above shows this isn't a small risk, it's close to guaranteed at realistic segment counts.

## Common Mistakes

- Reporting only the segments that happened to clear $p<0.05$, with no correction — p-hacking, even unintentionally
- Using Bonferroni when testing hundreds of segments — often needlessly conservative, missing real effects
- Forgetting that correction alone doesn't guard against a result that's a fluke of *one time period* — that's what [[Rolling-Origin Validation]] is for, and both safeguards are needed together

## Interview / discussion questions

- Derive $P(\text{at least one false positive})$ for $k$ independent tests at significance $\alpha$.
- What's the practical difference between controlling family-wise error rate and false discovery rate?
- Why is requiring a segment to win across multiple time splits a *different* safeguard than a multiple-comparisons correction?

## Prerequisites

[[Hypothesis Test]], [[P-Value]]

## Related concepts

[[Benjamini-Hochberg Procedure]], [[Permutation Test]], [[Rolling-Origin Validation]]

## Tags

#category/statistics #topic/hypothesis-testing #math/probability

## One-line summary

> Testing $k$ hypotheses at $\alpha=0.05$ each pushes the chance of *some* false positive toward certainty as $k$ grows ($1-(1-\alpha)^k$), which is why segment-by-segment results need an explicit correction like Benjamini-Hochberg before anyone acts on them.
