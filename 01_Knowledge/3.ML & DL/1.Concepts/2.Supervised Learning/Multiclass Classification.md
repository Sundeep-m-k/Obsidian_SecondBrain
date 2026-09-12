# Multiclass Classification

## What is it?

**Multiclass Classification** (also called **multinomial classification**) is a [[Classification]] task where each input is assigned to exactly one of $K > 2$ **mutually exclusive** classes.

$$y \in \{1, 2, \ldots, K\}$$

---

## Examples

| Problem | Classes |
|---|---|
| Handwritten digit recognition | 0, 1, 2, …, 9 ($K=10$) |
| Language identification | English, French, Spanish, … ($K=100+$) |
| Image classification (ImageNet) | 1000 object categories ($K=1000$) |
| Medical diagnosis | Healthy, Disease A, Disease B, Disease C |
| News topic | Politics, Sports, Tech, Entertainment |

---

## Output Representation

### Softmax Output
The model produces a vector of logits $z \in \mathbb{R}^K$, then converts to probabilities via **softmax**:

$$P(Y = k \mid x) = \frac{e^{z_k}}{\sum_{j=1}^{K} e^{z_j}}, \quad k = 1, \ldots, K$$

**Properties:**
- All outputs in $(0, 1)$
- Sum to exactly 1: $\sum_{k=1}^K P(Y=k \mid x) = 1$
- Invariant to constant shifts: $\text{softmax}(z + c) = \text{softmax}(z)$

**Prediction:** $\hat{y} = \arg\max_k P(Y=k \mid x) = \arg\max_k z_k$

### One-Hot Label Encoding

True label $y = c$ (class $c$) is encoded as a vector:
$$\mathbf{y} = [0, \ldots, 0, \underbrace{1}_{c\text{-th position}}, 0, \ldots, 0] \in \{0,1\}^K$$

---

## Loss Function: Categorical Cross-Entropy

$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n}\sum_{k=1}^{K} y_k^{(i)} \log P(Y=k \mid x^{(i)})$$

With one-hot labels, only the true class term survives:
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n} \log P(Y = c^{(i)} \mid x^{(i)})$$

Minimising this = **maximising the log-probability of the correct class** = MLE under a categorical distribution.

**Numerical stability:** Never compute $\exp(z_k)$ directly for large $z_k$ — overflow. Use the **log-sum-exp trick**:
$$\log\sum_{j} e^{z_j} = z_{\max} + \log\sum_{j} e^{z_j - z_{\max}}$$

---

## Reduction to Binary Classification

Two classic strategies to reduce multiclass to binary:

### One-vs-Rest (OvR) / One-vs-All (OvA)
Train $K$ binary classifiers. Classifier $k$ distinguishes class $k$ from all others.

$$\hat{f}_k(x) = P(Y = k \mid x) \quad \text{(binary classifier)}$$

**Predict:** $\hat{y} = \arg\max_k \hat{f}_k(x)$

**Problem:** Each classifier sees highly imbalanced data (1 class vs. all others).

### One-vs-One (OvO)
Train $\binom{K}{2} = \frac{K(K-1)}{2}$ binary classifiers, one for each pair of classes.

**Predict:** Majority vote among all pairwise classifiers.

**Advantage:** Each classifier sees balanced data (2 classes only).  
**Disadvantage:** $O(K^2)$ classifiers — expensive for large $K$.

---

## Native Multiclass Methods

Many algorithms handle multiclass directly without reduction:

| Algorithm | Multiclass strategy |
|---|---|
| **Softmax Regression** | Direct: one weight vector per class |
| **Decision Trees / RF** | Direct: leaf node predicts distribution |
| **kNN** | Direct: majority vote among $k$ neighbours |
| **Naive Bayes** | Direct: apply Bayes' rule for each class |
| **Neural Networks** | Direct: $K$-way softmax output layer |
| **SVM** | Typically OvO or OvR |

---

## Softmax Regression (Multinomial Logistic Regression)

For each class $k$, learn a weight vector $\theta_k \in \mathbb{R}^d$:

$$P(Y = k \mid x) = \frac{e^{\theta_k^T x}}{\sum_{j=1}^{K} e^{\theta_j^T x}}$$

**Parameters:** $\Theta \in \mathbb{R}^{K \times d}$ (one row per class).

**Decision boundary** between class $j$ and class $k$:
$$\theta_j^T x = \theta_k^T x \quad \Rightarrow \quad (\theta_j - \theta_k)^T x = 0$$

This is a **linear boundary** — Softmax Regression is a linear classifier.

**Identifiability:** One weight vector is redundant (adding a constant to all $\theta_k$ doesn't change predictions). Convention: fix $\theta_K = 0$ (the last class as reference). This reduces parameters from $K \times d$ to $(K-1) \times d$.

---

## Evaluation Metrics

### Accuracy
$$\text{Accuracy} = \frac{\text{Number of correct predictions}}{n}$$

Misleading for imbalanced classes.

### Per-class Metrics
Compute binary metrics for each class $k$ using OvR framing:

$$\text{Precision}_k = \frac{TP_k}{TP_k + FP_k}, \quad \text{Recall}_k = \frac{TP_k}{TP_k + FN_k}, \quad F_{1,k} = 2 \cdot \frac{P_k R_k}{P_k + R_k}$$

### Macro Average
$$\text{Macro-}F_1 = \frac{1}{K}\sum_{k=1}^{K} F_{1,k}$$
Treats all classes equally. Sensitive to rare-class performance.

### Weighted Average
$$\text{Weighted-}F_1 = \sum_{k=1}^{K} \frac{n_k}{n} F_{1,k}$$
Weighted by class frequency.

### Confusion Matrix ($K \times K$)
Entry $(i,j)$ = number of examples of true class $i$ predicted as class $j$. Diagonal = correct predictions. Off-diagonal = mistakes.

---

## Label Smoothing

Instead of hard one-hot labels, use soft targets:
$$y_k^{\text{smooth}} = \begin{cases} 1 - \varepsilon & k = c \text{ (true class)} \\ \varepsilon/(K-1) & k \neq c \end{cases}$$

Typically $\varepsilon = 0.1$. Prevents the model from becoming overconfident. Acts as regularisation. Improves calibration.

---

## Connections

- [[Classification]] — multiclass is the general $K>2$ case
- [[Binary Classification]] — the $K=2$ special case
- [[Logistic Regression]] — generalises to softmax for multiclass
- [[Decision Boundary]] — $K$ classes create $K(K-1)/2$ pairwise boundaries
- [[Loss Function]] — categorical cross-entropy

---

## One-line Summary

> Multiclass classification assigns each input to one of $K$ mutually exclusive classes, typically via a softmax output that produces a probability distribution over all classes, trained by minimising categorical cross-entropy.
