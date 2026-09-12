# Machine Learning

## What is it?

**Machine Learning (ML)** is a sub-field of [[Artificial Intelligence]] where systems **learn patterns from data** and improve their performance on a task **without being explicitly programmed** for that task.

> Classical programming: `Data + Rules → Output`  
> Machine Learning: `Data + Output → Rules` (the rules are *learned*)

Formal definition (Tom Mitchell, 1997):

> *"A computer program is said to learn from experience **E** with respect to some task **T** and performance measure **P**, if its performance at **T**, as measured by **P**, improves with experience **E**."*

---

## The Three Ingredients

Every ML problem needs:

1. **Data** — examples the system can learn from. See [[Training Data]].
2. **A Model** — a mathematical structure with adjustable parameters. See [[Model]].
3. **A Learning Algorithm** — a procedure to adjust the model's parameters so it performs well on data.

---

## The Core Idea: Function Approximation

At its heart, ML is about learning a **function** $f$ that maps inputs to outputs:

$$f : \mathcal{X} \rightarrow \mathcal{Y}$$

Where:
- $\mathcal{X}$ = input space (e.g., all possible images, all possible sentences)
- $\mathcal{Y}$ = output space (e.g., labels, numbers, categories)

We never know the true $f$. We approximate it with a **model** $\hat{f}$ parameterised by $\theta$:

$$\hat{f}(x ; \theta) \approx f(x)$$

Learning = finding the $\theta$ that makes $\hat{f}$ as close to $f$ as possible, as measured on training data.

---

## Types of Machine Learning

### By supervision signal:

| Type | Has Labels? | Goal | Example |
|---|---|---|---|
| **Supervised** | Yes | Learn input→output mapping | Spam detection |
| **Unsupervised** | No | Find structure in data | Customer clustering |
| **Semi-supervised** | Partially | Use few labels + lots of unlabelled data | Medical imaging |
| **Self-supervised** | Labels from data itself | Learn representations | GPT pre-training |
| **Reinforcement** | Reward signal | Learn policy to maximise reward | Game playing, robotics |

See: [[Supervised Learning]], [[Unsupervised Learning]]

### By output type:

| Task | Output | Example |
|---|---|---|
| **Classification** | Discrete category | Is email spam? (Yes/No) |
| **Regression** | Continuous number | What is the house price? |
| **Clustering** | Group assignments | Which customers are similar? |
| **Generation** | New data samples | Write a sentence, generate an image |
| **Ranking** | Ordered list | Search results ordering |

---

## The ML Workflow

```
1. Define the Problem
        ↓
2. Collect & Prepare Data
        ↓
3. Choose a Model Architecture
        ↓
4. Define a Loss Function
        ↓
5. Optimise (Training)
        ↓
6. Evaluate & Validate
        ↓
7. Deploy (Inference)
        ↓
8. Monitor & Iterate
```

---

## The Loss Function

The **loss function** $\mathcal{L}(\theta)$ quantifies how wrong your model is.

Training = minimising the loss over the training set $\mathcal{D} = \{(x^{(i)}, y^{(i)})\}_{i=1}^{n}$:

$$\hat{\theta} = \arg\min_{\theta} \frac{1}{n} \sum_{i=1}^{n} \mathcal{L}(y^{(i)},\ \hat{f}(x^{(i)};\theta))$$

Common loss functions:

| Name | Formula | Used for |
|---|---|---|
| Mean Squared Error (MSE) | $\frac{1}{n}\sum(y - \hat{y})^2$ | Regression |
| Mean Absolute Error (MAE) | $\frac{1}{n}\sum|y - \hat{y}|$ | Regression (robust to outliers) |
| Binary Cross-Entropy | $-[y\log\hat{p} + (1-y)\log(1-\hat{p})]$ | Binary classification |
| Categorical Cross-Entropy | $-\sum_k y_k \log \hat{p}_k$ | Multi-class classification |
| Hinge Loss | $\max(0, 1 - y\hat{y})$ | SVMs |

---

## Optimisation: Gradient Descent

The most fundamental optimisation algorithm in ML.

**Idea:** Move $\theta$ in the direction that decreases $\mathcal{L}$ most steeply — i.e., opposite to the gradient.

$$\theta \leftarrow \theta - \eta \nabla_\theta \mathcal{L}(\theta)$$

Where:
- $\eta$ = **learning rate** (step size; a hyperparameter you choose)
- $\nabla_\theta \mathcal{L}$ = gradient of loss w.r.t. parameters

**Variants:**

| Variant | Uses | Update rule |
|---|---|---|
| Batch GD | Full dataset | $\theta \leftarrow \theta - \eta \frac{1}{n}\sum_i \nabla \mathcal{L}_i$ |
| Stochastic GD (SGD) | One sample | $\theta \leftarrow \theta - \eta \nabla \mathcal{L}_i$ |
| Mini-batch GD | Small batch of $b$ samples | Most common in practice |
| Adam | Mini-batch + adaptive LR | Standard for deep learning |

Adam update rule (for reference):
$$m_t = \beta_1 m_{t-1} + (1-\beta_1)g_t$$
$$v_t = \beta_2 v_{t-1} + (1-\beta_2)g_t^2$$
$$\theta_t = \theta_{t-1} - \eta \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

Where $g_t$ is the gradient at step $t$, $\beta_1 \approx 0.9$, $\beta_2 \approx 0.999$, $\epsilon \approx 10^{-8}$.

---

## Bias-Variance Tradeoff

Every ML model's error on unseen data can be decomposed as:

$$\text{Expected Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$$

| Term | Meaning | Caused by |
|---|---|---|
| **Bias** | Model is systematically wrong | Model too simple (underfitting) |
| **Variance** | Model is inconsistent across datasets | Model too complex (overfitting) |
| **Noise** | Inherent randomness in data | Can't fix this |

- **Underfitting:** High bias, low variance. Model can't capture the pattern.
- **Overfitting:** Low bias, high variance. Model memorises training data, fails on new data.

The goal: find the sweet spot — complex enough to capture the pattern, not so complex it memorises noise.

---

## Regularisation (Fighting Overfitting)

Add a penalty term to the loss to discourage overly complex models:

$$\mathcal{L}_{\text{reg}}(\theta) = \mathcal{L}(\theta) + \lambda \cdot \Omega(\theta)$$

| Method | Penalty $\Omega(\theta)$ | Effect |
|---|---|---|
| **L2 (Ridge)** | $\|\theta\|_2^2 = \sum_j \theta_j^2$ | Shrinks weights toward zero |
| **L1 (Lasso)** | $\|\theta\|_1 = \sum_j |\theta_j|$ | Drives some weights to exactly zero (sparse) |
| **Elastic Net** | $\alpha\|\theta\|_1 + (1-\alpha)\|\theta\|_2^2$ | Combines both |
| **Dropout** | Randomly zero out neurons during training | Neural network regularisation |

$\lambda$ = regularisation strength (hyperparameter).

---

## Hyperparameters vs Parameters

| | Parameters $\theta$ | Hyperparameters |
|---|---|---|
| **What** | Learned weights inside the model | Settings you choose before training |
| **Set by** | Training (optimisation) | You (manually or via search) |
| **Examples** | Neural net weights, linear regression coefficients | Learning rate, number of layers, batch size, $\lambda$ |

---

## Evaluation: Train / Validation / Test Split

Never evaluate on the data you trained on. Standard splits:

```
Full Dataset
├── Training set     (~60–80%)  ← model learns from this
├── Validation set   (~10–20%)  ← tune hyperparameters on this
└── Test set         (~10–20%)  ← final, unbiased evaluation (use ONCE)
```

**Cross-validation (k-fold):** When data is scarce, split data into $k$ folds, train on $k-1$, validate on 1, rotate, average results. Common: $k=5$ or $k=10$.

---

## Key Evaluation Metrics

### Regression
- **MSE:** $\frac{1}{n}\sum(y-\hat{y})^2$ — penalises large errors heavily
- **RMSE:** $\sqrt{\text{MSE}}$ — same units as $y$
- **MAE:** $\frac{1}{n}\sum|y-\hat{y}|$ — more robust to outliers
- **R² (coefficient of determination):** $1 - \frac{\sum(y-\hat{y})^2}{\sum(y-\bar{y})^2}$ — proportion of variance explained (1 = perfect, 0 = as good as predicting the mean)

### Classification
From the confusion matrix (binary):

|  | Predicted Positive | Predicted Negative |
|---|---|---|
| **Actual Positive** | TP (True Positive) | FN (False Negative) |
| **Actual Negative** | FP (False Positive) | TN (True Negative) |

$$\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}$$

$$\text{Precision} = \frac{TP}{TP + FP} \quad \text{(of all predicted positive, how many were right?)}$$

$$\text{Recall} = \frac{TP}{TP + FN} \quad \text{(of all actual positive, how many did we catch?)}$$

$$F_1 = 2 \cdot \frac{\text{Precision} \times \text{Recall}}{\text{Precision} + \text{Recall}} \quad \text{(harmonic mean)}$$

**ROC-AUC:** Area under the Receiver Operating Characteristic curve. Measures ranking ability regardless of threshold. AUC = 1 is perfect; AUC = 0.5 is random.

---

## Common ML Algorithms (High-level Map)

| Algorithm | Type | When to use |
|---|---|---|
| Linear Regression | Supervised, regression | Continuous output, linear relationship |
| Logistic Regression | Supervised, classification | Binary or multi-class |
| Decision Trees | Supervised | Interpretable, non-linear |
| Random Forest | Supervised | Robust ensemble of trees |
| Gradient Boosting (XGBoost, LightGBM) | Supervised | Tabular data, competitions |
| Support Vector Machine (SVM) | Supervised | Small-medium data, high-dimensional |
| k-Nearest Neighbours (kNN) | Supervised | Simple baseline, non-parametric |
| Neural Networks | Supervised / Unsupervised | Images, text, audio, complex patterns |
| k-Means | Unsupervised, clustering | Grouping unlabelled data |
| PCA | Unsupervised, dimensionality reduction | Compress features, visualise |

---

## Connections

- [[Artificial Intelligence]] — the parent field
- [[Supervised Learning]] — learning with labelled data
- [[Unsupervised Learning]] — learning without labels
- [[Training Data]] — the fuel for learning
- [[Features]] — the inputs to the model
- [[Labels]] — the targets in supervised learning
- [[Model]] — the learned function
- [[Inference]] — using the model
- [[Generalization]] — whether the model works on new data

---

## One-line Summary

> Machine Learning is the art of designing systems that automatically discover patterns in data by optimising a mathematical objective — turning raw examples into a function that generalises to new, unseen situations.
