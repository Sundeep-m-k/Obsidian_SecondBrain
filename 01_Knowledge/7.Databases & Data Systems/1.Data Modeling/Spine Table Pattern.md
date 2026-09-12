# Spine Table Pattern

## What is it?

The **spine table pattern** structures an analysis or ML project around one central table (the "spine") representing the core unit of analysis at a chosen grain, with other tables joined in as validated **enrichment**.

## Formal Framing: Grain

The **grain** of a table is the answer to "what does one row represent?" — formally, the set of columns $\{c_1, \ldots, c_k\}$ such that $(c_1, \ldots, c_k)$ is a candidate key (uniquely identifies a row). Choosing a spine means fixing:

$$\text{grain(spine)} = (\text{entity}, \text{time or version})$$

e.g. `(customer_id, order_id)` for "one row per customer order." Every enrichment join must either match this grain exactly, or explicitly aggregate/pivot to it — silently joining a table at a *finer* grain (e.g. per line item) onto a spine at a *coarser* grain (per order) without aggregating first will silently multiply rows.

---

## Spine + Enrichment, Validated

| Role | Purpose |
|---|---|
| **Spine** | Fixes the grain; carries core identifying and outcome-relevant fields |
| **Enrichment** | Joined via [[Join Validation]]'s coverage/match-rate checks, adds detail the spine doesn't carry |

## Worked Example: Detecting a Grain Mismatch

Spine `orders_denorm` is assumed to have grain `(order_id)`, $|A| = 1{,}500{,}000$ rows — but suppose, unexpectedly, $\text{distinct}(order\_id) = 1{,}230{,}000$. That gap (270,000 duplicate `order_id` values) signals the *actual* grain is finer than assumed — likely `(order_id, line_item_id)`, one row per item in an order rather than one row per order — and any downstream logic assuming one row per order will double-count roughly 22% of orders' revenue. Always check $|A| \stackrel{?}{=} |\text{distinct}(\text{assumed key})|$ before trusting a spine's grain.

## Why It Matters

The spine decision is load-bearing: get the grain wrong and every downstream feature, label, and join inherits the mistake. Worth treating with the same rigor as the unit-of-prediction decision in [[Grouped Train-Test Split]] — get the unit wrong, and everything downstream is contaminated.

## Common Mistakes

- Assuming a grain without verifying $|A| = |\text{distinct}(\text{key})|$
- Trusting a spine-enrichment join without measuring coverage/match rate
- Switching the spine mid-project without re-validating that prior joins and features still make sense at the new grain

## Interview / discussion questions

- How would you empirically verify a table's actual grain rather than assuming it from documentation?
- Why is choosing the spine table one of the most consequential early decisions in a data project?

## Prerequisites

[[Data Warehouse]], [[Denormalization]]

## Related concepts

[[Join Validation]], [[Schema Trust]], [[Fact & Dimension Tables]], [[Keys & Constraints]]

## Tags

#category/databases #topic/data-modeling #math/set-theory

## One-line summary

> The spine's grain is a candidate key you must verify empirically ($|A| = |\text{distinct}(\text{key})|$) — an unverified grain assumption is the single most common way a "spine" table silently multiplies or under-counts everything built on it.
