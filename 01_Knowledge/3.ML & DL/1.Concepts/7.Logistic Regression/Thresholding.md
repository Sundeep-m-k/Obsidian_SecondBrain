# Thresholding

## What is it?

**Thresholding** converts the continuous probability output $\hat{p} \in (0,1)$ of a classifier into a discrete class label $\hat{y} \in \{0,1\}$.

$$\hat{y} = \begin{cases}1 & \text{if } \hat{p} \geq \tau \\ 0 & \text{if } \hat{p} < \tau\end{cases}$$

$\tau$ is the **decision threshold** (default: $\tau = 0.5$).

---

## Why Tune the Threshold?

The default $\tau = 0.5$ is rarely optimal. The best threshold depends on the **relative cost of FP vs. FN**:

| Application | Cost asymmetry | Adjust threshold |
|---|---|---|
| Cancer screening | FN (missing cancer) far worse than FP | Lower $\tau$ (e.g., 0.2) → higher recall |
| Spam filter | FP (blocking legit email) worse than FN | Higher $\tau$ (e.g., 0.7) → higher precision |
| Fraud detection | FN (missed fraud) worse | Lower $\tau$ |
| Drug approval | FP (false efficacy) worse | Higher $\tau$ |

---

## Effect of Changing $\tau$

As $\tau$ decreases (more lenient):
- More examples classified as class 1
- Recall ↑ (catch more true positives)
- Precision ↓ (more false positives)
- FPR ↑

As $\tau$ increases (more strict):
- Fewer examples classified as class 1
- Recall ↓
- Precision ↑
- FPR ↓

The **ROC curve** plots (FPR, TPR) = (FPR($\tau$), Recall($\tau$)) for all $\tau \in [0,1]$.

The **Precision-Recall curve** plots (Recall($\tau$), Precision($\tau$)) for all $\tau$.

---

## Choosing the Optimal Threshold

**Method 1: Youden's J statistic**
$$\tau^* = \arg\max_\tau (TPR(\tau) - FPR(\tau)) = \arg\max_\tau J$$
Maximises the gap between true and false positive rates.

**Method 2: F1 maximisation**
$$\tau^* = \arg\max_\tau F_1(\tau)$$

**Method 3: Cost-based**
$$\tau^* = \arg\min_\tau \left[ c_{FP} \cdot FP(\tau) + c_{FN} \cdot FN(\tau) \right]$$

where $c_{FP}, c_{FN}$ are the costs of each error type.

The theoretical optimal threshold (with known costs):
$$\tau^* = \frac{c_{FP}}{c_{FP} + c_{FN}}$$

---

## Connections

- [[Probability Output]] — the value being thresholded
- [[Binary Classification]] — the output after thresholding
- [[Decision Boundary in Logistic Regression]] — the hyperplane where $\hat{p} = \tau$
- [[Logistic Regression]] — the model

---

## One-line Summary

> Thresholding converts a predicted probability into a class label by comparing it to a cutoff $\tau$ — choosing the right threshold (not the default 0.5) is critical and depends on the relative cost of false positives vs. false negatives in the application.
