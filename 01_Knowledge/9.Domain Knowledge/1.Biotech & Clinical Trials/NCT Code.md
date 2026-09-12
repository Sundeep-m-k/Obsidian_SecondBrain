# NCT Code

## What is it?

An **NCT code** (e.g. `NCT01234567`) is the unique identifier for a trial registered on ClinicalTrials.gov — the closest thing to a universal, standardized ID in clinical trial data, far more reliable than free-text trial names.

## Why It Matters for Joining Data

NCT codes are commonly the join key linking internal pipeline records to canonical trial-level detail (design, enrollment, sponsor, dates). Applying [[Join Validation]]'s formulas directly:

$$\text{Coverage} = \frac{|\{i \in A : \text{nct\_code}(i) \neq \text{null}\}|}{|A|}, \qquad \text{Match rate} = \frac{|\{i : \text{nct\_code}(i) \in K_{\text{trials}}\}|}{|\{i : \text{nct\_code}(i) \neq \text{null}\}|}$$

## Worked Example

$|A| = 1{,}500{,}000$ pipeline rows, $1{,}434{,}000$ have a non-null NCT code → coverage $=95.6\%$. Of those, all $1{,}434{,}000$ match a row in the trials table → match rate $=100\%$. Two numbers, computed once, that convert "I assume this join works" into "I measured that it works" for the rest of the project.

## What It Does and Doesn't Guarantee

A clean NCT code guarantees you're looking at the *right* trial — it says nothing about whether every field on the matched record is populated or reliable (see [[Schema Trust]]'s population-rate check, applied per-column on the joined table). Coverage also isn't universal: the missing 4.4% likely represents very early-stage or non-US-registered activity, worth understanding rather than silently dropping.

## Common Mistakes

- Assuming 100% coverage without checking
- Treating "has a matched NCT record" as equivalent to "every downstream field on that record is trustworthy" — those are two separate checks (join validation vs. schema trust)

## Interview / discussion questions

- Why is an NCT code a more reliable join key than a free-text trial name?
- What two numbers (per [[Join Validation]]) would you compute before trusting an NCT-based join?

## Prerequisites

[[Join Validation]]

## Related concepts

[[Drug Pipeline]], [[Clinical Trial Phases]], [[Schema Trust]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> An NCT code is a standardized join key with measurable coverage and match-rate properties (95.6%/100% in a real example) — reliable as an identifier, but the completeness of what it joins to is a separate check.
