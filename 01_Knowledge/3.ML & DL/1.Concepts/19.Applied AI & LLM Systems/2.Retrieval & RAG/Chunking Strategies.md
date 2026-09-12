# Chunking Strategies

## What is it?

**Chunking** splits a large document into smaller pieces before embedding and storing them for retrieval. This isn't a minor implementation detail — the chunking strategy directly determines what a retrieval system is even *capable* of finding, and a well-tuned RAG pipeline with poor chunking will underperform regardless of how good the embedding model or LLM is.

---

## Why Chunk Size Is a Real Tradeoff

**Chunks too large**: a chunk's embedding is a single vector representing everything in it — if a chunk covers several distinct topics, its embedding becomes a blurred average that matches *none* of those topics precisely, hurting retrieval precision. Large chunks also waste [[LLM Inference Fundamentals|context window]] budget on irrelevant surrounding content once retrieved.

**Chunks too small**: a chunk can lose necessary context — a sentence fragment retrieved in isolation, without the paragraph explaining what it refers to, may be uninterpretable or misleading to the LLM even if it was the technically "correct" match. Very small chunks also multiply the total number of vectors to store and search.

There's no universal correct chunk size — it depends on document structure and the granularity of questions the system needs to answer, which is why chunking strategy is usually tuned empirically against a representative set of real queries, not set once and assumed correct.

---

## Common Strategies

**Fixed-size chunking** — split every $N$ tokens/characters, optionally with overlap between consecutive chunks (so information near a chunk boundary isn't only ever visible on one side of a hard cut). Simple, fast, works acceptably as a baseline; ignores document structure entirely, so a chunk boundary can land in the middle of a sentence or a table.

**Recursive/structure-aware chunking** — split along natural document boundaries (paragraphs, then sentences if a paragraph is still too large, headers/sections for structured documents) — see [[Paragraph Segmentation]], [[Sentence Boundary Detection]] for the NLP mechanics underlying this. Respects the document's actual structure, generally producing more coherent chunks than a blind fixed-size cut.

**Semantic chunking** — use embedding similarity between adjacent sentences to decide chunk boundaries dynamically, splitting where the topic actually shifts rather than at a fixed size. More adaptive to content, more expensive to compute (requires embedding at a finer granularity first, just to decide where to chunk).

**Overlap** — a fixed amount of shared content between consecutive chunks (e.g. the last 50 tokens of chunk $n$ repeated as the first 50 of chunk $n+1$) reduces the risk of a critical piece of information being split exactly across a chunk boundary and therefore fully retrievable in neither chunk.

---

## Interview Questions

**Why does chunk size matter for retrieval quality, not just storage efficiency?** A chunk's embedding is one vector representing its entire content — a chunk spanning multiple unrelated topics produces a blurred embedding that matches none of them well, directly hurting retrieval precision, while an overly small chunk can lose the surrounding context needed to interpret it correctly even when retrieved.

**When would you use semantic chunking over simple fixed-size chunking?** When document content varies significantly in topic density and a fixed size would frequently cut across genuine topic boundaries — semantic chunking adapts the split points to where the content itself actually shifts, at the cost of extra compute to determine those boundaries.

**Why include overlap between chunks?** To reduce the risk that a piece of information needed to answer a question sits exactly at a chunk boundary and ends up split across two chunks, fully present in neither — overlap ensures boundary-adjacent content appears whole in at least one chunk.

## Connections

- [[RAG Architecture]] — chunking is the first, foundational decision in any RAG pipeline
- [[Vector Search and Databases]] — what actually gets embedded and stored is determined by chunking
- [[Paragraph Segmentation]], [[Sentence Boundary Detection]] — the NLP mechanics structure-aware chunking relies on
- [[LLM Inference Fundamentals]] — retrieved chunks compete for context-window budget

## One-line Summary

> Chunking determines what a retrieval system can possibly find — too large and embeddings blur across topics, too small and context is lost — with structure-aware or semantic chunking generally outperforming naive fixed-size splits, tuned empirically against real queries rather than assumed correct.
