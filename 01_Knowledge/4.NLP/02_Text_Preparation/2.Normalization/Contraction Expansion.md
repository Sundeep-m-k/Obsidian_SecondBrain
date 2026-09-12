# Contraction Expansion

tags: #nlp #text-preprocessing #normalization #english
links: [[Lowercasing]] [[Accent Folding]] [[Spelling Correction]] [[Tokenization — BPE]]

---

## Definition + Intuition

**Contraction expansion** converts contracted word forms into their full equivalents:
`don't → do not`, `I'm → I am`, `it's → it is`, `they've → they have`.

In English, contractions merge two words (sometimes changing spelling) into one via an apostrophe. Expanding them ensures consistent representation and simplifies downstream processing.

> **Intuition**: "Don't" and "do not" mean the same thing but are represented as completely different token sequences. A bag-of-words model or keyword search that sees "don't" as one opaque token misses the "not" — which is critical for negation, sentiment, and entailment. Expansion makes the negation explicit and consistent.

---

## Key Properties / Types

### English Contraction Types

| Category | Contracted | Expanded |
|----------|-----------|---------|
| **Verb + not** | `don't`, `isn't`, `can't`, `won't` | `do not`, `is not`, `cannot`, `will not` |
| **Pronoun + be** | `I'm`, `you're`, `he's`, `we're` | `I am`, `you are`, `he is`, `we are` |
| **Pronoun + have** | `I've`, `you've`, `they've` | `I have`, `you have`, `they have` |
| **Pronoun + will** | `I'll`, `she'll`, `it'll` | `I will`, `she will`, `it will` |
| **Pronoun + would/had** | `I'd`, `they'd` | `I would`/`I had` (ambiguous!) |
| **Pronoun + is/has** | `he's` | `he is` / `he has` (ambiguous!) |
| **Copula contractions** | `that's`, `there's`, `here's` | `that is`, `there is`, `here is` |
| **Informal** | `gonna`, `wanna`, `gotta` | `going to`, `want to`, `got to` |

### Ambiguity in Contractions

Several contractions are genuinely ambiguous:
```
"He's leaving."     → "He is leaving."     (progressive)
"He's left."        → "He has left."       (perfect)
→ Context determines which expansion is correct.

"They'd go."        → "They would go."     (conditional)
"They'd gone."      → "They had gone."     (past perfect)
```

Resolution requires POS tagging or a language model.

### Implementation

**Rule-based** (fast, English-specific):
```python
CONTRACTIONS = {
    "don't": "do not", "doesn't": "does not", "didn't": "did not",
    "can't": "cannot", "couldn't": "could not", "won't": "will not",
    "wouldn't": "would not", "shouldn't": "should not",
    "i'm": "i am", "you're": "you are", "he's": "he is",
    "she's": "she is", "it's": "it is", "we're": "we are",
    "they're": "they are", "i've": "i have", "you've": "you have",
    "we've": "we have", "they've": "they have",
    "i'll": "i will", "you'll": "you will", "he'll": "he will",
    "she'll": "she will", "we'll": "we will", "they'll": "they will",
    "i'd": "i would", "you'd": "you would", "he'd": "he would",
    "gonna": "going to", "wanna": "want to", "gotta": "got to",
}

import re
def expand_contractions(text):
    text = text.lower()
    for contracted, expanded in CONTRACTIONS.items():
        text = re.sub(r'\b' + contracted + r'\b', expanded, text)
    return text
```

---

## Math / Formal Notation

Contraction expansion as a token-level rewriting rule:

$$T(w) = \begin{cases}
\text{expand}(w) & \text{if } w \in \mathcal{C} \\
w & \text{otherwise}
\end{cases}$$

where $\mathcal{C}$ is the contraction dictionary.

**Effect on vocabulary**: contractions become two tokens after expansion → vocabulary size increases slightly but each token is now a known, frequent word rather than a rare contracted form.

---

## Examples (Concrete)

```
Before: "I don't think it's working and they won't fix it."
After:  "I do not think it is working and they will not fix it."

Sentiment impact:
  "This isn't bad" → "This is not bad"
  Without expansion: "isn't" might score as a neutral token
  After expansion: "not bad" gives the correct negative-then-hedged signal

Negation detection:
  "can't stop"  → "can not stop" → negation of "stop" is now explicit
  "won't work"  → "will not work" → explicit negation
```

---

## How It Connects to ML / NLP

- **Sentiment analysis**: negation words (`not`, `never`, `no`) are key sentiment modifiers. If they are hidden inside contractions, rule-based sentiment systems miss them.
- **Keyword matching / IR**: searching for "do not" won't match "don't" unless one or both are expanded/contracted.
- **Neural models**: BERT and GPT tokenise `"don't"` as `["don", "\'", "t"]` or `["don't"]` depending on the vocabulary — the representation is unambiguous but non-compositional. For tasks requiring explicit negation detection, expansion helps classical models more than neural ones.

**Cross-links:**
- [[Lowercasing]] — applied together before expansion
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — sentiment as classification; negation is a key feature

---

## Common Interview Questions

**Q: Should you expand contractions before fine-tuning BERT?**
A: Generally no. BERT was pretrained on text with natural contractions intact. Expanding them changes the token sequence from what the model saw during pretraining, potentially hurting performance. Contraction expansion is most useful for classical ML pipelines (TF-IDF + SVM) where explicit negation tokens matter.

**Q: How do you handle ambiguous contractions like "he's" or "they'd"?**
A: For simple rule-based expansion, use the most frequent expansion ("he is" > "he has" for "he's" in most contexts). For precision, first run a POS tagger or dependency parser to determine the correct sense, then expand accordingly.

---

## Common Mistakes / Gotchas

- **Expanding apostrophes in possessives**: `"John's book"` → `"John is book"` is wrong. The apostrophe in `'s` can be either a contraction (he's → he is) or a possessive marker. Use word boundary and POS context to distinguish.
- **Casing after expansion**: `"I'm"` → `"I am"` (lowercase `am`), `"I'm"` at sentence start → `"I am"` (capital `I`).
- **Expanding in code or technical text**: `it's` in a sentence is a contraction; `it's` inside a Python error message or SQL query might not be. Apply expansion only to natural language spans.

---

## Further Reading

- `contractions` Python library: https://pypi.org/project/contractions/
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — text normalisation
