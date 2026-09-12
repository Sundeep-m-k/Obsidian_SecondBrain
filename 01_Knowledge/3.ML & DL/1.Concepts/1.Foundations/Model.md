## What is it?

In [[Machine Learning]], a **model** is a mathematical function with adjustable parameters that maps inputs to outputs. It is the artifact produced by training — the learned representation of the patterns in the [[Training Data]].

$$\text{Model}: x \mapsto \hat{y} = \hat{f}(x;\theta)$$

Where:
- $x \in \mathbb{R}^d$ = input feature vector
- $\hat{y}$ = predicted output
- $\hat{f}$ = the functional form of the model
- $\theta$ = the **parameters** (learned from data)

> Before training: $\theta$ is random or zero. After training: $\theta$ encodes the patterns learned from data.

---

## The Hypothesis Class

A **hypothesis class** $\mathcal{H}$ is the set of all functions a model can represent as its parameters vary:

$$\mathcal{H} = \{\hat{f}(\cdot;\theta) : \theta \in \Theta\}$$

- A linear model's hypothesis class = all linear functions.
- A neural network's hypothesis class = all functions representable by that architecture.

**Capacity** = how expressive/complex the hypothesis class is. Higher capacity → can fit more complex functions, but also easier to overfit.

Training = **search** within $\mathcal{H}$ for the $\hat{f}$ that best explains the training data.

---

## Parameters vs. Hyperparameters (revisited)

| | Parameters $\theta$ | Hyperparameters |
|---|---|---|
| **Learned during** | Training (gradient descent / closed form) | Before training (you set them) |
| **Examples** | Weights and biases of a neural net | Number of layers, learning rate, regularisation strength |
| **Scale** | Can be millions to billions | Typically tens |

---

## The Zoo of Model Types

### 1. Linear Models

**Linear Regression:**
$$\hat{y} = \theta^T x = \theta_0 + \theta_1 x_1 + \cdots + \theta_d x_d$$

Parameter count: $d + 1$ (including bias $\theta_0$).  
The decision surface is a **hyperplane** in $\mathbb{R}^d$.

**Logistic Regression:**
$$\hat{p}(y=1 \mid x) = \sigma(\theta^T x) = \frac{1}{1 + e^{-\theta^T x}}$$

Hypothesis class: all linear decision boundaries.

**Expressiveness:** Can only learn linearly separable patterns (in the original feature space). Extend to non-linear patterns using feature engineering (polynomial features, basis functions).

---

### 2. Decision Trees

A binary tree where:
- Each **internal node** tests a feature: `if x_j ≤ threshold`
- Each **leaf node** gives a prediction (class label or real value)

**Depth $D$:** A tree of depth $D$ has at most $2^D$ leaves.  
**Parameter count:** Number of (feature, threshold) pairs at internal nodes + predictions at leaves.

Decision trees partition the feature space into **axis-aligned rectangular regions**.

**Expressiveness:** Arbitrary shapes but axis-aligned. Can represent any function given sufficient depth. In practice, limited by overfitting.

---

### 3. Ensemble Models

Combine multiple models (called **base learners** or **weak learners**) whose errors tend to cancel.

**Bagging (Bootstrap Aggregation):**
- Train $M$ trees on bootstrap samples of the data.
- Aggregate predictions:
  - Classification: $\hat{y} = \text{majority vote}(\hat{y}_1, \ldots, \hat{y}_M)$
  - Regression: $\hat{y} = \frac{1}{M}\sum_{m=1}^{M} \hat{y}_m$
- **Random Forest** = bagging + random feature subsets per split.

**Why it works:** The variance of the average of $M$ uncorrelated models with variance $\sigma^2$ is:
$$\text{Var}\left(\frac{1}{M}\sum_{m=1}^M \hat{y}_m\right) = \frac{\sigma^2}{M}$$
(Reduces variance while bias stays the same.)

**Boosting:**
- Train models sequentially. Each model focuses on the mistakes of the previous.
- **AdaBoost:** Reweight training examples; misclassified examples get higher weight.
- **Gradient Boosting:** Fit each new model to the **residuals** (negative gradient of the loss):
$$\hat{y}_m(x) = \hat{y}_{m-1}(x) + \eta \cdot h_m(x)$$
where $h_m$ is the $m$-th tree fit to the residuals and $\eta$ is the learning rate.

**XGBoost / LightGBM / CatBoost** are highly optimised implementations of gradient boosting, dominant on tabular data.

---

### 4. Support Vector Machines (SVM)

See [[Supervised Learning]] for the full derivation. Key points:
- Finds the **maximum-margin hyperplane**.
- Parameters: support vectors (the training examples that define the margin), dual coefficients $\alpha_i$, and bias $b$.
- **The kernel trick** makes the model non-parametric in terms of the original features.
- For large datasets, SVMs are expensive ($O(n^2)$ to $O(n^3)$).

---

### 5. Neural Networks

The most expressive and general-purpose model class.

**Architecture:**
```
Input layer x ∈ ℝ^d
    → Hidden layer 1: a^(1) = σ(W^(1)x + b^(1))
    → Hidden layer 2: a^(2) = σ(W^(2)a^(1) + b^(2))
    → ...
    → Output layer: ŷ
```

**Notation:**
- $W^{(l)} \in \mathbb{R}^{n_l \times n_{l-1}}$ = weight matrix for layer $l$
- $b^{(l)} \in \mathbb{R}^{n_l}$ = bias vector for layer $l$
- $n_l$ = number of neurons in layer $l$
- $\sigma$ = activation function (applied element-wise)

**A single neuron:**
$$a_j^{(l)} = \sigma\left(\sum_i w_{ji}^{(l)} a_i^{(l-1)} + b_j^{(l)}\right)$$

**Total parameters:**
$$\text{Parameters} = \sum_{l=1}^{L} \left(n_l \cdot n_{l-1} + n_l\right) = \sum_{l=1}^{L} n_l(n_{l-1} + 1)$$

**Universal Approximation Theorem:** A feedforward neural network with at least one hidden layer and a non-linear activation function can approximate any continuous function on a compact subset of $\mathbb{R}^d$ to arbitrary precision, given enough neurons. (Hornik, 1989)

This is an existence result — it doesn't tell you how to train the network or how big it needs to be.

---

## Activation Functions

The non-linearity that allows neural networks to go beyond linear models.

| Name | Formula | Range | Notes |
|---|---|---|---|
| **Sigmoid** | $\sigma(z) = \frac{1}{1+e^{-z}}$ | $(0, 1)$ | Output layers for binary classification; vanishing gradient problem |
| **Tanh** | $\tanh(z) = \frac{e^z - e^{-z}}{e^z + e^{-z}}$ | $(-1, 1)$ | Zero-centred; still vanishing gradient |
| **ReLU** | $\max(0, z)$ | $[0, \infty)$ | Default for hidden layers; fast; "dying ReLU" problem |
| **Leaky ReLU** | $\max(0.01z, z)$ | $\mathbb{R}$ | Fixes dying ReLU |
| **GELU** | $z \cdot \Phi(z)$ | $\mathbb{R}$ | Used in Transformers (BERT, GPT) |
| **Softmax** | $\frac{e^{z_k}}{\sum_j e^{z_j}}$ | $(0,1)^K$, sum=1 | Output layer for multi-class |
| **Swish** | $z \cdot \sigma(z)$ | $\mathbb{R}$ | Smooth, non-monotonic; used in EfficientNet |

**Vanishing gradient problem:** For sigmoid/tanh, gradients $\frac{d\sigma}{dz} \approx 0$ for large $|z|$. In deep networks, gradients shrink exponentially during backpropagation → early layers learn very slowly. ReLU solves this by having gradient = 1 for positive inputs.

---

## Backpropagation (How Neural Nets Are Trained)

**Backpropagation** is an efficient algorithm to compute $\frac{\partial \mathcal{L}}{\partial \theta}$ for all parameters $\theta$ simultaneously, using the **chain rule of calculus**.

**Forward pass:** Compute the output $\hat{y}$ layer by layer.

**Backward pass:** Compute gradients layer by layer from output back to input.

For the last layer: $\delta^{(L)} = \frac{\partial \mathcal{L}}{\partial a^{(L)}}$

For earlier layers (chain rule):
$$\delta^{(l)} = \left(W^{(l+1)T} \delta^{(l+1)}\right) \odot \sigma'(z^{(l)})$$

Gradients of weights:
$$\frac{\partial \mathcal{L}}{\partial W^{(l)}} = \delta^{(l)} (a^{(l-1)})^T$$

Then update: $W^{(l)} \leftarrow W^{(l)} - \eta \frac{\partial \mathcal{L}}{\partial W^{(l)}}$

---

## Key Model Architectures (Quick Reference)

| Architecture | Best for | Key innovation |
|---|---|---|
| **MLP (Feedforward NN)** | Tabular data, general purpose | Fully connected layers |
| **CNN (Convolutional NN)** | Images, spatial data | Local filters, weight sharing |
| **RNN / LSTM / GRU** | Sequences, time series | Hidden state carries context |
| **Transformer** | Text, sequences, images | Self-attention mechanism |
| **GAN** | Generative modelling | Generator vs. Discriminator game |
| **VAE** | Generative + latent representation | Probabilistic encoder |
| **Graph NN (GNN)** | Graph-structured data | Message passing on graphs |

---

## Model Selection

Given multiple model types and hyperparameter settings, how do you choose?

**Occam's Razor:** Prefer the simpler model that explains the data equally well.

**AIC (Akaike Information Criterion):**
$$\text{AIC} = 2k - 2\ln(\hat{\mathcal{L}})$$

**BIC (Bayesian Information Criterion):**
$$\text{BIC} = k\ln(n) - 2\ln(\hat{\mathcal{L}})$$

where $k$ = number of parameters, $n$ = sample size, $\hat{\mathcal{L}}$ = maximised likelihood.  
BIC penalises complexity more strongly for large $n$.

In practice: **cross-validated performance on validation set** is the most reliable criterion.

---

## Saving and Serialising Models

In practice, trained models are **serialised** (saved to disk) after training for later use during [[Inference]].

Common formats:
- **Pickle** (Python-specific, not portable)
- **ONNX** (Open Neural Network Exchange — cross-framework, cross-platform)
- **SavedModel / .pt** (TensorFlow / PyTorch native formats)
- **PMML** (Predictive Model Markup Language — for traditional ML)

---

## Connections

- [[Machine Learning]] — training produces a model
- [[Training Data]] — the data used to set model parameters
- [[Features]] — the input $x$ to the model
- [[Labels]] — what the model learns to predict
- [[Supervised Learning]] — trains a model with labelled data
- [[Inference]] — using a trained model to make predictions
- [[Generalization]] — whether the model works beyond training data

---

## One-line Summary

> A model is a parameterised mathematical function — the artifact produced by training — that maps input features to predictions; its hypothesis class determines what patterns it can represent, and its trained parameters encode what it has learned from data.
