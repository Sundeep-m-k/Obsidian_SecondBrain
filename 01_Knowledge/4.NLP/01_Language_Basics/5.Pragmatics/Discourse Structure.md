# Discourse Structure

tags: #nlp #pragmatics #discourse #coherence #rsa
links: [[Speech Acts]] [[Implicature and Grice]] [[Coreference]] [[Abstractive Summarization]] [[Discourse Parsing]]

---

## Definition + Intuition

**Discourse** refers to language above the sentence level — how sentences connect into coherent paragraphs, conversations, and texts. **Discourse structure** is the set of relations and organization patterns that make a text hang together as a coherent whole.

> **Intuition**: Compare:
> (A) "John fell. Mary laughed."
> (B) "John fell because he slipped. Mary laughed, which made him feel embarrassed."
>
> (A) is grammatical but leaves the connection implicit. (B) makes the relations explicit (`because`, `which`). But even (A) is interpreted coherently — readers automatically infer the causal and temporal links. Discourse structure is what makes this possible.

---

## Key Properties

### Coherence Relations (RST)

**Rhetorical Structure Theory (RST, Mann & Thompson 1988)** represents text as a tree of **nucleus–satellite** pairs connected by rhetorical relations:

| Relation | Description | Signal |
|----------|-------------|--------|
| **Elaboration** | Satellite adds detail to nucleus | — |
| **Cause** | Satellite is cause/reason for nucleus | because, since, therefore |
| **Contrast** | Nuclei contrast with each other | but, however, while |
| **Evidence** | Satellite provides evidence for nucleus | — |
| **Condition** | Satellite is condition for nucleus | if, unless |
| **Concession** | Satellite concedes something, nucleus remains | although, despite |
| **Background** | Satellite provides context for nucleus | — |
| **Temporal** | Temporal relation between spans | then, after, before |

### Discourse Connectives
Explicit markers of discourse relations:
```
Causal:     because, since, therefore, thus, consequently
Contrastive: but, however, although, yet, while, whereas
Temporal:   then, next, after, before, when, while
Additive:   and, also, furthermore, in addition, moreover
```

When absent (implicit relations), models must infer the relation — much harder.

### Entity-Based Coherence (Centering Theory)
Coherence is maintained by tracking **discourse entities** across sentences.

**Centering** captures salience of entities:
- **Backward-looking center (Cb)**: most salient entity from previous utterance
- **Forward-looking centers (Cf)**: entities evoked in current utterance, ranked by grammatical role (subject > object > other)

A text is coherent if entities are mentioned consistently and transitions are smooth.

---

## Math / Formal Notation

**RST parsing** as span-based classification:

Given text spans $[i, j]$ and $[j+1, k]$, predict:
1. Whether a relation exists
2. The relation type $r \in \mathcal{R}$
3. Which span is nucleus, which is satellite

$$P(r, \text{nuclearity} \mid \text{span}_1, \text{span}_2) = \text{softmax}(W \cdot [\vec{h}_1; \vec{h}_2])$$

**Coherence scoring** as pointwise or listwise ranking:

$$P(\text{coherent} \mid \text{text}) = \sigma(f(\text{text}))$$

where $f$ can be a neural model that reads the full text. Coherence models are trained by permuting sentences in real texts as negative examples.

---

## Examples (Concrete)

```
Text: "Max fell. He was angry. The floor was slippery."
                                                     
RST tree:
         [ROOT: Elaboration]
        /                    \
  [Max fell]      [Elaboration: Why he fell / how he felt]
                  /                          \
          [He was angry]          [The floor was slippery]
                                       [Cause for Max fell]
```

Implicit relations: "Max fell **because** the floor was slippery" and "**As a result**, he was angry."

---

## How It Connects to ML / NLP

| Discourse Concept | NLP/ML Application |
|---|---|
| RST relations | Automatic discourse parsing; summarization (nuclei = important) |
| Coherence | Coherence scoring for NLG evaluation; essay scoring |
| Entity tracking | Coreference resolution; pronoun resolution |
| Discourse connectives | Discourse relation classification; implicit connective prediction |

**Summarization connection**: In RST trees, **nuclei** carry the essential information; **satellites** add elaboration. Extractive summarization can select nucleus sentences from the RST tree — this is a coherence-informed approach to summarization.

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — discourse relation classification
- [[Abstractive Summarization]] — RST-guided extractive methods

---

## Further Reading

- Mann, W.C. & Thompson, S.A. (1988). *Rhetorical Structure Theory: Toward a Functional Theory of Text Organization.* Text.
- Grosz, B.J. & Sidner, C.L. (1986). *Attention, Intentions, and the Structure of Discourse.* Computational Linguistics.
- Jurafsky & Martin, *Speech and Language Processing* Ch. 22 — discourse
