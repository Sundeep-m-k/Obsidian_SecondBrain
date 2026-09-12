# Entity Resolution (Record Linkage)

## What is it?

**Entity resolution** (record linkage) decides when two records — possibly with different IDs, spellings, or formats — refer to the same real-world entity. It comes up whenever a dataset lacks one clean, universal identifier for the thing you care about.

## Formal Framing: Tiered Confidence Keys

Define a sequence of candidate matching functions $\kappa_1, \kappa_2, \ldots, \kappa_T$ ordered from highest to lowest confidence (most to least specific/reliable). For a row $i$, resolve using the **first** tier that produces a valid key:

$$\text{tier}(i) = \min\{ t : \kappa_t(i) \neq \text{null} \}, \qquad \text{key}(i) = \kappa_{\text{tier}(i)}(i)$$

Concretely, three tiers — resolving customer identity across a CRM export and an e-commerce orders database that don't share a common primary key:

| Tier $t$ | $\kappa_t(i)$ | Confidence |
|---|---|---|
| 1 | `customer_id + account_id` (both resolved system IDs, e.g. via a prior SSO link) | High |
| 2 | `normalize(email) ` (lowercased, whitespace-stripped) | Medium |
| 3 | `normalize(full_name) + zip_code` (both free text / loosely structured) | Low — fallback |

Every row is tagged with $\text{tier}(i)$, not just $\text{key}(i)$ — the tier is itself a feature that downstream analysis can filter or weight by.

---

## Worked Example: Auditing Tier Distribution

Across 788,000 candidate customer rows: Tier 1 resolves 436,000 (55.3%), Tier 2 resolves 210,000 (26.6%), Tier 3 resolves 142,000 (18.0%). Reporting this breakdown — not just the final merged count — tells a reader that nearly 1-in-5 customers rest on the weakest, free-text-only match, which should inform how much to trust results for that slice specifically (e.g. run headline numbers on Tier 1+2 only as a robustness check).

## Why Getting the Key Wrong Contaminates Everything Downstream

If two rows that are actually the same real-world entity get assigned different keys — or two different real entities collapse onto the same key — every label, feature, and count derived from grouping by that key inherits the error. This is why validating candidate keys empirically (checking the tier distribution, spot-checking a sample per tier) matters more than picking one key and hoping.

## A Related Trap: The "Obvious" Shortcut Column

A field that looks like it should already provide the resolved link (e.g. a `master_customer_id` column that looks foreign-key-shaped) can be essentially unpopulated. Always check the actual population rate — $\frac{|\{i : \text{col}(i) \neq \text{null}\}|}{|A|}$ — before trusting a shortcut key, rather than assuming it exists just because a plausible-looking column does.

## Common Mistakes

- Rolling up by whichever ID is available without checking whether the same entity fragments across multiple keys (e.g. the same person appearing as three different `customer_id`s because they checked out as a guest twice before creating an account)
- Using a broad categorical field (e.g. `company_name` free text) as a stand-in for a properly resolved ID — it can scatter the same entity across several buckets, especially with historically inconsistent labeling ("Acme Inc." vs. "Acme, Inc." vs. "ACME")
- Trusting a shortcut column without checking its population rate

## Interview / discussion questions

- Formally define the tiered-key resolution function and explain why tagging $\text{tier}(i)$ matters as much as $\text{key}(i)$ itself.
- What's the risk of using a broad category as a stand-in for a properly resolved entity identity?
- How would you audit whether your entity-resolution scheme is silently fragmenting the same real-world thing across multiple keys?

## Prerequisites

[[Join Validation]]

## Related concepts

[[Confidence-Tiered Matching]], [[Schema Trust]], [[Spine Table Pattern]], [[Keys & Constraints]]

## Tags

#category/databases #topic/data-quality #math/set-theory

## One-line summary

> Entity resolution via tiered keys resolves each row with the highest-confidence matching function available ($\text{tier}(i) = \min\{t : \kappa_t(i) \neq \text{null}\}$), tagging every row with its tier so downstream work can measure and account for match confidence, not just accept a merged result blindly.
