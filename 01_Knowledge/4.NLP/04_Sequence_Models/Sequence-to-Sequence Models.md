---
tags: [nlp, sequence-models, seq2seq, encoder-decoder]
---

# Sequence-to-Sequence Models

## What is it?

A **seq2seq** model maps a variable-length input sequence to a variable-length output sequence — the general architecture behind machine translation, summarization, and any task where input and output lengths aren't fixed or equal. It consists of two RNNs ([[Recurrent Neural Network (RNN)]], typically [[Long Short-Term Memory (LSTM)]] or [[Gated Recurrent Unit (GRU)]] in practice): an **encoder** and a **decoder**.

## Architecture

$$\underbrace{x_1, x_2, \ldots, x_T}_{\text{source sequence}} \xrightarrow{\text{encoder RNN}} \underbrace{c}_{\text{context vector}} \xrightarrow{\text{decoder RNN}} \underbrace{y_1, y_2, \ldots, y_{T'}}_{\text{target sequence}}$$

**Encoder**: runs an RNN over the entire source sequence, producing a final hidden state $h_T$ that is treated as a fixed-size **context vector** $c = h_T$ — a compressed summary of the whole input.

**Decoder**: a separate RNN, initialized with $h_0^{dec} = c$, that generates the output one token at a time, conditioning each step on the previous output token and its own hidden state:

$$P(y_t \mid y_{<t}, x) = \text{softmax}(W_{hy} h_t^{dec} + b_y), \qquad h_t^{dec} = f(h_{t-1}^{dec}, y_{t-1}, c)$$

**Training**: uses **teacher forcing** — the decoder is fed the *true* previous target token $y_{t-1}$ during training (not its own prediction), trained with token-level cross-entropy [[Loss Function]] summed across the output sequence.

**Inference**: no ground truth is available, so the decoder feeds its own previous prediction back in — either greedily (argmax at each step) or via **beam search** (tracking the top-$k$ partial sequences by cumulative probability rather than committing greedily).

## Worked Example

Translating "the cat sat" → "le chat s'est assis": the encoder processes "the," "cat," "sat" sequentially, producing $c$, a single vector meant to capture "there is a cat, past tense, sitting action." The decoder then generates "le" (conditioned on $c$ alone), then "chat" (conditioned on $c$ and "le"), then "s'est," then "assis," each step narrowing down the next French token given everything generated so far plus the compressed source summary.

## The Bottleneck Problem

The entire source sentence — however long — is compressed into **one fixed-size vector** $c$. For short sentences this is fine; for long sentences (30+ words), $c$ has to represent everything simultaneously, and early-sentence information tends to get overwritten by later encoder steps (the same signal-decay issue as [[Vanishing and Exploding Gradients in RNNs]], now affecting *content* retention, not just gradient flow). Empirically, seq2seq translation quality degrades sharply as source sentence length increases, precisely because of this bottleneck.

**This is the single most important limitation to understand in this module**: it is the exact problem that attention was invented to solve. Attention lets the decoder look back at *all* encoder hidden states $h_1, \ldots, h_T$ at every decoding step (with a learned, dynamic weighting), rather than being forced through one fixed $c$ — turning a fixed-size bottleneck into a lookup over the full source sequence.

## Why It Matters

Seq2seq is the architectural template — encode, then decode conditioned on that encoding — that every subsequent generation architecture still follows, up through modern Transformer-based encoder-decoder models (T5, BART) and decoder-only LLMs (which fold "encoding" and "decoding" into one autoregressive stream). Understanding *why* the fixed context vector fails is the cleanest way to understand *why* attention exists.

## Advantages and Limitations

**Advantages**: general-purpose framework for any variable-length-in, variable-length-out task; conceptually simple, cleanly separates "understand the input" from "generate the output."

**Limitations**: fixed-size bottleneck degrades on long sequences (the core motivation for attention); sequential decoding is slow at inference (one token at a time, can't parallelize generation); exposure bias — teacher forcing during training means the model never practices recovering from its own mistakes, which it must do at inference.

## Common Mistakes

- Believing $c$ can hold arbitrary amounts of information just because it's a "summary" — it's a fixed-dimensional vector, so there's a hard information-theoretic ceiling on how much of a long sentence it can preserve
- Confusing teacher forcing (training-time convenience, feeding ground truth) with how the model actually generates at inference (autoregressive, feeding its own predictions) — this train/inference mismatch is exposure bias
- Assuming greedy decoding and beam search produce the same output — beam search explores multiple hypotheses and typically produces higher-quality (though slower, less diverse) output than greedy argmax-at-each-step

## Interview / discussion questions

- Draw the encoder-decoder architecture and explain what the context vector $c$ represents.
- What is teacher forcing, and why does it create a train/inference mismatch (exposure bias)?
- Explain the seq2seq bottleneck problem precisely — why does translation quality degrade with source sentence length?
- How does attention directly resolve the bottleneck problem described here?

## Prerequisites

[[Recurrent Neural Network (RNN)]], [[Long Short-Term Memory (LSTM)]], [[Loss Function]]

## Related concepts

[[Gated Recurrent Unit (GRU)]], [[Vanishing and Exploding Gradients in RNNs]]

## Tags

#category/nlp #topic/sequence-models #math/probability

## One-line summary

> Seq2seq encodes a variable-length source into one fixed-size context vector $c$ and decodes a variable-length target from it token by token — a clean general framework whose fixed-$c$ bottleneck degrades on long sequences and is the exact problem attention (module 05) was invented to solve.
