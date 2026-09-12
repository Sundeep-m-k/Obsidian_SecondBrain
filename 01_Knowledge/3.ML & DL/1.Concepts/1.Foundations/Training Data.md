## What is it?

**Training data** is the collection of examples that a [[Machine Learning]] model learns from. It is the empirical evidence from which the model extracts patterns, adjusts its parameters, and builds its ability to make predictions.

> Without data, there is no machine learning. The model is only as good as the data it was trained on.

---

## Formal Definition

A training dataset is a finite set of examples:

$$\mathcal{D}_{\text{train}} = \{(x^{(i)}, y^{(i)})\}_{i=1}^{n} \quad \text{(supervised)}$$

or

$$\mathcal{D}_{\text{train}} = \{x^{(i)}\}_{i=1}^{n} \quad \text{(unsupervised)}$$

Where:
- $n$ = number of training examples (the **sample size**)
- $x^{(i)} \in \mathbb{R}^d$ = the **feature vector** of the $i$-th example (see [[Features]])
- $y^{(i)}$ = the **label** or target of the $i$-th example (see [[Labels]])

**The fundamental assumption:** Training data is assumed to be drawn **independently and identically distributed (i.i.d.)** from some unknown true distribution $p_{\text{data}}(x, y)$:

$$\mathcal{D}_{\text{train}} \sim p_{\text{data}}$$

The model must generalise from this sample to the full distribution. See [[Generalization]].

---

## The Data Splits

You never use all your data for training. The standard split:

```
Full Dataset  (N examples)
│
├── Training Set     (~60–80% of N)
│     └── The model sees this during optimisation.
│         Parameters θ are adjusted to minimise loss here.
│
├── Validation Set   (~10–20% of N)
│     └── Used to tune hyperparameters and monitor overfitting.
│         Model does NOT train on this. You can look at it many times.
│
└── Test Set         (~10–20% of N)
      └── Used ONCE at the very end to report final performance.
          Never used during model development. Touching it early = data leakage.
```

**Why separate test set?** If you tune your model based on test set performance, the test set is no longer an unbiased estimate of generalisation. You overfit to the test set.

---

## Cross-Validation

When data is scarce, use **k-fold cross-validation** to get a reliable performance estimate without wasting data:

1. Divide training data into $k$ equal folds.
2. For each fold $i$:
   - Train on all folds except $i$
   - Evaluate on fold $i$
3. Average the $k$ evaluation scores.

$$\text{CV Score} = \frac{1}{k}\sum_{i=1}^{k} \text{Score}_i$$

Typical values: $k = 5$ or $k = 10$.

**Stratified k-fold:** For classification, ensure each fold has the same class distribution as the full dataset. Important for imbalanced data.

**Leave-One-Out CV (LOOCV):** $k = n$ — train on $n-1$ examples, test on 1, repeat $n$ times. Unbiased but expensive for large datasets.

---

## Properties of Good Training Data

### 1. Representative
Data must represent the real distribution the model will encounter at deployment. If your test distribution differs from training distribution, performance will degrade — this is called **distribution shift** or **covariate shift**.

$$p_{\text{train}}(x) \neq p_{\text{test}}(x) \Rightarrow \text{performance drop}$$

### 2. Sufficient Volume
More data generally means better models. The relationship between sample size $n$ and error $\epsilon$ is roughly:

$$\epsilon \propto \frac{1}{\sqrt{n}} \quad \text{(statistical learning theory)}$$

The **VC dimension** (Vapnik-Chervonenkis dimension) of a model class tells you how many examples you need:

$$n \geq \frac{1}{\epsilon}\left(d_{VC} \ln \frac{1}{\epsilon} + \ln \frac{1}{\delta}\right)$$

to achieve error $\leq \epsilon$ with probability $\geq 1-\delta$. Deep learning models have enormous VC dimension, needing massive datasets.

### 3. High Quality / Accurate Labels
Label noise — wrong labels in the training set — degrades model performance. Even 5–10% label noise can significantly hurt accuracy.

### 4. Diverse
Covering the variation in the real world. A model trained only on daytime photos will struggle at night.

### 5. Balanced (for classification)
Imbalanced classes lead to biased models. A dataset that is 99% class A and 1% class B will produce a model that predicts class A nearly always.

---

## Data Quality Problems

| Problem | Description | Fix |
|---|---|---|
| **Missing values** | Features are NaN or blank | Imputation (mean, median, model-based), or drop |
| **Outliers** | Extreme values far from the distribution | Cap, remove, or use robust methods |
| **Label noise** | Incorrect labels | Re-annotation, robust loss functions, label smoothing |
| **Class imbalance** | One class dominates | Oversampling (SMOTE), undersampling, class weights |
| **Duplicate examples** | Same example appears multiple times | Deduplication |
| **Data leakage** | Future/test info leaks into training | Careful preprocessing pipelines |
| **Selection bias** | Data not representative of deployment | Better sampling strategy |
| **Distribution shift** | Train & test distributions differ | Domain adaptation, diverse collection |

---

## Data Augmentation

**Artificially expand the training set** by applying label-preserving transformations.

### Images
- Random crop, flip (horizontal/vertical), rotation
- Colour jitter (brightness, contrast, saturation, hue)
- Gaussian noise, blur
- Cutout / Random Erasing
- Mixup: $\tilde{x} = \lambda x_i + (1-\lambda) x_j$, $\tilde{y} = \lambda y_i + (1-\lambda) y_j$

### Text
- Synonym replacement
- Random insertion, deletion, swap of words
- Back-translation (translate to French, then back to English)

### Time Series
- Time warping
- Window slicing
- Gaussian noise injection

Augmentation reduces overfitting and improves generalisation, especially when labelled data is scarce.

---

## Data Preprocessing Pipeline

Before training, raw data must be cleaned and transformed:

### Feature Scaling

Most algorithms (gradient descent, kNN, SVM, PCA) are sensitive to the scale of features. Always scale before using these.

**Standardisation (Z-score normalisation):**
$$x_j' = \frac{x_j - \mu_j}{\sigma_j}$$
After: mean = 0, std = 1.

**Min-Max Normalisation:**
$$x_j' = \frac{x_j - \min_j}{\max_j - \min_j}$$
After: values in $[0, 1]$.

**When to use which:**
- Standardise when you expect approximately Gaussian features or use algorithms that assume zero-mean.
- Min-max when you need bounded values (e.g., pixel values for neural nets).

### Encoding Categorical Variables

**One-hot encoding:** For a feature with $K$ categories, create $K$ binary features:
- "Color: {Red, Green, Blue}" → `[1, 0, 0]` (Red), `[0, 1, 0]` (Green), `[0, 0, 1]` (Blue)

**Ordinal encoding:** Map categories to integers — only meaningful when there is a real ordering (e.g., Low=1, Medium=2, High=3).

**Embedding:** For high-cardinality categories (e.g., user IDs, words), learn a dense vector representation of size $d$. Standard in NLP (word embeddings).

### Handling Missing Values
- **Drop:** remove rows or columns with too many missing values
- **Mean/median imputation:** fill with the column mean (continuous) or mode (categorical)
- **Model-based imputation:** train a model to predict the missing value from other features
- **Flag + impute:** add a binary `is_missing` indicator column, then impute — lets the model learn that missingness itself is informative

---

## The Data Generating Process

It's useful to think about *where* data comes from:

$$x^{(i)}, y^{(i)} \sim p(x, y)$$

We can decompose:
$$p(x, y) = p(y \mid x) \cdot p(x)$$

- $p(x)$ = **marginal distribution of inputs** (what inputs look like in the world)
- $p(y \mid x)$ = **conditional distribution of labels given inputs** (the "true" relationship we want to learn)

The model learns an approximation $\hat{p}(y \mid x) \approx p(y \mid x)$ from a finite sample.

**The fundamental problem:** We only see a finite sample from $p(x,y)$. There are infinitely many functions consistent with this sample. Which one to choose? This is the **inductive bias** of the algorithm — the implicit assumptions the algorithm makes to prefer one hypothesis over another.

---

## Data Efficiency and Scaling Laws

Empirical observation (especially in deep learning):

$$\text{Loss} \propto N^{-\alpha}$$

where $N$ is the amount of training data and $\alpha$ is a scaling exponent (varies by domain, model, and compute). This means:

- Doubling data reduces loss by a predictable amount.
- Large models often continue improving with more data far beyond what smaller models need.
- The famous **Chinchilla scaling laws** (Hoffmann et al., 2022) found that for language models, the optimal number of training tokens $\approx 20 \times$ the number of model parameters.

---

## Connections

- [[Machine Learning]] — training data is what makes ML possible
- [[Features]] — the $x$ part of each training example
- [[Labels]] — the $y$ part (in supervised learning)
- [[Supervised Learning]] — explicitly requires labelled training data
- [[Unsupervised Learning]] — uses unlabelled training data
- [[Model]] — learns from training data
- [[Generalization]] — whether the patterns from training data transfer to new data

---

## One-line Summary

> Training data is the finite sample of labelled or unlabelled examples from which a model learns; its quality, size, and representativeness are the single biggest determinants of model performance.
