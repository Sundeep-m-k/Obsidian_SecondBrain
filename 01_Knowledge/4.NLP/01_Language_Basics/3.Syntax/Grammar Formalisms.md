# Grammar Formalisms

tags: #nlp #syntax #formal-grammars #parsing #theory
links: [[CFG and PCFG]] [[Parse Trees]] [[Constituency vs Dependency]] [[Head-Driven Phrase Structure]]

---

## Definition + Intuition

A **grammar formalism** is a formal system for describing the syntactic structure of a language — specifying the rules by which well-formed sentences are generated. Different formalisms make different trade-offs between expressive power, computational efficiency, and linguistic adequacy.

> **Intuition**: A grammar formalism is a blueprint language for describing languages. Just as you can describe the same building with architectural blueprints, engineering schematics, or a 3D model — each capturing different aspects — different grammar formalisms capture different structural properties of language.

---

## Key Properties / Types

### Chomsky Hierarchy

Grammars are organized by generative power:

```
Type 0: Unrestricted grammars        → Turing-complete (any recursively enumerable language)
Type 1: Context-sensitive grammars   → CSGs; can count (aⁿbⁿcⁿ)
Type 2: Context-free grammars        → CFGs; most NLP uses this level
Type 3: Regular grammars             → Finite automata; regular expressions
                                        (most restrictive)
```

**Natural language**: Mostly CFG-level, but some constructions (Swiss German cross-serial dependencies) require mildly context-sensitive power → **Mildly Context-Sensitive (MCS)** formalisms.

### Key Formalisms

| Formalism | Power | Key Feature | NLP Use |
|-----------|-------|-------------|---------|
| **CFG** | CF | Phrase structure rules | Constituency parsing |
| **PCFG** | CF + probs | Probabilistic rules | Statistical parsing |
| **TAG (Tree-Adjoining Grammar)** | MCS | Adjoining operation; long-range deps | Linguistic theory; some parsers |
| **CCG (Combinatory Categorial Grammar)** | MCS | Category combination rules; semantic interface | Neural CCG parsers |
| **LFG (Lexical Functional Grammar)** | CF | Separate c-structure and f-structure | Linguistic theory |
| **HPSG (Head-Driven Phrase Structure)** | CF | Feature structures; head-driven | Grammar engineering |
| **Dependency Grammar** | CF | Binary head-dependent relations | Dependency parsing |
| **Minimalist Grammar** | MCS | Move, Merge operations | Theoretical linguistics |

### CFG (Context-Free Grammar)
See [[CFG and PCFG]] for full treatment.

**Limitations**: Cannot express:
- Agreement across unbounded distance
- Cross-serial dependencies (Swiss German)
- Some scrambling phenomena

### CCG (Combinatory Categorial Grammar)

Every word is assigned a **category** (type). Categories combine via **combinatory rules**.

Basic categories: `N`, `NP`, `S`
Functional categories: `NP/N` (takes N to its right to make NP), `S\NP` (takes NP to its left to make S)

```
"The big dog barked"

the:     NP/N          (takes N → NP)
big:     N/N           (takes N → N; adjective modifier)
dog:     N
barked:  S\NP          (takes NP to left → S)

Application:
  big dog:  (N/N) N    → N          (forward application)
  the dog:  (NP/N) N   → NP         (forward application)
  the dog barked: (S\NP) NP → S     (backward application: barked+NP→S)
```

**CCG advantages**:
- Transparent semantic interface — categories directly encode semantic types
- Handles unbounded dependencies naturally
- One grammar for syntax + semantics

### LFG (Lexical Functional Grammar)

LFG maintains two parallel structures:
- **c-structure**: constituent phrase structure (CFG-like tree)
- **f-structure**: attribute-value matrix encoding grammatical functions

```
c-structure:       f-structure:
    S               [PRED 'bite'
   / \               SUBJ [PRED 'dog']
  NP  VP             OBJ  [PRED 'cat']
  |   / \            TENSE past     ]
 dog V   NP
     |   |
    bit  cat
```

F-structure captures grammatical relations abstractly — subject and object are the same regardless of whether the sentence is active or passive.

---

## Math / Formal Notation

### Formal Language Classes

A formal language $L \subseteq \Sigma^*$ is in class $\mathcal{C}$ if there exists a grammar of type $\mathcal{C}$ that generates exactly $L$.

**Containment**: $\mathcal{L}_3 \subsetneq \mathcal{L}_2 \subsetneq \mathcal{L}_{MCS} \subsetneq \mathcal{L}_1 \subsetneq \mathcal{L}_0$

**Parsing complexity by formalism**:
| Formalism | Parsing Complexity |
|-----------|-------------------|
| Regular | $O(n)$ |
| CFG (CYK) | $O(n^3)$ |
| TAG | $O(n^6)$ |
| CCG | $O(n^5)$ or $O(n^3)$ with supertagging |
| MCS in general | $O(n^6)$ |

Neural parsers typically achieve near-linear time by using greedy or beam-search transition systems — sacrificing optimality guarantees for speed.

---

## How It Connects to ML / NLP

| Formalism | Current NLP Relevance |
|-----------|----------------------|
| CFG/PCFG | Constituency parsing; grammar checking |
| CCG | Semantic parsing; logical form construction |
| Dependency grammar | Dependency parsing (spaCy, Stanford); universal application |
| LFG/HPSG | Grammar engineering; rule-based systems; linguistic annotation |
| TAG | Theoretical linguistics; some specialized parsers |

**Neural parsers largely bypass formalism details**: modern parsers (Kitaev & Klein 2018 for constituency; Dozat & Manning 2017 for dependency) use BERT + minimal structural decoding. The grammar formalism provides the output representation, not the parsing algorithm.

---

## Common Interview Questions

**Q: Why is CCG attractive for semantic parsing?**
A: CCG categories are typed — `NP/N` directly corresponds to the semantic type $e \rightarrow e$ (a function from entities to entities). This makes the syntax-semantics interface transparent: each combination rule in CCG corresponds to a semantic function application. This compositionality makes CCG ideal for mapping sentences to logical forms.

**Q: Why don't NLP systems use the most linguistically complete formalisms (TAG, HPSG)?**
A: More expressive formalisms are harder to parse (higher complexity), harder to learn (require larger annotated corpora), and harder to maintain. CFG and dependency grammar cover ~99% of naturally occurring constructions efficiently. The marginal gain from TAG/HPSG power is not worth the computational cost for most NLP applications.

---

## Common Mistakes / Gotchas

- **Conflating formalism and algorithm**: a formalism specifies what structures are valid; an algorithm specifies how to find them. CYK is an algorithm for CFGs; many algorithms exist for the same formalism.
- **Assuming context-free is always sufficient**: certain constructions in some languages (Swiss German, some scrambling) provably require more than CFG power. In practice, approximations work.

---

## Further Reading / Paper References

- Chomsky, N. (1956). *Three Models for the Description of Language.* IRE Transactions.
- Steedman, M. (2000). *The Syntactic Process.* (CCG) MIT Press.
- Bresnan, J. (2001). *Lexical-Functional Syntax.* (LFG) Blackwell.
- Joshi, A.K. (1985). *Tree Adjoining Grammars.* (TAG)
- Jurafsky & Martin, *Speech and Language Processing* Ch. 12
