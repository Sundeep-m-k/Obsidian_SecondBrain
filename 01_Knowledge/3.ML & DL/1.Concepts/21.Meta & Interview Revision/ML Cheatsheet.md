# ML Cheatsheet

A single-page reference for every core formula, rule, and decision you need in ML. Self-contained — sufficient to reconstruct the basics from memory.

**2026-09 update**: sections 1–15 (Foundations through RL) reflect the original build. Sections 16–24 extend coverage to everything added since — classical algorithms, evaluation/interpretability/tuning, data prep/MLOps, Deep Learning, Applied AI/LLM systems, and DS/Analytics — so this stays a true whole-system reference rather than one frozen mid-vault-growth. See [[ML & DL Overview]] for the full module map each section below maps to.

---

## 1. The ML Problem

| Scenario | Task | Output | Loss |
|---|---|---|---|
| Predict house price | Regression | $\hat{y} \in \mathbb{R}$ | MSE |
| Email spam? | Binary classification | $\hat{p} \in (0,1)$ | BCE |
| Which digit (0-9)? | Multi-class | $\hat{p}_k$, $k=1..K$ | CCE |
| Group customers | Clustering | Cluster labels | None (unsupervised) |

---

## 2. Core Models

### Linear Regression
$$\hat{y} = \theta^T x, \quad J = \frac{1}{2n}\|X\theta - y\|^2, \quad \hat{\theta} = (X^TX)^{-1}X^Ty$$

### Logistic Regression
$$\hat{p} = \sigma(\theta^T x) = \frac{1}{1+e^{-\theta^Tx}}, \quad J = -\frac{1}{n}\sum[y\log\hat{p}+(1-y)\log(1-\hat{p})]$$

### Softmax Regression (Multi-class)
$$P(Y=k\mid x) = \frac{e^{\theta_k^Tx}}{\sum_j e^{\theta_j^Tx}}, \quad J = -\frac{1}{n}\sum_i\log\hat{p}_{c^{(i)}}$$

---

## 3. Gradient Updates

**General GD:**
$$\theta \leftarrow \theta - \eta\nabla_\theta J(\theta)$$

**Linear Regression gradient:**
$$\nabla_\theta J = \frac{1}{n}X^T(X\theta - y)$$

**Logistic Regression gradient (same form!):**
$$\nabla_\theta J = \frac{1}{n}X^T(\hat{p} - y)$$

**With L2 regularisation (add to any gradient, skip $\theta_0$):**
$$\nabla_\theta J_{\text{reg}} = \nabla_\theta J + \lambda\theta$$

---

## 4. Key Activation Functions

| Name | Formula | Derivative |
|---|---|---|
| Sigmoid | $\sigma(z)=\frac{1}{1+e^{-z}}$ | $\sigma(z)(1-\sigma(z))$ |
| Tanh | $\tanh(z)=\frac{e^z-e^{-z}}{e^z+e^{-z}}$ | $1-\tanh^2(z)$ |
| ReLU | $\max(0,z)$ | $\mathbf{1}[z>0]$ |
| Softmax | $\frac{e^{z_k}}{\sum_j e^{z_j}}$ | (see cross-entropy combined) |

---

## 5. Loss Functions

| Task | Loss | Formula |
|---|---|---|
| Regression | MSE | $\frac{1}{n}\sum(y-\hat{y})^2$ |
| Regression | MAE | $\frac{1}{n}\sum|y-\hat{y}|$ |
| Binary classification | BCE | $-\frac{1}{n}\sum[y\log\hat{p}+(1-y)\log(1-\hat{p})]$ |
| Multi-class | CCE | $-\frac{1}{n}\sum\log\hat{p}_{c}$ |
| SVM | Hinge | $\frac{1}{n}\sum\max(0,1-y\hat{f})$ |

---

## 6. Regularisation

| Type | Penalty | Effect | Prior |
|---|---|---|---|
| L2 (Ridge) | $\frac{\lambda}{2}\|\theta\|^2$ | Shrink toward 0 | Gaussian |
| L1 (Lasso) | $\lambda\|\theta\|_1$ | Drive to exactly 0 (sparse) | Laplace |
| Elastic Net | $\lambda_1\|\theta\|_1+\frac{\lambda_2}{2}\|\theta\|^2$ | Both | — |

Ridge closed form: $\hat{\theta} = (X^TX + n\lambda I)^{-1}X^Ty$

---

## 7. Bias-Variance Decomposition

$$\mathbb{E}[(y-\hat{f}(x))^2] = \underbrace{(f(x)-\mathbb{E}[\hat{f}])^2}_{\text{Bias}^2} + \underbrace{\mathbb{E}[(\hat{f}-\mathbb{E}[\hat{f}])^2]}_{\text{Variance}} + \sigma_\epsilon^2$$

- High Bias → Underfitting → simpler model trains poorly
- High Variance → Overfitting → train much better than val

---

## 8. Evaluation Metrics

### Classification (Binary)

$$\text{Precision} = \frac{TP}{TP+FP}, \quad \text{Recall} = \frac{TP}{TP+FN}, \quad F_1 = \frac{2PR}{P+R}$$
$$\text{Accuracy} = \frac{TP+TN}{n}, \quad \text{AUC} = P(\hat{p}^+ > \hat{p}^-)$$

### Regression

$$\text{RMSE} = \sqrt{\frac{1}{n}\sum(y-\hat{y})^2}, \quad R^2 = 1 - \frac{\sum(y-\hat{y})^2}{\sum(y-\bar{y})^2}$$

---

## 9. Feature Scaling

$$\text{Standardise: } x' = \frac{x-\mu_{\text{train}}}{\sigma_{\text{train}}}, \quad \text{Min-Max: } x' = \frac{x-\min}{\max-\min}$$

Always fit on training set only. Apply same transform to val/test.

---

## 10. Data Splits

```
All Data (N)
├── Train    (70-80%)  ← fit θ
├── Val      (10-15%)  ← tune hyperparams
└── Test     (10-15%)  ← evaluate ONCE, at the end
```

---

## 11. K-Means

$$J = \sum_k\sum_{i\in C_k}\|x^{(i)}-\mu_k\|^2$$

Assign: $c^{(i)} = \arg\min_k\|x^{(i)}-\mu_k\|^2$  
Update: $\mu_k = \frac{1}{|C_k|}\sum_{i\in C_k}x^{(i)}$

---

## 12. PCA

1. Centre: $\tilde{X} = X - \bar{x}$
2. Covariance: $\Sigma = \frac{1}{n}\tilde{X}^T\tilde{X}$
3. Eigendecomp: $\Sigma = V\Lambda V^T$
4. Project: $Z = \tilde{X}V_k$ (top $k$ eigenvectors)
5. EVR$_k = \lambda_k / \sum_j \lambda_j$ (choose $k$ for 95% cumulative EVR)

---

## 13. Decision Rules

| Situation | Do this |
|---|---|
| Output continuous | Linear Regression |
| Output binary | Logistic Regression |
| Output multi-class | Softmax / Neural Net |
| Many irrelevant features | Lasso (L1) |
| Correlated features | Ridge (L2) |
| Train error high | More complexity, more features |
| Val >> Train error | Regularise, more data |
| Class imbalance | Class weights, tune threshold, use F1 |
| Many features, few examples | Regularise heavily; consider PCA first |
| New project baseline | Linear/Logistic Regression first, always |

---

## 14. RL Quick Reference

| Symbol | Meaning |
|---|---|
| $s, a, r, \gamma$ | State, action, reward, discount factor |
| $G_t = \sum_k\gamma^k r_{t+k}$ | Return (discounted cumulative reward) |
| $V^\pi(s) = \mathbb{E}[G_t\mid S_t=s]$ | State value function |
| $Q^\pi(s,a) = \mathbb{E}[G_t\mid S_t=s,A_t=a]$ | Q-function |
| $\pi^*(s) = \arg\max_a Q^*(s,a)$ | Optimal policy |
| TD error: $\delta_t = r_t+\gamma V(s_{t+1})-V(s_t)$ | Temporal difference error |

---

## 15. Common Mistakes Summary

| Mistake | Fix |
|---|---|
| Data leakage | Split first; fit preprocessing on train only |
| Evaluate on train data | Use val/test set |
| MSE for classification | Use cross-entropy |
| No scaling before GD | Standardise features |
| Always use 0.5 threshold | Tune threshold on val set |
| Ignore class imbalance | Use class weights, F1, AUC-PR |
| No baseline | Always start with simple baseline |
| Feature selection outside CV | Do feature selection inside each fold |

---

## 16. Classical Algorithms Quick Reference

| Algorithm | Core idea | Needs scaling? | Key hyperparameter |
|---|---|---|---|
| [[Decision Tree]] | Recursive axis-aligned splits | No | Max depth |
| [[Random Forest]] | Bagged trees + feature subsampling | No | # trees, max depth |
| [[Gradient Boosting]] | Sequential error correction | No | Learning rate, # trees |
| [[Support Vector Machines]] | Max-margin boundary, kernel trick | **Yes** | $C$, kernel, $\gamma$ |
| [[Naive Bayes]] | $P(y)\prod_j P(x_j\mid y)$, feature independence | No | Smoothing $\alpha$ |
| [[K-Nearest Neighbors]] | Majority vote among $k$ nearest points | **Yes** | $k$ |

Bagging → reduces **variance** ([[Bagging]]). Boosting → reduces **bias** ([[Boosting]]).

---

## 17. Confusion Matrix & Beyond

$$\begin{array}{c|cc} & \hat{y}=1 & \hat{y}=0 \\\hline y=1 & TP & FN \\ y=0 & FP & TN\end{array}$$

See [[Confusion Matrix]] for the worked imbalance example (96.5% accuracy, 33% precision). AUC-ROC for balanced data; **AUC-PR for imbalanced data** ([[ROC and PR Curves]]). [[Calibration and Probability Evaluation]]: a "70% confidence" prediction should be right ~70% of the time — check with a reliability diagram or Brier score, don't assume it.

---

## 18. Hyperparameter Search & Interpretability

| Need | Reach for |
|---|---|
| Few hyperparams, small grid | [[Hyperparameter Search Methods|Grid Search]] |
| More hyperparams, cheap trials | [[Hyperparameter Search Methods|Random Search]] (usually beats grid at same budget) |
| Expensive trials (deep nets) | [[Hyperparameter Search Methods|Bayesian Optimization]] |
| "Which features matter overall" | [[Feature Importance]] (permutation > impurity-based) |
| "Why this one prediction" — defensible | [[SHAP]] |
| "Why this one prediction" — fast, model-agnostic | [[LIME]] |

---

## 19. Data Prep & MLOps in One Table

| Stage | Rule |
|---|---|
| Missing values | <5% delete/mean, 5–20% KNN/MICE, >20% consider dropping the feature |
| Categorical encoding | Ordinal → label encode; nominal, few categories → one-hot; many categories → target encoding |
| [[Outlier Detection and Treatment\|Outliers]] | Detect via IQR/Z-score; trees robust, linear models sensitive |
| [[Feature Scaling]] | Standardize (unbounded) or normalize (bounded); fit on train only; skip for trees |
| [[Class Imbalance Handling]] | Class weights first, SMOTE if still poor, resample **after** split |
| [[Data Drift and Concept Drift\|Production drift]] | Data drift = input distribution shifts; concept drift = $P(y\mid x)$ itself shifts — monitor both |
| [[Deployment Strategies\|Deployment]] | Shadow → canary → full rollout, not a single big-bang switch |

---

## 20. Deep Learning Quick Reference

$$z^{(l)}=W^{(l)}a^{(l-1)}+b^{(l)}, \quad a^{(l)}=\sigma(z^{(l)}) \qquad \text{[[Forward Propagation]]}$$
$$\delta^{(l)} = (W^{(l+1)\top}\delta^{(l+1)})\odot\sigma'(z^{(l)}) \qquad \text{[[Backpropagation]]}$$

| Concept | One-liner |
|---|---|
| [[Activation Functions\|ReLU]] | Default hidden activation; gradient is 1 for $z>0$, avoids saturation |
| [[Weight Initialization\|He / Xavier]] | He for ReLU, Xavier for sigmoid/tanh — mismatching reintroduces vanishing/exploding gradients |
| [[Batch and Layer Normalization]] | BatchNorm normalizes across the batch; LayerNorm across each example's own features (why Transformers use LayerNorm) |
| [[Dropout]] | Randomly zero units during training only; implicit ensembling, fights co-adaptation |
| [[Optimizers in Deep Learning\|Adam]] | Momentum + per-parameter adaptive LR; the practical default |
| [[Attention Mechanism]] | $\text{softmax}(QK^\top/\sqrt{d_k})V$ — connects any two positions directly, no recurrence needed |
| [[Transformer Architecture]] | Self-attention + positional encoding + FFN + residual/LayerNorm; decoder-only (GPT) = causal masking, encoder-only (BERT) = bidirectional |
| [[Transformer End-to-End Walkthrough]] | "Explain a Transformer from scratch" — tokens → embeddings → attention (± causal mask) → vocab projection → logits → sampling, with a worked numerical example all the way through |

---

## 21. Applied AI / LLM Quick Reference

| Concept | One-liner |
|---|---|
| [[LLM Inference Fundamentals\|Context window]] | Hard limit from attention's ~quadratic cost; exceeding it silently truncates |
| [[Sampling and Decoding Strategies]] | Temperature reshapes the distribution before sampling; top-k caps candidate *count*; top-p caps candidate *cumulative probability* (adapts to confidence, top-k doesn't) |
| [[Query Rewriting]] | Expansion/decomposition/multi-query/HyDE/conversational rewriting — fixes queries too vague or context-dependent to retrieve well on their own |
| [[Dense vs Sparse Retrieval]] | BM25 (sparse) = exact terms; embeddings (dense) = semantic meaning; combine both ([[Reranking and Hybrid Search]]) |
| [[RAG Architecture]] | Retrieve → (rerank) → insert into prompt → generate; reduces but doesn't eliminate hallucination |
| [[RAG Evaluation]] | Context precision/recall (retrieval) vs. faithfulness/answer relevance (generation) — separate failure modes |
| [[Caching Strategies for LLM Systems]] | Prompt caching = reuse computation for an exact prefix; semantic caching = reuse a whole response for a *similar* query — mechanically different, different risks |
| [[Workflows vs Agents]] | Default to a fixed workflow when steps are knowable in advance; agent only when they genuinely aren't |
| [[Agent Failure Modes and Guardrails\|Agent guardrails]] | Iteration limits, least-privilege tool access, human-in-the-loop for high-stakes actions |
| [[Prompt Injection and Production Reliability\|Prompt injection]] | Untrusted fetched content interpreted as instructions — no clean fix like parameterized SQL; mitigate via scoping + validation |
| [[Design a Production RAG System for 10 Million Documents]] | The system-design anchor — composes every row above into one interview-format walkthrough |

---

## 22. Data Science / Analytics Quick Reference

| Concept | One-liner |
|---|---|
| [[Bayes' Theorem]] | Posterior ∝ likelihood × prior — ignoring the prior ("base rate neglect") is why a 95%-accurate test on a rare condition gives a much-lower-than-95% true positive rate |
| [[Central Limit Theorem]] | Sample means trend normal as $n$ grows, *regardless* of the population's own shape — why normal-based CIs work on non-normal data |
| [[Covariance and Correlation]] | Correlation only captures *linear* association — $\rho=0$ doesn't mean unrelated ($Y=X^2$ is a counterexample) |
| [[A-B Testing]] | Randomization breaks confounding; never peek-and-stop early (inflates false positives) |
| [[Correlation vs Causation]] | Confounding, reverse causation, selection bias, coincidence — only randomization (or quasi-experimental methods) establishes causation |
| [[Time Series Fundamentals]] | Decompose trend/seasonality/residual; check stationarity before ARIMA; **temporal split, never random** |
| [[Framing Ambiguous Business Problems]] | Clarify the goal and the decision it informs *before* naming a technique |
| [[Metric Selection and Diagnosing Change]] | Rule out measurement error → known external cause → segment → then investigate causally |
| [[Model vs Rule Decisions]] | Rule wins: simple pattern, scarce data, interpretability required. Model wins: complex/shifting pattern, enough data+monitoring maturity |

See [[Probability Foundations Index]] for the full random-variables/distributions/expectation/variance/Bayes/CLT layer this table assumes.

---

## 23. Whole-System Decision Tree (Start Here Under Time Pressure)

```
Is the problem even ML-shaped? ────────► No → see [[Model vs Rule Decisions]]
        │ Yes
Labeled target? ── No ──► Unsupervised (§11-12) or [[RAG Architecture|retrieval]]
        │ Yes
Output type? ── Continuous → Linear Regression baseline (§1-6)
             └─ Discrete  → Logistic Regression baseline, then classical algos (§16)
        │
Tabular + enough data + no strict interpretability need?
        └─ Yes → Gradient Boosting / Random Forest
Sequential/text/image + enough data?
        └─ Yes → Deep Learning (§20), or LLM + RAG/agent (§21) if it's a language task
Need to explain a specific decision? ─► [[SHAP]] / [[LIME]] (§18)
Need a business decision, not just a model? ─► [[Framing Ambiguous Business Problems]] first (§22)
```

---

## 24. Common Mistakes Summary — Extended (16-22)

| Mistake | Fix |
|---|---|
| Reading raw linear coefficients on unscaled features as "importance" | Scale first, or use permutation importance instead ([[Feature Importance]]) |
| Trusting one LIME explanation as globally valid | LIME is local only — use [[Feature Importance]] for global claims |
| Tuning hyperparameters on the test set | That's leakage — tune on validation, evaluate on test once |
| Random train/test split on time series | Temporal split only — random split leaks the future into training |
| A/B test: peeking daily, stopping at first $p<0.05$ | Fixed sample size decided upfront, or valid sequential testing |
| Assuming RAG eliminates hallucination | It reduces, not eliminates — the model can still ignore/misread context |
| Using a full autonomous agent when steps are knowable upfront | Use a fixed workflow — cheaper, faster, more debuggable ([[Workflows vs Agents]]) |
| No iteration limit on an agent loop | Always cap iterations — a loop has no inherent termination guarantee |
