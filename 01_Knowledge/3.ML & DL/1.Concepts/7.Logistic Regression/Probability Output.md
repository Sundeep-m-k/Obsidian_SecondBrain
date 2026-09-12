# Probability Output

## What is it?

The **probability output** of logistic regression is the predicted probability that input $x$ belongs to class 1:

$$\hat{p} = P(Y=1 \mid x;\theta) = \sigma(\theta^T x) = \frac{1}{1+e^{-\theta^T x}} \in (0,1)$$

This is a **soft prediction** — a continuous value that encodes the model's confidence, as opposed to a hard 0/1 class label.

---

## From Probability to Class Label

The probability is converted to a hard label via thresholding:
$$\hat{y} = \mathbf{1}[\hat{p} \geq \tau]$$

Default: $\tau = 0.5$.

**Interpreting $\hat{p}$:**
- $\hat{p} = 0.9$: model is 90% confident this is class 1.
- $\hat{p} = 0.51$: barely above threshold — low confidence.
- $\hat{p} = 0.02$: model strongly predicts class 0.

---

## Calibration

A model is **well-calibrated** if its predicted probabilities match empirical frequencies:

> When a calibrated model predicts $\hat{p} = 0.7$, approximately 70% of such examples are actually class 1.

**Reliability diagram:** Plot mean predicted probability vs. actual fraction of positives in bins. A perfectly calibrated model lies on the diagonal.

**Logistic regression is naturally well-calibrated** when trained with cross-entropy loss on enough data.

**Recalibration (if needed):**
- **Platt scaling:** Fit logistic regression on logits of a calibration set: $p_{\text{cal}} = \sigma(a \cdot z + b)$.
- **Isotonic regression:** Non-parametric monotone recalibration.
- **Temperature scaling:** Divide logits by scalar $T$: $\hat{p}_k = \sigma(z/T)$.

See [[Calibration]] for the fuller treatment — the Expected Calibration Error formula, a worked numeric example, and why good ROC-AUC does *not* imply good calibration (they're mathematically independent properties).

---

## Connections

- [[Sigmoid Function]] — produces the probability
- [[Logistic Regression]] — the model
- [[Thresholding]] — converts probability to class label
- [[Binary Classification]] — the task
- [[Calibration]] — the full formal treatment of what's introduced above
- [[Brier Score]], [[Proper Scoring Rule]] — how to score a probability output honestly, not just its ranking ability

---

## One-line Summary

> The probability output of logistic regression is a calibrated estimate of class membership in $(0,1)$, produced by the sigmoid function — it encodes confidence, not just class, enabling threshold tuning for different cost structures.
