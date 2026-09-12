---
tags: [nlp, static-embeddings, word2vec, skip-gram, cbow, negative-sampling, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[GloVe]] [[FastText]] [[PMI and PPMI]] [[Co-occurrence Matrix]] [[BERT Embeddings]]"
---

# Word2Vec

## Definition + Intuition

**Word2Vec** (Mikolov et al., 2013) is a family of shallow neural network models that learn dense word embeddings from raw text using a **self-supervised** prediction objective. It comes in two architectures: **CBOW** (Continuous Bag of Words) and **Skip-gram**.

**Intuition**: Instead of counting co-occurrences (PMI) or factorising matrices (LSA), Word2Vec trains a neural network to *predict* words from context. The hidden layer's weights — never used for the actual prediction task — become the word embeddings. The embeddings learn because words in similar contexts must make similar predictions → similar contexts → similar weights.

The key contribution was not the architecture (a single hidden layer), but the training tricks — **negative sampling** and **subsampling of frequent words** — that made training on billion-word corpora feasible on a single machine.

**Famous result**: Linear arithmetic in embedding space: `king − man + woman ≈ queen`. This emerges from the distributional structure of the corpus, not from explicit encoding of gender relationships.

---

## Key Properties / Types

### CBOW — Continuous Bag of Words

**Task**: Predict the target word $w_t$ from its context words $\{w_{t-k}, \ldots, w_{t-1}, w_{t+1}, \ldots, w_{t+k}\}$.

- Input: average (or sum) of context word embeddings
- Output: probability distribution over vocabulary
- **Fast to train**, averages out context — works well for frequent words
- Better for smaller datasets

### Skip-gram

**Task**: Predict each context word from the target word $w_t$.

- Input: target word embedding
- Output: one probability distribution per context word
- **Slower to train** (multiple predictions per word), but learns better representations for rare words
- Better for large datasets and rare word representation
- The standard choice in practice

### Training Objectives

**Full softmax** (original, expensive):
$$P(c | w) = \frac{\exp(v_c \cdot v_w)}{\sum_{c' \in V} \exp(v_{c'} \cdot v_w)}$$

Computing the denominator over all $|V|$ words is $O(|V|)$ per step — too slow.

**Negative sampling** (practical): Replace full softmax with binary classification — distinguish real context pairs from noise-sampled pairs:
$$\mathcal{L} = \log \sigma(v_c \cdot v_w) + \sum_{i=1}^{k} \mathbb{E}_{c_i \sim P_n} \left[\log \sigma(-v_{c_i} \cdot v_w)\right]$$

Only $k+1$ words updated per step ($k \approx 5$–$20$). Training scales to billions of words.

**Hierarchical softmax** (alternative): Uses a Huffman binary tree over vocabulary — $O(\log |V|)$ per step. Better for rare words, but slower than negative sampling in practice.

---

## Math / Formal Notation

**Model parameters**: Two embedding matrices:
- $W \in \mathbb{R}^{|V| \times d}$: **input embeddings** (the final word vectors used downstream)
- $W' \in \mathbb{R}^{|V| \times d}$: **output/context embeddings** (discarded after training)

**Skip-gram with negative sampling (SGNS) loss**:
$$\mathcal{L} = -\sum_{(w,c) \in D^+} \log \sigma(v_c \cdot v_w) - \sum_{(w,c) \in D^-} \log \sigma(-v_c \cdot v_w)$$

where $D^+$ = real (word, context) pairs, $D^-$ = noise-sampled pairs.

**Noise distribution** for negative sampling:
$$P_n(w) \propto \text{count}(w)^{3/4}$$

The $3/4$ exponent smooths the unigram distribution — rare words are sampled more often than their raw frequency would suggest.

**Subsampling of frequent words**: Each word $w$ is discarded during training with probability:
$$P(\text{discard}) = 1 - \sqrt{\frac{t}{\text{freq}(w)}}$$

where $t \approx 10^{-5}$ is a threshold. Common words ("the", "a") are heavily subsampled. This speeds training and improves representation of content words.

**Connection to PPMI** (Levy & Goldberg, 2014):
$$W W'^\top \approx \text{PPMI}(w, c) - \log k$$

SGNS implicitly factorises the shifted PPMI matrix. This makes Word2Vec theoretically grounded in classical distributional semantics.

---

## Examples (Concrete)

**Word analogies** (skip-gram trained on Google News, 300d):
```
king − man + woman = queen       (gender analogy)
Paris − France + Italy = Rome    (capital-country analogy)
biggest − big + small = smallest (morphological analogy)
```

These work because the embedding space encodes linear relational structure.

**Nearest neighbours for "python"** (trained on programming corpus):
→ `java, ruby, perl, php, javascript` (programming language sense)

**Nearest neighbours for "python"** (trained on biology corpus):
→ `boa, anaconda, cobra, viper, serpent` (snake sense)

This shows the key limitation: one word = one vector. Both senses share the same embedding.

**Python (gensim)**:
```python
from gensim.models import Word2Vec

sentences = [["the", "cat", "sat"], ["the", "dog", "ran"], ...]

model = Word2Vec(
    sentences,
    vector_size=300,   # embedding dimension
    window=5,          # context window ±5
    min_count=5,       # ignore words with freq < 5
    sg=1,              # 1=skip-gram, 0=CBOW
    negative=10,       # number of negative samples
    epochs=5
)

model.wv["king"]                           # 300-dim vector
model.wv.most_similar("king", topn=5)      # nearest neighbours
model.wv.most_similar(positive=["king","woman"], negative=["man"])  # analogy
```

---

## How It Connects to ML / NLP

| Word2Vec Concept | ML/NLP Link |
|-----------------|-------------|
| Embedding layer = learned lookup table | [[3.ML & DL/1.Concepts/3.Linear Regression/Parameters.md]] — embeddings are model parameters updated by gradient descent |
| Binary cross-entropy loss (negative sampling) | [[3.ML & DL/1.Concepts/4.Loss and Cost/Logistic Loss.md]] — SGNS loss is logistic (binary cross-entropy) |
| SGD over pairs | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — stochastic gradient descent with learning rate schedule |
| Subsampling frequent words | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — reduces over-representation of common words, a form of data regularisation |
| Rare word representation | [[3.ML & DL/1.Concepts/8.Model Behavior/Bias Variance Tradeoff.md]] — skip-gram better for rare words (higher variance reduction than CBOW) |
| Negative sampling distribution | [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — noise distribution shapes what the model learns |
| Implicit PPMI factorisation | [[3.ML & DL/1.Concepts/11.Recommender Systems/Matrix Factorization.md]] — SGNS is matrix factorisation by gradient descent |
| Word vectors as features | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Vector.md]] — pre-trained vectors initialise downstream model inputs |

---

## Common Interview Questions

**Q: What is the difference between CBOW and Skip-gram?**
A: CBOW predicts a target word from its context (averages context vectors → predicts centre). Skip-gram predicts each context word from the target word. Skip-gram is slower but produces better embeddings for rare words, because it makes $2k$ predictions per word instead of one. CBOW is faster and works better for frequent words with sufficient data.

**Q: What is negative sampling and why is it used?**
A: Full softmax over the vocabulary is $O(|V|)$ per gradient step — too slow for $|V| = 10^6$. Negative sampling replaces this with binary classification: is this (word, context) pair real or noise-sampled? Only $k+1$ embeddings are updated per step instead of all $|V|$, reducing computation by 4–5 orders of magnitude.

**Q: Why does Word2Vec use a $3/4$ power for the noise distribution?**
A: The raw unigram distribution is very skewed — a few words are extremely common. Sampling from it would mean negative examples are almost always stopwords, giving the model a trivial discrimination task. The $3/4$ power smooths this, giving rare words a higher probability of being selected as negatives.

**Q: What does "king − man + woman = queen" demonstrate?**
A: It shows that relational structure is encoded as linear offsets in the embedding space. The vector difference `king − man` encodes the concept of "royalty without male-ness". Adding `woman` shifts to the female royal concept. This works because words in similar relational contexts learn similar geometric relationships. It is not perfect — the analogy test fails for ~70% of analogies.

**Q: What is the key limitation of Word2Vec?**
A: Each word has exactly one vector regardless of context — it cannot handle polysemy. "Bank" gets the same vector in "river bank" and "bank account". ELMo and BERT solve this with contextual representations.

---

## Common Mistakes / Gotchas

- **Using pre-trained vectors from wrong domain**: Word2Vec trained on Google News will have poor representations for medical, legal, or code text. Domain-specific training or fine-tuning is needed.
- **Ignoring OOV words**: Word2Vec has no mechanism for words not seen during training. Solutions: (1) use FastText (subword embeddings), (2) use the average of known sub-words, (3) use a special `<UNK>` token.
- **Not normalising before similarity**: Raw Word2Vec vectors are not unit-normalised. Always L2-normalise before computing cosine similarity.
- **Conflating input and output embeddings**: Word2Vec trains two matrices $W$ and $W'$. Downstream tasks typically use $W$ (input embeddings). Some applications average $W$ and $W'$ (can improve performance slightly).
- **Using CBOW for rare words**: If your vocabulary has many rare domain-specific terms, use skip-gram. CBOW averages context, which dilutes the signal for rare words.

---

## Further Reading / Paper References

- Mikolov et al. (2013). Efficient Estimation of Word Representations in Vector Space — original Word2Vec paper arxiv:1301.3781
- Mikolov et al. (2013). Distributed Representations of Words and Phrases — negative sampling, subsampling arxiv:1310.4546
- Levy & Goldberg (2014). Neural Word Embedding as Implicit Matrix Factorization — SGNS ≈ PPMI factorisation arxiv:1411.2738
- Goldberg & Levy (2014). word2vec Explained — clear mathematical derivation
- Gensim documentation: `gensim.models.Word2Vec`
- Jurafsky & Martin, SLP Ch. 6 — Word Vectors, full textbook treatment
