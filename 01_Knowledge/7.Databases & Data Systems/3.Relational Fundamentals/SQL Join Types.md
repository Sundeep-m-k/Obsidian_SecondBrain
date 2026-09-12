---
tags: [databases, relational-fundamentals, sql]
---

# SQL Join Types

## What is it?

A **join** combines rows from two or more tables based on a related column. The join *type* determines what happens to rows on either side that **don't** find a match — which is the part that actually matters for correctness, since matching rows behave identically under every join type.

## Core Concepts, Worked on One Example

```sql
customers(customer_id, name)
orders(order_id, customer_id, amount)
```
Suppose `customers` has 5 rows (`customer_id` 1–5), and `orders` has orders for customers 1, 2, 2, 3 — note customer 4 and 5 have never ordered, and there's no `customer_id = 6` in `customers` at all (orders is clean here, but imagine a data-quality issue where one exists).

| Join type | SQL | Keeps | Result here |
|---|---|---|---|
| **INNER JOIN** | `FROM customers c JOIN orders o ON c.customer_id = o.customer_id` | Only rows with a match on both sides | 4 rows (customers 1, 2, 2, 3) |
| **LEFT JOIN** | `FROM customers c LEFT JOIN orders o ON ...` | All left rows, matched or not (nulls for unmatched right columns) | 6 rows (adds customers 4, 5 with null order columns) |
| **RIGHT JOIN** | `FROM customers c RIGHT JOIN orders o ON ...` | All right rows, matched or not | Same as INNER here (every order has a valid customer) |
| **FULL OUTER JOIN** | `FROM customers c FULL OUTER JOIN orders o ON ...` | All rows from both sides, matched or not | 6 rows (union of LEFT and RIGHT behavior) |
| **CROSS JOIN** | `FROM customers CROSS JOIN orders` | Every combination of every row (Cartesian product) | 5 × 4 = 20 rows — almost never what you actually want |

## Formal Definition

For tables $A$ and $B$ with join condition $\theta$ (e.g. $A.k = B.k$):

$$A \bowtie_\theta B = \{(a, b) : a \in A, b \in B, \theta(a, b) \text{ true}\} \quad \text{(inner join)}$$
$$A \mathbin{⟕}_\theta B = (A \bowtie_\theta B) \cup \{(a, \text{null}) : a \in A, \nexists b \in B \text{ s.t. } \theta(a,b)\} \quad \text{(left join)}$$

Full outer join is the union of left and right joins; cross join is $\theta \equiv \text{true}$ for every pair — every row of $A$ paired with every row of $B$, with $|A \times B| = |A| \cdot |B|$ result rows.

## Why "Which Join?" Is a Correctness Question, Not a Style Choice

Picking INNER JOIN when you meant LEFT JOIN silently drops rows with no match — e.g. "average order amount per customer" computed with an INNER JOIN silently excludes customers who've never ordered, which might be exactly the population a churn or engagement analysis needs to see (as zero, not as absent). This connects directly to [[Join Validation]]: an inner join's match rate tells you exactly how many rows a LEFT JOIN would have kept that the INNER JOIN silently dropped.

## Common Mistakes

- Defaulting to INNER JOIN out of habit when the analysis actually needs to represent "no match" as a real value (e.g. zero orders), not as row absence
- Using a LEFT JOIN and then filtering on a right-table column with a plain `WHERE` clause (`WHERE o.amount > 100`) — this silently turns the LEFT JOIN back into an INNER JOIN, because unmatched rows have `o.amount = NULL`, and `NULL > 100` is neither true nor false, so the row is dropped. The filter belongs in the `ON` clause instead if unmatched rows should be kept.
- Writing an accidental CROSS JOIN by forgetting a join condition, silently multiplying row counts — this is the SQL-level version of the row-count blowup warned about in [[Join Validation]]

## Advantages and Limitations

Each join type is a precise tool for a precise question — "what should happen to non-matches?" — so there's no universally "correct" choice; the mistake is picking one without deliberately asking that question.

## Interview / discussion questions

- Write the set-builder definition of an inner join and a left join, and explain the difference precisely.
- Why does putting a right-table filter in `WHERE` instead of `ON` silently convert a LEFT JOIN into an INNER JOIN?
- How does the row-count sanity check from [[Join Validation]] relate to choosing the correct join type?
- When would a CROSS JOIN be intentional rather than a bug?

## Prerequisites

[[Keys & Constraints]]

## Related concepts

[[Join Validation]], [[Keys & Constraints]], [[Normalization]]

## Tags

#category/databases #topic/relational-fundamentals #math/set-theory

## One-line summary

> Every join type agrees on matching rows and disagrees only on what happens to non-matches — INNER drops them, LEFT/RIGHT/FULL preserve them as nulls on one or both sides, and CROSS ignores match conditions entirely — so choosing the right one is a correctness decision, not a style preference, and filtering a LEFT JOIN's right-table columns in `WHERE` instead of `ON` is the single most common way to accidentally undo it.
