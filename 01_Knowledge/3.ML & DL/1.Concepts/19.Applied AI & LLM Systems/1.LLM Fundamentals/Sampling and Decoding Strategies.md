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

**The failure mode worth naming**: greedy decoding is provably *not* the same as finding the highest-probability *entire sequence* — greedily picking the best next token at every step can lock in an early choice that forecloses a better overall continuation (a well-known limitation motivating beam search, which tracks several candidate sequences in parallel rather than committing to one token at a time — outside this note's scope, but worth knowing the term exists).

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

## Connections

- [[Transformer End-to-End Walkthrough]] — where these logits come from, and this note's shared worked example
- [[LLM Inference Fundamentals]] — the broader context (context windows, structured outputs) this decoding step sits within
- [[Observability and Evaluation for LLM Systems]] — why deterministic decoding matters for regression testing prompt/model changes
- [[Activation Functions]] — softmax, the function every decoding strategy here operates on top of

## One-line Summary

> Temperature reshapes the probability distribution before sampling (sharper below $T{=}1$, flatter above), top-k caps candidates by a fixed count, and top-p caps them by cumulative probability mass, adapting to the model's actual confidence — greedy decoding (temperature effectively 0) trades all variety for full reproducibility, which is exactly when it's the right choice.
