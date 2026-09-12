---
tags: [databases, warehousing, dimensional-modeling]
---

# Star & Snowflake Schema

## What is it?

A **star schema** arranges one central [[Fact & Dimension Tables|fact table]] surrounded by directly-joined dimension tables — visually resembling a star. A **snowflake schema** takes the same idea further, normalizing the dimension tables themselves into sub-dimensions, so the diagram "snowflakes" outward into more layers.

## Star Schema

```
                dim_date
                    |
dim_customer -- fact_sales -- dim_product
                    |
                dim_store
```

Every dimension joins directly to the fact table — one join "hop" from fact to any dimension. `dim_product` here is denormalized: `category` and `subcategory` live as plain text columns directly inside `dim_product`, even though they could in principle be their own table.

## Snowflake Schema

```
dim_customer -- fact_sales -- dim_product -- dim_category -- dim_department
                    |
                dim_date
```

Here `dim_product` no longer stores `category` as text — it stores a `category_key` pointing to a separate `dim_category` table, which itself might point to `dim_department`. Querying `department` now requires an extra join hop compared to the star schema.

## The Formal Tradeoff

| | Star | Snowflake |
|---|---|---|
| **Dimension tables** | Denormalized (flat, some redundancy) | Normalized (see [[Normalization]]) |
| **Join complexity** | Fewer joins — fact to dimension directly | More joins — fact to dimension to sub-dimension |
| **Storage** | More redundant (e.g. "Electronics" department name repeated across every category row) | Less redundant |
| **Query performance** | Generally faster (fewer joins, simpler plans) | Generally slower (more joins) but sometimes still fine at modern warehouse scale |
| **Update maintenance** | Renaming a department means updating it in every affected dimension row | Renaming a department means updating one row in `dim_department` |

This is the exact same normalized-vs-denormalized tradeoff from [[Denormalization]], applied specifically to dimension tables rather than the fact table.

## Why Star Usually Wins in Practice

Dimension tables are typically much smaller than fact tables (thousands to low millions of rows, vs. fact tables with billions), so the storage redundancy a star schema accepts is cheap — the entire point of normalizing was to avoid redundancy at scale, but dimension-table redundancy is rarely at meaningful scale. Meanwhile the query-simplicity win (fewer joins, easier for both humans and query planners) is realized on *every single query*. This is why star schema is the dominant pattern in modern analytics warehouses, with snowflaking reserved for cases where a dimension itself is large or has attributes that change independently and need separate tracking.

## Why It Matters

Recognizing which pattern a warehouse uses (or choosing one when designing) is one of the first things a data engineer or analytics engineer does when approaching a new schema — it immediately tells you how many joins a typical question will need and where redundancy risk lives.

## Common Mistakes

- Over-normalizing (snowflaking) a warehouse for the sake of theoretical purity, adding join complexity that buys little given how small dimension tables usually are relative to fact tables
- Assuming star schema means *no* normalization anywhere — the fact table's foreign keys to dimensions are themselves a normalization decision; only the dimension attributes are deliberately kept flat
- Forgetting that renaming/updating a value in a star schema's denormalized dimension can require updating many rows at once, unlike a snowflaked sub-dimension

## Interview / discussion questions

- Draw a star schema and a snowflake schema for the same business process, and explain what changes between them.
- What's the practical tradeoff a warehouse designer makes by choosing star over snowflake?
- Why is dimension-table redundancy in a star schema usually an acceptable cost, when the same redundancy in a fact table would not be?

## Prerequisites

[[Fact & Dimension Tables]], [[Normalization]], [[Denormalization]]

## Related concepts

[[Data Warehouse]], [[Slowly Changing Dimensions]]

## Tags

#category/databases #topic/warehousing #topic/dimensional-modeling

## One-line summary

> Star schema keeps dimension tables flat/denormalized for fewer joins and simpler queries; snowflake schema normalizes dimensions further to reduce redundancy at the cost of extra join hops — star usually wins in practice because dimension tables are small relative to fact tables, so the redundancy it accepts is cheap while the query-simplicity win compounds across every query.
