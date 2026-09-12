# Benjamini-Hochberg Procedure (FDR Correction)

## What is it?

The **Benjamini-Hochberg (BH) procedure** corrects p-values across many simultaneous [[Hypothesis Test]]s by controlling the **False Discovery Rate (FDR)** — the expected *proportion* of false positives among everything called "significant" — rather than the probability of any single false positive at all (which is what [[Multiple Comparisons Problem]]'s stricter alternative, Bonferroni, controls).

$$\text{FDR} = \mathbb{E}\left[\frac{V}{\max(R,1)}\right]$$

where $V$ = number of false discoveries and $R$ = total number of discoveries (rejections).

---

## Algorithm

1. Run all $k$ tests, obtain p-values $p_1, \ldots, p_k$.
2. Sort ascending: $p_{(1)} \leq p_{(2)} \leq \cdots \leq p_{(k)}$.
3. Choose target FDR level $q$ (e.g. $0.10$).
4. Find the largest rank $i$ such that:

$$p_{(i)} \leq \frac{i}{k} \cdot q$$

5. Reject all tests with rank $\leq i$ (i.e. all $p_{(j)} \leq p_{(i)}$).

---

## Worked Example

Six segment-level p-values, $k=6$, target $q=0.10$:

| Rank $i$ | $p_{(i)}$ | Threshold $\frac{i}{k}q$ | $p_{(i)} \leq$ threshold? |
|---|---|---|---|
| 1 | 0.004 | 0.0167 | Yes |
| 2 | 0.009 | 0.0333 | Yes |
| 3 | 0.031 | 0.0500 | Yes |
| 4 | 0.070 | 0.0667 | No |
| 5 | 0.250 | 0.0833 | No |
| 6 | 0.510 | 0.1000 | No |

The largest rank where $p_{(i)} \leq \frac{i}{k}q$ holds is $i=3$ (rank 4 fails even though $0.070 < 0.10$ — the comparison is against $\frac{i}{k}q$, not $q$ itself). So the first 3 ranked tests are declared significant; segments 4-6 are not, even though segment 4's raw $p=0.070$ would have passed an uncorrected $\alpha=0.10$ cutoff.

---

## Intuition

The threshold each p-value must clear scales with its rank: the smallest p-value needs to beat a strict bar ($\frac{1}{k}q$), but the bar relaxes proportionally moving down the sorted list ($\frac{i}{k}q$). This lets more true discoveries through than a flat Bonferroni threshold ($\alpha/k$ for every test) while still controlling the *expected proportion* of false discoveries.

## Why It Matters

At FDR $=0.10$, roughly 10% of segments called "the model wins here" are expected to be false positives — an explicit, chosen tradeoff. This should gate any segment-level model override, typically alongside a stricter consistency requirement like winning every cutoff in [[Rolling-Origin Validation]] (not just clearing the FDR bar), since BH controls "how many tests" risk but not "is this a fluke of one time period" risk.

## Common Mistakes

- Confusing FDR (expected proportion among *discoveries*) with family-wise error rate (probability of *any* false positive) — different questions, different math
- Applying BH but skipping the separate consistency-across-time check
- Picking $q=0.10$ without being explicit that it means accepting ~10% noise among "wins"

## Interview / discussion questions

- Walk through the BH algorithm on a small worked example and explain why rank 4 in the table above doesn't get rejected despite $p<q$.
- What does FDR $=0.10$ actually promise you, in plain language?
- Why require both BH correction *and* consistency across multiple time-based splits before shipping an override?

## Prerequisites

[[Multiple Comparisons Problem]], [[P-Value]]

## Related concepts

[[Permutation Test]], [[Rolling-Origin Validation]]

## Tags

#category/statistics #topic/hypothesis-testing #math/probability

## One-line summary

> Benjamini-Hochberg sorts p-values and compares each to a rank-scaled threshold $\frac{i}{k}q$, letting more true discoveries through than Bonferroni while still capping the expected proportion of false discoveries at $q$.
