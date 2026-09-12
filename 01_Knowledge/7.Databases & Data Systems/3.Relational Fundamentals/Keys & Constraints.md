---
tags: [databases, relational-fundamentals, sql]
---

# Keys & Constraints

## What is it?

A **key** is a column (or set of columns) that identifies rows in a table or links rows across tables. A **constraint** is a rule the database enforces automatically, rejecting any write that would violate it. Together they're what make a relational schema *trustworthy by construction* rather than trustworthy by hope — the database itself refuses bad data, instead of relying on every downstream query to re-check it.

## Core Concepts

| Term | Definition | Example (e-commerce schema) |
|---|---|---|
| **Primary key (PK)** | Column(s) that uniquely identify every row; cannot be null | `customers.customer_id` |
| **Candidate key** | Any column set that *could* serve as PK (unique + not null); a table may have several, only one is chosen as PK | `customers.email` (also unique) |
| **Foreign key (FK)** | A column whose values must match a primary key in another table | `orders.customer_id` → `customers.customer_id` |
| **Composite key** | A key made of more than one column | `order_items.(order_id, product_id)` |
| **Unique constraint** | Enforces no duplicate values, but (unlike PK) allows null | `customers.email UNIQUE` |
| **Not null constraint** | Column must always have a value | `orders.order_date NOT NULL` |
| **Check constraint** | Value must satisfy a boolean expression | `CHECK (quantity > 0)` |

## Formal Framing

A primary key $K = \{c_1, \ldots, c_k\}$ on table $T$ must satisfy two properties:

$$\textbf{Uniqueness: } \forall i \neq j \in T,\ (c_1(i), \ldots, c_k(i)) \neq (c_1(j), \ldots, c_k(j))$$
$$\textbf{Minimality: } \nexists K' \subsetneq K \text{ such that } K' \text{ is also unique}$$

Minimality matters: `(order_id, product_id, customer_id)` might be unique on `order_items`, but if `(order_id, product_id)` alone is already unique, the full triple isn't a *minimal* key — it's carrying a redundant column that isn't needed to distinguish rows.

A foreign key formalizes **referential integrity**: for FK column $f$ on table $A$ referencing primary key $p$ on table $B$,

$$\forall i \in A,\ f(i) = \text{null} \ \lor\ f(i) \in \{p(j) : j \in B\}$$

This is exactly the coverage/match-rate idea from [[Join Validation]] — except here it's enforced by the database at write time, not measured after the fact. A FK constraint is what makes match rate *provably* 100% by construction, rather than something you have to check.

## Worked Example: What Constraints Prevent

Without a FK constraint on `orders.customer_id`, nothing stops an application bug from inserting `orders.customer_id = 99999` when no such customer exists — an "orphan" order that will silently vanish from any inner join to `customers`, inflating the mismatch you'd otherwise have to discover via [[Join Validation]]. With the FK constraint in place, that insert is *rejected outright* — the bug surfaces immediately at write time, as a loud error, instead of quietly at read time, as a mysteriously shrinking join months later.

```sql
CREATE TABLE orders (
    order_id     INT PRIMARY KEY,
    customer_id  INT NOT NULL REFERENCES customers(customer_id),
    order_date   DATE NOT NULL,
    status       VARCHAR(20) CHECK (status IN ('pending', 'shipped', 'delivered', 'cancelled'))
);
```

## Why It Matters

Constraints push data-quality guarantees as far upstream as possible — into the schema itself — instead of leaving every analyst and every downstream pipeline to re-derive the same checks by hand. This is the schema-design counterpart to [[Schema Trust]]: a well-constrained schema needs *less* verification, because the database has already ruled out entire categories of bad data.

## Advantages and Limitations

**Advantages**: guarantees hold for every row, always, with no extra code; catches bugs at the earliest possible point (write time).

**Limitations**: constraints have a write-time performance cost (every insert/update must be checked); many real-world warehouses relax or drop FK constraints on large analytical tables for load speed, which is exactly why [[Join Validation]] and [[Schema Trust]] exist — you can't always assume the schema is enforcing what its column names imply.

## Common Mistakes

- Assuming a column *named* like a foreign key (`customer_id`) is actually constrained as one — many analytical warehouses store this relationship without enforcing it, making empirical [[Join Validation]] necessary even when a "foreign key" is documented
- Choosing a non-minimal composite key, which weakens the guarantee and confuses anyone reading the schema
- Forgetting that `UNIQUE` allows multiple nulls (in most databases), unlike a primary key

## Interview / discussion questions

- What's the difference between a primary key and a unique constraint?
- Formally define referential integrity, and explain how it relates to the coverage/match-rate formulas in [[Join Validation]].
- Why might a production data warehouse deliberately not enforce foreign key constraints on its largest tables?

## Prerequisites

Basic relational database concepts

## Related concepts

[[Normalization]], [[Join Validation]], [[Schema Trust]], [[ACID Transactions]]

## Tags

#category/databases #topic/relational-fundamentals #math/set-theory

## One-line summary

> Keys uniquely identify and link rows; constraints make the database itself reject invalid data at write time — a foreign key constraint is what turns "this join should work" from a hope into a mathematically guaranteed 100% match rate.
