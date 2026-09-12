# RAG Evaluation

## What is it?

Evaluating a [[RAG Architecture|RAG]] system means measuring two genuinely separate things — did retrieval find the right content, and did generation use it correctly — because a RAG system can fail at either stage independently, and a single end-to-end "is the answer right" metric can't distinguish which one is actually broken.

---

## Retrieval Metrics

**Context Precision** — of the chunks retrieved, what fraction were actually relevant to the query? Low precision means [[RAG Architecture|context dilution]] — irrelevant chunks crowding out useful ones.

**Context Recall** — of the chunks that *should* have been retrieved to fully answer the query, what fraction actually were? Low recall means the answer is missing information regardless of how well generation performs, since the model was never given what it needed.

These are the same precision/recall tradeoff from [[Classification Metrics]], applied to retrieved chunks instead of predicted class labels.

## Generation Metrics

**Faithfulness (Groundedness)** — does the generated answer's content actually follow from the retrieved context, or does it introduce claims the context doesn't support (a RAG-specific form of [[Hallucination Mitigation|hallucination]], happening even when retrieval succeeded)?

**Answer Relevance** — does the generated answer actually address the query being asked, independent of whether it's grounded in the context (a faithful-but-off-topic answer still fails the user)?

## LLM-as-Judge

Because "is this answer faithful to the context" and "is this answer relevant to the query" are semantic judgments that a simple string-matching metric (like exact match or ROUGE) can't reliably make, RAG evaluation commonly uses **another LLM as an automated judge** — prompted with the query, retrieved context, and generated answer, asked to score faithfulness/relevance according to a defined rubric. This scales far better than human evaluation for iterating quickly on system changes, at the cost of inheriting the judge model's own biases and blind spots — an LLM judge can be fooled by fluent-sounding but subtly wrong answers, and its judgments should be spot-checked against human evaluation periodically rather than trusted unconditionally forever.

```
Judge prompt sketch:
"Given this CONTEXT and this ANSWER, is every claim in the ANSWER
directly supported by the CONTEXT? Rate faithfulness 1-5 and explain."
```

## Building an Evaluation Set

A representative set of (query, expected relevant chunks, reference answer) triples — ideally covering the actual variety of real user queries, not just easy cases — is what every metric above is computed against. Without this, RAG evaluation degenerates into anecdotal "it seems to work on the examples I tried," which doesn't catch regressions when the pipeline changes (a new chunking strategy, a different embedding model, a reranker swap).

---

## Interview Questions

**Why can't you evaluate a RAG system with a single end-to-end "is the answer correct" metric?** Because the system can fail at retrieval (never finding the right content) or at generation (having the right content but not using it correctly) independently, and a single pass/fail metric can't tell you which stage to fix — separating context precision/recall from faithfulness/relevance localizes the actual failure.

**What's the difference between context recall and faithfulness?** Context recall measures whether retrieval found everything needed to answer the query at all; faithfulness measures whether the generated answer's claims are actually supported by whatever context was retrieved — a system can have perfect recall and still hallucinate unsupported claims, or have poor recall and still generate a faithful (if incomplete) answer from what little was retrieved.

**What's the main risk of using an LLM as a judge to evaluate RAG outputs?** The judge model has its own biases and blind spots — it can be fooled by fluent, confident-sounding answers that are subtly wrong, or systematically favor certain answer styles — so LLM-judge scores should be periodically validated against human evaluation rather than trusted as ground truth indefinitely.

## Connections

- [[Classification Metrics]] — context precision/recall are the same precision/recall concept applied to retrieved chunks
- [[RAG Architecture]] — the pipeline these metrics are diagnosing
- [[Hallucination Mitigation]] — faithfulness evaluation directly measures this failure mode
- [[Confusion Matrix]] — the underlying counts behind any precision/recall-style retrieval metric

## One-line Summary

> RAG evaluation must separate retrieval quality (context precision/recall) from generation quality (faithfulness, answer relevance) since either can fail independently — LLM-as-judge scales this better than human evaluation but needs periodic validation against it, and both require a real evaluation set, not anecdotal spot-checks.
