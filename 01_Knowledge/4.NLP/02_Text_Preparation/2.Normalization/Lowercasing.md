# Lowercasing

tags: #nlp #text-preprocessing #normalization #vocabulary
links: [[Accent Folding]] [[Contraction Expansion]] [[Spelling Correction]] [[Normalization]] [[Stemming vs Lemmatization]]

---

## Definition + Intuition

**Lowercasing** converts all characters in a text to their lower-case equivalents. It is the simplest and most common text normalisation step — and also one of the most consequential.

> **Intuition**: "Apple", "APPLE", and "apple" are three different strings but one concept. For most NLP tasks, a model that sees all three as the same token generalises better with less data. But for some tasks ("US" ≠ "us", "IT" ≠ "it"), the case carries meaning. Lowercasing is a vocabulary compression decision — you trade potential information loss for reduced sparsity.

---

## Key Properties / Types

### When to Lowercase

| Task | Lowercase? | Reason |
|------|-----------|--------|
| **IR / search** | ✅ Yes | "Python" and "python" should match the same query |
| **Sentiment analysis** | ✅ Yes | Case rarely affects sentiment |
| **Topic modelling** | ✅ Yes | Same words regardless of capitalisation |
| **NER** | ❌ No | Capitalisation is a strong NER feature ("Apple" = company, "apple" = fruit) |
| **POS tagging** | ❌ No | Sentence-initial capitalisation affects tagging |
| **Machine translation** | ❌ No | Target language may need proper case output |
| **Coreference** | ❌ No | "He" vs "he" carry different positional signals |
| **LLM pretraining** | ❌ No | Models learn case patterns from data |

### True-Casing

**True-casing** is the inverse task: given lowercased text, restore the natural capitalisation. Used as a post-processing step in ASR and MT pipelines.

### Unicode Case Folding

Lowercasing is not always a single `str.lower()` call for non-ASCII text:

```python
# German ß (eszett)
"Straße".lower()  → "straße"   # standard lowercase
"Straße".casefold() → "strasse"  # casefold: ß → ss (for comparison)

# Greek final sigma
"Σίσυφος".lower() → "σίσυφος"   # Σ → σ initially, ς finally
```

`str.casefold()` (Python) or ICU's case folding is more aggressive than `str.lower()` — designed for comparison (two strings are equivalent regardless of case/script variant).

---

## Math / Formal Notation

**Vocabulary reduction**:

Let $V$ = original vocabulary, $V_l$ = lowercased vocabulary.

$$V_l = \{w.lower() : w \in V\}$$
$$|V_l| \leq |V|$$

In practice, lowercasing reduces English vocabulary by ~20–30% (many words appear only capitalised at sentence start).

**Effect on frequency counts**: lowercase merges counts:
$$f_l(\text{"apple"}) = f(\text{"apple"}) + f(\text{"Apple"}) + f(\text{"APPLE"}) + \ldots$$

This reduces sparsity — previously rare forms get merged with their common counterparts.

---

## Examples (Concrete)

```
Original:  "The QUICK Brown Fox Jumps Over THE LAZY Dog."
Lowercased: "the quick brown fox jumps over the lazy dog."

Original:  "US GDP grew 2.3% in Q3. It was driven by IT sector gains."
Lowercased: "us gdp grew 2.3% in q3. it was driven by it sector gains."
           ← "US"/"us", "IT"/"it" are now ambiguous
```

**NER failure from lowercasing:**
```
"Amazon reported strong earnings." → amazon (company)? river? → NER fails
"Apple announced a new product."  → apple (company)? fruit? → NER fails
```

Lowercased NER models lose their strongest signal for detecting proper nouns.

---

## Common Interview Questions

**Q: Should you lowercase text before training BERT?**
A: It depends on which BERT variant. `bert-base-uncased` was pretrained on lowercased text — you should lowercase inputs. `bert-base-cased` was pretrained on original-case text — do not lowercase. Using the wrong preprocessing degrades performance because the token vocabulary changes (e.g. "Apple" is not in the uncased vocabulary; it gets split into subwords differently).

**Q: Is lowercasing always safe for multilingual text?**
A: No. Some languages have case distinctions that carry grammatical meaning (e.g. German nouns are always capitalised). Turkish has a dotless-i (`ı`) / dotted-i (`İ`) distinction — `"İ".lower()` in English locale gives `"i"` but should give `"i"` with dot (which is `"İ"` → `"i"` in Turkish locale). Always use locale-aware case operations for non-English text.

---

## Common Mistakes / Gotchas

- **Lowercasing cased BERT inputs**: using `bert-base-uncased` preprocessing on `bert-base-cased` model (or vice versa) silently degrades performance.
- **Lowercasing acronyms**: "USA", "NASA", "HTTP" lose their identity after lowercasing — models may fail to recognise them.
- **Turkish locale bug**: Python's `str.lower()` uses the system locale for `İ`/`I` in Python 2; Python 3's `str.lower()` is locale-independent but may still produce wrong results for Turkish. Use `str.lower()` with `locale` module or ICU for correct Turkish casing.

---

## Further Reading

- Unicode Case Folding: https://www.unicode.org/reports/tr44/#Caseless_Matching
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — text normalisation
