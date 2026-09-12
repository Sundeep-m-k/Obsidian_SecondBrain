---
tags: [index, moc, home]
---

# Second Brain — Home

Entry point for the whole vault. Everything below is powered by Dataview, so it stays current automatically as notes get added — nothing here needs manual upkeep.

---

## Subjects

```dataview
TABLE length(rows) AS "Notes"
FROM "01_Knowledge"
WHERE file.name != "README" AND file.name != "Home"
GROUP BY regexreplace(file.folder, "^(01_Knowledge/[^/]+)/?.*$", "$1") AS Subject
SORT Subject ASC
```

## Placeholder Subjects — Not Started Yet

Anything tagged `#status/placeholder` is a scaffolded subject with no real content yet. This list empties out on its own as you fill each one in.

```dataview
LIST
FROM #status/placeholder
SORT file.folder ASC
```

## Recently Updated

```dataview
TABLE file.folder AS "Location", file.mtime AS "Modified"
FROM "01_Knowledge" OR "02_Projects"
SORT file.mtime DESC
LIMIT 15
```

## Active Projects

```dataview
TABLE file.mtime AS "Updated"
FROM "02_Projects"
SORT file.mtime DESC
```

---

## Manual Quick Links

Things worth a direct link rather than a query — top-level maps of content for the deepest subjects.

- [[01_Knowledge/4.NLP/NLP Index|NLP — full track index]]
- [[01_Knowledge/3.ML & DL/1.Concepts/13.Meta Understanding/ML Cheatsheet|ML & DL — cheatsheet]]
- [[02_Projects/Drug-Pipeline-Advancement-Forecasting/MOC|Drug Pipeline Advancement Forecasting — project MOC]]

---

## How This Vault Is Organized

`01_Knowledge/` holds timeless, reusable concept notes — one note per idea, cross-linked, never tied to a specific job or project. `02_Projects/` holds applied work — each project gets one MOC note that narrates the build and links out to the concept notes it depends on, keeping project-specific facts and numbers separate from general concepts. `99_Templates/` holds note templates (e.g. the paper-reading template for `10.Research`).

Within `01_Knowledge`, numbered subject folders (`1.`, `2.`, `3.`...) roughly reflect a learning-path order; numbered subfolders inside each subject reflect that subject's own internal progression (foundations before advanced topics). New subjects get added as new subject folders, not stuffed into existing ones — the "don't nest unrelated disciplines" mistake got made and fixed once already, no need to repeat it.

## Tag Conventions

- `#category/<subject>` — which subject a note belongs to (e.g. `#category/statistics`)
- `#topic/<area>` — finer-grained area within a subject (e.g. `#topic/hypothesis-testing`)
- `#math/<branch>` — which branch of math a note leans on (e.g. `#math/probability`)
- `#status/placeholder` — scaffolded subject/folder with no real content yet
- `#project/<name>` — applied project notes

## One-line Summary

> This is a living index, not a snapshot — every table above re-queries the vault each time you open it, so it never goes stale the way a hand-written list would.
