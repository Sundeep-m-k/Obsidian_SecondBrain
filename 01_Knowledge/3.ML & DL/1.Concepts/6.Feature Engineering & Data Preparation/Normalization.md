# Normalization

## What is it?

**Normalization** (also called **min-max scaling**) rescales each feature to the range $[0, 1]$ (or $[a, b]$ for a custom range).

$$x_j' = \frac{x_j - \min_j}{\max_j - \min_j}$$

Where $\min_j$ and $\max_j$ are the minimum and maximum of feature $j$ in the **training set**.

---

## General $[a, b]$ Scaling

$$x_j' = a + \frac{(x_j - \min_j)(b-a)}{\max_j - \min_j}$$

Common: $[0,1]$ or $[-1, 1]$.

---

## After Normalisation

- Values in $[0, 1]$.
- Distribution shape preserved (same skew, same relative spacing).
- Mean is no longer 0 (unless data was symmetric).

---

## When to Use

✅ Neural networks with sigmoid/tanh activations (outputs naturally in $(0,1)$ or $(-1,1)$)  
✅ Image pixels: divide by 255 to map $[0, 255] \to [0, 1]$  
✅ When you need bounded values  
❌ When data has significant outliers — one extreme value compresses all others  

---

## Sensitivity to Outliers

A single outlier at $x = 10{,}000$ when all other values are $\leq 100$ maps everything to near zero:
$$x_j' = \frac{x - 0}{10{,}000 - 0} \approx 0 \quad \text{for all normal values}$$

**Fix:** Cap outliers before normalising, or use [[Standardization]] (robust to outliers).

---

## Connections

- [[Feature Scaling]] — normalisation is one scaling method
- [[Standardization]] — the alternative (zero-mean, unit-variance)
- [[Feature Engineering]] — part of the preprocessing pipeline

---

## One-line Summary

> Normalization rescales features to $[0,1]$ by subtracting the minimum and dividing by the range — simple and interpretable, but sensitive to outliers which can compress the whole distribution toward zero.
