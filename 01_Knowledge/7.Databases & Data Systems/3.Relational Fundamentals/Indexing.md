---
tags: [databases, relational-fundamentals, performance]
---

# Indexing

## What is it?

An **index** is an auxiliary data structure that lets the database find rows matching a condition without scanning every row in a table — the database equivalent of a book's index letting you find a topic without reading every page.

## Why It Matters

Without an index, any filter (`WHERE`) or join condition requires a **full table scan**: $O(n)$ in the number of rows. A well-chosen index turns that into $O(\log n)$ lookup cost or better. On a 50-million-row `orders` table, that's the difference between a query returning in milliseconds versus minutes — and it's the single highest-leverage lever most engineers have over query performance.

## Core Concepts

Most relational databases implement indexes as a **B-tree** (balanced tree) by default:

$$\text{Lookup cost (B-tree)} = O(\log_b n)$$

where $n$ is the number of rows and $b$ is the tree's branching factor (typically large, so in practice this is a handful of disk reads even for hundreds of millions of rows).

| Index type | Best for | Cost |
|---|---|---|
| **B-tree** (default) | Equality and range queries (`=`, `<`, `BETWEEN`, `ORDER BY`) | $O(\log n)$ lookup, extra write cost |
| **Hash index** | Equality only (`=`), not ranges | $O(1)$ lookup, no range support |
| **Composite index** | Queries filtering on multiple columns together | Only helps if query uses columns in the index's *prefix* order |
| **Covering index** | Query needs only indexed columns — avoids touching the table at all | Fastest possible read for that query shape |

## Worked Example: Composite Index Column Order Matters

An index on `(customer_id, order_date)` speeds up:
```sql
SELECT * FROM orders WHERE customer_id = 501 AND order_date > '2026-01-01';  -- fast
SELECT * FROM orders WHERE customer_id = 501;                                 -- fast (prefix match)
```
but does **not** meaningfully speed up:
```sql
SELECT * FROM orders WHERE order_date > '2026-01-01';  -- slow: order_date isn't the index's leading column
```
The rule: a composite index on $(c_1, c_2, \ldots, c_k)$ only accelerates queries that filter on a *prefix* of that column list, starting from $c_1$. This is a direct, practical consequence of how a B-tree is physically sorted — it's ordered first by $c_1$, then $c_2$ within each $c_1$ value, and so on.

## The Write-Cost Tradeoff

Every index must be updated on every `INSERT`, `UPDATE`, or `DELETE` that touches an indexed column — so indexes trade write speed and storage for read speed:

$$\text{Total index maintenance cost} \propto (\text{number of indexes}) \times (\text{write volume})$$

A table with 10 indexes pays that update cost 10 times over on every write. This is why OLTP tables (frequent writes, see [[OLTP vs OLAP]]) tend to be indexed sparingly and precisely, while OLAP/warehouse tables (infrequent bulk loads, heavy reads) can afford many more indexes — the write cost is paid rarely, in large batches, while the read benefit is realized constantly.

## Why It Matters

Indexing is the concrete mechanism behind [[Join Validation]]'s implicit assumption that a join "should" be fast: a join on an unindexed foreign key forces a full scan of one side for every row of the other — often the actual root cause when a query that "should" be instant instead takes minutes.

## Common Mistakes

- Indexing every column "just in case" — each index has a real write-cost and storage cost, and unused indexes are pure overhead
- Not matching composite index column order to actual query patterns — the most selective, most commonly filtered column should usually lead
- Assuming an index exists on a foreign key just because the FK constraint exists — many databases do **not** automatically index FK columns, and an unindexed FK is a common, silent source of slow joins

## Advantages and Limitations

**Advantages**: turns $O(n)$ scans into $O(\log n)$ lookups; the single most effective query-performance lever available without changing application logic.

**Limitations**: slows down writes; consumes additional storage; a poorly chosen index (wrong column order, or indexing a low-cardinality column like a boolean flag) provides little to no benefit while still paying the write cost.

## Interview / discussion questions

- Why does a B-tree index give $O(\log n)$ lookup cost, and what does that mean concretely at 50 million rows vs. 500 million?
- Explain why a composite index on $(a, b)$ doesn't help a query that filters only on $b$.
- What's the tradeoff a database makes by adding an index, and when would you deliberately choose *not* to add one?
- Why might an unindexed foreign key be a common, silent source of slow joins?

## Prerequisites

[[Keys & Constraints]]

## Related concepts

[[Join Validation]], [[OLTP vs OLAP]], [[Query Execution & Optimization]]

## Tags

#category/databases #topic/relational-fundamentals #topic/performance #math/complexity

## One-line summary

> An index turns an $O(n)$ table scan into an $O(\log n)$ lookup by maintaining a sorted structure (typically a B-tree) alongside the table, at the cost of extra write overhead — and a composite index only helps queries that filter on a *prefix* of its column order, which is the single most common reason an "obviously indexed" query is still slow.
