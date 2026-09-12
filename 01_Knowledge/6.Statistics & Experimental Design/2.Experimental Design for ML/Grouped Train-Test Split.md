# Grouped Train-Test Split

## What is it?

A **grouped train/test split** ensures every row sharing a group identifier (same molecule, same patient, same customer) lands entirely in one split — never spread across train and test.

## Formal Framing

Let $g(i)$ map row $i$ to its group (e.g. molecule ID). A standard random split assigns a split label independently per row:

$$\pi_{\text{row}}: i \mapsto \{\text{train, val, test}\}, \quad \text{i.i.d. per } i$$

A grouped split instead assigns the split at the group level and propagates it to every member:

$$\pi_{\text{group}}: g \mapsto \{\text{train, val, test}\}, \qquad \text{split}(i) = \pi_{\text{group}}(g(i))$$

This guarantees $\{i : g(i) = g_0\} \subseteq \text{one split}$ for every group $g_0$ — no group is ever split across train and test.

---

## Why Plain Random Splitting Fails

$\pi_{\text{row}}$ assumes rows are exchangeable — that seeing row $i$ in training tells you nothing extra about row $j \neq i$ in test. But if $g(i) = g(j)$ (same molecule, different trial), the model can partially memorize molecule-specific quirks from row $i$ during training, then look artificially accurate on row $j$ in test — not because it generalizes, but because it's already seen a near-duplicate. This is [[Data Leakage]] induced purely by the split procedure, with no individual feature at fault.

## Worked Example

Molecule M has 4 trial-phase rows: Phase 1 (2019), Phase 1/2 (2020), Phase 2 (2021), Phase 2/3 (2022). Under $\pi_{\text{row}}$, a coin flip per row might place the 2019 and 2021 rows in train and the 2020, 2022 rows in test. The model then sees "Molecule M, early phase, succeeded" during training and is asked to predict "Molecule M, later phase" in test — leaking molecule-specific signal (sponsor identity, therapeutic area, historical success pattern) that a genuinely new molecule wouldn't provide. Under $\pi_{\text{group}}$, all 4 rows for Molecule M go to the same split, every time.

---

## How It Works in Practice

1. Identify the grouping key $g$ — the real-world entity that must not be split (molecule, patient, user).
2. Split at the group level: assign whole groups to train/val/test (e.g. ~60/20/20 by group count, not row count — check both, since groups can have very different row counts).
3. Verify: $\forall g_0,\ |\{s : \text{split}(i)=s,\ g(i)=g_0\}| \leq 1$ (each group appears in exactly one split).

## Why It Matters

This is one of the two evaluation safeguards (alongside [[Rolling-Origin Validation]]) required before trusting that a model honestly beats a baseline — without it, validation/test metrics measure "how well does the model recognize entities it's already partly seen," not genuine generalization.

## Common Mistakes

- Splitting by row when the true unit of leakage is a group — the single most common way ML projects unknowingly overstate accuracy
- Choosing the wrong grouping key (e.g. splitting by indication when the real risk is at the molecule level)
- Splitting by group count and getting an unbalanced split by row count (a few huge groups can dominate one split) — worth checking both

## Interview / discussion questions

- Formally, why can test accuracy be inflated by a bad split even when no individual feature leaks the label?
- How do you choose the grouping key for a grouped split?
- What's the difference between the leakage a grouped split prevents and feature-level [[Data Leakage]]?

## Prerequisites

[[Data Leakage]], basic train/val/test splitting (see [[Machine Learning]])

## Related concepts

[[Rolling-Origin Validation]], [[Generalization]], [[Test Error]]

## Tags

#category/statistics #topic/experimental-design

## One-line summary

> A grouped split assigns train/val/test at the level of the real-world entity ($\pi_{\text{group}}$), not the row ($\pi_{\text{row}}$), so no entity's data can be partially memorized in training and then "recognized" in test.
