# Batch and Layer Normalization

## What is it?

**Batch Normalization (BatchNorm)** and **Layer Normalization (LayerNorm)** both normalize a layer's activations to have zero mean and unit variance *during training*, then apply a learned scale and shift — stabilizing the distribution of inputs each layer sees, which speeds up and stabilizes training of deep networks. They differ only in *which dimension* they normalize across.

$$\hat{x} = \frac{x-\mu}{\sqrt{\sigma^2+\epsilon}}, \quad y = \gamma\hat{x}+\beta$$

$\gamma,\beta$ are learned parameters (letting the network undo the normalization if that's actually optimal); $\epsilon$ is a small constant preventing division by zero.

---

## Why Normalize Activations At All: Internal Covariate Shift

As training updates earlier layers' weights, the *distribution* of inputs to every later layer keeps shifting — each layer is perpetually chasing a moving target. This slows convergence and makes deep networks sensitive to [[Weight Initialization]] and [[Learning Rate]] choice. Normalizing each layer's inputs to a stable distribution (mean 0, variance 1) removes much of this shifting-target problem, letting higher learning rates be used safely and reducing sensitivity to initialization.

## BatchNorm: Normalize Across the Batch

For each feature, BatchNorm computes $\mu,\sigma^2$ **across the examples in the current mini-batch**. This works well for large batches (statistics are a stable estimate) but has two real weaknesses: it behaves differently at train time (batch statistics) vs. inference time (a running average of batch statistics accumulated during training, since there's no "batch" of one example), and it degrades with small batch sizes, where a batch's mean/variance become noisy estimates.

## LayerNorm: Normalize Across the Features

For each *individual example*, LayerNorm computes $\mu,\sigma^2$ **across that example's own features** (or, for a sequence model, across one token's feature vector), independent of batch size or other examples. This makes it identical at train and inference time (no batch-dependent statistics to reconcile) and unaffected by batch size — which is exactly why **Transformers use LayerNorm, not BatchNorm**: sequence lengths vary, batch sizes during inference are often 1, and per-example independence matters more than it does for CNNs.

| | Normalizes across | Train/inference consistency | Standard use |
|---|---|---|---|
| BatchNorm | The batch, per feature | Different (batch stats vs. running average) | CNNs, standard feedforward nets |
| LayerNorm | Features, per example | Identical | Transformers, RNNs |

---

## Why This Also Helps With Vanishing/Exploding Gradients

By keeping activation magnitudes in a stable, controlled range at every layer, normalization directly reduces the risk that [[Backpropagation]]'s repeated multiplication through many layers explodes or vanishes — it's a complementary fix to [[Weight Initialization]] and [[Activation Functions|ReLU]], addressing the same underlying problem from the training-dynamics side rather than the initialization side.

---

## Interview Questions

**What's the actual difference between BatchNorm and LayerNorm?** BatchNorm normalizes each feature across the examples in a batch; LayerNorm normalizes each example across its own features. That's why LayerNorm behaves identically at train and inference time (no batch-dependent statistics) while BatchNorm needs a running average at inference — and why LayerNorm doesn't degrade with small or variable batch sizes.

**Why do Transformers use LayerNorm instead of BatchNorm?** Sequence lengths vary and inference often runs on a single example (batch size 1), where BatchNorm's per-feature batch statistics become meaningless or unavailable — LayerNorm's per-example normalization doesn't depend on batch composition at all.

**What problem does normalization solve, described at a mechanism level?** It stabilizes the distribution of each layer's inputs across training (rather than each layer chasing a constantly shifting target as earlier layers update), which allows higher learning rates and reduces sensitivity to weight initialization — and, as a side effect, keeps activation magnitudes controlled enough to reduce vanishing/exploding gradients.

## Connections

- [[Backpropagation]] — normalization is one of the standard fixes for vanishing/exploding gradients
- [[Weight Initialization]] — a complementary technique addressing the same underlying instability
- [[Sequence Models Index]] (RNN/LSTM) and Transformer architecture — LayerNorm's standard homes
- [[Dropout]] — the other standard "training deep networks" technique, addressing overfitting rather than optimization stability

## One-line Summary

> Both BatchNorm and LayerNorm normalize activations to a stable distribution and then rescale with learned parameters — BatchNorm across the batch (fast, but batch-size-dependent and train/inference-inconsistent), LayerNorm across each example's own features (why Transformers use it instead).
