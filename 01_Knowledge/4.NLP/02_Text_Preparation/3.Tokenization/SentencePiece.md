# SentencePiece

tags: #nlp #tokenization #subword #sentencepiece #multilingual
links: [[Tokenization — BPE]] [[WordPiece and Unigram LM]] [[Tokenization for CJK Languages]] [[Agglutinative Languages]]

---

## Definition + Intuition

**SentencePiece** (Kudo & Richardson, 2018) is a language-independent subword tokenizer and detokenizer. Unlike BPE and WordPiece — which require pre-tokenized (whitespace-split) input — SentencePiece treats the input as a raw character sequence including spaces, enabling truly language-agnostic tokenization.

> **Intuition**: BPE assumes you can split on spaces first, then apply subword rules within words. That breaks for Japanese (no spaces), Thai, Tibetan, and many other languages. SentencePiece treats the space as just another character (`▁`) and learns from raw text — the space naturally becomes part of the vocabulary (e.g. `▁the`, `▁is`), and word boundaries are implicit. No language-specific pre-processing needed.

---

## Key Properties / Types

### Key Design Decisions

**1. No pre-tokenization**: input is raw text; spaces are treated as regular characters (encoded as `▁` U+2581).

**2. Two algorithm choices**: BPE or Unigram LM (both implemented; Unigram LM is default and generally preferred).

**3. Lossless tokenization**: original text can be recovered exactly from the token sequence (detokenization is trivial: replace `▁` with space).

**4. Language-agnostic**: same code/config for English, Japanese, Arabic, Thai, etc.

**5. Vocabulary includes**:
- Learned subword units
- Single characters (fallback for rare characters)
- Special tokens: `<unk>`, `<s>`, `</s>` (and user-defined specials)
- Byte fallback tokens (optional): `<0xE3>` etc. for truly OOV bytes

### Space as a Character — The `▁` Convention

```
Input:  "Hello world"
         ↓
Treated as: "▁Hello▁world"

Tokenization might give:
  ["▁Hello", "▁world"]     ← if both are in vocabulary
  ["▁He", "llo", "▁wor", "ld"]  ← if split

The ▁ marks "this token starts a new word (follows a space)"
```

This is the **inverse** of WordPiece's `##` convention:
- WordPiece: `##ing` means "continuation" (non-initial)
- SentencePiece: `▁ing` means "word-initial" (following a space)

### Training SentencePiece

```python
import sentencepiece as spm

spm.SentencePieceTrainer.train(
    input='corpus.txt',
    model_prefix='my_model',
    vocab_size=32000,
    character_coverage=0.9995,  # fraction of chars to cover (use 1.0 for CJK)
    model_type='unigram',       # or 'bpe'
    pad_id=0, unk_id=1, bos_id=2, eos_id=3,
)

sp = spm.SentencePieceProcessor()
sp.load('my_model.model')
sp.encode("Hello world", out_type=str)
# → ['▁Hello', '▁world']
sp.encode("Hello world", out_type=int)
# → [3, 10]  ← vocabulary IDs
```

### Models Using SentencePiece

| Model | Algorithm | Vocab Size |
|-------|-----------|-----------|
| T5 | Unigram | 32,000 |
| XLM-R | BPE | 250,002 |
| mBART | BPE | 250,000 |
| ALBERT | Unigram | 30,000 |
| LLaMA | BPE | 32,000 |
| Gemma | BPE | 256,000 |

---

## Math / Formal Notation

### Lossless Encoding

SentencePiece guarantees: for any text $s$ and its token sequence $\mathbf{t} = \text{encode}(s)$:

$$\text{decode}(\mathbf{t}) = s$$

by representing spaces as `▁` within tokens. This is not guaranteed by WordPiece without additional post-processing.

### Character Coverage Parameter

The `character_coverage` parameter $\alpha$ controls what fraction of unique characters to include in the vocabulary:

$$|\{c \in \mathcal{V} : c \text{ is a single char}\}| / |\Sigma| \geq \alpha$$

For alphabetic scripts (Latin, Cyrillic): `character_coverage=0.9995` — only the most frequent characters; rare ones become `<unk>`.

For CJK scripts: `character_coverage=1.0` — every character should be covered since each Chinese/Japanese character has distinct meaning.

---

## Examples (Concrete)

```python
# Japanese (no spaces in original)
sp.encode("東京は日本の首都です", out_type=str)
# → ['▁東京', 'は', '日本', 'の', '首都', 'です']
# SentencePiece found "東京" (Tokyo) and "日本" (Japan) as vocabulary units

# Arabic
sp.encode("مرحبا بالعالم", out_type=str)
# → ['▁مرح', 'با', '▁بالع', 'الم']
# Right-to-left text handled transparently

# Code-switching (English + Spanish)
sp.encode("Me gusta NLP and machine learning", out_type=str)
# → ['▁Me', '▁gusta', '▁NLP', '▁and', '▁machine', '▁learning']
```

**Subword regularization sampling:**
```python
# Multiple tokenizations during training (Unigram LM only)
for _ in range(5):
    print(sp.sample_encode_as_pieces("tokenization", -1, 0.1))
# → ['▁token', 'ization']
# → ['▁token', 'iz', 'ation']
# → ['▁t', 'oken', 'ization']
# → ['▁tokenization']          ← sometimes whole word
# → ['▁token', 'i', 'z', 'a', 'tion']
```

---

## How It Connects to ML / NLP

SentencePiece is the go-to tokenizer for multilingual and generative models:
- **T5, mT5**: SentencePiece with Unigram LM; used for text-to-text generation across 101 languages
- **LLaMA/Llama-2**: SentencePiece BPE; vocabulary designed for code + multilingual text
- **Cross-lingual transfer**: shared SentencePiece vocabulary across languages allows cross-lingual embeddings to directly share token IDs for cognates and loanwords

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — subword regularization as data augmentation
- [[Tokenization — BPE]] — SentencePiece implements BPE as one option
- [[Tokenization for CJK Languages]] — SentencePiece handles CJK naturally

---

## Common Interview Questions

**Q: What is the main advantage of SentencePiece over BPE/WordPiece for multilingual NLP?**
A: SentencePiece requires no language-specific pre-tokenization. BPE and WordPiece need text to be whitespace-tokenized first — this works for space-delimited languages but fails for Chinese, Japanese, and Thai (no spaces) and produces unnatural splits for agglutinative languages. SentencePiece treats raw bytes/characters as input, learning word boundaries from the data. This makes it truly language-agnostic and the standard for multilingual models (XLM-R, mT5, LLaMA).

**Q: How does SentencePiece's `▁` prefix differ from BERT's `##` prefix?**
A: They encode opposite things. BERT's `##` marks **continuations** (non-initial subwords): `["play", "##ing"]` means `ing` follows `play`. SentencePiece's `▁` marks **word starts** (tokens that follow a space): `["▁Hello", "▁world"]` means both are word-initial. The `▁` convention makes detokenization trivial (replace `▁` with space) and naturally handles any language since spaces are learned rather than assumed.

---

## Common Mistakes / Gotchas

- **Not setting `character_coverage=1.0` for CJK**: with default 0.9995 coverage, rare Chinese characters become `<unk>`. For Chinese NLP, always use 1.0.
- **Forgetting `add_bos=True` / `add_eos=True`** when encoding sequences for models that expect `<s>` / `</s>` boundary tokens (T5, LLaMA).
- **Comparing SentencePiece and WordPiece token IDs**: they use different vocabulary files; IDs are not interchangeable.

---

## Further Reading / Paper References

- Kudo, T. & Richardson, J. (2018). *SentencePiece: A simple and language independent subword tokenizer.* [[arxiv:1808.06226]]
- Kudo, T. (2018). *Subword Regularization.* ACL. [[arxiv:1804.10959]]
- SentencePiece GitHub: https://github.com/google/sentencepiece
