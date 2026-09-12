# Tokenization — BPE (Byte Pair Encoding)

tags: #nlp #tokenization #subword #bpe #vocabulary
links: [[Whitespace Tokenization]] [[Rule-Based Tokenizers]] [[WordPiece and Unigram LM]] [[SentencePiece]] [[Tokenization for CJK Languages]] [[Morphemes — Free and Bound]]

---

## Definition + Intuition

**Byte Pair Encoding (BPE)** is a data-compression algorithm adapted for NLP subword tokenization. Starting from individual characters, it iteratively merges the most frequent adjacent pair of symbols, building a vocabulary of subword units up to a target size.

The resulting tokenizer can represent any word as a sequence of subword tokens — including words never seen during training — making OOV tokens essentially impossible.

> **Intuition**: BPE starts by treating every character as a separate "word." Then it notices that `t` and `h` always appear together and merges them into `th`. Then `th` and `e` merge into `the`. Eventually common words and morphemes become single tokens (`the`, `ing`, `un`), while rare words are split into recognisable subparts. It discovers approximately-morphological units purely from co-occurrence statistics — no linguistic knowledge needed.

---

## Key Properties / Types

### The BPE Algorithm

**Training phase:**

1. **Initialise vocabulary** with all unique characters in the corpus + `</w>` end-of-word marker
2. **Tokenise corpus** at character level: `"low"` → `l o w </w>`
3. **Count all adjacent symbol pairs** across the corpus
4. **Merge the most frequent pair** → add merged symbol to vocabulary
5. **Repeat** steps 3–4 until vocabulary reaches target size $|V|$

```
Corpus: "low low lowest newer newer wider"

Initial tokens: l o w </w>  l o w </w>  l o w e s t </w>  n e w e r </w>  ...

Iteration 1: most frequent pair = (e, r) → merge to "er"
  vocab adds: "er"
  tokens: ... n ew er </w> ...   ← wait, let's redo properly

Actually tracking:
  pair counts: (l,o)=3, (o,w)=3, (w,</w>)=3, (e,w)=3, (w,e)=4, (e,r)=2 ...
  
  most frequent: (e,r)=2 (actually we track all)
  ... after several merges:
  "low"    → ["low</w>"]
  "newest" → ["new", "est</w>"]
  "newer"  → ["new", "er</w>"]
  "wider"  → ["wid", "er</w>"]
```

**Inference phase** (tokenising new text):
Apply learned merge rules in the same order they were learned (greedy, left-to-right):

```
New word: "lowest"
Start: l o w e s t </w>
Apply merges in order until no more apply:
  (l,o) → lo → "lo w e s t </w>"
  (lo,w) → low → "low e s t </w>"
  (e,s) → es → "low es t </w>"
  (es,t) → est → "low est </w>"
  (est,</w>) → "est</w>" → "low est</w>"
Result: ["low", "est</w>"] → ["low", "est"]
```

### Vocabulary Size Trade-offs

| Vocabulary Size | Effect |
|----------------|--------|
| Small (~1k) | Many splits; long sequences; better OOV handling |
| Medium (~32k) | Common words are single tokens; most English words covered |
| Large (~100k) | Most words unsplit; less morphological sharing; larger embedding table |

**Typical vocabulary sizes:**
- GPT-2: 50,257
- GPT-4: ~100,000
- BERT (uncased): 30,522
- XLM-R: 250,002 (multilingual)

### BPE Variants

**Original BPE** (Sennrich et al. 2016): word-level pre-tokenization + character-level BPE within words.

**Byte-level BPE** (GPT-2, GPT-3, GPT-4): runs BPE on raw UTF-8 bytes (256 base units), not characters. Handles any Unicode text with a fixed base vocabulary. Never produces true OOV.

**SentencePiece BPE**: no pre-tokenization; treats spaces as characters; language-agnostic. See [[SentencePiece]].

---

## Math / Formal Notation

### BPE Merge Rule Learning

Let $\mathcal{D}$ be a corpus of word-frequency pairs: $\{(w_i, f_i)\}$.

At each iteration $t$, the pair score is simply the count:

$$\text{score}(a, b) = \sum_{w : (a,b) \subseteq \tau_t(w)} f_w$$

where $\tau_t(w)$ is the current tokenisation of word $w$ at iteration $t$.

The greedy merge:

$$m^* = \arg\max_{(a,b)} \text{score}(a, b)$$

Add $m^* = ab$ to vocabulary $\mathcal{V}$; apply merge throughout corpus.

**Total merges**: if starting vocabulary = $|\Sigma|$ (characters), and target vocabulary = $|V|$, then number of merges $= |V| - |\Sigma|$.

### Compression Ratio

BPE was originally a data compression algorithm (Philip Gage, 1994). The compression ratio on text:

$$\text{compression} = \frac{|\text{tokens after BPE}|}{|\text{characters before BPE}|}$$

A ratio of 0.25 means 4 characters per token on average — typical for English with 32k vocabulary.

### Tokenization Fertility

**Fertility** of a word = number of subword tokens it is split into. High fertility → word is rare/OOV-like.

For NLP tasks, average token/word ratio across a dataset is a useful diagnostic:
- English (GPT tokenizer, 32k vocab): ~1.3 tokens/word
- Turkish (same tokenizer): ~3.5 tokens/word ← vocabulary not designed for Turkish

---

## Examples (Concrete)

### Step-by-Step BPE Training

```
Corpus (simplified): 
  "aab" × 3, "ab" × 2, "b" × 1
  
Initial: a a b  (×3),  a b  (×2),  b  (×1)

Pair counts:
  (a,a): 3    (a,b): 3+2=5    (b,END): 3+2+1=6

Merge (b,END) → b$:  a a b$  (×3),  a b$  (×2),  b$  (×1)

Pair counts:
  (a,a): 3    (a,b$): 3+2=5

Merge (a,b$) → ab$: a ab$  (×3),  ab$  (×2),  b$  (×1)

Pair counts:
  (a,ab$): 3

Merge (a,ab$) → aab$: aab$  (×3),  ab$  (×2),  b$  (×1)

Final vocabulary: {a, b, b$, ab$, aab$}
```

### Real Tokenization Comparison

```
Word: "tokenization"

Character-level:  t-o-k-e-n-i-z-a-t-i-o-n  (12 tokens)
BPE (32k):        token-ization              (2 tokens — GPT tokenizer)
BPE (1k):         t-ok-en-iz-ation           (5 tokens)
Word-level:       tokenization               (1 token — if in vocab)
```

```python
# Hugging Face tokenizers
from transformers import GPT2Tokenizer
tok = GPT2Tokenizer.from_pretrained("gpt2")

tok.tokenize("tokenization")       # → ['token', 'ization']
tok.tokenize("antidisestablishment") # → ['ant', 'id', 'ise', 'st', 'ablishment']
tok.tokenize("COVID-19")           # → ['C', 'OV', 'ID', '-', '19']
tok.tokenize("🎉party")            # → ['ð', 'Ł', 'Ĳ', 'party']  ← byte-level emoji
```

---

## How It Connects to ML / NLP

| BPE Property | Impact on NLP Models |
|---|---|
| Subword units | Shared representations across morphologically related words |
| Fixed vocabulary | Finite output space → softmax over vocabulary is well-defined |
| No OOV tokens | Rare words always get a tokenisation (though possibly many subwords) |
| Language-agnostic | Same algorithm for any language |
| Vocabulary size hyperparameter | Controls granularity; affects model capacity needed |

**Why BPE improves NLP over word-level:**
- `"run"`, `"running"`, `"runner"` share the subword `"run"` → parameter sharing
- Rare words like `"antidisestablishmentarianism"` are tokenised into recognisable morphemes → model can process them without ever seeing the full form
- Multilingual models can share subwords across languages (`"in"` shared between English and German)

**Byte-level BPE advantage**: handles emoji, code, and any Unicode without preprocessing. GPT-3/4 uses byte-level BPE — `🎉` is encoded as its UTF-8 bytes, which are part of the 256-symbol base vocabulary.

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Vectorization.md]] — vocabulary determines the embedding table dimensions
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — subword sharing improves generalisation to unseen word forms
- [[Morphemes — Free and Bound]] — BPE approximates morpheme segmentation

---

## Common Interview Questions

**Q: What is the difference between BPE tokenization and character-level tokenization?**
A: Character-level splits every word into individual characters — tiny vocabulary (~100 symbols), very long sequences, no sharing of morphological information across words. BPE builds up from characters but merges frequent pairs, creating subword units that often correspond to meaningful morphemes. BPE gives shorter sequences than character-level (more efficient) while still handling OOV words and sharing subword representations.

**Q: Why does the order of BPE merges matter at inference time?**
A: BPE merges are applied greedily in the order they were learned. Applying them in a different order could produce different tokenisations. The learned order encodes frequency priority — common subwords like `"the"` are formed early and thus take precedence over longer subwords that contain them. At inference, the standard approach is to apply all merges that can be applied, in order, giving a deterministic tokenisation.

**Q: What is byte-level BPE and why do GPT models use it?**
A: Byte-level BPE uses the 256 raw UTF-8 byte values as the base vocabulary, rather than characters. This means every possible input (including emoji, Chinese characters, code, binary data) can be tokenised without any OOV symbols. GPT-2 introduced this for robustness to arbitrary Unicode text. The trade-off is that non-ASCII characters (e.g. Chinese characters, emoji) need multiple bytes (tokens) each, making Asian-language text relatively longer in token count.

---

## Common Mistakes / Gotchas

- **Applying training-time merge rules at inference differently**: always apply merges in the exact order they were learned. Different BPE libraries (Hugging Face `tokenizers`, SentencePiece, OpenNMT) may differ in edge cases — use the same library for training and inference.
- **Comparing model outputs across different tokenizers**: a model trained with GPT-2 tokenizer cannot directly use BERT tokenizer's output. Vocabulary IDs are not interchangeable.
- **Expecting BPE to learn morphology**: BPE learns frequency-based merges, not linguistic morphology. `"walked"` might tokenise as `["walk", "ed"]` (morphologically correct) or `["wal", "ked"]` (wrong) depending on what the corpus contains.
- **Forgetting the space prefix convention**: GPT tokenizer represents `"word"` at start of a sentence as `" word"` (with a leading space). This is how BPE distinguishes mid-word subwords from word-initial ones. Many bugs arise from ignoring this.
- **Very long words crashing inference**: pathological inputs with very long strings (e.g. DNA sequences, minified JavaScript) produce very long token sequences. Set a maximum sequence length guard before tokenisation.

---

## Further Reading / Paper References

- Gage, P. (1994). *A New Algorithm for Data Compression.* C Users Journal. — original BPE
- Sennrich, R., Haddow, B. & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units.* ACL. [[arxiv:1508.07909]] — BPE for NLP
- Radford, A. et al. (2019). *Language Models are Unsupervised Multitask Learners.* (GPT-2) — byte-level BPE
- Hugging Face tokenizers: https://huggingface.co/docs/tokenizers
- Jurafsky & Martin, *Speech and Language Processing* Ch. 2 — subword tokenisation
