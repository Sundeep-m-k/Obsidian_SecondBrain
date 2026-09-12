# Head-Driven Phrase Structure Grammar (HPSG)

tags: #nlp #syntax #formal-grammars #feature-structures #hpsg
links: [[Grammar Formalisms]] [[CFG and PCFG]] [[Parse Trees]] [[Constituency vs Dependency]]

---

## Definition + Intuition

**Head-Driven Phrase Structure Grammar (HPSG)** (Pollard & Sag, 1987, 1994) is a constraint-based, lexicalist grammatical framework. It represents linguistic knowledge as typed **feature structures** (attribute-value matrices) and uses **unification** as its core combinatorial operation.

Key principles:
1. **Lexicalism**: most grammatical information lives in the lexicon (lexical entries), not in phrase-structure rules
2. **Head-driven**: the syntactic/semantic properties of a phrase are largely determined by its **head** daughter
3. **Constraint-based**: grammaticality is a matter of satisfying constraints, not derivation by rules

> **Intuition**: HPSG is like a database schema for language. Every word and phrase is a record with typed fields (PHON, SYN, SEM, etc.). Combining two words/phrases is like merging two records — fields must be compatible (unification). The head daughter "controls" what the merged record looks like. This makes it very precise but also very expensive to parse.

---

## Key Properties / Types

### Feature Structures (AVMs — Attribute-Value Matrices)

Every linguistic object is an AVM:

```
Word "dog":
[PHON  ⟨dog⟩
 SYN   [HEAD  [POS   noun
               AGR   [NUM  sg
                      PER  3rd]]
        COMPS ⟨⟩]       ← takes no complements
 SEM   [INDEX s
        RESTR {dog(s)}]]
```

**Types** organize feature structures into hierarchies:
```
sign
  └── word
  └── phrase

head-feature
  └── noun
  └── verb
  └── adjective
  └── prep
```

### Core Principles

**Head Feature Principle (HFP)**: The HEAD value of a phrase equals the HEAD value of its head daughter.
→ This is why "the big [dog]_head" is a noun phrase — the head (dog) is a noun.

**Valence Principle**: Complements are selected and licensed by the head.
- A verb's COMPS list specifies what complements it needs
- When a complement is realized, it is removed from the COMPS list

**Semantics Principle**: The semantic content of a phrase is the semantic content of its head daughter (with contributions from non-head daughters).

### Unification

Combining two feature structures succeeds iff all shared attributes have compatible values:

```
[AGR [NUM sg]] ∪ [AGR [PER 3rd]] = [AGR [NUM sg, PER 3rd]]  ✓
[AGR [NUM sg]] ∪ [AGR [NUM pl]] = FAIL                       ✗ (clash)
```

This is used to enforce agreement:
- Subject must agree with verb in person and number
- HPSG enforces this by unifying the AGR feature of subject and verb

---

## Math / Formal Notation

An HPSG grammar $G$ is a tuple $(T, \mathcal{F}, \mathcal{P}, \mathcal{L})$ where:
- $T$ = type hierarchy (a partial order with inheritance)
- $\mathcal{F}$ = feature declarations (which features are appropriate for which types)
- $\mathcal{P}$ = principles (constraints on well-formed feature structures)
- $\mathcal{L}$ = lexicon (a set of lexical entries as feature structures)

**Parsing as constraint satisfaction**: a parse is a feature structure satisfying all principles and the input phonology.

**Subsumption**: $f_1 \sqsubseteq f_2$ if $f_1$ is more specific than $f_2$ (has at least all features of $f_2$, with more specific values).

**Unification**: $f_1 \sqcup f_2$ = most general structure satisfying both $f_1$ and $f_2$.
If $f_1$ and $f_2$ conflict → **unification failure** → no parse.

---

## Examples (Concrete)

### Agreement via Unification
```
Subject NP: [AGR [NUM sg, PER 3rd]]    (the dog)
Verb:        [AGR [NUM sg, PER 3rd]]    (barks — 3sg form)

Unification: [AGR [NUM sg, PER 3rd]]   ✓ → grammatical

Verb:        [AGR [NUM pl]]             (*bark — plural form)
Unification: [AGR [NUM sg, PER 3rd]] ∪ [AGR [NUM pl]] → FAIL  ✗ → ungrammatical
```

### Valence (Subcategorization) via COMPS List
```
Verb "sees":
[SYN [HEAD verb
      COMPS ⟨NP[acc]⟩]]   ← needs one accusative NP complement

Phrase "sees the cat":
  "sees" COMPS ⟨NP[acc]⟩, "the cat" = NP[acc]
  → Unification removes NP from COMPS
  → Result: [SYN [HEAD verb, COMPS ⟨⟩]]   ← VP, no remaining complements needed
```

---

## How It Connects to ML / NLP

| HPSG Concept | NLP/ML Connection |
|---|---|
| Feature structure | Structured prediction; attribute-value classifiers |
| Unification | Constraint propagation; intersection of partial information |
| Head-driven | Head word features in dependency parsing |
| Lexicalism | Morphological lexicons; subword models as implicit lexicon |
| HPSG treebanks | Training data (LinGO Redwoods, English Resource Grammar) |

**HPSG in modern NLP**: Rarely used directly — too expensive to parse and requires expert grammar engineering. But:
- The **English Resource Grammar (ERG)** is a large HPSG grammar used for deep linguistic analysis
- HPSG treebanks (LinGO Redwoods) provide rich syntactic/semantic annotation
- Feature structure ideas influenced structured prediction in ML
- Head-word features in neural parsers reflect HPSG's head-driven principle

**Why neural models won**: A large-scale HPSG grammar requires hundreds of person-years of expert development and gives coverage of ~85% of naturally occurring sentences. A neural parser trained on Penn Treebank gives ~95% accuracy on the same coverage in a fraction of the engineering time.

---

## Common Interview Questions

**Q: What is the fundamental difference between HPSG and CFG?**
A: CFG has rewriting rules that generate strings. HPSG has constraints that must be satisfied — it is declarative, not generative. HPSG uses unification to combine feature structures, which enforces agreement, subcategorization, and other dependencies naturally. CFG must add separate agreement rules or use features as extensions (X-bar theory). HPSG is more linguistically adequate; CFG is more computationally efficient.

**Q: What is the head of a phrase in HPSG?**
A: The head is the daughter that is syntactically most prominent — it determines the category of the whole phrase. In "the big dog," the head is "dog" (noun) → the phrase is a noun phrase. In "quickly ran," the head is "ran" (verb) → the phrase is a verb phrase. The Head Feature Principle propagates the head's properties to the mother node.

---

## Common Mistakes / Gotchas

- **HPSG ≠ CFG with features**: HPSG is constraint-based, not derivational. The same surface structure can satisfy HPSG constraints without any explicit "derivation."
- **Feature structures are not flat**: they are recursive (features can have feature structure values). Unification can fail at any depth.
- **Grammar engineering ≠ NLP**: HPSG grammars like the ERG are precision tools for linguistically deep analysis. They are not the right tool for high-throughput, noisy-text NLP.

---

## Further Reading / Paper References

- Pollard, C. & Sag, I.A. (1987). *Information-Based Syntax and Semantics.* CSLI Publications. — original HPSG
- Pollard, C. & Sag, I.A. (1994). *Head-Driven Phrase Structure Grammar.* University of Chicago Press.
- Copestake, A. (2002). *Implementing Typed Feature Logic.* CSLI Publications. — formal implementation
- Flickinger, D. (2000). *On Building a More Efficient Grammar by Exploiting Types.* NLE. — English Resource Grammar
- Müller, S. *HPSG — Head-Driven Phrase Structure Grammar.* (Free online textbook)
