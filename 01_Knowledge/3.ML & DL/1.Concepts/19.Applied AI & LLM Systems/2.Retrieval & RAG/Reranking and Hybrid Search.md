# Reranking and Hybrid Search

## What is it?

**Hybrid search** runs [[Dense vs Sparse Retrieval|sparse and dense retrieval]] in parallel and combines their candidate results, since the two fail in complementary ways. **Reranking** then re-scores that combined candidate set with a more accurate (but more expensive) model, to put the truly best results at the top before they're passed to the LLM.

---

## Why a Two-Stage Pipeline: Cheap Retrieval, Expensive Reranking

Retrieval (BM25 or vector search) needs to be fast enough to search across potentially millions of documents — this rules out running a full, expensive comparison model against every candidate. **Reranking** exploits the fact that once retrieval has narrowed the field to a small candidate set (say, the top 50–100 results), it becomes computationally affordable to run a much more accurate, more expensive scoring model against just that small set. This "retrieve broadly and cheaply, then rerank narrowly and precisely" pattern is the standard structure of virtually every serious retrieval pipeline, not unique to RAG.

## How Combining Sparse + Dense Results Works

**Reciprocal Rank Fusion (RRF)** is a common, simple way to merge two ranked lists without needing to calibrate their scores against each other (BM25 scores and cosine similarity scores aren't on comparable scales, so directly averaging them is meaningless):

$$\text{RRF}(d) = \sum_{\text{retriever} \in \{sparse, dense\}} \frac{1}{k + \text{rank}_{\text{retriever}}(d)}$$

A document ranked highly by *either* retriever gets a high combined score, without needing the retrievers' raw scores to be on the same scale — $k$ (typically 60) dampens the impact of rank differences deep in either list.

## Cross-Encoder Reranking

A **cross-encoder** reranker processes the query and each candidate document *together*, in a single forward pass through a model (typically a fine-tuned [[BERT Embeddings|BERT]]-style model), letting it model fine-grained interactions between query and document terms that a dense retriever's separate query/document embeddings cannot capture. This is more accurate than the embedding similarity used for initial retrieval, but far too slow to run against the entire corpus — which is exactly why it's reserved for reranking a small candidate set, not used for retrieval itself.

```
Retrieval (fast, whole corpus):  BM25 + dense vector search  →  top ~100 candidates
Reranking (slow, small set):     cross-encoder scores each of the ~100  →  top ~5 to the LLM
```

---

## Interview Questions

**Why not just use a cross-encoder for retrieval directly, since it's more accurate?** A cross-encoder must process the query jointly with every candidate document, which is far too slow to run against an entire corpus of millions of documents — it's only computationally feasible against the small candidate set retrieval has already narrowed down to.

**How does Reciprocal Rank Fusion combine sparse and dense retrieval results?** It sums each document's reciprocal rank across both retrievers rather than trying to combine their raw scores directly, sidestepping the problem that BM25 scores and cosine similarities aren't on comparable scales — a document ranked highly by either method scores well overall.

**What's the general pattern behind "retrieve broadly, then rerank narrowly," and where else does it show up?** Use a fast, cheap method to narrow a huge candidate space down to a small, manageable set, then apply a slower, more accurate method only to that small set — the same pattern underlies recommendation systems (broad candidate generation, then precise ranking) and many other large-scale search problems, not just RAG.

## Connections

- [[Dense vs Sparse Retrieval]] — the two retrieval methods hybrid search combines
- [[Vector Search and Databases]] — supplies the dense-retrieval candidates being combined
- [[RAG Architecture]] — where this two-stage retrieve-then-rerank pipeline sits in the full pipeline
- [[BERT Embeddings]] — the architecture cross-encoder rerankers are typically built on

## One-line Summary

> Hybrid search combines sparse and dense retrieval's complementary strengths via rank fusion (not raw score averaging), and reranking then applies a more accurate but expensive cross-encoder to just that narrowed candidate set — a "cheap broad retrieval, expensive narrow reranking" pattern common well beyond RAG.
