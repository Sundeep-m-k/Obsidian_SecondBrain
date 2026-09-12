# Dropout

## What is it?

**Dropout** is a regularization technique specific to neural networks: during training, each unit is randomly "dropped" (its output set to zero) with probability $p$ on every forward pass, forcing the network to not rely too heavily on any single unit or specific co-adapted group of units.

$$a^{(l)}_i \leftarrow \begin{cases} 0 & \text{with probability } p \\ a^{(l)}_i / (1-p) & \text{with probability } 1-p\end{cases}$$

The $1/(1-p)$ scaling ("inverted dropout," the standard modern implementation) keeps the expected magnitude of the layer's output the same whether or not dropout is active, so nothing needs to change at inference time.

---

## Why This Prevents Overfitting

Without dropout, units can develop **co-adaptation** — unit A only produces something useful in the specific presence of units B and C, a fragile, overfit pattern that captures training-set-specific quirks rather than a generally useful feature. Randomly removing different units on every forward pass prevents any unit from being able to rely on any specific other unit always being present, forcing each to learn features that are useful more independently and robustly. This is the same [[Bias Variance Tradeoff]] every other [[Regularization]] technique manages, applied at the level of individual units rather than a weight-magnitude penalty like [[L2 Regularization]].

## Dropout as Implicit Ensembling

Because a different random subset of units is active on every forward pass, training with dropout is loosely equivalent to training an enormous ensemble of thinner sub-networks that share weights, and averaging their predictions — directly analogous to [[Bagging]]'s variance-reduction mechanism, but achieved by randomly zeroing units within one network rather than training many separate models on bootstrap samples.

---

## Train vs. Inference

Dropout is active **only during training** — at inference time, all units are used (no randomness), which is why inverted dropout's $1/(1-p)$ scaling during training matters: it makes the *expected* activation magnitude match what inference will actually see, so no rescaling is needed at inference time. Forgetting to disable dropout at inference (e.g. leaving a model in "training mode") silently introduces randomness into what should be a deterministic prediction — a common, easy-to-miss bug.

```python
import torch.nn as nn
model = nn.Sequential(nn.Linear(128, 64), nn.ReLU(), nn.Dropout(p=0.5), nn.Linear(64, 10))
model.train()  # dropout active
model.eval()   # dropout disabled — required before inference
```

---

## Choosing the Dropout Rate

Typical values: $p=0.5$ for larger fully-connected layers, lower ($p=0.1$–$0.2$) for convolutional layers or smaller networks, since convolutional layers already share parameters heavily (via their filters) and have less redundancy to spare. Too high a rate under-utilizes network capacity (effectively [[Underfitting]]); too low provides little regularization benefit.

---

## Interview Questions

**Why does randomly zeroing units during training reduce overfitting?** It prevents units from co-adapting — relying on specific other units always being present — forcing each unit to learn features useful somewhat independently, which is a form of implicit ensembling over many thinner shared-weight sub-networks.

**Why is dropout scaled by $1/(1-p)$ during training instead of doing something at inference time?** So the expected activation magnitude during training matches what inference (with no units dropped) will produce — this "inverted dropout" convention means inference requires no rescaling at all, just running the network normally with dropout disabled.

**What's the relationship between dropout and bagging?** Both reduce variance by combining many slightly different models' predictions — bagging trains separate models on bootstrap-resampled data, while dropout implicitly trains an enormous number of weight-sharing sub-networks by randomly deactivating different units on each forward pass.

## Connections

- [[Regularization]], [[Bias Variance Tradeoff]] — the general problem dropout addresses, at the unit level instead of the weight-magnitude level
- [[Bagging]] — the closest classical-ML analog, both reducing variance via implicit/explicit ensembling
- [[Batch and Layer Normalization]] — the other standard "training deep networks" technique, addressing optimization stability rather than overfitting
- [[Overfitting]] — what dropout is specifically fighting

## One-line Summary

> Dropout randomly zeroes units during training to prevent co-adaptation, acting as an implicit ensemble over many weight-sharing sub-networks — active only during training, disabled at inference, and the neural-network-specific analog of what L2 regularization and bagging do elsewhere.
