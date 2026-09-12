# Indication vs. Therapeutic Area

## What is it?

An **indication** is the specific disease/condition a drug targets (e.g. "relapsed multiple myeloma"). A **therapeutic area (TA)** is a broad category grouping many indications (e.g. "Oncology," "Immunology").

## Formalizing Fragmentation

Let $d^*$ be a true underlying disease, and let $\text{ind}(i)$ be the raw indication string recorded for row $i$. Fragmentation occurs when:

$$\{i : \text{disease}(i) = d^*\} \text{ maps to } \{\text{ind}_1, \text{ind}_2, \ldots, \text{ind}_m\}, \quad m > 1$$

— the same true disease appears under $m$ different strings (and often different TA labels too), because indication text isn't perfectly standardized across sources.

## Worked Example: HIV-Related Indication Fragmentation

A real, measured case: rows describing HIV-related indications split across therapeutic area labels as follows:

| Assigned TA | Row count |
|---|---|
| Unclassified | 3,925 |
| Infectious Diseases | 3,868 |
| Immunology | 650 |

Total: 8,443 rows, one disease, three buckets. Rolling up by TA (treating each bucket as a separate group) would compute three different, artificially small-sample advancement rates for what is actually one coherent disease population — none of the three individually reflects the true HIV-program advancement rate, and none has the statistical power the pooled 8,443 rows would provide.

## Why Therapeutic Area Is a Feature, Not an Identity

TA is useful as a **feature** (coarse signal a model can use) but unreliable as **identity** — see [[Drug Pipeline]] for why identity fragmentation biases every downstream rate. The fix is not "ban TA" — it's "never use TA as the join/group key for defining what a program *is*."

## Measured Impact of Blind vs. QA'd Fragmentation Fixes

A blind, automated rollup by indication ID moved headline advancement rates by $-5.1$pp (p1→2) and $-2.5$pp (p2→3) — large enough to be "too risky to ship without hand QA." After manual review confirmed only genuine fragments (`fragmentation_qa_review_v1.csv`) and merged only those, the topline moved by just $-0.1$pp — meaning most of the apparent $-5.1$pp effect from the blind fix was itself noise from over-merging, not real fragmentation. This is a concrete demonstration of why an automated fix needs a QA gate: the "fix" can introduce as much distortion as the problem it solves.

## Common Mistakes

- Assuming indication text is clean enough to group by directly, without checking for fragmentation
- Treating TA as a precise, stable disease category rather than a broad, sometimes inconsistent bucket
- Applying a fully automated fragmentation fix without a QA gate — as the $-5.1$pp vs. $-0.1$pp comparison shows, the automated version can be wrong in its own way

## Interview / discussion questions

- Walk through the HIV fragmentation example and explain why the pooled rate is more trustworthy than any of the three per-TA rates.
- Why did the blind automated fix move rates by $-5.1$pp while the QA'd fix only moved them $-0.1$pp — what does that gap tell you?
- Why can't therapeutic area substitute for a properly resolved program identity?

## Prerequisites

[[Drug Pipeline]]

## Related concepts

[[Entity Resolution]], [[Confidence-Tiered Matching]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> Indication fragmentation is measurable (one HIV-related disease split 3,925/3,868/650 across three TA buckets), and fixing it blindly can distort results as much as the fragmentation itself — a QA'd fix moved the topline by only −0.1pp versus −5.1pp for the blind version.
