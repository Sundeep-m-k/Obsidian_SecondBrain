---
aliases: [Span ID — full methodological note, T1 deep dive]
tags: [autowiki, task/span-id, type/method-review, type/implementation-map]
status: draft
---

# Span Identification — Full Note (Codebase-Aligned)

> **Purpose:** Single reference for what the repo *actually* implements for Task 1, what gets logged, which metrics mean what, and what to report in a thesis/paper vs what needs extra experiments.

**Code anchors (library):** `src/span_identification/`  
**Entry script:** `scripts/02_Span_identification/01_run_span_id.py`  
**HF training:** `src/span_identification/hf_trainer.py`  
**Token dataset build:** `src/span_identification/preprocess.py`  
**Splits / examples:** `src/span_identification/dataset.py`  
**Post-hoc metrics:** `src/span_identification/evaluator.py`  
**Trainer metrics:** `src/span_identification/span_metrics.py`

---

## 1. Task definition (what we optimize)

**Input:** Plain text at one of three granularities: sentence, paragraph, or full article.

**Output:** A set of **character spans** `(char_start, char_end)` that should be hyperlinked. Downstream tasks choose targets; Task 1 only finds mention *boundaries*.

**Learning formulation:** Primarily **token classification** (sequence labeling) with decoding from predicted tags back to spans, then to character spans when needed.

---

## 2. Ground truth and “what is a positive span”

### 2.1 Where labels come from
- Processed wiki JSONL (see data pipeline / Task 0). Each unit carries a `links` list with offsets into the unit’s text.

### 2.2 Internal vs external links
- When `internal_only=True` (default for `ensure_splits` in the run script), **`link_type == "internal"`** only. Non-internal links are excluded from gold spans.

**Report in writing:** Fraction of links dropped by type (requires a one-off stats script or notebook — not automatic in core training).

### 2.3 Offset conventions (critical)
From `extract_spans_from_links` / preprocessing:
- **Article granularity:** `plain_text_char_start` / `plain_text_char_end` (absolute in article plain text).
- **Sentence / paragraph:** `plain_text_rel_char_start` / `plain_text_rel_char_end` (relative to unit text).

**Report:** State which fields are used per granularity; reviewers care because errors here look like “model is bad” when it’s alignment.

---

## 3. Train / val / test splits

### 3.1 Split unit
- Splits are by **`article_id`** (group units, then shuffle **articles** with seed, then assign 70% / 15% / 15% by default from `configs/span_id/base.yaml`).

### 3.2 Files
- Raw split units live under: `data/span_id/<domain>/splits/<train|val|test>_<granularity>.jsonl`
- `split_meta.json` records domain, granularity, seed, ratios, sizes.

### 3.3 Recreate policy
- `split.recreate_if_exists` controls whether existing split files are re-used.

### 3.4 Subtle but important: two split-creation paths
- **`ensure_splits`** loads existing split files OR creates new ones from processed JSONL.
- **`build_token_dataset`** calls **`create_splits` directly** on loaded units (with a seed) to build train/dev/test token JSONL.

**What to report:** Same split seed and procedures; if you ever change processed data or seed without deleting split files, document it as a new **corpus version**.

---

## 4. Label schemes and tagging algorithm

### 4.1 Supported schemes (code-level)
- **`BIO`**, **`BILOU`** are first-class for training data build and HF metrics.
- `tokenization.py` also implements **BIEOS** and **IO** helpers; `preprocess.assign_labels` explicitly implements BIO/BILOU/IO.

### 4.2 How spans become token labels
- Text is tokenized with **`return_offsets_mapping=True`**, `truncation=True`, `max_length=model.max_length` (default **512**).
- For each gold char span, tokens whose offsets **overlap** the span receive span labels; subtokens of one mention share B/I/L/U (BILOU) or B/I (BIO).

**Report:** Subword fragmentation effects; long articles are **truncated** — mention hits beyond truncation are silently affected unless you audit.

### 4.3 Storage layout for token datasets
- Tokenized JSONL is stored under:
  - `data/span_id/<domain>/token_data/<granularity>_<model>_<label_scheme>/train.jsonl` (and dev/test)
- **BIO and BILOU do not share the same directory** (scheme is in the path).

---

## 5. Models and training (canonical path = HuggingFace)

### 5.1 Active implementation
- **Canonical:** `hf_trainer.train_and_evaluate` using `AutoModelForTokenClassification` + `Trainer`.
- **Deprecated / reference only:** `trainer.py`, `models/token_classifier.py` (do not describe as active in papers unless you verify a script still calls them).

### 5.2 Optimization
- `TrainingArguments` uses standard AdamW + linear schedule defaults from config (`training.*`, `weight_decay`, `warmup_ratio`).
- `max_grad_norm` appears in YAML (`training.max_grad_norm`) — **verify** it is passed into `TrainingArguments` in `hf_trainer.py` when writing “we clipped gradients at X” (if not wired, don’t claim it).

### 5.3 Best checkpoint selection
- `metric_for_best_model="eval_exact_span_f1"` (span-level exact boundary F1 computed inside `compute_span_metrics_for_trainer`).
- Not the same as “seqeval token F1” shown as `eval_f1_seqeval`.

### 5.4 Early stopping (config vs code)
- `configs/span_id/base.yaml` defines an `early_stopping` block.
- **Action for honest reporting:** confirm whether `EarlyStoppingCallback` is actually attached in `hf_trainer.py`. If not, paper should say “fixed epochs with best-checkpoint selection on val,” not “early stopping.”

---

## 6. Metrics glossary (do not mix definitions)

### 6.A HF evaluation metrics (`span_metrics.compute_span_metrics_for_trainer`)
Returned keys include (prefix `eval_*`):
- **Token seqeval:** `eval_f1_seqeval`, precision, recall on BIO/BILOU tag sequences.
- **Exact span (token-decoded spans):** `eval_exact_span_precision`, `eval_exact_span_recall`, `eval_exact_span_f1`.
- **Relaxed / overlap span:** `eval_relaxed_span_*` (overlap between predicted vs gold **token spans**).
- **`eval_exact_match_pct`:** for each example with ≥1 gold span, fraction of gold spans exactly matched; then averaged across those examples.

### 6.B Post-hoc metrics (`evaluator.evaluate_example` + `aggregate_metrics`)
Used heavily for **baselines**:
- **`span_*`:** exact match between predicted and gold **character spans**.
- **`char_f1`:** treats characters as a set — overlap-based character F1 (different from trainer’s “relaxed span F1”).
- **`overlap_*`:** overlap span P/R/F at character-aligned evaluation path.
- **`exact_match_pct`:** None if no gold spans; otherwise matched gold / total gold.

### 6.C What the research CSV stores for models (`01_run_span_id.py`)
From `train_and_evaluate` return mapping:
- `span_f1`, `span_precision`, `span_recall` ← **`eval_exact_span_*` on test**
- `val_span_f1` ← validation **`eval_exact_span_f1`**
- **`char_f1` in CSV** ← `eval_relaxed_span_f1` on test (**trainer’s relaxed span**, *not* `evaluator.char_f1`)

**Paper rule:** Rename CSV columns mentally: `char_f1` is **relaxed span F1 (trainer)**, unless you recompute character metrics yourself.

---

## 7. Experiment sweep dimensions

`01_run_span_id.py` sweeps (unless config narrows):
- **domains**
- **granularities:** sentence, paragraph, article
- **models** (HF checkpoints)
- **label schemes:** BIO vs BILOU
- **seeds**
- **data_fractions** (subsample **training** rows inside HF dataset builder)

**Run identity:**
- **`run_id`:** timestamp `YYYYMMDD_HHMMSS` for a whole sweep
- **`checkpoint_path`:** directory including domain, granularity, sanitized model name, label scheme, seed, fraction

---

## 8. Baselines (`baselines.py`)

| Name | Idea | Reporting caveat |
|------|------|---------------------|
| `rule_capitalized` | Regex for Title Case phrases | Cheap; not entity-aware |
| `heuristic_anchor` | Looser “wiki-like” capitalized runs | Still surface-form |
| `random` | Random spans with n/length influenced by **gold** | **Not a label-free random baseline** — document honestly |

Baselines are evaluated on **both val and test** in `01_run_span_id.py`, and CSV stores test metrics as `span_f1` etc., with `val_span_f1` from val.

---

## 9. Statistical aggregation and tests

### 9.1 Aggregate across seeds
- `scripts/02_Span_identification/aggregate_seed_results.py` → mean ± std for model rows.

### 9.2 Compare models vs baseline p-values
- `scripts/02_Span_identification/compare_models.py` uses `bootstrap_significance` from `stats.py`.

**Report:** Describe paired bootstrap assumptions; note baseline padding logic in script when sample counts differ.

---

## 10. Inference for analysis (`predict_from_checkpoint`)

Used by error analysis:
- Loads tokenized test JSONL (pred path) and **raw** split JSONL (**same row order**; truncates if mismatch).
- Decodes using **`char_offsets`** stored in token JSONL to return **gold and pred char spans**.

**Failure modes to mention if present:**
- Missing `char_offsets` → fallback behavior uses token indices (debug path).
- Truncation reduces mentions in tail of long texts.

---

## 11. Error analysis (`error_analysis.py` + `03_error_analysis.py`)

Exports:
- Aggregate TP/FP/FN counts, length buckets, precision/recall/F1
- FP/FN samples JSONL

**Known analysis pitfall:**
- Baseline error-analysis script historically used **val for baselines** and **test for models** in some flows — **always document which split each table uses.**

---

## 12. Human evaluation stub

- `scripts/02_Span_identification/04_human_eval_sample.py` samples `heuristic_anchor` on **val** into `research/human_eval/` and writes a minimal annotation schema.

**Upgrade for a strong paper:** formal rubric + agreement + adjudication.

---

## 13. Reproducibility checklist (minimum credible story)

- [ ] Processed corpus version (build date / hash if available)
- [ ] Split seed + `split_meta.json`
- [ ] Exact config file path used (`span_id.yaml` + machine overrides)
- [ ] Token dataset directory (includes **model name + label scheme**)
- [ ] `run_id` and `checkpoint_path` for each row you cite
- [ ] Library versions (`transformers`, `torch`) in appendix
- [ ] Clarify metric mapping (Section 6)

---

## 14. Open questions / next analyses (research backlog)

1. **Calibration:** output probabilities / expected F1 calibration for downstream NIL thresholds.
2. **Truncation audit:** how many gold spans are partly/fully lost due to 512 cap (by granularity).
3. **Overlap / near-miss taxonomy:** boundary errors vs wrong mention vs extra mentions.
4. **Label noise:** editor behavior vs “mentionhood” — inter-annotator or heuristic audit.
5. **Statistical rigor:** bootstrap CIs for key metrics at span level, not only seed variance.

---

## Related vault notes

- [[Task 0 - Dataset and Ground Truth - Method Review]] (if you created it)
- [[Task 2 - Article Retrieval - Method Review]]
- [[Task 3 - Linking Pipeline - Method Review]]

## Revision log

- 2026-04-13 — Generated from repository structure and source files (`span_identification` package + `01_run_span_id.py`).