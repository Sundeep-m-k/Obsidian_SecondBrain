---
tags: [nlp, sparse-representations, bag-of-words, text-to-numbers, classical-nlp]
links: "[[03_Text_to_Numbers/Index]] [[TF-IDF]] [[PMI and PPMI]] [[Tokenization — BPE]]"
---

# Bag of Words

## Definition + Intuition

The **Bag of Words (BoW)** model represents a text document as an unordered collection of its words, ignoring grammar and word order but keeping word counts. Each document becomes a vector of length $|V|$ (vocabulary size) where each dimension records how many times that word appears.

**Intuition**: Imagine tipping a document into a bag and shaking it — the words are all there, but you've lost the order. "The dog bit the man" and "The man bit the dog" produce identical BoW vectors. The model bets that *which* words appear tells you more about a document's topic than *how* they're ordered.

This bet is often correct for topic classification, spam detection, and document retrieval — wrong for sentiment (negation) and all tasks requiring syntax.

---

## Key Properties / Types

**One-Hot Encoding (special case)**
Each word is represented as a vector of zeros with a single 1 at its position in the vocabulary. Documents are then a sum of their word one-hot vectors.

**Count Vectors**
Raw frequency: dimension $i$ = number of times word $i$ appears in the document. Most common BoW variant.

**Binary Vectors**
Dimension $i$ = 1 if word $i$ appears at least once, 0 otherwise. Loses frequency information but reduces effect of repeated words.

**Document-Term Matrix (DTM)**
The matrix $M \in \mathbb{R}^{D \times V}$ where $D$ = number of documents, $V$ = vocabulary size. Row $i$ = BoW vector for document $i$. Column $j$ = distribution of word $j$ across documents.

Properties of BoW vectors:
- **Sparse**: Most entries are 0 (a document uses $\ll |V|$ unique words)
- **High-dimensional**: $|V|$ often 50k–500k
- **Interpretable**: dimension $i$ directly corresponds to word $i$
- **Order-invariant**: syntactic structure is lost
- **No OOV handling**: words not in vocabulary are silently ignored

---

## Math / Formal Notation

**Vocabulary**: $V = \{w_1, w_2, \ldots, w_{|V|}\}$ — the set of all unique tokens seen during training.

**Count vector for document $d$**:
$$\text{BoW}(d)_i = \text{count}(w_i, d) = \sum_{t \in d} \mathbf{1}[t = w_i]$$

**Document-Term Matrix**:
$$M_{ij} = \text{count}(w_j, d_i)$$

**Cosine similarity** between two document vectors (standard similarity metric for BoW):
$$\text{sim}(d_1, d_2) = \frac{\text{BoW}(d_1) \cdot \text{BoW}(d_2)}{|\text{BoW}(d_1)| \cdot |\text{BoW}(d_2)|}$$

**Jaccard similarity** (for binary vectors):
$$J(d_1, d_2) = \frac{|S_1 \cap S_2|}{|S_1 \cup S_2|}$$
where $S_i$ is the set of words appearing in $d_i$.

---

## Examples (Concrete)

**Vocabulary**: `{cat, dog, sat, mat, the}` → indices `{0:cat, 1:dog, 2:sat, 3:mat, 4:the}`

| Document | Text | BoW Vector |
|----------|------|-----------|
| $d_1$ | "the cat sat on the mat" | `[1, 0, 1, 1, 2]` |
| $d_2$ | "the dog sat on the mat" | `[0, 1, 1, 1, 2]` |
| $d_3$ | "cat cat cat" | `[3, 0, 0, 0, 0]` |

Cosine similarity: $\text{sim}(d_1, d_2) = \frac{0+0+1+1+4}{\sqrt{7}\cdot\sqrt{7}} \approx 0.857$ — correctly identifies these as similar documents.

**Python (scikit-learn)**:
```python
from sklearn.feature_extraction.text import CountVectorizer

docs = ["the cat sat on the mat", "the dog sat on the mat", "cat cat cat"]
vectorizer = CountVectorizer()
X = vectorizer.fit_transform(docs)
print(vectorizer.vocabulary_)  # {'cat': 0, 'dog': 2, 'mat': 3, 'on': 4, 'sat': 5, 'the': 6}
print(X.toarray())
# [[1 0 1 1 1 1 2]
#  [0 1 1 1 1 1 2]
#  [3 0 0 0 0 0 0]]
```

---

## How It Connects to ML / NLP

| BoW Concept | ML/NLP Application |
|------------|-------------------|
| Document-term matrix | Direct input to [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] (Naïve Bayes, SVM, Logistic Regression) |
| Vocabulary = feature space | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — each vocab word is a feature |
| Sparse high-dim vectors | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — L1 regularization critical for sparse feature selection |
| Count normalization | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Scaling.md]] — raw counts must be normalized |
| DTM as matrix | [[3.ML & DL/1.Concepts/10.Unsupervised Learning/PCA.md]] — PCA/SVD reduces DTM to dense embeddings (→ LSA) |

**Downstream tasks where BoW is still competitive**:
- **Spam classification**: word presence alone is highly predictive
- **Topic classification** (short, topically coherent docs): fast, interpretable
- **Naïve Bayes**: theoretically optimal classifier for BoW under word-independence assumption

**Where BoW fails**:
- **Sentiment**: "not bad" → same BoW as "bad not" — negation is invisible
- **Semantics**: "car" and "automobile" are orthogonal dimensions — no notion of synonymy
- **Long-range dependency**: any task requiring word order

---

## Common Interview Questions

**Q: What are the main limitations of Bag of Words?**
A: (1) Loses word order and syntactic structure. (2) Treats synonyms as orthogonal (no semantic similarity). (3) High-dimensional sparse vectors (curse of dimensionality). (4) Common words dominate — fixed by TF-IDF. (5) No OOV handling for words unseen at training time.

**Q: What is the document-term matrix and how is it used?**
A: A matrix $M \in \mathbb{R}^{D \times V}$ where rows are documents and columns are vocabulary words. Entry $M_{ij}$ = count of word $j$ in document $i$. Used directly as input to ML classifiers, or decomposed via SVD for LSA word embeddings.

**Q: When would you choose BoW over word embeddings?**
A: When interpretability matters, when training data is small (embeddings may overfit), when the task is topic-level (word order doesn't matter), or when computational resources are limited (BoW requires no training).

**Q: What is the difference between BoW and one-hot encoding?**
A: One-hot encodes individual *words* — each word is a vector of zeros with one 1. BoW encodes *documents* — each document is the sum (or count-weighted combination) of its word one-hot vectors.

---

## Common Mistakes / Gotchas

- **Not removing stopwords**: High-frequency words like "the", "a", "is" dominate all vectors. Stopword removal or TF-IDF weighting is essential.
- **Not normalizing document length**: A 10-word doc and a 1000-word doc will have very different count magnitudes. Always use L2 normalization or term frequency (divide by doc length) before computing similarity.
- **Fitting vectorizer on test set**: The vocabulary must be built on training data only. Test documents with OOV words silently lose those words.
- **Assuming word independence**: Naïve Bayes BoW assumes feature independence, which is false but often works fine empirically.
- **Ignoring n-grams**: Unigram BoW misses "not good" → bigram BoW `CountVectorizer(ngram_range=(1,2))` captures short phrases.

---

## Further Reading / Paper References

- Joachims, T. (1998). Text Categorization with Support Vector Machines — seminal SVM + BoW paper
- McCallum & Nigam (1998). A Comparison of Event Models for Naïve Bayes Text Classification
- Scikit-learn docs: `sklearn.feature_extraction.text.CountVectorizer`
- Jurafsky & Martin, SLP Ch. 6 — Vector Semantics and Embeddings (context for BoW limitations)
