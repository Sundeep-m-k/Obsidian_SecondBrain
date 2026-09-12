# ROC and PR Curves

## What is it?

**ROC (Receiver Operating Characteristic) and PR (Precision-Recall) curves** visualize how a classifier's performance changes across all possible decision thresholds. They reveal the precision-recall trade-off and enable threshold selection.

Both curves are **threshold-independent summaries**:
- Plot one metric against another as threshold $\tau$ varies from 0 to 1
- Integrate under the curve (AUC) to get a single number (0 to 1)
- AUC = probability model ranks a random positive higher than a random negative (for ROC)

---

## The ROC Curve

### Definition

**ROC** plots **True Positive Rate (TPR)** vs. **False Positive Rate (FPR)** across all thresholds:

$$\text{TPR}(\tau) = \text{Recall}(\tau) = \frac{TP(\tau)}{TP(\tau) + FN(\tau)}$$

$$\text{FPR}(\tau) = \frac{FP(\tau)}{FP(\tau) + TN(\tau)}$$

**Interpretation:**
- **TPR:** "Of actual positives, how many did we catch?" (Recall)
- **FPR:** "Of actual negatives, how many did we falsely flag?" (False alarm rate)

### Reading the ROC Curve

```
TPR (Recall)
 1.0 ┤     ╱╱╱╱
     │    ╱    ← Model: good ranking
     │   ╱
 0.5 │  ╱  ← Random: diagonal
     │ ╱ ╱ ← Model: poor ranking
     │╱╱╱╱
 0.0 ┗━━━━━━━ FPR
     0   0.5   1.0
```

**Ideal:**
- Upper-left corner: TPR ≈ 1, FPR ≈ 0 (catch all positives, few false alarms)
- Curve should **bow above the diagonal** (random is worse)

**Perfect classifier:** Curve passes through (0, 1) — 100% TPR at 0% FPR.

**Random classifier:** Curve is the diagonal (45°) — for any FPR, TPR equals FPR.

---

### AUC-ROC

**AUC** = **Area Under the Curve**. Ranges from 0 to 1.

$$\text{AUC-ROC} = P(\text{model ranks positive > negative})$$

**Interpretation:**
- AUC = 1.0: Perfect ranking (all positives ranked higher than all negatives)
- AUC = 0.5: Random (no better than coin flip)
- AUC = 0.0: Backwards (always ranks negatives higher)

**In practice:**
- AUC ≥ 0.9: Excellent
- AUC ≥ 0.8: Good
- AUC ≥ 0.7: Acceptable
- AUC ≈ 0.5: Useless

**Key property:** AUC is **threshold-independent**—it measures ranking ability, not which threshold you pick. Two models with the same AUC may have very different optimal thresholds.

---

### When to Use AUC-ROC

**Use when:**
- Binary classification with probabilistic output
- You want a single summary of ranking ability
- Classes are roughly balanced

**Don't use when:**
- Classes are heavily imbalanced (e.g., 1% positive)
- False positives and false negatives have very different costs

**Why not for imbalance?** AUC-ROC averages performance over all FPRs, including the high-FPR region that's rarely relevant for imbalanced data. See [[Class Imbalance Evaluation]].

---

## The PR Curve (Precision-Recall Curve)

### Definition

**PR curve** plots **Precision** vs. **Recall** across all thresholds:

$$\text{Precision}(\tau) = \frac{TP(\tau)}{TP(\tau) + FP(\tau)}$$

$$\text{Recall}(\tau) = \frac{TP(\tau)}{TP(\tau) + FN(\tau)}$$

### Reading the PR Curve

```
Precision
 1.0 ┤     ╱
     │    ╱ ← Model: good
     │   ╱
 0.5 ┤ ╱╱  ← Random: flat at baseline
     │╱
 0.0 ┗━━━━━━━ Recall
     0   0.5   1.0
```

**Ideal:** Upper-right corner (Recall ≈ 1, Precision ≈ 1).

**Baseline (random classifier):** Horizontal line at $P = \frac{\text{# positives}}{n}$.

For balanced data: baseline ≈ 0.5.
For imbalanced data (1% positive): baseline ≈ 0.01.

---

### AUC-PR

**AUC-PR** = Area under Precision-Recall curve. Ranges from 0 to 1.

**Interpretation:** Average precision across all recall levels.

**In practice:**
- AUC-PR ≥ 0.9: Excellent
- AUC-PR ≥ 0.7: Good
- AUC-PR ≥ 0.5: Acceptable
- AUC-PR = baseline: Random; model is useless

**Key property:** AUC-PR is **sensitive to class imbalance**. It penalizes false positives more heavily (via precision) when positives are rare. Good for imbalanced data.

---

## ROC vs. PR: When to Use Each?

| Property | ROC | PR |
|----------|-----|-----|
| **Class balance** | Balanced | Imbalanced |
| **FP cost focus** | Low cost FP | High cost FP |
| **Metric space** | TPR vs. FPR | Precision vs. Recall |
| **Baseline** | 0.5 (diagonal) | ~0.01-0.5 (varies) |
| **Best use** | General ranking comparison | Imbalanced, cost-sensitive |

**Practical rule:**
- **Balanced data (40-60% positive):** Use AUC-ROC
- **Imbalanced data (< 20% positive):** Use AUC-PR
- **Highly imbalanced (< 5% positive):** AUC-PR is essential

See [[Class Imbalance Evaluation]] for more.

---

## Worked Example

**Scenario:** Binary classifier on 100 patients (30 with disease, 70 without).

**Predictions** (sorted by confidence descending):

```
Rank  True Label  Predicted Prob
1     Positive    0.95
2     Positive    0.88
3     Negative    0.82  ← FP at this threshold
4     Positive    0.75
...
30    Negative    0.51
31    Negative    0.49
...
100   Negative    0.01

Baseline (random): P = 30/100 = 0.30
```

**Computing metrics at one threshold** (e.g., $\tau = 0.5$, top 30 predictions):

Assume top 30 contain 25 TP, 5 FP.

| Metric | Value |
|--------|-------|
| TP | 25 |
| FP | 5 |
| FN | 5 (30 - 25) |
| TN | 65 (70 - 5) |
| TPR (Recall) | 25/30 = 0.83 |
| FPR | 5/70 = 0.07 |
| Precision | 25/30 = 0.83 |

**Point on ROC:** (0.07, 0.83)
**Point on PR:** (0.83, 0.83)

Repeat for all thresholds → full curves.

---

## Threshold Selection from Curves

### From ROC Curve

**Youden's J statistic:**
$$J = \text{max}(TPR - FPR)$$

Maximizes the gap between true and false positive rates. Good for balanced error costs.

Find the point on ROC farthest from the diagonal (45° line).

### From PR Curve

**F1 Maximisation:**
$$\tau^* = \arg\max_\tau F_1(\tau) = \arg\max_\tau \frac{2 \cdot P(\tau) \cdot R(\tau)}{P(\tau) + R(\tau)}$$

Find the point on PR curve maximizing $F_1$.

**Cost-based selection:**
If FP costs $c_{FP}$ and FN costs $c_{FN}$:

$$\text{Total cost} = c_{FP} \cdot FP + c_{FN} \cdot FN$$

Find threshold minimizing this cost.

See [[Thresholding]] for detailed guidance.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Using AUC-ROC for imbalanced data | Misleading; doesn't reflect real scenario | Use AUC-PR instead |
| Comparing models on different datasets | AUC depends on data; not portable | Compare on same test set |
| Trusting AUC alone | Single number hides weakness (e.g., low recall) | Plot the curve; inspect visually |
| Using 0.5 threshold after computing AUC | AUC optimizes ranking, not threshold | Select threshold from ROC or PR curve |
| Confusing ROC and PR | Different metrics, different baselines | Remember: ROC = TPR vs FPR; PR = P vs R |

---

## Interview Questions

**Q: What does AUC-ROC measure and how do you interpret it?**

AUC-ROC measures the area under the ROC curve. Mathematically, it's the probability that a randomly chosen positive example is ranked higher (has higher predicted probability) than a randomly chosen negative example. AUC = 1 is perfect ranking, AUC = 0.5 is random (no better than coin flip). Good for balanced data; less suitable for imbalanced data where AUC-PR is preferred. See [[Class Imbalance Evaluation]].

---

**Q: When would you use PR curve instead of ROC curve?**

PR curve (Precision-Recall) is better for imbalanced datasets because it focuses on the positive class and doesn't average over low-relevance regions (high FPR). For example, with 1% positive class, ROC's baseline is 0.5 but PR's baseline is ~0.01, making PR more sensitive to model quality. Use PR when positives are rare and expensive; use ROC for balanced data. See [[Class Imbalance Evaluation]].

---

**Q: How do you select an optimal threshold from a curve?**

From ROC: Use Youden's J = max(TPR - FPR); find the point on the curve farthest from the diagonal. From PR: Maximize F1 score (point where P·R is highest). Or use cost-based approach: if FP costs $c_{FP}$ and FN costs $c_{FN}$, choose threshold minimizing $c_{FP} \cdot FP + c_{FN} \cdot FN$. Always tune on validation set, never test set. See [[Thresholding]].

---

**Q: Can two models with the same AUC-ROC have different optimal thresholds?**

Yes. AUC measures ranking ability across all thresholds; it doesn't specify which threshold to use. Two models can have identical AUC but very different ROC curves—one might have low false positive rate at low recall, the other high. The optimal threshold depends on the cost of FP vs. FN in your specific application. Select threshold separately after computing AUC. See [[Thresholding]].

---

**Q: Why is AUC-ROC misleading for imbalanced data?**

AUC-ROC averages performance across all FPRs, including high-FPR regions irrelevant to imbalanced problems. With 99% negatives, even a bad FPR of 0.1 (10% of 99 examples = 9-10 false alarms) seems acceptable, but AUC treats it as half the performance. AUC-PR penalizes FP via precision, making it more honest about imbalanced scenarios. With 1% positives, AUC-PR baseline is 0.01; any above that is meaningful progress. See [[Class Imbalance Evaluation]].

---

**Q: Explain the baseline of an ROC curve vs. a PR curve.**

ROC baseline is the diagonal (slope 1, representing random classifier at AUC = 0.5). PR baseline is a horizontal line at $y = \frac{\text{# positives}}{n}$ (the random classifier's precision). For balanced data (50% positive), PR baseline is 0.5 (same as ROC). For imbalanced data (1% positive), PR baseline is 0.01, making model quality differences visible. A model with AUC-PR = 0.5 is 50× better than random on imbalanced data; AUC-ROC = 0.75 is only 1.5× better.

---

## Connections

- [[Classification Metrics]] — Precision, Recall, TPR, FPR defined here
- [[Thresholding]] — How to select threshold from curves
- [[Class Imbalance Evaluation]] — When to use each curve
- [[Calibration and Probability Evaluation]] — Calibration complements AUC
- [[Common Mistakes in ML#12 - Reporting Accuracy on Imbalanced Data]] — Why curves matter

---

## One-line Summary

> ROC curves (TPR vs. FPR) visualize ranking ability via AUC; PR curves (Precision vs. Recall) are better for imbalanced data; use AUC as a threshold-independent summary of model quality, then select the actual threshold from the curve based on application costs.
