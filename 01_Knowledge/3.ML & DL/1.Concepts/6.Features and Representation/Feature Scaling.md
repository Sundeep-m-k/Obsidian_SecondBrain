# Feature Scaling

## What is it?

**Feature scaling** is the process of transforming features to similar numerical ranges so that no single feature dominates due to its scale rather than its actual predictive importance.

---

## Why It's Necessary

**For gradient-based algorithms:** When features have vastly different scales, the cost surface becomes elongated (high condition number). Gradient descent oscillates in the steep directions and barely moves in the shallow ones → very slow convergence.

**Example:**
- $x_1$ = house size in sq ft (range: 500–5000)
- $x_2$ = number of bedrooms (range: 1–6)

Without scaling: $J(\theta)$ changes rapidly in the $\theta_1$ direction and slowly in $\theta_2$. GD uses tiny steps to avoid overshooting $\theta_1$, making $\theta_2$ converge extremely slowly.

With scaling: both features in $[0,1]$ or $(-1,1)$ → contours are circular → GD takes direct steps to minimum.

**For distance-based algorithms** (kNN, k-Means, SVM): Euclidean distance is dominated by large-scale features without scaling.

**Does NOT matter for:** Decision trees, Random forests, Gradient boosted trees (they use thresholds, not distances or dot products).

---

## Rule: Always Fit Scaling on Training Set Only

$$\mu_j = \frac{1}{n_{\text{train}}}\sum_{i \in \text{train}} x_j^{(i)}, \quad \sigma_j = \sqrt{\frac{1}{n_{\text{train}}}\sum_{i \in \text{train}}(x_j^{(i)} - \mu_j)^2}$$

Apply to validation and test using **training set** statistics:
$$x_j^{(i)'} = \frac{x_j^{(i)} - \mu_j^{\text{train}}}{\sigma_j^{\text{train}}}$$

**Never fit the scaler on the full dataset** — that leaks test information into training.

---

## Connections

- [[Standardization]] — the most common scaling method
- [[Normalization]] — min-max scaling
- [[Feature Engineering]] — scaling is part of the preprocessing pipeline
- [[Gradient Descent]] — scaling dramatically improves convergence

---

## One-line Summary

> Feature scaling transforms all features to comparable numerical ranges, preventing large-scale features from dominating gradient descent or distance computations — always computed on the training set and then applied identically to all splits.
