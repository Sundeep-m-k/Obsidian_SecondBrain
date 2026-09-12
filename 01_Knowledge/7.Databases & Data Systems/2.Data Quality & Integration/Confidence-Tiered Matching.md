# Confidence-Tiered Matching

## What is it?

**Confidence-tiered matching** is the general pattern of resolving a match (identity, join, or label) using the *best available* signal, ranked by trustworthiness, rather than requiring one perfect key or accepting one uniformly low-quality key for everything.

## Formal Pattern

Given candidate matching strategies $\kappa_1 \succ \kappa_2 \succ \cdots \succ \kappa_T$ (ordered by decreasing reliability, $\succ$ meaning "more trustworthy than"), for each record $i$:

$$\text{tier}(i) = \min\{t : \kappa_t(i) \text{ succeeds}\}$$

The critical, often-skipped step: **persist $\text{tier}(i)$ alongside the match itself**. Without it, a Tier-3 fuzzy text match is indistinguishable downstream from a Tier-1 exact-ID match, even though they carry very different risk of being wrong — this is exactly the same structure used in [[Entity Resolution]]'s tiered key and [[Baseline Estimator]]'s backoff ladder (there, $\kappa_t$ = "grouping specificity level $t$," and success = "clears the minimum sample threshold").

---

## Quantifying the Value of Tiering

Suppose Tier 1 matches have an estimated error rate $\epsilon_1 = 0.5\%$, Tier 2 $\epsilon_2 = 3\%$, Tier 3 $\epsilon_3 = 12\%$ (estimated via manual audit of a sample from each tier). If 60% of matches land in Tier 1, 25% in Tier 2, 15% in Tier 3, the **blended** error rate is:

$$\bar\epsilon = 0.60(0.5\%) + 0.25(3\%) + 0.15(12\%) = 0.3\% + 0.75\% + 1.8\% = 2.85\%$$

Reporting $\bar\epsilon$ alone hides that Tier 3 alone (15% of records) contributes $1.8\%$ of the total $2.85\%$ error — nearly two-thirds of the overall error concentrated in one-sixth of the data. Segmenting results by tier surfaces this; a single blended number buries it.

## Why It Matters

Tagging confidence lets downstream analysis make an informed choice — restricting a strict analysis to high-confidence matches only, or reporting results segmented by tier so a reader can judge how much to trust each part.

## Common Mistakes

- Matching records without recording *which* strategy resolved each match
- Treating all resolved matches as equally reliable in downstream reporting
- Never checking the tier distribution — if most records land in the lowest tier, that's a material caveat for everything built on top

## Interview / discussion questions

- Derive the blended error rate formula and explain why it can understate the risk concentrated in the lowest tier.
- Why is it important to record which confidence tier resolved each match, not just the final matched value?
- How is tiered matching structurally the same pattern as a backoff ladder in estimation?

## Prerequisites

[[Entity Resolution]]

## Related concepts

[[Baseline Estimator]], [[Schema Trust]]

## Tags

#category/databases #topic/data-quality #math/set-theory

## One-line summary

> Confidence-tiered matching always tags which tier resolved each match — a blended error rate across tiers can hide that most of the risk concentrates in the lowest-confidence tier, which segmenting by tier exposes.
