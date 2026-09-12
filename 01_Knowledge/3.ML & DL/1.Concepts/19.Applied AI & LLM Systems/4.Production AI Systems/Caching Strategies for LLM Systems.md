# Caching Strategies for LLM Systems

## What is it?

"Caching" in an LLM system actually refers to at least four mechanically distinct techniques that get conflated in casual conversation — they reuse different things, invalidate under different conditions, and carry different risks. Treating them as one topic is a real, common mistake; this note exists specifically to keep them separate.

| | What's reused | Match requirement | Where it lives |
|---|---|---|---|
| **Prompt caching** | Server-side computation for a repeated prompt *prefix* | Exact (or exact-prefix) match | LLM provider / inference server |
| **Semantic caching** | A previous full *response* | Semantic similarity of the query | Application layer |
| **Ordinary application/response caching** | Any previous computed result | Exact key match (e.g. hash of full input) | Application layer |
| **KV cache** | Per-token attention Key/Value tensors *within one generation* | N/A — internal to a single request | Inference server, during one autoregressive generation |

---

## Prompt Caching

**What's reused**: the internal computation (attention Key/Value tensors, specifically — see [[Transformer End-to-End Walkthrough]]'s step 4) for a prompt's *prefix*, when a later request sends the same prefix again. If a system prompt plus a large retrieved document is 3,000 tokens and only the final 50-token user question changes between calls, prompt caching lets the provider skip recomputing those 3,000 tokens' worth of attention every single time.

**Why exact/prefix matching, specifically**: the cached computation is only valid if the *exact same tokens, in the exact same order*, precede the point of reuse — attention over a prefix depends on every token in it, so even a single-token difference anywhere in the cached prefix invalidates the cache for everything after that point. This is why prompt caching is typically described as prefix-based: `[cached system prompt][cached document][NEW: varying question]` works; `[cached system prompt][NEW: varying instruction][cached document]` does not, because the varying part now sits *before* the part meant to be cached.

**Provider-side vs. application-side**: most major LLM APIs implement this automatically server-side (detecting a repeated prefix and reusing cached computation transparently) rather than requiring the application to manage it — the application's job is simply to structure prompts so the *stable* parts (system instructions, large static context) come first and the *varying* parts (the actual per-request question) come last, maximizing how much of each call can hit the cache.

**Impact**: reduces both latency (skips recomputation) and cost (many providers bill cached tokens at a reduced rate) for the cached portion specifically — it does not reduce cost/latency for the non-cached (varying) portion of the prompt, or for output generation at all.

**Cache invalidation**: typically time-based (a cache entry expires after some window of inactivity) since providers must eventually evict unused cached prefixes — an application relying heavily on prompt caching for a rarely-hit prefix may not benefit if requests are spaced far enough apart that the cache expires between them.

---

## Semantic Caching

**What's reused**: an entire previous *response*, served for a *new* request whose query is semantically similar to a previously-seen one — not textually identical.

**Mechanism**: embed the incoming query (see [[Word2Vec]]/[[BERT Embeddings]]/[[Sentence Embeddings]] for how), compare it via [[Vector Search and Databases|vector similarity search]] against embeddings of previously-answered queries, and if the closest match exceeds a similarity **threshold**, return that cached response directly — skipping the LLM call (and any retrieval) entirely.

**Why this is a fundamentally different mechanism from prompt caching**: prompt caching requires exact prefix reuse and saves partial computation; semantic caching requires only *approximate meaning* reuse and saves the *entire* call. "What's your refund policy?" and "how do refunds work?" would never trigger prompt caching (different token sequences entirely) but could both hit the same semantic cache entry.

**Risks specific to semantic caching**:
- **False semantic matches** — two queries can embed as similar while actually needing different answers ("what's the price of the basic plan" vs. "what's the price *difference* between plans" might embed closely but have very different correct responses); an overly loose similarity threshold serves confidently wrong cached answers.
- **Freshness** — a cached response can go stale if the underlying information changes (a policy update, a price change) and the cache doesn't know to invalidate; semantic caching needs an explicit invalidation strategy (time-based expiry, or invalidation tied to the underlying data changing) that exact-match caches enforce implicitly less often.
- **Context/user isolation** — a response cached for one user or tenant can leak into another's results if the cache doesn't segment by user, permissions, or tenant — a serious problem when responses were personalized or drew on access-controlled data (see [[Prompt Injection and Production Reliability]]'s treatment of tenant isolation for the same underlying concern applied elsewhere in the pipeline).
- **Safety review bypass** — if responses pass through a safety/moderation check before being served, a semantically cached response might bypass that check on a repeat "hit," serving previously-approved content that may no longer be appropriate in the new context.

---

## Ordinary Application/Response Caching

The traditional software-engineering cache — key the cache by an exact hash of the full request, serve an identical cached response only for byte-identical repeat requests. Simpler and safer than semantic caching (no similarity-threshold judgment calls, no false-match risk) but far less useful for LLM applications specifically, since near-identical-but-not-identical user phrasing is the norm, not the exception — most real user queries never repeat exactly, so an exact-match cache alone captures relatively little of the traffic a semantic cache would.

---

## KV Cache — Not the Same Kind of Thing At All

The [[Transformer End-to-End Walkthrough|KV cache]] described in the Transformer walkthrough operates entirely *within a single generation request* — it stores each layer's Key/Value tensors for already-generated tokens so each new token in an autoregressive generation doesn't require recomputing attention for the whole sequence from scratch. It has nothing to do with reuse *across different requests* (which is what the three caches above are about) — it's a within-request optimization, not a cross-request one. Confusing it with prompt caching is an easy mistake since both involve "reusing K/V tensors," but they operate at completely different scopes: KV cache is per-request, per-generation; prompt caching is cross-request, at the prefix level.

---

## Interview Questions

**What's the actual mechanical difference between prompt caching and semantic caching?** Prompt caching reuses server-side *computation* for an exact-matching prompt prefix, saving partial processing time on a request that must still be sent; semantic caching reuses an entire previous *response* for a semantically similar (not identical) query, potentially skipping the LLM call entirely — one is a computation optimization requiring exact matching, the other is a full-response shortcut requiring only approximate meaning match.

**Why is prompt caching typically described as "prefix" caching specifically?** Attention computation over a sequence depends on every preceding token, so cached computation is only valid if the exact same tokens precede the reuse point in the exact same order — a single differing token anywhere in the supposedly-cached prefix invalidates everything computed after it, which is why applications are advised to put stable content (system prompts, static context) before variable content (the actual per-request question) in the prompt.

**What's the biggest risk unique to semantic caching that prompt caching doesn't have?** False semantic matches — two genuinely different queries can embed as similar enough to cross a similarity threshold while needing different correct answers, silently serving a wrong cached response; prompt caching has no equivalent risk since it requires exact prefix matching, not approximate similarity.

**Why must a semantic cache be isolated per user/tenant in a multi-tenant system?** A cached response computed for one user (potentially using personalized or access-controlled data) could otherwise be served to a different user whose semantically similar query shouldn't return that same content — an information leak across tenant/permission boundaries, not merely a correctness bug.

**Is the KV cache used during autoregressive generation the same thing as prompt caching?** No — the KV cache operates within one single generation request (avoiding recomputing attention for already-generated tokens as new ones are produced), while prompt caching operates across separate requests (reusing computation for a repeated prompt prefix between different API calls) — they share the underlying Key/Value tensor concept but solve different problems at different scopes.

## Connections

- [[Latency and Cost Optimization]] — prompt caching as one of several cost/latency levers
- [[Transformer End-to-End Walkthrough]] — the KV cache's mechanism, explained in the generation pipeline
- [[Vector Search and Databases]] — the similarity-search infrastructure semantic caching is built on
- [[RAG Evaluation]] — a semantic cache hit should ideally be evaluated with the same rigor as a fresh generation, not assumed safe
- [[Prompt Injection and Production Reliability]] — tenant/context isolation concerns recur here for the same underlying reason

## One-line Summary

> Prompt caching reuses server-side computation for an exact-matching prompt prefix, semantic caching reuses an entire previous response for a semantically similar (not identical) query, ordinary application caching reuses results only for byte-identical requests, and the KV cache is a within-request generation optimization unrelated to reuse across requests at all — four different mechanisms, each with a different invalidation and risk profile, routinely (and incorrectly) discussed as if they were one thing called "caching."
