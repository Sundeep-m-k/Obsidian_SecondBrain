# Permutation Test

## What is it?

A **permutation test** is a non-parametric [[Hypothesis Test]] that estimates a [[P-Value]] by directly simulating $H_0$: shuffle group labels many times, recompute the statistic of interest each time, and see how often the shuffled version is at least as extreme as the real, unshuffled result. No assumption about the underlying distribution of the data is required — the null distribution is built empirically from the data itself.

---

## Algorithm

Given two groups' scores (e.g. model B's per-example results vs. baseline's), and an observed statistic $t_{\text{obs}}$ (e.g. mean difference):

1. Pool all $n = n_1 + n_2$ observations together.
2. Randomly split the pool into two groups of sizes $n_1, n_2$ (a random permutation of the group labels).
3. Recompute the statistic $t^{(b)}$ on this shuffled split.
4. Repeat $B$ times (typically $B = 10{,}000$).
5. Estimate the p-value:

$$p = \frac{1 + \#\{b : t^{(b)} \geq t_{\text{obs}}\}}{1 + B}$$

The $+1$ in numerator and denominator accounts for the observed arrangement itself being one valid permutation — this also sets the **smallest possible p-value** at $\frac{1}{B+1}$, which matters when you need a very small p-value to survive a [[Benjamini-Hochberg Procedure]] correction across many tests.

---

## Worked Example

Segment "Psychiatry, 4+ trials": baseline advance rate $0.31$ over $n_1=120$ programs, model $0.39$ over $n_2=118$ programs. Observed difference $t_{\text{obs}} = 0.08$. Running $B=10{,}000$ random reshuffles of which programs get the "model" vs "baseline" label and recomputing the rate difference each time, suppose $210$ of the $10{,}000$ shuffles produce a difference $\geq 0.08$:

$$p = \frac{1+210}{1+10{,}000} \approx 0.0211$$

This single segment clears $\alpha=0.05$ — but if this is one of 60 segments tested, that raw $p\approx0.02$ must go through [[Benjamini-Hochberg Procedure]] before it can justify a segment-level override.

---

## Why It's Useful Here Specifically

Real-world outcome data (advancement rates, skewed enrollment counts) rarely matches the clean assumptions (normality, equal variance) that a t-test needs. A permutation test sidesteps that by building the null distribution empirically — it asks "if group membership were meaningless, how often would a gap this big appear by chance?" directly from the data's own variability.

## Common Mistakes

- Shipping any segment with $p<0.05$ without correcting for the number of segments tested — see [[Multiple Comparisons Problem]]
- Using too few permutations (e.g. $B=100$) to reliably resolve a small p-value — the resolution floor is $\frac{1}{B+1}$
- Permuting the wrong thing — what gets shuffled must correspond exactly to what $H_0$ claims is irrelevant (e.g. shuffle *labels*, not *features*)

## Interview / discussion questions

- Derive why the smallest p-value a permutation test with $B$ shuffles can report is $\frac{1}{B+1}$.
- Why doesn't a permutation test require assuming a particular data distribution, unlike a t-test?
- Why must permutation-test results across many segments be corrected before acting on them?

## Prerequisites

[[Hypothesis Test]], [[P-Value]]

## Related concepts

[[Multiple Comparisons Problem]], [[Benjamini-Hochberg Procedure]], [[Rolling-Origin Validation]]

## Tags

#category/statistics #topic/hypothesis-testing #math/probability

## One-line summary

> A permutation test builds the null distribution by reshuffling labels thousands of times and counting how often a gap this large appears by chance — assumption-light, but its p-value resolution is capped at $1/(B+1)$ and still needs multiple-comparisons correction across segments.
