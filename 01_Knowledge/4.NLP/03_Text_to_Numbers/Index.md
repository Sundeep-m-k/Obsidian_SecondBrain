---
tags: [nlp, index, moc, text-to-numbers, representations, embeddings]
links: "[[NLP Index]] [[02_Text_Preparation/Index]] [[04_Learning_Core/Index]]"
---

# 03 — Text to Numbers (Index)

Position in vault: `01_Knowledge/4.NLP/03_Text_to_Numbers/`
Purpose: How raw text tokens become numeric representations that machine learning models can process. This section covers the full spectrum from classical sparse vectors to contextual neural embeddings.
Prerequisite: [[01_Knowledge/4.NLP/02_Text_Preparation/Index]] — tokens must exist before they can be embedded. Understanding BPE and WordPiece directly motivates subword embeddings.

---

## Why Text-to-Numbers Matters

Machine learning models only accept numbers. Every NLP architecture requires a mapping from discrete text symbols to continuous vector spaces. The quality of this mapping determines the quality of everything downstream.

The history of this section mirrors the history of NLP itself:

- **1990s–2000s**: Sparse, count-based vectors (BoW, TF-IDF) — hand-engineered, interpretable, high-dimensional
- **2013**: Dense static embeddings (Word2Vec, GloVe) — learned, low-dimensional, captures semantics
- **2018**: Contextual embeddings (ELMo, BERT) — one representation per token per context, solves polysemy
- **2020s**: Large model embeddings — emergent representations from pretraining at scale

Each paradigm solved problems of the previous one but introduced new ones. Understanding all layers is necessary because: (1) sparse methods are still used in IR and low-resource settings, (2) static embeddings are computationally cheap and interpretable, (3) contextual embeddings are the current standard for most tasks.

---

## Section Map

### 📊 1. Sparse Representations — Count-Based Vectors

Classical text representations. High-dimensional, interpretable, no training required.

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Bag of Words]] | Document-term matrix, vocabulary, count vectors, one-hot encoding | TF-IDF, Feature Engineering |
| [[TF-IDF]] | Term frequency, inverse document frequency, IDF variants, normalization | IR, document similarity |
| [[PMI and PPMI]] | Pointwise mutual information, PPMI matrix, co-occurrence window, association | GloVe motivation, distributional semantics |
| [[Co-occurrence Matrix]] | Context window, SVD, LSA, truncated SVD, spectral word vectors | Static Embeddings, Dimensionality Reduction |

Key insight: Sparse methods produce vectors in $\mathbb{R}^{|V|}$ where $|V|$ is vocabulary size (often 100k+). Most entries are zero. They are interpretable (dimension $i$ = word $i$), but suffer from the **curse of dimensionality** and cannot capture synonymy — "car" and "automobile" are orthogonal vectors.

---

### 🧲 2. Static Embeddings — Dense Word Vectors

Learned dense representations. $\mathbb{R}^d$ where $d \approx 50$–$300$. Fixed per word regardless of context.

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Word2Vec]] | CBOW, Skip-gram, negative sampling, softmax, analogies, word arithmetic | GloVe, downstream NLP tasks |
| [[GloVe]] | Global co-occurrence, log-bilinear model, weighted least squares, bias terms | Word2Vec, PMI/PPMI |
| [[FastText]] | Subword n-grams, character n-gram embeddings, OOV handling, morphological awareness | SentencePiece, multilingual NLP |

Key insight: Static embeddings map every word to one vector regardless of context. "Bank" gets the same vector in "river bank" and "bank account." This is the central limitation that contextual embeddings solve. But static embeddings are: (1) fast, (2) interpretable, (3) still competitive for many tasks, (4) required when you need to embed individual words offline.

---

### 🌊 3. Contextual Embeddings — One Vector per Token per Context

Neural representations where the same word gets different vectors in different contexts.

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[ELMo]] | Bi-LSTM language model, layer-wise representations, feature extraction, task-specific weighting | BERT, transfer learning |
| [[BERT Embeddings]] | Transformer encoder, [CLS] token, subword pooling, fine-tuning vs feature extraction | Fine-tuning, probing |
| [[Sentence Embeddings]] | Mean pooling, [CLS] pooling, SBERT, contrastive learning, semantic similarity | RAG, retrieval, clustering |

Key insight: BERT produces representations that encode **both** the identity of a word **and its full sentential context**. This allows a single model to handle polysemy, anaphora, and syntactic role — previously requiring separate specialized systems.

---

### 🔤 4. Subword and Character Representations — Below the Word Level

Representations at the character or subword level. Crucial for morphologically rich languages and OOV handling.

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Character Embeddings]] | CNN over characters, character-level LMs, OOV handling, morphological features | Subword Embeddings, FastText |
| [[Subword Embeddings]] | BPE embeddings, WordPiece embeddings, embedding layer in transformers, subword pooling | Tokenization — BPE, BERT |

Key insight: Modern LLMs operate at the **subword** level — not words, not characters, but subword units produced by BPE/WordPiece/SentencePiece. The embedding layer is the first transformation applied and has the largest number of parameters in proportion to model size for small models.

---

### 🔧 5. Feature Engineering — Handcrafted NLP Features

Explicit feature design for classical ML pipelines and feature-augmented neural models.

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Linguistic Features]] | POS tags, NER labels, dependency labels, parse features, gazetteers, lexical resources | Classical ML pipeline |
| [[Dimensionality Reduction (NLP)\|Dimensionality Reduction]] | PCA, SVD, LSA, UMAP, t-SNE, truncated SVD, random projections | Co-occurrence Matrix, visualization — see also the general-purpose [[Dimensionality Reduction]] note in ML&DL |

Key insight: Feature engineering is not obsolete. In low-resource settings (few labeled examples), handcrafted features can outperform neural methods. Hybrid approaches that augment BERT with explicit features (syntax, entity type) often outperform BERT alone on structured tasks.

---

## The Representation Spectrum

```
Sparse ←————————————————————————————————→ Dense
High-dim ←——————————————————————————————→ Low-dim
Count-based ←———————————————————————————→ Learned
Context-independent ←———————————————————→ Contextual
Interpretable ←—————————————————————————→ Opaque
No training ←———————————————————————————→ Requires pretraining

BoW → TF-IDF → PPMI → LSA → Word2Vec → GloVe → FastText → ELMo → BERT
```

---

## Key Cross-Links to `3.ML & DL/`

| Text-to-Numbers Concept | Links to ML Concept |
|------------------------|---------------------|
| Embedding layer = linear lookup table | [[Parameters]] |
| TF-IDF weighting | [[Feature Scaling]] |
| SVD / LSA | [[Matrix Factorization]] |
| Word2Vec softmax | [[Loss Function]] *(see the Categorical Cross-Entropy section — no standalone "Cross Entropy" note exists yet)* |
| Negative sampling | *(gap — Noise Contrastive Estimation doesn't exist yet; candidate for the Sequence Models / Attention module)* |
| BERT pretraining | *(gap — Transfer Learning doesn't exist yet as its own note; candidate for a future Model Behavior addition)* |
| Cosine similarity for word vectors | *(gap — Similarity Metrics doesn't exist yet; candidate for Features and Representation)* |
| Intrinsic evaluation (analogies) | [[Classification]] *(see its Evaluation Metrics section — no standalone metrics note exists yet)* |

---

## Common Exam / Interview Questions from This Section

1. What is the difference between BoW and TF-IDF? When would you use each?
2. What is TF-IDF and why does it down-weight common words?
3. What is the distributional hypothesis and how does it motivate word embeddings?
4. Explain Word2Vec Skip-gram with negative sampling. What is the loss function?
5. What is the difference between CBOW and Skip-gram? Which works better for rare words?
6. How does GloVe differ from Word2Vec? What objective does it optimize?
7. What is FastText's key innovation? How does it handle out-of-vocabulary words?
8. What is the limitation of static embeddings that ELMo solved?
9. How does BERT produce contextual embeddings? What are the pretraining tasks?
10. What is the difference between fine-tuning and feature extraction from BERT?
11. How do you produce sentence embeddings from BERT? What are the trade-offs of [CLS] vs mean pooling?
12. What is SBERT and why is it needed for semantic similarity at scale?
13. What is LSA? How does SVD produce dense word vectors from a co-occurrence matrix?
14. Why does character-level modeling help with OOV and morphologically rich languages?
15. What is PMI? What is PPMI and why is it preferred?

---

## Reading Order (Recommended)

**Fast track** (most NLP-relevant): Bag of Words → TF-IDF → Word2Vec → GloVe → BERT Embeddings → Sentence Embeddings → Subword Embeddings

**Complete track**: Follow section order (1 → 2 → 3 → 4 → 5)

**IR / search track**: Bag of Words → TF-IDF → PMI and PPMI → Co-occurrence Matrix → Dimensionality Reduction → Sentence Embeddings

**Low-resource track**: Linguistic Features → FastText → Character Embeddings → Subword Embeddings → Dimensionality Reduction

---

Next section: [[04_Learning_Core/Index]] — Probability, Classical ML, Neural Fundamentals
Previous section: [[01_Knowledge/4.NLP/02_Text_Preparation/Index]]
