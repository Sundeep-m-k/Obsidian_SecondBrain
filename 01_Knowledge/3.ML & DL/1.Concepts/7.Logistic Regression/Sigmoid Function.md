# Sigmoid Function

## What is it?

The **sigmoid function** (also called the **logistic function**) is a smooth S-shaped curve that maps any real number to a value in $(0, 1)$.

$$\sigma(z) = \frac{1}{1 + e^{-z}}$$

---

## Plot and Key Values

```
σ(z)
  1 ┤                  ────────
    │             ╱
0.5 ┤────────────●────────────   ← σ(0) = 0.5
    │       ╱
  0 ┤────────
    └──────────────────────────── z
       -5   -3   -1    1   3   5
```

| $z$ | $\sigma(z)$ |
|---|---|
| $-\infty$ | 0 |
| $-3$ | 0.047 |
| $-1$ | 0.269 |
| $0$ | 0.500 |
| $1$ | 0.731 |
| $3$ | 0.953 |
| $+\infty$ | 1 |

---

## Properties

**Range:** $\sigma(z) \in (0, 1)$ — never exactly 0 or 1.

**Symmetry:** $\sigma(-z) = 1 - \sigma(z)$.

**Derivative — the key formula:**
$$\sigma'(z) = \frac{d\sigma}{dz} = \sigma(z)(1 - \sigma(z))$$

This is elegant: the derivative at any point depends only on the function value at that point. Useful for backpropagation.

**Proof:**
$$\frac{d}{dz}\frac{1}{1+e^{-z}} = \frac{e^{-z}}{(1+e^{-z})^2} = \frac{1}{1+e^{-z}} \cdot \frac{e^{-z}}{1+e^{-z}} = \sigma(z)(1-\sigma(z))$$

**Vanishing gradient problem:** When $|z|$ is large, $\sigma(z) \approx 0$ or $\approx 1$, so $\sigma'(z) \approx 0$. Gradients flowing through many sigmoid layers shrink exponentially → deep sigmoid networks are hard to train. ReLU solves this.

**Inverse:** The inverse of sigmoid is the **logit** function:
$$\sigma^{-1}(p) = \log\frac{p}{1-p} = \text{logit}(p)$$

---

## Connection to Probability

For logistic regression, the sigmoid maps the linear score $z = \theta^T x$ to a valid probability:
$$P(Y=1 \mid x) = \sigma(\theta^T x)$$

Since $\sigma(z) \in (0,1)$, this is always a valid probability. No threshold needed to keep outputs in range.

---

## Sigmoid vs. Other Activation Functions

| Activation | Formula | Range | Derivative |
|---|---|---|---|
| Sigmoid | $\frac{1}{1+e^{-z}}$ | $(0,1)$ | $\sigma(z)(1-\sigma(z))$ |
| Tanh | $\frac{e^z-e^{-z}}{e^z+e^{-z}}$ | $(-1,1)$ | $1-\tanh^2(z)$ |
| ReLU | $\max(0,z)$ | $[0,\infty)$ | $\mathbf{1}[z>0]$ |

Sigmoid is used in:
- Output layer for binary classification (produces probability)
- Gates in LSTM (input gate, forget gate, output gate)

---

## Connections

- [[Logistic Regression]] — sigmoid is the activation
- [[Probability Output]] — sigmoid produces the probability
- [[Log Odds]] — logit is the inverse of sigmoid
- [[Logistic Loss]] — loss involving $\log\sigma(z)$

---

## One-line Summary

> The sigmoid function squashes any real number into $(0,1)$, making it the ideal output activation for producing probabilities in binary classification — its elegant derivative $\sigma(z)(1-\sigma(z))$ enables efficient gradient computation, though it suffers from vanishing gradients in deep networks.
