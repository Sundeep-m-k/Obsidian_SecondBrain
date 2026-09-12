# Error Minimization

## What is it?

**Error minimization** is the core principle of supervised ML training: find the parameter values $\hat{\theta}$ that minimise the prediction error (the [[Cost Function]]) on the training set.

$$\hat{\theta} = \arg\min_{\theta} J(\theta) = \arg\min_{\theta} \frac{1}{n}\sum_{i=1}^{n}\ell(y^{(i)}, \hat{f}(x^{(i)};\theta))$$

This principle is called **Empirical Risk Minimization (ERM)**.

---

## Empirical Risk Minimization (ERM)

**True risk** (what we actually care about — unknown):
$$R(\theta) = \mathbb{E}_{(x,y)\sim p_{\text{data}}}\left[\ell(y, \hat{f}(x;\theta))\right]$$

**Empirical risk** (what we compute on training data):
$$\hat{R}(\theta) = \frac{1}{n}\sum_{i=1}^{n}\ell(y^{(i)}, \hat{f}(x^{(i)};\theta))$$

**ERM:** Minimise empirical risk as a proxy for true risk:
$$\hat{\theta}_{\text{ERM}} = \arg\min_\theta \hat{R}(\theta)$$

**Why ERM works:** By the Law of Large Numbers, empirical risk converges to true risk as $n \to \infty$:
$$\hat{R}(\theta) \xrightarrow{n\to\infty} R(\theta)$$

**Why ERM can fail:** For small $n$, $\hat{R}$ is a noisy estimate of $R$. Minimising $\hat{R}$ too hard → overfitting.

---

## Methods of Error Minimization

### Closed-Form (Analytical)
Only possible for specific loss + model combinations (notably MSE + linear model):

$$\hat{\theta} = (X^TX)^{-1}X^Ty$$

Exact, but $O(d^3)$ complexity — infeasible for large $d$.

### Iterative: Gradient Descent
For any differentiable loss, iteratively descend the gradient:

$$\theta \leftarrow \theta - \eta \nabla_\theta J(\theta)$$

Converges to the minimum for convex $J$. For non-convex (neural nets), converges to a local minimum or saddle point.

See [[Gradient Descent]] for full details.

### Constrained Minimization (SVM)
Some objectives have constraints:

$$\min_{\theta,b} \frac{1}{2}\|\theta\|^2 \quad \text{subject to } y^{(i)}(\theta^Tx^{(i)}+b) \geq 1 \quad \forall i$$

Solved via Lagrange multipliers and the dual formulation (quadratic programming).

---

## Structural Risk Minimization (SRM)

An extension of ERM that balances empirical risk with model complexity:

$$\hat{\theta}_{\text{SRM}} = \arg\min_\theta \hat{R}(\theta) + \lambda \cdot \text{Complexity}(\theta)$$

This is exactly regularised training. SRM is motivated by VC theory: minimise an upper bound on true risk.

---

## Connections

- [[Cost Function]] — the empirical risk we minimise
- [[Gradient Descent]] — the iterative minimisation algorithm
- [[Objective Function]] — cost + penalty = full objective
- [[Generalization]] — gap between empirical and true risk
- [[Regularization]] — prevents over-minimising empirical risk

---

## One-line Summary

> Error minimization is the training principle: find parameters that make the model's predictions as close to correct as possible on training data, using Empirical Risk Minimization as an approximation of the true generalisation error.
