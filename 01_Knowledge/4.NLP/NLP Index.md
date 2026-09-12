# NLP — Track Index

tags: #nlp #index #moc
links: [[00_Start_Here/README|Home]] [[01_Language_Basics/Index]] [[02_Text_Preparation/Index]] [[03_Text_to_Numbers/Index]] [[Sequence Models Index]]

---

> **Position in vault**: `01_Knowledge/4.NLP/`
> **Purpose**: The top-level map for the whole NLP track — every module index below links back here, and this is where the overall roadmap and status live.
> **Prerequisite**: General ML fundamentals help (`3.ML & DL/1.Concepts/1.Foundations`), but the track is written to be self-contained.

---

## Why NLP Gets Its Own Track

NLP sits at the intersection of linguistics and machine learning. The linguistics half (why language is structured the way it is) doesn't fit under `3.ML & DL`, and the ML half (how models learn from text) leans on general ML concepts already covered there without duplicating them. This track is the bridge: linguistic foundations → preprocessing → representation → modeling → tasks, with heavy cross-links back into `3.ML & DL`, `6.Statistics & Experimental Design`, and `5.Mathematics` rather than re-explaining general concepts locally.

## Module Map

| Module                                               | Status        | Notes | Covers                                                                                  |
| ---------------------------------------------------- | ------------- | ----- | --------------------------------------------------------------------------------------- |
| [[01_Language_Basics/Index\|01 — Language Basics]]   | ✅ Built       | 25    | Phonology, morphology, syntax, semantics, pragmatics, typology                          |
| [[02_Text_Preparation/Index\|02 — Text Preparation]] | ✅ Built       | 21    | Cleaning, normalization, tokenization, segmentation, annotation formats                 |
| [[03_Text_to_Numbers/Index\|03 — Text to Numbers]]   | ✅ Built       | 15    | Sparse vectors, static & contextual embeddings, subword/char representations            |
| ~~04 — Sequence Models~~ → [[Sequence Models Index\|moved]] | ✅ Built, relocated | 5 | RNN, vanishing/exploding gradients, LSTM, GRU, seq2seq — moved to `3.ML & DL/18.Deep Learning/2.Sequence Models/`, see note below |
| ~~05 — Attention & Transformers~~ → built elsewhere | ✅ Built, canonical home moved | — | Self-attention, multi-head attention, the Transformer architecture, positional encoding — built under [[Attention Mechanism]] / [[Transformer Architecture]] (`3.ML & DL/18.Deep Learning/4.Attention & Transformers/`); link to it, don't duplicate |
| 06 — NLP Tasks                                       | ⬜ Not started | —     | NER, text classification, machine translation, summarization, question answering        |
| 07 — Evaluation                                      | ⬜ Not started | —     | BLEU, ROUGE, perplexity, task-specific metrics                                          |
| 08 — Modern LLMs                                     | ⬜ Not started (canonical home moves too) | —     | Pretraining objectives, fine-tuning, prompting, RAG — being built under `3.ML & DL/19.Applied AI & LLM Systems/`, not here |

## A Scope Change Worth Documenting

The original plan (visible in `03_Text_to_Numbers/Index.md`'s "next section" pointer) named module 04 "Learning Core: Probability, Classical ML, Neural Fundamentals." That module was never built — and shouldn't be, as originally scoped. Probability and classical ML are now covered in depth under [[6.Statistics & Experimental Design]] and [[5.Mathematics]], and general ML fundamentals live in `3.ML & DL`. Rebuilding them inside NLP would duplicate content that already exists and would need to be kept in sync forever. Module 04 was renamed **Sequence Models** and covers what's genuinely NLP/DL-specific: RNNs, LSTMs, GRUs, and sequence-to-sequence architectures.

**2026-09 update**: those five Sequence Models notes have themselves moved to `3.ML & DL/18.Deep Learning/2.Sequence Models/`. RNN/LSTM/GRU/Seq2Seq/vanishing-exploding-gradients are architecture-general (used identically for time series, audio, etc.), not NLP-specific, so their canonical home is now the Deep Learning module — an AI/ML interview-prep pass looking for general DL architecture would never think to look under NLP for them. Their *application to text* is still what modules 02/03 above build toward; the architecture notes themselves just live one level up now. Same logic applies going forward: Attention/Transformers (05) and the LLM-era content (08) get built as general DL/Applied-AI content under `3.ML & DL`, with this track linking to them rather than re-deriving them for text specifically.

If a note here needs a general concept (e.g. gradient descent, overfitting, cross-entropy), it links out to the existing note in `3.ML & DL` rather than re-teaching it locally.

## Cross-Track Conventions

- Every module index has its own "Key Cross-Links to `3.ML & DL/`" table — check there before assuming a concept needs a new NLP-local note
- Underscore-vs-space filenames were a real bug here (fixed 2026-07-13) — new notes in this track should use spaces in filenames, matching the rest of the vault, not underscores
- Module-local indexes use `[[NLP Index]]` in their `links:` frontmatter to point back here — keep that convention for module 04 onward

## Reading Order

**Full track, front to back**: 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08

**Fastest path to "understand a modern LLM pipeline"**: 02 (Tokenization) → 03 (Static + Contextual Embeddings) → 04 (Sequence Models, for historical context) → 05 (Attention & Transformers) → 08 (Modern LLMs)

**Linguistics-first (for genuine understanding, not just pipeline mechanics)**: 01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 in full

## One-line Summary

> This is the map for the NLP track: four modules built (linguistics, preprocessing, representation, sequence models — the last deliberately rescoped to avoid duplicating the general ML content that now lives elsewhere), and four more planned (attention/Transformers, tasks, evaluation, modern LLMs).
