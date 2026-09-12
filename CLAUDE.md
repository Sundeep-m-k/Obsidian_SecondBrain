# CLAUDE.md — Second Brain Vault

Operating instructions for Claude when working inside this Obsidian vault. This vault is the **persistent source of truth** — it survives across conversations, this file does not carry memory of past sessions. Re-derive context by reading the vault itself, not by assuming a prior conversation's conclusions still hold.

## What this vault is

A personal knowledge base + interview-prep system, structured around one core split:
- `01_Knowledge/` — timeless, reusable **concept notes**. One note per idea, cross-linked, never tied to a specific job, employer, or project.
- `02_Projects/` — applied work. Each project gets one MOC note that narrates the build and links out to the `01_Knowledge` concepts it used. Project-specific facts/numbers/decisions live here, never in the concept notes they cite.
- `02_Research/` — papers (via `99_Templates/Paper_Template.md.md`), research ideas, literature maps, and an in-progress "Autowiki".
- `00_Start_Here/README.md` — the living vault index, powered by Dataview queries (subject note counts, placeholder subjects, recently updated, active projects). It re-queries automatically; don't hand-edit its tables to "fix" them — fix the underlying notes/tags instead.
- `99_Templates/` — note templates. Use them when creating a note of a type a template already covers (currently: paper reading notes).

Numbered top-level subject folders under `01_Knowledge` (`1.`, `2.`, `3.`...) roughly follow a learning-path order; numbered subfolders inside each subject reflect that subject's own internal progression (foundations before advanced topics). **New subjects get their own new numbered folder — never nested inside an unrelated existing subject.** This mistake has been made and fixed before; don't repeat it.

## Before creating anything: search first

1. Search the vault (filename and content, e.g. `grep -ri` / `find -iname`) for the concept by its likely names, synonyms, and abbreviations before creating a note. Obsidian resolves `[[wikilinks]]` by filename vault-wide regardless of folder, so a match anywhere in the vault counts, not just in the "obvious" folder.
2. If a note for the concept already exists: **update it**, don't create a sibling with a slightly different name or scope. Read it fully first (see "Never overwrite" below).
3. If a *closely related but distinct* note exists (e.g. `Feature Scaling` vs `Normalization` vs `Standardization`), check whether the new content actually belongs in that note before adding a new one — this vault already has cases where near-duplicate notes were created in different build passes and never reconciled (e.g. `Feature Scaling.md` under one subject's early module vs. a near-identical note added later under a different numbered module, with no cross-link between them). Don't repeat that pattern.
4. If genuinely new and atomic, place it in the folder whose sibling notes it most resembles in scope — see "When uncertain where something belongs" below.

## Never do these without explicit permission

- **Never bulk-reorganize or bulk-delete files.** No mass renames, no folder restructuring, no "cleanup" passes across many files, even if a review identified real problems — propose the change and get a yes first.
- **Never overwrite substantial existing content without reading and reviewing it first.** A short stub can be safely expanded; a note with real worked examples, tables, or hard-won project-specific detail (e.g. `Schema Trust.md`, project MOCs) must be read in full and reasoned about before any edit — preserve what's good, don't just replace it.
- **Never restructure folder hierarchy or renumber module folders** as a side effect of adding content, even when you notice an inconsistency (e.g. the numbered-but-unnamed folders `15`/`16`/`17`/`18` under ML & DL vs. the `N.Descriptive Name` convention everywhere else). Flag inconsistencies; don't silently fix them.
- Don't invent new top-level tag namespaces or template formats ad hoc — extend the existing conventions below.

## House style, as observed in the vault

**Frontmatter**: Only MOC/Overview/index notes carry YAML frontmatter (a `tags:` list). Ordinary atomic concept notes typically have **no YAML frontmatter** — they open directly with an H1 title. Match whichever pattern the sibling notes in that folder already use; don't add frontmatter to atomic notes in a folder where none of the siblings have it.

**Atomic concept note shape** (the dominant pattern, seen across Databases, Statistics, most of ML & DL, etc.):
```
# Title

## What is it?
(definition, formalized with notation/LaTeX where the concept warrants it)

(body sections as needed — key properties, worked examples, comparison tables,
common mistakes, interview/discussion questions — depth should match the
concept's actual importance, not be padded to hit a template)

## Prerequisites          (optional, some subjects use this)
[[Concept A]]

## Related concepts / ## Connections   (naming varies slightly by subject —
match the sibling notes in the same folder)
[[Concept B]], [[Concept C]]

## Tags                   (some subjects; a hashtag line, not YAML)
#category/<subject> #topic/<area> #math/<branch>

## One-line summary
> A single sentence a reader could act on without reading the rest of the note.
```

**MOC / Overview / Index notes**: YAML frontmatter with `tags: [index, moc]` (or `[..., placeholder]` if the subject has no real content yet), then sections like: Purpose, Section Map (a table of subfolders → covered concepts → build status ✅/⬜), Why This Structure, Key Cross-Links to Other Subjects, Common Exam/Interview Questions, One-line Summary. Every subject folder is expected to have exactly one of these at its top level — if one is missing (as with `ML & DL` and `Domain Knowledge` currently), that's a gap to flag, not one to silently fill in without being asked.

**Tag conventions** (from `00_Start_Here/README.md` — the canonical definition, don't reinvent):
- `#category/<subject>` — which subject a note belongs to
- `#topic/<area>` — finer-grained area within a subject
- `#math/<branch>` — which branch of math a note leans on
- `#status/placeholder` — scaffolded subject/folder with no real content yet
- `#project/<name>` — applied project notes

**Links**: Use `[[wikilinks]]` for any cross-reference to another concept, whether same-folder or cross-subject (e.g. a Databases note linking into Statistics is normal and expected, not a smell). When adding or editing a note that references concept X, check whether X's own note should link back — bidirectional links between clearly related concepts are the norm here (see `L1 Regularization.md` ↔ `L2 Regularization.md`, or the Drug Pipeline project MOC linking out to a dozen `01_Knowledge` concepts it used). Before adding a link, confirm the target note actually exists under that exact filename — Obsidian link resolution is filename-based, and this vault currently has real breakage from targets that don't exist yet (e.g. a linked-but-never-created `Confusion Matrix` note) and from numbering drift between a link and the file it meant to point to (e.g. a link to `15.3_PR_Curves` when the real file is `15.3_Calibration_and_Probability_Evaluation.md`). Don't add another broken link; if the target should exist but doesn't, say so rather than linking speculatively.

**Naming**: Match the exact filename casing/spacing convention already used in that folder (e.g. `Title Case With Spaces.md` in most subjects; `15.1_Snake_Case_Title.md` in the ML & DL evaluation/data-prep/MLOps modules). Don't mix conventions within a folder.

## Keep knowledge and project content separate

- Reusable, timeless concept explanations → `01_Knowledge`.
- What a specific project actually did, decided, rejected, or discovered (including project-specific numbers, dataset quirks, or one-off bugs) → the project's own MOC/notes under `02_Projects`, with links out to the general `01_Knowledge` concept it applied.
- If project work surfaces a genuinely general, reusable insight (the vault already does this well — e.g. `Schema Trust.md` reads as a concept distilled from real project pain, but lives in Databases as a general note, not inside the project folder), extract it into `01_Knowledge` and link it from the project MOC — don't leave general-purpose insight stranded inside project-specific files, and don't pollute a concept note with project-specific specifics either.

## When uncertain where something belongs

Look at 2-3 existing notes in the candidate folders and match their scope, depth, and naming before deciding — don't guess from the folder name alone. If a concept plausibly fits two folders (e.g. a preprocessing technique that could go under an early "Features & Representation"-style module or a later "Data Preparation"-style module), check whether the vault already has both and whether they've diverged before adding to either — and mention the ambiguity rather than silently picking one.

## Compatibility

- Keep everything valid, plain Obsidian Markdown: standard `[[wikilink]]` syntax, `$...$`/`$$...$$` for LaTeX (MathJax, as used throughout), and fenced code blocks for code/pseudocode. Don't introduce syntax Obsidian can't render.
- Don't hand-edit the Dataview query blocks in `00_Start_Here/README.md` to "fix" what they display — they're generated views over the vault's real state; fix the underlying notes/tags/folders instead and the views update themselves.
- Preserve existing `.obsidian/` configuration; don't touch plugin settings.

## Reporting, not just doing

When asked to review or audit part of the vault, give the honest assessment (redundancy, staleness, broken links, missing coverage) without pre-emptively fixing it — this vault's owner wants to see the diagnosis and decide what to build before anything is changed. Cite exact file paths as evidence for every claim. Only move to making changes once the plan is explicitly approved.
