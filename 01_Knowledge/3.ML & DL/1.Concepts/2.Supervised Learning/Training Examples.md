# Training Examples

## What is it?

A **training example** (also called a **training sample**, **data point**, or **instance**) is a single observation in the [[Training Data]] used to train a model.

In [[Supervised Learning]], each training example is a pair:

$$(x^{(i)},\ y^{(i)})$$

Where:
- $x^{(i)} \in \mathbb{R}^d$ = the **feature vector** (the input)
- $y^{(i)}$ = the **label** (the correct output)
- The superscript $^{(i)}$ indexes the $i$-th example, $i = 1, \ldots, n$

The full training set has $n$ examples:
$$\mathcal{D} = \{(x^{(1)}, y^{(1)}),\ (x^{(2)}, y^{(2)}),\ \ldots,\ (x^{(n)}, y^{(n)})\}$$

---

## Anatomy of a Training Example

Consider predicting house price from house features:

| Component | Symbol | Example value | Meaning |
|---|---|---|---|
| Feature 1: size | $x_1^{(i)}$ | 1500 (sq ft) | Input dimension 1 |
| Feature 2: rooms | $x_2^{(i)}$ | 3 | Input dimension 2 |
| Feature 3: age | $x_3^{(i)}$ | 20 (years) | Input dimension 3 |
| Full feature vector | $x^{(i)}$ | $[1500, 3, 20]^T$ | $\in \mathbb{R}^3$ |
| Label | $y^{(i)}$ | 320,000 (\$) | Target output |

---

## How Training Examples Drive Learning

The model learns by seeing many examples and adjusting parameters $\theta$ to minimise the total loss:

$$\hat{\theta} = \arg\min_{\theta} \frac{1}{n}\sum_{i=1}^{n} \mathcal{L}(y^{(i)},\ \hat{f}(x^{(i)};\theta))$$

Each example "votes" on what the parameters should be via its contribution to the gradient:

$$\nabla_\theta \mathcal{L} = \frac{1}{n}\sum_{i=1}^{n} \nabla_\theta \mathcal{L}_i$$

More examples → more votes → better estimate of the true gradient → better generalisation.

---

## How Many Examples Do You Need?

From statistical learning theory, to achieve generalisation error $\leq \epsilon$ with probability $\geq 1-\delta$:

$$n \geq \frac{1}{\epsilon}\left(d_{VC}\ln\frac{1}{\epsilon} + \ln\frac{1}{\delta}\right)$$

where $d_{VC}$ is the VC dimension of the model class. More complex models need more training examples.

**Rules of thumb:**
- Linear models: $n \gg d$ (at least 10× more examples than features)
- Deep learning: often $n \gg 10{,}000$ for image/text tasks
- The more complex the model, the more examples you need

---

## The i.i.d. Assumption

Training examples are assumed to be **independent and identically distributed (i.i.d.)**:

$$x^{(1)}, x^{(2)}, \ldots, x^{(n)} \overset{i.i.d.}{\sim} p_{\text{data}}(x)$$

**Independent:** Knowing example $i$ gives no information about example $j$.  
**Identically distributed:** All examples come from the same underlying distribution.

**Violations in practice:**
- Time series data (sequential dependency)
- Medical data from the same patient (repeated measures)
- Geographic data (spatial correlation)
- Data collection bias (certain examples more likely to be collected)

When i.i.d. is violated, standard theory breaks down and specialised methods are needed.

---

## Connections

- [[Training Data]] — the collection of all training examples
- [[Features]] — the $x^{(i)}$ part of each example
- [[Labels]] — the $y^{(i)}$ part of each example
- [[Supervised Learning]] — learns from (example, label) pairs
- [[Generalization]] — can the model do well on examples it hasn't seen?

---

## One-line Summary

> A training example is a single $(input, label)$ pair that the model learns from — the collection of all training examples is the evidence from which all patterns are extracted.
