# Inflection vs Derivation

tags: #nlp #morphology #stemming #lemmatization #pos-tagging
links: [[Morphemes — Free and Bound]] [[Stemming vs Lemmatization]] [[Agglutinative Languages]] [[POS Tagging]]

---

## Definition + Intuition

Both inflection and derivation involve attaching bound morphemes (affixes) to a base. They differ in what they produce:

**Inflection**: modifies a word to express grammatical features (tense, number, case, gender, person) *without changing the core meaning or word class*.

**Derivation**: creates a genuinely *new word* — often changing the word class (part of speech) and/or core meaning.

> **Intuition**:
> - **Inflection** is like conjugating a verb in a spreadsheet — you're filling in the same cell for different grammatical contexts. `walk → walks → walked → walking` — it's still the same verb "walk," just in different grammatical clothes.
> - **Derivation** is like creating a new row in the spreadsheet. `walk → walker` — "walker" is a *new word*, a noun, not just a different form of the verb "walk."

---

## Key Properties / Types

### Inflection

| Feature | English Examples | Other Languages |
|---------|-----------------|-----------------|
| **Tense** | `walk → walked` | Spanish: `caminar → caminé` |
| **Number** | `cat → cats` | German: `Kind → Kinder` |
| **Person** | `walk → walks` (3rd sg) | Latin: `amo, amas, amat` |
| **Case** | (mostly lost in English) | Russian: `студент → студенту` (dat.) |
| **Gender** | (mostly lost in English) | French: `beau → belle` |
| **Aspect** | (via auxiliaries in English) | Slavic languages: perfective vs imperfective |
| **Mood** | `If I were you...` (subjunctive) | Spanish: `hablar → hable` |

**Properties of inflection:**
1. Does not change word class (noun stays noun, verb stays verb)
2. Does not create new dictionary entries — all inflected forms are under one lemma
3. Often **obligatory** in context (you must mark tense in English)
4. **Regular**: most words follow the same paradigm; irregular forms are exceptions (`go → went`)

### Derivation

| Process | Example | Class change |
|---------|---------|-------------|
| **Nominalization** | `teach → teacher` | V → N |
| **Adjectivalization** | `beauty → beautiful` | N → Adj |
| **Verbalization** | `sharp → sharpen` | Adj → V |
| **Adverbialization** | `quick → quickly` | Adj → Adv |
| **Negative prefix** | `happy → unhappy` | Adj → Adj (same class, new meaning) |

**Properties of derivation:**
1. May or may not change word class
2. Creates new dictionary entries with their own definitions
3. **Optional** — not grammatically required
4. Often **irregular/unpredictable** — `destroy → destruction` (not `destroyal`); `sing → singer` but `rob → robber` not `rob → *robber` in a different sense
5. May change pronunciation significantly

### The Inflection–Derivation Ordering Principle
Derivational morphemes appear *closer to the root*; inflectional morphemes appear *further from the root* (at the periphery):

```
un    +    system    +    atic    +    al    +    ly    +    ize    +    d
^deriv     ^root          ^deriv       ^deriv    ^deriv    ^deriv       ^inflect
```

Wait — let's do a cleaner example:
```
friend  +  li  +  ness  +  (no inflection here — no plural)
root      deriv   deriv

govern  +  ment  +  s
root      deriv    inflect  ← inflection always outermost
```

This is the **Affix Ordering Constraint** (or the Mirror Principle in Minimalist syntax).

---

## Math / Formal Notation

**Paradigm**: the complete set of inflected forms for a lexeme.

For a regular English verb, the paradigm has 5 forms:
$$\text{Paradigm}(\text{walk}) = \{\text{walk, walks, walked, walking, walked}\}$$
(base, 3sg-present, past, progressive, past-participle — the last two are identical for regular verbs)

**Morphological reinflection** (a key NLP task):

Given a source form and its morphological features, and a target feature bundle, predict the target form:

$$f(\text{walked},\ [\text{PAST}] \rightarrow [\text{PRES.3SG}]) = \text{walks}$$

This is a sequence-to-sequence problem:
$$P(t_1, \ldots, t_n \mid s_1, \ldots, s_m,\ \text{source\_tags},\ \text{target\_tags})$$

**Lemmatization** as a classification problem over the inflection paradigm:

$$\text{lemma}(w) = \arg\max_{\ell \in \mathcal{L}} P(\ell \mid w, \text{context})$$

---

## Examples (Concrete)

**Inflection — same lemma, different grammatical context:**
```
Lemma: GO
  go, goes, went, gone, going   ← all listed under "go" in a dictionary

Lemma: MOUSE
  mouse, mice                   ← irregular plural, same lemma

Lemma: GOOD
  good, better, best            ← suppletive (different root!), same lemma
```

**Derivation — new words, new dictionary entries:**
```
nation (N)     → national (Adj)  → nationalize (V)  → nationalization (N)
teach (V)      → teacher (N)     → teachable (Adj)
psychology (N) → psychological (Adj) → psychologically (Adv)
```

**Ambiguous case — is `-er` inflectional or derivational?**
```
tall → taller    ← inflectional comparative (same word class, grammatical)
teach → teacher  ← derivational agentive (new noun)
```
Same surface suffix, different morphological status. Context and semantic interpretation decide.

---

## How It Connects to ML / NLP

| Concept | NLP/ML Relevance |
|---------|-----------------|
| Inflection paradigm | Lemmatization target; morphological tagging |
| Derivation | Vocabulary explosion; out-of-vocabulary words |
| Lemmatization | Reduces sparsity for bag-of-words models |
| Stemming vs lemmatization | Stemming ignores this distinction; lemmatization respects it |
| Reinflection | Seq2seq task; standard morphology benchmark |
| Paradigm completion | Low-resource NLP; data augmentation |

**Why this matters for models:**

1. **Sparsity reduction**: `walked`, `walks`, `walking` are all forms of `walk`. A bag-of-words model that doesn't lemmatize treats these as 4 different features. Lemmatization (or inflection-aware tokenization) reduces this sparsity — [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Scaling.md]]

2. **Subword tokenization**: BPE/WordPiece often correctly split inflectional suffixes (`walk` + `##ing`) but struggle with derivational ones that change the root (`destroy` + `##ction` vs `destruct` + `##ion`).

3. **Cross-lingual transfer**: Languages with rich inflectional morphology (Turkish, Finnish) have much sparser data per lemma. Models trained on English (poor inflectional morphology) transfer poorly to these languages without explicit morphological normalization. — [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]]

---

## Common Interview Questions

**Q: Why is lemmatization better than stemming for IR/classification?**
A: Stemming uses heuristic rules to chop off suffixes and doesn't distinguish inflection from derivation, producing non-words (`studies → studi`). Lemmatization maps to the true dictionary form (`studies → study`), which is linguistically meaningful. For downstream tasks that benefit from interpretability or exact string matching, lemmatization is preferred. For pure IR recall (query expansion), stemming's recall gain may outweigh precision loss.

**Q: What is morphological tagging and how is it done?**
A: Morphological tagging assigns a bundle of grammatical features to each word token: `[VERB, PAST, 3SG, INDICATIVE]` etc. It is a sequence labeling task, typically solved with BiLSTM-CRF or fine-tuned transformer. UniMorph provides a cross-lingual feature schema for annotation.

**Q: How do transformers handle inflection vs derivation?**
A: Transformers learn contextualized representations that implicitly encode morphological information. Probing studies show that BERT's representations encode person, number, and tense in early layers and more abstract semantic features in later layers. However, they are not explicitly morphology-aware — they rely on seeing enough training examples of each form.

---

## Common Mistakes / Gotchas

- **Assuming stemming = lemmatization**: Stemming strips suffixes mechanically (`better → bett`). Lemmatization uses morphological knowledge to find the dictionary form (`better → good`). They are not the same.
- **Applying inflection rules to derived forms**: `nationally` is derived from `national`, not inflected from `nation`. Running a stemmer on it may produce the wrong result.
- **Ignoring derivation in vocabulary analysis**: Derivation is a major source of OOV words. A model trained on `teach` may not handle `unteachable` without seeing it, even if it knows `un-` and `-able` as subword units.
- **Assuming English morphology generalizes**: English has nearly no case inflection, minimal gender. Most world languages have far richer inflectional systems. English-centric NLP intuitions fail badly on Arabic, Finnish, Turkish.

---

## Further Reading / Paper References

- Spencer, A. & Zwicky, A.M. (eds.) (1998). *The Handbook of Morphology.* Blackwell. — comprehensive reference
- Cotterell, R. et al. (2018). *The CoNLL–SIGMORPHON 2018 Shared Task: Universal Morphological Reinflection.* — standard benchmark
- Vylomova, E. et al. (2020). *SIGMORPHON 2020 Shared Task 0: Typologically Diverse Morphological Inflection.* [[arxiv:2006.11572]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2.7 — morphology for NLP
- Peters, M. et al. (2018). *Deep Contextualized Word Representations.* (ELMo) — probing for morphological features [[arxiv:1802.05365]]
