---
tags: [databases, relational-fundamentals, sql]
---

# Normalization

## What is it?

**Normalization** is the process of structuring a relational schema to eliminate redundancy and update anomalies, by progressively enforcing a series of rules called **normal forms**. Each higher normal form fixes a specific category of problem that the previous one still allows.

## Why It Matters

An unnormalized schema doesn't just waste storage — it creates **anomalies**: ways the same fact can end up in two places and go out of sync. Normalization is the formal, teachable procedure for avoiding that, and every real schema is a deliberate tradeoff along the normalization spectrum (see [[Denormalization]] for why you'd deliberately undo some of this for analytics).

## The Normal Forms, Worked on One Example

Start from a single, unnormalized `orders` table that mixes everything together:

| order_id | customer_name | customer_email | product_name | product_price | quantity |
|---|---|---|---|---|---|
| 1 | Alice Chen | alice@x.com | Widget A | 9.99 | 2 |
| 1 | Alice Chen | alice@x.com | Widget B | 14.99 | 1 |
| 2 | Bob Ruiz | bob@x.com | Widget A | 9.99 | 3 |

### 1NF — First Normal Form
**Rule**: every column holds a single, atomic value; no repeating groups or comma-separated lists in one cell.
The table above is already 1NF (one product per row) — but if instead a single row had `products = "Widget A, Widget B"` in one cell, that would violate 1NF. Fix: one row per (order, product) pair, as shown.

### 2NF — Second Normal Form
**Rule**: 1NF, plus every non-key column must depend on the *entire* primary key, not just part of it (only relevant when the key is composite).
Here the natural key is `(order_id, product_name)`. But `customer_name` and `customer_email` depend only on `order_id`, not on `product_name` — a **partial dependency**. This is why Alice's name and email are redundantly repeated across her two order lines. Fix: split into `orders(order_id, customer_id)` and a separate `customers(customer_id, customer_name, customer_email)` table.

### 3NF — Third Normal Form
**Rule**: 2NF, plus no non-key column depends on another non-key column (**transitive dependency**).
Suppose the table also had `product_category` and `category_discount_rate` — the discount rate depends on `product_category`, not directly on the row's key. That's transitive: `key → product_category → category_discount_rate`. Fix: split into a separate `categories(category, discount_rate)` table.

### BCNF — Boyce-Codd Normal Form
**Rule**: a stricter version of 3NF — for every functional dependency $X \to Y$, $X$ must be a superkey (a key or a superset of one). Handles edge cases 3NF misses when a table has multiple overlapping candidate keys.

## Formal Definition: Functional Dependency

$X \to Y$ ("$X$ functionally determines $Y$") means: for any two rows $i, j$, if $X(i) = X(j)$, then $Y(i) = Y(j)$ — knowing $X$ always tells you $Y$. Normalization is fundamentally about identifying every functional dependency in a table and making sure each one is represented by exactly one table, keyed on its determinant.

## The Result: Fully Normalized Schema

```sql
customers(customer_id PK, name, email)
categories(category PK, discount_rate)
products(product_id PK, name, price, category FK → categories.category)
orders(order_id PK, customer_id FK → customers.customer_id, order_date)
order_items(order_id FK, product_id FK, quantity, PRIMARY KEY(order_id, product_id))
```

Now Alice's email lives in exactly one row. Change it once, and every order automatically reflects the update — no risk of one order line showing the old email and another showing the new one.

## Why It Matters (Update Anomalies, Named)

| Anomaly | What happens in the unnormalized table |
|---|---|
| **Update anomaly** | Alice changes her email → must update it in every one of her order rows, or the data goes inconsistent |
| **Insertion anomaly** | Can't add a new customer until they place an order, since customer data only exists inside order rows |
| **Deletion anomaly** | Deleting Bob's only order deletes all record that Bob exists as a customer |

Normalization eliminates all three by giving every fact exactly one home.

## Common Mistakes

- Normalizing analytical/reporting tables as aggressively as transactional ones — see [[Denormalization]] for why analytics tables often deliberately trade this away for read speed
- Confusing 2NF's partial dependency (only matters with a composite key) with 3NF's transitive dependency (matters even with a single-column key)
- Assuming "normalized" means "no redundancy at all" — normalization removes redundancy of *facts*, not all repetition (e.g. a FK value legitimately repeats once per referencing row, and that's fine)

## Advantages and Limitations

**Advantages**: eliminates update/insertion/deletion anomalies; minimizes storage of redundant facts; makes the schema's dependencies self-documenting.

**Limitations**: more tables means more joins to reconstruct a full picture, which costs read performance — the exact tradeoff [[Denormalization]] exists to address for read-heavy analytical workloads.

## Interview / discussion questions

- Define 1NF, 2NF, and 3NF, and give an example of a table that violates each one specifically.
- What is a functional dependency, and how does it formally define normalization?
- Name the three update anomalies normalization prevents, with a concrete example of each.
- Why would a data warehouse deliberately denormalize a schema that's already in 3NF?

## Prerequisites

[[Keys & Constraints]]

## Related concepts

[[Denormalization]], [[Keys & Constraints]], [[Star & Snowflake Schema]]

## Tags

#category/databases #topic/relational-fundamentals #math/set-theory

## One-line summary

> Normalization progressively removes functional-dependency redundancy (1NF → 2NF → 3NF → BCNF) so every fact lives in exactly one place, eliminating update/insertion/deletion anomalies — at the cost of needing more joins to reassemble a full picture, which is precisely what [[Denormalization]] trades back for analytical read speed.
