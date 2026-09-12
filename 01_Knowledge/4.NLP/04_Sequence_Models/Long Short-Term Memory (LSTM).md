---
tags: [nlp, sequence-models, lstm, neural-networks]
---

# Long Short-Term Memory (LSTM)

## What is it?

An **LSTM** is a recurrent architecture that replaces the plain RNN's single multiplicative hidden state with a **cell state** $C_t$ that information flows through mostly *additively*, controlled by three learned **gates** — sigmoid-activated vectors in $(0,1)$ that act as soft on/off switches. This additive path is precisely what fixes [[Vanishing and Exploding Gradients in RNNs]].

## The Math

At each step, given input $x_t$ and previous hidden/cell state $h_{t-1}, C_{t-1}$:

$$\begin{aligned}
f_t &= \sigma(W_f [h_{t-1}, x_t] + b_f) &&\text{forget gate — what to discard from } C_{t-1}\\
i_t &= \sigma(W_i [h_{t-1}, x_t] + b_i) &&\text{input gate — what to write into } C_t\\
\tilde C_t &= \tanh(W_C [h_{t-1}, x_t] + b_C) &&\text{candidate values to write}\\
C_t &= f_t \odot C_{t-1} + i_t \odot \tilde C_t &&\text{cell state update (additive!)}\\
o_t &= \sigma(W_o [h_{t-1}, x_t] + b_o) &&\text{output gate — what to expose}\\
h_t &= o_t \odot \tanh(C_t) &&\text{hidden state (visible output)}
\end{aligned}$$

where $\odot$ is elementwise multiplication and $[h_{t-1}, x_t]$ denotes concatenation.

## Why the Additive Update Fixes Vanishing Gradients

Compare the two recurrences:

- **Plain RNN**: $h_t = \tanh(W_{hh}h_{t-1} + \ldots)$ — every step **multiplies** by $W_{hh}$, so the gradient path through $T$ steps involves a product of $T$ matrices → exponential shrink/blowup.
- **LSTM cell state**: $C_t = f_t \odot C_{t-1} + i_t \odot \tilde C_t$ — the $C_{t-1}$ term is **added**, scaled only elementwise by $f_t$ (not multiplied by a shared weight matrix repeatedly). If the network learns $f_t \approx 1$ for a time step where nothing should be forgotten, gradient flows through that step almost unchanged: $\frac{\partial C_t}{\partial C_{t-1}} = f_t$, which can be kept close to 1 by the forget gate itself, rather than being at the mercy of a fixed $\|W_{hh}\|$.

This is often called the **constant error carousel**: the cell state provides a gradient highway that doesn't require repeated matrix multiplication to preserve signal over long distances.

## Worked Example: Forget Gate in Action

Suppose the model is tracking subject-verb agreement: "The **keys** to the cabinet ... **are** on the table." At the token "keys," the input gate writes the (plural) subject into $C_t$. At every intervening token ("to," "the," "cabinet"), the model can learn $f_t \approx 1$ (keep remembering "plural subject") and $i_t \approx 0$ (nothing new worth writing over it) — so the plural information survives essentially unchanged in $C_t$ across several irrelevant tokens, until the verb "are" needs it. A plain RNN's single multiplicative hidden state has no mechanism to selectively "protect" that one fact from decay.

## Why It Matters

LSTM was the dominant sequence architecture in NLP for roughly a decade (machine translation, speech recognition, tagging) before attention-based Transformers replaced it starting around 2017. Understanding gating is also a direct prerequisite for understanding attention: attention generalizes the idea of "learn what to keep and what to ignore" from a fixed, sequential, gate-based mechanism into a fully dynamic, all-pairs mechanism.

## Advantages and Limitations

**Advantages**: handles much longer dependencies than plain RNNs; the gating mechanism is learned, not hand-designed, so it adapts per-task.

**Limitations**: still sequential (can't parallelize across time within a sequence, unlike Transformers); more parameters and compute per step than a plain RNN (4 weight matrices vs. 1); still degrades on very long sequences (hundreds+ of tokens) compared to attention, which has a constant-length path between any two positions regardless of distance.

## Common Mistakes

- Thinking the forget gate "erasing" information is a bug — it's the entire point: selective forgetting is what lets the network hold onto only what matters
- Confusing the cell state $C_t$ (long-term memory, mostly additive) with the hidden state $h_t$ (short-term/output, a gated, squashed view of $C_t$) — they play different roles
- Assuming LSTMs fully solve vanishing gradients — they mitigate it substantially, not eliminate it; very long sequences (1000+ tokens) still degrade

## Interview / discussion questions

- Walk through all three gates and explain what each one controls.
- Why is the cell-state update additive rather than multiplicative, and why does that matter for gradient flow?
- What is the "constant error carousel," and how does it relate to $f_t \approx 1$?
- How does LSTM's gating mechanism conceptually foreshadow attention?

## Prerequisites

[[Recurrent Neural Network (RNN)]], [[Vanishing and Exploding Gradients in RNNs]], [[Sigmoid Function]]

## Related concepts

[[Gated Recurrent Unit (GRU)]], [[Sequence-to-Sequence Models]]

## Tags

#category/nlp #topic/sequence-models #math/calculus

## One-line summary

> LSTM replaces the RNN's purely multiplicative recurrence with an additive, gated cell state $C_t = f_t \odot C_{t-1} + i_t \odot \tilde C_t$, letting the network learn what to keep and what to discard at each step — the additive path is what prevents gradients from vanishing over long sequences.
