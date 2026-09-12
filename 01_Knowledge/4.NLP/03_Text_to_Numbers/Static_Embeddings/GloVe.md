---
tags: [nlp, static-embeddings, glove, co-occurrence, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Word2Vec]] [[FastText]] [[Co-occurrence Matrix]] [[PMI and PPMI]]"
---

# GloVe

## Definition + Intuition

**GloVe (Global Vectors for Word Representation)** (Pennington et al., 2014) learns word embeddings by training a log-bilinear model to predict global **log co-occurrence counts** across the entire corpus. Unlike Word2Vec, which trains on local context windows one at a time, GloVe explicitly uses the **global co-occurrence statistics** of the corpus.

**Intuition**: Word2Vec sees word pairs one at a time (online/stochastic). GloVe first builds the full co-occurrence matrix from the corpus, then trains embeddings to reconstruct it. This makes GloVe a hybrid: it has the interpretable objective of co-occurrence matrix methods (LSA) and the efficient neural training of Word2Vec.

**Key insight from the paper**: The ratio of co-occurrence probabilities is more informative than raw co-occurrence probabilities. Consider:

| Ratio | ice | steam |
|-------|-----|-------|
| $P(k \mid \text{ice}) / P(k \mid \text{steam})$ for $k = $ solid | **large** | small |
| $k = $ gas | small | **large** |
| $k = $ water | medium | medium |
| $k = $ fashion | ~1 | ~1 |

The ratio discriminates relevant from irrelevant context words. GloVe trains embeddings to encode these ratios in dot products.

---

## Key Properties / Types

**GloVe vs Word2Vec — key differences**:

| Property | Word2Vec (SGNS) | GloVe |
|----------|----------------|-------|
| Training signal | Local window pairs | Global co-occurrence matrix |
| Objective | Predict context word | Reconstruct log co-occurrence |
| Loss | Binary cross-entropy | Weighted least squares |
| Theoretical basis | Implicit PPMI factorisation | Explicit log co-occurrence factorisation |
| Training | Stochastic (online) | Batch or mini-batch |
| OOV handling | None | None |
| Practical performance | Similar | Similar |

In practice, both produce embeddings of similar quality. GloVe is sometimes easier to reason about theoretically.

**Weighting function $f(X_{ij})$**:

GloVe uses a weighting function to reduce the impact of very frequent co-occurrences (which are often uninformative):
$$f(x) = \begin{cases} (x/x_{\max})^\alpha & \text{if } x < x_{\max} \\ 1 & \text{otherwise} \end{cases}$$

Typical values: $x_{\max} = 100$, $\alpha = 3/4$. Very frequent pairs (e.g., "the" with anything) are down-weighted; rare but genuine pairs are weighted fairly.

---

## Math / Formal Notation

**Co-occurrence matrix**: $X_{ij}$ = number of times word $j$ appears in the context of word $i$ (distance-weighted).

**Core observation**: For three words $i$, $j$, $k$, the ratio of conditional probabilities should be encoded in word vectors:
$$F(v_i, v_j, \tilde{v}_k) = \frac{P_{ik}}{P_{jk}}$$

where $v_i, v_j$ are word vectors and $\tilde{v}_k$ is the context vector for $k$.

**GloVe objective** (solving for $F$):

$$v_i \cdot \tilde{v}_j + b_i + \tilde{b}_j = \log X_{ij}$$

**Weighted least squares loss**:
$$\mathcal{L} = \sum_{i,j=1}^{|V|} f(X_{ij}) \left( v_i \cdot \tilde{v}_j + b_i + \tilde{b}_j - \log X_{ij} \right)^2$$

Parameters to learn: word vectors $v_i \in \mathbb{R}^d$, context vectors $\tilde{v}_j \in \mathbb{R}^d$, bias terms $b_i, \tilde{b}_j \in \mathbb{R}$.

**Final word vectors**: Average of word and context vectors:
$$v_{\text{final},i} = v_i + \tilde{v}_i$$

This averaging exploits the symmetry of the problem and empirically improves performance.

**Connection to PMI**: Taking the log:
$$v_i \cdot \tilde{v}_j \approx \log X_{ij} - b_i - \tilde{b}_j \approx \log P(i,j) - \log P(i) - \log P(j) = \text{PMI}(i,j)$$

GloVe's dot product approximates PMI — the same quantity PPMI captures explicitly and SGNS captures implicitly.

---

## Examples (Concrete)

**Analogies** (trained on 840B tokens Common Crawl, 300d):
```
man − woman + queen = king
Paris − France + Germany = Berlin
walked − walk + run = ran
```

**Semantic nearest neighbours for "frog"**:
→ `frogs, toad, litoria, rana, toadstool` (similar distributional context in biology text)

**Named entity clustering**: GloVe embeddings naturally cluster countries, cities, person names when visualised with t-SNE, because they co-occur in similar contexts in news corpora.

**Python (loading pre-trained GloVe)**:
```python
import numpy as np

def load_glove(path, dim=300):
    embeddings = {}
    with open(f"{path}/glove.6B.{dim}d.txt", encoding="utf-8") as f:
        for line in f:
            values = line.split()
            word = values[0]
            vector = np.array(values[1:], dtype=np.float32)
            embeddings[word] = vector
    return embeddings

glove = load_glove("./glove.6B")

# Word arithmetic
king = glove["king"]
man  = glove["man"]
woman = glove["woman"]
target = king - man + woman  # should be close to glove["queen"]

# Find nearest neighbour
from numpy.linalg import norm
sims = {w: np.dot(target, v) / (norm(target) * norm(v)) for w, v in glove.items()}
print(sorted(sims, key=sims.get, reverse=True)[:5])
# ['queen', 'king', 'princess', 'monarch', 'throne']
```

**Training GloVe from scratch (C implementation)**:
```bash
# Build co-occurrence matrix
./vocab_count -min-count 5 -verbose 2 < corpus.txt > vocab.txt
./cooccur -memory 4.0 -vocab-file vocab.txt -verbose 2 -window-size 10 \
    < corpus.txt > cooccurrences.bin
./shuffle -memory 4.0 -verbose 2 < cooccurrences.bin > cooccurrences.shuf.bin
./glove -save-file vectors -threads 8 -input-file cooccurrences.shuf.bin \
    -x-max 100 -iter 15 -vector-size 300 -binary 2 -vocab-file vocab.txt
```

---

## How It Connects to ML / NLP

| GloVe Concept | ML/NLP Link |
|--------------|-------------|
| Weighted least squares objective | [[3.ML & DL/1.Concepts/4.Loss and Cost/Mean Squared Error.md]] — GloVe loss is MSE with a weighting function |
| Minimising reconstruction error | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — trained via AdaGrad (adaptive gradient descent) |
| Dot product ≈ log co-occurrence | [[3.ML & DL/1.Concepts/3.Linear Regression/Hypothesis Function.md]] — bilinear model (dot product) as the hypothesis |
| Weighting function reducing outlier influence | [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — $f(X_{ij})$ prevents high-frequency pairs from dominating |
| Bias terms $b_i$, $\tilde{b}_j$ | [[3.ML & DL/1.Concepts/3.Linear Regression/Parameters.md]] — bias absorbs marginal log-probabilities |
| Pre-trained GloVe as feature | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — GloVe initialises embedding layers in downstream models |
| Global matrix = batch learning | [[3.ML & DL/1.Concepts/5.Optimization/Batch Gradient Descent.md]] — GloVe trains on the full co-occurrence matrix |

**Pre-trained GloVe resources**:
- `glove.6B` — trained on 6B tokens Wikipedia + Gigaword, dims 50/100/200/300
- `glove.840B.300d` — trained on 840B tokens Common Crawl, 300d
- `glove.twitter.27B` — trained on 27B tweet tokens, 25/50/100/200d (best for informal text)

**When to use which pre-trained embedding**:
- Formal text (news, academic) → `glove.6B` or `glove.840B`
- Social media, informal → `glove.twitter`
- Biomedical → retrain GloVe on PubMed or use BioWordVec
- Code → retrain on GitHub corpora

---

## Common Interview Questions

**Q: How does GloVe differ from Word2Vec?**
A: Word2Vec trains on local sliding windows stochastically — it sees word pairs one at a time. GloVe first builds the global co-occurrence matrix from the full corpus, then trains a log-bilinear model to reconstruct log co-occurrence counts using weighted least squares. GloVe explicitly optimises a global objective; Word2Vec implicitly approximates it locally. Both produce similar quality embeddings in practice.

**Q: What does GloVe's loss function optimise?**
A: Weighted least squares reconstruction of log co-occurrence counts: $\mathcal{L} = \sum_{i,j} f(X_{ij})(v_i \cdot \tilde{v}_j + b_i + \tilde{b}_j - \log X_{ij})^2$. The weighting $f(X_{ij})$ down-weights very frequent pairs. The model learns that word vector dot products should approximate PMI.

**Q: Why does GloVe average the word and context vectors?**
A: The GloVe objective is symmetric: the roles of word and context are interchangeable (both $v_i \cdot \tilde{v}_j$ and $v_j \cdot \tilde{v}_i$ should equal $\log X_{ij}$). Averaging exploits this symmetry and empirically improves word similarity benchmarks by ~1–2 points compared to using word vectors alone.

**Q: What is the weighting function in GloVe and why is it needed?**
A: $f(X_{ij}) = \min(1, (X_{ij}/x_{\max})^{3/4})$. Without it, very frequent co-occurrences (e.g., "the" with any word) would dominate the loss function and the model would spend most of its capacity learning to represent these uninformative pairs. The weighting caps influence of high-frequency pairs and gives appropriate weight to rare but genuine associations.

---

## Common Mistakes / Gotchas

- **Treating GloVe as fundamentally different from Word2Vec**: They both approximate PMI and produce similar embeddings. The theoretical and empirical differences are smaller than the literature sometimes implies.
- **Not averaging word and context vectors**: Many implementations return only $v_i$. Using $v_i + \tilde{v}_i$ consistently improves benchmark performance at zero cost.
- **Using `glove.6B` for domain-specific text**: General web/Wikipedia GloVe has poor coverage for technical domains (medical, legal, scientific). Retrain on domain data or use domain-adapted embeddings.
- **OOV handling**: GloVe has no subword model. OOV words require a fallback (zero vector, mean of known words, or switching to FastText). Never silently use zero vectors without flagging this.
- **Memory**: `glove.840B.300d` is 5.6 GB. Load lazily or use `gensim.downloader` which provides memory-mapped access.

---

## Further Reading / Paper References

- Pennington, J., Socher, R. & Manning, C.D. (2014). GloVe: Global Vectors for Word Representation arxiv:1405.0312
- Levy, O. & Goldberg, Y. (2014). Linguistic Regularities in Sparse and Explicit Word Representations — comparison of PPMI, SVD, SGNS, GloVe
- Levy, O., Goldberg, Y. & Dagan, I. (2015). Improving Distributional Similarity with Lessons Learned from Word Embeddings — best practices for GloVe/Word2Vec hyperparameters
- Stanford GloVe page: https://nlp.stanford.edu/projects/glove/ — pre-trained vectors download
- Jurafsky & Martin, SLP Ch. 6.5 — GloVe, textbook treatment
