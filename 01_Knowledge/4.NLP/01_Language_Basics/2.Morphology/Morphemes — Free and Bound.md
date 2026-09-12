# Morphemes — Free and Bound

tags: #nlp #morphology #tokenization #subword
links: [[Inflection vs Derivation]] [[Stemming vs Lemmatization]] [[Agglutinative Languages]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]]

---

## Definition + Intuition

A **morpheme** is the smallest unit of language that carries meaning. Unlike phonemes (which carry no meaning themselves), morphemes are the atoms of *meaning* in a language.

- **Free morpheme**: can stand alone as a word. `cat`, `run`, `happy`
- **Bound morpheme**: cannot stand alone; must attach to another morpheme. `-ing`, `-ed`, `un-`, `-ness`

> **Intuition**: Think of morphemes as LEGO bricks of meaning. Some bricks are complete objects by themselves (free). Others are connectors that only make sense attached to something else (bound). The word `unhappiness` = `un-` + `happy` + `-ness` — one free morpheme flanked by two bound morphemes, each adding a layer of meaning.

---

## Key Properties / Types

### Free Morphemes
| Type | Description | Examples |
|------|-------------|---------|
| **Lexical (content)** | Carries core semantic content | `dog`, `run`, `blue`, `fast` |
| **Functional (grammatical)** | Expresses grammatical relationships | `the`, `and`, `of`, `is` |

### Bound Morphemes
| Type | Description | Examples |
|------|-------------|---------|
| **Prefix** | Attaches before the root | `un-happy`, `pre-view`, `re-do` |
| **Suffix** | Attaches after the root | `walk-ing`, `kind-ness`, `play-ed` |
| **Infix** | Inserted inside the root | `abso-bloody-lutely` (English expletive infixation); common in Tagalog |
| **Circumfix** | Attaches around the root | German `ge-mach-t` (done) |
| **Clitic** | Phonologically bound but syntactically free | English `'s` in "Mary's", `n't` in "can't" |

### Roots, Stems, and Affixes
```
unhappiness
│
├── un-        (prefix, bound, derivational)
├── happy      (root, free morpheme)
└── -ness      (suffix, bound, derivational)

played
│
├── play       (root, free morpheme)
└── -ed        (suffix, bound, inflectional)
```

**Root**: the irreducible core morpheme carrying primary meaning.
**Stem**: the base to which inflectional affixes attach (may itself be derived: `happi-` in `happiness`).
**Affix**: any bound morpheme (prefix, suffix, infix, circumfix).

### Morpheme Productivity
Some morphemes are **productive** — they can freely combine with new words:
- `-able`: `teachable`, `Googleable`, `microwaveable` ← highly productive
- `un-`: `unhappy`, `unclear`, `unGoogleable`

Others are **cranberry morphemes** — appear in only one or a few words with no independent meaning:
- `cran-` in cranberry (nowhere else)
- `twi-` in twilight
- `luke-` in lukewarm

---

## Math / Formal Notation

A word $w$ can be decomposed into a sequence of morphemes:

$$w = m_1 \oplus m_2 \oplus \ldots \oplus m_k$$

where $\oplus$ denotes concatenation and $m_i \in \mathcal{M}$ (the morpheme lexicon).

**Morphological segmentation** is the task of recovering this decomposition:

$$P(m_1, m_2, \ldots, m_k \mid w) = \prod_{i=1}^{k} P(m_i \mid m_1, \ldots, m_{i-1}, w)$$

In practice, unsupervised morpheme segmentation (e.g. Morfessor) uses a minimum description length (MDL) objective:

$$\mathcal{L} = \underbrace{|\text{lexicon}|}_{\text{model cost}} + \underbrace{\sum_{w} \log P(w \mid \text{model})^{-1}}_{\text{corpus cost}}$$

This is essentially a compression argument: a good morpheme segmentation lets you describe the corpus concisely. Sound familiar? BPE tokenization is the same idea.

**BPE connection:**
BPE merge rules approximate morpheme boundaries because morpheme boundaries are high-entropy character transitions. The most frequent character pairs (which BPE merges) tend to be within morphemes; rare pairs tend to be at boundaries.

---

## Examples (Concrete)

**Morpheme counting:**
```
"cats"         → cat + -s                           (2 morphemes)
"running"      → run + -ing                          (2 morphemes)
"unbelievable" → un- + believe + -able               (3 morphemes)
"antidisestablishmentarianism" 
               → anti- + dis- + establish + -ment + -arian + -ism
                                                     (6 morphemes)
```

**Bound morpheme changing meaning dramatically:**
```
happy    → unhappy     (negation via un-)
do       → undo        (reversal via un-)
cover    → uncover     (different reversal — discover ≠ disuncover)
```

**Same surface form, different morpheme count:**
```
"flies" (noun, plural) = fly + -s        (2 morphemes)
"flies" (verb, 3sg)    = fly + -s        (2 morphemes — same!)
"He flies a kite"      — context needed to resolve
```

---

## How It Connects to ML / NLP

| Morphology Concept | NLP/ML Application |
|---|---|
| Morpheme = unit of meaning | Motivation for subword tokenization (BPE, WordPiece) |
| Free morpheme | Corresponds roughly to a full token in whitespace tokenization |
| Bound morpheme | Handled by subword units in BPE (`##ing`, `##ed`) |
| Morphological segmentation | Preprocessing for agglutinative language models |
| Productive morphology | Why vocabulary cannot be closed — OOV words always possible |
| Cranberry morphemes | Hard cases for subword models — no compositional meaning |

**Why morphology matters for tokenization:**
A vocabulary-based model (e.g. word-level LM) fails on morphologically rich languages because the vocabulary explodes. Turkish `evlerinizden` ("from your houses") is one word but contains 5 morphemes. BPE's solution is to learn subword units that approximate morphemes without linguistic supervision.

**Cross-links:**
- [[3.ML & DL/1.Concepts/1.Foundations/Features.md]] — morphemes as features for text classification
- [[3.ML & DL/1.Concepts/6.Features and Representation/Vocabulary Pruning]] — morphology explains why rare words are common
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — models that understand morphology generalize better to unseen word forms

---

## Common Interview Questions

**Q: What is the relationship between morphemes and tokens in modern NLP?**
A: Subword tokenizers (BPE, WordPiece, SentencePiece) approximate morpheme segmentation. They split words at boundaries that are statistically rare character transitions — which correlate with morpheme boundaries. However, they are not linguistically informed; they learn these boundaries from corpus statistics alone, so they often do not align perfectly with true morpheme boundaries.

**Q: Why does morphological analysis matter for machine translation?**
A: In morphologically rich languages (Finnish, Turkish, Arabic), one word can encode what English expresses as several words + prepositions. A word-level MT model would see `evlerinizden` as a single unknown token. Morphological analysis splits it into meaningful units that can be translated compositionally.

**Q: What is an allomorph?**
A: Different phonological forms of the same morpheme. The English plural morpheme `-s` has three allomorphs: `/s/` (cats), `/z/` (dogs), `/ɪz/` (buses). They are the same morpheme (same meaning, same grammatical function) with different sounds depending on the preceding phoneme.

---

## Common Mistakes / Gotchas

- **Confusing morphemes with syllables**: `cat` = 1 syllable = 1 morpheme. `button` = 2 syllables = 1 morpheme. `cats` = 1 syllable = 2 morphemes. They are independent dimensions.
- **Confusing morpheme boundaries with BPE split points**: BPE splits `running` as `run` + `##ning` not `run` + `##ing` if `ning` appeared more frequently as a unit. Statistical and linguistic boundaries diverge.
- **Ignoring morphology for multilingual NLP**: English is morphologically poor — this is unusual. Most of the world's languages are more morphologically complex. Models trained only on English data develop blind spots around morphology.
- **Treating all suffixes as inflectional**: `-ness`, `-tion`, `-able` are derivational (create new words); `-ed`, `-ing`, `-s` are inflectional (mark grammatical features). The distinction matters for stemming algorithms.

---

## Further Reading / Paper References

- Matthews, P.H. (1991). *Morphology.* Cambridge University Press. — foundational textbook
- Creutz, M. & Lagus, K. (2005). *Unsupervised Morpheme Segmentation and Morphology Induction.* (Morfessor) [[TACL link]]
- Sennrich, R., Haddow, B. & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units.* (BPE paper) [[arxiv:1508.07909]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — morphology for NLP
- Cotterell, R. et al. (2016). *The SIGMORPHON 2016 Shared Task—Morphological Reinflection.* — morphology benchmarks
