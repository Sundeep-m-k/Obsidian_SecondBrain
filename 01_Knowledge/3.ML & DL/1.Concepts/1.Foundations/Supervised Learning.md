# Supervised Learning

## What is it?

**Supervised Learning** is the most common type of [[Machine Learning]]. The system learns a mapping from inputs to outputs using a dataset of **labelled examples** — pairs $(x, y)$ where $x$ is the input and $y$ is the correct answer.

> "Supervised" = there is a **supervisor** (the labels) telling the model what the right answer is.

The goal: learn a function $\hat{f}$ such that $\hat{f}(x) \approx y$ for new, unseen $x$.

---

## Formal Setup

**Training dataset:**
$$\mathcal{D} = \{(x^{(1)}, y^{(1)}),\ (x^{(2)}, y^{(2)}),\ \ldots,\ (x^{(n)}, y^{(n)})\}$$

- $x^{(i)} \in \mathbb{R}^d$ — input feature vector with $d$ features
- $y^{(i)}$ — the label (continuous or categorical)
- $n$ — number of training examples

**Task:** Find parameters $\theta$ such that:
$$\hat{f}(x;\theta) \approx y \quad \text{for new, unseen } x$$

This is done by minimising a **loss function** over the training set:
$$\hat{\theta} = \arg\min_{\theta} \frac{1}{n} \sum_{i=1}^{n} \mathcal{L}(y^{(i)},\ \hat{f}(x^{(i)};\theta))$$

---

## The Two Tasks

### 1. Regression
- $y$ is **continuous** (a real number)
- Goal: predict a quantity
- Examples: house price, stock return, temperature forecast, patient age

**Standard loss — Mean Squared Error (MSE):**
$$\mathcal{L}_{\text{MSE}} = \frac{1}{n}\sum_{i=1}^{n}(y^{(i)} - \hat{y}^{(i)})^2$$

### 2. Classification
- $y$ is **discrete** (a category)
- Goal: assign a class label
- Examples: spam/not-spam, digit recognition, disease diagnosis

**Standard loss — Cross-Entropy:**

Binary (two classes):
$$\mathcal{L}_{\text{BCE}} = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log\hat{p}^{(i)} + (1 - y^{(i)})\log(1 - \hat{p}^{(i)})\right]$$

Multi-class ($K$ classes):
$$\mathcal{L}_{\text{CE}} = -\frac{1}{n}\sum_{i=1}^{n}\sum_{k=1}^{K} y_k^{(i)} \log \hat{p}_k^{(i)}$$

where $\hat{p}_k^{(i)}$ is the predicted probability of class $k$ for example $i$.

---

## Key Algorithms

### Linear Regression
Assume $y$ is a linear function of $x$:
$$\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \cdots + \theta_d x_d = \theta^T x$$

(absorbing bias into $\theta$ with $x_0 = 1$)

**Closed-form solution (Normal Equations):**
$$\hat{\theta} = (X^T X)^{-1} X^T y$$

Where $X$ is the $n \times d$ design matrix (rows = examples, columns = features).

**Geometric interpretation:** Finds the hyperplane that minimises the sum of squared vertical distances to all data points.

**When to use:** Output is continuous, relationship is approximately linear, data is not huge.

---

### Logistic Regression (Binary Classification)
Despite the name, this is a **classifier**, not a regressor.

Model the probability of class 1:
$$\hat{p} = \sigma(\theta^T x) = \frac{1}{1 + e^{-\theta^T x}}$$

The **sigmoid function** $\sigma(z) = \frac{1}{1+e^{-z}}$ squashes any real number into $(0, 1)$.

**Decision boundary:** Predict class 1 if $\hat{p} \geq 0.5$, i.e., if $\theta^T x \geq 0$.

The decision boundary is a **linear hyperplane** in feature space.

Trained by minimising Binary Cross-Entropy (see above) using gradient descent.

**Multi-class extension (Softmax Regression):**
$$\hat{p}_k = \frac{e^{\theta_k^T x}}{\sum_{j=1}^{K} e^{\theta_j^T x}}$$

This is the **softmax** function — ensures all class probabilities sum to 1.

---

### Decision Trees
Split the input space into regions using a series of if-then rules.

**Training:** Greedily choose the split that maximally reduces impurity.

**Gini Impurity** (used in CART):
$$G = 1 - \sum_{k=1}^{K} p_k^2$$

where $p_k$ = fraction of class $k$ in the node.

**Information Gain** (used in ID3/C4.5):
$$IG = H(\text{parent}) - \sum_{\text{children}} \frac{n_{\text{child}}}{n_{\text{parent}}} H(\text{child})$$

where entropy $H = -\sum_k p_k \log_2 p_k$.

At each node: choose the feature and threshold that maximises the information gain (or minimises Gini impurity).

**Strengths:** Interpretable, handles mixed types, no feature scaling needed.  
**Weaknesses:** Prone to overfitting (deep trees memorise data). Fix: prune the tree, or use ensembles.

---

### Random Forest
An **ensemble** of decision trees, trained on bootstrapped samples of the data and random subsets of features. Predictions are aggregated by majority vote (classification) or averaging (regression).

**Key ideas:**
- **Bagging:** Each tree is trained on a bootstrap sample (sample with replacement).
- **Feature randomisation:** At each split, only consider a random subset of features (typically $\sqrt{d}$ for classification).
- **Aggregation:** Reduces variance dramatically compared to a single tree.

---

### Support Vector Machine (SVM)

For binary classification, find the hyperplane that maximises the **margin** — the distance between the hyperplane and the nearest data points from each class.

The decision boundary: $\theta^T x + b = 0$

Points on the margin: $\theta^T x + b = \pm 1$

**Margin width:** $\frac{2}{\|\theta\|}$

**Optimisation problem (hard margin):**
$$\min_{\theta, b} \frac{1}{2}\|\theta\|^2 \quad \text{subject to } y^{(i)}(\theta^T x^{(i)} + b) \geq 1 \quad \forall i$$

**Soft margin (real data with noise):**
$$\min_{\theta, b} \frac{1}{2}\|\theta\|^2 + C\sum_{i=1}^{n} \xi_i$$

where $\xi_i$ are slack variables allowing misclassification, and $C$ controls the tradeoff.

**Kernel trick:** For non-linearly separable data, implicitly map features to a high-dimensional space:
$$K(x, x') = \phi(x)^T \phi(x')$$

Common kernels:
- Linear: $K(x,x') = x^T x'$
- RBF (Gaussian): $K(x,x') = \exp\left(-\frac{\|x-x'\|^2}{2\sigma^2}\right)$
- Polynomial: $K(x,x') = (x^T x' + c)^d$

---

### k-Nearest Neighbours (kNN)

**No training phase.** At inference time, find the $k$ closest training examples and:
- **Classification:** majority vote of their labels
- **Regression:** average of their values

**Distance metric (usually Euclidean):**
$$d(x, x') = \sqrt{\sum_{j=1}^{d}(x_j - x'_j)^2}$$

$k=1$: overfit. $k=n$: always predict the majority class. Choose $k$ via cross-validation.

**Curse of dimensionality:** In high dimensions, all points are roughly equidistant. kNN degrades in high-dimensional spaces.

---

## Probabilistic View: Bayes Optimal Classifier

The theoretically best possible classifier minimises expected loss. For 0-1 loss (misclassification rate):

$$f^*(x) = \arg\max_k P(Y = k \mid X = x)$$

This is called the **Bayes optimal classifier**. Its error rate is the **Bayes error** (irreducible error). No classifier can do better.

In practice, we don't know $P(Y \mid X)$ — we estimate it from data.

---

## The Problem of Overfitting

A model that perfectly fits training data but fails on new data is **overfitting**.

Diagnostics:
- Training loss is low but validation loss is high.
- Large gap between train accuracy and validation accuracy.

Solutions:
- More training data
- Regularisation (L1/L2, dropout)
- Simpler model
- Data augmentation
- Early stopping (stop training when validation loss stops improving)

---

## Assumptions to Check

| Algorithm | Key Assumptions |
|---|---|
| Linear Regression | Linearity, independence of errors, homoscedasticity, normality of residuals |
| Logistic Regression | Linear decision boundary in feature space, independent examples |
| Naive Bayes | Features are conditionally independent given the label |
| SVM | Optimal hyperplane exists; kernel choice matters |

---

## Connections

- [[Machine Learning]] — the parent framework
- [[Training Data]] — the labelled examples used
- [[Features]] — the $x$ inputs
- [[Labels]] — the $y$ targets
- [[Model]] — the function being learned
- [[Generalization]] — whether the model works on unseen data
- [[Unsupervised Learning]] — the contrast: no labels

---

## One-line Summary

> Supervised learning is learning by example: given many $(input, correct\text{ }answer)$ pairs, find a function that produces the right answer for inputs it has never seen before.
