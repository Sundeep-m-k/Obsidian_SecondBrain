# Classification Metrics

## What is it?

**Classification metrics** are quantitative measures that evaluate the performance of a binary (or multi-class) classifier. They translate raw predictions into interpretable summaries of how often the model is right, wrong, and at what cost.

A classifier produces:
- **Hard predictions**: $\hat{y} \in \{0, 1\}$ (class label)
- **Soft predictions**: $\hat{p} \in (0, 1)$ (probability of class 1)

Metrics measure how well these match the true labels $y$.

---

## The Confusion Matrix

The **confusion matrix** is the foundation of all binary classification metrics. It counts four types of outcomes:

| | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actually Positive** | TP (True Positive) | FN (False Negative) |
| **Actually Negative** | FP (False Positive) | TN (True Negative) |

**Example:** Predicting cancer (positive) vs. no cancer (negative) for 100 patients:

```
              Predicted Positive    Predicted Negative
Actually +ve            45 (TP)              5 (FN)
Actually -ve            8 (FP)             42 (TN)
```

**Key insight:** The confusion matrix reveals the *type* of errors (FP vs. FN), not just their *count*. This matters because FP and FN have different costs in real applications.

---

## The Four Basic Metrics

### 1. Accuracy

$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN} = \frac{\text{correct}}{\text{total}}$$

**What it is:** Proportion of all predictions that are correct.

**When to use:** Balanced datasets where all errors cost the same.

**When NOT to use:** Imbalanced datasets (e.g., 99% negative class). A model predicting "always negative" gets 99% accuracy but catches 0% of positives. ⚠️ See [[Common Mistakes in ML#12 - Reporting Accuracy on Imbalanced Data]].

**Example:** With 45 TP, 5 FN, 8 FP, 42 TN:
$$\text{Accuracy} = \frac{45+42}{100} = 0.87 = 87\%$$

---

### 2. Precision

$$\text{Precision} = \frac{TP}{TP + FP} = \frac{\text{true positives}}{\text{all predicted positives}}$$

**What it is:** "Of all the cases I flagged as positive, how many were actually positive?"

**Interpretation:**
- High precision: Few false alarms. When the model says "positive," it's usually right.
- Low precision: Many false alarms.

**When to use:** When false positives are costly.
- **Spam filter:** Don't block legitimate emails (FP is bad).
- **Loan approval:** Don't approve bad loans (FP is bad).

**Example:** Precision = $\frac{45}{45+8} = 0.849 = 84.9\%$. Of all 53 cases flagged as cancer, 45 actually had cancer.

---

### 3. Recall (Sensitivity, True Positive Rate)

$$\text{Recall} = \frac{TP}{TP + FN} = \frac{\text{true positives}}{\text{all actual positives}}$$

**What it is:** "Of all the cases that were actually positive, how many did I catch?"

**Interpretation:**
- High recall: Catch most positives. Few cases slip through.
- Low recall: Miss many positives.

**When to use:** When false negatives are costly.
- **Cancer screening:** Don't miss cancer (FN is bad).
- **Credit card fraud:** Don't let fraud go undetected (FN is bad).

**Example:** Recall = $\frac{45}{45+5} = 0.90 = 90\%$. Of 50 actual cancer cases, we caught 45.

---

### 4. F1 Score

$$F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2 \cdot TP}{2 \cdot TP + FP + FN}$$

**What it is:** Harmonic mean of precision and recall. Balances both metrics.

**Interpretation:**
- $F_1 = 1$: Perfect (all TP, no FP, no FN).
- $F_1 = 0$: Worthless (either $P=0$ or $R=0$).
- Punishes extreme imbalance between P and R.

**When to use:** Imbalanced data where you care about both false positives and false negatives, but don't have explicit costs.

**Example:** $F_1 = 2 \cdot \frac{0.849 \cdot 0.90}{0.849 + 0.90} = 0.875 = 87.5\%$

**Key property:** Unlike accuracy, $F_1$ is robust to class imbalance. Even if negatives dominate, $F_1$ only depends on TP, FP, FN.

---

## Precision vs. Recall Trade-off

**Central fact:** Precision and recall are in tension. Moving the threshold $\tau$ changes both.

$$\text{Lower threshold } \tau \Rightarrow \text{ predict "positive" more often}$$
- More TP (good) ✅
- More FP (bad) ❌
- Recall ↑, Precision ↓

$$\text{Higher threshold } \tau \Rightarrow \text{ predict "positive" less often}$$
- Fewer FP (good) ✅
- Fewer TP (bad) ❌
- Recall ↓, Precision ↑

**You must choose:** Do you prefer false alarms (FP) or missed cases (FN)? This determines the optimal threshold. See [[Thresholding]].

---

## Macro vs. Micro Averaging (Multi-class)

For $K > 2$ classes, you can average metrics two ways:

### Macro Average
Compute metric for each class separately, then average:
$$\text{Macro F}_1 = \frac{1}{K} \sum_{k=1}^K F_{1,k}$$
**Each class weighted equally.** Use when all classes matter.

### Micro Average
Aggregate TP, FP, FN across all classes, then compute metric:
$$\text{Micro F}_1 = \frac{2 \sum_k TP_k}{2 \sum_k TP_k + \sum_k FP_k + \sum_k FN_k}$$
**Larger classes weighted more.** Use when class frequency reflects real importance.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Using accuracy for imbalanced data | Misleading; trivial model achieves high accuracy | Use Precision, Recall, F1, or AUC-PR |
| Ignoring class imbalance entirely | Model may learn to predict majority class | Check recall for minority class |
| Setting threshold to 0.5 by default | Suboptimal when FP and FN costs differ | Tune threshold; see [[Thresholding]] |
| Reporting only one metric | Single metric hides weakness (high P, low R) | Report P, R, F1 together |
| Confusing Precision with Recall | Easy to mix up; remember "of what" | Precision = "of predicted", Recall = "of actual" |

---

## Interview Questions

**Q: What is the confusion matrix and why is it useful?**

The confusion matrix counts four outcomes: TP, TN, FP, FN. It's useful because it reveals the *type* of errors (false positives vs. false negatives), not just how many errors occurred. This matters because real applications have asymmetric costs—missing a cancer diagnosis (FN) is worse than a false alarm (FP). See [[Common Mistakes in ML#2 - Not Handling Class Imbalance]].

---

**Q: When would you use Precision vs. Recall?**

**Precision** = "of all predicted positives, how many are correct?" Use when false positives are costly: spam filters (don't block legitimate email), loan approval (don't approve bad loans).

**Recall** = "of all actual positives, how many did I find?" Use when false negatives are costly: cancer screening (don't miss cancer), fraud detection (don't let fraud pass).

In practice, you optimize for whichever cost is higher in your domain. See [[Thresholding]].

---

**Q: Why is Accuracy a bad metric for imbalanced data?**

With 99% negative examples, a model that always predicts "negative" achieves 99% accuracy but has 0% recall on the positive class. You learn nothing about whether the model actually works on the minority class. Instead, report Precision, Recall, F1, or AUC-PR, which are insensitive to class imbalance. See [[Common Mistakes in ML#12 - Reporting Accuracy on Imbalanced Data]].

---

**Q: What does F1 measure and when should I use it?**

F1 = harmonic mean of Precision and Recall. It balances both: if one is very low, F1 is pulled down. Use F1 when you care about both false positives and false negatives but don't have explicit costs, or when you want a single-number summary for imbalanced data. F1 is resistant to class imbalance: even if negatives dominate, F1 only depends on TP, FP, FN—not TN.

---

**Q: How do Macro and Micro averaging differ for multi-class metrics?**

**Macro:** Compute the metric per class, then average. Each class weighted equally. Use when all classes matter.

**Micro:** Aggregate TP, FP, FN across all classes, then compute. Larger classes weighted more. Use when class frequency is meaningful.

In practice: Macro F1 = 0.7, Micro F1 = 0.8 usually means your model is good at predicting common classes but weak on rare ones.

---

**Q: Given Precision = 0.9 and Recall = 0.5, what does this tell you?**

High precision, low recall means the model is conservative: when it predicts positive, it's usually right (90% correct), but it misses many actual positives (only catches 50%). This suggests the threshold is too high. Lower it to increase recall (catch more positives) at the cost of precision (more false alarms). Your F1 = 2·(0.9·0.5)/(0.9+0.5) = 0.643, mediocre because of the recall weakness. See [[Thresholding]].

---

**Q: Why use F1 instead of just reporting P and R separately?**

F1 gives a single number; P and R give two. F1 is useful for model selection—if you have 100 candidate models, F1 lets you rank them on one metric. However, F1 hides the direction of weakness (is precision or recall the bottleneck?), so best practice is F1 + separate P and R. See [[Common Mistakes in ML#12 - Reporting Accuracy on Imbalanced Data]].

---

**Q: How does threshold affect Precision and Recall?**

Lower threshold (predict positive more often): Recall ↑, Precision ↓ (more TP but more FP).
Higher threshold (predict positive less often): Recall ↓, Precision ↑ (fewer FP but fewer TP).
This trade-off is unavoidable; you must choose based on the cost of FP vs. FN in your application. See [[ROC and PR Curves]] for how to visualize and optimize this. See [[Thresholding]].

---

## Connections

- [[Confusion Matrix]] — the foundation
- [[Thresholding]] — how threshold affects P and R
- [[ROC and PR Curves]] — visualizing the precision-recall trade-off
- [[Class Imbalance Evaluation]] — when F1 and AUC-PR matter most
- [[Common Mistakes in ML#12 - Reporting Accuracy on Imbalanced Data]] — pitfall to avoid
- [[Classification Pipeline]] — how metrics fit into the workflow

---

## One-line Summary

> Classification metrics (Accuracy, Precision, Recall, F1) translate predictions into interpretable summaries; Precision and Recall are in tension with threshold, and F1 is the balanced metric for imbalanced data — always report multiple metrics, never just Accuracy.
