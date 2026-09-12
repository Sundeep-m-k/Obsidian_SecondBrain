# Vector Search and Databases

## What is it?

**Vector search** finds the $k$ nearest vectors to a query vector out of a large collection — the computational engine underneath [[Dense vs Sparse Retrieval|dense retrieval]]. A **vector database** is infrastructure purpose-built to store embeddings and answer these nearest-neighbor queries efficiently at scale (millions to billions of vectors), which a naive linear scan cannot do fast enough for production use.

---

## Why Naive Nearest-Neighbor Search Doesn't Scale

Finding the exact $k$ nearest vectors to a query by brute-force comparison against every stored vector is $O(n)$ per query — this is exactly [[K-Nearest Neighbors|KNN's]] "no training phase" tradeoff, applied at a scale (millions/billions of documents) where an $O(n)$ scan per query is far too slow for real-time applications.

## Approximate Nearest Neighbor (ANN) Search

Vector databases solve this by returning *approximately* the nearest neighbors, trading a small, usually-acceptable amount of retrieval accuracy for orders-of-magnitude faster lookups. Two dominant approaches:

**HNSW (Hierarchical Navigable Small World)** — builds a multi-layer graph where each vector is a node connected to its approximate neighbors; search starts at a sparse top layer and descends through denser layers, narrowing in on the true nearest neighbors without ever comparing against most of the dataset. The most common ANN algorithm in modern vector databases (Pinecone, Weaviate, Qdrant, pgvector).

**IVF (Inverted File Index)** — clusters vectors (conceptually similar to [[K Means]]) into partitions ahead of time; a query is only compared against vectors in the partitions nearest to it, rather than the whole dataset — trading some accuracy (a true nearest neighbor sitting just across a partition boundary can be missed) for large speed gains.

Both trade some recall (finding the *true* nearest neighbors) for latency — a real, explicit engineering tradeoff, usually tunable via a parameter controlling how much of the index gets searched.

---

## What a Vector Database Actually Provides

Beyond raw ANN search, a production vector database typically provides: metadata filtering (retrieve only vectors matching a filter, e.g. "documents from this user" alongside the similarity search), CRUD operations on embeddings as documents change, and horizontal scaling across a large corpus. Options range from dedicated vector databases (Pinecone, Weaviate, Qdrant, Milvus) to vector-search extensions of general-purpose databases (`pgvector` for Postgres) — the latter is often the pragmatic choice when the application already uses that database and doesn't need extreme scale, avoiding an extra piece of infrastructure.

---

## Interview Questions

**Why can't a production semantic search system just brute-force compare a query against every stored embedding?** Brute-force comparison is $O(n)$ per query, which doesn't scale to the millions-to-billions of documents a real system handles with acceptable latency — approximate nearest-neighbor structures like HNSW trade a small amount of retrieval accuracy for orders-of-magnitude faster search.

**What's the core tradeoff HNSW and IVF both make?** Both accept some risk of missing the *true* nearest neighbor (approximate rather than exact search) in exchange for search that's far faster than a full scan — a true nearest neighbor near a partition/graph boundary can occasionally be missed, but in practice recall stays high enough for most applications while latency improves dramatically.

**When would you use pgvector instead of a dedicated vector database?** When the application already uses Postgres, the scale doesn't demand a specialized system's extreme performance, and avoiding an additional piece of infrastructure to operate and keep in sync outweighs the specialized performance a dedicated vector database would offer.

## Connections

- [[Dense vs Sparse Retrieval]] — vector search is the infrastructure dense retrieval runs on
- [[K-Nearest Neighbors]] — the exact-search algorithm ANN methods approximate at scale
- [[K Means]] — IVF's clustering-based partitioning is conceptually similar
- [[RAG Architecture]] — where vector search sits in the overall retrieval pipeline
- [[Chunking Strategies]] — determines what actually gets embedded and stored

## One-line Summary

> Vector databases make nearest-neighbor search over embeddings tractable at scale via approximate methods (HNSW, IVF) that trade a small amount of recall for large speed gains over brute-force comparison — the infrastructure layer beneath every dense-retrieval or semantic-search application.
