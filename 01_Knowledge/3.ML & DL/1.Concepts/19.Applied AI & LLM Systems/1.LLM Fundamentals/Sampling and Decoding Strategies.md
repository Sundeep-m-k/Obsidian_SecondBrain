# Sampling and Decoding Strategies

## What is it?

Once a Transformer produces logits over the vocabulary for the next token (see [[Transformer End-to-End Walkthrough]], step 10), a **decoding strategy** decides which token actually gets chosen. This is a genuinely consequential choice, not an afterthought — the same model, same logits, produces very different generation behavior (deterministic and safe vs. creative and occasionally incoherent) purely as a function of which decoding strategy and parameters are used downstream of an identical forward pass.

---

## Shared Example Used Throughout This Note

Logits for the next token over vocabulary `{the, cat, sat, mat}`: $[1.742,\ 2.875,\ 4.617,\ 2.309]$ (the same numbers produced in [[Transformer End-to-End Walkthrough]]'s worked example). Plain softmax ($T{=}1$) gives:
$$P = [\text{the}{:}0.042,\ \text{cat}{:}0.132,\ \text{sat}{:}0.751,\ \text{mat}{:}0.075]$$

---

## Greedy Decoding

Always pick $\arg\max$ of the distribution — here, `"sat"` (0.751). **Fully deterministic**: same input always produces the same output, every single time. No randomness anywhere in the process.

**When this is the right choice**: code generation, structured data extraction, factual short-answer Q&A, anything where consistency and reproducibility matter more than variety — and, practically, anywhere a bug report needs to be reproducible.

**The failure mode worth naming**: greedy decoding is provably *not* the same as finding the highest-probability *entire sequence* — greedily picking the best next token at every step can lock in an early choice that forecloses a better overall continuation. This is exactly the limitation **beam search**, covered next, was invented to reduce.

## Beam Search

Greedy decoding commits to one token per step and never reconsiders. **Beam search** instead tracks the $k$ (the **beam width**) most promising *entire sequences so far* at every step, expands each of them by one token, and keeps only the best $k$ resulting sequences — trading more compute for a better chance of finding a high-probability full sequence than greedy's single-path search allows.

### Mechanism

1. Start from the current sequence(s) held in the beam (initially just the prompt).
2. For each of the $k$ beams, compute the next-token distribution and consider extending it by every vocabulary token.
3. Score each resulting candidate sequence by its **cumulative sequence probability** — the product of every token's conditional probability so far, $P(\text{seq}) = \prod_t P(w_t \mid w_{<t})$.
4. Keep only the top $k$ candidate sequences by this score (**pruning**) — discard the rest, regardless of which beam they came from.
5. Repeat, extending the surviving $k$ sequences, until each has emitted an end-of-sequence (**EOS**) token or a maximum length is reached.
6. A beam that emits EOS is **completed** and set aside (no longer expanded); once all $k$ beams are completed (or the max length is hit), return the completed sequence with the best final score.

### Why Scores Are Accumulated in Log-Probability Space

Multiplying many probabilities together (each $\le 1$) underflows to numerically indistinguishable-from-zero after enough steps — a real floating-point problem, not just a style preference. Taking logs converts the product into a **sum**, $\log P(\text{seq}) = \sum_t \log P(w_t\mid w_{<t})$, which is numerically stable and, since $\log$ is monotonically increasing, preserves the same ranking as the raw probabilities would.

### Worked Numerical Example

Vocabulary `{A, B, C, EOS}`, **beam width $k=2$**. All probabilities below are illustrative (not from a real model) but the arithmetic is exact — computed as joint probabilities, then converted to $\log$ (natural log) at the end for verification.

**Step 1** — first-token distribution: $P(A){=}0.40,\ P(B){=}0.40,\ P(C){=}0.15,\ P(\text{EOS}){=}0.05$. Top $k{=}2$: **A** and **B** survive (tied at 0.40; C and EOS are pruned). $\log P(\text{"A"}) = \log P(\text{"B"}) = \ln(0.40) \approx -0.916$.

**Step 2** — expand both beams. Conditional distributions (illustrative): given "A": $P(B{\mid}A){=}0.5,\ P(\text{EOS}{\mid}A){=}0.3,\ P(C{\mid}A){=}0.2$; given "B" (symmetric): $P(A{\mid}B){=}0.5,\ P(\text{EOS}{\mid}B){=}0.3,\ P(C{\mid}B){=}0.2$.

| Candidate | Joint probability | $\log P$ |
|---|---|---|
| "A B" | $0.40{\times}0.5=0.20$ | $-1.609$ |
| "A EOS" | $0.40{\times}0.3=0.12$ | $-2.120$ |
| "A C" | $0.40{\times}0.2=0.08$ | $-2.526$ |
| "B A" | $0.40{\times}0.5=0.20$ | $-1.609$ |
| "B EOS" | $0.40{\times}0.3=0.12$ | $-2.120$ |
| "B C" | $0.40{\times}0.2=0.08$ | $-2.526$ |

Six candidates generated (2 beams × 3 vocab extensions each — EOS/A/B/C minus repeats), **pruned to the top $k{=}2$**: "A B" and "B A" (both 0.20) survive; the other four are discarded — a concrete instance of step 4's pruning, and note that *both* surviving beams came from extending with a non-EOS token, since 0.20 beat every EOS-terminated candidate at this step.

**Step 3** — expand the two survivors. Conditional distributions (illustrative, again symmetric): given "A B": $P(\text{EOS}{\mid}AB){=}0.6,\ P(C{\mid}AB){=}0.25,\ P(A{\mid}AB){=}0.15$; given "B A" (symmetric): same three probabilities over EOS/C/B.

| Candidate | Joint probability | $\log P$ |
|---|---|---|
| "A B EOS" | $0.20{\times}0.6=0.12$ | $-2.120$ |
| "A B C" | $0.20{\times}0.25=0.05$ | $-2.996$ |
| "A B A" | $0.20{\times}0.15=0.03$ | $-3.507$ |
| "B A EOS" | $0.20{\times}0.6=0.12$ | $-2.120$ |
| "B A C" | $0.20{\times}0.25=0.05$ | $-2.996$ |
| "B A B" | $0.20{\times}0.15=0.03$ | $-3.507$ |

Top $k{=}2$: **"A B EOS"** and **"B A EOS"** (both $0.12$) — both surviving beams happen to terminate at this step. Both are now **completed**; with no active beams left to expand, decoding stops here. **Final selection**: tied at $\log P = -2.120$ — either "A B" or "B A" (EOS stripped) is a valid final output; a real system breaks the tie deterministically (e.g. earliest-found, or vocabulary order).

This trace demonstrates every mechanism at once: cumulative scoring (multiplying joint probabilities / summing log-probabilities), pruning six candidates down to two at each step regardless of which parent beam produced them, and EOS ending a beam's expansion rather than the whole search.

### Length Bias, and Why Beam Search Often Prefers Shorter Sequences

Every additional token multiplies the sequence's joint probability by another factor $\le 1$ — so, **all else equal, a shorter completed sequence needs fewer such multiplications and tends to retain a higher raw cumulative probability than a longer one**, even when the longer sequence is actually higher-quality on a per-token basis. Concretely, compare two hypothetical completed beams:

- **Hypothesis 1** (short, 2 tokens, mediocre per-token confidence: 0.3, then 0.5): joint $= 0.15$, $\log P \approx -1.897$.
- **Hypothesis 2** (longer, 4 tokens, strong per-token confidence: 0.6 each): joint $= 0.6^4 = 0.1296$, $\log P \approx -2.043$.

**Raw score** ranks Hypothesis 1 higher ($-1.897 > -2.043$) purely because it's shorter — despite Hypothesis 2 being the more confident sequence at every single step. **Length normalization** fixes this by dividing by sequence length (or length raised to a tunable exponent $\alpha \approx 0.6$–$0.7$ in some implementations) before comparing:

$$\text{normalized score} = \frac{\log P(\text{seq})}{\text{length}}$$

Hypothesis 1 normalized: $-1.897/2 = -0.949$. Hypothesis 2 normalized: $-2.043/4 = -0.511$. **The ranking flips** — Hypothesis 2 now wins ($-0.511 > -0.949$), correctly reflecting that it was the more confident sequence per token, once sequence length stops being an unfair thumb on the scale.

### Complexity

Beam width $k$ multiplies both compute and memory relative to greedy decoding ($k{=}1$): at every step, $k$ beams are each expanded across the full vocabulary, scored, and pruned back down to $k$ — roughly $O(k)$ times the forward-pass and bookkeeping cost of greedy per generation step, and $O(k)$ times the memory to hold the active hypotheses (and their [[Transformer End-to-End Walkthrough|KV caches]], one per beam) simultaneously. Doubling beam width roughly doubles both cost and memory — a real, direct latency/quality tradeoff for a production system, not a free quality upgrade.

**Is a larger beam always better?** No — beyond a fairly small width (commonly 4–10 in practice), returns diminish quickly and can even reverse: larger beams have been empirically observed to sometimes produce *worse* output on open-ended generation (more generic, less coherent) even though they're searching a strictly larger space for the highest-scoring sequence — evidence that "highest probability" and "best output, as judged by a person" are related but not identical objectives.

### Beam Search Is Not Universally "Better" Than Sampling

Beam search searches for the single highest-scoring sequence — which is exactly the right objective for tasks with one (or a narrow band of) correct-ish answers: **machine translation**, **structured sequence generation**, and **summarization** in settings where faithfulness to the source matters more than variety. It is a poor fit for **open-ended conversational generation**, where the training distribution has many valid continuations and always picking the single most probable path tends to produce **generic, repetitive, or bland text** — sampling-based decoding (temperature/top-p) is generally preferred there specifically *because* it preserves the diversity beam search's search-for-the-best-single-answer objective suppresses. Neither is "better" in general — the right choice depends on whether the task has a narrow target (favors beam search) or a wide space of acceptable outputs (favors sampling).

### Comparing All Five Decoding Strategies

| | Deterministic? | Diversity | Compute cost | Memory cost | Typical use case |
|---|---|---|---|---|---|
| Greedy | Yes | None | Baseline ($k{=}1$) | Baseline | Code gen, structured extraction, anywhere reproducibility matters most |
| Beam search | Yes (for a fixed beam width) | Low — actively searches for one best sequence | $O(k)\times$ greedy | $O(k)\times$ greedy | Machine translation, summarization, other narrow-target sequence generation |
| Temperature sampling | No | Tunable via $T$ | Same as greedy | Same as greedy | General-purpose generation, tuned for the task |
| Top-k sampling | No | Bounded by fixed $k$ | Same as greedy | Same as greedy | Creative generation with a hard ceiling on how unlikely a token can be |
| Top-p sampling | No | Adapts to model confidence | Same as greedy | Same as greedy | Open-ended conversational generation, brainstorming |

## Temperature

Rescale logits by dividing by $T$ *before* softmax: $\text{softmax}(z/T)$.

$$T{=}1: P=[0.042,\ 0.132,\ 0.751,\ 0.075] \qquad T{=}2: P=[0.120,\ 0.212,\ 0.508,\ 0.160]$$

**What temperature mechanically does**: it doesn't add noise on top of the distribution — it changes the distribution's *shape* before any sampling happens, by compressing the gaps between logits. $T<1$ sharpens the distribution (more confident, closer to greedy — at $T\to0$ it becomes exactly greedy); $T>1$ flattens it (more uniform, more randomness, more likely to produce something unexpected or incoherent). $T=0$ is a special case handled as greedy decoding directly, since dividing by zero is undefined.

**Common misconception**: "temperature adds randomness to the model." It doesn't touch the model at all — the model's logits are identical regardless of temperature; only the *sampling* step downstream changes.

## Top-k Sampling

Keep only the $k$ highest-logit tokens, discard everything else, renormalize among survivors, then sample.

$$k{=}2 \text{ (from } T{=}1 \text{ distribution): candidates } \{\text{sat}, \text{cat}\} \to \text{renormalized } P=[\text{sat}{:}0.851,\ \text{cat}{:}0.149]$$

**What problem this solves**: without any cutoff, even a token with tiny probability (0.001%) has *some* chance of being sampled, and across a long generation, rare-token sampling accumulates into a real chance of picking something incoherent. Top-k caps the "bad tail" at a fixed size.

**The problem top-k *doesn't* solve**: $k$ is fixed regardless of the model's actual confidence. If the model is extremely confident (one token at 0.98), top-$k{=}50$ still considers 49 essentially-irrelevant tokens as live candidates. If the model is genuinely uncertain across many plausible tokens, top-$k{=}50$ might exclude a token that deserved consideration. This fixed-count blindness to the shape of the actual distribution is exactly what top-p was designed to fix.

## Top-p (Nucleus) Sampling

Sort tokens by probability descending, keep the smallest prefix whose cumulative probability exceeds $p$, renormalize, sample.

$$p{=}0.9 \text{ (from } T{=}1\text{): sat (0.751)} \to \text{cumulative } 0.751;\ {+}\text{cat (0.132)}\to0.883;\ {+}\text{mat (0.075)}\to0.958 \ (\ge0.9)$$

Nucleus $= \{\text{sat, cat, mat}\}$ (3 tokens), renormalized to $[0.784, 0.138, 0.078]$.

**Why this is the fix for top-k's blindness**: the *size* of the nucleus adapts automatically to how peaked or flat the underlying distribution already is — a very confident distribution produces a small nucleus (maybe just 1-2 tokens); a very uncertain, flat distribution produces a large one. Top-p asks "how many tokens does the model's own confidence justify considering," rather than fixing that count in advance.

---

## Interactions Between Parameters

**Temperature + top-k**: temperature is applied *first* (reshaping the distribution), *then* top-k selects from the reshaped distribution. Raising temperature before a fixed top-k can change *which* tokens even qualify for the top-$k$ cutoff (a token that was rank 6 at $T{=}1$ might become rank 3 at $T{=}2$, since temperature changes relative rankings' *closeness* even though it doesn't change their *order*) — practically, temperature changes how sharply top-k's fixed candidate set is weighted, but not which set gets selected in this simple case (temperature doesn't reorder logits, only rescales them, so the top-k *set* of tokens is temperature-invariant even though their relative probabilities within that set are not).

**Temperature + top-p**: this interaction is more consequential — since top-p's nucleus *size* depends on the distribution's shape, and temperature directly changes that shape, raising temperature before top-p typically *widens* the nucleus (a flatter distribution needs more tokens to reach the same cumulative probability $p$). Lowering temperature *narrows* it. This means "temperature 0.7, top-p 0.9" and "temperature 1.3, top-p 0.9" can select from meaningfully different-sized candidate sets even with an identical $p$.

**Common production default**: temperature in the 0.3–0.7 range combined with top-p around 0.9–0.95 for most general-purpose generation — low enough temperature to avoid incoherence, top-p providing an adaptive safety net against an unlucky low-probability sample.

---

## Deterministic vs. Stochastic Decoding — When Each Matters

**Deterministic (greedy, or temperature effectively 0)**: reproducible, testable, auditable — the same input always gives the same output, which matters enormously for debugging a production system, for regression testing prompt changes (see [[Observability and Evaluation for LLM Systems]]), and for any application where two identical requests giving different answers would itself look like a bug.

**Stochastic (temperature > 0, sampling-based)**: necessary for creative or exploratory generation (brainstorming, varied phrasing across many outputs), and in agentic systems where trying a different approach on retry is actually desirable rather than repeating an identical failed action.

---

## Common Misconceptions

- **"Lower temperature always means better quality."** Not quite — very low temperature can produce repetitive, degenerate text (the model gets stuck favoring the same few high-probability continuations, sometimes looping), which is why 0 isn't automatically the "best" setting even for factual tasks; a small amount of temperature can actually improve output quality by avoiding repetition loops.
- **"Top-k and top-p do the same thing, just parameterized differently."** They solve different problems — top-k fixes candidate *count*, top-p fixes candidate *cumulative probability mass* — and they behave identically only by coincidence for any specific distribution, diverging exactly when the distribution's confidence varies across different generation steps (which it always does in practice).
- **"You should pick either top-k or top-p, not both."** Many production systems apply both together (e.g. top-k as a hard ceiling, top-p as the adaptive filter within it) — they're complementary constraints, not mutually exclusive alternatives.

---

## Interview Questions

**Given a peaked distribution vs. a flat distribution, how would top-k and top-p behave differently?** Top-k always returns exactly $k$ candidates regardless of shape — potentially including near-irrelevant tokens from a peaked distribution, or excluding a plausible one from a flat distribution; top-p's candidate count shrinks automatically for a peaked distribution (fewer tokens needed to reach cumulative probability $p$) and grows for a flat one — it adapts to the model's actual confidence, top-k doesn't.

**Why would you use greedy decoding in a production system, given that it's "less creative"?** Whenever reproducibility matters more than variety — code generation, structured extraction, anything needing to be regression-tested or debugged reliably; a non-deterministic output for an identical input is itself often a defect in these contexts, not a feature.

**Does raising temperature change which token has the highest probability?** No — temperature rescales logits by a positive constant, which preserves their relative order; it changes *how concentrated* probability is around that top token, not *which* token is on top (that would require the underlying logits themselves to change).

**Why might a system combine top-k and top-p rather than choosing one?** They constrain different things — top-k caps the absolute number of candidates (a hard ceiling against ever considering too many low-probability tokens), top-p caps candidates by cumulative probability mass (adapting to confidence) — using both gives a hard worst-case bound while still adapting within it.

**A generation system produces oddly repetitive text at very low temperature — why, and what would you change?** Very low (near-zero) temperature makes the model pick its single most likely continuation almost every time, which can trap it in a repetition loop once a repeated phrase becomes locally "most probable" again; raising temperature slightly, or adding a repetition penalty (a separate technique not covered in depth here), typically fixes this.

**What is beam search, and how does it differ from greedy decoding?** Greedy decoding commits to the single best next token at every step and never reconsiders; beam search keeps the top-$k$ highest-scoring *entire sequences* at each step, expanding and re-pruning all of them together, which lets it find a better overall sequence than greedy's single, irrevocable path — at $k{\times}$ the compute and memory cost.

**Beam search vs. top-k sampling — what's the actual difference in objective?** Top-k samples randomly from the $k$ most probable *next tokens* at a single step, aiming for varied, natural-feeling output; beam search deterministically keeps the $k$ most probable *entire sequences so far*, aiming to find the single highest-scoring complete sequence — one is stochastic single-step filtering, the other is deterministic multi-step search.

**Why use log-probabilities instead of raw probabilities when scoring beams?** Multiplying many probabilities (each $\le 1$) underflows to zero in floating point after enough steps; summing log-probabilities is numerically stable and preserves the identical ranking, since $\log$ is monotonically increasing.

**Why does beam search have a length bias?** Every additional token multiplies the cumulative probability by another factor $\le 1$, so shorter sequences need fewer such multiplications and mechanically tend to retain higher raw cumulative probability than longer ones, independent of per-token quality — length normalization (dividing by sequence length) corrects for this, as shown in the worked example above where it flips which of two hypotheses ranks higher.

**Is a larger beam width always better?** No — cost and memory scale directly with $k$, and beyond a fairly small width, output quality on open-ended tasks can actually get *worse* (more generic/bland), not better, even though the search is technically covering more of the sequence space.

**Why isn't beam search used for every chatbot response, given that it searches more thoroughly than greedy?** Open-ended conversational generation has many valid continuations, and beam search's objective — find the single highest-probability sequence — tends to produce generic, repetitive text in that setting; sampling-based decoding (temperature/top-p) preserves the diversity a conversational response benefits from, which beam search actively searches away from.

**When would you choose beam search over sampling-based decoding?** When the task has a narrow, well-defined target rather than many acceptable outputs — machine translation, summarization, or other structured sequence generation — where finding the single best-scoring sequence is actually the right objective, unlike open-ended generation.

## Connections

- [[Transformer End-to-End Walkthrough]] — where these logits come from, and this note's shared worked example
- [[LLM Inference Fundamentals]] — the broader context (context windows, structured outputs) this decoding step sits within
- [[Observability and Evaluation for LLM Systems]] — why deterministic decoding matters for regression testing prompt/model changes
- [[Activation Functions]] — softmax, the function every decoding strategy here operates on top of

## One-line Summary

> Greedy decoding and beam search are both deterministic sequence-*search* strategies (beam search searching more broadly, at $k\times$ the cost, and needing length normalization to avoid an unfair bias toward shorter sequences); temperature, top-k, and top-p are token-level *sampling* strategies trading determinism for diversity — beam search suits narrow-target tasks like translation, sampling suits open-ended generation, and neither is universally better.
