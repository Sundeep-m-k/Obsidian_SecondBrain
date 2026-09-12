# Design a Production RAG System for 10 Million Documents

## What This Note Is

A **system-design anchor note** — the vault's other notes each cover one concept well; this one composes a dozen of them into the kind of 45-minute answer an actual Applied AI Engineer / AI-ML Engineer / Forward Deployed Engineer system-design interview demands. Every design decision below links to the atomic note that justifies it in depth — this note's value is in the *composition and tradeoff reasoning*, not re-deriving mechanisms already covered elsewhere.

---

## Step 0: Clarify Requirements Before Designing Anything

A strong answer starts here, out loud, before touching architecture — jumping straight to "here's my design" without this step is the single most common way this question goes badly (see [[Framing Ambiguous Business Problems]]).

**Assumed requirements** (stated explicitly, since a real interview would have you ask):
- 10 million documents, **continuously updated** (not a static corpus)
- Hundreds to thousands of **concurrent users**, **interactive** latency expectations (seconds, not minutes)
- **Citations required** — every answer must point to its source
- **Cost matters** — this isn't a blank-check research system
- **Enterprise data with access controls** — not every user can see every document

**Functional requirements**: answer natural-language questions grounded in the document corpus; cite sources; respect per-user/tenant access permissions; handle documents being added, updated, and deleted.

**Non-functional requirements**: p95 end-to-end latency target (e.g. <3s for interactive use); cost per query budget; freshness (how stale can an answer be after a document changes); availability/reliability under load.

---

## Ingestion

**Parsing**: documents arrive in heterogeneous formats — PDFs (including scanned/image PDFs needing OCR, and PDFs with tables that plain text extraction mangles), HTML, Word docs, Markdown. A production pipeline needs format-specific parsers, not one universal text extractor, and needs to **explicitly detect and route tables/structured content differently** from prose (a table flattened into a run of text loses its row/column relationships — the numbers stay, the meaning connecting them doesn't; downstream retrieval on a table needs either a structure-preserving representation or a separate table-QA path).

**Cleaning**: strip boilerplate (headers, footers, navigation chrome from HTML), fix encoding issues, normalize whitespace — see [[Preprocessing Pipelines]] for the general discipline this applies.

**Deduplication**: at 10M documents, exact and near-duplicate documents are a near-certainty (the same policy PDF uploaded under three different names, a document and its near-identical revision both still in the corpus). Undetected duplicates directly hurt retrieval — multiple near-identical chunks compete for the same top-k slots, crowding out genuinely different relevant content. Content-hash catches exact duplicates cheaply; a similarity threshold on embeddings catches near-duplicates, at the cost of an extra embedding-and-compare pass during ingestion.

**Metadata, document IDs, versioning**: every ingested document gets a stable ID, a version number, source/access-control metadata (which tenant, which permission groups can see it), and ingestion/update timestamps — this metadata is what later makes access-control filtering, freshness tracking, and safe updates possible at all. Treat metadata schema design as a first-class ingestion decision, not an afterthought bolted on later.

---

## Chunking

Compare, per [[Chunking Strategies]]:

| Strategy | Fit for this system |
|---|---|
| Fixed-token | Simple baseline; risks cutting mid-table or mid-sentence at 10M-document scale where content is highly heterogeneous |
| Recursive/structure-aware | Better default here — respects document structure (headers, paragraphs) which enterprise documents (policies, manuals) reliably have |
| Semantic | Best retrieval quality, but the extra embedding-at-chunking-time cost matters at 10M documents — reserve for a later optimization pass once basic infrastructure is validated, not the v1 build |
| Table-aware (custom) | Necessary alongside whichever of the above is chosen — tables need row/column-preserving serialization (e.g. one chunk per logical table, rendered as markdown-table text) rather than being flattened into prose-style chunks |

**Chunk size and overlap tradeoff, decided for this scale**: moderate chunk size (roughly paragraph-to-section length) with ~10-15% overlap, tuned empirically against a held-out set of real user queries once the system exists (see [[Chunking Strategies]] — there's no universally correct number, only one validated against actual retrieval behavior for *this* corpus and *these* queries).

---

## Embeddings

**Model selection**: choose an embedding model balancing retrieval quality, dimensionality (higher dimension = better quality but more storage and slower similarity search), and inference cost at ingestion scale (embedding 10M+ chunks is itself a real compute cost, not a one-time afterthought).

**Batch embedding**: ingestion embeds in large batches, not one document at a time — the same batching-for-throughput logic as [[Inference vs Training]], applied to an offline bulk job rather than live serving.

**What happens when the embedding model changes** (a frequent, high-value follow-up question — answered explicitly since it's easy to get wrong): **you cannot mix vectors from two different embedding models in one index** — they live in different, incompatible vector spaces, so a similarity score between an old-model vector and a new-model query embedding is meaningless, not just "a bit worse." The correct approach is **dual indexing**: stand up a second, parallel vector index built with the new model, re-embed the full corpus into it (a real, non-trivial batch job at 10M documents), validate retrieval quality on the new index against a held-out evaluation set, then cut traffic over — either atomically or via a gradual rollout — and only then decommission the old index. Never partially re-embed into the *same* index while the old model's vectors are still present; that silently produces a broken, mixed-space index that degrades retrieval in a way that's hard to diagnose after the fact.

---

## Storage

Four distinct stores, deliberately separated rather than one system doing everything:

- **Source-of-truth document storage** (object storage / a document database) — the original documents, full text, immutable history for audit — the thing everything else is derived *from* and can be rebuilt *from* if a downstream index is corrupted or needs rebuilding.
- **Metadata store** (a relational or document database) — document IDs, versions, access-control lists, timestamps — queried for permission filtering *before or during* retrieval, not as an afterthought applied to results.
- **Vector index** ([[Vector Search and Databases]]) — the embeddings, built for approximate nearest-neighbor search at scale (HNSW/IVF).
- **Lexical/BM25 index** (optional but recommended here) — a separate inverted-index structure for exact term matching, feeding [[Dense vs Sparse Retrieval|hybrid retrieval]].

Keeping these separate (rather than, say, storing everything only in the vector database) means each can scale and be rebuilt independently — re-embedding for a model migration touches only the vector index, not the source-of-truth store.

---

## Retrieval

```
User query
  → Query processing / rewriting (optional — see Query Rewriting)
  → BM25 search               ┐
  → Dense (vector) search     ┤→ Hybrid fusion (Reciprocal Rank Fusion)
  → Metadata / security (ACL) filtering  ← applied here, not after generation
  → Reranker (cross-encoder)
  → Final context (top-k chunks)
```

**Why hybrid retrieval likely outperforms dense-only here specifically**: at 10M enterprise documents, exact terms matter enormously — product codes, policy section numbers, specific named entities, internal jargon — exactly the case [[Dense vs Sparse Retrieval]] identifies as sparse retrieval's strength and dense retrieval's weakness. A pure-embedding system will reliably retrieve semantically-similar-but-wrong documents for a query containing a specific code or ID that the embedding model was never trained to treat as distinctive. Hybrid retrieval costs more (running two retrieval systems and fusing results) but is close to necessary at this scale and document heterogeneity.

**Where access-control filtering happens, and why it matters where**: filtering **must** happen as part of — or immediately after — retrieval, before reranking and generation, never as a post-hoc check on the final answer. Retrieving a chunk the user isn't authorized to see, even if it's later filtered out before being shown, still means that chunk's content influenced which reranked/generated result the user ultimately received (and, if filtering happens even later, could leak directly). The safest implementation pushes ACL filtering into the vector/BM25 query itself (metadata-filtered search) rather than retrieving broadly and filtering after the fact.

---

## Reranking

**Bi-encoder vs. cross-encoder** (see [[Reranking and Hybrid Search]]): the retrieval stage above uses bi-encoders (query and document embedded independently, compared by similarity) because that's what makes searching millions of documents fast. A **cross-encoder** processes the query and each candidate jointly, capturing finer-grained interaction the bi-encoder's separate embeddings miss — more accurate, far too slow to run against the full corpus, which is exactly why it's reserved for reranking the already-narrowed candidate set (typically the top 50-100 from hybrid retrieval down to the top 5-10 actually sent to generation).

**When reranking is worth the latency here**: given interactive latency requirements and the corpus's heterogeneity (many documents plausibly relevant to a broad query), reranking is worth it — the accuracy gain on a 50-100-candidate set costs a bounded, predictable amount of added latency, unlike attempting to rerank the full retrieved pool from a 10M-document corpus directly (see "Why not rerank 10,000 documents?" below).

---

## Prompt Construction

**Context selection**: the reranked top-k chunks, ordered — position within the prompt affects how much attention the model pays to each chunk (a genuine, empirically-observed effect, not just a theoretical concern), so don't assume the model treats first and last context identically.

**Token budget**: the [[LLM Inference Fundamentals|context window]] is finite and shared between system instructions, retrieved context, conversation history, and the user's query — a fixed token budget must be allocated across these deliberately (e.g. reserve a fixed portion for instructions/query, use the remainder for as many top-ranked chunks as fit) rather than greedily stuffing in every retrieved chunk.

**Citations**: instruct the model to cite which specific retrieved chunk supports each claim (by chunk ID or source reference) — this is both a user-trust feature and a [[RAG Evaluation|faithfulness evaluation]] mechanism, since a claim with no traceable citation is a direct signal of potential [[Hallucination Mitigation|hallucination]].

**Instructions**: explicit system instructions defining scope ("answer only from the provided context," "say you don't know if the context doesn't contain the answer") — directly reduces the failure mode of the model answering from parametric memory instead of the retrieved content.

---

## Generation

**Model choice**: balance quality against latency and cost at the expected query volume (hundreds-to-thousands of concurrent users) — a smaller/faster model may be entirely sufficient once retrieval quality is strong, since a well-grounded prompt reduces how much the generation step needs to "know" on its own; reserve the largest available model for cases that specifically need it.

**Structured responses**: if the product needs machine-parseable output (e.g. a structured citation format, a JSON answer envelope) use [[LLM Inference Fundamentals|constrained decoding/structured outputs]] rather than hoping free-form prose parses reliably.

---

## Evaluation

Kept explicitly separate per pipeline stage — a single end-to-end pass/fail metric can't localize which stage actually failed (see [[RAG Evaluation]]):

**Retrieval evaluation**: Recall@K (of the chunks that should have been retrieved, what fraction were, within the top K), MRR (Mean Reciprocal Rank — how high up the *first* relevant result ranks, useful when there's typically one clearly-best answer chunk), nDCG (when relevance is graded rather than binary — some retrieved chunks are more relevant than others, not just relevant/irrelevant).

**Generation evaluation**: groundedness/faithfulness (does the answer's content follow from the retrieved context), correctness (is the answer actually right, checked against a reference where available), citation quality (are the cited sources the ones that actually support the claim, not just present).

**End-to-end/product metrics**: user satisfaction (explicit feedback, thumbs up/down), task completion rate, latency (p50/p95/p99, not just average), cost per query.

---

## Observability

Every request should log: the original query, any rewritten query, the full retrieved candidate set (before and after reranking, before and after ACL filtering — separately, so a "why didn't this retrieve" investigation can tell which stage dropped a candidate), the final context sent to the model, the generated response, per-stage latency, token counts, and which model/index versions were in use (critical for diagnosing an issue that only appears after an embedding-model or reranker upgrade). See [[Observability and Evaluation for LLM Systems]] for why LLM-specific observability differs from traditional uptime/latency monitoring.

---

## Failure Modes

| Failure | Cause | Mitigation |
|---|---|---|
| No relevant retrieval | Query/document vocabulary mismatch, poor chunking, or genuinely absent content | [[Query Rewriting]], hybrid retrieval, explicit "I don't know" instruction when nothing relevant is found |
| Hallucination | Model answers from parametric memory instead of context, or over-extrapolates from a partial match | Grounding instructions, citation requirements, [[Hallucination Mitigation]]'s layered defenses |
| Stale documents | A document changed but the index wasn't updated, or update propagation lagged | Explicit freshness SLAs, incremental re-indexing pipeline (see Updates below) |
| Duplicate chunks | Ingestion didn't dedupe, or near-duplicate documents both got indexed | Ingestion-time dedup (content hash + similarity threshold) |
| Wrong ACL filtering | Filtering applied too late, or metadata incorrectly tagged at ingestion | Filter at the retrieval query itself, audit metadata tagging, fail closed (deny by default) on any ACL ambiguity |
| Bad query rewriting | Rewrite drifts from original intent, especially in HyDE's confidently-wrong-hypothetical failure mode | Retrieve on both original and rewritten queries, merge (see [[Query Rewriting]]) |
| Reranker failure | Cross-encoder trained on different-domain data ranks poorly on this corpus's actual content | Evaluate reranker quality on this corpus specifically before deploying; fall back to bi-encoder ranking if reranker confidence is low |
| Embedding drift/version mismatch | Partial re-embedding mixed two incompatible vector spaces in one index | Dual indexing during any embedding-model migration, never partial in-place re-embedding (see Embeddings above) |

---

## Scaling

**Sharding/index partitioning**: at 10M documents, a single vector index instance may not fit in memory or serve query volume fast enough — partition by tenant (natural boundary given access control already segments by tenant) or by document collection, querying only relevant shards for a given request.

**Caching**: prompt caching for repeated system instructions/context; semantic caching for genuinely repeated question patterns across users (with the risks and tenant-isolation requirements covered in [[Caching Strategies for LLM Systems]]).

**Batching**: batch embedding at ingestion time (already covered above); batching live inference requests where interactive latency budgets allow some queueing.

**Asynchronous ingestion**: document ingestion (parsing, chunking, embedding, indexing) runs as an async pipeline, decoupled from the query-serving path entirely — a slow or backed-up ingestion pipeline should never add latency to a live user query.

**Retrieval latency vs. model latency**: at scale, these are separate budgets to track and optimize independently — a fast retrieval stage feeding a slow generation model (or vice versa) needs different fixes, so p95 latency should be broken down by stage, not reported only end-to-end.

---

## Cost

**Major cost centers**: (1) generation — tokens processed per query, scaling with both context size and output length; (2) embedding — both the initial 10M-document bulk embed and ongoing embedding of new/updated documents; (3) vector search infrastructure — index hosting/serving cost, which scales with corpus size and query volume; (4) reranking — an additional model call per query.

**"How would you reduce cost by 50%?"** In priority order: (1) reduce context size sent to generation — tighter chunking, more selective top-k, since generation cost scales directly with tokens processed; (2) route simpler queries to a smaller/cheaper model rather than the largest model by default; (3) implement semantic caching for genuinely repeated query patterns; (4) use prompt caching for the stable parts of every prompt (system instructions, any static context); (5) batch embedding jobs efficiently rather than embedding synchronously per-document; (6) reconsider whether reranking every query is necessary, or only queries with ambiguous/low-confidence retrieval.

---

## Latency

**Approximate breakdown by stage** (illustrative, not universal — actual numbers depend entirely on infrastructure): query rewriting (if used) ~100-300ms, hybrid retrieval ~50-200ms, reranking ~100-300ms, generation ~1-3s (the dominant cost, since it's proportional to output length and model size). Generation is almost always the largest single latency contributor.

**"How would you reduce latency by 50%?"** In priority order: (1) streaming — doesn't reduce total generation time but dramatically improves perceived latency (see [[Latency and Cost Optimization]]); (2) reduce output length where the use case allows (shorter, more direct answers); (3) use a smaller/faster model for generation, especially once retrieval quality is high enough that generation doesn't need to "do as much work"; (4) skip reranking for queries where initial retrieval confidence is already high, rather than reranking unconditionally; (5) parallelize BM25 and dense retrieval rather than running them sequentially; (6) prompt caching to skip recomputing stable prefix content.

---

## Security

**Authorization before retrieval, not after**: covered above under Retrieval — this is worth restating as its own security concern, since retrieving unauthorized content and merely hiding it from the final answer is still a partial data exposure (the content influenced ranking/generation) and a potential leak vector.

**Tenant isolation**: applies to every store — the vector index, the metadata store, and any semantic cache — a query from tenant A must never retrieve, rank against, or serve a cached response derived from tenant B's data.

**Prompt injection from retrieved documents**: a document in the corpus itself could contain text crafted to be interpreted as an instruction rather than content (see [[Prompt Injection and Production Reliability]]) — a real risk specifically because *retrieved* content, unlike the system prompt, is untrusted by construction (anyone with ingestion permissions, or a compromised source document, could plant it).

**Data exfiltration**: guard against a crafted query designed to make the model reveal content from documents the user shouldn't have access to, or to reveal system-prompt instructions verbatim — output-side validation, not just input-side ACL filtering, matters here.

**Logging sensitive context**: observability logs (see above) capture full retrieved context and generated responses — these logs themselves need the same access controls and retention policies as the underlying documents, or observability becomes its own data-leak surface.

---

## Updates

Documents update continuously (a stated requirement) — updates and deletes must propagate safely:

- **Update**: re-parse, re-chunk, re-embed only the changed document (not the whole corpus), then atomically swap the new chunks/vectors in and remove the old ones for that document ID — never leave both old and new chunks for the same document live simultaneously, or retrieval can surface stale and current content for the same query.
- **Delete**: remove from the vector index, BM25 index, and metadata store together — a delete that only removes from one store leaves retrievable-but-orphaned content behind.
- **Propagation lag**: define an explicit freshness SLA (e.g. "updates reflected in retrieval within N minutes") rather than leaving it implicit — this is a real product decision, not just an engineering detail, since some use cases need near-real-time freshness and others don't.

---

## Embedding Migration

Already answered in full under **Embeddings** above — restated here since it's exactly the kind of explicit interviewer follow-up this note is built to survive: **dual indexing**, never partial in-place re-embedding. Build the new index fully, validate it against a held-out evaluation set, cut over deliberately, decommission the old index only after cutover is confirmed successful.

---

## Tradeoff Questions (Interviewer Follow-Ups)

1. **Why not put all 10M documents directly into an LLM context?** Context windows are bounded by attention's quadratic cost (see [[Transformer End-to-End Walkthrough]]) and cost/latency scale with tokens processed — 10M documents is orders of magnitude beyond any context window, and even if it fit, cost per query would be prohibitive; retrieval exists precisely to select only the relevant slice per query.
2. **Why use BM25 if we already have embeddings?** Dense retrieval misses precise exact-term matches (product codes, specific IDs, jargon) that BM25 catches directly — see [[Dense vs Sparse Retrieval]]; the two fail in complementary, not overlapping, ways.
3. **Why not rerank 10,000 documents?** Cross-encoder reranking processes the query jointly with *each* candidate — computationally infeasible at that scale with interactive latency requirements; it's reserved for a small, already-narrowed candidate set (tens to low hundreds) for exactly this reason.
4. **Where would Redis (or similar) help?** As the backing store for prompt/semantic caching, and potentially for a hot metadata cache (frequently-accessed ACL/document-metadata lookups) to avoid a database round-trip on every retrieval request.
5. **Would you cache retrieval results?** Yes, cautiously — cache the retrieved-candidate-set for identical or near-identical queries, but invalidate aggressively on any document update affecting the cached results, and never cache across tenant boundaries.
6. **How do you handle PDFs with tables?** Dedicated table-extraction at ingestion (structure-preserving serialization, e.g. markdown tables), not generic text extraction that flattens rows/columns into unstructured prose — see Ingestion above.
7. **How do you evaluate chunk size?** Empirically, against a held-out set of real user queries with known-correct answer chunks, measuring Recall@K and downstream faithfulness across candidate chunk sizes — not by intuition or a single universal default (see [[Chunking Strategies]]).
8. **How do you prevent one customer from retrieving another customer's data?** ACL/tenant metadata attached at ingestion, enforced as a filter on the retrieval query itself (not a post-hoc check), applied consistently across the vector index, BM25 index, and any semantic cache — with fail-closed behavior on any ambiguity.
9. **What happens if the reranker itself is slow under load?** Either scale reranker serving horizontally, reduce how many candidates get reranked (narrower initial retrieval), or make reranking conditional on retrieval confidence rather than unconditional.
10. **How would you detect that retrieval quality has degraded in production?** Track Recall@K/MRR against a periodically-refreshed evaluation set, monitor user feedback signals (thumbs down rate), and watch for drift in query patterns that no longer match the corpus (see [[Data Drift and Concept Drift]]).
11. **What's your rollback plan if a new embedding model performs worse in production than expected?** Dual indexing means the old index still exists — route traffic back to it, which is exactly why the migration strategy above never deletes the old index until the new one is validated.
12. **How do you handle a query in a language the corpus is mostly not written in?** Depends on the embedding model's multilingual capability and whether cross-lingual retrieval is a stated requirement — if not explicitly required, flag this as a scoping question rather than assuming it's handled.
13. **What if two documents directly contradict each other (an old policy and its replacement, both still indexed)?** This is exactly what document versioning and update propagation (see Updates) are meant to prevent — the old version should be superseded/removed, not left live alongside its replacement; if both are legitimately still relevant (e.g. historical context), metadata should distinguish them explicitly so retrieval/generation can reason about recency.
14. **How would you support conversational, multi-turn RAG?** Conversational [[Query Rewriting]] to make follow-up queries self-contained before retrieval, plus conversation history management within the token budget (see Prompt Construction).
15. **Why separate the source-of-truth document store from the vector index instead of storing everything in the vector database?** Each store scales and gets rebuilt independently — an embedding-model migration only touches the vector index; a corrupted or need-to-rebuild vector index can be regenerated from the source-of-truth store without any data loss, which wouldn't be possible if the vector database were the only copy of the content.

---

## Connections

- [[Chunking Strategies]], [[Query Rewriting]], [[Dense vs Sparse Retrieval]], [[Vector Search and Databases]], [[Reranking and Hybrid Search]], [[RAG Architecture]], [[RAG Evaluation]] — every retrieval-stage decision above draws directly on these
- [[Caching Strategies for LLM Systems]], [[Latency and Cost Optimization]] — the cost/latency sections above
- [[Prompt Injection and Production Reliability]], [[Observability and Evaluation for LLM Systems]] — the security and observability sections above
- [[Hallucination Mitigation]] — the grounding/citation reasoning in Prompt Construction and Failure Modes
- [[Data Drift and Concept Drift]], [[Model Monitoring in Production]] — production monitoring concerns that apply here identically to classical ML systems
- [[Framing Ambiguous Business Problems]] — the requirement-clarification discipline this note opens with

## One-line Summary

> A production RAG system at this scale is a composition of a dozen individually-simple decisions — hybrid retrieval over dense-only, dual indexing over in-place re-embedding, ACL filtering at retrieval not after, per-stage evaluation not one end-to-end metric — and the interview bar isn't knowing each decision exists, it's being able to justify why each one is the right tradeoff for *these* specific requirements.
