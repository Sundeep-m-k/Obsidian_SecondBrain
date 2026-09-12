# K-Nearest Neighbors

## What is it?

**K-Nearest Neighbors (KNN)** is an instance-based classifier (or regressor) with no training phase at all: to predict a new point, find the $k$ closest training points by distance and predict the majority class among them (or the average target value, for regression).

$$\hat{y} = \text{mode}\big(\{y_i : x_i \in N_k(x)\}\big)$$

where $N_k(x)$ is the set of $k$ nearest training points to $x$, usually by Euclidean distance. Not to be confused with [[K Means]] — that's an unsupervised clustering algorithm with a similar-sounding name and a completely different mechanism (it learns cluster centroids; KNN never "learns" anything, it just looks things up).

---

## "Lazy Learning": No Training Phase

KNN has no fitting step — "training" is just storing the dataset. All the computation happens at prediction time, scanning the stored data to find the $k$ nearest points. This is the opposite of every model covered elsewhere in this vault, which front-loads computation into a training phase to make inference cheap. KNN inverts that tradeoff: cheap/instant "training," expensive inference that grows with dataset size.

---

## Choosing $k$

Small $k$ (e.g. $k=1$) fits the training data very tightly — low bias, high variance, sensitive to noise and outliers in exactly the way an overfit model is. Large $k$ smooths the decision boundary — higher bias, lower variance, and at the extreme ($k=n$) it just predicts the overall majority class regardless of $x$. This is the same [[Bias Variance Tradeoff]] every other model faces, expressed through a single hyperparameter instead of model complexity. $k$ is typically chosen via [[Cross Validation Strategy]]; odd values of $k$ are conventional for binary classification to avoid ties.

---

## Why Feature Scaling Is Non-Negotiable Here

KNN's entire prediction is a distance computation, so it inherits the same problem as SVM's margin: an unscaled large-range feature dominates the distance and silently determines the "nearest" neighbors regardless of what the other features say. Always apply [[Feature Scaling]] before using KNN — this matters more here than almost anywhere else in the vault, since distance *is* the entire mechanism.

## The Curse of Dimensionality

As the number of features grows, all points tend to become roughly equidistant from each other in high-dimensional space, and the notion of "nearest" neighbor stops being meaningful. KNN degrades sharply in high dimensions unless paired with [[Dimensionality Reduction]] (e.g. [[PCA]]) first — this is a much bigger practical concern for KNN than for tree-based or margin-based methods.

---

## Advantages / Disadvantages

**Advantages:** simple to understand and implement, no training time, naturally handles multi-class problems, decision boundary can be arbitrarily complex (non-parametric — no assumption about the underlying function shape).

**Disadvantages:** slow at inference on large datasets (naive implementation is $O(n)$ per prediction; approximate nearest-neighbor structures like KD-trees or ball trees help but don't eliminate the cost), requires the entire training set in memory, degrades badly in high dimensions, extremely sensitive to feature scale and irrelevant features.

## When to Use / When Not To

**Use** for small-to-medium datasets with a modest number of informative features, as a quick baseline, or when the true decision boundary is known to be irregular/non-parametric. **Avoid** for large datasets where inference latency matters (a trained parametric model is far cheaper at prediction time), or high-dimensional data without first reducing dimensionality.

---

## Interview Questions

**Why does KNN have no training phase, and what does that cost you?** It just stores the data; all cost is deferred to inference, which means prediction time scales with dataset size — the opposite tradeoff from a model like logistic regression, which is expensive to fit once but cheap to query forever after.

**What happens with $k=1$ vs. a very large $k$?** $k=1$ perfectly memorizes the nearest single point per prediction — low bias, high variance, easily fooled by a single noisy neighbor. A very large $k$ averages over most of the dataset — high bias, low variance, and at $k=n$ it ignores $x$ entirely and predicts the global majority class.

**Why is KNN especially sensitive to feature scaling, more so than most models?** Because distance computation *is* the entire prediction mechanism, not just one factor among many — an unscaled feature doesn't just slow convergence (as with gradient descent), it directly and silently changes which points count as "nearest."

**What is the curse of dimensionality and why does it hurt KNN specifically?** In high dimensions, distances between all pairs of points converge toward similar values, so "nearest" stops carrying useful signal — a problem that's much sharper for a purely distance-based method like KNN than for a tree-based model that only ever looks at one feature at a time.

## Connections

- [[K Means]] — similarly named, unrelated mechanism (unsupervised clustering vs. supervised lookup)
- [[Feature Scaling]] — required, for the same reason as SVM
- [[Dimensionality Reduction]], [[PCA]] — the standard fix for the curse of dimensionality
- [[Bias Variance Tradeoff]] — $k$ is KNN's single complexity knob
- [[Cross Validation Strategy]] — the standard way to choose $k$

## One-line Summary

> KNN predicts by majority vote among the $k$ closest training points with no training phase at all — simple and non-parametric, but its entire mechanism is distance, so it demands scaled features and degrades badly in high dimensions.
