---
tags: [databases, sql]
---

# Common Table Expressions (CTEs)

## What is it?

A **CTE** (`WITH ... AS (...)`) names a temporary, query-scoped result set that can be referenced later in the same query — a way to break a complex query into readable, sequential steps instead of nesting subqueries inside subqueries.

## Core Syntax

```sql
WITH cte_name AS (
    SELECT ...
)
SELECT ... FROM cte_name ...;
```

Multiple CTEs can be chained, each able to reference the ones defined before it:

```sql
WITH customer_totals AS (
    SELECT customer_id, SUM(amount) AS total_spent
    FROM orders
    GROUP BY customer_id
),
top_customers AS (
    SELECT customer_id, total_spent
    FROM customer_totals
    WHERE total_spent > 1000
)
SELECT c.name, t.total_spent
FROM top_customers t
JOIN customers c ON c.customer_id = t.customer_id
ORDER BY t.total_spent DESC;
```

## Why It Matters: Readability vs. the Nested-Subquery Alternative

The same query without CTEs forces nesting subqueries inside the `FROM` clause, read inside-out:

```sql
SELECT c.name, t.total_spent
FROM (
    SELECT customer_id, total_spent
    FROM (
        SELECT customer_id, SUM(amount) AS total_spent
        FROM orders GROUP BY customer_id
    ) AS customer_totals
    WHERE total_spent > 1000
) AS t
JOIN customers c ON c.customer_id = t.customer_id
ORDER BY t.total_spent DESC;
```

Same result, same execution plan in most databases — CTEs are (in most systems) a **readability construct**, not a performance one. The value is that each step gets a name and reads top-to-bottom in the order a person actually thinks through the problem: "first compute totals per customer, then filter to the big spenders, then join in their names."

## Recursive CTEs

A CTE can reference *itself*, enabling traversal of hierarchical or graph-shaped data that a plain query can't express — e.g. an employee-manager reporting chain of unknown depth:

```sql
WITH RECURSIVE reporting_chain AS (
    -- anchor: start with the target employee
    SELECT employee_id, manager_id, 0 AS depth
    FROM employees
    WHERE employee_id = 501

    UNION ALL

    -- recursive step: walk up to each manager
    SELECT e.employee_id, e.manager_id, r.depth + 1
    FROM employees e
    JOIN reporting_chain r ON e.employee_id = r.manager_id
)
SELECT * FROM reporting_chain;
```

This terminates when the recursive step produces no new rows (e.g. reaching an employee with no manager) — formally, it's computing the **transitive closure** of the `manager_id` relationship starting from one row.

## Why It Matters

CTEs are the practical, everyday tool for making a multi-step analytical query maintainable — most real business questions ("active customers, their order counts, filtered to repeat buyers, joined to their region") naturally decompose into 3-5 sequential steps, and a CTE chain mirrors that decomposition directly in the SQL itself.

## Common Mistakes

- Assuming a CTE is automatically materialized (computed once, cached) — in many databases (notably PostgreSQL before v12) a non-recursive CTE referenced multiple times may be re-executed each time, which can matter for performance on expensive CTEs
- Forgetting the `UNION ALL` (not `UNION`) and a proper termination condition in a recursive CTE, causing infinite recursion
- Over-nesting CTEs so deeply that the readability benefit is lost — at some point, a temp table or view is clearer

## Interview / discussion questions

- What's the practical difference between a CTE and a subquery, in terms of both readability and (database-dependent) execution?
- Write a recursive CTE to find all descendants of a node in a hierarchy table, and explain the anchor/recursive-step structure.
- When would you prefer a CTE chain over a single deeply nested query?

## Prerequisites

[[SQL Join Types]], [[Aggregation & GROUP BY]]

## Related concepts

[[Window Functions]], [[Query Execution & Optimization]]

## Tags

#category/databases #topic/sql

## One-line summary

> A CTE names an intermediate result so a complex query can be read top-to-bottom as sequential steps instead of nested inside-out subqueries — mostly a readability tool, except in the recursive form (`WITH RECURSIVE`), which is genuinely the only way to express hierarchical/graph traversal of unknown depth in standard SQL.
