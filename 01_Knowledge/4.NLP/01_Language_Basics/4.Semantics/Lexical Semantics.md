# Lexical Semantics

tags: #nlp #semantics #word-meaning #lexical-resources #word-sense
links: [[Compositional Semantics]] [[Semantic Roles]] [[Word Sense and Polysemy]] [[Frame Semantics]] [[Word2Vec — Skip-gram and CBOW]] [[GloVe]]

---

## Definition + Intuition

**Lexical semantics** is the study of word meaning — what individual words mean, how word meanings relate to each other, and how meaning varies with context.

Key questions:
- What does it mean for two words to mean the "same thing" (synonymy)?
- What is the relationship between *dog* and *animal*?
- How does *bank* mean both a financial institution and a river bank?
- Why does *run* mean something different in "run a company" vs "run a race"?

> **Intuition**: Lexical semantics is the study of the dictionary — but a richer dictionary that captures not just definitions but the network of relationships between all words, and how each word's meaning shifts in context. This directly motivates why word embeddings (Word2Vec, GloVe) encode semantic similarity — they operationalize lexical semantic relationships numerically.

---

## Key Properties / Types

### Lexical Relations

**Synonymy**: same or similar meaning
- `couch` ≈ `sofa`; `big` ≈ `large`
- True synonymy is rare — words with identical meaning in all contexts are almost nonexistent; most "synonyms" differ in register, connotation, or distribution

**Antonymy**: opposite meaning
- *Gradable antonyms*: `hot/cold`, `tall/short` — differ on a scale; allow "very hot"
- *Complementary pairs*: `alive/dead`, `present/absent` — no middle ground
- *Converse pairs*: `buy/sell`, `parent/child` — presuppose each other

**Hyponymy (is-a)**: specific-to-general relation
- `dog` is a hyponym of `animal`; `spaniel` is a hyponym of `dog`
- `animal` is a **hypernym** of `dog`
- These relations form a taxonomy (WordNet's backbone)

**Meronymy (part-of)**:
- `wheel` is a meronym of `car`; `finger` is a meronym of `hand`
- *Part meronymy*: finger is part of hand
- *Member meronymy*: tree is a member of forest
- *Substance meronymy*: water is substance of ocean

**Polysemy vs Homonymy**:
- **Polysemy**: one word form, multiple *related* meanings: `mouth` (body part; river mouth) — meanings share a historical origin
- **Homonymy**: one word form, multiple *unrelated* meanings: `bank` (financial institution; river bank) — meanings are historically unrelated (different etymologies)

**Metonymy**: using a related concept to refer to something:
- "The White House announced..." (building → administration)
- "I read Shakespeare" (author → works)

### Semantic Fields
A semantic field is a set of words that cover a conceptual domain:

```
COOKING: boil, fry, bake, sauté, roast, steam, grill, simmer
EMOTION: happy, sad, angry, fearful, surprised, disgusted, joyful
MOVEMENT: run, walk, crawl, jump, skip, march, stride
```

Words within a semantic field tend to occur in similar contexts and have high cosine similarity in word embedding space — this is the **distributional hypothesis**.

---

## Math / Formal Notation

### Distributional Semantics

The **distributional hypothesis** (Harris, 1954; Firth, 1957):
> "You shall know a word by the company it keeps."

Operationalized: words with similar meanings appear in similar contexts.

**Word-context matrix** $M \in \mathbb{R}^{|V| \times |C|}$:
$$M_{ij} = f(w_i, c_j)$$

where $f$ is some co-occurrence weighting function.

**Pointwise Mutual Information (PMI)**:
$$\text{PMI}(w, c) = \log \frac{P(w, c)}{P(w) \cdot P(c)} = \log \frac{\text{count}(w, c) \cdot N}{\text{count}(w) \cdot \text{count}(c)}$$

High PMI → word and context co-occur more than expected by chance.

**Positive PMI (PPMI)**:
$$\text{PPMI}(w, c) = \max(0, \text{PMI}(w, c))$$

Negative PMI values are unreliable (sparse data); clamping at 0 improves performance.

**Word similarity** via cosine of distribution vectors:
$$\text{sim}(w_1, w_2) = \cos(\vec{w_1}, \vec{w_2}) = \frac{\vec{w_1} \cdot \vec{w_2}}{|\vec{w_1}|\ |\vec{w_2}|}$$

Range: $[-1, 1]$; high value = similar distribution = similar meaning.

### Semantic Similarity Benchmarks

Standard evaluation datasets for word similarity:
- **WordSim-353**: 353 word pairs with human similarity ratings
- **SimLex-999**: 999 pairs; distinguishes *similarity* from *association*
- **MEN**: 3000 pairs

Evaluation metric: Spearman correlation $\rho$ between model scores and human ratings.

---

## Examples (Concrete)

### Hyponymy Hierarchy (WordNet-style)
```
entity
  └── physical object
        └── organism
              └── animal
                    └── mammal
                          └── canine
                                └── dog
                                      └── spaniel
                                            └── cocker spaniel
```

### Semantic Relations in Analogy Tasks
Word2Vec captures lexical relations as vector offsets:
```
vec("king") - vec("man") + vec("woman") ≈ vec("queen")
vec("Paris") - vec("France") + vec("Germany") ≈ vec("Berlin")
vec("better") - vec("good") + vec("bad") ≈ vec("worse")
```
The arithmetic works because synonyms cluster, hypernyms are directions, antonyms are near-opposing directions.

### Polysemy Example
```
"bank"
  Sense 1: financial institution
    → "I deposited money at the bank"
  Sense 2: side of a river
    → "We sat on the bank of the Thames"

These are homonymous (different etymologies: Old French "banque" vs Germanic "banke")
```

```
"mouth"
  Sense 1: body part (facial)
    → "She opened her mouth"
  Sense 2: river outlet
    → "The mouth of the Amazon"
  Sense 3: opening of container
    → "The mouth of the bottle"

These are polysemous (all related by spatial/opening metaphor)
```

---

## How It Connects to ML / NLP

| Lexical Semantic Concept | NLP/ML Application |
|---|---|
| Distributional hypothesis | Motivation for word embeddings (Word2Vec, GloVe, BERT) |
| Synonymy | Paraphrase detection; query expansion in IR |
| Hyponymy | Knowledge graph construction; NLI (hypernym implies hyponym) |
| Polysemy | Word sense disambiguation; contextualized embeddings |
| Semantic similarity | Sentence similarity; duplicate detection |
| Semantic fields | Topic modeling; domain adaptation |
| WordNet | External knowledge for text classification, WSD |

**From count-based to neural lexical semantics:**

1. **Count-based** (PPMI + SVD): explicit co-occurrence → dimensionality reduction → dense vectors
2. **Prediction-based** (Word2Vec): learn vectors by predicting context words → implicitly captures PMI [[Word2Vec — Skip-gram and CBOW]]
3. **Contextual** (BERT): different vector per occurrence of the same word → naturally handles polysemy [[BERT Embeddings]]

The progression from 1→2→3 is essentially the story of lexical semantic representation in NLP.

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — word vectors are feature vectors over lexical space
- [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Dimensionality Reduction.md]] — SVD reduces the word-context matrix
- [[3.ML & DL/1.Concepts/1.Foundations/Generalization.md]] — distributional hypothesis = generalization assumption for words

---

## Common Interview Questions

**Q: What is the distributional hypothesis and how does it justify word embeddings?**
A: The distributional hypothesis states that words with similar meanings tend to appear in similar contexts ("you shall know a word by the company it keeps"). This justifies representing words by their context distributions: if two words always appear with similar neighbors, they must have similar meanings. Word2Vec and GloVe operationalize this by learning dense vector representations that preserve distributional similarity. Words with high cosine similarity in embedding space tend to be synonyms or semantically related.

**Q: What is the difference between similarity and relatedness?**
A: Similarity requires sharing semantic features (`cat`/`dog` — both are pets, animals, mammals). Relatedness is a broader notion including any semantic association, including functional or encyclopedic (`cat`/`mouse` — low similarity but high relatedness). SimLex-999 was designed to measure similarity, not relatedness. Word2Vec tends to capture both; the distinction matters for some tasks.

**Q: Why can't static embeddings handle polysemy?**
A: Static embeddings (Word2Vec, GloVe) assign one vector per word type, regardless of context. The vector for "bank" is a conflation of all its senses. In dense corpora, the vector will be pulled toward the most frequent sense and lose information about rarer senses. Contextualized embeddings (ELMo, BERT) produce different vectors per occurrence, naturally representing polysemy.

---

## Common Mistakes / Gotchas

- **Treating synonymy as identical meaning**: "big" and "large" are similar but not interchangeable in all contexts ("big brother" ≠ "large brother"). True synonymy is extremely rare.
- **Confusing polysemy and homonymy**: they look the same on the surface but polysemy = related meanings (historically connected), homonymy = unrelated meanings (different word origins). Matters for WSD methods.
- **Cosine similarity captures association, not just meaning**: "doctor" and "hospital" have high cosine similarity (high co-occurrence) but are not synonyms — they are associated. Distinguish similarity from association.
- **WordNet coverage gaps**: WordNet covers common English vocabulary well but misses domain-specific terms, new words, and proper nouns. Don't assume WordNet covers your domain.

---

## Further Reading / Paper References

- Cruse, D.A. (1986). *Lexical Semantics.* Cambridge University Press. — foundational reference
- Firth, J.R. (1957). "A synopsis of linguistic theory 1930-55." — origin of distributional hypothesis
- Turney, P.D. & Pantel, P. (2010). *From Frequency to Meaning: Vector Space Models of Semantics.* JAIR. — comprehensive survey
- Miller, G.A. (1995). *WordNet: A Lexical Database for English.* CACM. — WordNet
- Mikolov, T. et al. (2013). *Efficient Estimation of Word Representations in Vector Space.* [[arxiv:1301.3781]] — Word2Vec
- Hill, F. et al. (2015). *SimLex-999: Evaluating Semantic Models with Genuine Similarity Estimation.* Computational Linguistics. — SimLex benchmark
