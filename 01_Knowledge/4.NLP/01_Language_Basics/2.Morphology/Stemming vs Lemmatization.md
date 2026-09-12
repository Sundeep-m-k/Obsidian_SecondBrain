# Stemming vs Lemmatization

tags: #nlp #morphology #preprocessing #text-normalization #ir
links: [[Morphemes — Free and Bound]] [[Inflection vs Derivation]] [[Bag of Words]] [[TF-IDF]] [[Normalization]]

---

## Definition + Intuition

Both stemming and lemmatization are **text normalization** techniques that reduce word forms to a common base. They differ in how they do it and how linguistically correct the result is.

**Stemming**: a heuristic process that chops off word endings using rules, without understanding the word's meaning or grammar. Fast, crude, may produce non-words.

**Lemmatization**: uses morphological analysis and a vocabulary/lexicon to return the **lemma** — the canonical dictionary form of a word. Slower, linguistically correct, always produces a real word.

> **Intuition**:
> - **Stemming** is like trimming a plant with garden shears, no questions asked — you cut the same amount off everything regardless of what the plant is.
> - **Lemmatization** is like a botanist who knows each plant, understands what it is, and carefully trims it back to its healthy base form.

---

## Key Properties / Types

### Stemming Algorithms

**Porter Stemmer** (1980, most classic):
A sequence of 5 rule phases applied in order. Examples of rules:
```
Phase 1: sses → ss     ("caresses" → "caress")
Phase 1: ies  → i      ("ponies"   → "poni")
Phase 1: s    → ∅      ("cats"     → "cat")
Phase 2: ational → ate ("relational" → "relate")
Phase 3: alize → al    ("digitalize" → "digital")
Phase 5: e    → ∅      ("probate"  → "probat")
```

**Snowball (Porter2)**: improved version, supports multiple languages.

**Lancaster Stemmer**: more aggressive, higher recall, lower precision than Porter.

**Lovins Stemmer**: earliest (1968), single-pass, 294 suffix rules.

| Stemmer | Aggressiveness | Output quality |
|---------|---------------|----------------|
| Porter | Moderate | Non-words common |
| Snowball | Moderate | Slightly better than Porter |
| Lancaster | High | Many non-words, high conflation |
| Lovins | High | Very aggressive |

### Lemmatization Approaches

**Dictionary lookup**: maintain a morphological lexicon mapping every inflected form to its lemma.
- Pros: perfect for known words
- Cons: fails on OOV; requires complete lexicon per language

**Morphological analysis**: parse the word into morphemes, then reconstruct the lemma.
- Example: `walked` → `[walk][PAST]` → lemma `walk`

**Context-sensitive lemmatization**: uses POS tag or context to disambiguate.
- `better` as adjective → lemma `good`
- `better` as verb ("I'll better my score") → lemma `better`

---

## Math / Formal Notation

**Stemming** as a function $\sigma: \Sigma^* \rightarrow \Sigma^*$:

$$\sigma(w) = \text{stem of } w$$

Properties (desirable but not guaranteed):
- $\sigma(\sigma(w)) = \sigma(w)$ (idempotent)
- $\sigma(w_1) = \sigma(w_2)$ if $w_1, w_2$ share a lemma (conflation)

**Lemmatization** as a context-sensitive function:

$$\lambda(w, \text{context}) = \text{lemma}(w)$$

For unambiguous words: $\lambda(w) = \text{lemma}(w)$ (context-free).
For ambiguous words: requires POS tag as input.

**Effect on vocabulary size:**

Let $V$ = original vocabulary, $V_s$ = stemmed vocabulary, $V_l$ = lemmatized vocabulary.

$$|V_l| \leq |V_s| \leq |V|$$

In practice:
- Lemmatization reduces vocabulary by ~10–40% depending on language
- Stemming can reduce by 20–60% (higher conflation = more reduction)
- For morphologically rich languages (Finnish, Turkish), reduction can exceed 80%

**Conflation rate vs. precision tradeoff:**
$$\text{Under-stemming error: } \sigma(\text{operate}) \neq \sigma(\text{operating})$$
$$\text{Over-stemming error: } \sigma(\text{news}) = \sigma(\text{new})$$

---

## Examples (Concrete)

**Side-by-side comparison:**

| Input | Porter Stem | Lemma (verb) | Lemma (noun) |
|-------|-------------|--------------|--------------|
| `running` | `run` | `run` | — |
| `ran` | `ran` | `run` | — |
| `better` | `better` | `be` (???) | `good` |
| `studies` | `studi` | `study` | `study` |
| `caring` | `care` | `care` | — |
| `generously` | `generous` | `generously` | — |
| `having` | `have` | `have` | — |
| `teeth` | `teeth` | — | `tooth` |

Note: Porter gets `better` wrong for verbs (returns `better` not `be`). Lemmatization on `ran` correctly returns `run` — but *only if it knows `ran` is a verb*.

**Real pipeline examples (Python):**
```python
import nltk
from nltk.stem import PorterStemmer, WordNetLemmatizer

ps = PorterStemmer()
wn = WordNetLemmatizer()

words = ["studies", "studying", "studied", "studious", "student"]

for w in words:
    print(f"{w:12} | stem: {ps.stem(w):10} | lemma: {wn.lemmatize(w)}")

# Output:
# studies      | stem: studi      | lemma: study
# studying     | stem: studi      | lemma: studying  ← lemmatizer needs POS!
# studied      | stem: studi      | lemma: studied   ← same problem
# studious     | stem: studi      | lemma: studious
# student      | stem: student    | lemma: student
```

With POS tag:
```python
wn.lemmatize("studying", pos='v')  # → "study"
wn.lemmatize("better", pos='a')    # → "good"
wn.lemmatize("better", pos='v')    # → "better"
```

---

## How It Connects to ML / NLP

| Use Case | Stemming | Lemmatization |
|----------|----------|---------------|
| **IR / Search** | ✅ Good for recall | ✅ Better precision |
| **Bag-of-words features** | ✅ Reduces sparsity | ✅ More interpretable |
| **Text classification** | ✅ Often sufficient | ✅ Sometimes better |
| **Neural NLP (BERT etc.)** | ❌ Usually skip | ❌ Usually skip |
| **Morphologically rich languages** | ⚠️ Often fails | ✅ Essential |
| **Keyword extraction** | ✅ Fast | ✅ Preferred |

**When to use which:**
- Modern neural models (BERT, GPT) do their own implicit normalization — explicit stemming/lemmatization often degrades performance by removing information the model could use.
- For **classical ML pipelines** (TF-IDF + LogReg, SVM), lemmatization reduces vocabulary size and sparsity → better generalization with limited data. [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]]
- For **information retrieval** (BM25), stemming increases recall (matching `running` to `run` query) but may hurt precision. Lemmatization is more controlled.
- For **low-resource NLP** on morphologically rich languages, lemmatization is often a required preprocessing step before any modeling.

**Effect on feature space:**

Without normalization, `[run, runs, running, ran]` are 4 separate features in a bag-of-words model. After stemming: `[run, run, run, ran]` → 2 features. After lemmatization: `[run, run, run, run]` → 1 feature.

This directly reduces dimensionality → [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Dimensionality Reduction.md]]

---

## Common Interview Questions

**Q: Should you lemmatize before training a BERT model?**
A: Generally no. BERT's WordPiece tokenizer already handles morphological variation by splitting words into subwords. Lemmatization before BERT can actually hurt performance by removing grammatical information (tense, number) that the model might use. Lemmatization is most useful for sparse, classical bag-of-words pipelines.

**Q: What is the difference between stemming and lemmatization in terms of computational cost?**
A: Stemming is O(n) in word length with small constant — it's a sequence of string operations. Lemmatization requires dictionary lookup or morphological parsing — typically O(n) per word but with a much larger constant (dictionary lookup, possibly POS tagging first). At scale, stemming is significantly faster.

**Q: When would you prefer stemming over lemmatization?**
A: When speed matters more than linguistic precision (large-scale IR indexing), when no lemmatizer exists for the target language, when the task benefits from aggressive conflation (e.g. query expansion in search).

---

## Common Mistakes / Gotchas

- **Stemming irregular forms**: Porter gets `ran → ran` (correct, but not normalized to `run`). For irregular morphology, rule-based stemmers fail. Lemmatization handles irregulars via dictionary.
- **Lemmatizing without POS tags**: `wn.lemmatize("better")` returns `better` (assumes noun). With POS=adj, it returns `good`. Always supply POS when available.
- **Stemming before neural models**: Almost always harmful. Modern tokenizers + transformers handle morphological variation better than any stemming heuristic.
- **Assuming stemming = normalization**: Stemming conflates words that shouldn't be conflated (`universe`, `university` both → `univers` in Porter). This is over-stemming and hurts precision.
- **Using English stemmers on non-English text**: Do not run Porter on French, Spanish, or German. Use language-specific stemmers (Snowball has language variants) or language-specific lemmatizers.

---

## Further Reading / Paper References

- Porter, M.F. (1980). *An Algorithm for Suffix Stripping.* Program, 14(3), 130–137. — original Porter paper
- Lovins, J.B. (1968). *Development of a Stemming Algorithm.* Mechanical Translation and Computational Linguistics.
- Paice, C.D. (1990). *Another Stemmer.* SIGIR Forum. — Lancaster stemmer
- Manning, C.D. et al. *Introduction to Information Retrieval.* Ch. 2 — canonical IR treatment of stemming/lemmatization (free online)
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — morphological analysis
- NLTK documentation: `nltk.stem` — practical Python reference
