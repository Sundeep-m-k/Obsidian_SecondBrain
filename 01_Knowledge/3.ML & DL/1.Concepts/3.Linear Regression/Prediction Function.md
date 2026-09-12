# Prediction Function

## What is it?

The **prediction function** is the [[Hypothesis Function]] applied to new, unseen data at [[Inference]] time, using the **fixed, trained parameters** $\hat{\theta}$.

$$\hat{y}_{\text{new}} = h_{\hat{\theta}}(x_{\text{new}}) = \hat{\theta}^T x_{\text{new}}$$

During training, the model searches for $\hat{\theta}$. After training, $\hat{\theta}$ is frozen and the prediction function is applied to any new input.

---

## Key Distinction

| Phase | What happens |
|---|---|
| **Training** | $\theta$ is updated; $\nabla_\theta J$ is computed; backward pass runs |
| **Prediction / Inference** | $\theta = \hat{\theta}$ is fixed; only forward pass runs; no gradients needed |

---

## The Output Type Depends on the Task

| Task | Model | Prediction function output |
|---|---|---|
| Regression | Linear regression | $\hat{y} = \hat{\theta}^T x \in \mathbb{R}$ |
| Binary classification | Logistic regression | $\hat{p} = \sigma(\hat{\theta}^T x) \in (0,1)$, then $\hat{y} = \mathbf{1}[\hat{p} \geq 0.5]$ |
| Multi-class | Softmax regression | $\hat{p}_k = \text{softmax}(\hat{\Theta}x)$, then $\hat{y} = \arg\max_k \hat{p}_k$ |

---

## Connections

- [[Hypothesis Function]] — the function used for prediction
- [[Parameters]] — $\hat{\theta}$ are the trained parameters
- [[Inference]] — the process of making predictions
- [[Linear Regression]] — prediction is $\hat{\theta}^T x$

---

## One-line Summary

> The prediction function is the trained model applied to new data — the same structure as the hypothesis function, but with parameters frozen at their learned values.
