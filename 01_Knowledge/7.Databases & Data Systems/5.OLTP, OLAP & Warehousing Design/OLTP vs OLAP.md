---
tags: [databases, warehousing, oltp, olap]
---

# OLTP vs OLAP

## What is it?

**OLTP** (Online Transaction Processing) and **OLAP** (Online Analytical Processing) are two fundamentally different workload patterns that push database design in opposite directions. Confusing which one you're building for is the root cause of a large fraction of real-world "why is this so slow" and "why is this so hard to keep consistent" problems.

## Core Comparison

| | OLTP | OLAP |
|---|---|---|
| **Purpose** | Run the live application (place an order, update a cart) | Analyze historical data (revenue trends, cohort behavior) |
| **Query shape** | Many small reads/writes, touching few rows | Few large reads, scanning/aggregating millions of rows |
| **Schema** | Normalized (see [[Normalization]]) — minimizes write anomalies | Denormalized (see [[Denormalization]]) / star schema — minimizes read/join cost |
| **Concurrency** | High — thousands of simultaneous users | Low — a handful of analysts/dashboards |
| **Consistency needs** | Strong, immediate (ACID, see [[ACID Transactions]]) | Often eventual (data refreshed hourly/daily is fine) |
| **Typical latency target** | Milliseconds | Seconds to minutes is often acceptable |
| **Example system** | Application's Postgres/MySQL instance | Snowflake, BigQuery, Redshift |

## Why the Same Schema Can't Serve Both Well

An OLTP schema optimized for `UPDATE orders SET status = 'shipped' WHERE order_id = 123` (touch one row, fast, safe under concurrency) is a bad shape for `SUM(amount) GROUP BY month, region` across 500 million historical rows (touches nearly everything, wants columnar scanning, doesn't care about single-row update speed). Running heavy analytical queries directly against a production OLTP database risks locking contention that slows down the live application — a classic operational incident, not just a performance nuisance.

## The Standard Solution: Separate Systems, Connected by a Pipeline

$$\text{OLTP database} \xrightarrow{\text{ETL/ELT}} \text{Data warehouse (OLAP)}$$

Production writes happen against the OLTP system; a pipeline periodically extracts, transforms, and loads that data into a separate OLAP-optimized [[Data Warehouse]], purpose-built for analytical queries without risking the live application. This is *why* data warehouses exist as separate systems rather than just running analytics against production — see the ETL/ELT roadmap item in this subject for how that pipeline itself works.

## Row-Oriented vs. Column-Oriented Storage

This is the physical storage difference that makes OLAP systems fast at aggregation:

- **Row-oriented** (typical OLTP): each row stored contiguously on disk — fast to fetch/update *one entire row*, which matches OLTP's access pattern
- **Column-oriented** (typical OLAP, e.g. Snowflake/BigQuery/Redshift): each *column* stored contiguously — a query like `SUM(amount)` only reads the `amount` column off disk, ignoring every other column entirely, which is exactly what an aggregation over millions of rows needs

$$\text{Row-store cost for } \texttt{SUM(amount)} \propto n \times (\text{all columns}) \qquad \text{Column-store cost} \propto n \times (\text{one column})$$

For a wide table (50+ columns), column storage can be an order of magnitude cheaper for a query that only touches 2-3 of them.

## Why It Matters

Every schema and indexing decision in this subject — [[Normalization]] vs. [[Denormalization]], transactional [[ACID Transactions]] guarantees, [[Indexing]] choices — is downstream of first answering "is this table serving OLTP or OLAP workloads?" Getting this wrong (e.g. heavily normalizing a reporting table, or running dashboards against the live production database) is a common, expensive mistake in real systems.

## Common Mistakes

- Running heavy analytical queries directly against the production OLTP database, risking lock contention with live traffic
- Normalizing a warehouse table as aggressively as an OLTP table, paying unnecessary join cost on every analytical query
- Assuming OLAP data needs to be as fresh as OLTP data — most analytical use cases tolerate hourly or daily refresh, which is a deliberate, load-bearing design relaxation, not a shortcut

## Interview / discussion questions

- What are the core differences between OLTP and OLAP workloads, and why does each demand a different schema design?
- Why is column-oriented storage well suited to OLAP, and row-oriented storage well suited to OLTP?
- Why do most production systems keep OLTP and OLAP as physically separate databases rather than one system serving both?

## Prerequisites

[[Normalization]], [[Denormalization]], [[ACID Transactions]]

## Related concepts

[[Data Warehouse]], [[Star & Snowflake Schema]], [[Indexing]]

## Tags

#category/databases #topic/warehousing #topic/architecture

## One-line summary

> OLTP (normalized, row-oriented, ACID-strict, optimized for many small concurrent writes) and OLAP (denormalized, column-oriented, eventually-consistent, optimized for large aggregating reads) are opposite design targets — which is why production applications and analytical warehouses are almost always physically separate systems, connected by a pipeline rather than sharing one schema.
