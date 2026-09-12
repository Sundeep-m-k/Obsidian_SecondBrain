# Join Validation

## What is it?

**Join validation** is the practice of empirically measuring whether a join between two tables actually works as assumed, before building any downstream logic on top of it.

## Formal Definition

Let $A$ be the base ("left") table with join key $k_A(i)$ for row $i$, and $B$ be the table being joined in, with key set $K_B = \{k_B(j) : j \in B\}$.

$$\text{Coverage} = \frac{|\{i \in A : k_A(i) \neq \text{null}\}|}{|A|}, \qquad \text{Match rate} = \frac{|\{i \in A : k_A(i) \in K_B\}|}{|\{i \in A : k_A(i) \neq \text{null}\}|}$$

A trustworthy join needs both numbers reported, not just "the query ran." A join can execute cleanly (no SQL error) while coverage or match rate is arbitrarily low.

## Worked Example

Spine table `orders` has $|A| = 1{,}500{,}000$ rows, and you're joining in `customers` on `customer_id`. Of these, $1{,}434{,}000$ orders have a non-null `customer_id` → coverage $= 1{,}434{,}000/1{,}500{,}000 = 0.956$ (95.6%). Of those, all $1{,}434{,}000$ find a match in the `customers` table → match rate $=1{,}434{,}000/1{,}434{,}000 = 1.00$ (100%). Reported together: "95.6% of orders have a customer ID, and of those, 100% match a known customer" — a single sentence that converts an assumption into a measured fact.

Contrast with a bad case: coverage $=0.98$ but match rate $=0.40$ — 98% of rows *have* a key, but 60% of those keys don't resolve to anything in the target table. This would silently drop 60% of otherwise-eligible rows from any inner join, without a single error message.

---

## Row-Count Sanity Check

After an inner join $A \bowtie B$, always compare $|A \bowtie B|$ to $|A|$:

- $|A \bowtie B| < |A|$ shrinkage: expected if match rate $< 1$, but the *magnitude* should match the measured match rate — a bigger-than-expected drop signals something's wrong beyond simple non-matches.
- $|A \bowtie B| > |A|$ growth: signals an unintended one-to-many relationship on the $B$ side — the join key isn't unique in $B$, and rows are being duplicated.

## Why It Matters

This is a "check once, then stop worrying" pattern — cheap insurance against silently losing or corrupting large chunks of data, and against unnoticed row duplication that would inflate downstream counts and inflate apparent sample sizes used in things like [[Baseline Estimator]]'s minimum-sample threshold.

## Common Mistakes

- Treating "the join ran without error" as evidence it's correct
- Not comparing $|A \bowtie B|$ to $|A|$ before/after — the single fastest sanity check available
- Assuming a documented foreign-key relationship is actually enforced/populated in practice

## Interview / discussion questions

- Write the formulas for coverage and match rate and explain why both are needed, not just one.
- What does it mean if a join causes row count to unexpectedly *increase*, and how would you investigate it?

## Prerequisites

Basic SQL joins

## Related concepts

[[Spine Table Pattern]], [[Schema Trust]], [[Entity Resolution]], [[SQL Join Types]], [[Keys & Constraints]]

## Tags

#category/databases #topic/data-quality #math/set-theory

## One-line summary

> Join validation means computing coverage ($k_A$ populated) and match rate ($k_A \in K_B$) explicitly, plus a row-count sanity check before/after — a join that "runs" is not the same as a join that's correct.
