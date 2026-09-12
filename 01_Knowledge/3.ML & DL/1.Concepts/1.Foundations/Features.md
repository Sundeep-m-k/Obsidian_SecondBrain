## What is it?

**Features** (also called **attributes**, **covariates**, **predictors**, or **input variables**) are the measurable properties of the data that a [[Machine Learning]] model uses as input to make predictions.

> A feature is an individual measurable property of the phenomenon being observed.

For a single data point, the features are collected into a **feature vector**:

$$x = (x_1, x_2, \ldots, x_d) \in \mathbb{R}^d$$

Where $d$ is the **dimensionality** (number of features).

---

## Concrete Examples

| Domain | Data Point | Features |
|---|---|---|
| House price prediction | A house | Square footage, number of rooms, postcode, age of building |
| Email spam detection | An email | Word frequencies, presence of certain phrases, sender address |
| Image classification | A photo | Pixel values (or learned convolutional features) |
| Medical diagnosis | A patient | Age, blood pressure, cholesterol level, test results |
| NLP | A sentence | Word embeddings, sentence length, POS tags |

---

## Notation

| Symbol | Meaning |
|---|---|
| $x^{(i)}$ | Feature vector of the $i$-th example |
| $x_j$ | The $j$-th feature (the $j$-th dimension) |
| $x_j^{(i)}$ | The $j$-th feature of the $i$-th example |
| $d$ | Number of features (dimensionality) |
| $n$ | Number of examples |
| $X$ | Design matrix of shape $n \times d$; row $i$ is $x^{(i)}$ |

**Design matrix:**
$$X = \begin{pmatrix} x_1^{(1)} & x_2^{(1)} & \cdots & x_d^{(1)} \\ x_1^{(2)} & x_2^{(2)} & \cdots & x_d^{(2)} \\ \vdots & & & \vdots \\ x_1^{(n)} & x_2^{(n)} & \cdots & x_d^{(n)} \end{pmatrix} \in \mathbb{R}^{n \times d}$$

---

## Types of Features

### By data type:

| Type | Description | Examples | Encoding |
|---|---|---|---|
| **Continuous / Numerical** | Real-valued measurements | Height (cm), temperature, price | Use directly (after scaling) |
| **Ordinal** | Ordered categories | Education: high school < bachelor's < master's | Integer encoding (preserves order) |
| **Nominal / Categorical** | Unordered categories | Country, colour, species | One-hot encoding |
| **Binary** | Two values | Yes/No, True/False, 0/1 | Use as 0/1 directly |
| **Text** | Free-form string | Product reviews, tweets | TF-IDF, word embeddings, etc. |
| **Image** | Grid of pixel values | Photos | Raw pixels or CNN features |
| **Time series** | Sequence of values over time | Stock prices, ECG | Lag features, FFT, RNNs |

### By construction:

| Type | Description |
|---|---|
| **Raw features** | Directly measured from the data source |
| **Derived features** | Computed from raw features (e.g., BMI = weight/height²) |
| **Interaction features** | Product of two features (e.g., $x_1 \cdot x_2$) |
| **Polynomial features** | Powers of features (e.g., $x_1^2, x_1^3$) |
| **Learned features** | Automatically extracted by a neural network (embeddings, convolutional filters) |

---

## Feature Engineering

**Feature engineering** is the process of transforming raw data into features that better represent the underlying problem to the model, improving model performance.

It requires domain knowledge and is often the difference between a mediocre model and a great one.

### Common Transformations

**Log transform** — for right-skewed features (e.g., income, population):
$$x' = \log(x + 1)$$
Compresses the range of large values, making the distribution more symmetric.

**Polynomial features** — capture non-linear relationships in linear models:
$$[x_1, x_2] \rightarrow [x_1, x_2, x_1^2, x_2^2, x_1 x_2]$$
A degree-$p$ polynomial of $d$ features creates $\binom{d+p}{p}$ features.

**Binning / Discretisation** — convert continuous to categorical:
$$x \rightarrow \text{bin}_k \quad \text{if } b_{k-1} \leq x < b_k$$

**Date/Time decomposition** — extract temporal structure:
`datetime` → `year`, `month`, `day_of_week`, `hour`, `is_weekend`

**Ratio features** — domain-specific ratios often carry signal:
$$\text{debt-to-income ratio} = \frac{\text{total debt}}{\text{annual income}}$$

---

## Feature Scaling

**Crucial for:** gradient descent, kNN, SVM, PCA, logistic regression.  
**Not needed for:** decision trees, random forests, gradient boosted trees (they use thresholds, not distances).

### Standardisation (Z-score):
$$x_j' = \frac{x_j - \mu_j}{\sigma_j}$$

After transformation: $\mu_j' = 0$, $\sigma_j' = 1$.

Compute $\mu_j$ and $\sigma_j$ **on the training set only**, then apply the same transformation to val/test sets. Never fit on the full dataset — that would leak test information.

### Min-Max Normalisation:
$$x_j' = \frac{x_j - \min_j}{\max_j - \min_j}$$

After: $x_j' \in [0, 1]$.

Sensitive to outliers (one extreme value compresses everything else).

### Robust Scaling (using quantiles):
$$x_j' = \frac{x_j - Q_{50}}{Q_{75} - Q_{25}}$$

Where $Q_{25}, Q_{50}, Q_{75}$ are the 25th, 50th, 75th percentiles. Robust to outliers.

---

## Feature Selection

**Why select features?**
- Reduces overfitting (fewer irrelevant features = less noise)
- Speeds up training and inference
- Improves interpretability
- Addresses the curse of dimensionality

### Filter Methods (model-agnostic, computed on data directly)

**Variance threshold:** Remove features with very low variance:
$$\text{Var}(x_j) = \frac{1}{n}\sum_i (x_j^{(i)} - \mu_j)^2 < \text{threshold}$$
A feature with near-zero variance is nearly constant — it carries no information.

**Correlation with target (for regression):**
$$r_{x_j, y} = \frac{\sum_i (x_j^{(i)} - \bar{x}_j)(y^{(i)} - \bar{y})}{\sqrt{\sum_i(x_j^{(i)}-\bar{x}_j)^2} \cdot \sqrt{\sum_i(y^{(i)}-\bar{y})^2}}$$

Select features with $|r| >$ some threshold.

**Mutual Information:**
$$I(X_j; Y) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x)p(y)}$$

Captures non-linear dependencies. $I = 0$ means independence.

**Chi-squared test** (for categorical features and categorical labels):
$$\chi^2 = \sum_i \frac{(O_i - E_i)^2}{E_i}$$

Tests whether the feature and label are statistically independent.

### Wrapper Methods (use model performance as the criterion)

- **Forward selection:** Start with no features. Greedily add the feature that most improves validation performance. Repeat.
- **Backward elimination:** Start with all features. Greedily remove the feature whose removal hurts performance least. Repeat.
- **Recursive Feature Elimination (RFE):** Train a model, remove the least important feature, retrain, repeat.

Wrappers are expensive but typically find better feature subsets.

### Embedded Methods (selection happens during model training)

**L1 (Lasso) Regularisation:**
$$\mathcal{L}_{\text{Lasso}} = \text{MSE} + \lambda \|\theta\|_1 = \text{MSE} + \lambda \sum_j |\theta_j|$$

L1 regularisation drives many weights $\theta_j$ to exactly zero, performing automatic feature selection. The non-zero features are selected.

**Tree-based feature importance:**
Decision trees and Random Forests compute feature importance as the total reduction in impurity (Gini or entropy) attributable to each feature:
$$\text{Importance}(j) = \sum_{\text{nodes where } j \text{ is used}} n_t \cdot \Delta I_t$$

where $n_t$ = samples at node $t$, $\Delta I_t$ = impurity reduction at node $t$.

---

## The Curse of Dimensionality

As the number of features $d$ grows, the geometry of high-dimensional spaces becomes pathological.

**Key problem:** The volume of a $d$-dimensional unit hypercube is $1^d = 1$. The volume of a $d$-dimensional hypersphere of radius $r=1$ is:
$$V_d = \frac{\pi^{d/2}}{\Gamma(d/2+1)}$$

This shrinks toward 0 as $d \to \infty$. Most of the volume of a high-dimensional hypercube is in the corners, not the centre. **Data becomes sparse.**

**Practical consequence:** In high dimensions:
- kNN fails — all pairwise distances become similar
$$\lim_{d\to\infty} \frac{\max_i d(q, x^{(i)}) - \min_i d(q, x^{(i)})}{\min_i d(q, x^{(i)})} = 0$$
- You need exponentially more data to maintain the same data density
- Models overfit easily

**Solutions:** Dimensionality reduction (PCA, autoencoders), feature selection, regularisation.

---

## Representation Learning

Rather than manually engineering features, **deep learning** learns features automatically:

```
Raw input x → [Layer 1] → low-level features → [Layer 2] → mid-level → ... → [Output]
```

In CNNs: layer 1 learns edges, layer 2 learns textures, layer 3 learns object parts.  
In NLP: word embeddings compress vocabulary into dense vectors where semantically similar words are nearby.

**Word embeddings** (Word2Vec, GloVe):
- Each word maps to a vector $w \in \mathbb{R}^{300}$ (typically)
- Trained to preserve semantic similarity: $\cos(\vec{king} - \vec{man} + \vec{woman}) \approx \vec{queen}$
- Cosine similarity: $\text{sim}(u, v) = \frac{u \cdot v}{\|u\| \|v\|}$

---

## Connections

- [[Machine Learning]] — features are the input representation
- [[Training Data]] — features come from the raw data
- [[Labels]] — features are paired with labels in supervised learning
- [[Model]] — the model operates on features
- [[Supervised Learning]] — model learns feature-to-label mapping
- [[Unsupervised Learning]] — features are clustered or compressed

---

## One-line Summary

> Features are the numerical representation of the input data — the language in which the model understands the world; choosing, transforming, and selecting the right features is often the most impactful step in building an effective ML system.
