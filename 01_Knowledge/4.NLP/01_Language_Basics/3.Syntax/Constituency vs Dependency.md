# Constituency vs Dependency

tags: #nlp #syntax #parsing #syntax-trees #dependency-parsing
links: [[Parse Trees]] [[Grammar Formalisms]] [[CFG and PCFG]] [[Constituency Parsing]] [[Dependency Parsing]] [[Semantic Role Labeling]]

---

## Definition + Intuition

There are two major frameworks for representing the **syntactic structure** of a sentence:

**Constituency (Phrase Structure)**: the sentence is a hierarchy of *phrases* (constituents). Words group into phrases, phrases group into larger phrases, up to a full sentence. Captures *what groups together*.

**Dependency**: the structure is a set of *directed binary relations* between individual words. One word (the **head**) governs another (the **dependent**). Captures *who depends on whom*.

> **Intuition**:
> Consider "The big dog bit the cat."
>
> **Constituency** asks: what are the chunks? → `[The big dog]_NP [bit [the cat]_NP]_VP` — it nests groups.
>
> **Dependency** asks: what word governs what? → `bit` is the root; `dog` is the subject of `bit`; `cat` is the object of `bit`; `big` modifies `dog`; `the` modifies `dog` and `cat`. It's a labeled arrow graph.

---

## Key Properties / Types

### Constituency Structure

A constituency tree is a **rooted, labeled tree** where:
- **Leaves** are words (terminal nodes)
- **Internal nodes** are phrase labels: `NP` (noun phrase), `VP` (verb phrase), `S` (sentence), `PP` (prepositional phrase), `ADJP`, etc.
- **Dominance**: a node dominates all nodes in its subtree
- **Constituency test**: a string is a constituent if it can be replaced by a pronoun, moved, or used as the answer to a question

**Phrase labels:**

| Label | Name | Example |
|-------|------|---------|
| `S` | Sentence | Full clause |
| `NP` | Noun Phrase | "the big dog" |
| `VP` | Verb Phrase | "bit the cat" |
| `PP` | Prepositional Phrase | "on the table" |
| `ADJP` | Adjective Phrase | "very happy" |
| `ADVP` | Adverb Phrase | "quite slowly" |

### Dependency Structure

A dependency tree is a **rooted, directed, labeled graph** where:
- Each word is a node
- Each arc is a **directed relation** from head → dependent
- The **root** of the tree is the main predicate (usually the verb)
- The arc label is a **grammatical relation**

**Universal Dependencies (UD) relation labels:**

| Label | Name | Example |
|-------|------|---------|
| `nsubj` | Nominal subject | dog ←nsubj— bit |
| `obj` | Object | bit —obj→ cat |
| `det` | Determiner | dog ←det— the |
| `amod` | Adjectival modifier | dog ←amod— big |
| `nmod` | Nominal modifier | cat ←nmod— street |
| `advmod` | Adverbial modifier | ran ←advmod— fast |
| `aux` | Auxiliary | running ←aux— is |
| `root` | Root | ROOT ←root— bit |

---

## Math / Formal Notation

### Constituency

A **context-free grammar** (CFG) defines constituency structures:

$$G = (N, \Sigma, R, S)$$

where $N$ = nonterminals (phrase labels), $\Sigma$ = terminals (words), $R$ = production rules, $S$ = start symbol.

Productions: $A \rightarrow \alpha$ where $A \in N$, $\alpha \in (N \cup \Sigma)^*$

Example rules:
$$S \rightarrow NP\ VP$$
$$NP \rightarrow DT\ JJ\ NN$$
$$VP \rightarrow VBD\ NP$$
$$NP \rightarrow DT\ NN$$

**A parse** is a derivation tree showing how $S$ generates the sentence via these rules.

**PCFG** adds probabilities:
$$P(\text{tree}) = \prod_{r \in \text{tree}} P(r)$$

### Dependency

A dependency structure is a directed graph $D = (V, A)$ where $V$ = words and $A \subseteq V \times V \times L$ ($L$ = arc labels).

**Well-formedness constraints:**
1. **Single head**: each word has at most one head
2. **Acyclicity**: no directed cycles
3. **Connectedness**: the structure forms a tree
4. **Projectivity** (optional): no crossing arcs when words are in linear order

**Projectivity**: a dependency arc $(h, d)$ is projective if for all words $w$ between $h$ and $d$, $w$ is reachable from $h$. Most natural language is mostly projective, but non-projective constructions exist ("Which book did you say you read?").

**Transition-based dependency parsing** (arc-standard system):
State: `(stack, buffer, arcs)`
Actions:
- `SHIFT`: move word from buffer to stack
- `LEFT-ARC(r)`: add arc `(buffer[0], stack[top])` with label `r`; pop stack
- `RIGHT-ARC(r)`: add arc `(stack[top], buffer[0])` with label `r`; pop stack

At each step, a classifier predicts which action to take — a multiclass classification problem: [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]]

---

## Examples (Concrete)

**Sentence**: "The cat sat on the mat."

### Constituency Tree
```
           S
         /   \
       NP     VP
      / \    /  \
    DT  NN  VBD  PP
    |   |   |   /  \
   The cat sat  P   NP
               |   / \
              on  DT  NN
                  |   |
                 the  mat
```

### Dependency Tree
```
ROOT
  └── sat (root)
       ├── cat (nsubj)
       │    └── The (det)
       └── on (obl)
            └── mat (nmod)
                 └── the (det)
```

**Same sentence, two completely different representations.** Note:
- Constituency shows `[The cat]_NP` as a unit — useful for knowing what is a "chunk"
- Dependency directly says `cat` is the subject of `sat` — useful for knowing who did what

**Comparing for MT:**

English: "He gave her the book"
```
Dependency: gave → her (iobj), gave → book (obj)
```

Japanese: "彼は 彼女に 本を あげた"
```
Dependency: あげた → 彼 (nsubj), あげた → 彼女 (iobj), あげた → 本 (obj)
```

Dependency structure is more cross-linguistically consistent than constituency — heads are the same even when word order changes. This is why dependency parses are used in cross-lingual NLP.

---

## How It Connects to ML / NLP

| Framework | NLP/ML Applications |
|-----------|-------------------|
| **Constituency** | Syntactic parsing, grammar checking, syntactic features for classification |
| **Dependency** | SRL, RE, QA, cross-lingual NLP, information extraction |
| **Both** | Pretraining objectives (syntax-aware BERT), interpretability |

**Practical usage:**
- **spaCy** produces dependency parses (not constituency trees) by default
- **Stanford Parser** produces constituency trees (and can convert to dependencies)
- **Stanford CoreNLP** provides both
- BERT implicitly encodes some syntactic structure — probing shows dependency relations are recoverable from BERT representations

**In NLP pipelines:**
Dependency parses are used for:
- Subject-verb-object extraction → [[Relation Extraction]]
- Named entity classification (syntactic context of an NE)
- Feature extraction for [[Semantic Role Labeling]]

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — transition-based parsing as classification
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — CRF loss for graph-structured prediction

---

## Common Interview Questions

**Q: What is the key practical difference between constituency and dependency parsing for downstream NLP tasks?**
A: Dependency parsing directly exposes grammatical relations (subject, object, modifier) between specific words — immediately useful for information extraction, SRL, and RE. Constituency parsing exposes phrasal groups — useful for syntax-sensitive tasks like coreference and constituency-based grammar. In practice, dependency parsing is more widely used in modern NLP pipelines because it is faster (linear or near-linear algorithms) and more directly interpretable.

**Q: What is projectivity and why does it matter for parsing algorithms?**
A: A parse is projective if no two dependency arcs cross when the sentence is written linearly. Projective parses can be recovered by efficient dynamic programming (O(n³) or O(n) with transitions). Non-projective parses require more expensive algorithms (pseudo-projective transformations, or graph-based parsers with non-projective extensions). Most natural language is mostly projective (~98% in English newswire), but free word-order languages (Czech, German) have more non-projective constructions.

**Q: How does BERT represent syntactic structure?**
A: BERT does not explicitly parse text. However, probing studies show that attention patterns in middle layers encode dependency relations (syntactic head can often be found via maximum attention weight), and that BERT representations encode constituency structure that can be extracted by linear probes. This suggests BERT implicitly learns syntactic structure through the pretraining objective.

---

## Common Mistakes / Gotchas

- **Confusing head and modifier**: In "the big dog," the head of the NP is `dog` (not `big`, not `the`). The head determines the syntactic category of the phrase. Modifiers depend on the head.
- **Assuming one correct parse**: Many sentences are genuinely syntactically ambiguous. "I saw the man with the telescope" has two valid parses (who has the telescope?). Parsers output one, but both may be correct.
- **Using constituency for cross-lingual work**: Phrase structure labels (`NP`, `VP`) are language-specific — the English CFG cannot parse Japanese. Universal Dependencies (UD) provides a consistent dependency annotation scheme for 100+ languages.
- **Conflating syntactic and semantic roles**: Syntactic subject (nsubj) ≠ semantic agent. "The window was broken by the ball" — "the window" is the syntactic subject but the semantic patient.

---

## Further Reading / Paper References

- Chomsky, N. (1957). *Syntactic Structures.* — origin of generative phrase structure grammar
- Tesnière, L. (1959). *Éléments de syntaxe structurale.* — origin of dependency grammar
- Nivre, J. (2003). *An Efficient Algorithm for Projective Dependency Parsing.* — arc-standard transition system
- Nivre, J. et al. (2016). *Universal Dependencies v1.* [[arxiv:1603.03380]] — cross-lingual UD scheme
- Hewitt, J. & Manning, C.D. (2019). *A Structural Probe for Finding Syntax in Word Representations.* [[arxiv:1905.06316]] — syntax in BERT
- Jurafsky & Martin, *Speech and Language Processing* Ch. 12 (constituency), Ch. 14 (dependency)
