# Query Rewriting

## What is it?

**Query rewriting** transforms a user's original query into one or more different queries better suited to retrieval, before running [[Dense vs Sparse Retrieval|retrieval]] itself — closing the gap between how users actually phrase questions and how relevant content is actually written and indexed.

---

## Why User Queries Often Retrieve Poorly

A user's query and the document containing the answer frequently use different vocabulary, different levels of specificity, or different framing entirely — "how do I get my money back" vs. a knowledge-base article titled "Refund Policy and Eligibility Criteria" share little lexical overlap despite being an exact match in intent. [[Dense vs Sparse Retrieval|Dense retrieval]] partially closes this gap (it matches on meaning, not exact words), but even embedding similarity degrades when a query is too short, too vague, or conversationally underspecified relative to the documents it needs to match — a one-line follow-up question in a chat ("what about the second one?") carries almost no retrievable signal on its own at all.

---

## Techniques

**Query expansion** — add synonyms, related terms, or likely alternate phrasings to the original query before retrieval, widening what can match. Simple and cheap; risk is adding enough noise to hurt precision if the expansion drifts from the original intent.

**Query decomposition** — split a complex, multi-part question into several simpler sub-queries, retrieve for each separately, then combine the results. *Example*: "compare the refund policies for premium and basic plans" decomposes into "refund policy premium plan" and "refund policy basic plan" — retrieving for the compound query directly often surfaces content about neither plan specifically.

**Multi-query retrieval** — generate several different phrasings of the *same* underlying question (via an LLM), run retrieval for each, and merge/deduplicate the combined candidate set. Increases the chance that at least one phrasing lexically or semantically matches the target document, at the cost of running retrieval multiple times per user query.

**HyDE (Hypothetical Document Embeddings)** — instead of embedding the user's query directly, ask an LLM to generate a *hypothetical answer* to the query first, then embed *that* generated answer and use it for retrieval. The intuition: a plausible-sounding hypothetical answer is often lexically and semantically closer to the *actual* answer document than the original question is (a question and its answer are often phrased very differently, but two answers to the same question — one real, one hypothetical — tend to be phrased similarly). Works well for knowledge-intensive queries; risk is that a confidently-wrong hypothetical answer can retrieve confidently-wrong-but-related content instead of the right content.

**Conversational query rewriting** — in a multi-turn chat, rewrite a context-dependent follow-up ("what about the second one?") into a self-contained query using the conversation history ("what is the refund window for the basic plan?"). Without this, follow-up questions in any multi-turn RAG system retrieve close to nothing useful, since the query itself carries almost no retrievable content in isolation.

---

## When Rewriting Hurts Retrieval

Rewriting isn't free — it's an extra LLM call (cost, latency) and an extra place for errors to enter the pipeline. **Rewriting can actively hurt** when: the original query was already precise and a rewrite introduces drift from the user's actual intent; a decomposition splits a query in a way that loses an important cross-reference between the parts; or HyDE's hypothetical document confidently asserts something wrong, pulling retrieval toward plausible-but-incorrect content instead of correct content the original query would have found directly. **Practical mitigation**: run both the original and rewritten queries in parallel and merge results (similar to [[Reranking and Hybrid Search|hybrid search]]'s fusion approach) rather than trusting the rewrite exclusively.

---

## Evaluation Considerations

Query rewriting needs to be evaluated as its own pipeline stage, not folded into overall [[RAG Evaluation]] — measure retrieval quality (context precision/recall) **with and without** rewriting enabled, on the same evaluation set, to isolate whether the rewriting step is actually helping. A rewrite that improves recall (finds more relevant chunks) but hurts precision (also pulls in more irrelevant ones) is a real, measurable tradeoff, not a strict win — and the net effect on the final generated answer's [[RAG Evaluation|faithfulness]] is what ultimately matters, since better retrieval that gets ignored or misused downstream doesn't help.

---

## Worked Example

**Original user query** (in a multi-turn conversation about a SaaS product): *"what about for the cheaper plan?"*

**Conversational rewrite** (using prior turn: "what's the refund window for the premium plan?"): *"What is the refund window for the basic (cheaper) plan?"*

**Multi-query expansion** of the rewritten query, generated by an LLM:
1. "refund window basic plan"
2. "how long to request a refund on the basic subscription"
3. "basic plan cancellation and refund eligibility"

**Retrieved candidates** (top hits across all 3 rewritten queries, deduplicated): a "Refund Policy" doc chunk, a "Plan Comparison" doc chunk, a "Cancellation FAQ" doc chunk.

**Reranked final context** (via [[Reranking and Hybrid Search|cross-encoder reranking]] against the *original conversational intent*, not just the rewritten queries): the "Refund Policy" chunk's basic-plan-specific section ranks highest and is what's actually passed to the generation step — the other two chunks, while topically related, are dropped for being less directly responsive.

This end-to-end trace is the concrete answer to "why bother rewriting at all": the original query alone (`"what about for the cheaper plan?"`) would have retrieved close to nothing useful in isolation — it has almost no standalone lexical or semantic content to match against.

---

## Interview Questions

**Why can a well-functioning dense retrieval system still benefit from query rewriting?** Dense retrieval matches on meaning, but it still needs the query itself to *carry* enough meaning to match against — a short, vague, or conversationally context-dependent query (a chat follow-up, an underspecified question) can be nearly unretrievable on its own regardless of how good the embedding model is; rewriting restores the missing content before retrieval even runs.

**What is HyDE, and why might generating a fake answer help retrieval?** HyDE embeds a hypothetical, LLM-generated answer to the query instead of the query itself — the intuition being that a plausible answer is often phrased more similarly to the real answer document than the original question is, since questions and answers frequently use different vocabulary and structure even when about the same topic.

**When would query rewriting actively hurt a RAG system?** When the rewrite drifts from the user's actual intent, when a decomposition loses an important relationship between sub-parts of a compound question, or when HyDE's generated hypothetical answer is confidently wrong and pulls retrieval toward incorrect-but-related content — mitigated by retrieving on both the original and rewritten queries and merging results rather than trusting the rewrite exclusively.

**How would you evaluate whether adding query rewriting actually improved a RAG system?** Run the same [[RAG Evaluation|retrieval evaluation]] (context precision/recall) with and without rewriting enabled on a fixed evaluation set to isolate its effect, and check the downstream generation quality (faithfulness/answer relevance) too — since improved retrieval that doesn't translate into a better final answer isn't actually a win.

**Why does conversational query rewriting specifically matter for multi-turn RAG systems?** A follow-up question ("what about the second one?") is only interpretable using prior conversation turns — retrieving on the follow-up query in isolation has almost nothing to match against, so the query must be rewritten into a self-contained form using conversation history before retrieval can work at all.

## Connections

- [[RAG Architecture]] — where this stage sits in the overall pipeline (right after the user query arrives, before retrieval)
- [[Dense vs Sparse Retrieval]] — what the rewritten query(ies) actually get matched against
- [[Reranking and Hybrid Search]] — often used together with multi-query retrieval to merge and re-score candidates from several rewritten queries
- [[RAG Evaluation]] — how to measure whether rewriting is actually helping, not just assumed to help
- [[Planning and Memory]] — conversational rewriting draws on the same conversation-history state agents use

## One-line Summary

> Query rewriting closes the gap between how users phrase questions and how relevant content is actually written — via expansion, decomposition, multi-query retrieval, HyDE, or conversational rewriting — but it's an extra failure point, not a free win, and needs to be evaluated against retrieval quality directly rather than assumed beneficial.
