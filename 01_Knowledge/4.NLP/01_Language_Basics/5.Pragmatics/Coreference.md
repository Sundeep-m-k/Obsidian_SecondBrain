# Coreference

tags: #nlp #pragmatics #coreference #discourse #entity-tracking
links: [[Discourse Structure]] [[Speech Acts]] [[Coreference Resolution]] [[NER — Named Entity Recognition]]

---

## Definition + Intuition

**Coreference** is the linguistic phenomenon where two or more expressions refer to the same real-world entity. The expressions that corefer are called **coreferential mentions**; the linked set of mentions is called a **coreference chain** or **entity cluster**.

**Coreference resolution**: the NLP task of finding all mentions in a text and grouping them into coreference chains (one chain per entity).

> **Intuition**: In "**Mary** told **her** friend that **she** had won the award," the reader needs to figure out who "her" and "she" refer to. Both likely refer to Mary. This is coreference — linking pronouns and definite descriptions back to the entities they name. Without it, machines can't track who did what across a document.

---

## Key Properties / Types

### Types of Coreference Expressions

| Type | Example | Notes |
|------|---------|-------|
| **Named entity** | "Barack Obama" | Proper noun; canonical mention |
| **Pronoun** | "he", "she", "it", "they" | Require antecedent resolution |
| **Definite NP** | "the president", "the man" | Require world knowledge |
| **Demonstrative** | "this company", "that report" | Proximity-sensitive |
| **Zero pronoun** | (pro-drop languages) | "Went to the store" → subject dropped |

### Coreference vs Related Phenomena

**Anaphora**: any expression whose interpretation depends on another expression (antecedent). Coreference is a type of anaphora where the anaphor and antecedent refer to the *same entity*.

**Cataphora**: anaphor precedes its antecedent: "**Before she** arrived, **Mary** called ahead."

**Bridging**: implicit reference via encyclopedic knowledge: "I bought a car. **The engine** was faulty." — "the engine" is understood as the engine of *that car* via bridging inference.

**Split antecedents**: "**John** met **Mary**. **They** went for coffee." — "they" refers to the set {John, Mary}.

### Coreference Chains Example
```
[Alice]₁ called [her]₁ sister [Kate]₂. 
[Kate]₂ said [she]₂ couldn't make it. 
[She]₂ had a meeting with [her]₂ boss.
[Alice]₁ was disappointed.

Chain 1: Alice, her(line 1), She(line 4) = Alice
Chain 2: Kate, Kate, she(line 2), she(line 3), her(line 3) = Kate
```

---

## Math / Formal Notation

Modern neural coreference resolution (Lee et al. 2017, 2018 — span-based model):

**Step 1**: Generate candidate mention spans $\mathcal{M} = \{(i, j) : 1 \leq i \leq j \leq n\}$

**Step 2**: Score each span as a mention:
$$s_m(i, j) = w_m \cdot \vec{g}(i, j)$$
where $\vec{g}(i, j)$ is a learned representation of span $[i, j]$ from BERT.

**Step 3**: For each candidate mention pair, score coreference:
$$s_c(i, j, k, l) = s_m(i,j) + s_m(k,l) + s_a(i,j,k,l)$$
where $s_a$ is the antecedent score.

**Step 4**: Training objective — maximizes likelihood of gold coreference clusters.

**Span representation**: typically:
$$\vec{g}(i,j) = [\vec{h}_i^{start}; \vec{h}_j^{end}; \hat{h}_{ij}^{attn}; \phi(i,j)]$$
- $\vec{h}_i^{start}$: BERT embedding at start of span
- $\vec{h}_j^{end}$: BERT embedding at end
- $\hat{h}_{ij}^{attn}$: attention-weighted span head
- $\phi(i,j)$: span width feature

---

## Examples (Concrete)

**Challenging cases:**

```
"The trophy doesn't fit in the suitcase because it is too small."
→ "it" = the suitcase (it's too small to fit the trophy)
Winograd Schema: requires world knowledge about physical containment.

"The town council refused the protesters a permit 
because they feared violence."
→ "they" = the town council? OR the protesters?
→ "they feared violence" → council is more likely afraid
Winograd Schema: requires reasoning about who fears whom.
```

**Gender-neutral and singular "they":**
```
"A customer called. They said the product was broken."
→ "They" = a customer (singular, gender-neutral)
Many coreference systems fail here — assume "they" is plural.
```

---

## How It Connects to ML / NLP

| Coreference Concept | NLP/ML Application |
|---|---|
| Mention detection | NER + span detection (classification) |
| Coreference clustering | Clustering problem; entity linking |
| Pronoun resolution | Foundation for reading comprehension, IE |
| Winograd schemas | Commonsense reasoning benchmark; LLM evaluation |
| Cross-sentence entity tracking | Multi-hop QA; event extraction |

**Coreference in downstream tasks:**
- **Information extraction**: "Tesla announced earnings. **The company** reported $25B in revenue." — coreference needed to attribute the report to Tesla.
- **Summarization**: Pronoun replacement — "Alice met Bob. **She** gave **him** a gift." needs coreference to generate "Alice gave Bob a gift."
- **QA**: Multi-hop questions: "Who is the CEO of the company that makes the Model 3?" — needs to link Model 3 → Tesla → CEO.

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — mention detection and coreference scoring as classification
- [[3.ML & DL/1.Concepts/1.Foundations/Model.md]] — span-based neural coreference model

---

## Common Interview Questions

**Q: What is the Winograd Schema Challenge and why is it hard?**
A: Winograd schemas are minimal sentence pairs where pronoun resolution requires commonsense world knowledge, not just syntactic heuristics. Example: "The trophy doesn't fit in the suitcase because **it** is too large/small." — changing one word flips which noun "it" refers to. Resolving these requires understanding physical constraints, social norms, or causal reasoning — not statistical patterns.

**Q: How do neural models solve coreference resolution?**
A: Modern systems (Lee et al. 2017+) use a span-based approach: enumerate all possible mention spans, score each pair for coreference using BERT representations, then build coreference chains. Training uses gold mention clusters from OntoNotes. The key insight is representing arbitrary spans via BERT embeddings and training end-to-end.

---

## Common Mistakes / Gotchas

- **Treating all pronouns as referring to the nearest NP**: English syntax allows this heuristic sometimes, but it fails on Winograd schemas, split antecedents, and cataphora.
- **Ignoring zero pronouns in pro-drop languages**: Spanish, Italian, Japanese, Chinese all allow subject omission. English-centric coreference models completely miss these cases in multilingual settings.
- **Conflating coreference and entity linking**: Coreference clusters mentions *within a document*. Entity linking maps mentions to a knowledge base (Wikidata, Freebase). They are different tasks, often done in sequence.

---

## Further Reading / Paper References

- Lee, K. et al. (2017). *End-to-end Neural Coreference Resolution.* EMNLP. [[arxiv:1707.07045]]
- Lee, K. et al. (2018). *Higher-Order Coreference Resolution with Coarse-to-Fine Inference.* NAACL. [[arxiv:1804.05392]]
- Levesque, H. et al. (2012). *The Winograd Schema Challenge.* KR.
- Jurafsky & Martin, *Speech and Language Processing* Ch. 22 — coreference
- OntoNotes corpus: https://catalog.ldc.upenn.edu/LDC2013T19
