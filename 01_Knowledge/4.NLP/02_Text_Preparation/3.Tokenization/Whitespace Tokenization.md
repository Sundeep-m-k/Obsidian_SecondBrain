# Whitespace Tokenization

tags: #nlp #tokenization #preprocessing #baseline
links: [[Rule-Based Tokenizers]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]] [[SentencePiece]] [[Tokenization for CJK Languages]]

---

## Definition + Intuition

**Whitespace tokenization** splits text into tokens by splitting on whitespace characters (space, tab, newline). It is the simplest possible tokenizer: every contiguous sequence of non-whitespace characters becomes one token.

```python
"Hello, world!".split()       # → ["Hello,", "world!"]
"one  two   three".split()    # → ["one", "two", "three"]  (collapses runs)
```

> **Intuition**: Whitespace tokenisation assumes that word boundaries are always marked by spaces. This works reasonably well for English and other space-delimited Western languages, but immediately breaks for Chinese (no spaces), German compounds (`Donaudampfschifffahrtsgesellschaft` = one word, should be several tokens), and anything with punctuation attached to words.

---

## Key Properties / Types

### What It Does and Doesn't Handle

| Input | Whitespace tokens | Problem |
|-------|-----------------|---------|
| `"Hello, world!"` | `["Hello,", "world!"]` | Comma/exclamation attached to token |
| `"can't stop"` | `["can't", "stop"]` | Contraction as one token |
| `"New York City"` | `["New", "York", "City"]` | Multi-word entity split |
| `"$100.00"` | `["$100.00"]` | Currency + number unsplit |
| `"http://example.com"` | `["http://example.com"]` | URL as one token (good!) |
| `"北京"` | `["北京"]` | Chinese: no spaces → whole phrase = 1 token |

### Python Variants

```python
text = "Hello,   world!\nHow are you?"

# str.split() — splits on any whitespace run
text.split()
# → ["Hello,", "world!", "How", "are", "you?"]

# str.split(' ') — splits on single space only
text.split(' ')
# → ["Hello,", "", "", "world!\nHow", "are", "you?"]  ← broken

# Best practice: str.split() with no args (handles all whitespace, strips leading/trailing)
```

### When Whitespace Tokenization Is Acceptable

- **Rapid prototyping** where exact tokenisation doesn't matter
- **Pre-tokenized input** (already tokenised data in PTB-style: `"Hello , world !"`)
- **Log/tabular data** where columns are space-separated
- **After a proper tokenizer has already run** — whitespace split on output of Moses/spaCy

---

## Math / Formal Notation

A whitespace tokenizer is a function $\tau: \Sigma^* \rightarrow \Sigma^{* *}$:

$$\tau(s) = [s_1, s_2, \ldots, s_n]$$

where $s_i$ are the maximal non-whitespace substrings of $s$.

The **out-of-vocabulary (OOV) rate** under a vocabulary $\mathcal{V}$:

$$\text{OOV}(\text{corpus}) = \frac{|\{t \in \tau(\text{corpus}) : t \notin \mathcal{V}\}|}{|\tau(\text{corpus})|}$$

Whitespace tokenization produces many unique tokens (punctuation-attached forms), increasing OOV rate compared to a tokenizer that separates punctuation.

---

## Examples (Concrete)

```python
# nltk whitespace tokenizer
from nltk.tokenize import WhitespaceTokenizer
tok = WhitespaceTokenizer()
tok.tokenize("It 's a fine day , isn 't it ?")
# → ["It", "'s", "a", "fine", "day", ",", "isn", "'t", "it", "?"]
# This is pre-tokenized Penn Treebank style — spaces before punctuation
```

---

## How It Connects to ML / NLP

- As a **baseline**: always run whitespace tokenization first to understand the raw token distribution.
- In **pipeline evaluation**: compare downstream task performance with whitespace vs BPE tokens — the gap shows the value of better tokenization.
- As **pre-processing for BPE**: SentencePiece and some BPE implementations first split on whitespace, then apply subword rules.

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — tokenization defines the feature space
- [[Tokenization — BPE]] — the upgrade from whitespace tokenization

---

## Common Interview Questions

**Q: Why is whitespace tokenization insufficient for real NLP?**
A: It conflates words with their attached punctuation ("word," and "word" become different tokens), doesn't handle contractions consistently, fails on languages without spaces (Chinese, Japanese, Thai), and produces a large vocabulary with many hapax legomena. All of these hurt downstream model performance through vocabulary explosion, OOV tokens, and missed normalisations.

---

## Further Reading

- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — tokenisation overview
