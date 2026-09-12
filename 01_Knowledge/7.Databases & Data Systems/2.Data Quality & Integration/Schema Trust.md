# Schema Trust

## What is it?

**Schema trust** is the discipline of treating a table or column's *name*, documentation, or apparent purpose as a hypothesis to verify — not a fact to rely on.

## The Verification Checklist, Formalized

For a candidate column $c$ in table $T$:

$$\text{population rate}(c) = \frac{|\{i \in T : c(i) \neq \text{null}\}|}{|T|}$$

1. **Population rate** — is $\text{population rate}(c)$ meaningfully above 0? A column can exist and be perfectly named while being 99.9% null.
2. **Distribution sanity** — do the non-null values look like real signal (reasonable range, expected categories) or like defaults/placeholders (e.g. all zeros, all the same string)?
3. **Freshness** — is $c$ still written to by current pipelines, or a relic of a deprecated process (check max(`updated_at`) or similar)?
4. **Cross-check** — does $c$ agree with an independently-trustworthy related field, on a sample?

Only after this passes should $c$ be treated as real signal rather than a name-shaped hope.

---

## Why This Is a Recurring Trap

A column literally named exactly what you're looking for (e.g. `customer_lifetime_value`, or a foreign-key-shaped ID column) is *maximally tempting* to skip-verify — which is exactly why it's the most dangerous case. Counterintuitively: **the better a column's name matches your need, the more scrutiny it deserves**, not less, because a well-named-but-empty column is precisely the kind of thing that gets used without a second look.

## Worked Example

`customers.lifetime_value_score` — name is a perfect match for a churn-risk or marketing-spend analysis. Checking population rate: $0$ of $591{,}000$ rows populated → $\text{population rate} = 0.0$. It turns out the column was added for a scoring model that was never shipped, and nothing ever writes to it. Distinguishing signal from noise here took one query; skipping it would have meant building a whole downstream analysis on a column that doesn't exist in practice.

## Why It Matters

Skipping this check is one of the most common ways a project quietly builds on sand. Documenting which tables/columns were checked and rejected (and why) turns a one-time discovery into reusable knowledge for anyone who touches the project later — including future-you.

## Common Mistakes

- Using a column because its name is a perfect match, without checking population rate or distribution
- Assuming a column used in an existing dashboard is automatically reliable enough for a new analytical use
- Not documenting *why* a tempting column was rejected, forcing future rediscovery of the same trap

## Interview / discussion questions

- Why should a column whose name perfectly matches your need get *more* scrutiny, not less?
- Walk through the four-step verification checklist and explain what failure mode each step catches.

## Prerequisites

[[Data Warehouse]]

## Related concepts

[[Join Validation]], [[Entity Resolution]], [[Confidence-Tiered Matching]], [[Keys & Constraints]]

## Tags

#category/databases #topic/data-quality #math/set-theory

## One-line summary

> A column's name is a hypothesis, not a guarantee — population rate, distribution sanity, freshness, and cross-checks are the four concrete tests that convert "this looks right" into "this is verified," and the most tempting names deserve the most scrutiny.
