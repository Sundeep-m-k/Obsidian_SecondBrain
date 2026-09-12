# Parameters

## What is it?

**Parameters** (also called **weights** and **biases**) are the learnable numerical values inside a [[Model]] that are adjusted during training to minimise the [[Loss Function]]. They encode everything the model has learned from data.

$$\theta = [\theta_0, \theta_1, \theta_2, \ldots, \theta_d]^T \in \mathbb{R}^{d+1}$$

Before training: $\theta$ is initialised randomly (or to zeros).  
After training: $\theta$ encodes the patterns learned from data.

---

## Parameters in Linear Regression

$$\hat{y} = \theta_0 + \theta_1 x_1 + \theta_2 x_2 + \cdots + \theta_d x_d$$

| Parameter | Name | Interpretation |
|---|---|---|
| $\theta_0$ | **Bias / Intercept** | Prediction when all features are zero |
| $\theta_1, \ldots, \theta_d$ | **Weights / Coefficients** | Change in $\hat{y}$ per unit increase in $x_j$, holding others fixed |

**Example:** If $\theta_1 = 500$ in a house price model where $x_1$ = size in sq ft, then each additional sq ft adds \$500 to the predicted price.

---

## Geometric Interpretation

In 2D ($d=1$):

$$\hat{y} = \theta_0 + \theta_1 x$$

- $\theta_0$ = **y-intercept** (where the line crosses the y-axis)
- $\theta_1$ = **slope** (steepness of the line)

In $d$ dimensions, $\theta$ defines a hyperplane:
- $\theta_0$ = the offset from the origin
- $\theta_1, \ldots, \theta_d$ = the normal vector direction

---

## Parameters vs. Hyperparameters

| | Parameters | Hyperparameters |
|---|---|---|
| **Examples** | $\theta_0, \theta_1, \ldots, \theta_d$ in linear regression; weights $W^{(l)}$ and biases $b^{(l)}$ in neural nets | Learning rate $\eta$, number of layers $L$, regularisation $\lambda$ |
| **Set by** | Optimisation (gradient descent, normal equations) | You, before training |
| **Change during training** | Yes | No |

---

## Scale of Parameters

| Model | Parameter Count |
|---|---|
| Simple linear regression ($d=1$) | 2 |
| Multiple linear regression ($d$ features) | $d+1$ |
| Logistic regression ($d$ features) | $d+1$ |
| Small neural net (2 layers, 100 neurons each) | ~$d \times 100 + 100 \times 100 + 100$ ≈ tens of thousands |
| GPT-3 | 175 billion |
| GPT-4 (estimated) | ~1 trillion |

---

## Initialisation Matters (for Neural Networks)

For linear regression, initialising $\theta = 0$ or randomly makes no difference (the cost surface is convex).

For neural networks, **initialising all weights to zero breaks symmetry** — all neurons compute the same gradient and learn the same thing. Always use random initialisation.

**Xavier/Glorot initialisation:** For layer with $n_{\text{in}}$ inputs and $n_{\text{out}}$ outputs:
$$W \sim \mathcal{N}\left(0, \sqrt{\frac{2}{n_{\text{in}} + n_{\text{out}}}}\right) \quad \text{or} \quad W \sim \text{Uniform}\left(-\sqrt{\frac{6}{n_{\text{in}}+n_{\text{out}}}}, \sqrt{\frac{6}{n_{\text{in}}+n_{\text{out}}}}\right)$$

**He initialisation** (for ReLU networks):
$$W \sim \mathcal{N}\left(0, \sqrt{\frac{2}{n_{\text{in}}}}\right)$$

Designed to keep variance of activations constant across layers.

---

## Connections

- [[Model Representation]] — parameters fill the model's structure
- [[Hypothesis Function]] — $\theta$ controls the hypothesis
- [[Gradient Descent]] — the algorithm that updates $\theta$
- [[Loss Function]] — parameters are optimised to minimise this
- [[Linear Regression]] — parameters are weights and bias
- [[Regularization]] — penalises large parameters

---

## One-line Summary

> Parameters are the numerical values inside a model that are learned from data — they are the entire "memory" of training, encoding patterns as weight magnitudes and directions that transform inputs into accurate predictions.
