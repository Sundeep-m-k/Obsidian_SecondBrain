---
tags: [nlp, contextual-embeddings, sentence-embeddings, sbert, contrastive-learning, semantic-similarity, rag, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[BERT Embeddings]] [[ELMo]] [[Dense Retrieval]] [[RAG]]"
---

# Sentence Embeddings

## Definition + Intuition

**Sentence embeddings** map an entire sentence (or paragraph) to a single fixed-size dense vector that captures its meaning. Unlike token-level embeddings (BERT's per-token output), a sentence embedding is a single point in $\mathbb{R}^d$ representing the whole input's semantics.

**Why this is non-trivial**: BERT produces excellent token embeddings, but naively aggregating them (e.g., averaging) gives poor sentence representations for similarity tasks. This is because BERT was pre-trained with MLM/NSP — objectives that don't optimise for *comparable* sentence-level representations. Two semantically identical sentences may produce very different BERT vectors.

**The solution — SBERT**: Sentence-BERT (Reimers & Gurevych, 2019) fine-tunes BERT using a **Siamese network** with a contrastive or cosine similarity objective on sentence pairs. This produces embeddings where **cosine similarity directly reflects semantic similarity** — making semantic search, clustering, and paraphrase detection efficient at scale.

**Why it matters for production**: With SBERT embeddings, finding the most similar sentence to a query among 10M candidates takes milliseconds via approximate nearest neighbour search (FAISS). With raw BERT, you'd need to run the full cross-encoder on every candidate pair — completely infeasible.

---

## Key Properties / Types

**Pooling strategies** (how to get one vector from BERT's token outputs):

| Strategy | Method | Best For |
|----------|--------|---------|
| `[CLS]` pooling | Take final hidden state of `[CLS]` token | Tasks trained with `[CLS]` (classification) |
| Mean pooling | Average all token embeddings | Semantic similarity (most common) |
| Max pooling | Element-wise max over all token embeddings | Sometimes better for classification |
| Weighted mean | Weight by attention or position | Task-specific, rare |

**Important**: `[CLS]` pooling from vanilla BERT performs poorly for semantic similarity. Mean pooling of BERT token embeddings is already better. SBERT (trained specifically for similarity) is far better than both.

**SBERT training objectives**:

1. **Natural Language Inference (NLI)** — classification objective
   - Siamese BERT encodes premise and hypothesis
   - Concatenate: `[u; v; |u-v|]` → softmax over {entailment, contradiction, neutral}
   - Forces similar sentences to have similar vectors

2. **Cosine similarity regression** — regression objective
   - Train on sentence pairs with gold similarity scores (e.g., STS-B)
   - Loss: MSE between cosine(u, v) and gold score

3. **Multiple Negatives Ranking (MNR)** — contrastive objective
   - Positive pair (query, relevant passage); all other passages in batch are negatives
   - Loss: cross-entropy over softmax of dot products
   - Most effective for retrieval applications

**Key SBERT models** (all via Hugging Face `sentence-transformers`):

| Model | Dim | Notes |
|-------|-----|-------|
| `all-mpnet-base-v2` | 768 | Best general purpose |
| `all-MiniLM-L6-v2` | 384 | 6 layers, 5× faster, 80% performance |
| `multi-qa-MiniLM-L6-cos-v1` | 384 | Optimised for QA/retrieval |
| `paraphrase-multilingual-mpnet-base-v2` | 768 | 50+ languages |

---

## Math / Formal Notation

**Siamese network** for SBERT:

```
Sentence A → BERT → Pool → u ∈ ℝᵈ
Sentence B → BERT → Pool → v ∈ ℝᵈ
(shared weights)
```

**Cosine similarity objective**:
$$\mathcal{L}_{cos} = \text{MSE}\left(\cos(u, v), y\right) \quad y \in [-1, 1]$$

**NLI classification objective**:
$$\mathcal{L}_{NLI} = \text{CrossEntropy}\left(W \cdot [u; v; |u-v|],\ \text{label}\right)$$

where $W \in \mathbb{R}^{3 \cdot d \times 3}$ and the three classes are entailment, neutral, contradiction.

**Multiple Negatives Ranking loss** (for retrieval):
$$\mathcal{L}_{MNR} = -\frac{1}{N} \sum_{i=1}^{N} \log \frac{\exp(q_i \cdot p_i / \tau)}{\sum_{j=1}^{N} \exp(q_i \cdot p_j / \tau)}$$

where $q_i$ = query embedding, $p_i$ = relevant passage embedding, $\tau$ = temperature, $N$ = batch size. Every other passage in the batch is an implicit negative.

**Cosine similarity** (the primary distance metric):
$$\text{cos}(u, v) = \frac{u \cdot v}{\|u\| \cdot \|v\|}$$

After L2 normalisation, $\text{cos}(u, v) = u \cdot v$ — dot product suffices, enabling fast FAISS indexing.

---

## Examples (Concrete)

**Semantic similarity**:
```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer('all-mpnet-base-v2')

sentences = [
    "A man is eating pasta.",
    "Someone is consuming noodles.",
    "A dog is playing in the park.",
    "The stock market crashed today.",
]

embeddings = model.encode(sentences)  # shape: (4, 768)

# Pairwise cosine similarities
cosine_scores = util.cos_sim(embeddings, embeddings)
print(cosine_scores)
# [[1.00, 0.82, 0.12, 0.05],   ← "eating pasta" vs all
#  [0.82, 1.00, 0.11, 0.04],   ← "consuming noodles" vs all
#  ...]

# Sentences 0 & 1 score ~0.82 (paraphrases)
# Sentences 0 & 2 score ~0.12 (unrelated)
```

**Semantic search** (query against a large corpus):
```python
import faiss
import numpy as np

# Encode corpus
corpus = [...]  # 1 million sentences
corpus_embeddings = model.encode(corpus, batch_size=256, show_progress_bar=True)
corpus_embeddings = corpus_embeddings / np.linalg.norm(corpus_embeddings, axis=1, keepdims=True)

# Build FAISS index (inner product = cosine similarity on normalised vectors)
index = faiss.IndexFlatIP(768)
index.add(corpus_embeddings.astype('float32'))

# Query
query = "How does photosynthesis work?"
q_emb = model.encode([query])
q_emb = q_emb / np.linalg.norm(q_emb)

# Find top-5 most similar
distances, indices = index.search(q_emb.astype('float32'), k=5)
for idx, score in zip(indices[0], distances[0]):
    print(f"{score:.3f}: {corpus[idx]}")
```

**Clustering**:
```python
from sklearn.cluster import KMeans
import numpy as np

embeddings = model.encode(documents)  # (N, 768)
kmeans = KMeans(n_clusters=10, random_state=42)
labels = kmeans.fit_predict(embeddings)
```

---

## How It Connects to ML / NLP

| Sentence Embedding Concept | ML/NLP Link |
|---------------------------|-------------|
| Siamese network training | [[3.ML & DL/1.Concepts/2.Supervised Learning/Binary Classification.md]] — NLI objective is 3-class classification |
| Contrastive / MNR loss | [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — contrastive loss is a specialised classification loss |
| Cosine similarity threshold | [[3.ML & DL/1.Concepts/7.Logistic Regression/Thresholding.md]] — paraphrase detection via similarity threshold |
| FAISS nearest neighbour | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Clustering.md]] — ANN search is a clustering/retrieval operation |
| Clustering with K-Means on embeddings | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/K Means.md]] — sentence embeddings enable semantic clustering |
| Mean pooling | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — pooling collapses token matrix to single feature vector |
| Fine-tuning BERT for similarity | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — small LR update to adapt pre-trained representations |
| Embedding quality vs speed tradeoff | [[3.ML & DL/1.Concepts/8.Model Behavior/Bias Variance Tradeoff.md]] — larger models give better embeddings at higher compute cost |

**RAG (Retrieval-Augmented Generation)**: Sentence embeddings are the core of RAG systems. Documents are embedded offline; at query time, the query is embedded and FAISS returns the top-$k$ relevant passages, which are then passed to the LLM as context. The embedding model quality directly determines retrieval recall.

---

## Common Interview Questions

**Q: Why doesn't vanilla BERT produce good sentence embeddings for semantic similarity?**
A: BERT was pre-trained for MLM and NSP — neither task requires *comparable* representations across sentences. BERT's anisotropic embedding space (embeddings cluster in a narrow cone) means cosine similarity is poorly calibrated. SBERT fine-tunes BERT with a contrastive objective that explicitly trains the model to place semantically similar sentences near each other in embedding space.

**Q: What is the difference between a bi-encoder and a cross-encoder?**
A: A **bi-encoder** (SBERT) encodes each sentence independently → two fixed vectors → cosine similarity. Fast at inference — encode once, compare offline. A **cross-encoder** concatenates the two sentences as input to BERT → one forward pass → similarity score. Much more accurate (attends to the pair jointly) but $O(N)$ at query time — doesn't scale to large corpora. Production systems use bi-encoder for retrieval (top-$k$) then cross-encoder for reranking (top-$k' \ll k$).

**Q: What is Multiple Negatives Ranking (MNR) loss?**
A: A contrastive loss for training retrieval models. In a batch of $N$ (query, passage) pairs, each query's matching passage is the positive; all other passages in the batch are implicit negatives. Loss is cross-entropy over softmax of dot products. Large batch size = more negatives = harder training signal. In-batch negatives are efficient because no extra negative mining is needed.

**Q: What is mean pooling and why is it preferred over `[CLS]` for sentence embeddings?**
A: Mean pooling averages the final-layer hidden states of all tokens (including special tokens, or excluding them). `[CLS]` only uses one token's state. In practice, mean pooling produces a more uniform use of BERT's capacity and consistently outperforms `[CLS]` pooling for semantic similarity on STS benchmarks when BERT is not specifically fine-tuned for similarity.

---

## Common Mistakes / Gotchas

- **Using `[CLS]` from un-fine-tuned BERT for similarity**: This performs at or near random on STS benchmarks. Always use mean pooling or a dedicated similarity model (SBERT).
- **Not normalising before cosine similarity**: If you use dot product instead of cosine similarity, sentence length and content type bias the scores. Always L2-normalise embeddings before dot product for semantic similarity.
- **Ignoring domain mismatch**: `all-mpnet-base-v2` is trained on general web text. For legal, medical, or code retrieval, domain-specific models (LegalBERT, BioBERT, CodeBERT) or domain fine-tuning is essential.
- **Encoding very long documents**: SBERT models have the same 512-token limit as BERT. For paragraphs/documents: (1) encode the first 512 tokens, (2) mean-pool over sliding windows, (3) use a hierarchical model. Truncation loses information for long documents.
- **Using sentence embeddings where cross-encoders are needed**: For tasks requiring fine-grained comparison (NLI, reading comprehension), bi-encoders are significantly weaker than cross-encoders. Use bi-encoders for retrieval (scale), cross-encoders for reranking (accuracy).

---

## Further Reading / Paper References

- Reimers, N. & Gurevych, I. (2019). Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks arxiv:1908.10084
- Gao et al. (2021). SimCSE: Simple Contrastive Learning of Sentence Embeddings arxiv:2104.08821
- Karpukhin et al. (2020). Dense Passage Retrieval for Open-Domain Question Answering (DPR) arxiv:2004.04906
- Sentence Transformers library: https://www.sbert.net — models, training, FAISS integration
- Hugging Face MTEB Leaderboard: https://huggingface.co/spaces/mteb/leaderboard — embedding model benchmarks
- Johnson et al. (2017). Billion-Scale Similarity Search with GPUs (FAISS) arxiv:1702.08734
