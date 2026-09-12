# Compositional Semantics

tags: #nlp #semantics #compositionality #logic #meaning-representation
links: [[Lexical Semantics]] [[Semantic Roles]] [[Word Sense and Polysemy]] [[Frame Semantics]] [[AMR Parsing]]

---

## Definition + Intuition

**Compositional semantics** studies how the meaning of a complex expression is built from the meanings of its parts and the rules for combining them.

The core principle is the **Principle of Compositionality** (Frege's Principle):
> *The meaning of a complex expression is a function of the meanings of its parts and the way they are syntactically combined.*

> **Intuition**: You've never heard the sentence "The purple elephant played jazz on Mars" — but you understand it immediately. Why? Because you know the meanings of `purple`, `elephant`, `played`, `jazz`, `Mars` and the rules for combining NPs, VPs, and PPs. Compositionality is what makes language *productive* — infinitely many sentences from finite words + rules.

---

## Key Properties / Types

### The Principle of Compositionality
- **Productivity**: finite vocabulary + finite rules → infinite sentences
- **Systematicity**: understanding "John loves Mary" implies understanding "Mary loves John"
- **Incrementality**: meaning builds as words arrive (important for real-time parsing)

### Formal Meaning Representations

**1. First-Order Logic (FOL)**
- Expresses sentences as logical formulas
- "Every dog barked": $\forall x [\text{dog}(x) \rightarrow \text{barked}(x)]$
- "Some cat slept": $\exists x [\text{cat}(x) \wedge \text{slept}(x)]$

**2. Lambda Calculus**
- Represents functions over entity/event variables
- Allows compositional semantic construction rule-by-rule

**3. Neo-Davidsonian Event Semantics**
- Introduces an event variable $e$
- "John ran": $\exists e [\text{run}(e) \wedge \text{agent}(e, \text{John})]$
- Allows optional arguments and modifications naturally

**4. Abstract Meaning Representation (AMR)**
- Graph-based, rooted DAG
- Nodes = concepts, arcs = relations
- More practical than FOL for NLP

**5. Distributional Compositional Semantics**
- Represent phrases as combinations of word vectors
- Various composition operations: addition, multiplication, tensor product, neural network

### Semantic Phenomena Requiring Compositionality

| Phenomenon | Example | Challenge |
|-----------|---------|-----------|
| Quantification | "Every student passed some exam" | Scope ambiguity |
| Negation | "not happy" ≠ "unhappy" | Negation scope |
| Intensionality | "John wants a unicorn" | No unicorn need exist |
| Presupposition | "The king of France is bald" | Presupposes France has a king |
| Modality | "John might leave" | Possible worlds semantics |
| Tense | "John had left" | Temporal reference |

---

## Math / Formal Notation

### Lambda Calculus for Compositional Semantics

**Lambda abstraction**: $\lambda x. \phi(x)$ — a function taking argument $x$ and returning $\phi(x)$

**Beta reduction** (function application): $(\lambda x. \phi(x))(a) \Rightarrow \phi(a)$

**Building NP meaning**:
```
"the dog":
  [[the]] = λP λQ . ∃!x [P(x) ∧ Q(x)]   (the unique x such that...)
  [[dog]]  = λx . dog(x)
  [[the dog]] = λQ . ∃!x [dog(x) ∧ Q(x)]   (apply [[the]] to [[dog]])
```

**Building VP meaning**:
```
[[barked]] = λx . barked(x)
```

**Building S meaning**:
```
[[the dog barked]]
  = [[barked]]([[the dog]])
  = (λx . barked(x))(λQ . ∃!x [dog(x) ∧ Q(x)])
  ... simplifies to: ∃!x [dog(x) ∧ barked(x)]
  "There is a unique dog and it barked"
```

### Scope Ambiguity

"Every student read some book":

**Reading 1** (∀ > ∃): Each student read (possibly different) some book:
$$\forall x [\text{student}(x) \rightarrow \exists y [\text{book}(y) \wedge \text{read}(x, y)]]$$

**Reading 2** (∃ > ∀): There is one specific book that every student read:
$$\exists y [\text{book}(y) \wedge \forall x [\text{student}(x) \rightarrow \text{read}(x, y)]]$$

Both are valid logical parses of the same sentence. Scope ambiguity is distinct from structural (syntactic) ambiguity.

### Vector Space Compositionality

For distributional semantics, composition operations:

**Additive**: $\vec{w_1 w_2} = \vec{w_1} + \vec{w_2}$ — baseline, ignores word order

**Multiplicative**: $\vec{w_1 w_2} = \vec{w_1} \odot \vec{w_2}$ — component-wise product; emphasizes shared features

**Weighted additive**: $\vec{w_1 w_2} = \alpha \vec{w_1} + \beta \vec{w_2}$ — learned weights

**Circular convolution** (holographic reduced representations):
$$\vec{w_1 w_2} = \vec{w_1} \star \vec{w_2}$$
where $(\vec{a} \star \vec{b})_k = \sum_j a_j b_{k-j \mod n}$ — encodes order information

**Neural composition** (most powerful):
$$\vec{w_1 w_2} = f(W_1 \vec{w_1} + W_2 \vec{w_2} + b)$$

This is the building block of recursive neural networks (RecNN, Tree-LSTM).

---

## Examples (Concrete)

### First-Order Logic Translation
```
"Every dog chased some cat"
∀x [dog(x) → ∃y [cat(y) ∧ chase(x, y)]]

"No student failed"
¬∃x [student(x) ∧ fail(x)]
= ∀x [student(x) → ¬fail(x)]

"John gave Mary a book"
∃e [give(e) ∧ agent(e, John) ∧ recipient(e, Mary) ∧ theme(e, Book)]
```

### AMR Representation
```
"The boy wants to go"

(w / want-01
   :ARG0 (b / boy)
   :ARG1 (g / go-01
             :ARG0 b))

Nodes: want-01, boy, go-01
Arcs: ARG0 (wanter), ARG1 (thing wanted), ARG0 (goer) [reusing "b"]
```

AMR captures that the "boy" is both the wanter and the implicit subject of "go" — a form of control that pure string representations miss.

### Non-Compositional Cases
These are exceptions to strict compositionality:
```
"Kick the bucket"  ≠ kick + the + bucket  (idiomatic: means "die")
"Hot dog"          ≠ hot + dog             (lexicalized compound)
"Red herring"      ≠ red + herring         (figurative)
"Let the cat out of the bag" = reveal a secret
```

Idioms are semantically opaque — the whole must be learned as a unit. This is why subword tokenizers can fail on idiomatic expressions.

---

## How It Connects to ML / NLP

| Compositionality Concept | NLP/ML Application |
|---|---|
| FOL / Lambda calculus | Semantic parsing (text → logic); KGQA |
| AMR | AMR parsing; structured NLG |
| Scope ambiguity | NLI; logical question answering |
| Vector composition | Sentence embeddings; phrase representations |
| Idiom / non-compositionality | Idiom detection; MWE handling |
| Recursive neural composition | Tree-LSTMs; syntactically structured models |
| Negation scope | Negation-aware NLI; sentiment analysis |

**Why compositionality matters for neural models:**

Transformers don't have an explicit compositionality mechanism — they use attention to combine information from any positions. This works well empirically but fails on:
1. Long-range compositional chains ("The man who the dog that the cat scratched bit left")
2. Novel compositional combinations unseen in training
3. Precise logical reasoning requiring scope resolution

**The "algebraic generalization" problem**: Can models systematically generalize to novel compositions? SCAN benchmark shows neural models fail at systematic compositional generalization. GECA, COGS, and similar benchmarks probe this directly. → [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]]

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Vector.md]] — phrase vectors as composed features
- [[3.ML & DL/1.Concepts/1.Foundations/Model.md]] — semantic parsing as a model of meaning

---

## Common Interview Questions

**Q: Why is compositionality important for language generalization?**
A: Compositionality is what allows understanding of novel sentences. If meaning were purely holistic (each sentence stored as a unit), language would not be learnable — there are infinitely many possible sentences. Compositionality means finite vocabulary + finite rules → infinite expressible meanings. Models that are more compositional generalize better to novel sentences unseen during training.

**Q: What is semantic parsing and how does it relate to compositionality?**
A: Semantic parsing is the task of mapping natural language sentences to formal meaning representations (FOL, lambda expressions, AMR, SQL, etc.). It is compositional by design — the parser builds the formal representation recursively, following the syntactic structure. Neural semantic parsers use seq2seq or graph generation to learn the mapping.

**Q: Why do large language models struggle with compositional reasoning?**
A: LLMs learn from distributional patterns — they are very good at recognizing patterns seen in training but can fail on systematic compositional combinations that are novel (even if all the parts are familiar). For example, an LLM trained on "walk twice" and "jump left" may not correctly execute "jump twice after walk left" without explicit chain-of-thought prompting.

---

## Common Mistakes / Gotchas

- **Assuming all language is compositional**: idioms, multi-word expressions, and metaphors are non-compositional or partially compositional. Strict compositionality fails for these cases.
- **Confusing semantic compositionality with syntactic structure**: compositionality is about *meaning* composition, not syntax. A sentence can be syntactically compositional but semantically non-compositional (idioms).
- **Ignoring scope ambiguity in NLI**: "Every student solved some problem" can have two meanings — a system that doesn't track quantifier scope will fail on logical inference involving these sentences.
- **Vector addition as a full model of composition**: simple addition ignores word order and syntactic structure. "Dog bites man" ≠ "Man bites dog" under addition — but the vectors are identical.

---

## Further Reading / Paper References

- Montague, R. (1970). *Universal Grammar.* Theoria. — formal compositional semantics
- Partee, B., ter Meulen, A. & Wall, R. (1990). *Mathematical Methods in Linguistics.* — lambda calculus for semantics
- Banarescu, L. et al. (2013). *Abstract Meaning Representation for Sembanking.* — AMR
- Lake, B.M. et al. (2018). *Generalization without Systematicity: On the Compositional Skills of Sequence-to-Sequence Recurrent Networks.* (SCAN) [[arxiv:1711.00350]]
- Socher, R. et al. (2013). *Recursive Deep Models for Semantic Compositionality Over a Sentiment Treebank.* [[arxiv:1310.4546]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 17 — logical semantics
