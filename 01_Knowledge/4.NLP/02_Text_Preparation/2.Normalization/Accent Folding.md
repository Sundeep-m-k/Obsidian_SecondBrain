# Accent Folding

tags: #nlp #text-preprocessing #normalization #unicode #multilingual
links: [[Lowercasing]] [[Encoding and Unicode]] [[Contraction Expansion]] [[Normalization]]

---

## Definition + Intuition

**Accent folding** (also called **diacritic removal** or **accent stripping**) is the process of replacing accented or modified characters with their plain ASCII base forms:
`é → e`, `ñ → n`, `ü → u`, `ç → c`, `ā → a`, `ø → o`.

> **Intuition**: A library catalogue that returns "Zürich" when you search "Zurich" needs accent folding — so that users who can't type accents still find the right results. But a spell-checker that treats "resume" and "résumé" as the same word is wrong — they are different words in English. Accent folding is a precision/recall trade-off: more recall (finds more matches), less precision (conflates distinct forms).

---

## Key Properties / Types

### How It Works: Unicode NFD + Strip Combining Marks

The standard algorithm:
1. **NFD decompose**: split precomposed characters into base + combining mark
   `é` (U+00E9) → `e` (U+0065) + ` ́` (U+0301, COMBINING ACUTE ACCENT)
2. **Remove combining marks**: drop all characters in Unicode category `Mn` (Mark, Nonspacing)
   `e` + ` ́` → `e`

```python
import unicodedata

def strip_accents(text):
    nfd = unicodedata.normalize('NFD', text)
    return ''.join(c for c in nfd
                   if unicodedata.category(c) != 'Mn')

strip_accents("café")     # → "cafe"
strip_accents("naïve")    # → "naive"
strip_accents("Zürich")   # → "Zurich"
strip_accents("Ångström") # → "Angstrom"
```

### When to Use / Avoid

| Scenario | Fold? | Reason |
|----------|-------|--------|
| **IR / search (cross-lingual queries)** | ✅ Yes | Users may type without accent keys |
| **English-only text classification** | ✅ Often | Rare accented chars; mostly loanwords |
| **Spanish, French, Portuguese NLP** | ❌ No | Accents distinguish meaning: `sí`/`si`, `año`/`ano` |
| **Arabic, Hebrew (vowel points)** | ❌ No | Vowel diacritics are semantically significant |
| **Vietnamese** | ❌ Never | Tone marks are phonemic — stripping changes word identity entirely |
| **BERT multilingual** | ❌ No | Model was trained with accents intact |

### Ligature and Compatibility Variant Folding

NFKC normalisation also handles:
```
ﬁ → fi   (fi ligature)
ﬀ → ff   (ff ligature)
ﬆ → st   (st ligature)
² → 2    (superscript 2)
ｈ → h   (full-width h)
```

These are not diacritics but compatibility variants — handled by NFKC, not just diacritic removal.

---

## Math / Formal Notation

Accent folding as a character-level mapping $\phi: \Sigma_\text{accented} \rightarrow \Sigma_\text{ASCII}$:

$$\phi(c) = \begin{cases}
\text{base}(c) & \text{if } c \text{ has a combining mark} \\
c & \text{otherwise}
\end{cases}$$

**Vocabulary compression**:
$$V_\text{folded} = \{\phi(w) : w \in V\}$$

For corpora with mixed accented/unaccented forms: $|V_\text{folded}| < |V|$, reducing sparsity.

---

## Examples (Concrete)

**Meaning-changing accent removal (DO NOT fold these):**
```
Spanish:
  sí (yes)     → si (if)          ← WRONG: changes meaning
  año (year)   → ano (anus)       ← WRONG: embarrassing error
  más (more)   → mas (but)        ← WRONG: changes meaning
  papá (dad)   → papa (potato)    ← WRONG: changes meaning

French:
  ou (or)      → où (where)       ← case where accent distinguishes
  a (has/3sg)  → à (to/at)        ← grammatical distinction
```

**Safe to fold (English loanwords):**
```
café    → cafe      (common English usage)
naïve   → naive     (common English spelling)
résumé  → resume    (dangerous! "resume" = continue; "résumé" = CV)
élite   → elite     (acceptable in informal contexts)
```

---

## How It Connects to ML / NLP

- **Search / IR**: accent folding index allows queries without diacritics to match accented documents. Elasticsearch implements this via `asciifolding` token filter.
- **Cross-lingual embeddings**: `mBERT` uses WordPiece with accent stripping for the multilingual model — this was a deliberate choice to share vocabulary across languages that use the same base characters.
- **OCR post-processing**: OCR on old documents often misses diacritics. Accent folding before matching helps align OCR output to a clean reference.

**Cross-links:**
- [[Encoding and Unicode]] — NFD decomposition is a Unicode operation
- [[Lowercasing]] — often applied together with accent folding

---

## Common Interview Questions

**Q: Why is BERT multilingual's accent stripping controversial?**
A: `bert-base-multilingual-cased` strips accents during tokenisation to increase vocabulary sharing across languages (e.g. `é` → `e` shares a token with English `e`). However, this loses information for languages where accents are phonemically meaningful (Vietnamese, Arabic). `XLM-R` does not strip accents — it uses SentencePiece on the raw Unicode text — and generally outperforms mBERT on accented languages.

**Q: What is the difference between accent folding and Unicode normalisation?**
A: Unicode normalisation (NFC/NFKC) ensures a canonical representation but preserves accents — `é` in NFC is `é` (one precomposed character). Accent folding goes further: it explicitly removes the combining marks after NFD decomposition, producing a plain base character. Normalisation is always safe; accent folding is only safe for tasks/languages where diacritics do not change meaning.

---

## Common Mistakes / Gotchas

- **Folding Vietnamese, Thai, or Arabic**: these languages use diacritics as core phonemic distinctions. Stripping them destroys all meaning. Never apply accent folding to these without language detection first.
- **Folding before tokenisation for multilingual models**: XLM-R and multilingual BERT tokenisers have their own normalisation — do not pre-fold unless you know the tokeniser expects it.
- **Treating `ß` as an accented character**: `ß` (German eszett) is not a base+combining mark. NFD does not decompose it. Use `str.casefold()` to get `ss`, or handle separately.

---

## Further Reading

- Unicode Normalization FAQ: https://unicode.org/faq/normalization.html
- Devlin, J. et al. (2019). *BERT: Pre-training of Deep Bidirectional Transformers.* [[arxiv:1810.04805]] — mBERT accent stripping decision
- Conneau, A. et al. (2020). *Unsupervised Cross-lingual Representation Learning at Scale.* (XLM-R) [[arxiv:1911.02116]] — no accent stripping
