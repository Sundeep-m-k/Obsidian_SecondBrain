# Molecule

## What is it?

A **molecule** (compound, drug candidate) is the actual chemical/biological entity being developed — distinct from the **program**, which is that molecule paired with a specific indication (see [[Drug Pipeline]]: $\text{program} = (\text{molecule\_id}, \text{indication\_id})$). The same molecule can be developed for multiple indications, each a separate program.

## Why Molecule Identity Is the Correct Grouping Key for Splitting

Recall [[Grouped Train-Test Split]]'s formalization: $\text{split}(i) = \pi_{\text{group}}(g(i))$. Here, $g(i) = \text{molecule\_id}(i)$ is the right choice of $g$ — every trial and program tied to the same molecule must land in the same split, otherwise the model can partially "see" a molecule during training and get an inflated score re-encountering it in test. Using a *finer* group (e.g. per-trial) or *coarser* group (e.g. per-TA) as $g$ would either under-protect against leakage (finer) or throw away far more data than necessary (coarser).

## The Naming Problem

Molecules are often free text (brand names, generic names, internal codes, chemical names) that vary across sources — the same molecule can appear under multiple spellings. Resolving this reliably uses the same tiered-key machinery as [[Entity Resolution]]: $\text{tier}(i) = \min\{t : \kappa_t(i) \neq \text{null}\}$, with $\kappa_1$ = resolved `molecule_id`, $\kappa_2$ = normalized name matching, etc.

## Common Mistakes

- Splitting train/test by row instead of by molecule, letting the same molecule leak across both splits
- Assuming molecule name strings are already standardized enough to group by directly
- Conflating "same molecule, different indication" (legitimately separate programs) with "same molecule, same indication, recorded inconsistently" (one program, fragmented — see [[Indication vs Therapeutic Area]])

## Interview / discussion questions

- Why must $g(i)$ in a grouped train/test split be molecule, not row or therapeutic area — walk through what goes wrong with each alternative.
- What real-world scenario does "same molecule, multiple indications" represent, and why should those count as separate programs?

## Prerequisites

None — foundational domain vocabulary

## Related concepts

[[Drug Pipeline]], [[Entity Resolution]], [[Grouped Train-Test Split]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> A molecule is the chemical/biological entity itself — half of a program's identity, and the correct grouping key $g(i)$ for train/test splitting, since a coarser or finer choice either under-protects against leakage or discards data unnecessarily.
