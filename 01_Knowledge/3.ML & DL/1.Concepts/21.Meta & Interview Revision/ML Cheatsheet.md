# ML Cheatsheet

A single-page reference for every core formula, rule, and decision you need in ML. Self-contained — sufficient to reconstruct the basics from memory.

---

## 1. The ML Problem

| Scenario | Task | Output | Loss |
|---|---|---|---|
| Predict house price | Regression | $\hat{y} \in \mathbb{R}$ | MSE |
| Email spam? | Binary classification | $\hat{p} \in (0,1)$ | BCE |
| Which digit (0-9)? | Multi-class | $\hat{p}_k$, $k=1..K$ | CCE |
| Group customers | Clustering | Cluster labels | None (unsupervised) |

---

## 2. Core Models

### Linear Regression
$$\hat{y} = \theta^T x, \quad J = \frac{1}{2n}\|X\theta - y\|^2, \quad \hat{\theta} = (X^TX)^{-1}X^Ty$$

### Logistic Regression
$$\hat{p} = \sigma(\theta^T x) = \frac{1}{1+e^{-\theta^Tx}}, \quad J = -\frac{1}{n}\sum[y\log\hat{p}+(1-y)\log(1-\hat{p})]$$

### Softmax Regression (Multi-class)
$$P(Y=k\mid x) = \frac{e^{\theta_k^Tx}}{\sum_j e^{\theta_j^Tx}}, \quad J = -\frac{1}{n}\sum_i\log\hat{p}_{c^{(i)}}$$

---

## 3. Gradient Updates

**General GD:**
$$\theta \leftarrow \theta - \eta\nabla_\theta J(\theta)$$

**Linear Regression gradient:**
$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y)$$

**Logistic Regression gradient (same form!):**
$$\nabla_\theta J = \frac{1}{n}X^T(\hat{p} - y)$$

**With L2 regularisation (add to any gradient, skip $\theta_0$):**
$$\nabla_\theta J_{\text{reg}} = \nabla_\theta J + \lambda\theta$$

---

## 4. Key Activation Functions

| Name | Formula | Derivative |
|---|---|---|
| Sigmoid | $\sigma(z)=\frac{1}{1+e^{-z}}$ | $\sigma(z)(1-\sigma(z))$ |
| Tanh | $\tanh(z)=\frac{e^z-e^{-z}}{e^z+e^{-z}}$ | $1-\tanh^2(z)$ |
| ReLU | $\max(0,z)$ | $\mathbf{1}[z>0]$ |
| Softmax | $\frac{e^{z_k}}{\sum_j e^{z_j}}$ | (see cross-entropy combined) |

---

## 5. Loss Functions

| Task | Loss | Formula |
|---|---|---|
| Regression | MSE | $\frac{1}{n}\sum(y-\hat{y})^2$ |
| Regression | MAE | $\frac{1}{n}\sum|y-\hat{y}|$ |
| Binary classification | BCE | $-\frac{1}{n}\sum[y\log\hat{p}+(1-y)\log(1-\hat{p})]$ |
| Multi-class | CCE | $-\frac{1}{n}\sum\log\hat{p}_{c}$ |
| SVM | Hinge | $\frac{1}{n}\sum\max(0,1-y\hat{f})$ |

---

## 6. Regularisation

| Type | Penalty | Effect | Prior |
|---|---|---|---|
| L2 (Ridge) | $\frac{\lambda}{2}\|\theta\|^2$ | Shrink toward 0 | Gaussian |
| L1 (Lasso) | $\lambda\|\theta\|_1$ | Drive to exactly 0 (sparse) | Laplace |
| Elastic Net | $\lambda_1\|\theta\|_1+\frac{\lambda_2}{2}\|\theta\|^2$ | Both | — |

Ridge closed form: $\hat{\theta} = (X^TX + n\lambda I)^{-1}X^Ty$

---

## 7. Bias-Variance Decomposition

$$\mathbb{E}[(y-\hat{f}(x))^2] = \underbrace{(f(x)-\mathbb{E}[\hat{f}])^2}_{\text{Bias}^2} + \underbrace{\mathbb{E}[(\hat{f}-\mathbb{E}[\hat{f}])^2]}_{\text{Variance}} + \sigma_\epsilon^2$$

- High Bias → Underfitting → simpler model trains poorly
- High Variance → Overfitting → train much better than val

---

## 8. Evaluation Metrics

### Classification (Binary)

$$\text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}, \quad F_1 = \frac{2PR}{P+R}$$
$$\text{Accuracy} = \frac{TP+TN}{n}, \quad \text{AUC} = P(\hat{p}^+ > \hat{p}^-)$$

### Regression

$$\text{RMSE} = \sqrt{\frac{1}{n}\sum(y-\hat{y})^2}, \quad R^2 = 1 - \frac{\sum(y-\hat{y})^2}{\sum(y-\bar{y})^2}$$

---

## 9. Feature Scaling

$$\text{Standardise: } x' = \frac{x-\mu_{\text{train}}}{\sigma_{\text{train}}}, \quad \text{Min-Max: } x' = \frac{x-\min}{\max-\min}$$

Always fit on training set only. Apply same transform to val/test.

---

## 10. Data Splits

```
All Data (N)
├── Train    (70-80%)  ← fit θ
├── Val      (10-15%)  ← tune hyperparams
└── Test     (10-15%)  ← evaluate ONCE, at the end
```

---

## 11. K-Means

$$J = \sum_k\sum_{i\in C_k}\|x^{(i)}-\mu_k\|^2$$

Assign: $c^{(i)} = \arg\min_k\|x^{(i)}-\mu_k\|^2$  
Update: $\mu_k = \frac{1}{|C_k|}\sum_{i\in C_k}x^{(i)}$

---

## 12. PCA

1. Centre: $\tilde{X} = X - \bar{x}$
2. Covariance: $\Sigma = \frac{1}{n}\tilde{X}^T\tilde{X}$
3. Eigendecomp: $\Sigma = V\Lambda V^T$
4. Project: $Z = \tilde{X}V_k$ (top $k$ eigenvectors)
5. EVR$_k = \lambda_k / \sum_j \lambda_j$ (choose $k$ for 95% cumulative EVR)

---

## 13. Decision Rules

| Situation | Do this |
|---|---|
| Output continuous | Linear Regression |
| Output binary | Logistic Regression |
| Output multi-class | Softmax / Neural Net |
| Many irrelevant features | Lasso (L1) |
| Correlated features | Ridge (L2) |
| Train error high | More complexity, more features |
| Val >> Train error | Regularise, more data |
| Class imbalance | Class weights, tune threshold, use F1 |
| Many features, few examples | Regularise heavily; consider PCA first |
| New project baseline | Linear/Logistic Regression first, always |

---

## 14. RL Quick Reference

| Symbol | Meaning |
|---|---|
| $s, a, r, \gamma$ | State, action, reward, discount factor |
| $G_t = \sum_k\gamma^k r_{t+k}$ | Return (discounted cumulative reward) |
| $V^\pi(s) = \mathbb{E}[G_t\mid S_t=s]$ | State value function |
| $Q^\pi(s,a) = \mathbb{E}[G_t\mid S_t=s,A_t=a]$ | Q-function |
| $\pi^*(s) = \arg\max_a Q^*(s,a)$ | Optimal policy |
| TD error: $\delta_t = r_t+\gamma V(s_{t+1})-V(s_t)$ | Temporal difference error |

---

## 15. Common Mistakes Summary

| Mistake | Fix |
|---|---|
| Data leakage | Split first; fit preprocessing on train only |
| Evaluate on train data | Use val/test set |
| MSE for classification | Use cross-entropy |
| No scaling before GD | Standardise features |
| Always use 0.5 threshold | Tune threshold on val set |
| Ignore class imbalance | Use class weights, F1, AUC-PR |
| No baseline | Always start with simple baseline |
| Feature selection outside CV | Do feature selection inside each fold |
