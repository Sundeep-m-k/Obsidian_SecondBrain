---
tags: [databases, relational-fundamentals, transactions]
---

# ACID Transactions

## What is it?

A **transaction** is a group of one or more database operations executed as a single, indivisible unit — either all of it happens, or none of it does. **ACID** is the set of four guarantees a database makes about every transaction: **Atomicity, Consistency, Isolation, Durability**.

## Core Concepts

| Property | Guarantee | What breaks without it |
|---|---|---|
| **Atomicity** | A transaction's operations all succeed, or all roll back — no partial completion | Money leaves one account but never arrives in the other, because the process crashed between the two writes |
| **Consistency** | A transaction can only move the database from one valid state to another (constraints, e.g. from [[Keys & Constraints]], always hold) | A transaction leaves an order referencing a customer_id that doesn't exist |
| **Isolation** | Concurrent transactions don't see each other's uncommitted intermediate state | Two transactions read the same inventory count, both decide there's stock, both sell the last unit |
| **Durability** | Once committed, a transaction's effects survive a crash | A confirmed order silently disappears after a server restart |

## Worked Example: Atomicity in a Purchase

Placing an order touches at least two tables: insert a row into `orders`, and decrement `inventory.quantity` for each purchased item.

```sql
BEGIN TRANSACTION;

INSERT INTO orders (order_id, customer_id, order_date)
VALUES (10432, 501, CURRENT_DATE);

UPDATE inventory
SET quantity = quantity - 1
WHERE product_id = 88 AND quantity >= 1;

-- if the UPDATE affected 0 rows (out of stock), the application rolls back:
-- ROLLBACK;
-- otherwise:
COMMIT;
```

Without wrapping both statements in one transaction, a crash between the `INSERT` and the `UPDATE` would record an order for an item whose inventory was never decremented — an inconsistent state that atomicity makes structurally impossible: either both statements commit, or neither does.

## Isolation Levels — the Formal Tradeoff

Full isolation (serializable — transactions behave as if run one at a time) is expensive; databases offer weaker levels that trade correctness guarantees for concurrency/speed:

| Level | Prevents | Still allows |
|---|---|---|
| **Read Uncommitted** | Nothing | Dirty reads (seeing another transaction's uncommitted writes) |
| **Read Committed** | Dirty reads | Non-repeatable reads (same query, run twice in one transaction, returns different results) |
| **Repeatable Read** | Dirty + non-repeatable reads | Phantom reads (a second query returns *new* rows that didn't exist on the first read) |
| **Serializable** | All of the above | — (behaves as if transactions ran strictly one at a time) |

Most production OLTP systems default to Read Committed — full serializability is rarely worth its throughput cost, and application logic (e.g. `WHERE quantity >= 1` above) often compensates for the gap.

## Why It Matters

ACID is what lets a database be treated as a source of truth under concurrent, potentially failing conditions — which is the actual environment every production system runs in (multiple users, network failures, server crashes, mid-write power loss). Understanding it is prerequisite to understanding why OLTP systems (built for ACID correctness on small, frequent writes) are architected so differently from OLAP/warehouse systems (built for fast reads over huge volumes, often relaxing some ACID guarantees for speed) — see [[OLTP vs OLAP]].

## Common Mistakes

- Assuming "the query didn't error" means the transaction is safe from concurrency bugs — isolation-level gaps (e.g. two transactions racing on the same inventory row under Read Committed) can cause real bugs that never throw an error
- Wrapping unrelated operations into one giant transaction, which increases lock contention and reduces concurrency for no correctness benefit
- Forgetting that a long-running uncommitted transaction can block other transactions (via locks), degrading system throughput even if it eventually commits successfully

## Advantages and Limitations

**Advantages**: makes reasoning about correctness under concurrency and failure tractable; the four guarantees compose, so application code doesn't need to hand-roll its own crash/concurrency recovery logic.

**Limitations**: full ACID (especially serializable isolation) has a real throughput cost, which is why distributed/NoSQL systems often relax it deliberately (see the eventual-consistency tradeoff, a future roadmap item in this subject) in exchange for horizontal scalability.

## Interview / discussion questions

- Define all four ACID properties and give a concrete failure scenario each one prevents.
- What's the difference between a dirty read, a non-repeatable read, and a phantom read?
- Why might a production system deliberately choose Read Committed over Serializable isolation?
- How does atomicity guarantee that a multi-table update either fully succeeds or leaves no trace?

## Prerequisites

[[Keys & Constraints]]

## Related concepts

[[OLTP vs OLAP]], [[Keys & Constraints]]

## Tags

#category/databases #topic/relational-fundamentals #topic/transactions

## One-line summary

> ACID (Atomicity, Consistency, Isolation, Durability) is the set of guarantees that make a transaction behave as one indivisible, crash-safe, concurrency-safe unit — isolation levels formalize exactly how much of that guarantee a system is willing to trade for throughput, with Read Committed as the common real-world default.
