---
tags: [nlp, index, moc, sequence-models, rnn, lstm]
links: "[[NLP Index]] [[03_Text_to_Numbers/Index]]"
---

# 04 — Sequence Models (Index)

> **Position in vault**: `01_Knowledge/4.NLP/04_Sequence_Models/`
> **Purpose**: How to process text as a *sequence* rather than a bag of independent tokens — the architectures that came before, and directly motivated, attention and Transformers.
> **Prerequisite**: [[03_Text_to_Numbers/Index]] (a sequence model consumes the embeddings that section produces), and [[Gradient Descent]] / [[Loss Function]] from `3.ML & DL`.
> **Scope note**: This module was originally planned as "Learning Core: Probability, Classical ML, Neural Fundamentals." That content now lives in [[6.Statistics & Experimental Design]] and [[5.Mathematics]] instead, to avoid duplicating it — see [[NLP Index]] for the full explanation. This module covers what's actually NLP/DL-specific and was still missing.

---

## Why Sequence Models Matter

Everything in `03_Text_to_Numbers` produces a vector *per token*, independent of position and order. But "dog bites man" and "man bites dog" have the same bag-of-words representation and opposite meanings. Sequence models are the first architectures that process tokens **in order**, carrying information forward from earlier tokens to later ones — the mechanism that makes word order, long-range dependencies, and generation (predicting the next token given everything before it) possible at all.

## Section Map

| Note | Key Concepts | Connects To |
|---|---|---|
| [[Recurrent Neural Network (RNN)]] | Hidden state recurrence, unrolling through time, BPTT | Vanishing gradients, LSTM |
| [[Vanishing and Exploding Gradients in RNNs]] | Why long sequences break plain RNNs, gradient norm through time | LSTM/GRU's gating fix, [[Gradient Descent]] |
| [[Long Short-Term Memory (LSTM)]] | Cell state, forget/input/output gates | GRU (simplified), attention (the eventual replacement) |
| [[Gated Recurrent Unit (GRU)]] | Update/reset gates, fewer parameters than LSTM | LSTM comparison |
| [[Sequence-to-Sequence Models]] | Encoder-decoder, the fixed-context-vector bottleneck | Directly motivates Attention (module 05) |

**Key insight**: Every note in this module builds toward one destination — understanding *why* attention and Transformers were invented. The seq2seq bottleneck (a whole sentence compressed into one fixed-size vector) is the specific problem attention solves; understanding the problem first makes the solution in module 05 obvious rather than magic.

---

## Key Cross-Links to `3.ML & DL/` and `6.Statistics & Experimental Design/`

| Sequence Model Concept | Links to |
|---|---|
| Backpropagation through time | [[Gradient Descent]] — same algorithm, applied across a time-unrolled computation graph |
| Vanishing gradients | [[Overfitting]]'s sibling failure mode — a training pathology, not a generalization one |
| Gating mechanisms (LSTM/GRU) | [[Sigmoid Function]] — gates are literally sigmoid outputs used as soft on/off switches |
| Seq2seq training | [[Loss Function]] — trained with the same cross-entropy loss as any classifier, just applied per output token |
| Choosing sequence length / truncation | [[Grouped Train-Test Split]]'s spirit — truncation choices can leak future tokens into a "past-only" prediction if done carelessly |

## Common Exam / Interview Questions

1. Why can't a plain feedforward network handle variable-length sequences the way an RNN can?
2. Derive why gradients vanish or explode over long sequences in a plain RNN — what's happening mathematically as you backpropagate through many time steps?
3. What specific mechanism in an LSTM prevents the vanishing gradient problem that plagues plain RNNs?
4. What's the difference between an LSTM's forget gate and its input gate?
5. Why is GRU sometimes preferred over LSTM in practice, and what's the tradeoff?
6. What is the seq2seq "bottleneck problem," and how does it motivate attention?

## Reading Order (Recommended)

Recurrent Neural Network (RNN) → Vanishing and Exploding Gradients in RNNs → Long Short-Term Memory (LSTM) → Gated Recurrent Unit (GRU) → Sequence-to-Sequence Models

---

*Next section: 05 — Attention & Transformers (not yet built)*
*Previous section: [[03_Text_to_Numbers/Index]]*
