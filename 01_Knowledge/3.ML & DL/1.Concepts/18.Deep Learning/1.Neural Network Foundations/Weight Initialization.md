# Weight Initialization

## What is it?

**Weight initialization** is how a network's weights are set before training begins. It sounds trivial but is a genuine failure point: a bad initialization can make [[Backpropagation]]'s vanishing/exploding gradient problem severe from the very first step, before any training has even had a chance to correct it.

---

## Why Zero (or Constant) Initialization Fails

If every weight in a layer starts at the same value, every unit in that layer computes the identical function of the input, receives the identical gradient during backpropagation, and updates identically forever — the **symmetry-breaking problem**. A layer with $n$ identically-initialized units effectively behaves like a layer with 1 unit, no matter how large $n$ is. Weights must be initialized *randomly* to break this symmetry, letting different units learn different features.

## Why "Just Use Small Random Numbers" Isn't Enough

Naively drawing weights from, say, $\mathcal{N}(0, 1)$ regardless of layer size causes the variance of activations to grow or shrink systematically as data passes through more layers — with large layers, weighted sums can easily explode; with many layers, they can easily vanish. Both directly worsen the vanishing/exploding gradient problem discussed in [[Backpropagation]] before training even starts.

## Xavier/Glorot Initialization

Designed to keep the variance of activations (and gradients) roughly constant across layers, for **sigmoid/tanh** activations:

$$W \sim \mathcal{U}\left(-\sqrt{\frac{6}{n_{in}+n_{out}}}, \ \sqrt{\frac{6}{n_{in}+n_{out}}}\right)$$

$n_{in}$, $n_{out}$ are the layer's fan-in and fan-out (number of input and output units). The scaling is derived by requiring the variance of a layer's output to equal the variance of its input, given the specific way sigmoid/tanh transform their input.

## He Initialization

The same goal, re-derived for **ReLU**, which zeroes out roughly half its inputs (everything negative) — a factor Xavier's derivation doesn't account for:

$$W \sim \mathcal{N}\left(0, \ \frac{2}{n_{in}}\right)$$

The extra factor of 2 compensates for ReLU killing half the signal on average. **He initialization is the standard default for ReLU-based networks**; using Xavier with ReLU (or vice versa, He with sigmoid/tanh) systematically mismatches the variance the derivation assumes, reintroducing a milder version of the vanishing/exploding problem both were designed to prevent.

```python
import torch.nn as nn
layer = nn.Linear(128, 64)
nn.init.kaiming_normal_(layer.weight, nonlinearity="relu")  # He initialization
nn.init.xavier_normal_(layer.weight)                        # Xavier/Glorot initialization
```

---

## Interview Questions

**Why can't all weights be initialized to zero?** Every unit in a layer would compute the same function, receive the same gradient, and update identically forever — the network never breaks the symmetry between units, so a layer with any number of units behaves like it has only one.

**What problem does He initialization solve that plain random initialization doesn't?** It scales the initial weight variance specifically to account for ReLU zeroing out roughly half its inputs, keeping the variance of activations (and gradients) stable across many layers instead of systematically shrinking or exploding as depth increases — a problem plain unscaled random initialization doesn't address at all.

**When would you use Xavier vs. He initialization?** Xavier/Glorot for sigmoid/tanh activations (its derivation assumes a symmetric, non-zeroing activation), He for ReLU and its variants (its derivation explicitly accounts for ReLU zeroing half its inputs) — mismatching the two reintroduces the variance instability both were built to prevent.

## Connections

- [[Backpropagation]] — poor initialization worsens vanishing/exploding gradients from the very first step
- [[Activation Functions]] — the specific activation determines which initialization scheme is appropriate
- [[Batch and Layer Normalization]] — a complementary technique that keeps activation variance stable *during* training, not just at initialization

## One-line Summary

> Weights must be initialized randomly (to break symmetry between units) and at the right scale for the activation function in use — He initialization for ReLU, Xavier/Glorot for sigmoid/tanh — or the vanishing/exploding gradient problem starts before training even begins.
