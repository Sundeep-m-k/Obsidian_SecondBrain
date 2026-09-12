# Rule-Based Tokenizers

tags: #nlp #tokenization #preprocessing #rules #regex
links: [[Whitespace Tokenization]] [[Tokenization — BPE]] [[SentencePiece]] [[Sentence Boundary Detection]]

---

## Definition + Intuition

**Rule-based tokenizers** use handcrafted regular expressions, finite-state automata, and linguistic rules to split text into tokens. They go beyond whitespace splitting by handling punctuation, contractions, abbreviations, URLs, numbers, and other linguistic phenomena.

> **Intuition**: Whitespace tokenization uses one rule ("split on space"). Rule-based tokenization uses a book of rules: separate sentence-final punctuation from words, but not abbreviations; split contractions; keep URLs intact; handle hyphenated compounds; deal with currencies, dates, and numbers. Each rule encodes linguistic knowledge about what constitutes a meaningful token in a given language.

---

## Key Properties / Types

### Core Tokenization Decisions

| Phenomenon | Decision | Example |
|-----------|---------|---------|
| **Sentence-final punctuation** | Split | `"Hello."` → `["Hello", "."]` |
| **Abbreviation period** | Keep | `"Dr. Smith"` → `["Dr.", "Smith"]` |
| **Decimal numbers** | Keep | `"3.14"` → `["3.14"]` |
| **URLs** | Keep as one token | `"http://x.com"` → `["http://x.com"]` |
| **Contractions** | Split on apostrophe | `"don't"` → `["do", "n't"]` |
| **Clitic 's** | Split | `"John's"` → `["John", "'s"]` |
| **Hyphenated compounds** | Language-dependent | `"state-of-the-art"` → varies |
| **Currency** | Split or keep | `"$100"` → `["$", "100"]` |

### Major Rule-Based Tokenizers

**Moses Tokenizer** — standard for MT preprocessing:
- Separates punctuation from words; language-specific rules for 30+ languages
- Still widely used as a preprocessing baseline

**Penn Treebank (PTB) Tokenizer** (NLTK):
```python
from nltk.tokenize import TreebankWordTokenizer
tok = TreebankWordTokenizer()
tok.tokenize("They'll save $100 in N.Y. at the store.")
# → ["They", "'ll", "save", "$", "100", "in", "N.Y.", "at", "the", "store", "."]
```

**spaCy Rule-Based Tokenizer** — two-pass approach:
1. Split on whitespace → rough tokens
2. Apply prefix rules (strip `"`, `(`, `$`), suffix rules (strip `.`, `,`, `"`, `)`), infix rules (split on `-`, `/`, `'`)
3. Apply exception dictionary (don't split `N.Y.`, `Dr.`, `e.g.`)

```python
import spacy
nlp = spacy.load("en_core_web_sm")
doc = nlp("Don't go to N.Y. without Dr. Smith's ticket.")
[tok.text for tok in doc]
# → ["Do", "n't", "go", "to", "N.Y.", "without", "Dr.", "Smith", "'s", "ticket", "."]
```

### Finite-State Transducer (FST) Approach

For industrial tokenizers, rules are compiled into a **finite-state transducer** — a finite automaton that both recognizes patterns and rewrites them simultaneously:

```
Input:  "don't"
FST:    d-o-n-'-t → d-o-n SPLIT '-t → ["don", "'t"]
```

FSTs are very fast (linear in input length) and can be composed — multiple rule layers compiled into one pass. Used in Google's Kestrel, Verbmobil, and other production NLP systems.

---

## Math / Formal Notation

A rule-based tokenizer is a sequence of rewriting rules:

$$R = [r_1, r_2, \ldots, r_k]$$

where each $r_i$ is a pair (pattern, replacement): $r_i = (p_i, s_i)$ with $p_i$ a regex and $s_i$ a substitution string.

Applied sequentially:
$$\text{tokens} = \text{split}(r_k(\ldots r_2(r_1(\text{text})) \ldots))$$

**Abbreviation detection heuristic**: a period after a token $w$ is sentence-final iff:
1. $w$ is not in the abbreviation list AND
2. $w$ is not a single uppercase letter AND
3. $w$ is not a sequence of initials (e.g., `U.S.A.`)

More precise methods use machine learning (see Sentence Boundary Detection).

---

## Examples (Concrete)

```
Input:  "The U.S.A. sent Dr. Jones to N.Y.C. on Jan. 3, 2024."

Naive whitespace:
  ["The", "U.S.A.", "sent", "Dr.", "Jones", "to", "N.Y.C.", "on", "Jan.", "3,", "2024."]

Rule-based (PTB-style):
  ["The", "U.S.A.", "sent", "Dr.", "Jones", "to", "N.Y.C.", "on", "Jan.", "3", ",", "2024", "."]
  ← comma split from "3,"; final period split from "2024."; abbreviation periods kept
```

**Clitic splitting (English):**
```
"John's books" → ["John", "'s", "books"]   ← possessive clitic
"It's raining"  → ["It", "'s", "raining"]  ← contraction clitic
"She'll leave"  → ["She", "'ll", "leave"]  ← future clitic
```

**Language-specific example (German):**
```
Compound: "Bundesministerium" → ["Bundesministerium"]
          (German compounds are one orthographic word — compound splitting is separate!)
```

---

## How It Connects to ML / NLP

| Rule-Based Tokenization Aspect | ML/NLP Relevance |
|-------------------------------|-----------------|
| Consistent punctuation splitting | Vocabulary size reduction; consistent subword units |
| Contraction splitting | Explicit negation tokens; consistent forms |
| Abbreviation handling | Reduces false sentence boundaries |
| URL/email as single tokens | Avoids spurious subword splits |
| Language-specific rules | Required for non-English pipelines |

**Rule-based vs learned tokenizers:**

| | Rule-Based | BPE/SentencePiece |
|---|---|---|
| Language coverage | Requires expert per language | Data-driven; any language |
| Consistency | Deterministic | Deterministic |
| Handling new words | Fixed rules; may fail | Subword fallback always works |
| Punctuation | Explicit control | Implicit via frequency |
| Speed | Very fast (FST: O(n)) | Fast (O(n log n)) |

Modern neural NLP mostly uses subword tokenizers (BPE, WordPiece), but rule-based tokenization is still the **first stage** in many pipelines — spaCy runs rule-based tokenization before its neural tagger/parser. Moses tokenization is still standard in MT preprocessing.

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — tokenization defines the feature atoms
- [[Sentence Boundary Detection]] — closely related; often done together

---

## Common Interview Questions

**Q: How does spaCy's tokenizer handle abbreviations vs sentence-final periods?**
A: spaCy uses an exception dictionary of known abbreviations (Dr., U.S., etc.) and heuristics (single uppercase letters, sequences ending in periods). The period after a known abbreviation is kept attached; a period after an unknown word that is followed by a capitalized word is treated as a sentence boundary. The exception dictionary is language-specific and can be extended.

**Q: When would you use a rule-based tokenizer over BPE?**
A: Rule-based tokenizers give explicit control over tokenization decisions that matter for specific applications: keeping URLs intact, splitting contractions in a specific way, handling domain-specific abbreviations (medical: `mg`, `mL`, `IV`). BPE learns tokenization from data and may split these in arbitrary ways. Rule-based is preferred when: (1) interpretability matters, (2) domain has specific tokenization requirements, (3) you need consistent, deterministic tokenization across different models/systems.

**Q: What is a finite-state transducer and how is it used in tokenization?**
A: An FST is a finite automaton that reads input symbols and produces output symbols simultaneously. In tokenization, it reads input characters and outputs either the character or a split marker. Multiple rewriting rules (prefix stripping, suffix stripping, contraction splitting) can be composed into a single FST, allowing all rules to be applied in one linear-time pass over the input. FSTs are used in production NLP systems (Google, Amazon) for high-throughput tokenization.

---

## Common Mistakes / Gotchas

- **Applying English rules to other languages**: apostrophe-based contraction rules don't apply to French (`j'ai` → `j' ai`), Italian, or Irish differently than English. Always use language-specific tokenization.
- **Period ambiguity in abbreviations**: `"St. Louis"` has a period that is not sentence-final. A tokenizer without an abbreviation list or contextual model will incorrectly split sentences here.
- **Tokenizing before spell-correction**: if you split `"isn't"` into `["is", "n't"]` before spelling correction, the spell-corrector won't recognize `"n't"` as anything to correct. Order matters: correct → tokenize, not tokenize → correct.
- **Hard-coding for one domain**: a rule-based tokenizer tuned for newswire (Penn Treebank) will struggle with social media (hashtags, @-mentions, emoji), code, or clinical notes. Extend rules or switch to a learned tokenizer.

---

## Further Reading / Paper References

- Koehn, P. et al. (2007). *Moses: Open Source Toolkit for Statistical Machine Translation.* ACL. — includes Moses tokenizer
- Manning, C.D. & Schütze, H. *Foundations of Statistical Natural Language Processing.* Ch. 4 — tokenisation
- Kaplan, R. & Kay, M. (1994). *Regular Models of Phonological Rule Systems.* — FST theory
- spaCy tokenization: https://spacy.io/usage/linguistic-features#tokenization
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — text normalisation and tokenisation
