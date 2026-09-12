# Word Sense and Polysemy

tags: #nlp #semantics #wsd #polysemy #lexical-ambiguity
links: [[Lexical Semantics]] [[Compositional Semantics]] [[BERT Embeddings]] [[Sentence Transformers]]

---

## Definition + Intuition

**Polysemy**: a single word form has multiple *related* senses. The senses share a common historical or metaphorical origin.

**Homonymy**: a single word form has multiple *unrelated* senses (coincidental phonological overlap).

**Word Sense Disambiguation (WSD)**: the NLP task of determining which sense of a word is intended in a given context.

> **Intuition**:
> - `mouth` — body part / river outlet / opening of a container. These senses are *related* (all "opening" metaphors) → polysemy.
> - `bank` — financial institution / river bank. These senses are historically *unrelated* (different etymological origins) → homonymy.
>
> In practice, the computational distinction rarely matters — both require context to resolve. WSD is essentially: "given this word in this context, which dictionary entry applies?"

---

## Key Properties / Types

### Types of Polysemy

**Systematic polysemy** — predictable, productive sense extensions:
| Pattern | Example |
|---------|---------|
| Animal → food | `chicken` (animal / food) |
| Container → contents | `glass` (drinking vessel / glass substance) |
| Activity → result | `construction` (the building process / the building) |
| Place → institution | `school` (physical place / institution) |
| Plant → fruit | `apple` (tree / fruit) |

**Metonymic extension**: meaning shifts by conceptual contiguity:
- `The kettle is boiling` (water inside the kettle, not the kettle itself)
- `I read Orwell` (works by Orwell, not the person)

**Metaphorical extension**: meaning shifts by structural analogy:
- `foot` of a mountain (bottom, by analogy with body)
- `head` of a company
- `leg` of a journey

### WordNet Senses
WordNet organizes words into **synsets** (sets of synonyms) representing distinct senses.

Example entry for "bank":
```
bank.n.01: a financial institution...
bank.n.02: sloping land beside a body of water...
bank.n.03: a supply or stock held in reserve...
bank.n.04: the funds held by a gambling establishment...
bank.n.05: a flight maneuver...
bank.v.01: tip sideways when banking...
bank.v.02: be in the banking business...
```

Each sense has: definition, example sentences, hyponyms, hypernyms, related synsets.

### WSD Approaches

| Approach | Description | Strength |
|----------|-------------|---------|
| **Dictionary-based** | Match context words to WordNet glosses (Lesk algorithm) | No training data needed |
| **Knowledge-based** | Use WordNet relations + graph algorithms | Interpretable |
| **Supervised** | Classify sense from labeled context windows | High accuracy (but needs sense-labeled data) |
| **Contextual embeddings** | Different BERT vectors per occurrence → nearest sense embedding | State-of-the-art, no labeled data |

---

## Math / Formal Notation

### Lesk Algorithm (Dictionary-Based WSD)

For word $w$ in context $C$, and candidate senses $s_1, \ldots, s_n$:

$$\hat{s} = \arg\max_{s_i} |\text{gloss}(s_i) \cap C|$$

where $\text{gloss}(s_i)$ is the set of words in the WordNet definition of sense $s_i$, and $C$ is the set of context words.

**Extended Lesk**: also includes glosses of related synsets (hypernyms, hyponyms) in the gloss set.

### Supervised WSD

Treat WSD as $K$-way classification over senses of target word $w$:

Given: sentence $w_1, \ldots, w_i, \ldots, w_n$ where $w_i$ is the target.
Features: context window $[w_{i-k}, \ldots, w_{i-1}, w_{i+1}, \ldots, w_{i+k}]$ (as BoW or embeddings).

$$P(s \mid \text{context}) = \text{softmax}(W \cdot \vec{h}_i + b)$$

where $\vec{h}_i$ is the contextual representation of position $i$.

**Cross-entropy training loss**:
$$\mathcal{L} = -\sum_{(c, s^*) \in \mathcal{D}} \log P(s^* \mid c)$$

This is exactly [[Logistic Loss]] extended to $K$ classes.

### Sense Embedding WSD

BERT gives a different vector $\vec{h}_i^{(\ell)}$ for each occurrence of word $w_i$. For WSD:

1. For each WordNet sense $s_k$, compute a **sense prototype** $\vec{p}_k$ = mean BERT vector over all training instances of that sense.
2. For a new occurrence, embed → compare to prototypes:
$$\hat{s} = \arg\max_{k} \cos(\vec{h}_i, \vec{p}_k)$$

This is **nearest-neighbor WSD** in embedding space — no training beyond the embedding model.

---

## Examples (Concrete)

### Disambiguating "plant"

```
Context 1: "The factory is a large manufacturing plant."
  → plant.n.04 (industrial facility)

Context 2: "She watered the plant on the windowsill."
  → plant.n.01 (living organism)

Context 3: "He will plant the seeds tomorrow."
  → plant.v.01 (put in the ground to grow)

Context 4: "They will plant evidence at the scene."
  → plant.v.02 (place secretly for deceptive purpose)
```

### The "hard" WSD cases
```
"interest"
  → financial interest (interest rate)
  → curiosity/attention (interested in)
  → stake/share (has an interest in the company)
  → legal interest (has an interest in the estate)
All related but distinct → systematic polysemy
```

```
"light"
  → electromagnetic radiation
  → not heavy
  → pale in color
  → not serious ("light reading")
  → to ignite ("light a match")
Very broad polysemy with both metonymic and metaphorical extensions
```

### BERT vs Word2Vec on polysemy

```python
# Word2Vec: one vector per word
word2vec["bank"] = [0.3, -0.1, 0.8, ...]  # "average" of all senses

# BERT: different vector per occurrence
bert_encode("I bank at Wells Fargo")[3]     = [0.2, 0.5, -0.3, ...]   # financial sense
bert_encode("The river bank flooded")[3]    = [-0.4, 0.1, 0.9, ...]   # geographical sense
# Cosine similarity between these: ~0.3 (very different)
# They cluster with other financial/geographical words respectively
```

---

## How It Connects to ML / NLP

| WSD Concept | NLP/ML Application |
|---|---|
| Sense disambiguation | IE, QA, MT (different senses translate differently) |
| Polysemy | Motivation for contextual vs static embeddings |
| WordNet senses | Knowledge source for downstream tasks |
| Lesk algorithm | Baseline for WSD; explainable system |
| Sense-level embeddings | WSD evaluation for contextual models; semantic change detection |

**Why polysemy matters for MT:**
```
"bank" in English → "Банк" (financial, Russian) or "берег" (river bank, Russian)
The correct translation requires WSD!
```

Statistical MT aligned `bank` with both Russian words; neural MT with contextual encoders handles this implicitly via attention.

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — WSD as multiclass classification
- [[3.ML & DL/1.Concepts/6.Features and Representation/Data Representation.md]] — contextual vs static representations
- [[3.ML & DL/1.Concepts/8.Model Behavior/Underfitting.md]] — static embeddings underfit polysemous words

---

## Common Interview Questions

**Q: How do contextualized embeddings (BERT) solve the polysemy problem?**
A: Static embeddings (Word2Vec, GloVe) assign one vector per word type — a single "bank" vector that is an average over all senses. BERT produces a different vector for each *occurrence* of a word, conditioned on the full sentence context. The vector for "bank" in a financial context clusters with financial terms; in a geographical context it clusters with water/land terms. This naturally separates word senses without any explicit WSD step.

**Q: What is the WSD "knowledge acquisition bottleneck"?**
A: High-accuracy WSD requires sense-annotated training data — labeled examples of each word in each of its senses. Creating such data (SemCor, OntoNotes sense annotations) is expensive and requires linguistic expertise. Furthermore, WordNet sense distinctions are sometimes extremely fine-grained, making annotation unreliable. This bottleneck is why BERT-based sense embeddings (no training data needed) are attractive.

**Q: What is word sense induction vs word sense disambiguation?**
A: WSD assumes a fixed sense inventory (WordNet) and assigns occurrences to known senses. WSI induces the sense inventory from data — it clusters occurrences into groups without a predefined list of senses. WSI is useful when WordNet senses are too fine-grained or when senses are domain-specific (e.g. medical senses of "discharge").

---

## Common Mistakes / Gotchas

- **Treating all polysemy as needing explicit disambiguation**: in most modern NLP, BERT handles polysemy implicitly through contextual representations. Explicit WSD is only needed when interpretability or a specific sense-level resource (WordNet) is required.
- **Over-relying on WordNet granularity**: WordNet sense distinctions can be extremely fine-grained (e.g. 3 senses of "house" that humans consistently confuse). Inter-annotator agreement on fine-grained WSD is often ~70–80% — not all sense distinctions are meaningful for downstream tasks.
- **Ignoring zero-shot polysemy in new domains**: a model trained on general text may not know that "discharge" in medical text refers to releasing a patient, not electrical discharge.
- **Confusing polysemy and vagueness**: vague words don't have discrete senses — "tall" is just underspecified for a particular height. Polysemous words have genuinely distinct, listable senses.

---

## Further Reading / Paper References

- Lesk, M. (1986). *Automatic Sense Disambiguation Using Machine Readable Dictionaries.* ACL.
- Yarowsky, D. (1995). *Unsupervised Word Sense Disambiguation Rivaling Supervised Methods.* ACL. — bootstrapping WSD
- Navigli, R. (2009). *Word Sense Disambiguation: A Survey.* ACM Computing Surveys.
- Miller, G.A. (1995). *WordNet: A Lexical Database for English.* CACM.
- Devlin, J. et al. (2019). *BERT: Pre-training of Deep Bidirectional Transformers.* [[arxiv:1810.04805]] — implicit WSD via contextual embeddings
- Loureiro, D. & Jorge, A. (2019). *Language Modelling Makes Sense: Propagating Representations through WordNet for Full-Coverage Word Sense Disambiguation.* ACL.
