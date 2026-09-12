# Transformer Architecture

## What is it?

The **Transformer** is a sequence-processing architecture built entirely out of [[Attention Mechanism|self-attention]] and feedforward layers — with **no recurrence at all**. This is the architecture underlying essentially every modern LLM (GPT, BERT, and everything descended from them), and understanding its handful of building blocks is the foundation for understanding LLM internals in [[Applied AI and LLM Systems Index|Applied AI & LLM Systems]].

---

## Why Dropping Recurrence Was the Point

An [[Recurrent Neural Network (RNN)|RNN]] must process a sequence one token at a time, in order — token $t$'s computation depends on token $t-1$'s completed computation, which makes RNNs inherently sequential and hard to parallelize. Self-attention computes relationships between *all* tokens in a sequence simultaneously, in one batched matrix operation, which is dramatically more parallelizable on GPU hardware — the practical reason Transformers could be trained on far more data than RNN-based models ever were, which is a large part of why they enabled the current scale of LLMs at all.

---

## The Building Blocks

**Multi-head self-attention** (see [[Attention Mechanism]]) — each token attends to every other token in the sequence.

**Positional Encoding** — self-attention has no inherent notion of token order (it treats the sequence as an unordered set of positions unless told otherwise), unlike an RNN, which processes tokens in order by construction. Positional encoding adds order information back in explicitly, typically as a fixed sinusoidal pattern added to each token's embedding:

$$PE(pos, 2i) = \sin\left(\frac{pos}{10000^{2i/d}}\right), \quad PE(pos, 2i+1) = \cos\left(\frac{pos}{10000^{2i/d}}\right)$$

Different frequencies at different embedding dimensions let the model distinguish both absolute position and relative distance between tokens. (Modern LLMs often use learned or relative positional schemes instead of this original fixed sinusoidal version, but the *purpose* — injecting order information that self-attention otherwise lacks — is unchanged.)

**Feedforward layers** — after self-attention mixes information across positions, a position-wise feedforward network ([[Multi-Layer Perceptron|MLP]], applied identically and independently to each position) adds per-token non-linear transformation capacity.

**Residual connections and [[Batch and Layer Normalization|LayerNorm]]** — each sub-layer's output is added back to its input (a residual/skip connection) before normalization, which is what makes it practical to stack dozens of Transformer layers without the [[Backpropagation|vanishing gradient]] problem that would otherwise cripple such depth.

$$\text{output} = \text{LayerNorm}(x + \text{Sublayer}(x))$$

---

## Encoder, Decoder, and Encoder-Decoder Variants

**Encoder-only** (e.g. BERT) — every token attends to every other token, including ones after it ("bidirectional" attention). Good for understanding tasks (classification, extracting representations) where the whole input is available at once.

**Decoder-only** (e.g. GPT) — each token can only attend to tokens *before* it ("causal" or "masked" self-attention), matching how text is actually generated one token at a time. This is the architecture behind essentially every modern generative LLM.

**Encoder-decoder** (e.g. the original Transformer, T5) — an encoder processes the full input bidirectionally, a decoder generates output causally while also attending back to the encoder's output (via **cross-attention** — the same Q/K/V mechanism, but Q comes from the decoder while K/V come from the encoder). Suited to sequence-to-sequence tasks like translation, where input and output are genuinely different sequences.

---

## Interview Questions

**Why did the Transformer replace RNN-based architectures for most sequence modeling?** Self-attention processes all positions in parallel in one operation, unlike an RNN's inherently sequential token-by-token processing, which enabled training on far larger datasets by exploiting GPU parallelism — the key practical enabler behind today's LLM scale.

**Why does a Transformer need positional encoding at all?** Self-attention computes relationships between tokens without any inherent notion of their order — it would treat a sentence and any reordering of its words identically without an explicit signal for position, unlike an RNN which processes tokens sequentially by construction and thus has order built in.

**What's the difference between an encoder-only, decoder-only, and encoder-decoder Transformer?** Encoder-only (BERT-style) lets every token attend bidirectionally to every other token, suited to understanding tasks; decoder-only (GPT-style) restricts each token to attending only to earlier tokens (causal masking), matching autoregressive generation; encoder-decoder combines a bidirectional encoder with a causal decoder that cross-attends to the encoder's output, suited to sequence-to-sequence tasks like translation.

**Why are residual connections important in a Transformer?** They let gradients flow directly through the addition rather than only through the (potentially many) stacked sub-layers, which is what makes training very deep Transformer stacks (dozens of layers) practical without the vanishing-gradient problem that depth alone would otherwise cause.

## Connections

- [[Attention Mechanism]] — the core mechanism this architecture is built entirely around
- [[Sequence-to-Sequence Models]] — the encoder-decoder framing this architecture generalizes and replaces the recurrence in
- [[Batch and Layer Normalization]] — LayerNorm is used throughout, for the reasons covered there
- [[BERT Embeddings]] (NLP) — a concrete encoder-only Transformer application
- LLM fundamentals (Applied AI & LLM Systems, module 19) — decoder-only Transformers are the architecture behind every model discussed there

## One-line Summary

> The Transformer replaces recurrence entirely with self-attention (parallelizable across all positions at once) plus positional encoding (to reintroduce order), feedforward layers, and residual+LayerNorm connections that make very deep stacks trainable — the architecture underlying essentially every modern LLM.
