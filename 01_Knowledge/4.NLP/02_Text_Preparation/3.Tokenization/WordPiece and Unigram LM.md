# WordPiece and Unigram LM

tags: #nlp #tokenization #subword #wordpiece #unigram #bert
links: [[Tokenization — BPE]] [[SentencePiece]] [[Whitespace Tokenization]] [[Rule-Based Tokenizers]]

---

## Definition + Intuition

**WordPiece** and **Unigram Language Model** are two alternative subword tokenization algorithms to BPE. All three learn a subword vocabulary from data, but they differ in how they select which subwords to include and how they tokenize new text.

| Algorithm | Selection Criterion | Tokenization | Used By |
|-----------|-------------------|-------------|---------|
| **BPE** | Merge most frequent pair | Greedy, deterministic | GPT-2/3/4, RoBERTa |
| **WordPiece** | Merge that maximises LM likelihood | Greedy, deterministic | BERT, DistilBERT |
| **Unigram LM** | Keep subwords that minimise loss | Viterbi (optimal) or sampling | SentencePiece, XLM-R, T5 |

> **Intuition**:
> - **BPE**: "Merge whatever two symbols appear most often together" — purely frequency-driven.
> - **WordPiece**: "Merge whatever two symbols, when merged, best help predict the next symbol" — likelihood-driven, slightly smarter.
> - **Unigram LM**: Start with a huge vocabulary, prune the least useful symbols until you reach the target size. Finds the optimal split for new words via Viterbi.

---

## Key Properties / Types

### WordPiece

**Developed by**: Google for Japanese/Korean segmentation; later used for BERT.

**Core difference from BPE**: instead of merging the most frequent pair, WordPiece merges the pair that maximises the **language model likelihood** of the corpus:

$$\text{score}(A, B) = \frac{\text{count}(AB)}{\text{count}(A) \times \text{count}(B)}$$

This is the **pointwise mutual information (PMI)** — it favors merging pairs that co-occur more than expected by chance, not just pairs that are frequent overall.

**WordPiece convention**: subwords that are not word-initial are prefixed with `##`:
```
"tokenization" → ["token", "##iza", "##tion"]
"playing"      → ["play", "##ing"]
"unbelievable" → ["un", "##believe", "##able"]
```

This lets the model distinguish `"ing"` (a word) from `"##ing"` (a suffix).

**BERT vocabulary construction**:
1. Initialise with all characters + special tokens
2. Apply WordPiece merges until vocabulary reaches 30,000 (uncased) or 28,996 (cased)
3. Apply NFKC Unicode normalisation + accent stripping (for multilingual BERT)

### Unigram Language Model Tokenizer

**Developed by**: Kudo (2018), implemented in SentencePiece.

**Key idea**: frame tokenization as a probabilistic language model over subword sequences.

**Training**:
1. Start with a large vocabulary (all substrings up to length $L$ + all characters)
2. Assign probabilities to each subword via EM
3. **Prune**: remove the $p\%$ of subwords whose removal increases corpus loss the least
4. Repeat until vocabulary reaches target size

**Tokenization**: use the **Viterbi algorithm** to find the most likely segmentation:

$$x^* = \arg\max_{x \in S(X)} \prod_{i=1}^{|x|} P(x_i)$$

where $S(X)$ is all possible segmentations of input $X$ and $P(x_i)$ is the unigram probability of subword $x_i$.

**Subword regularization**: at training time, sample from the distribution of valid segmentations (not just the Viterbi best). This data augmentation exposes the model to multiple tokenizations of the same word → better robustness.

---

## Math / Formal Notation

### WordPiece Score

For current vocabulary $\mathcal{V}$ and candidate merge $(A, B)$:

$$\text{score}_\text{WP}(A, B) = \log P(AB) - \log P(A) - \log P(B) = \text{PMI}(A, B)$$

where probabilities are estimated from the current corpus:

$$P(t) = \frac{\text{count}(t)}{\sum_{t' \in \mathcal{V}} \text{count}(t')}$$

Merging $A$ and $B$ into $AB$ is beneficial iff $\text{PMI}(A, B) > 0$ — i.e., they co-occur more than independently.

### Unigram LM Tokenizer

**Vocabulary probability assignment (EM)**:

E-step: for each word $X$ in corpus, compute marginal probability of each subword $x_i$ summed over all segmentations:

$$\gamma(x_i) = \sum_{x : x_i \in x} P(x \mid X)$$

M-step: update probabilities:

$$P(x_i) \propto \sum_{X \in \mathcal{D}} f(X) \cdot \gamma(x_i)$$

where $f(X)$ = frequency of word $X$.

**Pruning criterion**: for each subword $t$, compute:

$$\Delta L(t) = \mathcal{L}(\mathcal{V}) - \mathcal{L}(\mathcal{V} \setminus \{t\})$$

Remove subwords with smallest $\Delta L$ (removing them hurts the corpus likelihood the least). Single characters always kept to ensure coverage.

**Total corpus log-likelihood** (objective being maximised):

$$\mathcal{L}(\mathcal{V}) = \sum_{X \in \mathcal{D}} f(X) \log P^*(X)$$

where $P^*(X) = \max_{x \in S(X)} \prod_i P(x_i)$ is the Viterbi probability.

---

## Examples (Concrete)

### WordPiece vs BPE Comparison

```
Word: "unhappiness"

BPE (GPT-2 tokenizer):
  ["un", "happiness"]   ← frequency-driven merge

WordPiece (BERT tokenizer):
  ["un", "##happ", "##iness"]  ← note ## prefix for non-initial subwords
  or possibly:
  ["un", "##happy", "##ness"]  ← depends on vocabulary

Unigram LM (SentencePiece/T5):
  ["▁un", "happiness"]   ← ▁ marks word-initial (instead of ##)
  or: ["▁unhappiness"]   ← if this whole word is in vocabulary
```

**BERT-specific WordPiece examples:**
```python
from transformers import BertTokenizer
tok = BertTokenizer.from_pretrained("bert-base-uncased")

tok.tokenize("tokenization")
# → ["token", "##ization"]

tok.tokenize("playing")
# → ["play", "##ing"]

tok.tokenize("COVID-19")
# → ["co", "##vid", "-", "19"]   ← lowercased because uncased model

tok.tokenize("unbelievable")
# → ["un", "##believable"]       ← "un" is in vocab; "believable" has prefix

tok.tokenize("Huggingface")
# → ["hugging", "##face"]        ← fortuitous split matching the brand!
```

### Subword Regularization (Unigram)

For word `"hello"`, instead of always using the Viterbi split `["▁hello"]`, during training sample alternative splits:
```
["▁h", "ello"]           P = 0.05
["▁he", "llo"]           P = 0.03
["▁hel", "lo"]           P = 0.04
["▁hell", "o"]           P = 0.05
["▁hello"]               P = 0.83  ← Viterbi best

Sample from this distribution during training → model sees all splits
```

This is equivalent to data augmentation and improves robustness: the model learns that `"hell"` + `"o"` and `"he"` + `"llo"` should have the same meaning as `"hello"`.

---

## How It Connects to ML / NLP

| Algorithm | Key Advantage | Key Disadvantage |
|-----------|--------------|-----------------|
| BPE | Simple, fast, deterministic | Greedy; not linguistically motivated |
| WordPiece | PMI-based; slightly better morphological units | Requires LM training; `##` convention |
| Unigram LM | Optimal (Viterbi); supports sampling; probabilistic | Slower to train; more complex |

**BERT tokenization pipeline:**
1. Lowercase (for uncased model)
2. Apply NFKC Unicode normalisation + accent stripping
3. Whitespace + punctuation splitting
4. Apply WordPiece merges

**XLM-R / T5 tokenization pipeline:**
1. SentencePiece with Unigram LM
2. No pre-tokenization (space is treated as a character → `▁`)
3. No Unicode normalisation (NFC only)

**Cross-links:**
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — Unigram LM is trained to maximise log-likelihood
- [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — EM for Unigram LM is an optimisation algorithm
- [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Vectorization.md]] — vocabulary → embedding table dimensions

---

## Common Interview Questions

**Q: What is the key algorithmic difference between BPE and WordPiece?**
A: BPE merges the pair with the highest raw frequency count. WordPiece merges the pair with the highest PMI score: `count(AB) / (count(A) * count(B))`. PMI normalises for how common each subword is independently, so WordPiece tends to merge pairs that are strongly associated (even if not the absolute most frequent), leading to slightly more linguistically meaningful subwords. In practice, outputs are very similar for English.

**Q: Why does BERT use `##` prefixes for subwords?**
A: The `##` prefix (WordPiece convention) indicates that a subword token is not word-initial — it must follow another token without a space. This lets the model distinguish `"##ing"` (verbal suffix, always follows a root) from `"ing"` (gerund used as a standalone word, as in "the thing"). Without this distinction, the embedding for the suffix and the word would be the same, losing positional information.

**Q: What is subword regularization and why does it help?**
A: Subword regularization (Kudo 2018) samples from multiple valid tokenizations of the same word during training, rather than always using the best (Viterbi) tokenization. This exposes the model to different segmentations of the same word and forces it to learn that they should have equivalent meaning — a form of data augmentation. It improves robustness and has been shown to help especially in low-resource and MT settings.

---

## Common Mistakes / Gotchas

- **Mixing BERT and GPT tokenizers**: BERT uses WordPiece with `##` prefix; GPT uses BPE with space prefix. They produce different tokenizations, different vocab IDs. Never use one model's tokenizer with another model's weights.
- **Forgetting `##` when reconstructing text from BERT tokens**: `["play", "##ing"]` reconstructs to `"playing"`, not `"play ##ing"`. Use `tokenizer.convert_tokens_to_string(tokens)` rather than `' '.join(tokens)`.
- **Assuming Unigram LM always gives the same tokenization**: the Viterbi path is deterministic, but subword regularization at training time samples stochastically. At inference, always use the Viterbi (greedy) path for reproducibility.
- **Tokenizing special tokens**: `[CLS]`, `[SEP]`, `[PAD]`, `[MASK]` are special tokens in the BERT vocabulary. They should be added after tokenization via the tokenizer's `add_special_tokens=True` argument, not before (or they'll be split into subwords).

---

## Further Reading / Paper References

- Schuster, M. & Nakamura, K. (2012). *Japanese and Korean Voice Search.* ICASSP. — original WordPiece
- Wu, Y. et al. (2016). *Google's Neural Machine Translation System.* [[arxiv:1609.08144]] — WordPiece in GNMT
- Devlin, J. et al. (2019). *BERT: Pre-training of Deep Bidirectional Transformers.* [[arxiv:1810.04805]] — BERT WordPiece
- Kudo, T. (2018). *Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates.* ACL. [[arxiv:1804.10959]] — Unigram LM + regularization
- Kudo, T. & Richardson, J. (2018). *SentencePiece: A simple and language independent subword tokenizer.* [[arxiv:1808.06226]]
