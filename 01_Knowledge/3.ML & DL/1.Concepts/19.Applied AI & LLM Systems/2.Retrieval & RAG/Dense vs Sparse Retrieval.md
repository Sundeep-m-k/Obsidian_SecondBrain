# Dense vs Sparse Retrieval

## What is it?

**Retrieval** finds the documents/passages most relevant to a query from a large collection — the first stage of any [[RAG Architecture|RAG]] pipeline, and the search half of any semantic search product. There are two fundamentally different ways to represent documents and queries for this: **sparse** (based on exact term matching) and **dense** (based on learned embeddings).

---

## Sparse Retrieval: BM25

**BM25** is the standard modern sparse retrieval algorithm, an evolution of [[TF-IDF]] that scores a document's relevance to a query based on term overlap, with two refinements TF-IDF lacks:

$$\text{BM25}(D,Q) = \sum_{t\in Q} \text{IDF}(t)\cdot\frac{f(t,D)\cdot(k_1+1)}{f(t,D)+k_1\left(1-b+b\frac{|D|}{\text{avgdl}}\right)}$$

- **Term frequency saturation**: a term appearing 10 times isn't 10x as relevant as it appearing once — the $f(t,D)$ term is designed to saturate rather than scale linearly, unlike raw TF-IDF.
- **Document length normalization**: a long document naturally contains more term occurrences by chance; the $|D|/\text{avgdl}$ ratio (controlled by $b$) corrects for this so long documents aren't unfairly favored.

BM25 excels at **exact keyword/entity matching** — a query for a specific product code, a person's name, or a rare technical term is found reliably, since sparse retrieval directly rewards exact term overlap. It fails on **semantic** queries where the right document uses different words for the same concept (a query for "canine" won't match a document that only says "dog").

## Dense Retrieval: Embeddings

**Dense retrieval** encodes both query and documents into fixed-size vectors (via a model like a [[Sentence Embeddings|sentence embedding]] model) and finds documents whose embeddings are closest to the query's embedding by cosine similarity or dot product — matching on *meaning* rather than exact wording.

$$\text{score}(q,d) = \frac{e_q \cdot e_d}{\|e_q\|\|e_d\|}$$

This correctly retrieves "canine" for a "dog" query, since a well-trained embedding model places semantically similar concepts near each other in vector space, regardless of exact word choice. It's weaker on precise lexical matching — a rare product code or exact numeric identifier might not be encoded distinctly enough in embedding space to be reliably retrieved, since the model was trained on semantic similarity, not exact-string recall.

---

## Why Hybrid Retrieval Is the Practical Default

Sparse and dense retrieval fail in *complementary* ways — sparse misses semantic matches, dense misses exact lexical matches. Production RAG systems typically run both retrieval methods and combine their results, rather than picking one — see [[Reranking and Hybrid Search]] for how those combined candidates get merged and re-scored.

| | Strength | Weakness |
|---|---|---|
| BM25 (sparse) | Exact term/entity matching, no training needed | Misses semantic/paraphrase matches |
| Dense (embeddings) | Semantic/paraphrase matching | Misses precise exact-string matching (codes, rare terms) |

---

## Interview Questions

**Why does BM25 outperform dense retrieval on a query for a specific product code or ID?** Dense retrieval matches on learned semantic similarity, which doesn't reliably distinguish precise exact strings the embedding model wasn't specifically trained to treat as distinct; BM25 directly rewards exact term overlap, so an exact code match scores highly regardless of surrounding semantic context.

**What two refinements does BM25 add over raw TF-IDF?** Term frequency saturation (repeated occurrences of a term contribute diminishing marginal relevance, rather than scaling linearly) and document length normalization (correcting for longer documents naturally containing more term matches by chance).

**Why do production RAG systems typically combine sparse and dense retrieval rather than choosing one?** They fail in complementary ways — sparse retrieval misses paraphrases/synonyms, dense retrieval misses precise exact-string matches — so combining both catches queries either method alone would miss.

## Connections

- [[TF-IDF]], [[Bag of Words]] — BM25's sparse-retrieval ancestors
- [[Sentence Embeddings]], [[BERT Embeddings]] — the embedding models dense retrieval is built on
- [[Reranking and Hybrid Search]] — how sparse and dense results get combined in practice
- [[RAG Architecture]] — where retrieval fits in the larger pipeline
- [[Vector Search and Databases]] — how dense retrieval is made efficient at scale

## One-line Summary

> Sparse (BM25) retrieval matches exact terms and excels at precise entity/code lookup; dense (embedding) retrieval matches meaning and excels at paraphrase/semantic queries — production systems combine both because their failure modes are complementary, not overlapping.
