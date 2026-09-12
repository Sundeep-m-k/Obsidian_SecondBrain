# Noise Removal

tags: #nlp #text-preprocessing #cleaning #pipeline
links: [[Encoding and Unicode]] [[HTML and Markup Stripping]] [[Deduplication]] [[Normalization]] [[Tokenization — BPE]]

---

## Definition + Intuition

**Noise removal** is the process of identifying and eliminating characters, tokens, or spans from raw text that carry no useful linguistic signal for a downstream NLP task — or that would actively confuse a model.

"Noise" is task-relative: what is noise for sentiment classification (HTML tags, boilerplate footers) may be signal for web page structure analysis.

> **Intuition**: Raw text from the web is like a voice recording in a noisy café. You want the speech signal, not the background hiss, the music, the cutlery clatter. Noise removal is the acoustic filter step — strip what doesn't carry the message before you try to understand the message.

---

## Key Properties / Types

### Categories of Textual Noise

| Category | Examples | Removal Strategy |
|----------|---------|-----------------|
| **Markup / structure** | `<div>`, `</p>`, `&amp;`, `{{template}}` | HTML parser; regex |
| **Boilerplate** | "All rights reserved", cookie banners, nav menus | Boilerplate detection; domain heuristics |
| **Special characters** | `\x00`, `\ufffd`, control chars, null bytes | Unicode category filter |
| **URLs / emails** | `https://example.com`, `user@domain.com` | Regex; replace with `<URL>` token |
| **Numbers** | Dates, prices, IDs | Normalise or replace with `<NUM>` |
| **Punctuation** | Excessive `!!!!`, `...........` | Normalise runs |
| **Emoticons / emoji** | 😂🔥💯 | Remove or replace with text |
| **Code fragments** | `if (x > 0) { return; }` | Language detection; heuristic filters |
| **Repeated whitespace** | `word     word` | Collapse to single space |
| **Mojibake** | `â€™` (UTF-8 read as Latin-1) | Fix encoding before cleaning |

### Noise Levels

**Character-level noise**: stray bytes, invisible characters, bidirectional control characters (can reverse text display), zero-width joiners/non-joiners.

**Token-level noise**: meaningless tokens that survive tokenization — usernames, hashtags, product codes, log entries.

**Document-level noise**: duplicate documents, near-duplicate boilerplate pages, machine-translated or auto-generated text (relevant for pretraining corpus cleaning).

### Key Heuristic Filters (used in LLM pretraining)

Inspired by the C4, MassiveWeb, and RedPajama pipelines:

1. **Line length filter**: remove lines < 3 words or > 1000 characters
2. **Symbol-to-word ratio**: remove documents where `#symbols / #words > 0.1`
3. **Digit-to-character ratio**: remove documents where `#digits / #chars > 0.15`
4. **Repeated n-gram ratio**: detect and remove documents with excessive repeated phrases
5. **Perplexity filter**: use a small LM to score text; remove very high-perplexity (garbled) text
6. **Language ID filter**: keep only target language (fastText langid)

---

## Math / Formal Notation

### Repetition Score

For a document $D$ of $n$ characters, the **duplicate character fraction** from repeated n-grams:

$$\text{dup\_frac}(D, k) = \frac{\sum_{g \in \text{top-10-grams}_k} \text{count}(g, D) \cdot k}{n}$$

Documents where $\text{dup\_frac}(D, k) > \tau$ (e.g. 0.20 for $k=5$) are removed. This catches SEO-spam, auto-generated text, and copy-paste artefacts.

### Symbol Noise Ratio

$$r_\text{sym}(D) = \frac{|\{c \in D : c \notin \text{alphanum} \cup \text{punct}\}|}{|D|}$$

A document with $r_\text{sym} > 0.1$ is likely a code file, a log, or a garbled encoding artefact.

### Perplexity Filter

Given a trigram language model $p_\text{LM}$ trained on clean text:

$$\text{PPL}(D) = \exp\left(-\frac{1}{n}\sum_{i=1}^{n} \log p_\text{LM}(w_i \mid w_{i-2}, w_{i-1})\right)$$

High PPL → garbled or foreign-language text. A threshold $\tau$ (e.g. 2000) is used to filter.

### Regular Expression Patterns (Common)

```python
import re

URL_RE    = r'https?://\S+|www\.\S+'
EMAIL_RE  = r'\b[\w.+-]+@[\w-]+\.[a-z]{2,}\b'
PHONE_RE  = r'\b(\+?\d[\d\s\-().]{7,})\b'
HTML_RE   = r'<[^>]+>'
MENTION_RE = r'@\w+'
HASHTAG_RE = r'#\w+'
CTRL_RE   = r'[\x00-\x08\x0b\x0c\x0e-\x1f\x7f-\x9f]'

def clean(text):
    text = re.sub(HTML_RE,    '',      text)
    text = re.sub(URL_RE,     '<URL>', text)
    text = re.sub(EMAIL_RE,   '<EMAIL>', text)
    text = re.sub(CTRL_RE,    '',      text)
    text = re.sub(r'\s+',     ' ',     text).strip()
    return text
```

---

## Examples (Concrete)

### Raw → Cleaned (web text)

```
RAW:
"<p class=\"intro\">Check out our new product!!! 😍
Visit https://shop.example.com/promo?ref=abc123
&copy; 2024 Example Corp. All rights reserved.\r\n\t"

CLEANED:
"Check out our new product! Visit <URL>"
```

Steps taken:
1. HTML tag stripped (`<p ...>`)
2. Emoji removed (or → text: "heart eyes")
3. Exclamation normalised (`!!!` → `!`)
4. URL replaced with `<URL>` token
5. HTML entity decoded (`&copy;` → `©`) then removed (boilerplate)
6. Whitespace collapsed

### Mojibake Example

```
BROKEN:  "I canâ€™t believe itâ€™s not butter"
FIXED:   "I can't believe it's not butter"
```

`â€™` is the UTF-8 sequence for `'` (U+2019 RIGHT SINGLE QUOTATION MARK) mis-decoded as Latin-1. Fix: re-encode as Latin-1 bytes, then decode as UTF-8.

```python
broken = "I canâ€™t believe it"
fixed  = broken.encode('latin-1').decode('utf-8')
# → "I can't believe it"
```

### Boilerplate detection

```
Document: Wikipedia article on "Paris"
Boilerplate spans: header nav, "Edit" links, "References" section header,
                   footer "Retrieved from...", cookie notice
Content spans:     lead paragraph, body sections
```

Tools: `jusText`, `trafilatura`, `newspaper3k` perform this automatically.

---

## How It Connects to ML / NLP

| Noise Type | Impact on Downstream Model | Treatment |
|-----------|--------------------------|-----------|
| HTML tags | Vocabulary pollution; OOV tokens | Strip before tokenization |
| Boilerplate | Dilutes semantic signal; overfits to format | Remove at document level |
| Special characters | Tokenizer confusion; subword fragmentation | Unicode-normalise |
| Repeated n-grams | Skews LM training; memorisation | Dedup / perplexity filter |
| Encoding errors | Wrong characters → wrong tokens | Fix encoding first (always) |
| URLs / emails | Sparse tokens; privacy risk | Replace with placeholders |

**Pipeline position**: Noise removal is always the first step — before tokenization, normalization, or any feature extraction. Garbage in → garbage out at every subsequent step.

**Cross-links:**
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — data quality is the upstream constraint on all model performance
- [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — noisy training data causes models to memorise artefacts
- [[3.ML & DL/1.Concepts/6.Features and Representation/Data Representation.md]] — representation quality depends on clean input

---

## Common Interview Questions

**Q: Why is noise removal task-dependent? Give an example.**
A: Noise is anything that degrades performance on the target task. HTML tags are noise for sentiment classification (they carry no sentiment) but are features for webpage structure classification. Usernames (`@elonmusk`) are noise for topic modelling but are essential for social network analysis. Always define "noise" relative to the task before writing cleaning rules.

**Q: Should you always remove URLs and numbers, or replace them with placeholder tokens?**
A: Replacement (`<URL>`, `<NUM>`) is usually better than deletion. Deletion changes sentence length and can destroy syntax ("Buy it at `<URL>`" → "Buy it at" — grammatically broken). Replacement preserves sentence structure and allows the model to learn that a URL/number appeared in that position, which is itself a feature (e.g. financial text has lots of `<NUM>` tokens).

**Q: What is mojibake and how do you detect and fix it?**
A: Mojibake is garbled text resulting from text being encoded in one encoding (e.g. UTF-8) and decoded in another (e.g. Latin-1). Detection: scan for characteristic sequences (`â€™`, `Ã©`, `Â`). Fix: re-encode as the wrong encoding (bytes), then decode as the correct one. Library: `ftfy` (Fix Text For You) handles this automatically for most common cases.

---

## Common Mistakes / Gotchas

- **Cleaning before fixing encoding**: always fix encoding (mojibake, BOM, null bytes) *before* applying any other cleaning rule. A regex designed to strip `"` won't match `â€œ`.
- **Aggressive URL removal deleting sentence subjects**: "Amazon.com announced..." — stripping the URL might remove the subject. Use NER-aware URL handling or replace with `<URL>` rather than delete.
- **Stripping punctuation for all tasks**: punctuation is essential for parsing, NER, and sentence boundary detection. Only remove punctuation if your task genuinely does not need it (e.g. pure bag-of-words topic modelling).
- **Not versioning cleaning code**: cleaning decisions (what threshold, what regex) have a large impact on model performance. Always version and document your cleaning pipeline as carefully as your model code.
- **Assuming the same rules work across languages**: a symbol-to-word ratio threshold calibrated for English may incorrectly flag Chinese (where "words" are characters). All cleaning heuristics need language-specific calibration.

---

## Further Reading / Paper References

- Raffel, C. et al. (2020). *Exploring the Limits of Transfer Learning with a Unified Text-to-Text Transformer.* (C4 dataset cleaning) [[arxiv:1910.10683]]
- Wenzek, G. et al. (2020). *CCNet: Extracting High Quality Monolingual Datasets from Web Crawl Data.* [[arxiv:1911.00359]] — perplexity filter
- Penedo, G. et al. (2023). *The RefinedWeb Dataset for Falcon LLM.* [[arxiv:2306.01116]] — aggressive dedup + quality filters
- `ftfy` library: https://ftfy.readthedocs.io — automatic encoding fix
- `trafilatura`: https://trafilatura.readthedocs.io — boilerplate removal
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — text normalisation overview
