---
tags: [nlp, sparse-representations, co-occurrence, lsa, svd, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[PMI and PPMI]] [[TF-IDF]] [[Word2Vec]] [[Dimensionality Reduction]]"
---

# Co-occurrence Matrix

## Definition + Intuition

A **co-occurrence matrix** records how often pairs of words appear together within a defined context window across a corpus. It is the raw count structure from which PPMI vectors and LSA embeddings are derived.

**Intuition**: If you read every sentence in a large corpus and, for each word, wrote down every other word that appeared within 5 words of it, you'd accumulate a table of counts: "how many times did word $i$ appear near word $j$?" That table — the co-occurrence matrix — encodes distributional meaning. Words with similar rows in this matrix (similar contexts) tend to have similar meanings.

**From matrix to embeddings**: The raw co-occurrence matrix is huge ($|V| \times |V|$, potentially $10^5 \times 10^5$) and sparse. Two operations transform it into usable representations:
1. **Weight by PPMI** → sparse, interpretable word vectors
2. **SVD (truncated)** → dense, low-dimensional word vectors = **Latent Semantic Analysis (LSA)**

---

## Key Properties / Types

**Word × Word Co-occurrence Matrix**
- Entry $M_{ij}$ = number of times word $j$ appears within a window of word $i$
- Size: $|V| \times |V|$, very sparse (most word pairs never co-occur)
- Symmetric if window is symmetric (±$k$ words), asymmetric if directional

**Word × Document Matrix (document-term matrix)**
- Entry $M_{ij}$ = count/TF-IDF of word $i$ in document $j$
- Used for document retrieval; SVD on this matrix = document-level LSA

**Context Window Size Effect**

| Window | Captures | Use Case |
|--------|----------|----------|
| ±1–2 words | Syntactic relations (subject-verb, adjective-noun) | Syntax-sensitive tasks |
| ±5–10 words | Semantic relations (topical similarity) | Semantic similarity |
| Whole document | Topical co-occurrence | Document retrieval (LSA) |

**Weighting Schemes**
- Raw counts: dominated by function words
- PPMI weighted: removes noise, highlights associations (→ see [[PMI and PPMI]])
- Distance-weighted: words closer to target get higher weight (used in GloVe)

---

## Math / Formal Notation

**Raw co-occurrence matrix**:
$$M_{ij} = \sum_{\text{doc}} \sum_{p} \mathbf{1}[w_p = w_i] \cdot \mathbf{1}[w_q = w_j, |p-q| \leq k]$$

where $k$ is the window size and $p, q$ are token positions.

**Truncated SVD (the core of LSA)**:

Any matrix $M \in \mathbb{R}^{m \times n}$ can be decomposed:
$$M = U \Sigma V^\top$$

where $U \in \mathbb{R}^{m \times m}$, $\Sigma \in \mathbb{R}^{m \times n}$ (diagonal, singular values), $V \in \mathbb{R}^{n \times n}$.

**Truncated SVD** keeps only the top $d$ singular values:
$$M \approx U_d \Sigma_d V_d^\top$$

where $U_d \in \mathbb{R}^{m \times d}$, $\Sigma_d \in \mathbb{R}^{d \times d}$, $V_d \in \mathbb{R}^{n \times d}$, $d \ll \min(m, n)$.

**Word vectors from LSA**: Row $i$ of $U_d \Sigma_d$ is the dense $d$-dimensional embedding for word $i$.

**Eckart-Young theorem**: The rank-$d$ truncated SVD is the *best* rank-$d$ approximation of $M$ in Frobenius norm:
$$\hat{M}_d = \arg\min_{\text{rank-}d\ A} \|M - A\|_F$$

This means LSA finds the $d$ dimensions of the co-occurrence matrix that explain the most variance — the latent semantic dimensions.

**Reconstruction quality**:
$$\text{Fraction of variance explained} = \frac{\sum_{i=1}^{d} \sigma_i^2}{\sum_{i=1}^{|V|} \sigma_i^2}$$

---

## Examples (Concrete)

**Toy corpus** (window = ±2):
```
"I like deep learning"
"I like NLP"
"I enjoy deep learning"
```

**Raw co-occurrence matrix** (symmetric):

| | I | like | deep | learning | NLP | enjoy |
|---|---|------|------|----------|-----|-------|
| **I** | 0 | 2 | 1 | 0 | 0 | 1 |
| **like** | 2 | 0 | 2 | 1 | 1 | 0 |
| **deep** | 1 | 2 | 0 | 2 | 0 | 2 |
| **learning** | 0 | 1 | 2 | 0 | 1 | 1 |
| **NLP** | 0 | 1 | 0 | 1 | 0 | 0 |
| **enjoy** | 1 | 0 | 2 | 1 | 0 | 0 |

After applying PPMI and truncated SVD ($d=2$), "like" and "enjoy" would have similar vectors (they appear in similar contexts), even though they never directly co-occur. This is latent semantic similarity.

**Python (scipy)**:
```python
import numpy as np
from scipy.sparse.linalg import svds
from sklearn.preprocessing import normalize

# M = PPMI weighted co-occurrence matrix (sparse)
# Truncated SVD to d=300 dimensions
U, S, Vt = svds(M, k=300)

# Word vectors: rows of U * S
word_vectors = U * S          # shape: (vocab_size, 300)
word_vectors = normalize(word_vectors)  # L2 normalise
```

---

## How It Connects to ML / NLP

| Co-occurrence Concept | ML/NLP Application |
|----------------------|-------------------|
| SVD = matrix factorisation | [[3.ML & DL/1.Concepts/11.Recommender Systems/Matrix Factorization.md]] — same mathematical operation used in collaborative filtering |
| Truncated SVD = dimensionality reduction | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Dimensionality Reduction.md]] — LSA is the NLP instance of this general technique |
| PCA vs SVD relationship | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/PCA.md]] — PCA on mean-centred data = SVD; LSA skips centering |
| Variance explained by singular values | [[3.ML & DL/1.Concepts/8.Model Behavior/Bias Variance Tradeoff.md]] — choosing $d$ trades off reconstruction error vs. dimensionality |
| Co-occurrence counts as training signal | [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — self-supervised signal from raw text |
| Window size = inductive bias | [[3.ML & DL/1.Concepts/1.Foundations/Model.md]] — choosing the window encodes assumptions about what context means |

**Historical position**: LSA (Deerwester et al., 1990) was the first successful dense word representation method. GloVe (2014) is conceptually a neural approximation of SVD on log co-occurrence. Word2Vec (2013) implicitly factorises a shifted PPMI matrix. Understanding the co-occurrence matrix makes all three methods interpretable.

---

## Common Interview Questions

**Q: What is Latent Semantic Analysis (LSA)?**
A: LSA applies truncated SVD to a term-document (or word-word) co-occurrence matrix, reducing it from $|V|$-dimensional sparse vectors to $d$-dimensional dense vectors. The resulting word vectors capture latent semantic structure: words that appear in similar contexts end up with similar vectors, even if they never directly co-occur.

**Q: What is the difference between SVD and PCA?**
A: PCA = SVD applied to the covariance matrix (mean-centred data). SVD is the more general operation applied directly to the data matrix. For word embeddings, LSA applies SVD directly to the co-occurrence matrix without mean-centring — making it computationally tractable for large sparse matrices.

**Q: How does window size affect the co-occurrence matrix?**
A: Small windows (±1–2) capture syntactic context — words in grammatical relation (adjective-noun, subject-verb). Large windows (±5–10) capture semantic/topical context — words that appear in the same discourse. The GloVe paper uses a distance-weighted window where closer words count more.

**Q: Why is truncated SVD preferred over full SVD for word vectors?**
A: Full SVD is computationally prohibitive for $|V| \times |V|$ matrices (often $10^5 \times 10^5$). Truncated SVD computes only the top $d$ singular values/vectors using iterative methods (Lanczos, randomised SVD), which is $O(|V| \cdot d)$ instead of $O(|V|^3)$. Additionally, retaining only top components acts as noise reduction — the lower singular values encode corpus noise.

---

## Common Mistakes / Gotchas

- **Using raw counts without PPMI**: Raw counts are dominated by function words. Always apply PPMI weighting before SVD for word-word matrices. For term-document matrices, use TF-IDF weighting.
- **Setting $d$ too low or too high**: Typical values: $d = 100$–$300$ for word embeddings. Too low → poor representation. Too high → overfitting to corpus noise. Tune on your intrinsic evaluation task.
- **Symmetric vs. asymmetric windows**: A symmetric window (±$k$) makes the matrix symmetric; the left and right context of a word are treated equally. Some researchers use asymmetric windows (left context only, or directional) to capture word-order information.
- **Memory**: A dense $|V| \times |V|$ matrix with $|V| = 100{,}000$ requires 40 GB (float32). Always use sparse matrix storage and sparse SVD solvers (`scipy.sparse.linalg.svds` or `sklearn.utils.extmath.randomized_svd`).
- **LSA vs Word2Vec performance**: LSA tends to underperform Word2Vec and GloVe on word analogy benchmarks. This is primarily because SGD-based training implicitly performs noise contrastive estimation, which is harder to replicate with batch SVD.

---

## Further Reading / Paper References

- Deerwester et al. (1990). Indexing by Latent Semantic Analysis — original LSA paper
- Turney & Pantel (2010). From Frequency to Meaning: Vector Space Models of Semantics — comprehensive survey of co-occurrence methods
- Levy & Goldberg (2014). Neural Word Embedding as Implicit Matrix Factorization — connects SVD/PPMI to Word2Vec arxiv:1411.2738
- Pennington et al. (2014). GloVe: Global Vectors for Word Representation — extends co-occurrence weighting to neural training arxiv:1405.0312
- Halko, Martinsson & Tropp (2011). Finding Structure with Randomness — randomised SVD algorithm used in sklearn
