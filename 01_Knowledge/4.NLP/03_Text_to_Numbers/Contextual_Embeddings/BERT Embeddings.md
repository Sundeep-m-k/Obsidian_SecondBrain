---
tags: [nlp, contextual-embeddings, bert, transformer, masked-lm, fine-tuning, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[ELMo]] [[Sentence Embeddings]] [[Subword Embeddings]] [[Transformer Architecture]] [[Pretrained Encoders]]"
---

# BERT Embeddings

## Definition + Intuition

**BERT (Bidirectional Encoder Representations from Transformers)** (Devlin et al., 2018) is a Transformer encoder pre-trained on masked language modelling over large text corpora. Its output is a **contextual embedding** for every input token — a vector that encodes both the token's identity and its full bidirectional context (every other token in the sequence).

**Intuition**: BERT reads the entire sentence at once (unlike LSTMs which read left-to-right or right-to-left) and allows every token to attend to every other token simultaneously via self-attention. This means the embedding for any token integrates information from the full context — not just nearby words, but any word in the sequence, regardless of distance.

The pre-training task — **Masked Language Modelling (MLM)** — forces BERT to learn deep bidirectional representations: predict a masked token from all surrounding tokens, which requires understanding syntax, semantics, and real-world knowledge simultaneously.

**Why it matters**: BERT (2018) shifted the paradigm from task-specific architectures to **pre-train once, fine-tune everywhere**. A single pre-trained BERT model, with a small task-specific head, achieved state-of-the-art on 11 NLP benchmarks simultaneously.

---

## Key Properties / Types

**BERT Variants**:

| Model | Layers | Hidden | Heads | Parameters | Notes |
|-------|--------|--------|-------|-----------|-------|
| BERT-base | 12 | 768 | 12 | 110M | Standard choice |
| BERT-large | 24 | 1024 | 16 | 340M | Larger, slower |
| RoBERTa-base | 12 | 768 | 12 | 125M | Better training; no NSP |
| DistilBERT | 6 | 768 | 12 | 66M | 40% smaller, 60% faster, 97% performance |
| ALBERT | 12 | 768 | 12 | 12M | Parameter sharing; very small |
| DeBERTa | 12 | 768 | 12 | 86M | Disentangled attention; best on GLUE |

**Pre-training tasks**:

1. **Masked Language Modelling (MLM)**: 15% of tokens are masked. Model predicts them from context. Of masked tokens: 80% replaced with `[MASK]`, 10% replaced with random word, 10% left unchanged. This trick prevents BERT from learning to only handle `[MASK]` tokens.

2. **Next Sentence Prediction (NSP)**: Given two sentences, predict whether B follows A in the original document. Motivated by tasks requiring cross-sentence reasoning. *Note*: RoBERTa (2019) showed NSP hurts or doesn't help — most modern models remove it.

**Special tokens**:
- `[CLS]` — prepended to every input; its final hidden state used as sequence representation for classification
- `[SEP]` — separates sentence pairs; appended at end of each sentence
- `[MASK]` — used during pre-training to replace masked tokens

**Input representation** = token embedding + segment embedding + positional embedding (all summed):
$$\text{input}_i = \text{TokenEmbed}(t_i) + \text{SegmentEmbed}(s_i) + \text{PositionEmbed}(i)$$

---

## Math / Formal Notation

**Self-attention** (the core operation in each Transformer layer):

For queries $Q$, keys $K$, values $V$ (all derived from the input $X$):
$$\text{Attention}(Q, K, V) = \text{softmax}\left(\frac{QK^\top}{\sqrt{d_k}}\right) V$$

**Multi-head attention** (BERT uses $h$ heads in parallel):
$$\text{MultiHead}(Q, K, V) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h) W^O$$

where $\text{head}_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)$.

**MLM Loss**:
$$\mathcal{L}_{MLM} = -\sum_{i \in \mathcal{M}} \log P(t_i \mid \mathbf{t}_{\setminus \mathcal{M}}; \Theta)$$

where $\mathcal{M}$ is the set of masked positions.

**BERT output** for sequence of $n$ tokens:
$$H = [h_1, h_2, \ldots, h_n] \in \mathbb{R}^{n \times d}$$

Each $h_i \in \mathbb{R}^d$ is the contextual embedding for token $i$ from the final (or chosen) layer.

**Fine-tuning** — add a task head on top:

| Task | What to use from BERT | Head |
|------|-----------------------|------|
| Sequence classification | $h_{[CLS]}$ | Linear($d$, num\_classes) |
| Token classification (NER) | $h_1, \ldots, h_n$ | Linear($d$, num\_labels) per token |
| Question answering | $h_1, \ldots, h_n$ | Two linear heads: start/end position |
| Sentence pair | $h_{[CLS]}$ from paired input | Linear($d$, num\_classes) |

---

## Examples (Concrete)

**Contextual disambiguation**:

```python
from transformers import BertTokenizer, BertModel
import torch

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertModel.from_pretrained('bert-base-uncased')
model.eval()

# Same word "bank", different contexts
sent1 = "I deposited money at the bank"
sent2 = "We sat on the bank of the river"

for sent in [sent1, sent2]:
    inputs = tokenizer(sent, return_tensors='pt')
    with torch.no_grad():
        outputs = model(**inputs)
    
    # Last hidden state: [batch=1, seq_len, 768]
    last_hidden = outputs.last_hidden_state
    
    # Find position of "bank"
    tokens = tokenizer.convert_ids_to_tokens(inputs['input_ids'][0])
    bank_idx = tokens.index('bank')
    bank_embedding = last_hidden[0, bank_idx]  # shape: (768,)
    print(f"'{sent}' → bank vector norm: {bank_embedding.norm():.3f}")

# The two bank vectors will be different — BERT has disambiguated the sense
```

**Layer analysis** (which layer to use):
```python
outputs = model(**inputs, output_hidden_states=True)
all_layers = outputs.hidden_states  # tuple of (num_layers+1) tensors

# Layer 0: embedding layer (context-independent)
# Layers 1-12: increasingly contextual
# Layer -1 (last): most contextual, best for most tasks
# Layer -2: sometimes better for similarity tasks

# Mean of last 4 layers: common best practice for embeddings
last_4 = torch.stack(all_layers[-4:])          # (4, batch, seq, 768)
mean_last_4 = last_4.mean(dim=0)               # (batch, seq, 768)
```

**Feature extraction vs fine-tuning**:
```python
# Feature extraction: freeze BERT, train only the head
for param in model.parameters():
    param.requires_grad = False

# Fine-tuning: update all parameters
for param in model.parameters():
    param.requires_grad = True
# Use small learning rate: 2e-5 to 5e-5
```

---

## How It Connects to ML / NLP

| BERT Concept | ML/NLP Link |
|-------------|-------------|
| Pre-training on MLM (self-supervised) | [[3.ML & DL/1.Concepts/1.Foundations/Unsupervised Learning.md]] — no labels needed; raw text provides training signal |
| Fine-tuning all parameters on downstream task | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — full backprop through all 110M params with small LR |
| `[CLS]` vector for classification | [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — linear head on `[CLS]` = logistic regression on BERT features |
| Overfitting on small datasets | [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — BERT fine-tuning with <1k examples often overfits; use DistilBERT or frozen BERT |
| WordPiece tokenisation | [[Subword Embeddings]] — BERT input is WordPiece subword tokens, not whole words |
| Token embedding + position + segment | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — input representation is sum of three embedding tables |
| Dropout in Transformer layers | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — BERT uses 0.1 dropout throughout |
| Learning rate warmup + decay | [[3.ML & DL/1.Concepts/5.Optimization/Learning Rate.md]] — linear warmup for 10% of steps, then linear decay |
| NSP as auxiliary task | [[3.ML & DL/1.Concepts/4.Loss and Cost/Objective Function.md]] — multi-task training objective (MLM + NSP combined) |

---

## Common Interview Questions

**Q: What is Masked Language Modelling and why does it produce bidirectional representations?**
A: MLM randomly masks 15% of input tokens and trains the model to predict them from context. Because the model must predict a masked token using *all* surrounding tokens (both left and right), it learns representations that integrate bidirectional context. Standard language models (GPT) only see left context; ELMo sees left and right separately; BERT sees both jointly.

**Q: What is the difference between feature extraction and fine-tuning with BERT?**
A: Feature extraction: BERT is frozen; its outputs are used as fixed input features to a separate task model. Fine-tuning: BERT's parameters are updated along with the task head during supervised training. Fine-tuning almost always outperforms feature extraction but requires more compute and risks catastrophic forgetting on small datasets.

**Q: What is the `[CLS]` token and when is it used?**
A: `[CLS]` is a special token prepended to every BERT input. Through self-attention across all layers, its final hidden state aggregates information from the entire sequence. It is used as the sequence-level representation for classification tasks. However, for semantic similarity and retrieval, mean pooling of all token embeddings outperforms `[CLS]` (as SBERT demonstrates).

**Q: What is the maximum input length for BERT and what happens with longer documents?**
A: BERT-base has a maximum sequence length of 512 tokens (positional embeddings are fixed at training time). For longer documents: (1) truncate to 512 tokens; (2) slide a window and pool representations; (3) use Longformer or BigBird which extend attention to 4096+ tokens.

**Q: Why does the 80/10/10 masking scheme exist in BERT?**
A: If `[MASK]` tokens only appeared during pre-training but never at fine-tuning, the model would learn to handle `[MASK]` specially and perform differently on real tokens. By leaving 10% of "masked" tokens unchanged and replacing 10% with random words, BERT must maintain good representations for *all* tokens, not just masked ones.

---

## Common Mistakes / Gotchas

- **Subword token alignment**: BERT tokenises into WordPiece subwords. "running" → `["run", "##ning"]`. For token-level tasks (NER), you must align labels to the *first* subword of each word and ignore `##` continuations. Forgetting this silently corrupts NER training.
- **Using `[CLS]` for semantic similarity**: `[CLS]` is trained for NSP classification, not semantic similarity. For sentence similarity/retrieval, use mean pooling or SBERT — `[CLS]` cosine similarity performs near random on semantic textual similarity benchmarks.
- **Fine-tuning with too large a learning rate**: BERT is catastrophically sensitive to learning rate. Use 1e-5 to 5e-5. Larger LR destroys pre-trained weights in the first epoch.
- **Not using a warmup schedule**: Fine-tuning without LR warmup often causes instability in early epochs. Always use linear warmup for 5–10% of total steps.
- **Truncating naively**: Simply truncating to 512 tokens loses the document tail. For classification, the most informative content may be in the middle or end. Consider head+tail (first 128 + last 382 tokens) or hierarchical encoding.

---

## Further Reading / Paper References

- Devlin et al. (2018). BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding arxiv:1810.04805
- Liu et al. (2019). RoBERTa: A Robustly Optimised BERT Pretraining Approach arxiv:1907.11692
- Sanh et al. (2019). DistilBERT, a distilled version of BERT arxiv:1910.01108
- Clark et al. (2019). What Does BERT Look At? An Analysis of BERT's Attention — probing BERT's internal representations
- Rogers, Kovaleva & Rumshisky (2020). A Primer in BERTology: What We Know About How BERT Works — comprehensive analysis arxiv:2002.12327
- Hugging Face Transformers: https://huggingface.co/docs/transformers
