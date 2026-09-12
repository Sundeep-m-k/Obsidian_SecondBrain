---
tags: [nlp, sparse-representations, pmi, ppmi, distributional-semantics, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Bag of Words]] [[Co-occurrence Matrix]] [[Word2Vec]] [[GloVe]]"
---

# PMI and PPMI

## Definition + Intuition

**Pointwise Mutual Information (PMI)** measures the association between two words — specifically, how much more often they co-occur than would be expected if they were statistically independent. It is the theoretical cornerstone of distributional semantics and the conceptual precursor to GloVe.

**Positive PMI (PPMI)** is PMI with all negative values replaced by zero. It is the practical form used in NLP, because negative PMI values are unreliable (based on non-co-occurrence counts which are noisy).

**Intuition**: PMI asks: "Given that I saw the word *ice*, how surprised am I to also see *cream* nearby?" If *cream* appears near *ice* far more than chance would predict, PMI is high. If they co-occur at exactly the rate chance predicts, PMI = 0. If they co-occur less than chance, PMI is negative.

The key insight: words that co-occur in similar contexts tend to have similar meanings (distributional hypothesis). PMI/PPMI captures this at the word-pair level; dense embeddings (Word2Vec, GloVe) learn to implicitly approximate it.

---

## Key Properties / Types

**PMI (raw)**
$$\text{PMI}(w, c) = \log \frac{P(w, c)}{P(w) \cdot P(c)}$$

Properties:
- = 0 when $w$ and $c$ are independent
- > 0 when $w$ and $c$ co-occur more than chance
- < 0 when $w$ and $c$ co-occur less than chance
- Negative values are noisy and unreliable — PPMI fixes this
- Biased toward rare words: a word that appears once with another word gets a very high PMI

**PPMI (Positive PMI)**
$$\text{PPMI}(w, c) = \max(\text{PMI}(w, c), 0)$$

Properties:
- Sets unreliable negatives to 0
- Still biased toward rare words
- Standard baseline for word vector quality benchmarks

**Shifted PPMI**
$$\text{SPPMI}(w, c) = \max(\text{PMI}(w, c) - \log k, 0)$$

Where $k$ is the number of negative samples — this connects PPMI directly to the Word2Vec negative sampling objective (Levy & Goldberg, 2014).

**Context distribution smoothing**
Raises context word counts to power $\alpha < 1$ (typically 0.75) before computing PMI — reduces bias toward rare context words:
$$P_\alpha(c) \propto \text{count}(c)^\alpha$$

---

## Math / Formal Notation

**Estimating probabilities from co-occurrence counts**:

Given a corpus and a context window of size $k$:
- $\text{count}(w, c)$ = number of times word $w$ appears within $k$ words of context word $c$
- $\text{count}(w)$ = total occurrences of $w$
- $\text{count}(c)$ = total occurrences of $c$
- $N$ = total co-occurrence pairs in corpus

$$P(w, c) = \frac{\text{count}(w, c)}{N}, \quad P(w) = \frac{\text{count}(w)}{N}, \quad P(c) = \frac{\text{count}(c)}{N}$$

Substituting into PMI:
$$\text{PMI}(w, c) = \log \frac{\text{count}(w, c) \cdot N}{\text{count}(w) \cdot \text{count}(c)}$$

**The PPMI matrix**:
Let $M \in \mathbb{R}^{|V| \times |V|}$ where $M_{wc} = \text{PPMI}(w, c)$.
Row $w$ of this matrix is the **PPMI vector** for word $w$ — a sparse, high-dimensional representation of its distributional context.

**Connection to Word2Vec (Levy & Goldberg 2014)**:
The skip-gram with negative sampling objective implicitly factorises the **shifted PMI matrix**:
$$\text{PMI}(w, c) - \log k$$

This means Word2Vec is approximately doing dimensionality reduction on the PPMI matrix.

---

## Examples (Concrete)

**Corpus**: 1,000 documents. Counts observed:
- $\text{count}(\text{ice, cream}) = 100$
- $\text{count}(\text{ice}) = 500$
- $\text{count}(\text{cream}) = 200$
- $N = 10{,}000$

$$\text{PMI}(\text{ice, cream}) = \log \frac{100 \times 10{,}000}{500 \times 200} = \log \frac{1{,}000{,}000}{100{,}000} = \log 10 \approx 2.30$$

Strong positive association — *ice* and *cream* co-occur 10× more than chance.

**Contrast**:
- $\text{PMI}(\text{the, cream}) \approx 0$ — "the" appears with everything at roughly background rate
- $\text{PMI}(\text{democracy, cream}) < 0$ — rarely co-occur; PPMI sets this to 0

**PPMI vector for "ice"** (simplified):

| Context word | PMI | PPMI |
|-------------|-----|------|
| cream | 2.30 | 2.30 |
| cold | 1.85 | 1.85 |
| snow | 1.42 | 1.42 |
| hockey | 1.10 | 1.10 |
| the | 0.02 | 0.02 |
| democracy | -1.20 | **0.00** |

Row "ice" in the PPMI matrix captures the distributional meaning of *ice*.

---

## How It Connects to ML / NLP

| PMI Concept | ML/NLP Application |
|------------|-------------------|
| PMI matrix row = word vector | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — distributional semantic representation |
| Co-occurrence probabilities | [[3.ML & DL/1.Concepts/1.Foundations/Features.md]] — association statistics as features |
| PPMI + SVD = LSA word vectors | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Dimensionality Reduction.md]] — truncated SVD on PPMI matrix |
| PMI implicit in Word2Vec | [[Word2Vec]] — Levy & Goldberg (2014) showed skip-gram ≈ shifted PMI factorisation |
| PMI implicit in GloVe | [[GloVe]] — log-bilinear model approximates PMI |
| Rare word bias in PMI | [[3.ML & DL/1.Concepts/8.Model Behavior/Bias.md]] — rare words inflate PMI; smoothing corrects this |

**When PPMI vectors are used directly**:
- Baseline for intrinsic evaluation benchmarks (SimLex-999, WordSim-353)
- Feature input to downstream classifiers when neural embeddings are too expensive
- Analysis tool: PPMI reveals which contexts define a word's meaning

**Conceptual importance**:
PMI is the theoretical foundation explaining *why* distributional word vectors work. Understanding PMI makes Word2Vec and GloVe intuitive rather than black-box.

---

## Common Interview Questions

**Q: What does PMI measure? Give an intuitive explanation.**
A: PMI measures how much two words co-occur compared to what chance would predict. If $\text{PMI}(w, c) > 0$, $w$ and $c$ are associated (co-occur more than expected). PMI = 0 means independence. PMI < 0 means the words avoid each other (rarer than chance).

**Q: Why is PPMI preferred over raw PMI?**
A: Negative PMI values are unreliable because they depend on non-occurrence counts — how often two words *didn't* co-occur. This is noisy for sparse co-occurrence matrices. PPMI sets negatives to 0, keeping only the reliable positive associations.

**Q: What is the relationship between PMI and Word2Vec?**
A: Levy & Goldberg (2014) proved that the word vectors learned by skip-gram with negative sampling (SGNS) are implicitly factorising the **shifted PMI matrix** — specifically $\text{PMI}(w, c) - \log k$ where $k$ is the negative sample count. This means Word2Vec is approximately doing SVD on a PPMI-like matrix, but stochastically and without explicitly constructing that matrix.

**Q: Why is PMI biased toward rare words?**
A: A rare word that co-occurs once with another rare word gets a very high PMI (both $P(w)$ and $P(c)$ are tiny, so the denominator is tiny). This is noisy — one co-occurrence doesn't establish genuine semantic association. Context distribution smoothing (raising counts to $\alpha = 0.75$) partially corrects this.

---

## Common Mistakes / Gotchas

- **Confusing PMI and conditional probability**: $\text{PMI}(w,c)$ is *not* the same as $P(c|w)$. PMI is symmetric (PMI($w$,$c$) = PMI($c$,$w$)); conditional probability is not.
- **Using raw PMI in practice**: Always use PPMI. Raw PMI negative values add noise without information.
- **Ignoring window size sensitivity**: A window of ±1 captures syntactic context; ±10 captures topical context. PPMI vectors computed with different windows encode fundamentally different kinds of meaning.
- **Treating PPMI vectors as competitive with modern embeddings**: PPMI + SVD (= LSA) is a strong baseline but inferior to Word2Vec/GloVe on most benchmarks because SVD lacks the nonlinear flexibility of neural training.
- **Forgetting that PMI is corpus-dependent**: PMI associations from a news corpus differ from those from a medical corpus. Domain shift matters.

---

## Further Reading / Paper References

- Church, K. & Hanks, P. (1990). Word association norms, mutual information, and lexicography — original PMI for NLP
- Turney, P. & Pantel, P. (2010). From Frequency to Meaning: Vector Space Models of Semantics — comprehensive survey
- Levy, O. & Goldberg, Y. (2014). Neural Word Embedding as Implicit Matrix Factorization — the key paper connecting PMI to Word2Vec arxiv:1411.2738
- Jurafsky & Martin, SLP Ch. 6.4 — PMI and PPMI, definitive textbook treatment
- Bullinaria & Levy (2007). Extracting semantic representations from word co-occurrence statistics — empirical PPMI analysis
