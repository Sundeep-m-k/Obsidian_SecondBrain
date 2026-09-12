---
tags: [nlp, feature-engineering, dimensionality-reduction, pca, svd, lsa, umap, tsne, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Co-occurrence Matrix]] [[Linguistic Features]] [[Bag of Words]] [[TF-IDF]]"
---

# Dimensionality Reduction

## Definition + Intuition

**Dimensionality reduction** transforms high-dimensional text representations (sparse BoW vectors in $\mathbb{R}^{|V|}$, linguistic feature vectors in $\mathbb{R}^{10^4+}$) into lower-dimensional dense representations in $\mathbb{R}^d$ where $d \ll |V|$. The goal: retain the information that matters for the downstream task while discarding noise and redundancy.

**Intuition**: A BoW vector for a 500-word vocabulary has 500 dimensions, but most documents only use 50–100 unique words. The majority of dimensions are zero. More importantly, many dimensions are *correlated* — "car" and "automobile" co-occur in similar documents. Dimensionality reduction finds the underlying latent dimensions that explain most of the variance — compressing the representation while preserving structure.

**Two use cases in NLP**:
1. **Compression before ML**: Reduce 100k-dim BoW → 300-dim dense vectors for downstream classifiers (LSA, PCA). Reduces computation and often improves generalisation.
2. **Visualisation**: Compress 300-dim word/sentence embeddings → 2D/3D for plotting (t-SNE, UMAP). Reveals clustering structure invisible in high dimensions.

---

## Key Properties / Types

**Linear methods** — assume the important structure lies in a linear subspace:

| Method | Input | Output | NLP Use |
|--------|-------|--------|---------|
| PCA | Dense, centred | Top-$d$ principal components | Visualisation, compression of feature matrices |
| SVD (Truncated) | Sparse or dense | Low-rank approximation | Co-occurrence matrix → LSA word vectors |
| LSA | TF-IDF or raw count matrix | Dense word/doc vectors | Topic modelling, document similarity |
| Random Projections | Any high-dim | Random low-dim | Fast approximation for large matrices |

**Non-linear methods** — capture non-linear manifold structure:

| Method | Preserves | NLP Use |
|--------|----------|---------|
| t-SNE | Local structure | Visualising word/sentence clusters |
| UMAP | Both local and global structure | Embedding visualisation, pre-clustering |
| Autoencoders | Reconstruction fidelity | Learned compression of text features |

**LSA = SVD applied to text matrices** (the most important for NLP):
- Apply to word × document TF-IDF matrix → document similarity, topic retrieval
- Apply to word × context PPMI matrix → word similarity (latent semantic word vectors)
- The two uses of LSA are conceptually different but mathematically identical

---

## Math / Formal Notation

**PCA**:

Given data matrix $X \in \mathbb{R}^{n \times p}$ (n samples, p features):
1. Centre: $\tilde{X} = X - \bar{X}$
2. Compute covariance: $\Sigma = \frac{1}{n}\tilde{X}^\top \tilde{X}$
3. Eigendecomposition: $\Sigma = V \Lambda V^\top$
4. Project: $Z = \tilde{X} V_d$ where $V_d$ = top-$d$ eigenvectors

Equivalently: SVD of $\tilde{X}$ gives $\tilde{X} = U\Sigma V^\top$; principal components are $U_d \Sigma_d$.

**Truncated SVD** (for NLP, applied directly without centering):
$$M \approx U_d \Sigma_d V_d^\top, \quad U_d \in \mathbb{R}^{m \times d}, \Sigma_d \in \mathbb{R}^{d \times d}, V_d \in \mathbb{R}^{n \times d}$$

- Word vectors: rows of $U_d \Sigma_d$
- Document vectors: rows of $V_d \Sigma_d$
- Word-document similarity: $(U_d \Sigma_d)(V_d \Sigma_d)^\top \approx M$

**Frobenius reconstruction error**:
$$\|M - U_d \Sigma_d V_d^\top\|_F^2 = \sum_{i=d+1}^{r} \sigma_i^2$$

Dropping dimensions $d+1, \ldots, r$ costs $\sum_{i=d+1}^r \sigma_i^2$ in reconstruction accuracy.

**Variance explained** (elbow method for choosing $d$):
$$\text{VE}(d) = \frac{\sum_{i=1}^d \sigma_i^2}{\sum_{i=1}^r \sigma_i^2}$$

**t-SNE objective** (for visualisation only):
$$\mathcal{L} = \text{KL}\left(P \| Q\right) = \sum_{i \neq j} p_{ij} \log \frac{p_{ij}}{q_{ij}}$$

where $p_{ij}$ = Gaussian-based similarity in high-dim space, $q_{ij}$ = Student-t based similarity in 2D space. Minimising KL divergence places similar high-dim points near each other in 2D.

**UMAP** uses a different manifold learning objective that better preserves global structure and is much faster than t-SNE for large datasets.

---

## Examples (Concrete)

**LSA on TF-IDF matrix**:
```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.decomposition import TruncatedSVD
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import Normalizer

corpus = [...]  # list of documents

# Step 1: TF-IDF (D × V matrix, sparse)
vectorizer = TfidfVectorizer(max_features=50000, sublinear_tf=True)
X = vectorizer.fit_transform(corpus)
print(X.shape)  # (num_docs, 50000)

# Step 2: Truncated SVD (LSA)
svd = TruncatedSVD(n_components=300, random_state=42)
X_lsa = svd.fit_transform(X)  # (num_docs, 300)
print(f"Variance explained: {svd.explained_variance_ratio_.sum():.3f}")

# Step 3: L2 normalise for cosine similarity
X_lsa = Normalizer(copy=False).fit_transform(X_lsa)
print(X_lsa.shape)  # (num_docs, 300), dense, normalised
```

**Word vectors from PPMI matrix via SVD**:
```python
from scipy.sparse.linalg import svds
import numpy as np

# ppmi_matrix: sparse (vocab_size × vocab_size) PPMI matrix
U, S, Vt = svds(ppmi_matrix, k=300)

# Word vectors: rows of U * S (or U alone — hyperparameter)
word_vectors = U * S[:, np.newaxis].T   # (vocab_size, 300)
word_vectors = word_vectors / np.linalg.norm(word_vectors, axis=1, keepdims=True)
```

**Visualising sentence embeddings with UMAP**:
```python
import umap
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')
sentences = [...]   # list of sentences
labels = [...]      # topic labels for colouring

embeddings = model.encode(sentences)   # (N, 384)

reducer = umap.UMAP(n_components=2, metric='cosine', random_state=42)
embedding_2d = reducer.fit_transform(embeddings)  # (N, 2)

# Plot
import matplotlib.pyplot as plt
plt.scatter(embedding_2d[:, 0], embedding_2d[:, 1], c=labels, cmap='tab10', s=5)
plt.colorbar()
plt.title("UMAP of sentence embeddings")
plt.show()
```

**Choosing $d$ with the elbow plot**:
```python
svd = TruncatedSVD(n_components=500)
svd.fit(X)
cumvar = np.cumsum(svd.explained_variance_ratio_)
plt.plot(cumvar)
plt.xlabel("Number of components")
plt.ylabel("Cumulative variance explained")
plt.axhline(0.9, color='red', linestyle='--', label='90% threshold')
plt.legend()
```

---

## How It Connects to ML / NLP

| Dimensionality Reduction Concept | ML/NLP Link |
|---------------------------------|-------------|
| PCA / SVD | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/PCA.md]] — PCA is the canonical dimensionality reduction technique |
| Dimensionality reduction as unsupervised learning | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Dimensionality Reduction.md]] — general concept in ML |
| Truncated SVD = matrix factorisation | [[3.ML & DL/1.Concepts/11.Recommender Systems/Matrix Factorization.md]] — same mathematics as collaborative filtering |
| Choosing $d$ = model complexity decision | [[3.ML & DL/1.Concepts/8.Model Behavior/Model Complexity.md]] — too low $d$ → underfitting; too high → no compression benefit |
| Variance explained threshold | [[3.ML & DL/1.Concepts/8.Model Behavior/Bias Variance Tradeoff.md]] — $d$ selection trades off bias (information loss) and variance (dimensionality) |
| Clustering on reduced embeddings | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/K Means.md]] — K-Means on LSA or UMAP-reduced embeddings for topic discovery |
| LSA features for downstream classification | [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — LSA vectors used as input features to logistic regression / SVM |
| SVD gradient via Frobenius norm | [[3.ML & DL/1.Concepts/4.Loss and Cost/Cost Function.md]] — SVD minimises Frobenius norm reconstruction error |
| Random projections (fast approximation) | [[3.ML & DL/1.Concepts/5.Optimization/Convergence.md]] — Johnson-Lindenstrauss lemma guarantees approximate distance preservation |

---

## Common Interview Questions

**Q: What is LSA and how does it differ from LDA?**
A: **LSA (Latent Semantic Analysis)** applies SVD to a TF-IDF or count matrix to produce low-dimensional word/document vectors. It is a linear method that finds orthogonal dimensions of maximum variance. **LDA (Latent Dirichlet Allocation)** is a probabilistic generative model that models documents as mixtures of topics and topics as distributions over words. LDA produces interpretable topics (top-$k$ words per topic); LSA does not. LDA is slower but better for explicit topic discovery; LSA is faster and better for retrieval.

**Q: When would you use t-SNE vs UMAP for NLP visualisation?**
A: Both reduce to 2D for visualisation. t-SNE excels at revealing local cluster structure but distorts global distances — clusters are meaningful but inter-cluster distances are not. t-SNE is slow ($O(n^2)$, impractical for $n > 50{,}000$). UMAP preserves both local *and* global structure, is much faster ($O(n \log n)$), and supports out-of-sample projection. For most NLP visualisation tasks, UMAP is preferred. Neither should be used for feature extraction — only for visualisation.

**Q: What is the difference between PCA and SVD for dimensionality reduction?**
A: PCA = SVD applied to the mean-centred data matrix (equivalently, SVD of the covariance matrix). SVD can be applied directly to any matrix without centering. For NLP: LSA applies SVD directly to the TF-IDF matrix (no centering) because centering a sparse matrix would make it dense and prohibitively memory-intensive. The mathematical operations are equivalent when data is centred.

**Q: How do you choose the number of dimensions $d$ for LSA?**
A: Two approaches: (1) **Variance threshold**: choose $d$ such that the top-$d$ singular values explain 80–90% of total variance (elbow plot). (2) **Downstream task performance**: sweep $d \in \{50, 100, 200, 300, 500\}$ and pick the value that maximises validation performance on your task. For NLP tasks, $d = 100$–$300$ typically works well.

---

## Common Mistakes / Gotchas

- **Using t-SNE for feature extraction**: t-SNE is for *visualisation only*. It is non-parametric (no explicit mapping), stochastic (different runs give different layouts), and does not preserve global distances. Never use t-SNE embeddings as input to a downstream model.
- **Forgetting to normalise after SVD**: SVD output vectors are not L2-normalised. For cosine similarity, always normalise after decomposition. Unnormalised LSA vectors give length-biased similarity scores.
- **Using PCA on sparse matrices**: sklearn's PCA requires dense matrices. For large sparse TF-IDF matrices, use `TruncatedSVD` which operates on sparse matrices directly (via Lanczos/randomised SVD). `PCA` on a 50k × 100k matrix would require 20 GB of RAM.
- **Centering sparse matrices**: Mean-centering a sparse matrix destroys sparsity — every zero becomes a non-zero. For text, always use `TruncatedSVD` (no centering) rather than `PCA`. The difference is negligible empirically for text.
- **Over-interpreting UMAP plots**: UMAP layouts depend on hyperparameters (`n_neighbors`, `min_dist`). The *shape* of clusters and the *distances between* clusters change significantly with these settings. Always inspect with multiple hyperparameter settings before drawing conclusions.

---

## Further Reading / Paper References

- Deerwester et al. (1990). Indexing by Latent Semantic Analysis — original LSA
- Blei, Ng & Jordan (2003). Latent Dirichlet Allocation — the LDA paper (topic model alternative)
- van der Maaten & Hinton (2008). Visualizing Data using t-SNE — the t-SNE paper
- McInnes, Healy & Melville (2018). UMAP: Uniform Manifold Approximation and Projection arxiv:1802.03426
- Halko, Martinsson & Tropp (2011). Finding Structure with Randomness: Probabilistic Algorithms for Constructing Approximate Matrix Decompositions — randomised SVD
- sklearn docs: `TruncatedSVD`, `PCA`, `TSNE`, and install `umap-learn` separately
- Johnson & Lindenstrauss (1984). Extensions of Lipschitz mappings — theoretical foundation for random projections
