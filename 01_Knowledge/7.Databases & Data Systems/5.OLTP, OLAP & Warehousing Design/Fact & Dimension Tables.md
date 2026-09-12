---
tags: [databases, warehousing, dimensional-modeling]
---

# Fact & Dimension Tables

## What is it?

**Dimensional modeling** organizes an analytical warehouse around two table types: **fact tables** (the measurable events/transactions) and **dimension tables** (the descriptive context those events happened in). This split is the foundation of [[Star & Snowflake Schema]] design and the dominant pattern for how real-world data warehouses are actually structured.

## Core Concepts

| | Fact table | Dimension table |
|---|---|---|
| **Contains** | Numeric, additive measures + foreign keys to dimensions | Descriptive attributes (text, categories, hierarchies) |
| **Grain** | One row per business event (e.g. one row per order line) | One row per entity (e.g. one row per product) |
| **Size** | Large, grows continuously (every new order adds rows) | Small(er), relatively stable |
| **Example** | `fact_orders(order_id, customer_key, product_key, date_key, quantity, amount)` | `dim_customer(customer_key, name, region, signup_date)` |

## Worked Example

```sql
fact_sales(
    sale_id,
    date_key      FK → dim_date.date_key,
    customer_key  FK → dim_customer.customer_key,
    product_key   FK → dim_product.product_key,
    quantity,
    amount
)

dim_date(date_key, full_date, day_of_week, month, quarter, year, is_holiday)
dim_customer(customer_key, name, email, region, signup_date, segment)
dim_product(product_key, name, category, subcategory, unit_price)
```

A typical analytical query joins the fact table to whichever dimensions the question needs:

```sql
SELECT d.quarter, p.category, SUM(f.amount) AS revenue
FROM fact_sales f
JOIN dim_date d ON d.date_key = f.date_key
JOIN dim_product p ON p.product_key = f.product_key
GROUP BY d.quarter, p.category;
```

## Why the Split, Formally

The grain of the fact table is fixed at the finest useful level of the business process (see [[Spine Table Pattern]] for the general "grain" concept) — one row per sale, not pre-aggregated — because you can always aggregate a fine grain up (`GROUP BY`), but you can never recover detail from a grain that's already been aggregated away. Dimensions are kept as separate, smaller tables specifically so their attributes (like `region` or `category`) don't need to be repeated on every single fact row — this is the same normalization principle from [[Normalization]] applied selectively: dimensions stay relatively normalized, while the fact table itself intentionally avoids embedding descriptive text, keeping it narrow and numeric for fast scanning.

## Types of Facts

| Fact type | Meaning | Example |
|---|---|---|
| **Additive** | Can be summed across any dimension | `amount` — sums correctly across product, date, customer |
| **Semi-additive** | Summable across some dimensions, not others | `account_balance` — summing across accounts makes sense, summing across time doesn't |
| **Non-additive** | Can't be meaningfully summed at all | `unit_price`, `discount_rate` — use `AVG()`, not `SUM()` |

Misidentifying a semi-additive or non-additive fact as additive (e.g. summing `account_balance` across months to get a "total") produces a number that looks plausible but means nothing — a subtle, common warehouse-design bug.

## Why It Matters

This pattern is the industry-standard vocabulary for data warehouse design (Kimball methodology) — nearly every BI tool, warehouse, and data-engineering job description assumes familiarity with "fact table," "dimension table," and "grain" as baseline terms.

## Common Mistakes

- Summing a semi-additive or non-additive measure across the wrong dimension, producing a number that looks valid but is meaningless
- Embedding descriptive attributes directly in the fact table instead of a dimension, bloating fact-table row size and duplicating text across millions of rows
- Choosing a fact table grain that's too coarse (pre-aggregated) for questions that later need finer detail — grain should match the finest level any anticipated question will need

## Interview / discussion questions

- What's the difference between a fact table and a dimension table, and why is the split useful?
- Explain additive vs. semi-additive vs. non-additive facts, with an example of each.
- Why should a fact table's grain be chosen as fine as practically useful, rather than pre-aggregated?

## Prerequisites

[[Normalization]], [[Spine Table Pattern]]

## Related concepts

[[Star & Snowflake Schema]], [[Slowly Changing Dimensions]], [[Data Warehouse]]

## Tags

#category/databases #topic/warehousing #topic/dimensional-modeling

## One-line summary

> A fact table holds fine-grained, mostly-numeric business events (one row per transaction) while dimension tables hold the descriptive context around them (customer, product, date) — kept separate so dimension attributes aren't duplicated across millions of fact rows, with additive/semi-additive/non-additive classification determining which aggregations are even valid.
