# Logistic Regression

## What is it?

**Logistic Regression** is the standard algorithm for **binary classification**. Despite its name, it is a **classifier**, not a regressor. It models the probability that an input $x$ belongs to class 1.

$$P(Y=1 \mid x) = \sigma(\theta^T x) = \frac{1}{1 + e^{-\theta^T x}}$$

Where $\sigma$ is the **sigmoid function**.

---

## Motivation: Why Not Linear Regression for Classification?

If we use linear regression to predict $y \in \{0, 1\}$:
- Predictions are unbounded ($\hat{y}$ can be < 0 or > 1) — not valid probabilities.
- MSE cost is non-convex with sigmoid — hard to optimise.
- The model is sensitive to outliers in feature space.

Logistic regression fixes this by squashing the linear combination through the sigmoid:
$$\underbrace{\theta^T x}_{\text{linear score}} \xrightarrow{\sigma} \underbrace{\hat{p} \in (0,1)}_{\text{probability}}$$

---

## The Model

**Step 1:** Compute a linear score (logit):
$$z = \theta^T x = \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d$$

**Step 2:** Squash through sigmoid:
$$\hat{p} = \sigma(z) = \frac{1}{1 + e^{-z}}$$

**Step 3:** Threshold to get a class label:
$$\hat{y} = \begin{cases}1 & \hat{p} \geq 0.5 \\ 0 & \hat{p} < 0.5\end{cases}$$

Default threshold 0.5 corresponds to $z = 0$, i.e., $\theta^T x = 0$.

---

## The Decision Boundary

The decision boundary is where $\hat{p} = 0.5$, i.e., $z = 0$:
$$\theta^T x = 0 \quad \Leftrightarrow \quad \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d = 0$$

This is a **linear hyperplane** in $\mathbb{R}^d$. Logistic regression is a **linear classifier**.

For non-linear boundaries: add polynomial features $x_1^2, x_2^2, x_1 x_2$, etc.

---

## Cost Function: Binary Cross-Entropy

Using MSE for classification causes a non-convex cost surface. The correct loss is the **binary cross-entropy** (logistic loss):

$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

This is **convex** in $\theta$ → gradient descent guaranteed to find the global minimum.

Derivation from MLE: see [[Logistic Loss]].

---

## Gradient of the Cost

$$\nabla_\theta J(\theta) = \frac{1}{n}\sum_{i=1}^{n}(\hat{p}^{(i)} - y^{(i)})x^{(i)} = \frac{1}{n}X^T(\hat{p} - y)$$

Where $\hat{p} = \sigma(X\theta) \in \mathbb{R}^n$.

**Update rule:**
$$\theta \leftarrow \theta - \frac{\eta}{n}X^T(\hat{p} - y)$$

This has the same elegant form as linear regression — (prediction error) × (input).

---

## Probabilistic Interpretation

Logistic regression models:
$$P(Y=1 \mid x; \theta) = \sigma(\theta^T x)$$
$$P(Y=0 \mid x; \theta) = 1 - \sigma(\theta^T x) = \sigma(-\theta^T x)$$

Compactly: $P(Y=y \mid x;\theta) = \hat{p}^y(1-\hat{p})^{1-y}$ for $y \in \{0,1\}$.

This is a **Bernoulli distribution** parameterised by $\hat{p}$. Training = MLE of $\theta$.

---

## Log-Odds Interpretation

The **log-odds** (logit) of class 1 is linear in features:
$$\log\frac{\hat{p}}{1-\hat{p}} = \theta^T x$$

Where $\frac{\hat{p}}{1-\hat{p}}$ is the **odds ratio** (how much more likely class 1 is than class 0).

**Coefficient interpretation:**
- A unit increase in $x_j$ multiplies the odds by $e^{\theta_j}$.
- If $\theta_j = 0.5$: $e^{0.5} \approx 1.65$ — each unit increase makes class 1 65% more likely (in odds).
- If $\theta_j < 0$: feature $j$ is evidence for class 0.

---

## Regularised Logistic Regression

To prevent overfitting, add regularisation:

**L2 (Ridge):**
$$J_{\text{reg}}(\theta) = -\frac{1}{n}\sum_i [y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})] + \frac{\lambda}{2}\sum_{j=1}^d \theta_j^2$$

Gradient with L2:
$$\nabla_\theta J_{\text{reg}} = \frac{1}{n}X^T(\hat{p}-y) + \lambda\theta$$ (don't regularise $\theta_0$)

**L1 (Lasso):** Promotes sparse solutions — some $\theta_j$ go to exactly 0 (automatic feature selection).

---

## Multi-class Extension: Softmax Regression

For $K > 2$ classes, generalise to softmax:
$$P(Y=k \mid x) = \frac{e^{\theta_k^T x}}{\sum_{j=1}^K e^{\theta_j^T x}}, \quad k = 1, \ldots, K$$

See [[Multiclass Classification]].

---

## Logistic Regression vs. Linear Regression

| | Linear Regression | Logistic Regression |
|---|---|---|
| Output | $\hat{y} \in \mathbb{R}$ | $\hat{p} \in (0,1)$ |
| Task | Regression | Binary Classification |
| Activation | None (identity) | Sigmoid $\sigma$ |
| Loss | MSE | Binary Cross-Entropy |
| Decision | N/A | $\hat{y} = \mathbf{1}[\hat{p} \geq 0.5]$ |
| Boundary | N/A | Linear hyperplane |

---

## Assumptions

1. **Linearity of log-odds** in features: $\log \frac{p}{1-p} = \theta^T x$
2. **Independence** of training examples
3. **No multicollinearity** (correlated features make coefficients unstable)
4. **Large $n$**: MLE needs sufficient data; rule of thumb: $\geq 10$ examples per feature

---

## Connections

- [[Sigmoid Function]] — the activation used
- [[Logistic Loss]] — the cost function
- [[Cost Function for Logistic Regression]] — average cross-entropy
- [[Decision Boundary in Logistic Regression]] — the hyperplane where $\hat{p}=0.5$
- [[Gradient Descent for Logistic Regression]] — how parameters are learned
- [[Binary Classification]] — the task
- [[Regularized Logistic Regression]] — adds L1/L2 penalty

---

## One-line Summary

> Logistic regression models the probability of the positive class by squashing a linear score through the sigmoid function — trained by maximising likelihood (minimising cross-entropy), it produces a linear decision boundary and provides probabilistic, interpretable outputs.
