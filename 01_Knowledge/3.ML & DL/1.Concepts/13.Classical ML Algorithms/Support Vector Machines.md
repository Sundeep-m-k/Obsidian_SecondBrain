# Support Vector Machines

## What is it?

A **Support Vector Machine (SVM)** is a classifier that finds the decision boundary maximizing the margin — the distance between the boundary and the nearest points of each class. Those nearest points are the **support vectors**; they're the only training points that determine the boundary at all.

$$\max_{w,b} \ \frac{2}{\|w\|} \quad \text{s.t.} \quad y_i(w^\top x_i + b) \geq 1 \ \ \forall i$$

Equivalently, minimize $\|w\|^2$ subject to the same constraints — a convex quadratic program with a unique global optimum, unlike gradient descent on a neural net.

---

## Why Maximize the Margin?

A [[Decision Boundary]] that barely separates the classes is fragile — a new point close to the boundary on either side easily gets misclassified. Maximizing the margin is a direct generalization argument: the boundary furthest from every training point is the one most likely to also separate unseen points correctly. This is the same instinct as [[Regularization]] — trading a perfect fit on training data for a boundary that generalizes.

---

## Soft Margin: Handling Non-Separable Data

Real data usually isn't perfectly separable. The **soft margin** SVM allows some points to violate the margin, penalized by slack variables $\xi_i$:

$$\min_{w,b,\xi} \ \frac{1}{2}\|w\|^2 + C\sum_i \xi_i \quad \text{s.t.} \quad y_i(w^\top x_i+b) \geq 1-\xi_i, \ \xi_i \geq 0$$

$C$ controls the tradeoff: large $C$ penalizes margin violations heavily (narrow margin, can overfit); small $C$ tolerates more violations (wide margin, can underfit). $C$ plays the same role as $1/\lambda$ does in [[L2 Regularization]].

---

## The Kernel Trick

Many datasets aren't linearly separable in their original feature space. Rather than explicitly transforming features into a higher-dimensional space where they might become separable, a **kernel function** computes the dot product *as if* that transformation had happened, without ever computing it:

$$K(x_i, x_j) = \phi(x_i)^\top \phi(x_j)$$

**RBF (Gaussian) kernel**, the most common: $K(x_i,x_j) = \exp(-\gamma \|x_i-x_j\|^2)$ — implicitly maps to infinite dimensions; $\gamma$ controls how far a single point's influence reaches (large $\gamma$ → narrow influence, overfitting risk; small $\gamma$ → wide influence, underfitting risk).

**Linear kernel**: $K(x_i,x_j) = x_i^\top x_j$ — just the original SVM, no transformation. Preferred when the feature count is already large relative to sample count (e.g. text with TF-IDF features), where the data is often already near-linearly-separable.

---

## Why Feature Scaling Matters Here

SVM optimizes a margin defined by Euclidean distance, so it's a distance-based method exactly like [[K Means]] — an unscaled large-range feature dominates the margin computation. Always apply [[Feature Scaling]] before fitting an SVM.

---

## Advantages / Disadvantages

**Advantages:** effective in high-dimensional spaces (even when features outnumber examples), memory-efficient at inference (only support vectors matter), the kernel trick handles non-linear boundaries without manual feature engineering, convex optimization guarantees a global optimum.

**Disadvantages:** doesn't scale well to very large datasets (training is $O(n^2)$ to $O(n^3)$ in the number of examples), no direct probability output (requires an extra calibration step, e.g. Platt scaling, to get one — see [[Calibration and Probability Evaluation]]), sensitive to the choice of kernel and its hyperparameters, hard to interpret relative to a [[Decision Tree]].

## When to Use / When Not To

**Use** for small-to-medium, high-dimensional datasets where a clear margin exists — classic text classification, bioinformatics with many features and few samples. **Avoid** for very large datasets (gradient boosting or a linear model will train far faster) or when a probability estimate or feature-level interpretability is required as a first-class output.

---

## Interview Questions

**What is the kernel trick and why does it matter?** It computes the dot product in a higher-dimensional (possibly infinite-dimensional) feature space without ever explicitly constructing that space, making non-linear decision boundaries tractable at the cost of only $O(n^2)$ kernel evaluations rather than an explicit feature transform.

**What does the hyperparameter $C$ control?** The tradeoff between margin width and margin violations — small $C$ favors a wide margin and tolerates misclassified/close points (more bias, less variance); large $C$ fits the training data more tightly (less bias, more variance, overfitting risk). It plays the same conceptual role as inverse regularization strength.

**Why does SVM need scaled features but a decision tree doesn't?** SVM's margin is a Euclidean-distance notion, so an unscaled large-range feature dominates it exactly the way it dominates KNN; trees split on per-feature thresholds and never compare magnitudes across features.

**Linear vs. RBF kernel — how do you choose?** Start linear if features already outnumber samples or a quick, interpretable baseline is wanted (e.g. text with TF-IDF); reach for RBF when the boundary is visibly non-linear and the dataset is small enough that the extra compute cost is affordable.

## Connections

- [[Decision Boundary]], [[Regularization]] — the margin-maximization idea is a direct analog of regularization's complexity penalty
- [[Feature Scaling]] — required, exactly as for KNN
- [[Calibration and Probability Evaluation]] — needed to get probability outputs from an SVM
- [[Decision Tree]], [[K Means]] — the other classical non-linear paradigms this note complements

## One-line Summary

> SVM finds the maximum-margin decision boundary using only the closest points (support vectors), and the kernel trick lets that boundary be non-linear without ever explicitly constructing a higher-dimensional feature space — at the cost of needing scaled features and not scaling gracefully to very large datasets.
