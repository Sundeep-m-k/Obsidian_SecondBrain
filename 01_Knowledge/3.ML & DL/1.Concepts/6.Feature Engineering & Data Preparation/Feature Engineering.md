# Feature Engineering

## What is it?

**Feature engineering** is the process of using domain knowledge to transform raw data into **features** that better represent the underlying problem to a machine learning model, thereby improving its performance.

> "Applied machine learning is basically feature engineering." — Andrew Ng

---

## Why It Matters

The same algorithm trained on better features almost always outperforms a more complex algorithm trained on raw features. Feature engineering bridges the gap between raw data and learnable patterns.

**The pipeline:**
```
Raw Data → [Feature Engineering] → Feature Vector → [Model] → Prediction
```

---

## Core Techniques

### 1. Mathematical Transformations

**Log transform** (for right-skewed data like income, counts):
$$x' = \log(x + 1)$$
Compresses large values, expands small ones. Makes distributions more Gaussian.

**Square root / Box-Cox:**
$$x'(\lambda) = \begin{cases} \frac{x^\lambda - 1}{\lambda} & \lambda \neq 0 \\ \log x & \lambda = 0 \end{cases}$$
Choose $\lambda$ that maximises normality of transformed $x$.

**Polynomial features:** Add $x^2, x^3, x_1 x_2$ to capture non-linear and interaction effects. See [[Polynomial Regression]].

### 2. Binning / Discretisation

Convert a continuous feature into categorical bins:
$$x' = k \quad \text{if } b_{k-1} \leq x < b_k$$

Useful when the relationship with $y$ is non-monotonic (e.g., age groups matter, not exact age). Loses some information but can reduce noise.

### 3. Encoding Categorical Features

**One-hot encoding:** For a feature with $K$ categories, create $K$ binary indicators.
- Adds sparsity; preferred for nominal categories.
- Watch out: for $K$ categories, only $K-1$ are needed (the last is implied) to avoid multicollinearity — called **dummy variable trap**.

**Target encoding:** Replace category $c$ with the mean of $y$ for examples with category $c$:
$$x'_c = \frac{1}{|S_c|}\sum_{i: x^{(i)}=c} y^{(i)}$$
Compact (single feature), but prone to data leakage if not done inside cross-validation folds.

### 4. Interaction Features

Multiply two features together to capture their joint effect:
$$x_{\text{interaction}} = x_j \cdot x_k$$

Example: "number of rooms × average room size" may capture house quality better than either alone.

### 5. Date and Time Decomposition

Extract multiple features from a timestamp:
- `year`, `month`, `day`, `hour`, `minute`
- `day_of_week` (0=Monday, 6=Sunday)
- `is_weekend` (binary)
- `days_since_event` (e.g., days since last purchase)
- Cyclical encoding for periodic features (e.g., hour):

$$\sin\!\left(\frac{2\pi \cdot \text{hour}}{24}\right), \quad \cos\!\left(\frac{2\pi \cdot \text{hour}}{24}\right)$$

This preserves the circular nature: 23:00 is close to 00:00.

### 6. Aggregation Features

From related records, compute statistics:
- Customer's mean/max/min/std of past purchases
- User's total clicks in the last 7 days
- Moving average, rolling standard deviation

### 7. Missing Value Indicators

Add a binary feature $m_j \in \{0,1\}$ indicating whether feature $j$ is missing, then impute the missing value. Lets the model learn from the fact of missingness itself.

### 8. Domain-Specific Features

Example (medical): body mass index:
$$\text{BMI} = \frac{\text{weight (kg)}}{\text{height (m)}^2}$$

Example (finance): debt-to-income ratio:
$$\text{DTI} = \frac{\text{total debt}}{\text{annual income}}$$

These encode domain knowledge directly into features.

---

## The Feature Engineering Process

1. **Understand the problem** — what are the real-world signals that determine $y$?
2. **Explore the data** — correlations, distributions, outliers
3. **Hypothesis** — which raw features capture this signal?
4. **Transform** — apply operations to extract signal
5. **Evaluate** — does adding the feature improve validation performance?
6. **Iterate**

---

## Automated Feature Engineering

For tabular data, tools like **Featuretools** automatically generate aggregation features from relational data. For text and images, **deep learning** learns features automatically from raw data (see [[Model]]'s representation learning section).

---

## Connections

- [[Features]] — the inputs produced by feature engineering
- [[Feature Vector]] — the vector $x$ resulting from engineering
- [[Feature Scaling]] — normalisation/standardisation applied after engineering
- [[Polynomial Regression]] — a specific feature engineering technique
- [[Multiple Features]] — multiple engineered features as inputs

---

## One-line Summary

> Feature engineering is the art of transforming raw data into informative numerical inputs using domain knowledge, mathematical transforms, and data-driven techniques — the quality of features is often more important than the choice of model.
