# Hypothesis Function

## What is it?

The **hypothesis function** $h_\theta(x)$ is the function that a model uses to make predictions. It maps an input $x$ to a predicted output $\hat{y}$.

$$h_\theta: \mathbb{R}^d \rightarrow \mathcal{Y}, \quad \hat{y} = h_\theta(x)$$

The subscript $\theta$ emphasises that the function's behaviour is controlled by the learnable parameters.

---

## Hypothesis Functions by Model Type

| Model | Hypothesis Function |
|---|---|
| Simple Linear Regression | $h_\theta(x) = \theta_0 + \theta_1 x$ |
| Multiple Linear Regression | $h_\theta(x) = \theta^T x$ |
| Polynomial Regression (degree 2) | $h_\theta(x) = \theta_0 + \theta_1 x + \theta_2 x^2$ |
| Logistic Regression (binary) | $h_\theta(x) = \sigma(\theta^T x) = \frac{1}{1+e^{-\theta^T x}}$ |
| Softmax Regression ($K$ classes) | $h_\theta(x) = \text{softmax}(\Theta x)$ |
| Neural Network | $h_\theta(x) = f_L \circ f_{L-1} \circ \cdots \circ f_1(x)$ |

---

## The Hypothesis Space

The set of all functions that the model can represent (over all possible $\theta$) is the **hypothesis space** $\mathcal{H}$:

$$\mathcal{H} = \{h_\theta : \theta \in \mathbb{R}^{d+1}\}$$

For linear regression: $\mathcal{H}$ = all linear functions in $d$ dimensions.  
For neural networks: $\mathcal{H}$ = all functions representable by that architecture.

Training = searching through $\mathcal{H}$ to find the best $h_\theta$.

---

## Connections

- [[Model Representation]] — defines the form of $h_\theta$
- [[Parameters]] — the $\theta$ that determine $h_\theta$
- [[Prediction Function]] — $h_\theta$ applied at inference time
- [[Linear Regression]] — $h_\theta(x) = \theta^T x$
- [[Logistic Regression]] — $h_\theta(x) = \sigma(\theta^T x)$

---

## One-line Summary

> The hypothesis function is the parameterised function the model uses to map inputs to predictions — it defines the structural form of predictions before training, and parameters are tuned to find the best function within that form.
