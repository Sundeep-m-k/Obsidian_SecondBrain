---
tags: [nlp, sequence-models, rnn, neural-networks]
---

# Recurrent Neural Network (RNN)

## What is it?

A **Recurrent Neural Network** processes a sequence $x_1, x_2, \ldots, x_T$ one element at a time, maintaining a **hidden state** $h_t$ that carries information forward from all previous elements.

$$h_t = \tanh(W_{hh} h_{t-1} + W_{xh} x_t + b_h), \qquad \hat y_t = \text{softmax}(W_{hy} h_t + b_y)$$

The same weights $W_{hh}, W_{xh}, W_{hy}$ are reused at every time step (**parameter sharing across time**) — the network doesn't need separate weights for "the 5th word" vs. "the 50th word."

## Unrolling Through Time

Conceptually, an RNN is a feedforward network if you "unroll" it across time steps, with $h_{t-1} \to h_t$ acting like a normal layer-to-layer connection, except the *same* weight matrices are reused at every step:

$$h_0 \xrightarrow{W} h_1 \xrightarrow{W} h_2 \xrightarrow{W} \cdots \xrightarrow{W} h_T$$

This unrolled view is what makes training possible via **backpropagation through time (BPTT)**: gradients flow backward through the unrolled graph exactly like a normal deep network, just with weight-sharing constraints (gradients from every time step accumulate into the *same* shared $W_{hh}$).

## Worked Example: One Forward Step

$h_{t-1} = [0.2, -0.1]$ (2-dim hidden state), $x_t$ embeds to $[0.5, 0.3, -0.2]$ (3-dim). With $W_{hh} \in \mathbb{R}^{2\times2}$, $W_{xh} \in \mathbb{R}^{2\times3}$: the pre-activation $z_t = W_{hh}h_{t-1} + W_{xh}x_t + b_h$ combines a weighted view of "everything remembered so far" ($W_{hh}h_{t-1}$) with "what's new right now" ($W_{xh}x_t$), then squashes through $\tanh$ to produce $h_t \in (-1,1)^2$ — the updated memory, ready to be combined with $x_{t+1}$ at the next step.

## Why It Matters

RNNs were the first architecture to genuinely model sequence order and variable length — a feedforward network needs a fixed input size and has no notion of "before" and "after." Every later sequence architecture (LSTM, GRU, and even attention/Transformers) is a response to a specific limitation of the plain RNN described here.

## Limitations

- **Sequential computation**: $h_t$ depends on $h_{t-1}$, so processing can't be parallelized across time steps within one sequence — a major speed disadvantage vs. Transformers (module 05)
- **[[Vanishing and Exploding Gradients in RNNs]]**: gradients through many time steps can shrink or blow up exponentially, making long-range dependencies hard to learn
- **Fixed-size hidden state bottleneck**: all information about the sequence so far is compressed into one vector $h_t$, regardless of sequence length

## Common Mistakes

- Assuming an RNN can be parallelized across time the way a Transformer can — it fundamentally can't, due to the $h_{t-1} \to h_t$ dependency
- Using a plain RNN for long sequences without realizing [[Vanishing and Exploding Gradients in RNNs]] will likely prevent it from learning long-range dependencies — this is exactly why [[Long Short-Term Memory (LSTM)]] exists
- Forgetting that weights are *shared* across time steps — the gradient w.r.t. $W_{hh}$ is a sum over contributions from every time step, not computed independently per step

## Interview / discussion questions

- Write out the RNN recurrence and explain what "parameter sharing across time" means and why it matters.
- Why is BPTT just standard backpropagation applied to the unrolled computational graph?
- What's the fundamental sequential bottleneck that limits RNN training speed compared to Transformers?

## Prerequisites

[[Gradient Descent]], [[Sigmoid Function]], basic neural network layers

## Related concepts

[[Vanishing and Exploding Gradients in RNNs]], [[Long Short-Term Memory (LSTM)]], [[Gated Recurrent Unit (GRU)]], [[Sequence-to-Sequence Models]]

## Tags

#category/nlp #topic/sequence-models #math/calculus

## One-line summary

> An RNN processes a sequence one step at a time with a shared-weight recurrence $h_t = \tanh(W_{hh}h_{t-1}+W_{xh}x_t+b_h)$, giving it a notion of order and memory that feedforward networks lack — at the cost of sequential (unparallelizable) computation and vulnerability to vanishing/exploding gradients over long sequences.
