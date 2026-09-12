---
tags: [databases, index, moc]
---

# Databases & Data Systems — Overview

> **Position in vault**: `01_Knowledge/7.Databases & Data Systems/`
> **Purpose**: Everything needed to reason about relational databases and data systems as a discipline in its own right — not just as project-specific war stories, but as the general toolkit for any data-based role (data engineer, analytics engineer, data analyst, backend engineer working with data).
> **Running example**: Most worked examples in this subject use the same generic e-commerce schema (`customers`, `orders`, `order_items`, `products`) so concepts stay comparable across notes instead of each pulling from a different domain.

---

## Why This Section Exists Separately from `3.ML & DL`

Databases are their own discipline — schema design, transactional correctness, and query performance are concerns that exist whether or not any modeling is happening downstream. This section is written to be useful on its own for interview prep or day-to-day data work, independent of the ML project that originally motivated it.

## Section Map

| Folder | Covers | Status |
|---|---|---|
| 1.Data Modeling | [[Data Warehouse]], [[Denormalization]], [[Spine Table Pattern]] | ✅ Built |
| 2.Data Quality & Integration | [[Join Validation]], [[Entity Resolution]], [[Confidence-Tiered Matching]], [[Schema Trust]] | ✅ Built |
| **3.Relational Fundamentals** | [[Keys & Constraints]], [[Normalization]], [[ACID Transactions]], [[Indexing]], [[SQL Join Types]] | ✅ Built |
| **4.SQL Querying & Optimization** | [[Window Functions]], [[Common Table Expressions (CTEs)]], [[Aggregation & GROUP BY]], [[Query Execution & Optimization]] | ✅ Built |
| **5.OLTP, OLAP & Warehousing Design** | [[OLTP vs OLAP]], [[Fact & Dimension Tables]], [[Star & Snowflake Schema]], [[Slowly Changing Dimensions]] | ✅ Built |
| 6. Data Pipelines & ETL/ELT | Batch vs. streaming, orchestration, idempotency, ETL vs. ELT | ⬜ Not started |
| 7. NoSQL & Distributed Data | CAP theorem, sharding/partitioning, replication, document/key-value/columnar stores | ⬜ Not started |

## Why This Structure

Modules 1-2 (originally built first) are the *applied, project-discipline* layer — how to work safely with a messy, real warehouse you didn't design. Modules 3-5 are the *foundational* layer underneath that — the relational theory, SQL mechanics, and warehouse-design patterns that 1-2 assume you already know. Modules 6-7 (roadmap) extend outward into how data actually moves and scales across systems — the part of a data engineer's job that's about pipelines and infrastructure rather than any single database.

**Recommended reading order for someone building this from scratch**: 3 (Relational Fundamentals) → 4 (SQL Querying) → 5 (Warehousing Design) → 1 (Data Modeling) → 2 (Data Quality & Integration) → 6/7 (roadmap). If you already know SQL and just want the project-discipline layer, 1 → 2 stands alone.

## Key Cross-Links to Other Subjects

| Databases Concept | Links to |
|---|---|
| Grain, join validation | [[Grouped Train-Test Split]] — both are about getting the unit of analysis right before anything downstream is trustworthy |
| Query execution cost ($O(\log n)$ lookups, join algorithm complexity) | General algorithmic complexity — same big-O reasoning as [[DSA Overview]] |
| Fact table grain | Same underlying concept as [[Spine Table Pattern]]'s grain, applied to warehouse design specifically |

## Common Exam / Interview Questions

1. Walk through 1NF → 2NF → 3NF on a concrete unnormalized table.
2. What's the difference between OLTP and OLAP, and why do they need separate schemas?
3. Explain the four ACID properties and one failure mode each one prevents.
4. Why does an unindexed foreign key commonly cause slow joins?
5. What's the difference between a star schema and a snowflake schema, and when would you choose each?
6. Walk through SCD Type 1 vs. Type 2 and explain what silently breaks if a dimension needing history is built as Type 1.
7. Given a slow query, how would you use `EXPLAIN ANALYZE` to diagnose it?

## One-line Summary

> This subject now runs from relational first principles (keys, normalization, ACID, indexing, joins) through practical SQL (window functions, CTEs, aggregation, query optimization) to warehouse design (OLTP/OLAP, star schema, SCDs), with the original applied data-quality discipline (data warehouse scoping, join validation, entity resolution, schema trust) sitting on top as the "how to work safely with a real, messy warehouse" layer — pipelines/ETL and NoSQL/distributed systems are the next phase.
