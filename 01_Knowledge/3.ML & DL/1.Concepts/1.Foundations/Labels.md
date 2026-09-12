## What is it?

**Labels** (also called **targets**, **outputs**, **response variables**, **ground truth**, or **annotations**) are the correct answers or desired outputs that a model learns to predict in [[Supervised Learning]].

$$\text{Each training example: } (x^{(i)},\ \underbrace{y^{(i)}}_{\text{label}})$$

- $x^{(i)}$ = the features (what we observe)
- $y^{(i)}$ = the label (what we want the model to predict)

---

## Types of Labels

### By output type:

| Type | Description | Space | Examples |
|---|---|---|---|
| **Binary label** | Two classes | $y \in \{0, 1\}$ or $\{-1, +1\}$ | Spam/not-spam, fraud/legit |
| **Multi-class label** | $K > 2$ mutually exclusive classes | $y \in \{1, 2, \ldots, K\}$ | Digit (0-9), species, sentiment (pos/neg/neutral) |
| **Continuous label** | Real-valued target | $y \in \mathbb{R}$ | House price, temperature, age |
| **Multi-label** | Multiple classes can be simultaneously true | $y \in \{0,1\}^K$ | Tags on an article (can be both "sports" AND "health") |
| **Ordinal label** | Ordered discrete categories | $y \in \{1,2,3,4,5\}$ | Star ratings, severity scores |
| **Structured label** | Complex structure | Sequence, tree, graph | Translation (output sentence), parse tree, bounding box |
| **Probability / Distribution** | Soft targets | $y \in [0,1]^K$, $\sum y_k = 1$ | Knowledge distillation |

---

## Label Encoding

Models work with numbers. Labels must be encoded numerically.

### Binary Classification
$$y \in \{0, 1\} \quad \text{or} \quad y \in \{-1, +1\}$$

Use $\{0, 1\}$ for logistic regression and neural nets (matches the sigmoid output).  
Use $\{-1, +1\}$ for SVMs (matches the hinge loss formulation).

### Multi-class Classification

**Integer encoding:** $y \in \{0, 1, \ldots, K-1\}$. Simple but implies an ordinal relationship.

**One-hot encoding:** $y \in \{0,1\}^K$, exactly one entry is 1.

Example with $K=3$ classes:
- Class 0 → $[1, 0, 0]$
- Class 1 → $[0, 1, 0]$
- Class 2 → $[0, 0, 1]$

One-hot is used with **cross-entropy loss**, which requires a probability distribution over classes.

### Regression
$y \in \mathbb{R}$. No encoding needed. Scale if necessary (e.g., normalise target to $[0,1]$ or standardise to zero mean).

---

## The Role of Labels in Loss Functions

Labels define the **ground truth** against which predictions are compared.

### For regression (MSE):
$$\mathcal{L} = \frac{1}{n}\sum_{i=1}^{n}(y^{(i)} - \hat{y}^{(i)})^2$$

The label $y^{(i)}$ is the target value. The loss penalises predictions $\hat{y}^{(i)}$ that are far from it.

### For binary classification (Binary Cross-Entropy):
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log \hat{p}^{(i)} + (1 - y^{(i)})\log(1 - \hat{p}^{(i)})\right]$$

When $y^{(i)} = 1$: loss = $-\log \hat{p}^{(i)}$ → penalises low predicted probability for the positive class.  
When $y^{(i)} = 0$: loss = $-\log(1 - \hat{p}^{(i)})$ → penalises high predicted probability for the positive class.

### For multi-class classification (Categorical Cross-Entropy):
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n}\sum_{k=1}^{K} y_k^{(i)} \log \hat{p}_k^{(i)}$$

With one-hot labels, only the term for the true class $c$ survives:
$$\mathcal{L} = -\frac{1}{n}\sum_{i=1}^{n} \log \hat{p}_{c^{(i)}}^{(i)}$$

This makes intuitive sense: maximise the predicted probability of the true class.

---

## How Labels Are Obtained

### Human Annotation
The most common source. Humans look at each example and assign the label.

**Quality control methods:**
- **Inter-annotator agreement:** measure consistency between annotators using Cohen's Kappa:
$$\kappa = \frac{p_o - p_e}{1 - p_e}$$
where $p_o$ = observed agreement, $p_e$ = expected agreement by chance.  
$\kappa > 0.8$ is considered strong agreement.

- **Majority vote:** show each example to multiple annotators, take the majority.
- **Gold standard questions:** include known answers to catch careless annotators.

**Crowdsourcing platforms:** Amazon Mechanical Turk, Scale AI, Appen.

### Programmatic / Weak Labelling
Use heuristics, rules, or knowledge bases to generate noisy labels automatically.

**Data Programming (Snorkel):** Define **Labelling Functions (LFs)** — simple heuristic rules — and combine them:

$$\lambda_j(x) \in \{-1, 0, 1\} \quad \text{(abstain, negative, positive)}$$

Multiple LFs are combined via a **label model** that estimates the accuracy of each LF and resolves conflicts:

$$p(y = 1 \mid \lambda_1, \ldots, \lambda_m) = \text{Label Model}(\lambda_1, \ldots, \lambda_m)$$

The output is a **probabilistic label** — a soft label in $[0,1]$ rather than a hard 0/1.

### Distant Supervision
Use existing structured knowledge (databases, ontologies) to generate labels:
- "This sentence mentions [entity1] and [entity2] which have relation [R] in the knowledge base → label this sentence as expressing relation [R]."

Noisy, but can produce large datasets cheaply.

### Self-supervision
Labels derived automatically from the data structure itself:
- **Language modelling:** label for token $t$ is the next token $t+1$
- **Masked LM (BERT):** label is the masked token
- **Contrastive:** label is "same image" (positive pair) or "different image" (negative pair)

---

## Label Noise

**Label noise** = incorrect labels in the training set. It is common in practice, especially with crowdsourced annotations.

**Types:**
- **Random noise:** Labels are flipped randomly with probability $\epsilon$. Also called Class-Conditional Label Noise.
- **Systematic / Adversarial noise:** Labels are wrong in a structured way (e.g., a specific annotator is consistently wrong).

**Effect on training:**
- With noise rate $\epsilon$, the optimal achievable accuracy is approximately $1 - \epsilon$ (you can't do better than your labels).
- For small $\epsilon$, most algorithms are relatively robust.
- For large $\epsilon$ (>20%), standard ERM breaks down.

**Noise transition matrix:**
$$T_{ij} = P(\tilde{y} = j \mid y = i)$$

Where $y$ is the true label and $\tilde{y}$ is the noisy observed label. Clean class probabilities:
$$p(\tilde{y} = j \mid x) = \sum_i T_{ij} \cdot p(y = i \mid x)$$

**Dealing with label noise:**
- **Label smoothing:** instead of hard 0/1 labels, use $\epsilon_{\text{smooth}} = 0.1$:
$$y_k^{\text{smooth}} = y_k (1 - \epsilon_{\text{smooth}}) + \frac{\epsilon_{\text{smooth}}}{K}$$
This prevents overconfident predictions and acts as regularisation.

- **Loss correction:** estimate the noise transition matrix $T$ and correct the loss
- **Robust losses:** e.g., MAE is more robust to noise than MSE; Generalised Cross-Entropy (GCE)
- **Confident learning (CleanLab):** identify likely mislabelled examples using model's predicted probabilities

---

## Class Imbalance

When some classes have far more training examples than others.

**Example:** 99% negative (no fraud), 1% positive (fraud). A model that always predicts negative achieves 99% accuracy but is useless.

**Metrics for imbalanced classification:**
- **Precision, Recall, F1-score** (prefer over accuracy)
- **ROC-AUC and PR-AUC** (area under precision-recall curve — better for severe imbalance)

**Solutions:**

| Method | Description |
|---|---|
| **Oversampling (SMOTE)** | Synthetically generate new minority class examples by interpolating between existing ones |
| **Undersampling** | Remove majority class examples |
| **Class weights** | Weight the loss by $w_k = n / (K \cdot n_k)$ — minority class gets higher weight |
| **Threshold adjustment** | Tune the decision threshold from default 0.5 to better suit the cost structure |

**SMOTE (Synthetic Minority Over-sampling Technique):**
For each minority class example $x^{(i)}$:
1. Find its $k$ nearest neighbours in the minority class.
2. Randomly select one neighbour $x^{(j)}$.
3. Generate: $x^{\text{new}} = x^{(i)} + \lambda (x^{(j)} - x^{(i)})$ where $\lambda \sim \text{Uniform}(0,1)$.

---

## Label Leakage

**Data leakage** is when the label (or information derived from it) accidentally appears in the features during training.

**Example:** You're predicting whether a patient will be diagnosed with cancer. If you include "received chemotherapy" as a feature, you have leakage — that feature is caused by the diagnosis, not the other way around.

Leakage causes the model to appear to perform perfectly during training and evaluation, but fail completely at deployment.

**Prevention:**
- Use domain knowledge to identify temporally or causally impossible features.
- Simulate deployment conditions: only use information that would genuinely be available at prediction time.

---

## Connections

- [[Machine Learning]] — labels are the supervisory signal
- [[Supervised Learning]] — requires labels for every training example
- [[Training Data]] — dataset consists of (feature, label) pairs
- [[Features]] — the $x$ half of the training pair
- [[Model]] — learns to predict labels from features
- [[Inference]] — at inference time, the label is unknown; we predict it
- [[Unsupervised Learning]] — no labels; the contrast

---

## One-line Summary

> Labels are the ground truth answers that tell the model what it should have predicted — they are the supervisory signal that drives learning, and their quality, quantity, and balance fundamentally constrain what any supervised model can achieve.
