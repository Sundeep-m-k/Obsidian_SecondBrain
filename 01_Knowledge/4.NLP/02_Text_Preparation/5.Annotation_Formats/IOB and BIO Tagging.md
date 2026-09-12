# IOB and BIO Tagging

tags: #nlp #annotation #sequence-labeling #ner #tagging-schemes
links: [[CoNLL Format]] [[BRAT Annotation]] [[NER — Named Entity Recognition]] [[POS Tagging]] [[CRF — Conditional Random Fields]]

---

## Definition + Intuition

**IOB** (Inside-Outside-Beginning) and **BIO** (Beginning-Inside-Outside) are annotation schemes for marking **spans** in text — most commonly used for Named Entity Recognition (NER), chunking, and any task where the output is a set of labeled text segments.

The problem they solve: how do you represent a multi-token span as a sequence of per-token labels?

> **Intuition**: Imagine you're highlighting named entities in a book with colored pens. For single words, one stroke does it. But for "New York City" (3 words), you need a way to say "this is the start of the entity, this is the middle, this is still the middle." IOB/BIO is that notation — it encodes span boundaries as token-level labels that a sequence classifier can predict one token at a time.

---

## Key Properties / Types

### The Tag Set

**BIO (= IOB2) — most common:**
| Tag | Meaning |
|-----|---------|
| `B-TYPE` | **B**eginning of an entity of type TYPE |
| `I-TYPE` | **I**nside (continuation of) an entity of type TYPE |
| `O` | **O**utside — not part of any entity |

**IOB1 (original IOB):**
| Tag | Meaning |
|-----|---------|
| `I-TYPE` | Inside an entity (includes first token, unless preceded by same type) |
| `B-TYPE` | **B**eginning — only used when two adjacent entities have the same type |
| `O` | Outside |

**BIOES / BILOU — extended schemes:**
| Tag | Meaning |
|-----|---------|
| `B-TYPE` | Beginning of multi-token entity |
| `I-TYPE` | Inside multi-token entity |
| `O` | Outside |
| `E-TYPE` | **E**nd (last token) of multi-token entity |
| `S-TYPE` | **S**ingle-token entity |

BIOES provides more information to the classifier — the model knows whether it's predicting the end of a span (E) or a standalone entity (S) vs a continuation (I), which aids learning.

**BILOU** is equivalent to BIOES with different letters (L = Last, U = Unit).

### Scheme Comparison

```
Text:    "Barack  Obama  visited  New   York   City  yesterday"
BIO:      B-PER   I-PER    O      B-LOC I-LOC  I-LOC    O
IOB1:     I-PER   I-PER    O      I-LOC I-LOC  I-LOC    O
BIOES:    B-PER   E-PER    O      B-LOC I-LOC  E-LOC    O
```

**Adjacent same-type entities** — where IOB1 vs BIO matters:
```
Text:    "London   Paris   are  cities"
          (LOC)   (LOC)

BIO:      B-LOC   B-LOC    O     O      ← always uses B at start → unambiguous
IOB1:     I-LOC   B-LOC    O     O      ← B only when same type is adjacent
```

IOB1 is ambiguous for the first token: `I-LOC` at start can mean either "single entity" or "beginning of multi-token entity." BIO (IOB2) resolves this by always using `B` at the start of an entity.

---

## Math / Formal Notation

**NER as sequence labeling:**

Given input $\mathbf{x} = x_1, x_2, \ldots, x_n$ (token sequence), predict label sequence:

$$\mathbf{y} = y_1, y_2, \ldots, y_n, \quad y_i \in \mathcal{T}$$

where $\mathcal{T}$ is the BIO tag set: $\{O, B\text{-PER}, I\text{-PER}, B\text{-LOC}, I\text{-LOC}, B\text{-ORG}, I\text{-ORG}, \ldots\}$

**Tag set size**: for $K$ entity types: $|\mathcal{T}| = 2K + 1$ (BIO) or $4K + 1$ (BIOES).

**Valid vs invalid tag transitions** — not all transitions are legal in BIO:
- `O → I-PER`: ❌ invalid (can't start an I-tag without B-tag)
- `B-PER → I-LOC`: ❌ invalid (type mismatch)
- `B-PER → I-PER`: ✅ valid
- `I-PER → B-LOC`: ✅ valid (end PER, start new LOC)

**CRF transition scores** capture these constraints as learned parameters $T_{ij}$ (transition from tag $i$ to tag $j$). Invalid transitions get very negative scores during training.

**Viterbi decoding** finds the most probable valid tag sequence:
$$\mathbf{y}^* = \arg\max_{\mathbf{y}} \sum_{t=1}^{n} s(x_t, y_t) + \sum_{t=2}^{n} T_{y_{t-1}, y_t}$$

where $s(x_t, y_t)$ is the emission score from the encoder (BERT, BiLSTM, etc.).

**Span extraction from BIO tags:**
```python
def extract_spans(tokens, tags):
    spans = []
    current = None
    for i, (tok, tag) in enumerate(zip(tokens, tags)):
        if tag.startswith("B-"):
            if current: spans.append(current)
            current = {"type": tag[2:], "start": i, "tokens": [tok]}
        elif tag.startswith("I-") and current and tag[2:] == current["type"]:
            current["tokens"].append(tok)
        else:
            if current: spans.append(current)
            current = None
    if current: spans.append(current)
    return spans
```

---

## Examples (Concrete)

### CoNLL-2003 NER (standard benchmark)

```
Word        POS   Chunk  NER
EU          NNP   B-NP   B-ORG
rejects     VBZ   B-VP   O
German      JJ    B-NP   B-MISC
call        NN    I-NP   O
to          TO    B-VP   O
boycott     VB    I-VP   O
British     JJ    B-NP   B-MISC
lamb        NN    I-NP   O
.           .     O      O
```

Entity spans extracted:
- `EU` → ORG (single token → B-ORG, no I needed)
- `German` → MISC
- `British` → MISC

### Nested entity problem (IOB limitation)

Standard BIO cannot represent nested entities:
```
"[Bank of [America]_ORG]_ORG announced..."
```
For nested NER, you need: (1) multiple annotation layers, (2) a span-based model, or (3) a special nested tagging scheme (e.g., stack-based).

### BIOES advantage for training

BIOES gives the model clearer signal:
```
"New   York   City"
B-LOC  I-LOC  E-LOC   ← model explicitly knows "City" ends the span

vs BIO:
B-LOC  I-LOC  I-LOC   ← model must infer span ends when non-I tag follows
```

Empirically, BIOES improves NER F1 by ~1–2 points over BIO by making span boundaries more explicit.

---

## How It Connects to ML / NLP

| IOB/BIO Concept | ML/NLP Application |
|---|---|
| Token-level label sequence | Any sequence labeling task: NER, chunking, SRL, slot filling |
| Valid tag transitions | CRF transition matrix; constrained Viterbi decoding |
| Span extraction from tags | Post-processing step in all NER pipelines |
| Tag set size | Output dimensionality for final classification layer |
| BIOES vs BIO | Architectural choice; affects training signal clarity |

**Pipeline in modern NER:**
```
Raw text
  → Tokenizer (WordPiece / BPE)
  → BERT encoder → contextual embeddings [h₁, h₂, ..., hₙ]
  → Linear projection → per-token logits over BIO tags
  → (Optional) CRF layer for valid transition constraints
  → Viterbi decoding → BIO tag sequence
  → Span extraction → [(Barack Obama, PER), (New York City, LOC)]
```

**Subword tokenization + BIO alignment issue:**
BERT tokenizes "Obama" as one token, but "Schwarzenegger" as ["Sch", "##war", "##zen", "##egger"]. BIO labels are word-level but BERT operates on subword tokens. Solutions:
1. Label only the first subword of each word (`B-PER` on "Sch", `X` on "##war" etc.)
2. Average subword representations before BIO classification
3. Use a span-based model that bypasses per-token labeling

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — BIO tagging = multiclass classification per token
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — cross-entropy over BIO tag distribution
- [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — training the token classifier end-to-end

---

## Common Interview Questions

**Q: Why use BIO tagging instead of directly predicting spans (start, end, type)?**
A: BIO tagging is a sequence labeling problem — every token gets a label, and the labels can be predicted left-to-right with shared parameters. Span-based models (predict all (start, end, type) triples) are more powerful (can handle nested entities, don't suffer from label propagation errors) but require scoring $O(n^2)$ candidate spans. BIO is $O(n)$ and integrates naturally with sequence models (LSTM, BERT).

**Q: What is the difference between IOB1 and IOB2 (BIO)?**
A: In IOB1, `B-TYPE` is only used when two adjacent entities of the same type follow each other — the first token of an entity otherwise gets `I-TYPE`. In IOB2 (BIO), `B-TYPE` is *always* used for the first token of any entity. IOB2 is unambiguous and is the modern standard; IOB1 is a historical artifact but still appears in older datasets (original CoNLL-2000 chunking).

**Q: How do you handle BIO tagging when using subword tokenizers like BERT?**
A: The most common approach is to assign the BIO label only to the first subword of each word and use a special `X` or ignore label for subsequent subwords. Only the first subword's representation is used for classification. An alternative is to use a span-based model that operates at the word level after pooling subword representations, bypassing the alignment problem entirely.

---

## Common Mistakes / Gotchas

- **`I` without preceding `B`**: During inference, a model might predict `I-PER` at the start of a sentence — this is an invalid transition. Always apply either: (1) a CRF layer to enforce validity, or (2) post-processing that converts invalid `I` tags to `B`.
- **Type mismatch transitions**: `B-PER` followed by `I-LOC` is invalid. Models without CRF layers can produce these. Post-process by either re-labeling the `I` tag to match the preceding `B`, or treating the `I` as the start of a new entity.
- **Comparing systems using different schemes**: F1 scores are not directly comparable between systems using BIO vs BIOES vs IOB1. Always check which scheme the evaluation script expects.
- **Subword → word label alignment**: Forgetting to handle subword tokens in BERT-based NER is a very common bug. Verify your tokenizer outputs word IDs so you can map subword predictions back to words.

---

## Further Reading / Paper References

- Ramshaw, L.A. & Marcus, M.P. (1995). *Text Chunking Using Transformation-Based Learning.* — original IOB paper
- Tjong Kim Sang, E.F. & De Meulder, F. (2003). *Introduction to the CoNLL-2003 Shared Task: Language-Independent Named Entity Recognition.* — BIO standard for NER
- Lafferty, J., McCallum, A. & Pereira, F. (2001). *Conditional Random Fields: Probabilistic Models for Segmenting and Labeling Sequence Data.* — CRF + BIO
- Devlin, J. et al. (2019). *BERT: Pre-training of Deep Bidirectional Transformers.* [[arxiv:1810.04805]] — BERT NER with BIO
- Jurafsky & Martin, *Speech and Language Processing* Ch. 8 — sequence labeling
