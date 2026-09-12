---
tags: [nlp, sequence-models, rnn, optimization]
---

# Vanishing and Exploding Gradients in RNNs

## What is it?

Training a [[Recurrent Neural Network (RNN)]] via backpropagation through time requires computing $\frac{\partial \mathcal{L}}{\partial h_t}$ for early time steps $t$, which involves a **product of Jacobians** across every intervening time step. That product can shrink toward zero (**vanishing**) or grow unboundedly (**exploding**) as the sequence gets longer.

## The Math

For hidden state recurrence $h_t = \tanh(W_{hh}h_{t-1} + \ldots)$, the gradient of the loss at final step $T$ with respect to an early hidden state $h_k$ is:

$$\frac{\partial \mathcal{L}}{\partial h_k} = \frac{\partial \mathcal{L}}{\partial h_T} \prod_{t=k+1}^{T} \frac{\partial h_t}{\partial h_{t-1}} = \frac{\partial \mathcal{L}}{\partial h_T} \prod_{t=k+1}^{T} \text{diag}(\tanh'(z_t)) \, W_{hh}$$

Each factor in the product contributes roughly a multiplication by $\|W_{hh}\|$ (scaled by $\tanh'$, which is $\leq 1$ and often much less than 1 away from $z=0$). Over $T-k$ time steps:

$$\left\|\frac{\partial h_T}{\partial h_k}\right\| \approx \|W_{hh}\|^{T-k} \cdot (\text{something} \leq 1)^{T-k}$$

If the dominant eigenvalue of $W_{hh}$ is $< 1$: the product **vanishes** exponentially in sequence length. If it's $> 1$: the product **explodes** exponentially.

## Worked Example

Suppose $\|W_{hh}\| \approx 0.9$ (a plausible trained value) and $\tanh'(z_t) \approx 0.5$ on average (moderate activations, not saturated). Effective per-step factor $\approx 0.9 \times 0.5 = 0.45$. Over 20 time steps: $0.45^{20} \approx 4 \times 10^{-7}$ — the gradient signal reaching step 1 from a loss computed at step 20 is roughly seven orders of magnitude smaller than the gradient at step 20 itself. In practice, this means the network effectively cannot learn dependencies spanning more than a handful of time steps — a real long-range dependency (e.g. subject-verb agreement across a 15-word clause) gets essentially zero training signal.

Conversely, with $\|W_{hh}\| \approx 1.5$: $1.5^{20} \approx 3{,}325$ — gradients blow up, causing unstable, wildly oscillating updates (visible as sudden loss spikes during training).

## Practical Fixes

| Fix | Addresses | Mechanism |
|---|---|---|
| **Gradient clipping** | Exploding only | Rescale the gradient vector if its norm exceeds a threshold — cheap, doesn't fix vanishing |
| **LSTM / GRU gating** | Vanishing (mainly) | Replace the multiplicative recurrence with an additive, gated cell-state update — see [[Long Short-Term Memory (LSTM)]] |
| **Careful initialization** (e.g. orthogonal $W_{hh}$) | Both, partially | Keeps the initial Jacobian product closer to norm-preserving |
| **Shorter sequences / truncated BPTT** | Vanishing (band-aid) | Limits how many steps gradients must travel, at the cost of losing long-range signal entirely |

Gradient clipping is a purely training-stability fix (caps the damage from explosion) — it does **not** give the network the ability to actually learn long-range dependencies the way gated architectures do.

## Why It Matters

This is the single biggest reason plain RNNs were replaced by LSTMs and GRUs for anything beyond short sequences, and — one level further out — a major motivation for attention mechanisms, which connect any two positions directly (constant-length gradient path) rather than through a chain of $T-k$ multiplicative steps.

## Common Mistakes

- Only applying gradient clipping and assuming that "solves" the RNN's long-range-dependency problem — it only prevents numerical blow-up, it doesn't restore vanished gradient signal
- Not checking $\|W_{hh}\|$'s effective spectral behavior when debugging why an RNN isn't learning long-range patterns
- Assuming this is unique to RNNs — any sufficiently deep network without residual/gating structure can suffer analogous vanishing gradients ([[Overfitting]] is a different failure mode entirely — this is a training pathology, not a generalization one)

## Interview / discussion questions

- Derive why the gradient $\frac{\partial h_T}{\partial h_k}$ involves a product of $T-k$ Jacobian terms, and explain what property of that product causes vanishing vs. exploding.
- Why does gradient clipping fix exploding gradients but not vanishing ones?
- How does LSTM's cell-state update avoid the repeated-multiplication structure that causes vanishing gradients?

## Prerequisites

[[Recurrent Neural Network (RNN)]], [[Gradient Descent]], chain rule (calculus)

## Related concepts

[[Long Short-Term Memory (LSTM)]], [[Gated Recurrent Unit (GRU)]]

## Tags

#category/nlp #topic/sequence-models #math/calculus

## One-line summary

> Backpropagating through $T-k$ time steps multiplies $T-k$ Jacobian terms together, so the gradient shrinks exponentially if $\|W_{hh}\|\tanh'(z) < 1$ (vanishing) or blows up if $>1$ (exploding) — gradient clipping patches the exploding case, but only gated architectures like LSTM actually fix vanishing.
