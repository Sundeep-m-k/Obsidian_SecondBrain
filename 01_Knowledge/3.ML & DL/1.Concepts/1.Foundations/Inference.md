# Inference

## What is it?

**Inference** (also called **prediction**, **serving**, or **deployment**) is the process of using a **trained** [[Model]] to make predictions on new, unseen data.

$$\text{Inference: } x_{\text{new}} \xrightarrow{\hat{f}(\cdot;\hat{\theta})} \hat{y}_{\text{new}}$$

> Training is learning. Inference is applying what was learned.

Training happens once (or periodically). Inference happens continuously, often millions of times per day in production systems.

---

## Training vs. Inference: Key Differences

| | Training | Inference |
|---|---|---|
| **Goal** | Optimise parameters $\theta$ | Use fixed $\theta$ to predict |
| **Data** | Training set $(x^{(i)}, y^{(i)})$ | New input $x$ only — label unknown |
| **Computational cost** | High (backward pass + optimiser) | Low (forward pass only) |
| **Frequency** | Once or periodically | Continuously in production |
| **Gradients** | Required | Not needed |
| **Batch size** | Large (mini-batches for efficiency) | Often 1 (single request) or small |

In PyTorch, this distinction is explicit:
```python
# Training mode
model.train()
output = model(x)
loss = criterion(output, y)
loss.backward()   # Compute gradients
optimizer.step()  # Update weights

# Inference mode
model.eval()
with torch.no_grad():  # Disable gradient computation
    output = model(x)  # Only forward pass
```

`torch.no_grad()` reduces memory usage and speeds up computation during inference.

---

## Types of Inference

### Batch Inference
- Process a **large number of examples at once** (offline).
- Examples are collected, the model runs on the whole batch, results are stored.
- Latency is not critical.
- **Use case:** Generating recommendations overnight, scoring all emails in a queue.

### Online / Real-time Inference
- Process **one example at a time** (or very small batches) as requests arrive.
- Latency is critical — milliseconds matter.
- **Use case:** Fraud detection at transaction time, autocomplete, search ranking.

### Edge Inference
- Run the model directly **on a device** (phone, IoT sensor, embedded system) rather than a server.
- Requires model compression (quantisation, pruning, distillation).
- **Use case:** Face recognition on your phone, voice commands on a smart speaker, autonomous driving.

### Streaming Inference
- Process a **continuous stream** of data in real time.
- **Use case:** Real-time sensor monitoring, live translation, video analysis.

---

## The Inference Pipeline

In production, inference is rarely just a single model forward pass. The full pipeline:

```
Raw Input
    ↓
1. Preprocessing
   (same transforms as training: scale, encode, tokenise, resize)
    ↓
2. Feature Extraction
   (optional: extract features from raw input)
    ↓
3. Model Forward Pass
   ŷ = f̂(x; θ̂)
    ↓
4. Post-processing
   (threshold, argmax, decode tokens, format output)
    ↓
5. Output
   (probability, class label, text, bounding box, etc.)
```

**Critical: Preprocessing must be identical at inference and training time.** If you standardised features during training using training-set mean and std, you must use those exact same values at inference. Storing these statistics is part of the model artefact.

---

## Probabilistic Inference

For probabilistic models, inference means computing **posterior distributions**, not just point estimates.

**Bayes' Theorem:**
$$p(\theta \mid \mathcal{D}) = \frac{p(\mathcal{D} \mid \theta) \cdot p(\theta)}{p(\mathcal{D})}$$

- $p(\theta)$ = **prior** — belief about parameters before seeing data
- $p(\mathcal{D} \mid \theta)$ = **likelihood** — how probable is the data given these parameters?
- $p(\theta \mid \mathcal{D})$ = **posterior** — updated belief after seeing data
- $p(\mathcal{D})$ = **evidence** (normalisation constant): $p(\mathcal{D}) = \int p(\mathcal{D} \mid \theta)p(\theta) d\theta$

**Predictive distribution:**
$$p(y^* \mid x^*, \mathcal{D}) = \int p(y^* \mid x^*, \theta) \cdot p(\theta \mid \mathcal{D})\ d\theta$$

This averages predictions over all possible parameter settings, weighted by their posterior probability — giving **uncertainty-aware predictions**.

In practice, this integral is intractable for large models. Approximations:
- **MAP (Maximum A Posteriori):** $\hat{\theta}_{\text{MAP}} = \arg\max_\theta p(\theta \mid \mathcal{D})$ — use the single most probable parameter setting.
- **MCMC:** Sample from the posterior using Markov Chain Monte Carlo.
- **Variational Inference (VI):** Approximate the posterior with a tractable distribution $q(\theta) \approx p(\theta \mid \mathcal{D})$ by minimising $KL(q \| p)$.
- **Monte Carlo Dropout:** At inference, keep dropout active and run multiple forward passes to sample from an approximate posterior.

---

## Output Interpretation

### Classification Outputs

**Softmax output** produces a **probability distribution** over $K$ classes:
$$\hat{p}_k = \frac{e^{z_k}}{\sum_{j=1}^K e^{z_j}} \quad \Rightarrow \quad \sum_{k=1}^K \hat{p}_k = 1$$

The **prediction** is:
$$\hat{y} = \arg\max_k \hat{p}_k$$

The **confidence** is $\max_k \hat{p}_k$.

**Calibration:** A model is **calibrated** if $\hat{p}_k$ reflects the true probability. When a calibrated model says 70% confidence, it should be right 70% of the time.

**Platt scaling:** After training, fit a logistic regression on the model's logits using a held-out calibration set:
$$p_{\text{calibrated}} = \sigma(a \cdot z + b)$$

**Temperature scaling:** Divide logits by a scalar $T$ before softmax:
$$\hat{p}_k = \frac{e^{z_k/T}}{\sum_j e^{z_j/T}}$$

$T > 1$: Softer (less confident) predictions.  
$T < 1$: Harder (more confident) predictions.

### Regression Outputs

The model outputs a single real number $\hat{y}$.

**Prediction interval** (not the same as confidence interval):
- A 95% prediction interval means 95% of new observations fall within the interval.
- For Gaussian errors with known variance $\sigma^2$:
$$\hat{y} \pm 1.96\sigma$$

---

## Decision Thresholds

For binary classification, the default decision threshold is 0.5:
$$\hat{y} = \begin{cases} 1 & \text{if } \hat{p} \geq 0.5 \\ 0 & \text{if } \hat{p} < 0.5 \end{cases}$$

But this is often not optimal. The threshold should reflect the **cost asymmetry**:
- In fraud detection: missing a fraud (FN) is much worse than a false alarm (FP). Lower threshold.
- In medical screening: missing a disease is worse than a false positive. Lower threshold.

**ROC Curve:** Plot True Positive Rate (Recall) vs. False Positive Rate at all thresholds.

$$TPR = \frac{TP}{TP + FN}, \quad FPR = \frac{FP}{FP + TN}$$

**Precision-Recall Curve:** Plot Precision vs. Recall at all thresholds. Better for imbalanced classes.

The optimal threshold depends on the specific cost structure of the application:
$$\text{Expected cost} = c_{FP} \cdot FP + c_{FN} \cdot FN$$

Minimise this by adjusting the threshold.

---

## Inference Efficiency

### Latency and Throughput

- **Latency:** time to process one request (milliseconds). Key for real-time systems.
- **Throughput:** requests processed per second. Key for batch systems.
- There is a tradeoff: batching increases throughput but increases latency for individual requests.

### Hardware

| Hardware | Best for | Notes |
|---|---|---|
| **CPU** | Small models, low-throughput | Cheap, general-purpose |
| **GPU** | Large batches, neural nets | High parallelism, expensive |
| **TPU** | Training/inference of large models | Google's custom ML hardware |
| **Edge chips (NPU)** | On-device inference | Low power, optimised for INT8 |

### Model Compression Techniques

**Quantisation:** Reduce numerical precision of weights.
- FP32 (32-bit float) → INT8 (8-bit integer): 4× memory reduction, ~4× faster on int hardware.
- **Post-training quantisation:** quantise after training (some accuracy loss).
- **Quantisation-aware training (QAT):** simulate quantisation during training for better accuracy.

**Pruning:** Remove weights that are close to zero.
- **Magnitude pruning:** Set $\theta_j = 0$ if $|\theta_j| < \text{threshold}$.
- **Structured pruning:** Remove entire neurons or filters.
- Typically fine-tune the pruned model to recover accuracy.

**Knowledge Distillation:** Train a small **student** model to mimic a large **teacher** model.
$$\mathcal{L}_{\text{distill}} = (1-\alpha)\mathcal{L}_{\text{CE}}(y, \hat{p}_s) + \alpha \mathcal{L}_{\text{KD}}(\hat{p}_t^T, \hat{p}_s^T)$$

Where:
- $\hat{p}_t^T$ = teacher's softened probabilities at temperature $T$
- $\hat{p}_s^T$ = student's softened probabilities at temperature $T$
- $\alpha$ = weight balancing both terms

The "dark knowledge" in the teacher's soft labels (e.g., a dog image being slightly cat-like) transfers to the student.

---

## Inference in Generative Models

For **language models** (GPT-style), inference = **text generation**.

**Autoregressive generation:** Generate one token at a time, feeding each token back as input:
$$p(x_1, \ldots, x_T) = \prod_{t=1}^{T} p(x_t \mid x_1, \ldots, x_{t-1})$$

**Decoding strategies:**

| Strategy | Description | Use case |
|---|---|---|
| **Greedy** | Always pick $\arg\max_k p(x_t = k)$ | Fast, but repetitive/suboptimal |
| **Beam search** | Keep top-$B$ partial sequences | Machine translation |
| **Top-k sampling** | Sample from top $k$ tokens | Creative text |
| **Top-p (nucleus) sampling** | Sample from smallest set of tokens with cumulative prob $\geq p$ | Creative text |
| **Temperature sampling** | Divide logits by $T$ before sampling; $T>1$ = more random | Tuning creativity/diversity |

---

## Connections

- [[Model]] — what is used during inference
- [[Training Data]] — what the model learned from before inference
- [[Features]] — what is fed to the model during inference
- [[Labels]] — what the model predicts (the label is unknown at inference time)
- [[Generalization]] — inference on new data tests generalisation
- [[Machine Learning]] — inference is the "production" phase of ML

---

## One-line Summary

> Inference is the process of feeding new, unseen data through a trained model's forward pass to obtain a prediction — it is the moment the learned function is actually used, and its speed, reliability, and accuracy determine the real-world value of the ML system.
