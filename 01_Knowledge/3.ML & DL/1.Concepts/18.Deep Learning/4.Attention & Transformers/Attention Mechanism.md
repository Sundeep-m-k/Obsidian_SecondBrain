# Attention Mechanism

## What is it?

**Attention** lets a model dynamically decide, for each output it's producing, *which parts of the input to focus on* — rather than compressing the entire input into one fixed-size vector and hoping nothing important got lost. It was invented specifically to solve the [[Sequence-to-Sequence Models|seq2seq bottleneck]]: a single fixed-size context vector is a hard information bottleneck for long sequences, and attention removes that bottleneck by letting the decoder look back at *every* encoder state, weighted by relevance, at every decoding step.

---

## The Core Mechanism: Query, Key, Value

Every attention computation reduces to three learned projections of the input:

- **Query (Q)** — "what am I looking for right now?"
- **Key (K)** — "what does each position offer, as a label to match against?"
- **Value (V)** — "what does each position actually contain, to be retrieved if it matches?"

$$\text{Attention}(Q,K,V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right)V$$

$QK^\top$ computes a similarity score between the query and every key (a dot product — higher when the query and key vectors point in similar directions). [[Sigmoid Function|Softmax]] turns these raw scores into a probability distribution over positions (the **attention weights**), and the output is a weighted average of the values, weighted by how relevant each position's key was to the query. Dividing by $\sqrt{d_k}$ (the key dimension) keeps the dot products from growing too large as dimensionality increases, which would otherwise push softmax into a saturated, near-one-hot regime with vanishing gradients.

---

## Self-Attention

When Q, K, and V are all derived from the *same* sequence (rather than a decoder attending to a separate encoder), this is **self-attention** — every position in a sequence gets to attend to every other position in that same sequence, directly, regardless of distance. This is the specific mechanism that replaced recurrence entirely in the Transformer: an [[Recurrent Neural Network (RNN)|RNN]] has to pass information through every intermediate time step to connect two distant tokens (which is exactly why [[Vanishing and Exploding Gradients in RNNs|gradients vanish over long sequences]]); self-attention connects any two tokens in a single computation step, independent of how far apart they are in the sequence.

## Multi-Head Attention

Rather than computing one attention distribution, **multi-head attention** runs several attention computations in parallel, each with its own learned Q/K/V projections, then concatenates and linearly combines the results:

$$\text{MultiHead}(Q,K,V) = \text{Concat}(\text{head}_1,\dots,\text{head}_h)W^O, \quad \text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$$

Each head can specialize in a different kind of relationship — one head might track syntactic dependency, another coreference, another local adjacency — giving the model several independent "views" of the sequence's relationships rather than forcing everything through one attention pattern.

---

## Why Attention Solves the Bottleneck, Concretely

A seq2seq encoder-decoder without attention compresses an entire input sentence into one fixed-size vector, regardless of sentence length — a 5-word sentence and a 50-word sentence get squeezed into the same-size representation, and information from early parts of a long sentence is disproportionately likely to be lost by the time the encoder reaches the end. With attention, the decoder never relies on a single compressed vector at all — it directly re-examines every encoder position, every time it generates an output, weighted by relevance to what it's currently producing.

---

## Interview Questions

**What problem does attention solve that a plain encoder-decoder RNN doesn't?** The fixed-size context vector bottleneck — a plain seq2seq model compresses an entire input sequence into one vector regardless of length, which loses information especially for long sequences; attention lets the decoder look back at every input position directly, weighted by relevance, instead of relying on one compressed summary.

**What are Query, Key, and Value, in plain terms?** Query is what the current position is looking for; Key is what each position advertises as its content, for matching against the query; Value is what actually gets retrieved once a match is found. The dot product of Query and Key produces relevance scores, softmax turns those into weights, and the output is a weighted sum of Values.

**Why divide by $\sqrt{d_k}$ in the attention formula?** Without it, dot products grow in magnitude as the key dimension increases, pushing softmax's input into a regime where it's nearly one-hot and its gradient is nearly zero — scaling keeps the dot products in a range where softmax's gradient stays informative.

**Why use multiple attention heads instead of one?** Different heads can learn to attend to different kinds of relationships in parallel (syntax, coreference, local proximity, etc.) using independent projections, giving the model several simultaneous "views" rather than forcing every relationship through a single attention pattern.

**How does self-attention address the vanishing-gradient problem that plagues RNNs on long sequences?** An RNN must pass information through every intermediate time step to connect two distant tokens, so the gradient between them shrinks (or explodes) with the number of steps between them; self-attention connects any two positions directly in one computation, with no dependency on their distance in the sequence.

## Connections

- [[Sequence-to-Sequence Models]] — the bottleneck problem attention was invented to solve
- [[Vanishing and Exploding Gradients in RNNs]] — the long-range-dependency problem self-attention sidesteps entirely
- [[Transformer Architecture]] — the architecture built entirely around self-attention, with no recurrence at all
- [[Transformer End-to-End Walkthrough]] — a worked numerical example threading this mechanism through a full input-to-output pipeline
- [[Recurrent Neural Network (RNN)]] — the architecture attention (and eventually the Transformer) replaced

## One-line Summary

> Attention computes a weighted average over all input positions using learned Query/Key/Value projections, letting a model connect any two positions directly regardless of distance — solving the seq2seq fixed-vector bottleneck and, in self-attention form, sidestepping the vanishing-gradient problem that limits RNNs on long sequences.
