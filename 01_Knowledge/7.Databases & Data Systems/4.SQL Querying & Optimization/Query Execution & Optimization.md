---
tags: [databases, sql, performance]
---

# Query Execution & Optimization

## What is it?

Every SQL query is compiled into an **execution plan** — the concrete sequence of physical operations (scans, joins, sorts, aggregations) the database will actually perform. The **query planner/optimizer** chooses this plan automatically, but not always well; reading and reasoning about execution plans is how you find and fix a slow query instead of guessing.

## SQL's Logical vs. Physical Order

SQL is *declarative* — you state what you want, not how to get it. The written order of clauses is not the order they execute:

$$\text{Logical order: } \texttt{FROM} \to \texttt{WHERE} \to \texttt{GROUP BY} \to \texttt{HAVING} \to \texttt{SELECT} \to \texttt{ORDER BY} \to \texttt{LIMIT}$$

The planner is free to choose yet another *physical* order and strategy underneath that logical contract, as long as the result is identical — e.g. it might apply a filter before a join if that's cheaper, even though `WHERE` is logically "after" `FROM`.

## Reading an Execution Plan

```sql
EXPLAIN ANALYZE
SELECT c.name, SUM(o.amount)
FROM customers c
JOIN orders o ON o.customer_id = c.customer_id
WHERE o.order_date >= '2026-01-01'
GROUP BY c.name;
```

A plan reveals, for each step: which physical operation is used (e.g. `Seq Scan` vs. `Index Scan`), the estimated vs. actual row count, and the cost. The two things worth checking first on any slow query:

| Symptom in the plan | Likely cause | Fix |
|---|---|---|
| `Seq Scan` on a large table with a selective `WHERE` | Missing index on the filtered column | Add an index — see [[Indexing]] |
| Estimated rows wildly off from actual rows | Stale table statistics | Run `ANALYZE` to refresh the planner's statistics |
| `Nested Loop` join on two large tables | Planner chose a join strategy poor for this data size | Often self-corrects with better statistics or an index; occasionally needs a query rewrite |
| Large `Sort` step before a `GROUP BY`/`ORDER BY` | No index supports the required order | An index on the sort/group column can let the database skip the explicit sort |

## Join Algorithms — Why the Planner's Choice Matters

| Algorithm | Cost | Best when |
|---|---|---|
| **Nested Loop Join** | $O(\lvert A\rvert \times \lvert B\rvert)$ without an index; $O(\lvert A\rvert \log \lvert B\rvert)$ with one on $B$ | One side is small, or an index supports the lookup |
| **Hash Join** | $O(\lvert A\rvert + \lvert B\rvert)$ (build a hash table on the smaller side, probe with the larger) | Both sides are large, no useful index, equality join |
| **Merge Join** | $O(\lvert A\rvert \log \lvert A\rvert + \lvert B\rvert \log \lvert B\rvert)$ (or $O(\lvert A\rvert+\lvert B\rvert)$ if both already sorted) | Both inputs are sorted (or cheaply sortable) on the join key |

A planner choosing Nested Loop on two million-row tables with no index is the single most common cause of an unexpectedly slow join — this is directly why [[Indexing]] matters for join performance, not just for `WHERE` filters.

## Worked Example: Diagnosing a Slow Query

A dashboard query filtering `orders WHERE status = 'pending'` takes 40 seconds on a 20-million-row table. `EXPLAIN ANALYZE` shows a `Seq Scan` with `rows removed by filter: 19,850,000` — the database read all 20 million rows just to keep 150,000. Adding `CREATE INDEX idx_orders_status ON orders(status);` lets the planner switch to an `Index Scan`, cutting the query to under a second — but only if `status` is selective enough (low cardinality columns like a boolean with a 50/50 split often don't benefit much, since the planner may reasonably decide a scan is still cheaper than jumping around via an index).

## Why It Matters

Most real-world "the database is slow" problems are query-plan problems, not hardware problems — throwing more compute at a query doing an accidental full scan on every request treats the symptom, not the cause. Being able to read `EXPLAIN` output is the difference between guessing and diagnosing.

## Common Mistakes

- Adding indexes reactively without ever checking the execution plan first, sometimes adding an index that the planner won't even use
- Assuming query performance problems are always about missing indexes — sometimes it's stale statistics, a poor join order, or a query shape that can't use an index at all (e.g. a filter wrapped in a function, like `WHERE UPPER(name) = 'ALICE'`, which usually can't use a plain index on `name`)
- Optimizing a query in isolation without checking whether it's run once a day (optimization may not be worth the effort) or a thousand times a second (small inefficiencies compound massively)

## Advantages and Limitations

Understanding execution plans doesn't replace good schema design (indexing, normalization) — it's the diagnostic tool that tells you *which* schema-level fix (from [[Indexing]], [[Normalization]], or [[Denormalization]]) actually applies to a specific slow query, rather than guessing.

## Interview / discussion questions

- What's the difference between SQL's logical clause order and a query's physical execution order?
- Compare nested loop, hash, and merge joins — when would a planner choose each one?
- How would you diagnose why a specific query is slow, step by step, using `EXPLAIN ANALYZE`?
- Why might wrapping a column in a function inside `WHERE` prevent the planner from using an index on that column?

## Prerequisites

[[Indexing]], [[SQL Join Types]]

## Related concepts

[[Indexing]], [[Aggregation & GROUP BY]], [[OLTP vs OLAP]]

## Tags

#category/databases #topic/sql #topic/performance #math/complexity

## One-line summary

> SQL is declarative — the planner chooses the actual physical execution plan (scan type, join algorithm, sort strategy) — and `EXPLAIN ANALYZE` is how you see and diagnose that choice directly, turning "the query is slow" from a guess into a specific, fixable bottleneck (usually a missing index, stale statistics, or a poor join algorithm choice on unindexed large tables).
