---
tags: [databases, warehousing, dimensional-modeling]
---

# Slowly Changing Dimensions (SCD)

## What is it?

A **slowly changing dimension** is a dimension table (see [[Fact & Dimension Tables]]) whose attribute values change occasionally over time — e.g. a customer moves regions, a product gets recategorized. **SCD types** are the standard, named strategies for handling what happens to historical fact-table joins when that change occurs.

## The Core Problem

Suppose `dim_customer.region` for customer 501 is `"West"`, and 10,000 historical `fact_sales` rows already reference `customer_key = 501`. The customer moves and `region` is updated to `"East"`. What should a report of *last year's* West-region revenue show now — should it still include customer 501's old sales, or not? The answer depends entirely on which SCD strategy the dimension uses, and getting it wrong silently rewrites history.

## SCD Type 1 — Overwrite (No History)

```sql
UPDATE dim_customer SET region = 'East' WHERE customer_key = 501;
```
Simplest option: the old value is gone. Every historical fact-table row joined to this customer now reports `"East"`, even for sales made while they lived in `"West"`. Appropriate when the old value genuinely doesn't matter (e.g. correcting a data-entry typo) — inappropriate for a real historical change, since it silently rewrites the past.

## SCD Type 2 — Add a New Row (Full History)

```sql
customer_key | customer_id | region | effective_date | end_date   | is_current
1001         | 501         | West   | 2024-01-01      | 2026-03-14 | false
1002         | 501         | East   | 2026-03-15      | NULL       | true
```
The **natural key** (`customer_id`) stays the same, but a *new* surrogate `customer_key` is created for the new version, with validity dates tracking which version was true when. Historical fact rows keep pointing to `customer_key = 1001` (correctly preserving "this sale happened while the customer was in West"); new sales get `customer_key = 1002`. This is the standard choice when historical accuracy matters, which is most of the time in a real analytical warehouse.

$$\text{Correct historical join: fact.customer\_key} = \text{dim.customer\_key (version active at fact's date)}$$

## SCD Type 3 — Add a New Column (Limited History)

```sql
customer_key | customer_id | current_region | previous_region
1001         | 501         | East           | West
```
Keeps exactly one prior value in an extra column — cheaper than Type 2, but can't track more than one change, and doesn't let historical facts automatically resolve to "the region at the time." Rarely used except for very specific, limited "what changed most recently" reporting needs.

## Why This Connects Back to Grain and Keys

SCD Type 2 is the dimensional-modeling instance of a broader idea already covered in [[Keys & Constraints]] and [[Spine Table Pattern]]: the **surrogate key** (`customer_key`) is deliberately *not* the same as the natural business key (`customer_id`), precisely so the same real-world entity can have multiple valid rows over time without violating uniqueness — each `customer_key` value uniquely identifies one *version* of the customer, not the customer overall.

## Why It Matters

Choosing the wrong SCD type is a common, expensive warehouse-design mistake: Type 1 on a dimension that should preserve history quietly corrupts every historical report that touches it, and the corruption is invisible unless someone specifically compares against un-overwritten source data.

## Common Mistakes

- Defaulting to Type 1 (simple overwrite) out of convenience on a dimension where historical accuracy actually matters
- Joining fact tables to a dimension using the natural key instead of the surrogate key under SCD Type 2 — this collapses all historical versions back into "whatever the current value is," defeating the entire point
- Forgetting to set `end_date`/`is_current` correctly when inserting a new SCD Type 2 row, leaving two rows simultaneously marked "current"

## Interview / discussion questions

- Explain SCD Types 1, 2, and 3, and give a business scenario where each is the right choice.
- Why does SCD Type 2 require a surrogate key distinct from the natural business key?
- What silently breaks in historical reporting if a dimension that should be Type 2 is implemented as Type 1 instead?

## Prerequisites

[[Fact & Dimension Tables]], [[Keys & Constraints]]

## Related concepts

[[Star & Snowflake Schema]], [[Spine Table Pattern]]

## Tags

#category/databases #topic/warehousing #topic/dimensional-modeling

## One-line summary

> Slowly changing dimensions define how a warehouse handles attribute changes over time — Type 1 overwrites and loses history, Type 2 preserves full history via new surrogate-keyed rows with validity date ranges, and Type 3 keeps only one prior value — with Type 2 the standard choice whenever historical accuracy in fact-table joins actually matters.
