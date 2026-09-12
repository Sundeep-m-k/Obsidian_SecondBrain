---
tags: [category/ml-dl, topic/applied-ai, index, moc]
---

# Applied AI & LLM Systems — Index

> **Position in vault**: `3.ML & DL/1.Concepts/19.Applied AI & LLM Systems/`
> **Purpose**: Applied, system-building LLM knowledge — the area this vault's Forward Deployed Engineer and Applied AI Engineer target roles were most underserved in before this module existed (0% coverage, confirmed in the 2026-09 gap analysis). Complements [[Deep Learning Index]] (the underlying architecture) with the practical layer of actually building and operating LLM-powered systems.
> **Prerequisite**: [[LLM Inference Fundamentals]] first — everything else in this module builds on tool calling and context-window constraints introduced there.

## Section Map

| Subfolder | Notes | Covers |
|---|---|---|
| 1. LLM Fundamentals | [[LLM Inference Fundamentals]] | Context windows, temperature/top-p, structured outputs, tool calling, prompting |
| 2. Retrieval & RAG | [[Dense vs Sparse Retrieval]], [[Vector Search and Databases]], [[Chunking Strategies]], [[Reranking and Hybrid Search]], [[RAG Architecture]], [[RAG Evaluation]] | The full retrieval-augmented-generation pipeline |
| 3. AI Agents | [[AI Agents Fundamentals]], [[Planning and Memory]], [[Workflows vs Agents]], [[Multi-Agent Systems]], [[Agent Failure Modes and Guardrails]] | Agent loops, when to use them, coordination, and failure modes |
| 4. Production AI Systems | [[Latency and Cost Optimization]], [[Observability and Evaluation for LLM Systems]], [[Hallucination Mitigation]], [[Prompt Injection and Production Reliability]] | Operating LLM systems reliably in production |

## Reading Order

LLM Fundamentals → Retrieval & RAG → AI Agents → Production AI Systems. Retrieval & RAG and AI Agents don't strictly depend on each other (a RAG pipeline is usually a fixed [[Workflows vs Agents|workflow]], not an agent), but most real systems combine both, so understanding RAG first makes the "agent with retrieval as one of its tools" pattern immediately clear once Agents is covered.

## Deliberately Not Duplicated Here

- **Tokenization** — canonical home is `4.NLP/02_Text_Preparation/3.Tokenization/` ([[Tokenization — BPE]], [[WordPiece and Unigram LM]], [[SentencePiece]])
- **Embeddings** — canonical home is `4.NLP/03_Text_to_Numbers/` ([[Word2Vec]], [[BERT Embeddings]], [[Sentence Embeddings]])
- **Transformer architecture itself** — covered in [[Transformer Architecture]] under `18.Deep Learning/4.Attention & Transformers/`; this module assumes it as a prerequisite rather than re-deriving it

## Key Cross-Links

| Applied AI Concept | Links to |
|---|---|
| Attention's quadratic cost → context window limits → cost/latency | [[Attention Mechanism]], [[LLM Inference Fundamentals]], [[Latency and Cost Optimization]] |
| RAG's retrieval stage | [[K-Nearest Neighbors]] (the exact-search algorithm ANN approximates), [[TF-IDF]] (BM25's ancestor) |
| RAG/agent evaluation metrics | [[Classification Metrics]] — precision/recall reapplied to retrieved chunks |
| Fine-tuning vs. RAG vs. prompting | [[Transfer Learning and Fine-Tuning]] |

## Common Exam / Interview Questions

1. Why does RAG reduce hallucination without eliminating it?
2. When would you choose a fixed workflow over a full autonomous agent?
3. Why do production systems combine sparse and dense retrieval rather than picking one?
4. What's the difference between context recall and faithfulness in RAG evaluation?
5. Why is prompt injection structurally harder to solve than SQL injection?
6. Walk through what you'd log to make an LLM agent's behavior debuggable in production.

## One-line Summary

> LLM Fundamentals covers the inference-time mechanics every downstream system depends on; Retrieval & RAG and AI Agents are the two dominant application architectures built on top of it; Production AI Systems covers what it actually takes to run any of the above reliably, cheaply, and safely.
