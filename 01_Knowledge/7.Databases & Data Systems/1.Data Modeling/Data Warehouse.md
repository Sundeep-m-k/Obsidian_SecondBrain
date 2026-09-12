# Data Warehouse

## What is it?

A **data warehouse** is a large, centralized database designed for analysis and reporting across an organization — as opposed to an operational (OLTP) database designed for fast, small read/writes that run the day-to-day product.

## Why It Matters

Real warehouses are large (often hundreds of tables), noisy, and full of half-finished or duplicated tables. Before writing any code, the practical first step is scoping: deciding what to ignore and what to trust — treating trust as something to be *measured*, not assumed.

## Formalizing "Trust" for a Candidate Table or Column

For any candidate column $c$ in table $T$, before using it:

$$\text{population rate}(c) = \frac{|\{i \in T : c(i) \neq \text{null}\}|}{|T|}$$

A column named exactly what you want (e.g. `customer_lifetime_value`) with $\text{population rate} \approx 0$ is a textbook trap — see [[Schema Trust]] for the full checklist. This single formula — computed *before* any analysis relies on the column — turns "this looks like the right column" into a falsifiable check.

## Common Structural Patterns

| Pattern | What it means |
|---|---|
| **Spine table** | One central table at the right grain (see [[Spine Table Pattern]]) |
| **Enrichment table** | A related table joined in, validated via [[Join Validation]] |
| **Denormalized view** | Pre-joined, flattened table (see [[Denormalization]]) |
| **Staging/mirror tables** | Intermediate or duplicate copies, often safe to ignore |

## Worked Example: Scoping a 192-Table Warehouse

Not every table is worth exploring. Take a mid-size e-commerce company's warehouse: 192 tables total. A practical scoping pass: exclude auth/identity tables (`users_auth`, `sessions` — not analytically relevant), internal tooling tables (`admin_audit_log`, `feature_flags` — different domain), UI/CMS tables (`homepage_banners`, `cart_ui_state` — presentation layer, not source data), staging/migration tables (`flyway_*`, `*_staging`), and duplicate `public.*` mirrors of properly-schemed tables. This can reduce ~192 candidate tables to the ~15-20 that plausibly matter for an analysis — `customers`, `orders`, `order_items`, `products`, `payments`, `shipments`, and a handful of dimension tables — converting "explore blind" into "explore with a falsifiable hypothesis about which tables matter," verified via population rate and [[Join Validation]] on each survivor.

## Common Mistakes

- Exploring a large warehouse without first narrowing to a small set of trustworthy spine/enrichment tables
- Trusting a column because its name matches what you want, without checking population rate
- Assuming a join works without measuring coverage/match rate

## Interview / discussion questions

- Why is a data warehouse structured differently from the operational database backing a live product?
- Write the population-rate formula and explain why it should be the *first* check on any candidate column, before anything else.

## Prerequisites

Basic SQL / relational database concepts

## Related concepts

[[Spine Table Pattern]], [[Denormalization]], [[Join Validation]], [[Schema Trust]], [[OLTP vs OLAP]], [[Star & Snowflake Schema]]

## Tags

#category/databases #topic/data-modeling #math/set-theory

## One-line summary

> A data warehouse centralizes data for analysis, but scale and messiness mean every candidate table or column needs a measured population rate and join validation before being trusted — never assumed from a name or schema alone.
