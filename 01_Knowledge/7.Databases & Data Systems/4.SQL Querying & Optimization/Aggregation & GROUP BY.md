---
tags: [databases, sql]
---

# Aggregation & GROUP BY

## What is it?

**Aggregation** collapses many rows into a summary value (`SUM`, `COUNT`, `AVG`, `MIN`, `MAX`). **`GROUP BY`** applies that collapse separately per distinct value of one or more columns, producing one output row per group.

## Core Concept: The Grain Change

`GROUP BY` fundamentally changes a table's grain (see [[Spine Table Pattern]] for the general concept of grain). Before: one row per order. After `GROUP BY customer_id`: one row per customer. This is worth stating explicitly because it's the most common source of confusion — a column that made sense at the pre-aggregation grain (e.g. `order_id`) has no single value once grouped, and must either be aggregated (`COUNT(order_id)`) or dropped from the `SELECT` list entirely.

```sql
SELECT
    customer_id,
    COUNT(order_id)   AS num_orders,
    SUM(amount)        AS total_spent,
    AVG(amount)        AS avg_order_value,
    MAX(order_date)    AS most_recent_order
FROM orders
GROUP BY customer_id;
```

## The Rule: Every Non-Aggregated Column Must Be in GROUP BY

Formally: for a query `SELECT c1, c2, agg(c3) FROM T GROUP BY c1, c2`, every selected column that isn't wrapped in an aggregate function must appear in the `GROUP BY` clause — because otherwise SQL has no defined rule for which of potentially many differing values to return for that column within a group. Most databases enforce this at parse time; a few (older MySQL configurations) silently allow it and return an arbitrary value, which is a well-known correctness trap.

## HAVING vs. WHERE — Filtering Before vs. After Aggregation

```sql
SELECT customer_id, SUM(amount) AS total_spent
FROM orders
WHERE order_date >= '2026-01-01'   -- filters rows BEFORE grouping
GROUP BY customer_id
HAVING SUM(amount) > 1000;          -- filters groups AFTER aggregation
```

`WHERE` cannot reference an aggregate (`WHERE SUM(amount) > 1000` is invalid) because at the point `WHERE` is evaluated, aggregation hasn't happened yet — this is a direct consequence of SQL's logical execution order: `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY`. `HAVING` exists specifically because `WHERE` runs too early to filter on an aggregate result.

## Worked Example: Grain Mismatch After a Join

Joining `orders` to `order_items` before aggregating changes the grain being summed:

```sql
-- WRONG: double-counts order amount once per line item
SELECT o.customer_id, SUM(o.amount)
FROM orders o
JOIN order_items oi ON oi.order_id = o.order_id
GROUP BY o.customer_id;

-- RIGHT: aggregate order_items separately first, or use SUM(DISTINCT) carefully,
-- or aggregate orders and order_items independently before joining
```

If an order has 3 line items, joining it to `order_items` produces 3 rows for that one order — and `SUM(o.amount)` now adds that order's total 3 times. This is exactly the row-multiplication failure mode described in [[Join Validation]]'s row-count sanity check ($|A \bowtie B| > |A|$ signals unintended fan-out) — except here it silently corrupts a sum rather than just inflating a row count, which makes it more dangerous because the query still "looks" reasonable.

## Why It Matters

Aggregation-after-join grain mismatches are one of the most common real-world sources of silently wrong numbers in reporting — the query runs, returns a plausible-looking number, and is wrong by exactly the average fan-out factor of the join. Always ask "does aggregating here happen at the correct grain, before or after this join?"

## Common Mistakes

- Summing a column from the "one" side of a one-to-many join without aggregating the "many" side first — silently multiplies the sum
- Attempting to filter on an aggregate with `WHERE` instead of `HAVING`
- Selecting a non-aggregated, non-grouped column and assuming the database picks a "sensible" value for it (undefined behavior, and outright rejected by most modern databases)

## Interview / discussion questions

- Explain SQL's logical execution order and why `HAVING` exists as distinct from `WHERE`.
- Why does joining a one-to-many relationship before aggregating the "one" side's value produce an inflated sum? Walk through a concrete example.
- What rule governs which columns can appear in `SELECT` alongside a `GROUP BY`?

## Prerequisites

[[SQL Join Types]]

## Related concepts

[[Window Functions]], [[Join Validation]], [[Spine Table Pattern]]

## Tags

#category/databases #topic/sql

## One-line summary

> `GROUP BY` changes a table's grain, collapsing rows into one per distinct group value — every non-aggregated selected column must appear in the grouping, `HAVING` filters after aggregation where `WHERE` can't, and aggregating a "one" side's value after joining in a "many" side is the single most common way to silently inflate a sum.
