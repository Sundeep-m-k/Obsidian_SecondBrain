# Encoding and Unicode

tags: #nlp #text-preprocessing #encoding #unicode #cleaning
links: [[Noise Removal]] [[HTML and Markup Stripping]] [[Tokenization — BPE]] [[Tokenization for CJK Languages]]

---

## Definition + Intuition

**Character encoding** is the mapping between abstract characters (letters, symbols, ideographs) and their byte-level representation on disk or in memory.

**Unicode** is the universal character standard that assigns a unique **code point** (integer) to every character in every writing system — currently over 149,000 characters across 154 scripts.

**UTF-8** is the dominant encoding of Unicode on the web: a variable-width encoding that uses 1–4 bytes per code point, is backward-compatible with ASCII, and is the default for almost all modern NLP.

> **Intuition**: Imagine every character in the world is assigned a unique ID number (that's Unicode). Encoding is the question: "how do I write that ID number in bytes?" ASCII uses 1 byte per ID but only covers 128 characters. UTF-8 uses 1–4 bytes depending on how large the ID is — common characters (ASCII, Latin) use 1 byte, rare characters use more. Encoding bugs happen when you assume the wrong mapping between IDs and bytes.

---

## Key Properties / Types

### Unicode Concepts

**Code point**: a unique integer assigned to each character. Written as `U+XXXX` in hex.
```
U+0041  → A  (LATIN CAPITAL LETTER A)
U+00E9  → é  (LATIN SMALL LETTER E WITH ACUTE)
U+4E2D  → 中 (CJK UNIFIED IDEOGRAPH — zhōng, "middle/China")
U+1F600 → 😀 (GRINNING FACE emoji)
U+200B  → ​  (ZERO WIDTH SPACE — invisible! causes tokenization bugs)
```

**Unicode planes**:
- **BMP (Basic Multilingual Plane)**: U+0000–U+FFFF — most common scripts
- **SMP (Supplementary Multilingual Plane)**: U+10000–U+1FFFF — emoji, historic scripts, musical notation

**Unicode categories** (used for filtering):
```
L  = Letter     (Lu=uppercase, Ll=lowercase, Lo=other)
N  = Number     (Nd=decimal digit, Nl=letter number)
P  = Punctuation
S  = Symbol     (So=other symbol, includes emoji)
Z  = Separator  (Zs=space, Zl=line, Zp=paragraph)
C  = Other      (Cc=control, Cf=format, Cs=surrogate)
```

### Encoding Formats

| Encoding | Bytes/char | Coverage | Common Use |
|----------|-----------|---------|-----------|
| **ASCII** | 1 | 128 chars (English only) | Legacy; US text |
| **Latin-1 (ISO-8859-1)** | 1 | 256 chars (Western European) | Old Western web |
| **UTF-8** | 1–4 | All Unicode | Modern web (95%+) |
| **UTF-16** | 2 or 4 | All Unicode | Windows, Java internals |
| **UTF-32** | 4 | All Unicode | Processing efficiency |
| **GB18030** | 1–4 | All Unicode + Chinese extensions | Chinese government standard |
| **Shift-JIS** | 1–2 | Japanese | Legacy Japanese systems |

### Unicode Normalisation Forms

The same visual character can have multiple Unicode representations:

```
"é"  can be:
  U+00E9          → LATIN SMALL LETTER E WITH ACUTE  (precomposed, NFC)
  U+0065 U+0301   → e + COMBINING ACUTE ACCENT       (decomposed, NFD)
```

The four normalisation forms:
| Form | Description | Use in NLP |
|------|-------------|-----------|
| **NFC** | Canonical decomposition, then canonical composition | Standard for text storage; use for most NLP |
| **NFD** | Canonical decomposition only | Useful for diacritic stripping |
| **NFKC** | Compatibility decomposition + composition | Fold compatibility variants (ﬁ→fi, ² →2, ａ→a) |
| **NFKD** | Compatibility decomposition only | Most aggressive normalisation |

**NFKC** is the most commonly used in NLP preprocessing — it collapses ligatures, full-width characters, and other compatibility variants into their canonical forms.

---

## Math / Formal Notation

### UTF-8 Encoding Scheme

The encoding of a code point $U$ into bytes follows:

| Code point range | Byte 1 | Byte 2 | Byte 3 | Byte 4 |
|-----------------|--------|--------|--------|--------|
| U+0000–U+007F | `0xxxxxxx` | — | — | — |
| U+0080–U+07FF | `110xxxxx` | `10xxxxxx` | — | — |
| U+0800–U+FFFF | `1110xxxx` | `10xxxxxx` | `10xxxxxx` | — |
| U+10000–U+10FFFF | `11110xxx` | `10xxxxxx` | `10xxxxxx` | `10xxxxxx` |

where `x` bits are filled with the binary representation of $U$.

**Example**: encode `é` (U+00E9 = 233 decimal = `11101001` binary):
- Falls in U+0080–U+07FF range → 2 bytes
- Template: `110xxxxx 10xxxxxx`
- U+00E9 = `000 11101001` → split as `00011` `101001`
- Bytes: `11000011 10101001` = `0xC3 0xA9`

**Detecting encoding errors programmatically**:

```python
import unicodedata

def has_encoding_issues(text):
    for char in text:
        # Replacement character — indicates failed decode
        if char == '\ufffd':
            return True
        # Control characters (except common whitespace)
        cat = unicodedata.category(char)
        if cat == 'Cc' and char not in '\t\n\r':
            return True
    return False
```

---

## Examples (Concrete)

### Common Encoding Bugs

**1. Mojibake (UTF-8 read as Latin-1)**
```
Correct UTF-8:  "I can't believe it's not butter"
As Latin-1:     "I canâ€™t believe itâ€™s not butter"

Fix:
text.encode('latin-1').decode('utf-8')
```

**2. BOM (Byte Order Mark) at file start**
```
File bytes: EF BB BF 48 65 6C 6C 6F
Decoded:    \ufeffHello        ← \ufeff is the BOM
Clean:      "Hello".lstrip('\ufeff')
```

**3. HTML entities not decoded**
```
Raw:    "&lt;p&gt;Hello &amp; goodbye&lt;/p&gt;"
Fixed:  import html; html.unescape(raw)
→       "<p>Hello & goodbye</p>"
```

**4. Full-width characters (common in East Asian web text)**
```
Raw:   "Ｈｅｌｌｏ　Ｗｏｒｌｄ"  (full-width ASCII + ideographic space)
NFKC:  "Hello World"              (normalised to ASCII)

import unicodedata
unicodedata.normalize('NFKC', raw)
```

**5. Zero-width characters (invisible, break tokenisation)**
```
"word​word"   ← contains U+200B ZERO WIDTH SPACE between the d and w
Looks like one word; tokeniser splits it as two.
Fix: strip all characters in Unicode category Cf (format chars)
```

### Python Encoding Toolkit

```python
import unicodedata
import ftfy

# Fix encoding issues automatically
clean = ftfy.fix_text(broken_text)

# Unicode normalise
nfc  = unicodedata.normalize('NFC', text)
nfkc = unicodedata.normalize('NFKC', text)

# Strip diacritics (NFD then remove combining marks)
def strip_diacritics(text):
    nfd = unicodedata.normalize('NFD', text)
    return ''.join(c for c in nfd
                   if unicodedata.category(c) != 'Mn')
# strip_diacritics("café") → "cafe"

# Remove all non-printable characters
def remove_control_chars(text):
    return ''.join(c for c in text
                   if unicodedata.category(c)[0] != 'C'
                   or c in '\t\n\r')
```

---

## How It Connects to ML / NLP

| Encoding Concept | NLP/ML Impact |
|---|---|
| UTF-8 variable width | Byte-level models (CANINE, ByT5) process raw bytes |
| Unicode normalisation | Tokenizer vocabulary size; character coverage |
| Full-width collapse (NFKC) | Prevents vocabulary explosion in East Asian text |
| Combining characters | Tokenizer split points; character count vs byte count |
| BOM / control chars | Corrupt first tokens; silence in byte-level models |
| Encoding mismatch | Incorrect characters → wrong tokens → wrong predictions |

**Byte-level NLP**: models like ByT5, CANINE, and GPT-2/3/4's tokenizer work at the UTF-8 byte level. Understanding UTF-8 encoding directly explains why these models have a vocabulary of 256 (one per byte value) and why they handle multilingual text without a separate tokenizer.

**BERT / WordPiece**: WordPiece operates on Unicode characters after NFKC normalisation + accent stripping (for multilingual BERT). The tokenizer documentation explicitly lists these preprocessing steps — encoding normalisation is baked into the tokenizer.

**SentencePiece**: also applies Unicode normalisation (configurable: NFC or NFKC) before learning the subword vocabulary. Consistent normalisation during training and inference is essential — a mismatch causes tokens to be split differently.

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Features and Representation/Data Representation.md]] — encoding determines the atomic units of representation
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — encoding bugs silently corrupt training data

---

## Common Interview Questions

**Q: Why is Unicode normalisation important before tokenisation?**
A: The same visual character (e.g. "é") can be represented as one code point (NFC precomposed) or two code points (NFD: base `e` + combining accent). Without normalisation, the same word in different normalisation forms produces different token sequences — hurting model generalisation. NFKC also collapses full-width characters (common in East Asian web text) to ASCII equivalents, preventing vocabulary explosion.

**Q: What is the difference between UTF-8 and UTF-16, and why does it matter for NLP?**
A: UTF-8 uses 1–4 bytes per character and is ASCII-compatible; UTF-16 uses 2 or 4 bytes and requires a BOM to specify byte order. UTF-8 dominates the web (~97% of web pages). UTF-16 is used internally by Java, JavaScript, and Windows APIs. Strings in Python 3 are Unicode (not any specific encoding) — encoding only matters when reading/writing bytes. The practical NLP implication: always specify `encoding='utf-8'` when opening files; never assume the system default.

**Q: How does UTF-8 encoding affect byte-level language models?**
A: Byte-level models (ByT5, CANINE, GPT tokenizer) treat each byte as a token. UTF-8's variable-width encoding means one character can be 1–4 bytes — so a Chinese character (3 bytes in UTF-8) becomes 3 tokens in a byte-level model. This makes byte-level models less efficient for non-ASCII text: the same content requires more tokens, increasing sequence length and compute. This is the trade-off vs. character-level or subword models.

---

## Common Mistakes / Gotchas

- **Forgetting to specify encoding when reading files**: `open('file.txt')` uses the system default encoding (often ASCII on some systems), which will fail or corrupt data on non-ASCII text. Always use `open('file.txt', encoding='utf-8')`.
- **Applying regex before Unicode normalisation**: a regex for `[a-zA-Z]` won't match `ａ` (full-width A, U+FF41) — normalise to NFKC first.
- **Assuming `len(text)` = number of characters**: in Python 3, `len(text)` returns code points, not bytes. But some characters are combining marks that visually merge — `len("é")` = 2 if NFD-encoded (base + combining), even though it looks like 1 character. For precise character counting, use NFC first.
- **Treating UTF-16 surrogates as valid characters**: code points U+D800–U+DFFF are surrogate pairs used internally in UTF-16. They are not valid characters in UTF-8 or in Python strings. Their presence indicates a UTF-16 → UTF-8 conversion bug.
- **Not handling BOM in CSV/TSV files**: files exported from Excel often have a UTF-8 BOM (`\ufeff`). This silently prepends a special character to the first field, breaking string comparisons. Use `encoding='utf-8-sig'` in Python to strip it automatically.

---

## Further Reading / Paper References

- The Unicode Standard: https://www.unicode.org/standard/standard.html — official reference
- UTF-8 RFC 3629: https://tools.ietf.org/html/rfc3629
- `ftfy` (Fix Text For You): https://ftfy.readthedocs.io — encoding repair library
- Xue, L. et al. (2022). *ByT5: Towards a Token-Free Future with Pre-trained Byte-to-Byte Models.* [[arxiv:2105.13626]] — byte-level NLP
- Clark, J.H. et al. (2022). *CANINE: Pre-training an Efficient Tokenization-Free Encoder.* [[arxiv:2103.06874]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — text normalisation
