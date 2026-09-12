# Naive Bayes

## What is it?

**Naive Bayes** is a probabilistic classifier built directly from Bayes' theorem, with one simplifying ("naive") assumption: every feature is conditionally independent of every other feature, given the class.

$$P(y \mid x_1,\dots,x_d) \propto P(y)\prod_{j=1}^{d} P(x_j \mid y)$$

The prediction is whichever class maximizes this product:
$$\hat{y} = \arg\max_y \ P(y)\prod_{j=1}^{d} P(x_j \mid y)$$

---

## Why "Naive"?

Real features are rarely independent given the class — in spam detection, "free" and "money" appearing together is more informative than either alone, which the independence assumption throws away. Despite this, Naive Bayes is a surprisingly strong baseline in practice: it only needs the *ranking* of classes to be right, not the exact probabilities, and the independence assumption's errors often cancel out across many features rather than compounding.

---

## The Three Common Variants

**Multinomial Naive Bayes** — features are counts (e.g. word counts in a document). Standard for text classification with [[Bag of Words]] or [[TF-IDF]] features.

**Bernoulli Naive Bayes** — features are binary (word present/absent, not count). Penalizes the *absence* of a feature explicitly, unlike Multinomial.

**Gaussian Naive Bayes** — features are continuous, each assumed to follow a Gaussian distribution per class: $P(x_j\mid y) = \mathcal{N}(x_j; \mu_{y,j}, \sigma_{y,j}^2)$.

---

## Why It's a Strong Baseline Despite the Independence Assumption

Training is a single pass over the data to estimate $P(y)$ and each $P(x_j\mid y)$ — no iterative optimization at all, which makes it extremely fast and a natural first model to try before reaching for something heavier. It also handles high-dimensional sparse data (thousands of vocabulary features) gracefully, since each feature's contribution is estimated independently rather than jointly, avoiding the curse of dimensionality that trips up distance-based methods like [[K Means]] or KNN.

## Laplace (Additive) Smoothing

If a word never appeared with a given class in training, $P(x_j\mid y)=0$, which zeroes out the entire product regardless of how strong the other evidence is. **Laplace smoothing** fixes this by adding a small count to every feature-class pair:

$$P(x_j \mid y) = \frac{\text{count}(x_j, y) + \alpha}{\text{count}(y) + \alpha \cdot |\text{vocab}|}$$

$\alpha=1$ is the standard default; without this, a single unseen word at inference time can silently override an otherwise confident prediction.

---

## Advantages / Disadvantages

**Advantages:** trains in one pass (no gradient descent), works well with high-dimensional sparse data, needs relatively little training data to estimate its parameters, naturally multi-class.

**Disadvantages:** the independence assumption means it usually isn't the most *accurate* model available; probability outputs tend to be poorly calibrated (overconfident) even when the class ranking is correct — see [[Calibration and Probability Evaluation]] before trusting its raw probabilities for anything beyond ranking.

## When to Use / When Not To

**Use** as a fast baseline for text classification (spam filtering, sentiment) or any high-dimensional, sparse-feature problem where training speed matters and features are plausibly near-independent. **Avoid** when features are strongly correlated and that correlation carries real signal (a [[Logistic Regression]] or tree-based model will use it; Naive Bayes cannot), or when calibrated probabilities are required downstream.

---

## Interview Questions

**Why call it "naive," and does that assumption actually break it in practice?** It assumes features are independent given the class, which is essentially always false in real data — but the model only needs the class *ranking* correct, not the exact probabilities, and it remains a strong, fast baseline especially for text.

**Why does Naive Bayes need Laplace smoothing?** Without it, any feature value unseen with a given class during training gives that class a likelihood of exactly zero for any input containing it, regardless of how much other evidence favors that class — smoothing prevents one unseen feature from silently overriding everything else.

**When would you reach for Naive Bayes over logistic regression?** When training speed and high-dimensional sparse features (e.g. large text vocabularies) matter more than squeezing out the last bit of accuracy, or as a quick baseline to beat before justifying a heavier model.

## Connections

- [[Bag of Words]], [[TF-IDF]] — the feature representations Multinomial/Bernoulli NB is typically fed
- [[Calibration and Probability Evaluation]] — NB's probability outputs are usually overconfident and need recalibration
- [[Hypothesis Test]] — Bayes' theorem underlies both, applied to classification here rather than significance testing
- [[Logistic Regression]] — the discriminative alternative that doesn't assume feature independence

## One-line Summary

> Naive Bayes predicts the class that maximizes $P(y)\prod_j P(x_j\mid y)$ under a (usually false but usually harmless) feature-independence assumption — fast, effective on high-dimensional sparse text data, but its probability outputs need recalibration before being trusted directly.
