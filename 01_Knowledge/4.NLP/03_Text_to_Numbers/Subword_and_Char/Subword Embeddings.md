---
tags: [nlp, subword-char, subword-embeddings, bpe, wordpiece, sentencepiece, tokenization, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Character Embeddings]] [[FastText]] [[Tokenization — BPE]] [[WordPiece and Unigram LM]] [[SentencePiece]] [[BERT Embeddings]]"
---

# Subword Embeddings

## Definition + Intuition

**Subword embeddings** are the dense vector representations assigned to subword units — tokens produced by algorithms like BPE, WordPiece, or SentencePiece. Every modern Transformer model (BERT, GPT, T5, LLaMA) operates entirely at the subword level. The **embedding layer** maps each subword token ID to a dense vector; this is always the first learned operation in any Transformer.

**Intuition**: Instead of choosing between word-level (can't handle OOV) and character-level (loses semantic identity), subword tokenisation finds a middle ground: split rare words into smaller pieces, keep common words intact. "playing" might be one token; "immunocompromised" might be split into "immune", "##o", "##compromised". Each piece gets its own embedding vector.

The key insight: the embedding layer is not just a preprocessing step — it is the first and most parameter-dense layer of a Transformer for small models. Learning good subword embeddings is learning the fundamental lexical building blocks the model works with.

---

## Key Properties / Types

**The embedding layer as a lookup table**:
$$\text{Embed}(t) = E[t] \in \mathbb{R}^d \quad \text{where } E \in \mathbb{R}^{|V| \times d}$$

- $|V|$ = vocabulary size (BERT: 30,522; GPT-4: ~100,000)
- $d$ = embedding dimension (BERT-base: 768; GPT-3: 12,288)
- Total parameters: $|V| \times d$ (BERT-base: $30{,}522 \times 768 \approx 23M$ params — 21% of total)
- Initialised randomly; learned jointly with the rest of the model

**Subword tokenisation algorithms** — how token IDs are assigned:

| Algorithm | Used By | Key Property |
|-----------|---------|-------------|
| BPE | GPT-2, RoBERTa, LLaMA | Frequency-based byte-pair merges |
| WordPiece | BERT, DistilBERT | Likelihood-based merges; `##` prefix for continuations |
| Unigram LM | ALBERT, T5 (via SP) | Probabilistic; can produce multiple tokenisations |
| SentencePiece | T5, LLaMA, mBART | BPE or Unigram on raw bytes; no pre-tokenisation |
| Byte-level BPE | GPT-2, GPT-4 | BPE on raw bytes; truly no OOV |

**WordPiece `##` prefix convention (BERT)**:
- "playing" → `["play", "##ing"]`
- "##ing" indicates this piece is a continuation of the previous token
- The embedding for `##ing` is different from `"ing"` — they are separate rows in the embedding table

**Subword pooling** — for word-level tasks (NER, POS), multiple subword tokens map to one word:

| Strategy | How | When |
|----------|-----|------|
| First subword | Use embedding of first subword only | Standard for NER (label assigned to first piece) |
| Mean pooling | Average all subword embeddings | Semantic tasks |
| Max pooling | Element-wise max | Sometimes better for classification |

---

## Math / Formal Notation

**Embedding lookup**:
$$h_0^{(i)} = E[t_i] + P[i] + S[s_i]$$

where:
- $E[t_i] \in \mathbb{R}^d$: token embedding for token $t_i$
- $P[i] \in \mathbb{R}^d$: positional embedding for position $i$
- $S[s_i] \in \mathbb{R}^d$: segment embedding (sentence A vs B, in BERT)

This sum is the input to the first Transformer layer.

**Gradient flow through embedding**:
The embedding matrix $E$ is updated by gradient descent only for the rows corresponding to tokens that appeared in the current batch:
$$E[t] \leftarrow E[t] - \eta \frac{\partial \mathcal{L}}{\partial E[t]} \quad \text{for each } t \text{ in batch}$$

Rare tokens receive fewer updates → worse embeddings → one reason why subword tokenisation matters: splitting rare words into common subwords means every piece gets adequate training signal.

**Tied embeddings** (weight tying):
Many LMs tie the input embedding matrix $E$ with the output projection matrix $W_o \in \mathbb{R}^{d \times |V|}$, so $W_o = E^\top$. This halves the parameter count and often improves performance (forces the input and output representations to be consistent).

$$P(\text{next token} = t \mid h) = \text{softmax}(E \cdot h)_t$$

**BPE vocabulary construction** (brief):
1. Initialise vocabulary with all characters
2. Count all adjacent pair frequencies
3. Merge most frequent pair → add to vocabulary
4. Repeat until target vocabulary size reached
5. Each merge rule defines a token; the embedding for that token is initialised randomly and learned

---

## Examples (Concrete)

**BERT WordPiece tokenisation**:
```python
from transformers import BertTokenizer

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')

# Regular word
print(tokenizer.tokenize("playing"))
# ['playing']  — common enough to be a single token

# Rare/compound word
print(tokenizer.tokenize("immunocompromised"))
# ['immune', '##oco', '##mp', '##rom', '##ised']  — split into 5 subwords

# OOV but compositional
print(tokenizer.tokenize("SARS-CoV-2"))
# ['sar', '##s', '-', 'co', '##v', '-', '2']  — handled via subwords + punctuation split

# Each of these tokens has its own row in the 30,522 × 768 embedding matrix
```

**Subword alignment for NER**:
```python
from transformers import BertTokenizer
import torch

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
sentence = ["John", "ate", "unbelievably", "fast"]
labels   = ["B-PER", "O", "O", "O"]

# Tokenize with word_ids to track alignment
encoding = tokenizer(sentence, is_split_into_words=True, return_tensors='pt')
word_ids = encoding.word_ids()  # [None, 0, 0, 1, 2, 2, 2, 2, 3, None]
# None = [CLS]/[SEP], integers = original word index

# Align labels: assign label to first subword, -100 (ignore) to rest
aligned_labels = []
prev_word_id = None
for word_id in word_ids:
    if word_id is None:
        aligned_labels.append(-100)          # [CLS] / [SEP]
    elif word_id != prev_word_id:
        aligned_labels.append(labels[word_id])  # first subword → real label
    else:
        aligned_labels.append(-100)          # continuation subword → ignore
    prev_word_id = word_id
```

**Embedding layer inspection**:
```python
from transformers import BertModel

model = BertModel.from_pretrained('bert-base-uncased')

# The embedding table
embed_matrix = model.embeddings.word_embeddings.weight
print(embed_matrix.shape)  # torch.Size([30522, 768])

# Get embedding for token 2054 ("king")
king_id = tokenizer.convert_tokens_to_ids("king")  # 2177
king_embed = embed_matrix[king_id]  # 768-dim vector
```

---

## How It Connects to ML / NLP

| Subword Embedding Concept | ML/NLP Link |
|--------------------------|-------------|
| Embedding matrix = parameter table | [[3.ML & DL/1.Concepts/3.Linear Regression/Parameters.md]] — $E \in \mathbb{R}^{\|V\| \times d}$ is a learned parameter |
| Sparse gradient updates per token | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — only tokens in each batch receive gradient updates |
| Rare subwords → few updates → poor embeddings | [[3.ML & DL/1.Concepts/8.Model Behavior/Underfitting.md]] — rare tokens underfit due to insufficient training signal |
| Tied embeddings = weight sharing | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — weight tying is a form of parameter regularisation |
| Embedding dim as bottleneck | [[3.ML & DL/1.Concepts/8.Model Behavior/Model Complexity.md]] — $d$ controls representation capacity vs. parameter budget |
| Positional embedding + token embedding | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Multiple Features.md]] — input is sum of multiple feature embeddings |
| BPE vocabulary size tuning | [[3.ML & DL/1.Concepts/21.Meta & Interview Revision/How to Choose Features.md]] — vocab size is a hyperparameter balancing OOV rate vs. sequence length |
| Subword tokenisation = data representation | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Data Representation.md]] — how raw text is structurally represented before any learning |

---

## Common Interview Questions

**Q: What is the embedding layer and why is it the first layer of every Transformer?**
A: The embedding layer is a learned lookup table $E \in \mathbb{R}^{|V| \times d}$ that maps discrete token IDs (integers) to continuous dense vectors. Transformers (and all neural networks) require continuous real-valued inputs — the embedding layer bridges the gap between discrete text tokens and the continuous vector space the model operates in. Every Transformer begins with this lookup.

**Q: What happens when a word is split into multiple subwords? How do you get a word-level representation?**
A: For word-level tasks (NER, POS), the standard approach is to use only the embedding of the **first subword** and assign labels only to first subwords. For retrieval or semantic tasks, **mean pooling** across all subword tokens is preferred. The word_ids mapping from Hugging Face tokenisers makes alignment straightforward.

**Q: What is weight tying in language models?**
A: Weight tying shares the input token embedding matrix $E$ with the output projection matrix that converts hidden states to vocabulary logits. Instead of two separate matrices (one for input, one for output), the output logit for token $t$ is $E[t] \cdot h$ where $h$ is the hidden state. This halves the embedding parameters and improves performance by enforcing that similar words have similar representations in both input and output spaces.

**Q: Why does vocabulary size matter, and what happens if it's too small or too large?**
A: Too small: rare words are split into many subwords → longer sequences → more attention operations, hits the context length limit faster, and long splits may lose coherent semantic identity. Too large: many rare tokens receive insufficient training signal → poor embeddings; embedding matrix becomes memory-expensive. Typical sweet spot: 32k–100k for monolingual, 250k for massively multilingual (mBERT uses 119,547 tokens).

---

## Common Mistakes / Gotchas

- **Forgetting `##` tokens in BERT for NER**: BERT's continuation tokens (`##ing`, `##ness`) must be handled in alignment — they should not receive independent NER labels. Use `word_ids()` from Hugging Face and assign -100 to continuation subwords so they're ignored in loss.
- **Different tokenisers produce different token counts**: "running" may be 1 token in BERT (`running`) but 2 tokens in GPT-2 (`run`, `ning`). Never assume token count is the same across different models — this matters for context length and alignment.
- **Treating subword embeddings as pre-trained frozen features**: Unlike Word2Vec/GloVe, BERT's subword embeddings are not meant to be used frozen for downstream tasks — they are updated during fine-tuning. Using frozen BERT embeddings as static features significantly underperforms fine-tuning.
- **Assuming OOV is impossible with subword models**: While subword models have very low OOV rates, byte-level BPE (GPT-2) is the only approach with *zero* OOV. WordPiece and SentencePiece still have `[UNK]` tokens for characters outside their vocabulary (rare in practice but possible with emojis, exotic scripts, or null bytes).
- **Confusing tokeniser and embedding**: The tokeniser converts text to token IDs (preprocessing). The embedding layer converts token IDs to vectors (model parameter). These are separate steps, often confused when they're both called "tokenisation" informally.

---

## Further Reading / Paper References

- Devlin et al. (2018). BERT — WordPiece tokenisation and embeddings arxiv:1810.04805
- Radford et al. (2019). GPT-2 — byte-level BPE tokenisation
- Kudo, T. & Richardson, J. (2018). SentencePiece: A simple and language independent subword tokeniser arxiv:1808.06226
- Press & Wolf (2017). Using the Output Embedding to Improve Language Models — weight tying arxiv:1608.05859
- Rust et al. (2021). How Good is Your Tokeniser? On the Monolingual Performance of Multilingual Language Models — subword fertility analysis across languages arxiv:2012.15613
- Hugging Face Tokenisers docs: https://huggingface.co/docs/tokenizers
