# Confusion Matrix

## What is it?

A **confusion matrix** is a table that cross-tabulates a classifier's predictions against the true labels, showing exactly which classes get confused for which. Every classification metric in this module — accuracy, precision, recall, F1 — is computed from the four counts it contains.

|  | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | True Positive (TP) | False Negative (FN) |
| **Actual Negative** | False Positive (FP) | True Negative (TN) |

---

## The Four Counts

- **True Positive (TP)** — predicted positive, actually positive. A correct catch.
- **False Positive (FP)** — predicted positive, actually negative. A false alarm (Type I error).
- **False Negative (FN)** — predicted negative, actually positive. A miss (Type II error).
- **True Negative (TN)** — predicted negative, actually negative. Correctly ignored.

Every metric in [[Classification Metrics]] is a ratio built from these four numbers:

$$\text{Accuracy} = \frac{TP+TN}{TP+TN+FP+FN}, \quad \text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}$$

---

## Why the Matrix Matters More Than Any Single Metric

A single number like accuracy hides *which kind* of mistake the model is making. Two models can have identical accuracy while one fails on false positives and the other on false negatives — a distinction that matters enormously when the two error types have different costs (e.g. a missed fraud case vs. a flagged legitimate transaction).

**Worked example** — a fraud model on 1000 transactions, 20 actually fraudulent:

|  | Predicted Fraud | Predicted Legit |
|---|---|---|
| **Actually Fraud** | 15 (TP) | 5 (FN) |
| **Actually Legit** | 30 (FP) | 950 (TN) |

Accuracy = (15+950)/1000 = 96.5% — looks great. But recall = 15/20 = 75% (a quarter of fraud is missed) and precision = 15/45 = 33% (two-thirds of fraud alerts are false alarms). The confusion matrix is what exposes this; accuracy alone hides it entirely. See [[Class Imbalance Evaluation]] for why accuracy is especially misleading on data shaped like this.

---

## Multi-Class Confusion Matrices

For $k$ classes, the matrix is $k \times k$: row $i$, column $j$ counts examples of true class $i$ predicted as class $j$. The diagonal is correct predictions; everything off-diagonal is a specific *kind* of error — which is often more actionable than an aggregate error rate, since it shows exactly which pairs of classes the model confuses (e.g. a digit classifier confusing 4s and 9s specifically, rather than failing uniformly).

---

## Interview Questions

**Why look at a confusion matrix instead of just accuracy?** Accuracy collapses two very different failure modes (false positives, false negatives) into one number and is misleading under class imbalance — the matrix shows the actual error composition, which is usually what determines whether a model is fit for purpose.

**A spam filter has 99% accuracy on 1% spam data — good?** Not necessarily — predicting "not spam" for everything also gets 99% accuracy. Check the confusion matrix: if TP (caught spam) is near zero, the model is worthless despite the headline accuracy. This is the same trap as the fraud example above.

**What's the difference between a false positive and a false negative, and why does the distinction matter for model selection?** FP = wrongly flagged positive, FN = wrongly missed positive; which one is worse is entirely domain-dependent (medical screening usually prioritizes minimizing FN; spam filtering usually prioritizes minimizing FP) — see [[Thresholding]] for how the decision threshold trades one against the other.

---

## Connections

- [[Classification Metrics]] — every metric here is derived from this matrix
- [[Thresholding]] — moving the decision threshold moves counts between all four cells
- [[Class Imbalance Evaluation]] — why the matrix (not accuracy) is the right starting point under imbalance
- [[ROC and PR Curves]] — visualizing the TP/FP tradeoff across every possible threshold at once

## One-line Summary

> The confusion matrix's four counts (TP, FP, FN, TN) are the raw material every classification metric is computed from — read it before trusting any single summary number, especially under class imbalance.
