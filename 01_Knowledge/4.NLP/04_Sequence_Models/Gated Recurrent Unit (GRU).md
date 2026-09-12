---
tags: [nlp, sequence-models, gru, neural-networks]
---

# Gated Recurrent Unit (GRU)

## What is it?

A **GRU** is a simplified gated recurrent architecture (Cho et al., 2014) that achieves most of what [[Long Short-Term Memory (LSTM)]] achieves — mitigating vanishing gradients via gating — with **two gates instead of four weight matrices**, and no separate cell state.

## The Math

$$\begin{aligned}
z_t &= \sigma(W_z [h_{t-1}, x_t] + b_z) &&\text{update gate — how much of the old state to keep}\\
r_t &= \sigma(W_r [h_{t-1}, x_t] + b_r) &&\text{reset gate — how much of the old state to use when proposing new content}\\
\tilde h_t &= \tanh(W_h [r_t \odot h_{t-1}, x_t] + b_h) &&\text{candidate hidden state}\\
h_t &= (1-z_t) \odot h_{t-1} + z_t \odot \tilde h_t &&\text{final update — linear interpolation}
\end{aligned}$$

## Key Structural Differences from LSTM

| | LSTM | GRU |
|---|---|---|
| **States carried** | Cell state $C_t$ + hidden state $h_t$ (two) | Hidden state $h_t$ only (one) |
| **Gates** | Forget, input, output (three) | Update, reset (two) |
| **Weight matrices** | 4 sets ($W_f, W_i, W_C, W_o$) | 3 sets ($W_z, W_r, W_h$) |
| **Update rule** | Additive cell-state update, separately gated output | Direct linear interpolation between old and candidate hidden state |
| **Parameters** | More (~4x a plain RNN cell) | Fewer (~3x a plain RNN cell) |

The GRU's update rule $h_t = (1-z_t)\odot h_{t-1} + z_t \odot \tilde h_t$ is a clean **convex combination**: $z_t=0$ means "ignore the candidate entirely, keep the old state" (protects long-range memory exactly like LSTM's $f_t\approx1, i_t\approx0$ combination), and $z_t=1$ means "fully overwrite with the new candidate."

## Why It Matters

GRU demonstrated that LSTM's full complexity (separate cell state, three independent gates) isn't strictly necessary to get most of the vanishing-gradient benefit — a useful lesson in architecture design: more gates isn't automatically better, and the *presence* of a gated additive/interpolated update path is the load-bearing idea, not the specific gate count.

## Advantages and Limitations

**Advantages**: fewer parameters → faster to train, less prone to overfitting on smaller datasets, often near-identical performance to LSTM in practice.

**Limitations**: on some tasks with very long or very complex long-range dependencies, LSTM's extra expressiveness (separate cell/hidden state, three independent gates) can still edge out GRU — there's no universal winner, and choosing between them is often empirical.

## Common Mistakes

- Assuming GRU is strictly "LSTM but worse" or "LSTM but better" — in practice they're comparable, and the right choice is dataset/task-dependent (start with GRU for speed, try LSTM if it underperforms)
- Forgetting that GRU has no separate cell state — everything flows through the single $h_t$, unlike LSTM's $C_t$/$h_t$ split
- Missing that the reset gate $r_t$ and update gate $z_t$ serve different purposes: $r_t$ controls how much old state feeds into the *candidate computation*, while $z_t$ controls the *final blend* between old state and candidate

## Interview / discussion questions

- Write the GRU update equations and compare them directly to LSTM's — what's structurally missing?
- Why is $h_t = (1-z_t)\odot h_{t-1} + z_t\odot\tilde h_t$ a "convex combination," and why does that help gradient flow the same way LSTM's forget gate does?
- When would you choose GRU over LSTM in practice, and why?

## Prerequisites

[[Recurrent Neural Network (RNN)]], [[Long Short-Term Memory (LSTM)]]

## Related concepts

[[Long Short-Term Memory (LSTM)]], [[Sequence-to-Sequence Models]]

## Tags

#category/nlp #topic/sequence-models #math/calculus

## One-line summary

> GRU simplifies LSTM into two gates and a single hidden state, updating via a convex combination $h_t=(1-z_t)h_{t-1}+z_t\tilde h_t$ that provides a comparable gradient-preserving path with fewer parameters — a leaner alternative that's often just as effective in practice.
