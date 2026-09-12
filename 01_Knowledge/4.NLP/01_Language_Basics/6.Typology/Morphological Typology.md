# Morphological Typology

tags: #nlp #typology #morphology #cross-lingual #multilingual
links: [[Language Families]] [[Low-Resource Languages]] [[Agglutinative Languages]] [[Stemming vs Lemmatization]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]]

---

## Definition + Intuition

**Morphological typology** classifies languages by how they encode grammatical information — specifically how they combine morphemes to form words. It is one of the most important variables for NLP system design because it directly determines vocabulary size, tokenization difficulty, sparsity, and the type of features a model needs.

> **Intuition**: Two chefs with the same ingredients (words, grammar rules) can cook very differently. One stacks ingredients in separate dishes (isolating), another blends everything into one sauce (fusional), another layers them precisely one-at-a-time (agglutinative), and another makes one mega-dish with everything inside (polysynthetic). Morphological typology is about which cooking style a language uses — and it radically changes how you need to process it.

---

## Key Properties / Types

### The Four Morphological Types (Full Comparison)

| Property | Isolating | Agglutinative | Fusional | Polysynthetic |
|----------|-----------|---------------|---------|---------------|
| **Morphemes/word** | ~1 | 2–10+ | 2–5 | 10–20+ |
| **Morpheme boundaries** | N/A | Clear | Blurred | Variable |
| **Features per affix** | N/A | 1 | Multiple | Multiple |
| **Allomorphy** | None | Low | High | High |
| **Vocab size** | Small | Huge | Large | Astronomical |
| **Word order** | Rigid | Flexible | Somewhat flexible | Very flexible |
| **Examples** | Mandarin, Vietnamese | Turkish, Finnish, Korean | Russian, Latin, Arabic | Yupik, Mohawk, Inuktitut |

### Isolating (Analytic) Languages

Grammar is expressed through **word order** and **free grammatical words** (prepositions, auxiliaries) rather than affixes.

```
Mandarin Chinese:
我    昨天   看   了    一   本    书
Wǒ   zuótiān kàn  le  yī  běn  shū
I    yesterday look PERF one CL   book
"I read a book yesterday."

Each word = one morpheme. Tense (past) expressed by "了" (le) — a separate particle.
```

**NLP implications:**
- Word segmentation is the main challenge (no spaces in Chinese/Japanese)
- Vocabulary is relatively small and closed
- No morphological analyzer needed
- Character-level or subword models work well
- Word order is extremely important for syntax

### Fusional (Inflectional) Languages

One affix encodes **multiple features simultaneously**. Morpheme boundaries are blurred by phonological processes (allomorphy, assimilation).

```
Russian: стол (stol) = table
  стола  (stola)  = table's (genitive singular)
  столу  (stolu)  = to table (dative singular)
  столом (stolom) = with table (instrumental singular)
  столы  (stoly)  = tables (nominative plural)
  столов (stolov) = of tables (genitive plural)

The suffix "-а" encodes GENITIVE + SINGULAR simultaneously — not separable.
```

```
Latin: amo = I love
  amo   = I love (1SG, PRES, IND, ACT)
  amas  = you love (2SG, ...)
  amat  = he/she loves (3SG, ...)
  amamus = we love (1PL, ...)
One form encodes person + number + tense + mood + voice.
```

**NLP implications:**
- Moderate vocabulary explosion
- Lemmatization important for reducing sparsity
- Morphological tagging requires fine-grained tagset
- Allomorphy makes rule-based analysis hard → statistical/neural analyzers

### Agglutinative Languages

See [[Agglutinative Languages]] for full treatment. Summary:
- One morpheme = one feature
- Clear boundaries between morphemes
- Turkish `ev-ler-iniz-den` = house-PL-2PL.POSS-ABL = "from your houses"

### Polysynthetic Languages

An entire clause can be expressed as a **single word**. Verbs incorporate subject, object, tense, aspect, evidentiality, and sometimes spatial information.

```
Yupik (Central Alaskan):
Tuntussuqatarniksaitengqiggtuq
= He had not yet said again that he was going to hunt reindeer

tuntu  -ssur  -qatar  -ni   -ksaite  -ngqiggte  -uq
reindeer hunt  intend  say   NEG     again       3SG.IND

One word = one full sentence in English.
```

**NLP implications:**
- Essentially infinite vocabulary
- Word-level models are useless
- Character or morpheme-level models are necessary
- Essentially no training data available
- Severely under-resourced

---

## Math / Formal Notation

### Measuring Morphological Complexity

**Morpheme-per-word ratio** (synthesis index):
$$\text{SynthIdx} = \frac{\text{total morphemes in corpus}}{\text{total words in corpus}}$$

| Language | Synthesis Index |
|----------|----------------|
| Mandarin | ~1.0 (isolating) |
| English | ~1.7 |
| German | ~2.0 |
| Turkish | ~2.8 |
| Finnish | ~3.2 |
| Yupik | ~7.2 (polysynthetic) |

**Fusion index** (agglutination score):
$$\text{FusionIdx} = \frac{\text{morphemes expressing single feature}}{\text{total morphemes}}$$

High fusion index → agglutinative. Low → fusional (many features per morpheme).

### Vocabulary Growth Models

For an isolating language, vocabulary $V(N)$ as a function of corpus size $N$ tokens:
$$V(N) \approx k \cdot N^\beta, \quad \beta \approx 0.4 \text{ (Heap's law)}$$

For an agglutinative language, vocabulary grows faster because word forms multiply:
$$V_{\text{agglutin.}}(N) \gg V_{\text{isolating}}(N) \text{ for same } N$$

This is why a Turkish word-level model needs a much larger vocabulary than an English one for equivalent coverage — and why subword models are necessary for agglutinative languages.

**Effect on OOV rate** at vocabulary size $|V|$:

$$\text{OOV}(|V|) = P(w \notin V) \approx \left(\frac{|V|}{V_\infty}\right)^{-\gamma}$$

For Turkish, $V_\infty$ is much larger than English, so the same $|V|$ gives higher OOV.

---

## Examples (Concrete)

### The Same Meaning Across Types

"I will not go to the store."

**Isolating (Mandarin):**
```
我 不 会 去 商店
wǒ bù huì qù shāngdiàn
I  NEG will go store
6 words, each a single morpheme
```

**Agglutinative (Turkish):**
```
Mağazaya gitmeyeceğim
mağaza + ya  +  git + me + yecek + im
store  + DAT +  go + NEG + FUT  + 1SG
2 words, 6 morphemes — one is a single complex word
```

**Fusional (Russian):**
```
Я не пойду в магазин
ya ne poyd-u v magazin
I NEG go-1SG.FUT to store
5 words; "-u" encodes 1SG+FUT simultaneously
```

**Polysynthetic (Mohawk concept):**
Would incorporate all this into the verb complex — subject, object, directionality, aspect, negation all as one verbal word.

---

### Tokenization Outcomes by Typology

Consider BPE with 32k vocabulary on "to store":

```
English:       to | store                    (2 tokens — trivial)
Turkish:       mağaza | ya | git | me | yece | ğim (6 tokens — reasonable)
Finnish:       kaup | pa | an                 (3 tokens for "to store")
Yupik:         [single massive word] → splits into 15+ BPE tokens
```

The average number of BPE tokens per word is a good proxy for morphological complexity.

---

## How It Connects to ML / NLP

| Typological Feature | NLP Impact | Solution |
|--------------------|-----------|---------|
| Isolating | Word segmentation needed; no morphology | Character models; CRF segmenters |
| Agglutinative | Vocabulary explosion; data sparsity | BPE/SentencePiece; morphological analyzers |
| Fusional | Moderate sparsity; fine-grained morphological tagging | Lemmatization; morphological taggers |
| Polysynthetic | Extreme sparsity; essentially no resources | Character/byte models; extremely limited NLP |
| High synthesis | Needs more BPE tokens per word | Larger vocabulary; language-specific tokenizer |
| High fusion | Lemmatization important | Morphological parser + lemmatizer |

**Cross-lingual transfer and typology:**

Transfer is harder as typological distance increases. Key factors:
1. **Morphological type**: agglutinative → fusional transfer is harder than fusional → fusional
2. **Word order**: SOV ↔ SVO requires structural alignment
3. **Script**: different scripts require shared subword units or transliteration
4. **Lexical overlap**: cognates help; loanwords help

**mBERT transfer performance roughly correlates with:**
- Shared script (+)
- Same language family (+)
- Similar morphological type (+)
- Similar word order (+)

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — cross-lingual generalization
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — morphological features as inputs
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — data quantity and typological complexity interact

---

## Common Interview Questions

**Q: Why does English dominate NLP research despite representing a small fraction of world languages?**
A: English has the most digital text, the most annotated resources (Penn Treebank, SQuAD, SNLI), and the most researchers who are native/fluent speakers. It is also morphologically simple (isolating-leaning within Indo-European), making it easy to tokenize and model. This creates a feedback loop: models work well on English → more English resources → more English research. The result is severe underrepresentation of morphologically rich, non-Indo-European, and low-resource languages.

**Q: How does morphological typology affect the choice of tokenization strategy?**
A: Isolating languages (Mandarin) → character-level or word-level segmentation; morphology not a concern, word segmentation is. Agglutinative (Turkish, Finnish) → subword (BPE, SentencePiece) with large vocabulary; character-level also viable. Fusional (Russian, Arabic) → subword + lemmatization for classical ML; subword alone for neural. Polysynthetic → byte-level or character-level; standard subword models produce enormous token sequences.

**Q: What is a morphological analyzer and when do you need one?**
A: A morphological analyzer takes a word form as input and returns its lemma + morphological feature bundle: `walks` → `{lemma: walk, POS: VERB, TENSE: PRES, PERSON: 3, NUMBER: SG}`. You need one for: (1) data preprocessing for classical ML on morphologically rich languages, (2) morphological tagging tasks, (3) low-resource languages where neural models can't learn morphology from sparse data, (4) any task requiring explicit grammatical features (case, aspect, evidentiality).

---

## Common Mistakes / Gotchas

- **Treating all languages as English-like**: the most common NLP mistake. English is an outlier in its morphological simplicity. Building a pipeline that assumes one-word ≈ one-morpheme will fail on Turkish, Finnish, Arabic, etc.
- **Assuming word boundaries are clear**: Mandarin, Japanese, Thai have no spaces. Vietnamese has spaces but at the syllable level, not the word level. Always check segmentation conventions.
- **One tokenizer for all languages**: a BPE tokenizer trained on English with 30k tokens will be pathologically bad for Turkish (too few subword units to cover morphological complexity) or Mandarin (most Chinese characters become separate tokens, wasting vocabulary on rare characters).
- **Typological distance ≠ transfer difficulty alone**: script, domain, and data quality matter too. English → Swahili (different family, agglutinative) may outperform English → Classical Latin (same family, dead language with no neural data) purely because of available resources.

---

## Further Reading / Paper References

- Whaley, L.J. (1997). *Introduction to Typology: The Unity and Diversity of Language.* SAGE.
- Comrie, B. (1989). *Language Universals and Linguistic Typology.* (2nd ed.) Blackwell.
- Sapir, E. (1921). *Language: An Introduction to the Study of Speech.* — classic typological discussion
- WALS (World Atlas of Language Structures): https://wals.info — searchable typological database
- Ponti, E.M. et al. (2019). *Modeling Language Variation and Universals: A Survey on Typological Linguistics for Natural Language Processing.* [[arxiv:1807.00914]]
- Wu, S. & Cotterell, R. (2019). *Exact Hard Monotonic Attention for Character-Level Transduction.* ACL. — morphological inflection across types
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — morphological typology basics
