# Agglutinative Languages

tags: #nlp #morphology #multilingual #tokenization #cross-lingual
links: [[Morphemes — Free and Bound]] [[Inflection vs Derivation]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]] [[Low-Resource Languages]]

---

## Definition + Intuition

A **morphological typology** classifies languages by how they form words from morphemes. The main types are:

- **Agglutinative**: morphemes stack transparently, one meaning per morpheme, clear boundaries.
- **Fusional (inflectional)**: one affix encodes multiple grammatical features simultaneously.
- **Isolating (analytic)**: words are mostly monomorphemic; grammar expressed through word order and separate function words.
- **Polysynthetic**: entire sentences can be expressed as one word; extreme agglutination.

An **agglutinative language** builds words by gluing (Latin: *agglutinare*) morphemes together in a transparent, compositional way — each morpheme contributes one meaning, and the boundaries between morphemes are clear.

> **Intuition**: In English, "from your houses" is three words. In Turkish, it's one: `evlerinizden`.
> ```
> ev    +  ler  +  iniz  +  den
> house   PLUR  2PL.POSS  from/ABL
> ```
> Every morpheme slot is filled transparently. This is agglutination.

---

## Key Properties / Types

### The Four Morphological Types

| Type | Characteristics | Examples |
|------|----------------|---------|
| **Agglutinative** | One morpheme = one feature; transparent boundaries | Turkish, Finnish, Hungarian, Swahili, Japanese, Korean |
| **Fusional** | One affix = multiple features; opaque boundaries | Russian, Latin, Arabic (partially), Spanish |
| **Isolating** | Minimal morphology; grammar via word order | Mandarin, Vietnamese, Thai |
| **Polysynthetic** | Verb incorporates arguments; very long words | Inuktitut, Yupik, many Amerindian languages |

### Agglutinative Properties
1. **Compositionality**: word meaning = sum of morpheme meanings
2. **Transparency**: morpheme boundaries are clear (low allomorphy)
3. **Productivity**: any valid combination of morphemes is a legal word
4. **Word length**: words can be extremely long (10+ morphemes)
5. **Vocabulary**: practically infinite — you can always add another morpheme

### Turkish Example — Showing Agglutination

```
git-me-yebil-ecek-ler-di
go-NEG-ABIL-FUT-3PL-PAST
"They would not be able to go"
= 1 word in Turkish, ~7 words in English
```

```
evlerinizden = ev + ler + iniz + den
               house + PL + your(pl) + from
```

### Finnish Case System (15 grammatical cases, all via agglutination)

```
talo    = house (nominative)
talon   = house's (genitive)
talossa = in the house (inessive)
talosta = from the house (elative)
taloon  = into the house (illative)
talolla = at the house (adessive)
```

---

## Math / Formal Notation

**Vocabulary explosion problem:**

For an isolating language (Mandarin), vocabulary grows roughly linearly with corpus size.

For an agglutinative language, the theoretical vocabulary size is:

$$|V_{\text{agglutin.}}| = \prod_{i=1}^{k} |S_i|$$

where $S_i$ is the set of options for morpheme slot $i$. For Turkish, with ~10 possible slots each having several options, this is combinatorially vast.

Turkish verb: subject agreement × tense × aspect × mood × polarity × ability... → thousands of valid forms per verb root.

**Impact on model perplexity:**

For a word-level language model, perplexity is:

$$\text{PPL} = 2^{H(P, Q)} = 2^{-\frac{1}{N}\sum_i \log_2 P(w_i)}$$

With a large, sparse vocabulary, many $P(w_i) \approx 0$ → high perplexity. Subword models reduce effective vocabulary size and avoid OOV tokens entirely.

**BPE compression on agglutinative vs isolating languages:**

For Turkish, BPE with 32k merges covers ~95%+ of morpheme-like units. For Mandarin, characters serve as natural units and BPE adds less value.

---

## Examples (Concrete)

**Turkish — a canonical agglutinative language:**
```
Türkiye'de = Türkiye + 'de    (Turkey + LOC: "in Turkey")
okul       = school
okulda     = at school
okuldan    = from school
okula      = to school
okullarda  = at schools (plural)
okullardaki = the one at schools

evlerinizden = ev + ler + iniz + den
(from your houses)
```

**Finnish — 15 cases:**
```
Helsinki + ssä = Helsingissä (in Helsinki) ← note: allomorphy ssä/ssa
Helsingistä    = from Helsinki
Helsinkiin     = into Helsinki
```

**Korean — verb agglutination:**
```
먹다       = eat (base)
먹고 싶다  = want to eat
먹고 싶었다 = wanted to eat
먹고 싶지 않다 = don't want to eat
먹고 싶지 않았을 것이다 = probably would not have wanted to eat
```

**Contrast with English (fusional/isolating):**
English `-s` in "he walks" encodes PRESENT + 3rd PERSON + SINGULAR in one morpheme — that's fusion, not agglutination. English `-ed` encodes PAST but doesn't specify person or number — partially isolating.

---

## How It Connects to ML / NLP

| Challenge | Description | Solution |
|-----------|-------------|---------|
| **OOV explosion** | Every new word form is a new token | Subword tokenization (BPE, SentencePiece) |
| **Data sparsity** | Each surface form seen rarely | Lemmatization + morphological normalization |
| **Translation** | One English phrase = one Turkish word | Seq2seq with subword units |
| **Cross-lingual transfer** | English-pretrained models struggle | mBERT, XLM-R; language-specific tokenizers |
| **POS tagging** | Each word has 100+ possible tags | Fine-grained tagset; morphological tagging |

**Why English-centric NLP breaks on agglutinative languages:**

1. Whitespace tokenization assumes words are separated by spaces (true for Turkish, false for some agglutinative languages, but even Turkish words are morphologically very long)
2. WordPiece/BPE with an English-trained vocabulary will produce terrible splits for Turkish
3. BERT trained only on English will fail to generalize to Turkish morphological patterns

**mBERT / XLM-R approach:**
Train a single model on 104 languages simultaneously. The shared subword vocabulary (250k tokens for XLM-R) includes enough Turkish/Finnish subwords to handle agglutinative morphology, though coverage is still imperfect.

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — cross-lingual generalization failure
- [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Vectorization.md]] — the vocabulary size problem
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — data sparsity per word form

---

## Common Interview Questions

**Q: Why is Turkish harder to build NLP models for than English, even with the same amount of data?**
A: Turkish is agglutinative — a single verb can have thousands of valid inflected forms. A word-level model sees each form as a separate token, with very few training examples per form. English has ~5 forms per verb; Turkish has thousands. This data sparsity per word form requires either subword tokenization or explicit morphological normalization to overcome.

**Q: Does BPE adequately handle agglutinative languages?**
A: Partially. BPE learns subword units that approximate morpheme boundaries through corpus statistics. For Turkish, BPE does reasonably well at splitting words into morpheme-like units. However, it's not linguistically guided — it splits based on frequency, not morphological structure. Language-specific morphological analyzers (e.g. Zemberek for Turkish) are more accurate but require expert development effort.

**Q: What is the difference between agglutinative and fusional morphology?**
A: In agglutinative morphology, each morpheme expresses one grammatical feature and boundaries are transparent (`ev+ler+iniz+den`). In fusional morphology, a single affix bundles multiple features, often with allomorphy that obscures the boundary (Latin `amo` = love + 1st person + singular + present + indicative + active — all in one form). Russian and Latin are prototypical fusional languages.

---

## Common Mistakes / Gotchas

- **Whitespace tokenization on agglutinative languages**: Works for Turkish (space-separated) but produces very long tokens that overwhelm vocabulary. Doesn't work at all for languages like Japanese (no spaces).
- **Applying English vocab size heuristics**: 30k vocabulary tokens is sufficient for English. For Turkish, you might need 100k+ for word-level coverage of the same corpus. Use subword vocabulary instead.
- **Assuming morpheme = token after BPE**: BPE tokens approximate morphemes but aren't the same. BPE splits `evlerinizden` into units based on frequency, not linguistic structure. May not align with true morpheme boundaries.
- **Ignoring morphological typology in multilingual research**: A model that works well on English + French + Spanish (all Indo-European fusional) may fail badly on Finnish + Turkish + Hungarian (Uralic/Turkic agglutinative) even with the same data.

---

## Further Reading / Paper References

- Sapir, E. (1921). *Language: An Introduction to the Study of Speech.* — original typology
- Comrie, B. (1989). *Language Universals and Linguistic Typology.* (2nd ed.)
- Sennrich, R. et al. (2016). *Neural Machine Translation of Rare Words with Subword Units.* [[arxiv:1508.07909]] — BPE motivation from morphologically rich MT
- Conneau, A. et al. (2020). *Unsupervised Cross-lingual Representation Learning at Scale.* (XLM-R) [[arxiv:1911.02116]]
- Çöltekin, Ç. (2010). *A Freely Available Morphological Analyzer for Turkish.* — Zemberek system
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — morphological typology
