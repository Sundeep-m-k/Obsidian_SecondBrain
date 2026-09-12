# Logistic Loss

## What is it?

**Logistic Loss** (also called **Log Loss**, **Binary Cross-Entropy**, or **Cross-Entropy Loss**) is the standard [[Loss Function]] for binary [[Classification]] tasks. It measures how far the predicted probability $\hat{p}$ is from the true label $y \in \{0, 1\}$.

$$\ell(y, \hat{p}) = -\left[y\log\hat{p} + (1-y)\log(1-\hat{p})\right]$$

---

## Case-by-Case Behaviour

**When true label $y = 1$:**
$$\ell = -\log\hat{p}$$

| $\hat{p}$ | Loss |
|---|---|
| 0.99 (confident correct) | $-\log(0.99) \approx 0.01$ (tiny loss) |
| 0.5 (uncertain) | $-\log(0.5) \approx 0.69$ |
| 0.01 (confident wrong) | $-\log(0.01) \approx 4.6$ (large loss) |

**When true label $y = 0$:**
$$\ell = -\log(1-\hat{p})$$

| $\hat{p}$ | Loss |
|---|---|
| 0.01 (confident correct) | $-\log(0.99) \approx 0.01$ (tiny loss) |
| 0.5 (uncertain) | $-\log(0.5) \approx 0.69$ |
| 0.99 (confident wrong) | $-\log(0.01) \approx 4.6$ (large loss) |

**Key insight:** Logistic loss **heavily penalises confident wrong predictions** via the $-\log$ function, which approaches $\infty$ as $\hat{p} \to 0$ (when true label is 1).

---

## Cost Function for Logistic Regression

$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

Where $\hat{p}^{(i)} = \sigma(\theta^T x^{(i)}) = \frac{1}{1+e^{-\theta^T x^{(i)}}}$.

---

## Statistical Derivation: Maximum Likelihood

Assume $y^{(i)} \mid x^{(i)} \sim \text{Bernoulli}(\hat{p}^{(i)})$.

**Likelihood:**
$$\mathcal{L}(\theta) = \prod_{i=1}^{n} \hat{p}^{(i)^{y^{(i)}}} (1-\hat{p}^{(i)})^{1-y^{(i)}}$$

**Negative log-likelihood:**
$$-\log \mathcal{L}(\theta) = -\sum_{i=1}^{n}\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

Dividing by $n$ gives $J(\theta)$. **Minimising logistic loss = MLE for a Bernoulli model.** This is why it is the theoretically correct loss for binary classification.

---

## Gradient

$$\nabla_\theta J(\theta) = \frac{1}{n}\sum_{i=1}^{n}(\hat{p}^{(i)} - y^{(i)})x^{(i)} = \frac{1}{n}X^T(\hat{p} - y)$$

**Strikingly**, the gradient has the same form as for MSE in linear regression: $\frac{1}{n}\sum(\text{prediction} - \text{target}) \times \text{input}$. This is not a coincidence — it follows from the sigmoid being the natural inverse link for the Bernoulli family.

---

## Why Not Use MSE for Classification?

If you use $J(\theta) = \frac{1}{n}\sum(\hat{p}^{(i)} - y^{(i)})^2$ with sigmoid output:

1. The cost surface becomes **non-convex** — gradient descent may get stuck.
2. The gradient **vanishes** when $\hat{p}$ is near 0 or 1 (sigmoid is flat there + squared error is small), so learning stops even when wrong.
3. Log loss avoids both: it's convex and the $-\log$ gradient is large when confident and wrong.

---

## Connections

- [[Loss Function]] — logistic loss is the classification-specific loss
- [[Cost Function]] — average logistic loss over training set
- [[Logistic Regression]] — trained by minimising logistic loss
- [[Binary Classification]] — the task this loss is designed for
- [[Sigmoid Function]] — $\hat{p} = \sigma(\theta^T x)$

---

## One-line Summary

> Logistic loss is the correct loss for binary classification — derived from maximum likelihood under the Bernoulli model, it heavily penalises confident wrong predictions via the $-\log$ function and produces a convex cost surface for logistic regression.
