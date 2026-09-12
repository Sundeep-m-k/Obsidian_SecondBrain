---
tags: [databases, sql, window-functions]
---

# Window Functions

## What is it?

A **window function** computes a value across a set of related rows (a "window") *without collapsing them into a single output row* — unlike `GROUP BY`, which reduces many rows to one per group. This is what makes window functions the tool for "value relative to its group" questions: running totals, rankings, moving averages, period-over-period comparisons.

## Core Syntax

```sql
function(column) OVER (
    PARTITION BY grouping_column   -- like GROUP BY, but doesn't collapse rows
    ORDER BY ordering_column       -- defines row order within each partition
    ROWS BETWEEN ... AND ...       -- optional: restricts the window further
)
```

## Worked Example: Rank Each Customer's Orders by Size

```sql
SELECT
    customer_id,
    order_id,
    amount,
    RANK() OVER (PARTITION BY customer_id ORDER BY amount DESC) AS order_rank,
    SUM(amount) OVER (PARTITION BY customer_id) AS customer_total,
    amount / SUM(amount) OVER (PARTITION BY customer_id) AS pct_of_customer_total
FROM orders;
```

| customer_id | order_id | amount | order_rank | customer_total | pct_of_customer_total |
|---|---|---|---|---|---|
| 501 | 1 | 80 | 1 | 120 | 0.667 |
| 501 | 2 | 40 | 2 | 120 | 0.333 |
| 502 | 3 | 50 | 1 | 50 | 1.000 |

Note every original row is preserved — `GROUP BY customer_id` alone could only have produced `customer_total`, collapsing to 2 rows total; the window function keeps all 3 rows while still attaching group-level context to each.

## Key Function Categories

| Category | Functions | Purpose |
|---|---|---|
| **Ranking** | `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `NTILE(n)` | Order rows within a partition; differ in tie-handling |
| **Aggregate as window** | `SUM()`, `AVG()`, `COUNT()`, `MIN()`, `MAX()` `OVER (...)` | Group-level stats attached to every row, not collapsed |
| **Offset** | `LAG(col, n)`, `LEAD(col, n)` | Access a previous/next row's value — the standard way to compute period-over-period change |
| **Distribution** | `PERCENT_RANK()`, `CUME_DIST()` | Relative standing within a partition |

`RANK()` vs `DENSE_RANK()` vs `ROW_NUMBER()` differ only in tie behavior: given tied values `[80, 80, 40]`, `ROW_NUMBER()` gives `1, 2, 3` (arbitrary tiebreak), `RANK()` gives `1, 1, 3` (ties share a rank, next rank skips), `DENSE_RANK()` gives `1, 1, 2` (ties share a rank, no gap).

## Worked Example: Month-over-Month Growth with LAG

```sql
SELECT
    month,
    revenue,
    LAG(revenue) OVER (ORDER BY month) AS prev_month_revenue,
    revenue - LAG(revenue) OVER (ORDER BY month) AS mom_change
FROM monthly_revenue
ORDER BY month;
```

This is the SQL-native way to compute a rolling comparison — the same conceptual operation as a **rolling-origin** setup in [[Rolling-Origin Validation]], but expressed as a query instead of a train/test split: each row only ever looks at data from *earlier* rows via `LAG`, never later ones.

## Why It Matters

Window functions are the standard tool for exactly the analytical questions that come up constantly in data roles — "top N per group," "running total," "percent of total," "change since last period" — expressed in pure SQL, without pulling data into a script. Interviewers ask about them specifically because a candidate who reaches for `GROUP BY` + a self-join to answer "top 3 orders per customer" is solving in $O(n^2)$ what a window function solves in one pass.

## Common Mistakes

- Confusing `GROUP BY` (collapses rows) with a windowed aggregate (preserves rows) — using the wrong one either loses needed detail or produces a result at the wrong grain
- Forgetting `PARTITION BY`, which silently computes the window over the *entire table* instead of per group
- Not specifying `ORDER BY` inside `OVER()` when using `LAG`/`LEAD`/`RANK` — these are meaningless without a defined row order

## Interview / discussion questions

- What's the difference between a window function and `GROUP BY`, precisely?
- Explain the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` on tied values.
- Write a query to find each customer's single largest order using a window function (hint: `RANK() ... WHERE order_rank = 1` requires a subquery, since window functions can't be filtered directly in `WHERE`).
- How would you compute month-over-month percentage growth using `LAG`?

## Prerequisites

[[SQL Join Types]], basic `GROUP BY` / aggregation

## Related concepts

[[Common Table Expressions (CTEs)]], [[Aggregation & GROUP BY]], [[Rolling-Origin Validation]]

## Tags

#category/databases #topic/sql #topic/analytics

## One-line summary

> Window functions compute group-relative values (rankings, running totals, period-over-period change) via `OVER (PARTITION BY ... ORDER BY ...)` without collapsing rows the way `GROUP BY` does — the standard, efficient tool for "value relative to its group" questions that a naive self-join would otherwise solve in quadratic time.
