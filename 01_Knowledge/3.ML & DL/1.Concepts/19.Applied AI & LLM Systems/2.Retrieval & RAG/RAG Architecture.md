# RAG Architecture

## What is it?

**Retrieval-Augmented Generation (RAG)** grounds an LLM's response in retrieved, up-to-date, or private documents rather than relying solely on what it memorized during pretraining — retrieve relevant content for the query, then include it in the prompt so the model generates its answer *using* that content.

---

## The Standard Pipeline

```
1. User query arrives
2. Chunking (offline, done ahead of time) — the document corpus was already
   split into chunks and embedded ([[Chunking Strategies]])
3. Query processing / rewriting (optional but often necessary for short,
   vague, or conversational queries — [[Query Rewriting]])
4. Retrieval — find candidate chunks relevant to the (rewritten) query
   ([[Dense vs Sparse Retrieval]], [[Vector Search and Databases]])
5. Reranking (optional but common) — re-score candidates for precision
   ([[Reranking and Hybrid Search]])
6. Prompt construction — insert the top-scoring chunks into the LLM's
   context alongside the query and instructions
7. Generation — the LLM produces an answer grounded in the retrieved content
```

## Why RAG Instead of Just Fine-Tuning or a Bigger Context Window

**Fine-tuning** ([[Transfer Learning and Fine-Tuning]]) bakes knowledge into a model's weights — expensive to update (retraining needed whenever the underlying information changes) and doesn't give the model access to information it wasn't trained on at all. **A larger context window** doesn't scale to a corpus of thousands of documents — cost and latency both grow with tokens processed, and [[Attention Mechanism|attention's quadratic cost]] makes truly huge contexts impractical regardless of price. **RAG** keeps the model's weights fixed and instead retrieves only the small, relevant slice of a much larger corpus for each specific query — the knowledge base can be updated by re-indexing documents, with no retraining required, and cost scales with what's retrieved per query rather than the size of the entire corpus.

## Why Grounding Reduces Hallucination

An LLM asked a factual question with no supporting context must rely entirely on what it memorized during pretraining, which may be outdated, absent for niche/private information, or simply misremembered — this is the core cause of hallucination on knowledge-intensive queries. Providing the actual relevant source text directly in the prompt gives the model something concrete to draw its answer from rather than confabulating from parametric memory, which measurably (though not perfectly) reduces factual hallucination — see [[Hallucination Mitigation]] for the limits of this and other complementary techniques.

---

## Common Failure Modes

**Retrieval failure** — the right document exists in the corpus but wasn't retrieved (poor chunking, retrieval method mismatched to the query type, or the query phrased in a way that doesn't match the document's phrasing at all). No amount of prompt engineering fixes a generation step that never received the needed information.

**Context dilution** — too many retrieved chunks, or chunks that are only marginally relevant, crowd out the genuinely useful content and consume [[LLM Inference Fundamentals|context-window]] budget the model needs for actually reasoning about the answer.

**Stale or conflicting sources** — the corpus contains multiple versions of similar information (an old policy document alongside its replacement) and retrieval surfaces the wrong one, or both, leaving the model to reconcile contradictory context it has no principled way to resolve.

**The model ignoring retrieved content anyway** — even with perfect retrieval, an LLM can still answer from its own parametric memory instead of the provided context, especially if the provided context conflicts with strong pretrained beliefs — grounding reduces but does not eliminate hallucination risk.

---

## Interview Questions

**Why choose RAG over just fine-tuning the model on the private/updated documents?** Fine-tuning is expensive to redo every time the underlying information changes and bakes knowledge into weights that are hard to audit or selectively update; RAG keeps the model fixed and updates the knowledge base by re-indexing documents, which is far cheaper to keep current and lets you point to exactly which source text an answer was grounded in.

**Why doesn't a larger context window make retrieval unnecessary?** Attention's cost grows roughly quadratically with sequence length, and both cost and latency scale with tokens processed — dumping an entire large corpus into every prompt is prohibitively expensive and slow even where the context window technically fits it, so retrieval is what makes only the relevant slice get included per query.

**A RAG system gives a wrong answer — how do you diagnose whether it's a retrieval problem or a generation problem?** Check whether the correct source document was actually retrieved and present in the context given to the model — if it wasn't retrieved at all, the failure is in retrieval/chunking, no prompt fix will help; if it was retrieved but the model still answered incorrectly or ignored it, the failure is in generation/prompting, and better retrieval alone won't help there either.

## Connections

- [[Chunking Strategies]], [[Query Rewriting]], [[Dense vs Sparse Retrieval]], [[Vector Search and Databases]], [[Reranking and Hybrid Search]] — the pipeline stages this note ties together
- [[Transfer Learning and Fine-Tuning]] — the alternative knowledge-injection approach RAG is usually compared against
- [[Hallucination Mitigation]] — RAG is one mitigation among several, not a complete fix
- [[RAG Evaluation]] — how to actually measure whether a RAG system is working
- [[LLM Inference Fundamentals]] — context-window limits directly bound how much retrieved content fits

## One-line Summary

> RAG retrieves relevant document chunks and includes them in the prompt rather than relying on the model's parametric memory or an unaffordably large context — cheaper to keep current than fine-tuning, and it reduces (but doesn't eliminate) hallucination, provided retrieval actually surfaces the right content in the first place.
