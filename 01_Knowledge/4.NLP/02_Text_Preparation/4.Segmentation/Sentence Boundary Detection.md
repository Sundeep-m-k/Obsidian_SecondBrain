# Sentence Boundary Detection

tags: #nlp #segmentation #sentence-splitting #preprocessing
links: [[Rule-Based Tokenizers]] [[Paragraph Segmentation]] [[Discourse Structure]] [[Noise Removal]]

---

## Definition + Intuition

**Sentence Boundary Detection (SBD)** — also called sentence segmentation or sentence splitting — identifies where one sentence ends and the next begins in running text. It is deceptively hard because the most common sentence boundary marker (`.`) is also used in abbreviations, numbers, URLs, and other non-boundary contexts.

> **Intuition**: Detecting sentence boundaries sounds trivial — "split on periods." But "Dr. Smith visited the U.S.A. in Jan. 2024." contains five periods, only one of which is a sentence boundary. SBD requires distinguishing sentence-final periods from abbreviation periods, decimal points, ellipses, and domain-specific notation.

---

## Key Properties / Types

### Why It's Hard

| Context | Example | Boundary? |
|---------|---------|----------|
| Sentence end | `"She left. He stayed."` | ✅ Yes (after "left") |
| Abbreviation | `"Dr. Smith came."` | ❌ No (after "Dr") |
| Initials | `"J.K. Rowling wrote it."` | ❌ No (after each initial) |
| Decimal | `"The value is 3.14 pi."` | ❌ No |
| URL | `"Visit http://x.com. Thanks."` | ✅ Yes (after ".com") |
| Ellipsis | `"And then... she left."` | ⚠️ Depends |
| Quotation | `"He said 'Go!' She agreed."` | ✅ Yes (after '!') |
| Parenthetical | `"(See Fig. 3.) The result..."` | ✅ Yes (after ')') |

### Approaches

**1. Rule-based (heuristic)**:
- Abbreviation list: `Dr., Mr., Mrs., Prof., U.S., Jan., ...`
- Regex for period + uppercase: split on `.` / `!` / `?` followed by whitespace + uppercase
- Exception: don't split if the preceding token is in the abbreviation list

**2. Punkt (Kiss & Strunk, 2006)** — unsupervised statistical:
- Learns abbreviation list from corpus
- Uses collocation statistics to distinguish abbreviations from sentence ends
- Used by NLTK's `sent_tokenize`

**3. ML-based** (spaCy, stanza):
- Trained classifier on each potential boundary (every period + surrounding context)
- Features: word before, word after, uppercase, digit, abbreviation status

**4. Transformer-based**:
- BERT fine-tuned on sentence boundary classification
- Each token labeled as sentence-start or continuation (sequence labeling)

### Implementations

```python
# NLTK Punkt
import nltk
sentences = nltk.sent_tokenize(text)

# spaCy
import spacy
nlp = spacy.load("en_core_web_sm")
doc = nlp(text)
sentences = [sent.text for sent in doc.sents]

# stanza
import stanza
nlp = stanza.Pipeline(lang='en', processors='tokenize')
doc = nlp(text)
sentences = [sent.text for sent in doc.sentences]
```

---

## Math / Formal Notation

### Punkt Algorithm

For each period-terminated token $w$, Punkt computes:

**Abbreviation probability** using a log-likelihood ratio:
$$\text{score}(w) = \log \lambda = \log \frac{P(\text{period} \mid \text{abbrev})}{P(\text{period} \mid \text{non-abbrev})}$$

A word is flagged as an abbreviation if $\text{score}(w) > \theta$ (threshold).

**Sentence boundary binary classification**:
$$P(\text{boundary} \mid w_i, w_{i+1}) = \sigma(f(w_i, w_{i+1}, \text{context}))$$

Features: is $w_i$ an abbreviation, is $w_{i+1}$ capitalised, is $w_i$ in the abbreviation list, punctuation type.

---

## Examples (Concrete)

```
Input: "Prof. Smith visited N.Y.C. on Jan. 5, 2024. He met Dr. Jones there."

Correct split:
  Sent 1: "Prof. Smith visited N.Y.C. on Jan. 5, 2024."
  Sent 2: "He met Dr. Jones there."

Naive period-split (wrong):
  "Prof." | "Smith visited N.Y.C." | "on Jan." | "5, 2024." | "He met Dr." | "Jones there."

Punkt / spaCy (correct):
  Recognises Prof., N.Y.C., Jan., Dr. as abbreviations
  Correctly identifies only the period after "2024" as a sentence boundary
```

---

## How It Connects to ML / NLP

- SBD is the **mandatory first step** before sentence-level classification, translation, or summarisation
- Errors in SBD propagate downstream: fused sentences hurt NER/parsing; over-split sentences lose cross-sentence context
- In RAG pipelines: sentence-level chunking for retrieval uses SBD

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — ML-based SBD is binary classification
- [[Paragraph Segmentation]] — next level up in the segmentation hierarchy

---

## Common Interview Questions

**Q: Why is sentence boundary detection a hard problem for rule-based systems?**
A: Because the most common boundary marker (`.`) is heavily overloaded: abbreviations (Dr., U.S.), decimals (3.14), URLs (x.com), and file extensions (.pdf) all use periods without being sentence boundaries. Rule-based systems require a comprehensive abbreviation list and many special cases. Statistical/ML approaches (Punkt, spaCy) learn these from corpus statistics.

**Q: How does Punkt detect abbreviations without a predefined list?**
A: Punkt uses the observation that abbreviations tend to occur before other non-sentence-boundary tokens, while sentence-final periods tend to be followed by capitalized words. It computes a log-likelihood ratio score for each period-terminated token based on collocation statistics and uses this to build an abbreviation list automatically from the corpus.

---

## Common Mistakes / Gotchas

- **Using NLTK's `sent_tokenize` for non-English text without the right model**: load the language-specific Punkt model for each language.
- **Applying SBD to already-segmented text**: if your corpus already has one sentence per line, don't run SBD on it — you'll re-split internal abbreviations.
- **Forgetting SBD before tokenisation**: most tokenisers operate at the sentence level; passing them a full paragraph gives suboptimal results (especially for models with maximum sequence lengths).

---

## Further Reading / Paper References

- Kiss, T. & Strunk, J. (2006). *Unsupervised Multilingual Sentence Boundary Detection.* Computational Linguistics. — Punkt algorithm
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2
- spaCy sentencizer: https://spacy.io/api/sentencizer
