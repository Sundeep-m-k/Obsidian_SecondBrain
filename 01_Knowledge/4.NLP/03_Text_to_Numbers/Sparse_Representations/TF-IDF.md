---
tags: [nlp, sparse-representations, tf-idf, information-retrieval, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Bag of Words]] [[PMI and PPMI]] [[Co-occurrence Matrix]]"
---

# TF-IDF

## Definition + Intuition

**TF-IDF (Term Frequency–Inverse Document Frequency)** is a numerical statistic that reflects how important a word is to a document within a collection (corpus). It is the product of two terms: **TF** (how often the word appears in *this* document) and **IDF** (how rare the word is across *all* documents).

**Intuition**: Raw word counts (BoW) are dominated by common words like "the", "is", "and" which appear in every document but carry no discriminative signal. TF-IDF penalises words that appear in many documents (low IDF) and rewards words that appear frequently in *this* document but rarely elsewhere (high TF, high IDF).

Concrete example: The word "the" appears in every document → IDF ≈ 0, TF-IDF ≈ 0. The word "mitochondria" appears in 3 biology papers out of 10,000 → very high IDF, so it gets a high TF-IDF score in those papers.

---

## Key Properties / Types

**Term Frequency (TF)**
How often a word appears in a document. Several variants:

| TF Variant | Formula | When to Use |
|-----------|---------|------------|
| Raw count | $\text{count}(t, d)$ | Rarely used alone |
| Relative frequency | $\frac{\text{count}(t, d)}{|d|}$ | Standard; normalises for doc length |
| Log-normalised | $1 + \log \text{count}(t, d)$ | Dampens effect of very frequent terms |
| Binary | $\mathbf{1}[\text{count}(t,d) > 0]$ | When frequency doesn't matter |
| Double normalisation | $0.5 + 0.5 \cdot \frac{\text{count}(t,d)}{\max_{t'}\text{count}(t', d)}$ | Prevents bias toward longer documents |

**Inverse Document Frequency (IDF)**
How rare a word is across the corpus. Several variants:

| IDF Variant | Formula | Effect |
|------------|---------|--------|
| Standard IDF | $\log \frac{N}{df(t)}$ | Words in all docs → 0 |
| Smooth IDF (sklearn) | $\log \frac{N+1}{df(t)+1} + 1$ | Prevents zero IDF for common words |
| Probabilistic IDF | $\log \frac{N - df(t)}{df(t)}$ | Laplace-smoothed variant |

Where $N$ = total documents, $df(t)$ = number of documents containing term $t$.

**TF-IDF score**:
$$\text{TF-IDF}(t, d, D) = \text{TF}(t, d) \times \text{IDF}(t, D)$$

**Document representation**: a TF-IDF vector in $\mathbb{R}^{|V|}$ — same dimensionality as BoW, but with re-weighted values. Typically L2-normalised before use.

---

## Math / Formal Notation

**Standard TF-IDF** (most common form):
$$\text{TF-IDF}(t, d) = \underbrace{\frac{\text{count}(t, d)}{|d|}}_{\text{TF}} \times \underbrace{\log \frac{N}{|\{d' \in D : t \in d'\}|}}_{\text{IDF}}$$

**Log-normalised TF with smooth IDF** (scikit-learn default):
$$\text{TF-IDF}(t, d) = \bigl(1 + \log \text{count}(t, d)\bigr) \times \left(\log \frac{N+1}{df(t)+1} + 1\right)$$

**BM25** — the modern IR extension of TF-IDF that saturates TF and adjusts for document length:
$$\text{BM25}(t, d) = \frac{\text{count}(t,d) \cdot (k_1 + 1)}{\text{count}(t,d) + k_1 \left(1 - b + b \cdot \frac{|d|}{\text{avgdl}}\right)} \cdot \log\frac{N - df(t) + 0.5}{df(t) + 0.5}$$

where $k_1 \in [1.2, 2.0]$ (TF saturation), $b = 0.75$ (length normalisation).

**L2 normalisation** (applied after computing TF-IDF):
$$\hat{v} = \frac{v}{||v||_2}$$

After L2 normalisation, cosine similarity reduces to dot product: $\text{sim}(d_1, d_2) = \hat{v}_1 \cdot \hat{v}_2$.

---

## Examples (Concrete)

**Corpus** ($N = 4$ documents):
```
d1: "the cat sat on the mat"
d2: "the cat ate the rat"
d3: "the dog chased the cat"
d4: "the programmer wrote code"
```

| Term | df | IDF (log N/df) | TF in d1 | TF-IDF in d1 |
|------|----|---------------|----------|--------------|
| the | 4 | log(4/4)=0.0 | 2/6=0.33 | **0.000** |
| cat | 3 | log(4/3)=0.29 | 1/6=0.17 | **0.048** |
| sat | 1 | log(4/1)=1.39 | 1/6=0.17 | **0.231** |
| mat | 1 | log(4/1)=1.39 | 1/6=0.17 | **0.231** |

"The" gets TF-IDF = 0 despite being the most frequent word in d1 — exactly the desired behaviour.

**Python (scikit-learn)**:
```python
from sklearn.feature_extraction.text import TfidfVectorizer

docs = ["the cat sat on the mat", "the cat ate the rat",
        "the dog chased the cat", "the programmer wrote code"]

tfidf = TfidfVectorizer()
X = tfidf.fit_transform(docs)

# Get TF-IDF scores for first document
feature_names = tfidf.get_feature_names_out()
doc0_scores = zip(feature_names, X[0].toarray()[0])
print(sorted(doc0_scores, key=lambda x: -x[1]))
# [('sat', 0.58), ('mat', 0.58), ('cat', 0.40), ('on', 0.40), ('the', 0.0)]
```

---

## How It Connects to ML / NLP

| TF-IDF Concept | ML/NLP Application |
|---------------|-------------------|
| TF-IDF vectors as features | Input to [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — direct input to Naïve Bayes, SVM, Logistic Regression |
| IDF = feature re-weighting | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Scaling.md]] — TF-IDF is a learned scaling of raw count features |
| L2 normalisation | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Normalization.md]] — required before cosine similarity |
| BM25 TF saturation | [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — prevents long docs from dominating via TF cap |
| IDF as log-probability | [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — IDF is related to negative log probability of a term |

**Where TF-IDF is used in practice**:
- **Search engines**: BM25 (TF-IDF variant) is the backbone of Elasticsearch/Solr full-text search
- **Keyword extraction**: top-k words by TF-IDF score per document
- **Document similarity / clustering**: cosine similarity on L2-normalised TF-IDF vectors
- **Feature input to ML classifiers**: still competitive for document classification when data is small
- **Hybrid RAG**: sparse TF-IDF retrieval combined with dense vector search (BM25 + FAISS)

**Why dense embeddings haven't fully replaced TF-IDF**:
- TF-IDF is exact keyword match — never misses a document containing the query term
- Dense embeddings are fuzzy — can miss rare or technical terms
- Production search systems use **hybrid retrieval**: TF-IDF (recall) + dense (precision)

---

## Common Interview Questions

**Q: What does IDF measure? Why is it important?**
A: IDF measures how rare a term is across the corpus. It down-weights words that appear in many documents (like "the", "is") which carry no discriminative power, and up-weights rare, topic-specific words (like "mitochondria" or "recursion"). Without IDF, common words dominate all vector representations.

**Q: What is the difference between TF-IDF and BM25?**
A: BM25 adds two improvements: (1) **TF saturation** — in TF-IDF, TF grows linearly, so a word appearing 100 times is 100× more important than one appearing once. BM25 uses a saturating function: benefit diminishes after a word appears many times. (2) **Document length normalisation** — BM25 adjusts for document length with a tunable parameter $b$, preventing long documents from unfairly dominating.

**Q: How do you handle words not seen during training in TF-IDF?**
A: OOV words are silently ignored — the vocabulary is fixed at fit time. Unlike neural embeddings, TF-IDF has no mechanism to handle OOV words. One solution: use character n-gram TF-IDF which can partially represent OOV words from character patterns.

**Q: Why is L2 normalisation applied to TF-IDF vectors before computing similarity?**
A: Without L2 normalisation, longer documents have higher magnitude vectors and will always have higher cosine similarity scores. Normalisation makes cosine similarity purely about the angle (direction) between vectors, independent of document length.

---

## Common Mistakes / Gotchas

- **Fitting on test data**: IDF must be computed on training corpus only. Including test documents inflates IDF values for rare test-only terms.
- **Not normalising**: Forgetting L2 normalisation before cosine similarity causes length bias — longer documents appear more similar to everything.
- **Using raw IDF without smoothing**: If a test term never appeared in training, $df(t) = 0$ → division by zero in standard IDF. Use smooth IDF: $\log\frac{N+1}{df(t)+1}$.
- **Expecting TF-IDF to capture synonymy**: "car" and "automobile" remain orthogonal dimensions. For semantic similarity, use dense embeddings + TF-IDF as a hybrid.
- **Ignoring stopword lists**: sklearn's `TfidfVectorizer(stop_words='english')` removes common English words, but domain-specific stopwords (e.g., "patient" in medical text) must be added manually.

---

## Further Reading / Paper References

- Sparck Jones, K. (1972). A statistical interpretation of term specificity and its application in retrieval — original IDF paper
- Robertson, S. & Zaragoza, H. (2009). The Probabilistic Relevance Framework: BM25 and Beyond
- Manning, Raghavan & Schütze, *Introduction to Information Retrieval* Ch. 6 — free online, definitive reference
- Scikit-learn docs: `sklearn.feature_extraction.text.TfidfVectorizer`
- Lin et al. (2021). A Few Brief Notes on DeepImpact, COIL, and a Conceptual Framework for Learned Sparse Retrieval — modern learned sparse representations
