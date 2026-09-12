# Deduplication

tags: #nlp #text-preprocessing #cleaning #deduplication #data-quality
links: [[Noise Removal]] [[HTML and Markup Stripping]] [[Encoding and Unicode]]

---

## Definition + Intuition

**Deduplication** is the process of identifying and removing duplicate or near-duplicate documents (or passages) from a text corpus, so that each piece of information appears at most once in the training data.

- **Exact deduplication**: two documents are identical at the character level.
- **Near-deduplication**: two documents are very similar but not identical (minor edits, template differences, scraped reprints).

> **Intuition**: The web is full of mirrors. A news article published by AP is reprinted verbatim on thousands of outlets. A product description is copy-pasted across hundreds of e-commerce pages. If you train on Common Crawl without deduplication, your model sees the same Wikipedia article ~400 times and the same AP story ~2000 times. This teaches the model to memorise boilerplate rather than learn language. Deduplication is removing the mirrors, keeping only originals.

---

## Key Properties / Types

### Why Deduplication Matters

1. **Memorisation**: models trained on repeated data memorise it verbatim — exact training examples can be extracted from GPT-style models if they appear often enough.
2. **Skewed frequency**: duplicated text makes some phrases artificially frequent, distorting the model's statistical intuitions.
3. **Benchmark contamination**: test sets (SQuAD, MMLU) may appear in training data via web crawls. Deduplication reduces the chance of test contamination.
4. **Training efficiency**: no value in processing the same document 400 times — deduplication reduces compute waste.
5. **Chinchilla finding**: training on deduplicated data allows the same model performance with fewer tokens → more token-efficient training.

### Deduplication Levels

| Level | What is compared | Granularity |
|-------|-----------------|------------|
| **Document-level** | Full documents | Coarse — fast, misses partial duplicates |
| **Paragraph-level** | Paragraph blocks | Medium — catches boilerplate paragraphs reprinted across docs |
| **Sentence-level** | Individual sentences | Fine — very slow; used for specific quality pipelines |
| **N-gram overlap** | Suffix arrays over n-grams | Efficient exact + near-dedup simultaneously |

### Exact Deduplication

**Hashing**: compute a hash of each document (after normalisation), store in a set. Reject documents whose hash already exists.

```python
import hashlib

def doc_hash(text):
    # normalise before hashing
    text = text.lower().split()           # case-fold + whitespace normalise
    text = ' '.join(text)
    return hashlib.md5(text.encode()).hexdigest()

seen = set()
deduped = []
for doc in corpus:
    h = doc_hash(doc)
    if h not in seen:
        seen.add(h)
        deduped.append(doc)
```

### Near-Deduplication with MinHash + LSH

**MinHash** approximates the **Jaccard similarity** between documents without comparing all pairs ($O(n^2)$).

**Jaccard similarity** between sets $A$ and $B$:
$$J(A, B) = \frac{|A \cap B|}{|A \cup B|}$$

where $A$ and $B$ are the sets of n-grams of two documents.

**MinHash** approximates $J(A, B)$ using $k$ hash functions:
$$\hat{J}(A, B) = \frac{1}{k} \sum_{i=1}^k \mathbb{1}[\min_{a \in A} h_i(a) = \min_{b \in B} h_i(b)]$$

Each $\min_{a \in A} h_i(a)$ is the $i$-th **MinHash value** — the minimum hash of any element in $A$ under hash function $h_i$.

**The key property**: $P[\min h_i(A) = \min h_i(B)] = J(A, B)$ — the probability that two sets share the same minimum hash equals their Jaccard similarity.

**LSH (Locality-Sensitive Hashing)** groups documents into **buckets** so that similar documents land in the same bucket with high probability, without comparing all pairs:

1. Split the $k$ MinHash values into $b$ bands of $r$ values each ($k = b \times r$)
2. Two documents are candidate duplicates if they share all $r$ values in at least one band
3. Only candidate pairs are compared fully

Tuning $b$ and $r$ controls the precision/recall trade-off:
- More bands → higher recall (fewer missed duplicates), lower precision
- Fewer bands → higher precision, higher recall misses

### Suffix Array Deduplication

Used in large-scale pretraining pipelines (Lee et al. 2021):
1. Concatenate all documents with separator tokens
2. Build a **suffix array** (sorted array of all suffixes of the concatenated string)
3. Find adjacent suffixes that share a long common prefix → these are near-duplicate passages
4. Remove one copy of each duplicated passage

Suffix array construction: $O(n \log n)$; subsequent dedup scan: $O(n)$.
This is the method used to deduplicate C4, The Pile, and RedPajama.

---

## Math / Formal Notation

### MinHash Signature Matrix

Given $n$ documents and $k$ hash functions, the **signature matrix** $M \in \mathbb{R}^{k \times n}$:

$$M[i, j] = \min_{a \in \text{shingles}(D_j)} h_i(a)$$

Jaccard estimate between documents $p$ and $q$:
$$\hat{J}(D_p, D_q) = \frac{|\{i : M[i,p] = M[i,q]\}|}{k}$$

**Error bound**: By the law of large numbers, $\hat{J}$ converges to $J$ with standard deviation $\frac{1}{\sqrt{k}}$. For $k = 200$, error $\approx 0.07$.

### LSH Probability Curve

Probability that a pair with similarity $s$ becomes a candidate:

$$P(\text{candidate} \mid s) = 1 - (1 - s^r)^b$$

This is an S-curve with a threshold at $s^* \approx (1/b)^{1/r}$. At $b=20, r=5$:
- $s = 0.5$: $P \approx 0.47$ (50% chance of being a candidate)
- $s = 0.8$: $P \approx 0.9998$ (near-certain candidate)
- $s = 0.3$: $P \approx 0.0$ (unlikely candidate)

---

## Examples (Concrete)

### Jaccard Similarity Example

```
Doc A shingles (3-grams): {"the cat sat", "cat sat on", "sat on the", "on the mat"}
Doc B shingles (3-grams): {"the cat sat", "cat sat on", "sat on a",  "on a chair"}

|A ∩ B| = {"the cat sat", "cat sat on"} = 2
|A ∪ B| = 6
J(A, B) = 2/6 ≈ 0.33
```

Both documents share the opening phrase but diverge — J=0.33 is below typical dedup threshold (0.7–0.8), so these would not be deduplicated.

### MinHash in Practice (Python)

```python
from datasketch import MinHash, MinHashLSH

def get_minhash(text, num_perm=128):
    m = MinHash(num_perm=num_perm)
    for word in text.lower().split():
        m.update(word.encode('utf-8'))
    return m

lsh = MinHashLSH(threshold=0.8, num_perm=128)
for i, doc in enumerate(corpus):
    m = get_minhash(doc)
    if not lsh.query(m):          # no near-duplicates found
        lsh.insert(f"doc_{i}", m)
    # else: skip this near-duplicate
```

### Real-World Impact (from Lee et al. 2021)

Training GPT-style LM on C4 with vs without deduplication:
- Perplexity: nearly identical (dedup slightly better)
- Memorisation: **9× less exact memorisation** with deduplication
- Training speed: deduplicated corpus is ~30% smaller → 30% fewer training steps

---

## How It Connects to ML / NLP

| Deduplication Method | Use Case | Scale |
|---------------------|---------|-------|
| MD5/SHA hash | Exact dedup; fast | Billions of docs |
| MinHash + LSH | Near-dedup; web crawl | Hundreds of millions |
| Suffix arrays | Passage-level dedup | Terabyte corpora |
| BM25 overlap | Benchmark contamination check | Thousands of test examples |
| Embedding similarity | Semantic dedup (experimental) | Millions (expensive) |

**Benchmark contamination check**: before publishing evaluation results, check whether test set examples appear (exactly or near-exactly) in your training data. Use suffix array search or BM25 retrieval.

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — memorisation of duplicates is a form of overfitting
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — dedup is a core component of data quality
- [[3.ML & DL/1.Concepts/10.Unsupervised Learning/Clustering.md]] — near-dedup via clustering is an alternative approach

---

## Common Interview Questions

**Q: What is the difference between exact and near deduplication, and when do you need each?**
A: Exact deduplication removes character-for-character identical documents — fast with hashing, zero false positives. Near-deduplication removes documents that are highly similar but not identical (e.g. the same article with a changed headline or minor edits). Near-dedup is necessary for web corpora where the same article is reprinted with minor modifications across thousands of sites. Near-dedup requires approximate algorithms (MinHash + LSH or suffix arrays) and always involves a similarity threshold trade-off.

**Q: How does MinHash approximate Jaccard similarity? Why is it probabilistic?**
A: For each of $k$ hash functions, MinHash computes the minimum hash value over all n-grams in a document. The probability that two documents share their minimum hash under any one function equals their Jaccard similarity. By averaging over $k$ functions, we get an unbiased estimate of Jaccard with variance $1/k$. It is probabilistic because hash collisions can occur — two different n-grams could hash to the same minimum value — but with a good universal hash family, this probability is well-controlled.

**Q: Why does deduplication reduce memorisation in language models?**
A: If a passage appears $n$ times in training data, the model receives $n$ gradient updates pushing it to predict that passage — it is essentially weighted $n$ times higher than a passage appearing once. Memorisation occurs when this weight is high enough that the model can reproduce the passage verbatim at inference. Deduplication removes repeated occurrences, so every passage is seen (at most once), reducing the gradient pressure toward memorisation of any specific text.

---

## Common Mistakes / Gotchas

- **Deduplicating before normalisation**: "The Cat Sat" and "the cat sat" are functionally the same but hash differently without case normalisation. Always normalise (lowercase, whitespace collapse, Unicode normalise) before hashing.
- **Choosing too high a Jaccard threshold for near-dedup**: threshold 0.9+ misses many reprints and translations. Threshold 0.5 is too aggressive — removes genuinely different documents discussing the same topic. 0.7–0.8 is typical for web corpora.
- **Deduplicating validation/test sets**: only deduplicate training data. Never remove examples from validation or test sets — this invalidates your evaluation.
- **Ignoring inter-split contamination**: after building train/val/test splits, check that no test example is a near-duplicate of any training example (not just within splits). This is benchmark contamination.
- **Forgetting document-level vs paragraph-level**: a document may be unique overall but contain individual boilerplate paragraphs (cookie notices, "About us" sections) repeated across the corpus. Document-level dedup misses these; paragraph-level dedup catches them.

---

## Further Reading / Paper References

- Broder, A.Z. (1997). *On the Resemblance and Containment of Documents.* — MinHash original paper
- Leskovec, J., Rajaraman, A. & Ullman, J. *Mining of Massive Datasets.* Ch. 3 — LSH and MinHash (free online)
- Lee, K. et al. (2022). *Deduplicating Training Data Makes Language Models Better.* ACL. [[arxiv:2107.06499]] — must-read
- Penedo, G. et al. (2023). *The RefinedWeb Dataset for Falcon LLM.* [[arxiv:2306.01116]]
- `datasketch` library: https://ekzhu.github.io/datasketch/ — MinHash + LSH in Python
