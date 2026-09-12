# Cost Function for Logistic Regression

## What is it?

The cost function for logistic regression is the **Binary Cross-Entropy** (also called Log Loss), averaged over all training examples.

$$J(\theta) = -\frac{1}{n}\sum_{i=1}^{n}\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right]$$

Where $\hat{p}^{(i)} = \sigma(\theta^T x^{(i)}) = \frac{1}{1+e^{-\theta^T x^{(i)}}}$.

---

## Rewriting as a Single Expression

Using the fact that $y \in \{0,1\}$, the loss per example equals:

$$\ell^{(i)} = \begin{cases} -\log\hat{p}^{(i)} & \text{if } y^{(i)} = 1 \\ -\log(1-\hat{p}^{(i)}) & \text{if } y^{(i)} = 0 \end{cases}$$

Both cases penalise confident wrong predictions logarithmically (approaching $\infty$ as $\hat{p} \to$ wrong end).

---

## Why This Loss and Not MSE?

If we used MSE: $J(\theta) = \frac{1}{n}\sum(\hat{p}^{(i)} - y^{(i)})^2$ with sigmoid, the cost surface is **non-convex** (many local minima) and the gradient **vanishes** when predictions are wrong and confident.

Binary cross-entropy:
- Produces a **convex** cost surface (proven via second-order analysis).
- Has large gradient when confidently wrong: $\frac{\partial \ell}{\partial \hat{p}} = -\frac{1}{\hat{p}}$ → $\infty$ as $\hat{p} \to 0$ for $y=1$.
- Derivation from **MLE** makes it statistically optimal.

---

## The Gradient

$$\frac{\partial J}{\partial \theta_j} = \frac{1}{n}\sum_{i=1}^{n}(\hat{p}^{(i)} - y^{(i)})x_j^{(i)}$$

In matrix form:
$$\nabla_\theta J = \frac{1}{n}X^T(\hat{p} - y)$$

**Key observation:** This is **identical in form** to the MSE gradient for linear regression: $\frac{1}{n}X^T(\hat{y}-y)$. Only the prediction formula differs ($\hat{p} = \sigma(\theta^Tx)$ vs. $\hat{y} = \theta^Tx$). This is not coincidence — both come from exponential family MLE.

---

## Connections

- [[Logistic Regression]] — the model
- [[Logistic Loss]] — the per-example loss
- [[Gradient Descent for Logistic Regression]] — uses this gradient
- [[Cost Function]] — general concept

---

## One-line Summary

> The logistic regression cost function is binary cross-entropy — convex in $\theta$, derived from MLE under the Bernoulli model, with a gradient of identical form to linear regression's MSE gradient.
