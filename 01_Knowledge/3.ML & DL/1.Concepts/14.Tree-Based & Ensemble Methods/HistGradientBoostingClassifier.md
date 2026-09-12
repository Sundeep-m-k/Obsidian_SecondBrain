# HistGradientBoostingClassifier (HGB)

## What is it?

`HistGradientBoostingClassifier` (HGB) is scikit-learn's **histogram-based** implementation of [[Gradient Boosting]], inspired by LightGBM. Same algorithm underneath, made much faster by changing *how splits are found*.

## The Key Trick: Histogram Binning

Standard gradient boosting scans every possible threshold over sorted continuous values at each node — $O(n\log n)$-ish per feature per split. HGB instead:

1. Pre-bins every continuous feature into $B$ discrete bins (typically $B=255$) once, up front — $O(n\log n)$ paid *once* per feature, not per split.
2. Builds a histogram of gradient/count sums per bin at each node — $O(n)$ per node.
3. Searches for the best split over the $B$ bins, not raw sorted values — $O(B)$ per node instead of $O(n)$.

## Why This Is Faster: the Complexity Comparison

| Step | Exact (per-value) split search | Histogram-based split search |
|---|---|---|
| Per-node split search | $O(n \cdot p)$ (scan every value, every feature) | $O(B \cdot p)$ ($B=255 \ll n$ for large datasets) |
| Total over a tree of depth $d$ | $O(n \cdot p \cdot d)$ | $O(B \cdot p \cdot d)$, plus one-time $O(n \log n \cdot p)$ binning |

For $n=1{,}000{,}000$ rows and $B=255$ bins, the per-node search is roughly $1{,}000{,}000/255 \approx 3{,}900\times$ cheaper — the source of HGB's practical speed on large tabular datasets like a data-warehouse-derived modeling table.

## Practical Properties

- **Native missing-value support** — learns which branch to send missing values down during training, no separate imputation
- **Native categorical support** (recent scikit-learn) — no manual one-hot encoding for moderate-cardinality categoricals
- Built-in early stopping via a validation fraction
- Accuracy close to XGBoost/LightGBM, simpler dependency footprint (ships in scikit-learn)

## Why It Matters in a Forecasting/Production Context

For a modeling table with tens/hundreds of thousands of rows, from-phase-only features, and structurally meaningful missing values, HGB is a natural default candidate against a simpler baseline ([[Baseline Estimator]]) — the comparison must run through an honest evaluation protocol ([[Grouped Train-Test Split]], [[Rolling-Origin Validation]]), never assumed to win just because it's more sophisticated.

## Advantages

Fast, memory-efficient on large tabular data; native missing values and categoricals; no extra library dependency.

## Limitations

Binning introduces a small approximation vs. exact-threshold boosting (usually negligible at $B=255$); still needs validation-based tuning of $\eta$/rounds/depth, same overfitting risks as any [[Gradient Boosting]] model; sophistication ≠ automatic superiority over a simpler baseline.

## Common Mistakes

- Assuming HGB automatically outperforms a domain-appropriate baseline without an honest, non-circular evaluation
- Not checking output calibration — good AUC doesn't imply trustworthy raw probabilities (see [[Calibration]], [[Brier Score]])

## Interview / discussion questions

- Derive why histogram binning changes per-node split search from $O(n)$ to $O(B)$, and quantify the speedup for a specific $n$ and $B$.
- How does HGB handle missing values differently from an implementation requiring imputation?
- Why evaluate HGB against a much simpler baseline before deploying it, rather than assuming it wins?

## Prerequisites

[[Gradient Boosting]], [[Decision Tree]]

## Related concepts

[[Gradient Boosting Libraries (XGBoost & LightGBM)]], [[Baseline Estimator]], [[Calibration]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/complexity

## One-line summary

> HistGradientBoostingClassifier speeds up gradient boosting by pre-binning continuous features into ~255 histogram bins, cutting per-node split search from $O(n)$ to $O(B)$ — same algorithm as Gradient Boosting, engineered for speed and scale on large tabular data.
