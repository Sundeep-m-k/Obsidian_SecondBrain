# Regularized Logistic Regression

## What is it?

**Regularized Logistic Regression** adds a penalty term to the binary cross-entropy cost to prevent [[Overfitting]] in classification.

---

## L2 Regularized Logistic Regression (Default in Practice)

$$J_{\text{L2}}(\theta) = -\frac{1}{n}\sum_{i=1}^n\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right] + \frac{\lambda}{2}\sum_{j=1}^d\theta_j^2$$

**Gradient:**
$$\frac{\partial J_{\text{L2}}}{\partial\theta_j} = \frac{1}{n}\sum_{i=1}^n(\hat{p}^{(i)} - y^{(i)})x_j^{(i)} + \lambda\theta_j, \quad j=1,\ldots,d$$
$$\frac{\partial J_{\text{L2}}}{\partial\theta_0} = \frac{1}{n}\sum_{i=1}^n(\hat{p}^{(i)} - y^{(i)}) \quad \text{(no regularisation on bias)}$$

**Update:**
$$\theta_j \leftarrow (1-\eta\lambda)\theta_j - \frac{\eta}{n}\sum_i(\hat{p}^{(i)}-y^{(i)})x_j^{(i)}$$

The weight decay factor $(1-\eta\lambda)$ continuously shrinks weights toward zero at each step.

---

## L1 Regularized Logistic Regression

$$J_{\text{L1}}(\theta) = -\frac{1}{n}\sum_{i=1}^n\left[y^{(i)}\log\hat{p}^{(i)} + (1-y^{(i)})\log(1-\hat{p}^{(i)})\right] + \lambda\sum_{j=1}^d|\theta_j|$$

Produces sparse solutions — zeros out irrelevant feature weights.

---

## The C Parameter in Scikit-Learn

In `sklearn.linear_model.LogisticRegression`, the convention is **inverse regularisation strength**: $C = 1/\lambda$.

- Large $C$ (e.g., 1000): weak regularisation (trust the data)
- Small $C$ (e.g., 0.01): strong regularisation (constrain weights)
- Default: $C = 1.0$ ($\lambda = 1$)

This is a common source of confusion — **C is the inverse of $\lambda$**.

---

## Effect on Decision Boundary

Regularisation **does not change the form** of the decision boundary (still a hyperplane), but it controls the **magnitude** of $\theta$:

- Smaller $\|\theta\|$ → shallower sigmoid transitions → less confident predictions → smoother, more uncertain boundary.
- Larger $\|\theta\|$ → steeper sigmoid → very confident predictions → sharp boundary.

Strong regularisation = model is less certain, stays closer to probability 0.5.

---

## Connections

- [[Logistic Regression]] — the base model
- [[L1 Regularization]] — sparse logistic regression
- [[L2 Regularization]] — ridge logistic regression
- [[Regularization]] — general concept
- [[Penalty Term]] — the added term

---

## One-line Summary

> Regularized logistic regression adds L1 or L2 penalties to the cross-entropy loss — L2 shrinks all weights (default, implemented via `C=1/λ` in sklearn), L1 zeros out irrelevant features, both controlled by the regularisation strength hyperparameter.
