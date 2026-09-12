---
aliases: [Span ID method review, T1 methodology]
tags: [autowiki, task/span-id, type/method-review]
status: draft
---

# Task 1 — Span Identification — Method Review

## 1. Purpose / Task Definition

**Goal:** Given a text unit from a wiki (sentence, paragraph, or full article), predict which **character spans** should become hyperlinks — i.e., which substrings should be linked to some target page in a later stage.

**Scope boundary (important):** Task 1 does **not** decide the final URL or target article title. It outputs **candidate spans** (mention boundaries). Target disambiguation is deferred to [[Task 2 - Article Retrieval - Method Review]] and end-to-end linking behavior to [[Task 3 - Linking Pipeline - Method Review]].

**Formal framing:** The implementation treats this primarily as **sequence labeling** (token classification) with decoding back to character offsets.

---

## 2. Inputs and Outputs

### Inputs
- Processed wiki text with gold internal-link annotations from the data pipeline (character-aligned anchor spans in ground truth).
- Training/eval units at chosen **granularities**: typically `sentence`, `paragraph`, `article`.

### Outputs (what the system must produce)
- Predicted spans as `(char_start, char_end)` intervals aligned to plain text.
- Optional token-level predictions internally, but the research metrics are span-centric.

---

## 3. Supervision: Where Labels Come From

- Supervision is derived from parsed wiki pages: mention boundaries come from actual internal links present in the corpus.
- Splits are intended to respect **document/article boundaries** (article-ID-based split configuration in config), reducing trivial leakage across units from the same article.

### Implications
- Label noise inherits from parsing/HTML cleaning and boundary conventions.
- Ambiguity in “should this phrase be linked?” is partially baked into what editors linked historically (not a pure linguistic gold standard).

---

## 4. Modeling Approach (Current System)

### Core model family
- Transformer encoder token classifiers (examples in project configs include BERT/RoBERTa/DeBERTa-style baselines).

### Labeling schemes
- BIO and BILOU encodings (two common span-encoding conventions).

### Training setup (conceptual)
- Standard supervised training with validation-driven early stopping (metric emphasizes validation span quality in the pipeline design).
- Multi-seed training is supported at the project level to study variance.

### Baselines
- Rule / heuristic / random baselines exist as sanity checks and floor references.

**Concept links (if you have them in vault):**
- [[Sequence Labeling]]
- [[BIO tagging]]
- [[BERT]]
- [[Token Classification]]

---

## 5. Evaluation Metrics (What “good” means here)

Primary orientation in this project:
- **Exact span F1** (and related span precision/recall) is treated as a primary diagnostic.
- Additional relaxed metrics (e.g., character overlap style metrics) appear for analysis (depending on implementation details).

### Why exact span F1 is strict
- Small boundary errors become full false positives/false negatives under exact-match aggregation.
- This is appropriate for hyperlinking where offsets matter — but it can undercount “almost correct” behavior unless complementary metrics are reported.

---

## 6. Error Analysis (How failures are inspected today)

### Mechanism
- Aggregate confusion via TP/FP/FN at the span-set level.
- Bucket errors loosely by span length (short vs long) for a coarse prior on error types.
- Export sampled FP/FN examples for qualitative review.

### Known methodological limitations (conceptual)
- Taxonomy is shallow: mostly “wrong span exists / missing span” rather than boundary-shift categories.
- Any analysis process that mixes validation for baselines and test for models can distort direct comparisons unless explicitly standardized.

---

## 7. What Task 1 Is *Good* At Demonstrating

- Clean **component isolation** for mention detection quality before retrieval complexity enters.
- Controlled study across **granularity** (short context vs long context).
- Practical multi-seed reporting for stability.

---

## 8. What Task 1 Is *Weak* At (Research Framing Risks)

- Mention detection + boundary precision is inherently hard; token tagging can struggle with:
  - nested/overlapping mentions (if present in data or labeling),
  - long-range dependencies inside `article` granularity,
  - systematic ambiguity (linking policy vs linguistic mentionhood).

---

## 9. Reproducibility Checklist (What a strong paper needs here)

- Fixed data snapshot identifier (corpus hash / build timestamp).
- Exact split files referenced.
- Model identifier, hyperparameters, seeds, and checkpoint selection rule.
- Clear statement whether results are reported on val vs test for each table row.

---

## 10. Open Questions (to drive literature critique next)

1. Is sequence labeling the right outer loop, or should mention detection be **span proposal + classification**?
2. How should near-miss boundaries be counted for hyperlinking (strict F1 vs overlap-tolerant criteria)?
3. What calibration does mention detection output need for downstream NIL / linking thresholds?
4. Does the data distribution match “editor behavior” more than “optimal linking policy”? If so, what’s the right evaluation narrative?

---

## Related Notes

- Next step (critique/survey): [[Task 1 - Literature Survey]]  (create if missing)
- Downstream dependency: [[Task 2 - Article Retrieval - Method Review]]
- End-to-end: [[Task 3 - Linking Pipeline - Method Review]]

## Source of truth in repo (for your own traceability)

- Library: `src/span_identification/`
- Entry scripts: `scripts/02_Span_identification/`
- Config patterns: `configs/span_id/`

---

## Revision log

- YYYY-MM-DD — First draft from methodological review discussion.