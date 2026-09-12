# Denormalization

## What is it?

**Denormalization** combines data that would normally live in separate, normalized tables into a single, wider table — trading storage redundancy and update complexity for simpler, faster reads.

## Normalized vs. Denormalized

| | Normalized | Denormalized |
|---|---|---|
| Structure | Data split across linked tables, each fact stored once | Data pre-joined into fewer, wider tables, facts may repeat |
| Reads | Require joins: cost roughly $O(\text{number of joins} \times \log n)$ per query with indexes | $O(1)$ table scan, no join cost |
| Writes/updates | Update one place | May need updating the same fact across many rows — risk of inconsistency if any copy is missed |
| Typical use | Operational/transactional systems | Analytics, reporting, ML-ready datasets |

The core tradeoff is a classic space/time-vs-consistency one: denormalization pays redundant storage and update risk up front, in exchange for avoiding repeated join cost on every read — worthwhile when reads vastly outnumber writes, which is almost always true for an analytics table that's built once and queried thousands of times.

---

## Why It Matters

A denormalized "snapshot" table (one row per order at time of purchase, with customer/product/price/shipping-status pre-joined) is often the natural **spine table** (see [[Spine Table Pattern]]) — already at the right grain, no reconstruction needed via joins every query.

## Staleness: The Tradeoff to Watch For

A denormalized table is a *materialized* function of its normalized sources: $\text{denorm}(t) = f(\text{source}_1(t), \ldots, \text{source}_k(t))$. If the pipeline computing $f$ doesn't rerun after the sources change, $\text{denorm}$ silently drifts from the true current state. Always check: when was this table last refreshed, and is it still consistent with the sources it claims to summarize?

## Common Mistakes

- Assuming a denormalized "convenience" table is automatically up to date — it can lag behind or omit fields the underlying normalized tables now have
- Not checking whether a denormalized table's grain actually matches the unit of analysis needed (see [[Spine Table Pattern]])

## Interview / discussion questions

- What's the tradeoff between a normalized schema and a denormalized one, in terms of read cost vs. update consistency risk?
- Why might an ML project prefer building features from a denormalized spine table rather than joining many normalized tables at query time?

## Prerequisites

Basic relational database concepts

## Related concepts

[[Data Warehouse]], [[Spine Table Pattern]], [[Normalization]], [[OLTP vs OLAP]]

## Tags

#category/databases #topic/data-modeling

## One-line summary

> Denormalization pre-joins data into wide tables to avoid repeated join cost on every read, at the price of redundancy and staleness risk — a good candidate spine table for analysis, but one whose freshness needs to be checked, not assumed.
