# Spelling Correction

tags: #nlp #text-preprocessing #normalization #spelling #noisy-text
links: [[Contraction Expansion]] [[Lowercasing]] [[Noise Removal]] [[Tokenization — BPE]]

---

## Definition + Intuition

**Spelling correction** maps misspelled or non-standard word forms to their correct canonical forms. It is a fundamental text normalisation step for noisy user-generated content (social media, search queries, SMS, OCR output).

Two modes:
- **Non-word error correction**: the misspelled form is not a valid word (`teh → the`, `recieve → receive`)
- **Real-word error correction**: the misspelled form is a valid word but contextually wrong (`there/their/they're`, `affect/effect`)

> **Intuition**: A model trained on clean Wikipedia text has never seen `"teh"`, `"definately"`, or `"recieve"` — these produce OOV tokens or weird subword splits. Spelling correction maps user input to the vocabulary the model knows, bridging the style gap between messy user text and clean training data.

---

## Key Properties / Types

### Error Taxonomy

| Type | Example | Detection |
|------|---------|-----------|
| **Typographic** | `teh`, `hte`, `wiht` | Transposition/substitution; not in dict |
| **Phonetic** | `definately`, `seperate` | Phonetically plausible; common patterns |
| **Cognitive** | `there/their/they're` | Valid word; wrong context |
| **Morphological** | `runned` instead of `ran` | Incorrect inflection |
| **Abbreviation** | `u`, `r`, `gr8`, `b4` | SMS/social media shorthand |
| **Keystroke** | Adjacent-key errors: `tje`, `helo` | QWERTY proximity |
| **OCR error** | `l` ↔ `1`, `O` ↔ `0`, `rn` ↔ `m` | Visual similarity |

### Approaches

**1. Edit distance (non-contextual)**
Find the dictionary word with minimum edit distance to the input:
$$\hat{w} = \arg\min_{w \in \mathcal{V}} d_\text{edit}(\text{input}, w)$$

**2. Noisy channel model (probabilistic)**
$$\hat{w} = \arg\max_{w \in \mathcal{V}} P(w) \cdot P(\text{input} \mid w)$$
where $P(w)$ = language model prior, $P(\text{input} \mid w)$ = confusion matrix / edit probability.

**3. Neural spelling correction**
Sequence-to-sequence model: `"I recieve the leter"` → `"I receive the letter"`

**4. Contextual / transformer-based**
Fine-tuned BERT for masked prediction: `"She [MASK] to the store."` → `"went"` for real-word errors.

### Edit Distance (Levenshtein)

Operations with cost 1 each:
- **Insertion**: `cat` → `cart` (insert `r`)
- **Deletion**: `cart` → `cat` (delete `r`)
- **Substitution**: `cat` → `bat` (substitute `c` → `b`)

**Damerau-Levenshtein** adds:
- **Transposition**: `teh` → `the` (swap `e` and `h`) — costs 1

---

## Math / Formal Notation

### Levenshtein Distance (Dynamic Programming)

For strings $s$ (length $m$) and $t$ (length $n$), define matrix $D \in \mathbb{R}^{(m+1) \times (n+1)}$:

**Base cases:**
$$D[i, 0] = i \quad \forall i \in [0, m]$$
$$D[0, j] = j \quad \forall j \in [0, n]$$

**Recurrence:**
$$D[i, j] = \begin{cases}
D[i-1, j-1] & \text{if } s[i] = t[j] \text{ (match)} \\
1 + \min\begin{cases} D[i-1, j] & \text{(delete from } s) \\ D[i, j-1] & \text{(insert into } s) \\ D[i-1, j-1] & \text{(substitute)} \end{cases} & \text{otherwise}
\end{cases}$$

**Complexity**: $O(mn)$ time and space. For spell-checking against a vocabulary of size $|\mathcal{V}|$: $O(|\mathcal{V}| \cdot L^2)$ where $L$ = avg word length.

**BK-Tree optimisation**: a tree structure that prunes the vocabulary search, allowing $O(\log |\mathcal{V}|)$ average lookup for spelling correction.

### Noisy Channel Model

**Prior** $P(w)$: language model probability of word $w$ — from unigram / n-gram LM.

**Confusion matrix** $P(\text{typo} \mid w)$: empirical probability of observing typo given the intended word, estimated from a corpus of known spelling errors.

$$\hat{w} = \arg\max_w \underbrace{P(w)}_{\text{prior}} \cdot \underbrace{P(x \mid w)}_{\text{confusion}}$$

Taking logs:
$$\hat{w} = \arg\max_w \left[\log P(w) + \log P(x \mid w)\right]$$

This is exactly Bayes' theorem applied to spelling: the prior weights common words, the likelihood weights how likely the observed typo came from word $w$.

---

## Examples (Concrete)

### Levenshtein Distance Example

```
s = "recieve",  t = "receive"
     r  e  c  i  e  v  e
  [0, 1, 2, 3, 4, 5, 6, 7]
r [1, 0, 1, 2, 3, 4, 5, 6]
e [2, 1, 0, 1, 2, 3, 4, 5]
c [3, 2, 1, 0, 1, 2, 3, 4]
e [4, 3, 2, 1, 1, 2, 3, 3]  ← 'i'/'e' swap at positions 4,5
i [5, 4, 3, 2, 1, 2, 3, 4]
v [6, 5, 4, 3, 2, 1, 2, 3]
e [7, 6, 5, 4, 3, 2, 1, 2]

d("recieve", "receive") = 2
```

One substitution (`i↔e` swap = 2 edits in standard Levenshtein, 1 in Damerau).

### Social Media Normalisation

```
Raw:      "omg cant believe u r leaving so soon gr8 knowing u tbh"
Expanded: "oh my god cannot believe you are leaving so soon great knowing you to be honest"

Steps: abbreviation expansion → contraction expansion → spelling check
```

### Python Implementation

```python
from spellchecker import SpellChecker  # pyspellchecker

spell = SpellChecker()

tokens = "I recieve the leter every moaning".split()
corrected = []
for word in tokens:
    correction = spell.correction(word)
    corrected.append(correction if correction else word)

print(' '.join(corrected))
# → "I receive the letter every morning"
```

---

## How It Connects to ML / NLP

| Task | Spelling Correction Role |
|------|------------------------|
| **Search / IR** | Query correction before retrieval ("did you mean?") |
| **Sentiment analysis of reviews** | Fix typos so sentiment words are recognised |
| **Clinical NLP** | Medical abbreviations and OCR errors in EHRs |
| **Chatbots** | Normalise user input before intent classification |
| **LLM fine-tuning data** | Clean OCR-scanned books before training |
| **ASR post-processing** | Fix phonetically plausible speech transcription errors |

**Real-word errors are much harder**: `"I effect the outcome"` (should be `"affect"`) — both words are valid, only context reveals the error. This requires language model probability, not just dictionary lookup.

**Neural models handle some spelling implicitly**: subword tokenisers (BPE) split unknown words into subwords — `"recieve"` might tokenise as `["rece", "##ive"]`, which is different from `["receive"]`. The model may or may not bridge these. For high-stakes applications (medical, legal), explicit spelling correction before tokenisation is safer.

**Cross-links:**
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Cost Function.md]] — noisy channel model is a MAP estimation problem
- [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — neural spell correction is trained with cross-entropy
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — OOV misspellings hurt generalisation

---

## Common Interview Questions

**Q: What is the noisy channel model for spelling correction?**
A: The noisy channel model treats the observed (potentially misspelled) word as the output of a noisy channel that corrupts the intended word. We want to find the intended word $w$ that maximises $P(w) \cdot P(\text{observed} \mid w)$. The prior $P(w)$ comes from a language model (common words are more likely intended). The confusion probability $P(\text{observed} \mid w)$ comes from a matrix of typical typographic errors. This is a direct application of Bayes' theorem.

**Q: When is edit distance not enough for spelling correction?**
A: Edit distance is purely character-level — it doesn't know that common misspellings of "definitely" are "definately", "definitly", and "definetly" (all distance ≥ 2 from the correct form). It also doesn't handle real-word errors (there/their), abbreviations (u/you), or phonetic misspellings where the phoneme is correct but the grapheme is wrong. Contextual models (neural LMs) are needed for these cases.

**Q: How does keyboard proximity affect spelling correction?**
A: Adjacent keys on a QWERTY keyboard are commonly transposed or substituted in typographic errors. A QWERTY-aware confusion matrix weights substitutions of adjacent keys (e.g. `s`/`a`, `n`/`m`) more likely than non-adjacent keys (`s`/`z`). This improves the correction accuracy for typographic errors compared to a uniform substitution cost.

---

## Common Mistakes / Gotchas

- **Over-correcting domain terminology**: spell checkers trained on general text will "correct" medical terms (`tachycardia` → ???), programming identifiers (`getUser`), or proper nouns. Always whitelist domain vocabulary.
- **Correcting social media intentionally creative spelling**: "goooood" (intentionally extended), "lol", "omg", "ikr" are not spelling errors — they are style features. Overcorrecting them loses sentiment signal.
- **Applying spelling correction to non-English text without a language-specific dictionary**: English spell checkers will incorrectly flag valid words in other languages.
- **Ignoring real-word errors**: most simple spell checkers only catch non-word errors. Production systems (Grammarly, LanguageTool) use grammar and context models for real-word error correction.

---

## Further Reading / Paper References

- Kernighan, M.D., Church, K.W. & Gale, W.A. (1990). *A Spelling Correction Program Based on a Noisy Channel Model.* COLING.
- Levenshtein, V.I. (1966). *Binary Codes Capable of Correcting Deletions, Insertions, and Reversals.* — original edit distance
- Damerau, F.J. (1964). *A Technique for Computer Detection and Correction of Spelling Errors.* CACM.
- Brill, E. & Moore, R.C. (2000). *An Improved Error Model for Noisy Channel Spelling Correction.* ACL.
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2.5 — minimum edit distance
- `pyspellchecker`: https://pyspellchecker.readthedocs.io
- `LanguageTool`: https://languagetool.org — open-source grammar + spelling
