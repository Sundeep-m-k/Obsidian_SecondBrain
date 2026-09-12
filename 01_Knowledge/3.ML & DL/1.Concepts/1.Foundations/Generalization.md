# Generalization

## What is it?

**Generalization** is the ability of a trained [[Model]] to perform well on **new, unseen data** — data it was not trained on.

This is the central goal of [[Machine Learning]]. A model that only works on training data is useless. A model that works on the true underlying distribution of data is what we want.

> Memorising training examples is not learning. Generalising from them is.

---

## Formal Definition

Let $p_{\text{data}}(x, y)$ be the true data distribution.

**True (generalisation) error:**
$$R(\hat{f}) = \mathbb{E}_{(x,y) \sim p_{\text{data}}}\left[\mathcal{L}(y, \hat{f}(x))\right]$$

This is the **expected loss on a fresh example drawn from the real world**. It's what we actually care about but can never compute directly (we don't know $p_{\text{data}}$).

**Empirical (training) error:**
$$\hat{R}(\hat{f}) = \frac{1}{n}\sum_{i=1}^{n}\mathcal{L}(y^{(i)}, \hat{f}(x^{(i)}))$$

This is the average loss on the training set. We can compute it. But minimising this is just memorisation.

**The gap:**
$$\underbrace{R(\hat{f})}_{\text{true error}} = \underbrace{\hat{R}(\hat{f})}_{\text{training error}} + \underbrace{\left(R(\hat{f}) - \hat{R}(\hat{f})\right)}_{\text{generalisation gap}}$$

The goal of ML theory: understand and control the generalisation gap.

---

## The Bias-Variance-Noise Decomposition

For squared error loss, the expected true error of a model $\hat{f}$ at a point $x$ decomposes as:

$$\mathbb{E}\left[(y - \hat{f}(x))^2\right] = \underbrace{\left(\mathbb{E}[\hat{f}(x)] - f(x)\right)^2}_{\text{Bias}^2} + \underbrace{\mathbb{E}\left[\left(\hat{f}(x) - \mathbb{E}[\hat{f}(x)]\right)^2\right]}_{\text{Variance}} + \underbrace{\sigma_\epsilon^2}_{\text{Irreducible Noise}}$$

Where:
- $f(x)$ = true function (unknown)
- $\mathbb{E}[\hat{f}(x)]$ = expectation over all possible training sets of size $n$
- $\sigma_\epsilon^2$ = variance of the noise in the labels

| Term | High when | Effect |
|---|---|---|
| **Bias** | Model is too simple | Systematic error — model consistently wrong in the same direction |
| **Variance** | Model is too complex | Model is inconsistent — different training sets → very different models |
| **Noise** | Labels are noisy | Irreducible; no model can eliminate this |

**Key insight:** Bias and variance trade off as you change model complexity.

```
         Error
           │
           │  Total Error
           │     ╲     ╱
           │      ╲   ╱
  Bias²    │       ╲ ╱
  ─────────┤────────X─────────
  Variance │       / ╲
           │      /   ╲
           │─────/─────╲──────
           └──────────────────── Model Complexity
                      ↑
               Sweet Spot
```

---

## Overfitting and Underfitting

### Overfitting
Model performs **much better on training data than on new data**.

- The model has learned the noise and idiosyncrasies of the training set.
- High variance, low bias.
- Training error ≪ Validation error.

**Example:** A degree-100 polynomial interpolating 10 data points perfectly but oscillating wildly between them.

### Underfitting
Model performs **poorly on both training and new data**.

- The model is too simple to capture the underlying pattern.
- High bias, low variance.
- Training error ≈ Validation error, but both are high.

**Example:** A linear model trying to fit a parabolic relationship.

---

## Diagnostic: Learning Curves

Plot training and validation error as a function of training set size $n$ or training epochs.

**High bias (underfitting):**
```
Error
  │ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ validation error
  │──────────────────── training error
  │ (both converge to high value)
  └───────────────────────── n
```
Both errors are high and converge. Adding more data won't help. Need a more complex model.

**High variance (overfitting):**
```
Error
  │─ ─ ─ ─ ─ ─ ─ ─ ─  validation error (still high)
  │
  │
  │              ──────── training error (very low)
  └───────────────────────── n
```
Large gap between training and validation error. Adding more data would help. Also try regularisation.

---

## Theoretical Bounds on Generalization

### VC Theory (Vapnik-Chervonenkis)

The **VC dimension** $d_{VC}(\mathcal{H})$ of a hypothesis class $\mathcal{H}$ is the largest number of points that can be **shattered** — classified correctly in every possible binary labelling.

- Linear classifiers in $\mathbb{R}^d$: $d_{VC} = d + 1$
- Decision stumps in $\mathbb{R}$: $d_{VC} = 2$
- Deep neural networks: $d_{VC}$ can be astronomically large

**Vapnik-Chervonenkis Bound:**

With probability $\geq 1 - \delta$ over training sets of size $n$:

$$R(\hat{f}) \leq \hat{R}(\hat{f}) + \sqrt{\frac{d_{VC} \ln(2n/d_{VC}) + \ln(4/\delta)}{n}}$$

**Interpretation:**
- Generalisation error ≤ Training error + complexity penalty
- More complex hypothesis class → larger penalty
- More data → smaller penalty
- The bound tightens as $n \to \infty$: eventually training error ≈ true error.

**Caveat:** VC bounds are often very loose (too pessimistic) in practice, especially for deep learning.

### PAC Learning (Probably Approximately Correct)

A hypothesis class $\mathcal{H}$ is **PAC-learnable** if for any $\epsilon, \delta > 0$, an algorithm exists that, given $n \geq \text{poly}(1/\epsilon, 1/\delta, d)$ examples, outputs a hypothesis with true error $\leq \epsilon$ with probability $\geq 1 - \delta$.

**Sample complexity** — how many examples do you need?
$$n \geq \frac{1}{2\epsilon^2}\ln\frac{|\mathcal{H}|}{\delta} \quad \text{(finite hypothesis class)}$$
$$n \geq \frac{1}{\epsilon}\left(d_{VC}\ln\frac{1}{\epsilon} + \ln\frac{1}{\delta}\right) \quad \text{(general case)}$$

### Double Descent (Modern Understanding)

Classical view: bias-variance tradeoff implies a U-shaped test error curve as model size increases.

**Modern observation (Belkin et al., 2019):** For very large models (overparameterised regime, where parameters > training examples), test error can **decrease again** after initial overfitting:

```
Error
  │
  │      ╭─╮
  │     ╱   ╲              ╲
  │    ╱     ╲              ╲
  │───╱       ╲──────────────╲──────
  └─────────────────────────────── Model Size
     classical  interpolation     overparameterised
     regime     threshold         regime
```

This "double descent" is observed in neural networks, random forests, and kernel machines. The mechanism is not yet fully theoretically understood, but it is consistently observed empirically. Large models trained past interpolation threshold can generalise well — contradicting the classical view.

---

## Strategies to Improve Generalisation

### 1. More Data
The most reliable way. More data reduces variance.
$$\text{Variance} \propto \frac{1}{n}$$

### 2. Regularisation
Add a penalty to prevent the model from becoming too complex.

**L2 regularisation:** $\mathcal{L}_{\text{reg}} = \mathcal{L} + \frac{\lambda}{2}\|\theta\|^2$

Has a Bayesian interpretation: equivalent to placing a Gaussian prior $p(\theta) = \mathcal{N}(0, 1/\lambda)$ on the weights. MAP estimation with this prior equals L2-regularised MLE.

**L1 regularisation:** $\mathcal{L}_{\text{reg}} = \mathcal{L} + \lambda\|\theta\|_1$

Bayesian interpretation: Laplace prior on weights. Promotes sparsity.

**Dropout (for neural nets):** During training, randomly set each neuron's activation to 0 with probability $p$ (typically 0.5 for hidden layers, 0.1-0.2 for input).

At test time, use all neurons but scale by $(1-p)$. This approximates averaging over an exponential number of sub-networks — a form of ensemble.

Effective dropout rate interpretation: With dropout probability $p$, each weight $w$ effectively becomes $w(1-p)$ in expectation.

**Early stopping:** Monitor validation loss during training. Stop when validation loss starts increasing:

```
Loss
  │
  │           ────── validation loss
  │        ╱
  │──────╱───────────────────
  │          training loss
  │ (stops here ↑)
  └────────────────────────── Epochs
```

### 3. Data Augmentation
Artificially expand training set with label-preserving transformations. See [[Training Data]].

### 4. Batch Normalisation
Normalise the activations of each layer during training:

$$\hat{a}^{(l)} = \frac{a^{(l)} - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}$$

Then scale and shift: $z^{(l)} = \gamma \hat{a}^{(l)} + \beta$ where $\gamma, \beta$ are learned parameters.

$\mu_B, \sigma_B^2$ are computed over the mini-batch.

**Effects:** Reduces internal covariate shift, acts as regularisation (adds noise via batch statistics), allows higher learning rates, faster convergence.

### 5. Cross-validation
Use $k$-fold CV to get a reliable estimate of generalisation performance, and to select models/hyperparameters that generalise best. See [[Training Data]].

### 6. Ensemble Methods
Average predictions from multiple models. Even if each model overfits, the errors tend to cancel:
$$R\left(\frac{1}{M}\sum_m \hat{f}_m\right) \leq \frac{1}{M}\sum_m R(\hat{f}_m)$$
(Jensen's inequality applied to convex loss functions)

---

## Distribution Shift

**Generalisation to the i.i.d. test set is not the only concern.** In deployment, the distribution may differ from training.

### Types of Distribution Shift

**Covariate shift:** $p_{\text{train}}(x) \neq p_{\text{test}}(x)$, but $p(y \mid x)$ is the same.
- Example: Train on photos taken in summer, deploy in winter.
- Fix: Importance weighting — reweight training examples by $\frac{p_{\text{test}}(x)}{p_{\text{train}}(x)}$.

**Label shift / Prior probability shift:** $p_{\text{train}}(y) \neq p_{\text{test}}(y)$, but $p(x \mid y)$ is the same.
- Example: Disease prevalence changes across populations.

**Concept drift:** $p(y \mid x)$ changes over time.
- Example: Spam filter — spammers adapt their messages.
- Fix: Periodic retraining, online learning.

**Dataset shift:** Both $p(x)$ and $p(y|x)$ change.

### Domain Generalisation
Train a model that performs well across multiple domains, even ones not seen during training. Active research area.

---

## The No Free Lunch Theorem

**Theorem (Wolpert & Macready, 1997):** Averaged over all possible problems (all possible data distributions), no learning algorithm performs better than any other.

**Implication:** There is no universally best ML algorithm. Every algorithm embeds **inductive bias** — assumptions about the form of the true function. Good generalisation comes from matching your algorithm's inductive bias to the structure of your problem.

Examples of inductive bias:
- Linear models: the relationship is linear.
- CNNs: important patterns are local and translation-invariant.
- RNNs: relevant context is sequential.
- Gaussian Processes: the function is smooth.

---

## Connections

- [[Machine Learning]] — generalisation is the central goal
- [[Model]] — a model's complexity affects generalisation
- [[Training Data]] — more data improves generalisation
- [[Supervised Learning]] — evaluated by test performance, not train performance
- [[Inference]] — generalisation determines how well inference works on real data
- [[Features]] — better features reduce the learning problem, improving generalisation
- [[Grouped Train-Test Split]] — the generalisation gap $R(\hat f) - \hat R(\hat f)$ is only honestly measurable if train/test don't share entities
- [[Rolling-Origin Validation]] — tests generalisation specifically across *time*, a distribution-shift case this note's "Concept Drift" section describes conceptually
- [[Data Leakage]] — the most common way a measured generalisation gap is silently wrong (too optimistic) rather than the model being genuinely bad

---

## One-line Summary

> Generalisation is the ability to perform well on unseen data — it is the whole point of machine learning, governed by the bias-variance tradeoff, controlled via regularisation and data, and bounded by theory in terms of model complexity and sample size.
