# Drug Pipeline Advancement Forecasting — Map of Content

## One-sentence version

A service that answers: given a drug program's molecule + indication + current phase, what's the probability it advances to the next phase, roughly how long might that take, and why — using SNAP's Postgres warehouse as the only source of truth, with **no ready-made label** to learn from. That last part is the hard problem this whole project is built around.

This note is the narrative spine of the project: it tells the story in the order it was actually built, and links out to the atomic, reusable concept notes for the underlying ideas rather than re-explaining them. Project-specific facts, numbers, and decisions live here; general concepts live in their own subject folders.

---

## Step 1 — Decide what NOT to touch

The warehouse has ~192 tables. Before writing any code, scope was narrowed hard:

- Ignored entirely: auth/identity, commercial/revenue/pricing, patents, deal-room UI tables, staging/flyway migration tables, duplicate `public.*` mirrors.
- Explicitly distrusted as labels despite looking tempting: `clinical_trial.probability_of_success` and `drug_pipeline_denorm.probability_of_success` — both empty/unused. `study_outcome` describes what a trial *measured*, not whether it *succeeded*. Textbook case of [[Schema Trust]] — a column named exactly what you want, that must not be used.

This converts "exploring blind" into "exploring with a falsifiable hypothesis about which ~15-20 tables actually matter." See also [[Data Warehouse]].

## Step 2 — Pick the spine and the enrichment source

- **Spine:** `drugs.drug_pipeline_denorm` (~1.5M rows) — one program-snapshot view with phase, status, indication, therapeutic area, dates.
- **Enrichment:** `clinical_trials.clinical_trial` (~591k rows) — canonical per-trial facts (design, enrollment, sponsor, dates), joined via `nct_code`.

The join was validated, not assumed: 95.6% of pipeline rows have an NCT code, and of those, 100% match a row in `clinical_trial`. See [[Spine Table Pattern]], [[Join Validation]], [[NCT Code]].

## Step 3 — Solve the identity problem: what *is* a "program"?

The spine table has no `molecule_id`/`indication_id` columns — just free-text `molecule_name` and `indication` strings. A separate table, `molecules.molecule_indication_link`, is 100% populated with both IDs for ~436k-788k pairs. This produced a **three-tier confidence key**:

| Tier | Key | Confidence |
|---|---|---|
| 1 | `molecule_id + indication_id` | High — both resolved |
| 2 | normalized `molecule_name + indication_id` | Medium |
| 3 | normalized `molecule_name + indication` text | Low — fallback |

Two discoveries that shaped this: `clinical_trials.drug_link.molecule_id` is 0% populated (a shortcut that looked real but was dead — see [[Schema Trust]] again); and using `therapeutic_area` alone as the rollup was tried and rejected — HIV-like indication strings scatter across different TAs (3,925 rows "Unclassified," 3,868 "Infectious Diseases," 650 "Immunology" — same disease, three buckets). TA is a **feature**, not the **identity**.

This is the load-bearing wall of the whole project. See [[Entity Resolution]], [[Confidence-Tiered Matching]], [[Drug Pipeline]], [[Indication vs Therapeutic Area]], [[Molecule]].

## Step 4 — Derive ground truth (there's no `advanced_to_next_phase` column)

Rules (documented in `ground_truth_rules.md`):

- **Success (label=1):** same program later shows evidence of a higher phase.
- **Failure (label=0):** either **timeout** (4 years pass, no higher phase) or **dead-end** (status Terminated/Withdrawn, no later higher phase).
- **Censored (label=null):** still active, under 4 years, no higher phase yet — **never trained as a failure**.

Concrete wrinkle handled explicitly: **combo phases**. Phase 1/2 counts as reaching Phase 2; Phase 2/3 counts as reaching Phase 3. This single choice moves the reported advance rate by ~24pp for p1→2 (42.6% inclusive vs. 19.1% strict) and ~6pp for p2→3. Decision: **train on inclusive, report both rates on every card** — transparency, not indecision.

Scope filter: keep Phase 1/1-2/2/2-3/3, drop Phase 4/ANDA/OTC/unapproved, require **industry sponsorship** — defined as "any linked NCT, at any point in the program's lifetime, had an INDUSTRY lead sponsor" (deliberately preserves academic→industry handoffs).

Result: **381,086 labeled programs** — 41,614/55,053/284,419 (success/fail/censored) for p1→2; 22,412/85,170/273,504 for p2→3.

See [[Censoring]], [[Clinical Trial Phases]], [[Trial Status]], [[Sponsor Type]].

## Step 5 — Build the modeling table with a leakage guard

Features restricted to **from-phase-only** signals (enrollment, sponsor type, design, regulatory flags *as of the current phase*) — never all-phase/lifetime aggregates. A program that eventually reaches Phase 3 naturally accumulates more NCTs over its life than one that dies at Phase 1; using lifetime NCT count as a feature would leak the answer into the input. ~71k industry transition rows produced.

See [[Data Leakage]].

## Step 6 — Two competing forecasting methods

1. **Analog baseline** (`src/forecasts/analogs.py`): backoff ladder — try TA × NCT-bin, back off to coarser buckets if too few matches, enforced by `min_analogs=40` (never estimate a probability from a handful of neighbors).
2. **ML models:** gradient-boosted trees ([[HistGradientBoostingClassifier]]) and [[Logistic Regression]] on from-phase features.

Rule going in: **ML only ships if it honestly beats the analog** — not "ML is fancier so we prefer it."

See [[Baseline Estimator]], [[Gradient Boosting]].

## Step 7 — The honest eval protocol (the intellectual core of the project)

Designed to prevent tuning on the number you're about to publish (circularity):

1. **[[Grouped Train-Test Split]]** (~60/20/20 by molecule) — same molecule can't appear in both train and test.
2. Candidate model + a discrete blend weight `{0, 0.25, 0.5, 0.75, 1.0}` between analog and ML, chosen **only on validation**.
3. **Test scored exactly once**, never re-tuned.
4. **[[Rolling-Origin Validation]]**: model must beat the analog on **≥3 of 4** historical cutoff years (2016/2018/2020/2022).
5. **Segment overrides**: only after passing the time gate, individual segments (e.g. TA × NCT-count bucket) tested with a **[[Permutation Test]]** corrected with **[[Benjamini-Hochberg Procedure]] at FDR=0.10**. Unclassified TA excluded by policy; a segment must win **4 of 4** time cuts (not 3 of 4) to ship — see [[Multiple Comparisons Problem]] for why this stricter bar matters at the segment level.
6. **Frozen config:** production code reads only `src/forecasts/frozen_config.py` — never the live eval JSON files. A re-run of the eval script can't silently change production behavior.

**Result:** global primary method is **analog** for both transitions. Only two narrow segments (`Psychiatry|4+`, `Pulmonology|4+` at p2→3) cleared the bar for ML. Not a failure of the ML work — the protocol doing its job: most ML gains were not honest gains once time-gated and multiple-test-corrected.

## Step 8 — Methodology audits (stress-testing the labels themselves)

Four audits run after the freeze:

- **Censoring bounds:** "drop censored" vs. "treat stalled-as-failed" — gap small (2.3pp/1.1pp at a 3-year stall); censored cohorts skew younger (recent trials still running, not silently-abandoned old ones). See [[Censoring]].
- **Combo-phase:** quantified above (Step 4).
- **Industry handoff:** confirms the "lifetime any-NCT INDUSTRY" rule retains academic→industry handoffs as designed (~7.8% of industry programs are mixed-sponsor). See [[Sponsor Type]].
- **Indication fragmentation:** same disease splits across different indication-ID strings. Blind rollup by ID moved rates by −5.1pp/−2.5pp — too risky to ship without hand QA, so a review sheet was built instead (`fragmentation_qa_review_v1.csv`); only manually-confirmed fragments get merged (barely moves the topline: −0.1pp). See [[Indication vs Therapeutic Area]], [[Entity Resolution]].

## Step 9 — Two spikes investigated and explicitly rejected

Negative results written up with the same rigor as positive ones, not hidden.

**MoA/route as a feature** (`moa_route_approval_spike_v1.md`):
- From-phase coverage: 2.3% route / 17.8% MoA — far below a 40%-null inventory gate.
- Adding it as a one-hot feature made the analog [[Brier Score]] *worse* for p1→2 (+0.010), only marginally helped ML for p2→3.
- Follow-up probe checked four alternate tables — best (NCT-level route) still ~24% coverage, still failing the gate. Dictionary-based MoA was actually *worse* than sparse free-text.
- **Decision: do not ship.** Documented as a named MVP gap, not silently dropped.

**Phase 3 → Approval** (same doc):
- Zero P3 clinical rows have a direct `fda_approval_date`.
- Exact molecule+indication matching to any FDA-dated row: 63 out of 108,579 (~0.06%).
- The tempting shortcut — "does the molecule have an FDA approval on *any* indication" — explicitly rejected: wrong unit, would credit a Phase 3 melanoma program as "approved" because the same molecule was approved for something unrelated. See [[Indication vs Therapeutic Area]] for why indication-level precision matters.
- Alternate FDA tables looked promising (`fda_drug_indication_link`, 18.8M rows) but were polluted — median ~2,000 indications linked per drug, saline and dextrose topping the list. The linkage table itself is essentially noise for this purpose. See [[Schema Trust]].
- **Decision: explicit reject**, not deferred — the gap is orders of magnitude, not a rounding error.

## Step 10 — Where it stands (2026-07-13)

| Layer | Status |
|---|---|
| Ground truth + product dataset | Done |
| Analog forecasts | Done (global primary) |
| ML + honest eval + freeze | Done |
| Segment overrides | Done (2 narrow segments only) |
| Combo policy | Done (train inclusive, report both) |
| MoA/route spike | Closed — do not ship |
| Phase 3→Approval spike | Closed — rejected |
| Thin API + demo | Exists, needs polish |
| Fragmentation hand QA | Open — needs manual labeling of the review CSV |
| Stakeholder (Narayanan) walkthrough | Not yet done |

---

## The actual intellectual center of gravity

Not the modeling — HGB vs. analog is a fairly standard comparison. It's Steps 3, 4, and 7: defining the unit of prediction ([[Entity Resolution]], [[Confidence-Tiered Matching]]), deriving labels honestly under censoring ([[Censoring]]), and building an eval protocol that can't lie to itself ([[Grouped Train-Test Split]], [[Rolling-Origin Validation]], [[Permutation Test]], [[Benjamini-Hochberg Procedure]]). That's the part worth being able to explain crisply — to a stakeholder or in an interview.

## Next steps

- Manual labeling of `fragmentation_qa_review_v1.csv`
- Stakeholder walkthrough
- Thin API/demo polish
- Deep-dive into the actual implementation (`src/forecasts/analogs.py`, `frozen_config.py`, `run_honest_eval.py`, labeling scripts) once available — line-by-line, against this map

## Tags

#project/drug-pipeline-forecasting #status/active #category/applied-ml
