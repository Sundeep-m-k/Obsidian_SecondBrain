# Classification

## What is it?

**Classification** is a type of [[Supervised Learning]] task where the model learns to assign an input to one of a finite set of **discrete categories** (classes).

$$\hat{f}: \mathbb{R}^d \rightarrow \{1, 2, \ldots, K\}$$

The output $y$ belongs to a finite set of classes, not a continuous range.

---

## Formal Setup

Given training data:
$$\mathcal{D} = \{(x^{(1)}, y^{(1)}), \ldots, (x^{(n)}, y^{(n)})\}, \quad y^{(i)} \in \{1, \ldots, K\}$$

The model learns a **decision function** (or a class-conditional probability):
$$\hat{y} = \arg\max_{k} P(Y = k \mid X = x)$$

---

## Types of Classification

| Type | Classes $K$ | Output | Example |
|---|---|---|---|
| **Binary** | $K = 2$ | 0 or 1 | Spam / Not spam |
| **Multi-class** | $K > 2$, mutually exclusive | One of $K$ labels | Digit (0–9), Species |
| **Multi-label** | Multiple labels simultaneously | Subset of labels | Article tags (Sports AND Health) |
| **Ordinal** | Ordered classes | Class with order constraint | Rating (1–5 stars) |
| **Hierarchical** | Classes have a tree structure | Node in taxonomy | ImageNet categories |

See: [[Binary Classification]], [[Multiclass Classification]]

---

## The Output: Probabilities vs. Hard Labels

Most classifiers output **probabilities** first, then convert to a hard label via a threshold or argmax.

**Softmax** (multi-class):
$$P(Y = k \mid x) = \frac{e^{z_k}}{\sum_{j=1}^{K} e^{z_j}}$$

**Sigmoid** (binary):
$$P(Y = 1 \mid x) = \frac{1}{1 + e^{-z}}$$

**Decision:** $\hat{y} = \arg\max_k P(Y=k \mid x)$, or $\hat{y} = \mathbf{1}[\hat{p} \geq \tau]$ for binary with threshold $\tau$.

---

## Loss Functions for Classification

### Binary Cross-Entropy (Log Loss)
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log \hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

### Categorical Cross-Entropy
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n}\sum_{k=1}^{K} y_k^{(i)} \log \hat{p}_k^{(i)}$$

With one-hot labels, this simplifies to:
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n} \log \hat{p}_{c^{(i)}}^{(i)}$$

i.e., minimise the negative log-probability of the correct class.

### Hinge Loss (for SVMs)
$$\mathcal{L} = \frac{1}{n}\sum_{i=1}^{n} \max(0, 1 - y^{(i)}\hat{f}(x^{(i)})), \quad y \in \{-1, +1\}$$

---

## Evaluation Metrics

### Confusion Matrix (Binary)

|  | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | TP | FN |
| **Actual Negative** | FP | TN |

$$\text{Accuracy} = \frac{TP+TN}{TP+TN+FP+FN}$$

$$\text{Precision} = \frac{TP}{TP+FP} \quad \text{(of predicted positives, how many correct?)}$$

$$\text{Recall (Sensitivity)} = \frac{TP}{TP+FN} \quad \text{(of actual positives, how many found?)}$$

$$\text{Specificity} = \frac{TN}{TN+FP} \quad \text{(of actual negatives, how many correctly rejected?)}$$

$$F_1 = 2 \cdot \frac{\text{Precision} \times \text{Recall}}{\text{Precision}+\text{Recall}}$$

$$F_\beta = (1+\beta^2) \cdot \frac{\text{Precision} \times \text{Recall}}{\beta^2 \cdot \text{Precision}+\text{Recall}}$$

($\beta > 1$: prioritise recall; $\beta < 1$: prioritise precision)

### ROC-AUC
- Plot TPR (Recall) vs. FPR at every threshold.
- **AUC = 1**: perfect classifier.
- **AUC = 0.5**: random classifier.
- AUC is threshold-independent — measures ranking ability.

### PR-AUC (Precision-Recall Area)
- Better than ROC-AUC for **imbalanced** datasets.
- High precision + high recall at all thresholds = high PR-AUC.

### Multi-class metrics
Average the binary metrics per class:
- **Macro average:** unweighted mean across classes (treats all classes equally)
- **Weighted average:** mean weighted by class frequency (accounts for imbalance)
- **Micro average:** aggregate TP, FP, FN across all classes, then compute

---

## Key Classification Algorithms

| Algorithm | Decision Boundary | Notes |
|---|---|---|
| Logistic Regression | Linear | Probabilistic, fast, interpretable |
| Linear SVM | Linear (max-margin) | Large margin, no probabilities |
| Kernel SVM | Non-linear | Powerful but slow for large data |
| Decision Tree | Axis-aligned steps | Interpretable, prone to overfit |
| Random Forest | Ensemble of trees | Robust, handles mixed features |
| k-NN | Voronoi boundaries | Non-parametric, lazy learner |
| Naive Bayes | Linear (in log space) | Fast, works with tiny data |
| Neural Network | Arbitrary | Most expressive |

---

## The Bayes Optimal Classifier

The theoretically best classifier minimises expected 0-1 loss:
$$f^*(x) = \arg\max_k P(Y = k \mid X = x)$$

Its error rate is the **Bayes error** (irreducible error). No classifier can beat it. We can only approach it with better models and more data.

---

## Connections

- [[Supervised Learning]] — classification is a supervised task
- [[Binary Classification]] — special case with $K=2$
- [[Multiclass Classification]] — special case with $K>2$
- [[Decision Boundary]] — the surface separating predicted classes
- [[Logistic Regression]] — the canonical probabilistic classifier
- [[Loss Function]] — cross-entropy is the standard classification loss
- [[Regression]] — the contrast: continuous outputs
- [[Decision Tree]], [[Random Forest]], [[Gradient Boosting]], [[HistGradientBoostingClassifier]] — the tree-based family from the algorithm table above, with full split-mechanics and worked examples in `14.Tree-Based & Ensemble Methods`
- [[Calibration]], [[Brier Score]], [[Proper Scoring Rule]] — ROC-AUC and PR-AUC above measure ranking ability only; these measure whether $\hat p$ itself is trustworthy, a separate axis entirely

---

## One-line Summary

> Classification is supervised learning for discrete outputs — the model learns to assign each input to a category by learning the boundaries between classes, evaluated by how often it assigns the correct category.
