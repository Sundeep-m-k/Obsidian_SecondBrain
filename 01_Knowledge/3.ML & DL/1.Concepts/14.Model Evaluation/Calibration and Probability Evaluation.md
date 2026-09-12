# Calibration and Probability Evaluation

## What is it?

**Calibration** measures whether predicted probabilities match observed frequencies. A well-calibrated model's predicted probability for an event reflects the actual probability that event occurs.

> When a calibrated model predicts $\hat{p} = 0.7$, approximately 70% of such examples are actually positive.

This is distinct from **ranking ability** (AUC-ROC). A model can have high AUC but poor calibration, or vice versa.

**Proper scoring rules** (like Brier score) quantify calibration honestly, not just ranking.

---

## Calibration: The Core Concept

### Example: Weather Forecasting

```
Model predictions on 100 days:
- Predicted 70% rain on 10 days → rained 8 times (80% observed)
- Predicted 50% rain on 20 days → rained 12 times (60% observed)
- Predicted 30% rain on 70 days → rained 18 times (26% observed)

Model is slightly miscalibrated: predicted 50-70%, observed 26-80%.
```

### Why Calibration Matters

**Insurance:** If your model predicts P(claim) = 0.05 but 10% of those customers actually claim, you've underpriced policies.

**Medical diagnosis:** If your model predicts P(disease) = 0.3 but 50% actually have it, patients get false reassurance.

**Credit:** If you predict P(default) = 0.02 but 5% actually default, risk is underestimated.

---

## Measuring Calibration

### 1. Reliability Diagram (Calibration Plot)

**Construction:**
1. Divide predictions into bins by predicted probability (e.g., [0.0-0.1), [0.1-0.2), ..., [0.9-1.0))
2. For each bin, plot:
   - **x-axis:** Mean predicted probability in bin
   - **y-axis:** Observed fraction of positives in bin
3. Draw diagonal (perfect calibration line)

**Interpretation:**
```
Observed Fraction
1.0 ┤         ╱ ← Perfect calibration
    │        ╱
    │      ╱ ← Model: well-calibrated
0.5 ┤     ╱╱
    │   ╱  ╲╲ ← Model: over-confident then under-confident
    │  ╱
0.0 ┣━━━━━━━ Mean Predicted Probability
    0  0.5  1.0
```

**Reading:**
- Points above diagonal: Model is overconfident (predicts low probability but event often occurs)
- Points below diagonal: Model is underconfident (predicts high probability but event rarely occurs)
- Points on diagonal: Perfect calibration for that bin

### 2. Expected Calibration Error (ECE)

$$\text{ECE} = \sum_{b=1}^B p_b \left| \text{acc}_b - \text{conf}_b \right|$$

Where:
- $B$ = number of bins
- $p_b$ = fraction of examples in bin $b$
- $\text{conf}_b$ = mean predicted probability in bin $b$
- $\text{acc}_b$ = fraction of actual positives in bin $b$

**Interpretation:**
- ECE = 0: Perfect calibration
- ECE > 0.1: Notably miscalibrated
- ECE < 0.05: Well-calibrated

**Example:**
```
Bin [0.4-0.6): 20 examples, mean prediction 0.5, 12 actual positives (acc=0.6)
Contribution: 0.20 * |0.6 - 0.5| = 0.02

Bin [0.7-0.9): 30 examples, mean prediction 0.8, 20 actual positives (acc=0.67)
Contribution: 0.30 * |0.67 - 0.8| = 0.039

Total ECE ≈ 0.059 (reasonably calibrated)
```

---

## Proper Scoring Rules

### Brier Score

The **Brier Score** is the mean squared error of probability predictions:

$$\text{BS} = \frac{1}{n} \sum_{i=1}^n (\hat{p}_i - y_i)^2$$

Where $y_i \in \{0, 1\}$ and $\hat{p}_i \in [0, 1]$.

**Interpretation:**
- BS = 0: Perfect (all predictions match reality)
- BS = 0.25: Useless (always predicting 0.5)
- BS = 1.0: Worst (always wrong and confident)

**Example:** Model predicts [0.9, 0.2, 0.8] on true labels [1, 0, 1]:
$$\text{BS} = \frac{1}{3}[(0.9-1)^2 + (0.2-0)^2 + (0.8-1)^2] = \frac{1}{3}[0.01 + 0.04 + 0.04] = 0.03$$

**Key property:** Brier score penalizes both **miscalibration** (being wrong) and **overconfidence** (being confidently wrong).

### Log Loss (Cross-Entropy)

$$\text{Log Loss} = -\frac{1}{n} \sum_{i=1}^n [y_i \log \hat{p}_i + (1-y_i) \log(1-\hat{p}_i)]$$

**Interpretation:**
- Measures surprise; lower is better
- Punishes confident wrong predictions heavily: $\log(0.01) \approx -4.6$ for $y=1$
- Used in training logistic regression; different from Brier score

**Relationship:**
- Brier score: uniformly penalizes error magnitude
- Log loss: exponentially penalizes confident errors
- For well-calibrated models, log loss ≈ $-\log(\text{BS} + \epsilon)$ (loose bound)

---

## Logistic Regression is Naturally Well-Calibrated

When trained with **binary cross-entropy loss** on sufficient data, logistic regression produces well-calibrated probabilities. This is a major advantage:

$$P(Y=1 | x; \theta) = \sigma(\theta^T x) \in (0, 1)$$

The sigmoid output directly represents probability of class 1. See [[Probability Output]].

---

## Recalibration Techniques

If a model is miscalibrated (e.g., due to class imbalance during training), you can recalibrate on a held-out calibration set:

### 1. Platt Scaling

Fit a logistic regression on the logits:
$$\hat{p}_{\text{cal}} = \sigma(a \cdot z + b)$$

Where $z = \theta^T x$ (model's logits), and fit $a, b$ on calibration set.

**Pros:** Simple, one additional feature
**Cons:** Assumes monotonic transformation of logits

### 2. Isotonic Regression

Non-parametric monotone recalibration. Maps logits to calibrated probabilities via piecewise-constant function.

**Pros:** More flexible than Platt scaling
**Cons:** Requires more data; can overfit on small calibration sets

### 3. Temperature Scaling

Divide logits by scalar $T > 0$:
$$\hat{p}_{\text{cal}} = \sigma(z / T)$$

**Interpretation:**
- $T = 1$: Original (no change)
- $T > 1$: Soften predictions (less confident)
- $T < 1$: Sharpen predictions (more confident)

**Pros:** Single hyperparameter; used in neural networks
**Cons:** Only adjusts confidence, not shape of calibration curve

---

## Calibration vs. Ranking: Independent Properties

**Critical insight:** High AUC ≠ Good calibration. They measure different things:

| | AUC-ROC | Calibration |
|---|---------|------------|
| **Measures** | Ranking ability | Probability accuracy |
| **Meaning** | P(model ranks positive > negative) | Do predicted probabilities match observed frequencies? |
| **Fix** | Threshold tuning won't help | Recalibration needed |
| **Example** | Model ranks 1000 positives > 1000 negatives: AUC = 1.0 | But predicts all positives P(+) = 0.6 when true rate is 0.7: miscalibrated |

**Decision tree:**
- High AUC, poor calibration → Recalibrate (Platt, isotonic, temperature scaling)
- Low AUC, good calibration → Model is fundamentally weak; no fix except redesign
- High AUC, good calibration → Ship it! ✅

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---------|---------|-----|
| Ignoring calibration entirely | Probabilities are wrong; decision-making fails | Plot reliability diagram; compute ECE |
| Assuming high AUC means good probabilities | AUC measures ranking, not probability accuracy | Evaluate calibration separately |
| Not checking calibration on test set | Recalibration on train set overfits | Reserve separate calibration set |
| Over-recalibrating | Too much recalibration causes overfitting | Use held-out calibration set; prefer simple methods |
| Confusing Brier score with log loss | Different penalties for error | Brier: uniform; Log loss: exponential |

---

## Interview Questions

**Q: What is calibration and why does it matter?**

Calibration measures whether predicted probabilities match observed frequencies. A well-calibrated model predicting P = 0.7 will have 70% of those cases actually positive. It matters for honest decision-making: in insurance, miscalibration leads to wrong pricing; in medicine, it leads to misguided treatment. Calibration is independent of ranking ability (AUC); you can have high AUC but poor calibration. See [[Probability Output]].

---

**Q: How do you measure calibration?**

**Reliability diagram:** Bin predictions by probability, plot mean predicted vs. observed fraction; points on diagonal are perfectly calibrated. **Expected Calibration Error (ECE):** Weighted average of bin calibration errors. **Brier Score:** Mean squared error of probabilities (BS = 0 is perfect). **Log Loss:** Cross-entropy penalty on confident errors (tighter). Logistic regression trained with cross-entropy is naturally well-calibrated. See [[Probability Output]].

---

**Q: What's the difference between Brier score and Log Loss?**

**Brier score** = mean squared error of probabilities; uniformly penalizes error magnitude. **Log Loss** = cross-entropy; exponentially penalizes confident errors (log of probability). Example: predicting 0.9 when truth is 0: Brier = 0.81, Log Loss = 2.3. Both measure calibration + confidence, but log loss is harsher on confident mistakes. Brier is easier to interpret (squared error); log loss is used in training. See [[Classification Metrics]].

---

**Q: Can a model have high AUC but poor calibration?**

Yes. AUC measures ranking (P(model ranks + > -)), not probability accuracy. Example: model predicts probabilities [0.6, 0.5, 0.4, 0.3, 0.2] for true labels [1, 1, 0, 0, 0]. Perfect ranking (AUC = 1.0) but calibration is wrong (predicts 55% positive on average, actual is 40%). They're independent properties. Fix: recalibrate with Platt scaling, isotonic regression, or temperature scaling. See [[Probability Output]].

---

**Q: If a model is miscalibrated, how do you fix it?**

Reserve a calibration set (separate from train/test). Fit a recalibration function on it: (1) **Platt scaling:** logistic regression on logits; simple, assumes monotonic. (2) **Isotonic regression:** non-parametric; more flexible. (3) **Temperature scaling:** divide logits by scalar T; used in deep learning. Fit on calibration set, apply to test set. Never recalibrate on training set (overfitting). See [[Probability Output]].

---

**Q: Why is logistic regression naturally well-calibrated?**

Logistic regression models P(Y=1|x) = σ(θ^T x) directly via MLE with binary cross-entropy loss. The sigmoid output is a proper probability, and training with cross-entropy enforces that predicted probabilities match observed frequencies. This is why logistic regression is so useful: not only does it classify, but its probability output is honest. See [[Probability Output]].

---

## Connections

- [[Probability Output]] — Related concept; recalibration methods detailed here
- [[Classification Metrics]] — Metrics assume hard labels; calibration works on soft
- [[Logistic Regression]] — Naturally well-calibrated
- [[Brier Score]] — Proper scoring rule for probabilities
- [[How to Choose Learning Rate]] — Training relates to calibration quality

---

## One-line Summary

> Calibration measures whether predicted probabilities match observed frequencies; it's independent of ranking ability (AUC), and can be fixed via Platt scaling, isotonic regression, or temperature scaling — assess with reliability diagrams and Brier score, remembering that logistic regression trained with cross-entropy is naturally well-calibrated.
