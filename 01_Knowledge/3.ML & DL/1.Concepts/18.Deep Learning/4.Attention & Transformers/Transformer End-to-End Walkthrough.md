# Transformer End-to-End Walkthrough

## What This Note Is

A **synthesis note** — not new theory. It exists to answer one specific interview question: *"Explain a Transformer from scratch."* Everything here is already covered, correctly and in more depth, by [[Tokenization — BPE]], [[Word2Vec]]/[[BERT Embeddings]], [[Attention Mechanism]], [[Transformer Architecture]], [[LLM Inference Fundamentals]], and [[Inference vs Training]] — this note's only job is to walk one input sequence through all of them **in order, with one small numerical example threading the whole way through**, so the pipeline is one continuous mental model instead of five separately-understood pieces.

---

## The Full Pipeline, at a Glance

```
Raw text → Tokenization → Token IDs → Embeddings → + Positional Encoding
  → [Q/K/V projection → Scaled dot-product attention (± causal mask)
     → Multi-head concat → Residual + LayerNorm → Feed-forward
     → Residual + LayerNorm] × N layers
  → Final hidden state → Vocabulary projection → Logits
  → (temperature / top-k / top-p) → Probability distribution → Sampled token
  → Append token, repeat (autoregressive loop)
```

---

## Worked Numerical Example

**Setup** (deliberately tiny, to keep every number checkable by hand): vocabulary `{the, cat, sat, mat}`, input sequence `"the cat sat"`, embedding dimension $d=2$. To keep the arithmetic focused on the attention mechanism itself rather than projection bookkeeping, this example uses **identity Q/K/V projection matrices** ($W_Q=W_K=W_V=I$, so $Q=K=V=x$) and **one attention head, one Transformer block**. Real models use learned, non-identity projections and stack many heads and blocks — the mechanism is identical, just repeated and composed. All numbers below are arbitrary/untrained, chosen for tractability, not semantic realism.

### 1. Tokenization → Token IDs

`"the cat sat"` → tokens `[the, cat, sat]` → IDs `[0, 1, 2]` (see [[Tokenization — BPE]] for how real tokenizers split words into subwords — this toy example uses whole-word tokens for simplicity).

### 2. Embeddings

Each ID looks up a row in an embedding table (see [[Word2Vec]] for why this lookup encodes semantic similarity as geometric proximity):
$$E(\text{the})=[1,0], \quad E(\text{cat})=[0,1], \quad E(\text{sat})=[1,1]$$

### 3. + Positional Encoding

Self-attention has no inherent sense of order (see [[Transformer Architecture]]), so we add the sinusoidal positional encoding at each position, using the formula from that note with $d=2$:
$$PE(0)=[\sin 0, \cos 0]=[0,1], \quad PE(1)=[\sin 1,\cos 1]\approx[0.841,0.540], \quad PE(2)=[\sin 2,\cos 2]\approx[0.909,{-}0.416]$$

$$x_0 = [1,0]+[0,1] = [1,1] \qquad x_1 = [0,1]+[0.841,0.540]=[0.841,1.540] \qquad x_2=[1,1]+[0.909,{-}0.416]=[1.909,0.584]$$

This is what actually enters the first Transformer block — not the raw embedding.

### 4. Q/K/V and Scaled Dot-Product Attention (for position 1, "cat")

With $Q=K=V=x$ (our simplification), compute how much position 1's query attends to every key:
$$q_1 \cdot k_0 = (0.841)(1)+(1.540)(1)=2.381, \quad q_1\cdot k_1 = 0.841^2+1.540^2=3.079, \quad q_1\cdot k_2 = 2.505$$

Scale by $\sqrt{d_k}=\sqrt2\approx1.414$ (see [[Attention Mechanism]] for why — unscaled dot products push softmax into a saturated, low-gradient regime): $[1.684,\ 2.178,\ 1.772]$.

**Softmax** → weights $[0.268,\ 0.439,\ 0.293]$ (these sum to 1 — this is the attention distribution: position 1 attends 26.8% to "the," 43.9% to itself, 29.3% to "sat").

**Weighted sum of values**:
$$\text{output}_1 = 0.268[1,1]+0.439[0.841,1.540]+0.293[1.909,0.584] = [1.196,\ 1.115]$$

### 5. Causal Masking — the Same Computation, With the Future Blocked Out

A decoder generating text must not let position 1 see position 2 ("sat") — that would let it cheat by looking at a token it's supposed to be predicting. **Causal masking** sets the score for any future position to $-\infty$ before softmax, so its weight becomes exactly 0. Recomputing position 1's attention **without** the position-2 score at all:

Scores over just $k_0,k_1$: $[1.684,\ 2.178]$ → softmax → $[0.379,\ 0.621]$ → output $= 0.379[1,1]+0.621[0.841,1.540] = [\mathbf{0.901,\ 1.335}]$.

**Compare to step 4's unmasked result, $[1.196, 1.115]$ — a materially different vector.** This is the concrete, numerical answer to "why does masking matter": it isn't a bookkeeping detail, it changes what the model actually computes at that position, because it changes which values get averaged into the output.

### 6. Multi-Head Attention (Conceptual — Same Math, Repeated)

Real Transformers don't do the above once — they do it $h$ times in parallel, each with its own learned $W_Q^{(i)}, W_K^{(i)}, W_V^{(i)}$, then concatenate the $h$ outputs and pass through one more linear layer $W^O$ (see [[Attention Mechanism]]). Numerically this is exactly step 4/5 repeated with different projection matrices and concatenated — no new mechanism, just more of the same computation run side by side so different heads can specialize in different relationships.

### 7. Residual Connection + LayerNorm

$$\text{residual} = \text{attention output} + x_1 = [0.901,1.335] + [0.841,1.540] = [1.742,\ 2.875]$$

Then [[Batch and Layer Normalization|LayerNorm]] normalizes this vector (per-example, across its own features — see that note for the exact formula) before it continues into the feed-forward sublayer. The residual add is why gradients can flow through dozens of stacked blocks without vanishing — see [[Backpropagation]]'s treatment of exactly this mechanism.

### 8. Feed-Forward Network

$$\text{FFN}(x) = W_2\,\text{ReLU}(W_1 x + b_1) + b_2$$

Applied identically and independently to every position — the same [[Multi-Layer Perceptron]] mechanics from Neural Network Foundations, just run per-token. Followed by another residual+LayerNorm (omitted numerically here for brevity — same pattern as step 7).

### 9. Stack $N$ Blocks

Steps 4–8 are **one Transformer block**. Real models stack many (GPT-3: 96 blocks). Each block's output becomes the next block's input; nothing new happens at the architecture level — the depth is what lets increasingly abstract relationships be built on top of earlier ones.

### 10. Vocabulary Projection → Logits

Treat the (simplified, one-block) output from step 7 as the final hidden state for the last position: $h=[1.742, 2.875]$. Project it through a $d \times |V|$ matrix onto the vocabulary — one score ("logit") per possible next token. With toy vocab rows `the=[1,0], cat=[0,1], sat=[1,1], mat=[0.5,0.5]`:

$$\text{logit(the)}=1.742, \quad \text{logit(cat)}=2.875, \quad \text{logit(sat)}=4.617, \quad \text{logit(mat)}=2.309$$

### 11. Softmax → Probability Distribution, and Where Sampling Parameters Enter

**Temperature $T=1$** (plain softmax): $P=[0.042,\ 0.132,\ 0.751,\ 0.075]$ for `[the, cat, sat, mat]`. **Greedy decoding** just takes $\arg\max$ → `"sat"` (this toy result isn't meant to be semantically sensible — the weights are arbitrary, not trained).

**Temperature $T=2$** (divide logits by $T$ before softmax): $P=[0.120,\ 0.212,\ 0.508,\ 0.160]$ — visibly flatter; "sat" still wins but far less dominantly. This is the numeric answer to "what does raising temperature actually do": it doesn't add randomness directly, it **compresses the gap between logits before they compete in softmax.**

**Top-k, $k=2$** (keep only the 2 highest-logit tokens, renormalize): candidates `{sat, cat}` → renormalized $P=[\text{sat}{:}0.851,\ \text{cat}{:}0.149]$. "the" and "mat" are excluded entirely, however small the improvement from including them would have been.

**Top-p (nucleus), $p=0.9$** (using the $T{=}1$ distribution, sort descending and keep the smallest prefix whose cumulative probability exceeds 0.9): `sat` (0.751) → cumulative 0.751; `+cat` (0.132) → 0.883, still under 0.9; `+mat` (0.075) → 0.958, now over 0.9 → nucleus = `{sat, cat, mat}`, renormalized to $[0.784, 0.138, 0.078]$.

**Top-k vs. top-p, made concrete by this example**: top-k always keeps exactly 2 candidates, *regardless of how confident the model is*; top-p kept 3 here because the distribution wasn't concentrated enough to cross 0.9 with only 2 — top-p adapts to the model's actual confidence, top-k doesn't. See [[Sampling and Decoding Strategies]] for the full treatment of these techniques, their interactions, and when each is preferred.

### 12. Sample a Token, Then Repeat (Autoregressive Loop)

Whichever token gets sampled (say "sat," under greedy decoding) is appended to the sequence, and the **entire pipeline above runs again** with the new, longer sequence — `"the cat sat"` becomes the input for predicting the *next* token after that. This repeats until an end-of-sequence token is generated or a length limit is hit. Every generated token requires one full pass through every block, which is the direct source of generation latency (see [[LLM Inference Fundamentals]] and [[Latency and Cost Optimization]]).

---

## Encoder-Only vs. Decoder-Only vs. Encoder-Decoder

Covered in full in [[Transformer Architecture]] — summary for this walkthrough's purposes: **encoder-only** (BERT) uses no causal mask, every position sees every other position, suited to understanding tasks. **Decoder-only** (GPT and most modern LLMs) uses the causal masking shown in step 5 above, generating autoregressively as walked through here. **Encoder-decoder** (T5, translation models) runs a bidirectional encoder once, then a causal decoder that also cross-attends to the encoder's output at every layer.

## Training vs. Inference — What Actually Differs

**Training**: the *entire* target sequence is available at once. Because of causal masking, the model can compute the prediction for *every* position in parallel in a single forward pass (predict position 1 from position 0, position 2 from positions 0-1, etc., all simultaneously) — this parallelism, not just self-attention's per-token computation, is a major reason Transformers train fast. Loss is cross-entropy between the predicted distribution and the actual next token, at every position, averaged.

**Inference (generation)**: the target doesn't exist yet — it's exactly what's being produced, one token at a time, autoregressively, as walked through in steps 1-12. This is fundamentally sequential (token $t{+}1$'s input includes token $t$, which didn't exist until the previous step finished) — no equivalent parallelism across generation steps exists, which is precisely why generation is comparatively slow even though the underlying architecture is highly parallelizable. See [[Inference vs Training]] for the general version of this distinction.

## The KV Cache

Notice steps 4-9 recompute $K$ and $V$ for *every* position, every single generation step — but positions 0 through $t{-}1$'s embeddings and hidden states haven't changed since the last step; only the newest token is new. The **KV cache** stores each layer's $K$ and $V$ vectors for every previously-processed position, so each new generation step only computes $Q$, $K$, $V$ for the *single new token* and reuses the cached $K,V$ for everything before it, rather than recomputing the whole sequence's $K,V$ from scratch at every step. This turns what would be $O(n^2)$ *repeated* work across a full generation into $O(n)$ new work per step (still $O(n^2)$ total across a full generation, but without the wasteful re-computation) — a purely inference-time engineering optimization; the mathematical result is identical either way.

## Why Attention Scales Poorly With Sequence Length

Step 4's score computation was 3 dot products for a 3-token sequence — a $3\times3$ matrix of pairwise scores. For $n$ tokens, that's $n^2$ pairwise scores, computed and stored at every layer, every head. This quadratic cost in both compute *and* memory is exactly the hard architectural limit behind [[LLM Inference Fundamentals|context window]] size — doubling the context length roughly quadruples the attention cost per layer, which is why context windows have a ceiling rather than being unbounded, and why the KV cache above (which still stores $O(n)$ growing state) becomes a real memory constraint for very long conversations.

---

## 30-Second Interview Answer

"A Transformer turns text into tokens, embeds them, and adds positional encoding since attention itself has no sense of order. Each block computes self-attention — every token forms a query, key, and value, and attention is a weighted average of values, weighted by how well each token's query matches every other token's key — then adds a residual connection, normalizes, and passes through a feed-forward layer. Stack that block many times. For generation, a causal mask stops each position from seeing future tokens, the final hidden state projects onto the vocabulary to get logits, softmax turns those into a probability distribution, and a token is sampled — using temperature/top-k/top-p to control how random that sampling is — then the whole sequence, one token longer, runs through again."

## 2-Minute Interview Answer

Extend the above with: (1) the actual attention formula $\text{softmax}(QK^\top/\sqrt{d_k})V$ and why the $\sqrt{d_k}$ scaling matters (unscaled dot products saturate softmax); (2) multi-head attention as running that computation several times in parallel with different learned projections, so different heads can specialize; (3) the encoder/decoder/encoder-decoder distinction and which one modern LLMs use (decoder-only, causal); (4) that training processes a whole sequence in parallel (teacher forcing) while inference is inherently sequential, one token at a time, which is why generation latency and the KV cache optimization exist; (5) that attention's cost grows quadratically with sequence length, which is the direct architectural reason context windows are bounded.

## Follow-Up Questions an Interviewer Could Ask Next

- "Derive why the gradient of softmax saturates without the $\sqrt{d_k}$ scaling." → [[Attention Mechanism]]
- "Why can't training also be done token-by-token like inference?" → it could, but it would throw away the parallelism causal masking enables during training — see Training vs. Inference above
- "What happens to the KV cache's memory footprint as context length grows?" → grows linearly with sequence length per layer/head, becoming a real serving constraint for long contexts — see [[Latency and Cost Optimization]]
- "How would you reduce attention's quadratic cost for very long sequences?" → sparse/local attention patterns, or retrieval instead of stuffing everything into context — see [[RAG Architecture]]
- "Walk through what changes if this were an encoder-only model instead of decoder-only." → step 5's causal mask disappears entirely; every position attends to every other position, both past and future
- "Why does raising temperature to a very large value eventually make sampling nearly uniform?" → dividing logits by a very large $T$ shrinks all of them toward 0, and softmax of nearly-equal inputs is nearly uniform — visible directly in this note's $T{=}2$ vs. $T{=}1$ comparison, taken to its limit

## Connections

- [[Tokenization — BPE]], [[Word2Vec]], [[BERT Embeddings]] — steps 1-2, covered canonically in NLP
- [[Attention Mechanism]], [[Transformer Architecture]] — steps 3-9, the architecture this note walks through numerically
- [[Batch and Layer Normalization]], [[Backpropagation]], [[Multi-Layer Perceptron]] — the mechanics inside each block
- [[Sampling and Decoding Strategies]] — the full treatment of step 11's temperature/top-k/top-p
- [[Inference vs Training]], [[LLM Inference Fundamentals]] — the practical/systems layer this walkthrough feeds into
- [[RAG Architecture]] — a direct consequence of attention's quadratic cost limiting context length

## One-line Summary

> Every stage of a Transformer — tokenize, embed, add position, attend (masked or not), normalize, feed-forward, stack, project to vocabulary, sample — is individually simple; what makes "explain a Transformer from scratch" hard is holding the whole chain in your head at once, which is exactly what threading one small numerical example through every stage, as this note does, is meant to fix.
