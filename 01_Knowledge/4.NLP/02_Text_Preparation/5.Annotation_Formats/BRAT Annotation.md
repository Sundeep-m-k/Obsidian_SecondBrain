# BRAT Annotation

tags: #nlp #annotation #tools #corpus-creation #standoff-annotation
links: [[CoNLL Format]] [[IOB and BIO Tagging]] [[XML and JSON Schemas]] [[NER — Named Entity Recognition]] [[Relation Extraction]]

---

## Definition + Intuition

**BRAT** (BRAT Rapid Annotation Tool) is a web-based annotation platform for creating structured annotations on text — including named entities, relations, events, and coreference chains. It uses a **standoff annotation** format: the original text is never modified; annotations are stored separately and reference the text via character offsets.

> **Intuition**: Imagine annotating a physical document with sticky notes. You never write on the document itself — you place colored tabs that say "characters 42–55 are a PERSON entity" or "this event caused that event." BRAT is the digital version of that — a clean separation between the source text and the annotations layered on top of it.

---

## Key Properties / Types

### Standoff vs Inline Annotation

**Inline annotation** (e.g., XML/SGML): tags embedded in the text
```xml
<text><PER>Barack Obama</PER> visited <LOC>Paris</LOC>.</text>
```
Problem: modifying the text breaks character offsets; nested entities cause malformed XML; hard to diff or version-control.

**Standoff annotation** (BRAT): annotations reference the original text via offsets
```
Original text file (ann.txt):
Barack Obama visited Paris.

Annotation file (ann.ann):
T1  Person 0 12   Barack Obama
T2  Location 21 26  Paris
```
The original text is never touched. Multiple annotation layers can be added independently.

### BRAT File Format

BRAT uses two paired files per document:
- `document.txt` — raw text
- `document.ann` — standoff annotations

**Annotation types in `.ann` files:**

| Prefix | Type | Example |
|--------|------|---------|
| `T` | Text span (entity/chunk) | `T1  Person 0 12  Barack Obama` |
| `R` | Relation (binary, directed) | `R1  employed-by Arg1:T1 Arg2:T3` |
| `E` | Event (with trigger + arguments) | `E1  Arrest:T4 Person:T1` |
| `A` | Attribute (modifies T/E) | `A1  Negated E1` |
| `N` | Normalization (links to KB) | `N1  Reference T1 Wikipedia:Barack_Obama` |
| `#` | Note/comment | `#1  AnnotatorNote T1  uncertain` |
| `*` | Equivalence (coreference) | `* Equiv T1 T5 T9` |

### Configuration (`annotation.conf`)

BRAT is configured per-project via `annotation.conf`:
```ini
[entities]
Person
Organization
Location
GPE

[relations]
employed-by   Arg1:Person, Arg2:Organization
located-in    Arg1:Organization, Arg2:Location

[events]
Arrest        Person:Person, Location?:Location

[attributes]
Negated       Arg:<EVENT>
Speculated    Arg:<EVENT>
```

This defines the ontology for a task. Annotators can only use entity/relation types listed here.

---

## Math / Formal Notation

**Character offset addressing:**

A text span annotation $T_i$ is defined as:
$$T_i = (\text{type},\ \text{start},\ \text{end},\ \text{surface})$$

where $[\text{start}, \text{end})$ is the half-open character interval in the source text.

The surface form is derived: $\text{surface} = \text{text}[\text{start}:\text{end}]$

**Relation annotation:**
$$R_i = (\text{rel\_type},\ T_{\text{arg1}},\ T_{\text{arg2}})$$

A directed binary relation between two text spans.

**Event annotation (more complex):**
$$E_i = (\text{event\_type},\ T_{\text{trigger}},\ [(T_{j_1}, \text{role}_1), (T_{j_2}, \text{role}_2), \ldots])$$

An event has a trigger span + a set of (argument, role) pairs.

**Inter-annotator agreement (IAA)** — measuring annotation quality:

$$\text{Cohen's } \kappa = \frac{P_o - P_e}{1 - P_e}$$

where $P_o$ = observed agreement, $P_e$ = expected agreement by chance.

For span-level tasks, **F1-based IAA** is more common:
$$\text{IAA} = \frac{2 \cdot |A_1 \cap A_2|}{|A_1| + |A_2|}$$

where $A_1$, $A_2$ are the annotation sets of two annotators (treating each other as "gold").

---

## Examples (Concrete)

### Full BRAT Annotation Example

**Source text** (`biomedical.txt`):
```
IL-2 gene expression and NF-kB activation through CD28 requires 
reactive oxygen production by 5-lipoxygenase.
```

**Annotation file** (`biomedical.ann`):
```
T1  Gene 0 4    IL-2
T2  Gene 20 25  NF-kB
T3  Protein 42 46  CD28
T4  Protein 82 96  5-lipoxygenase
T5  Process 5 20   gene expression
T6  Process 26 36  activation
E1  Gene_expression:T5 Theme:T1
E2  Activation:T6 Theme:T2 Cause:T3
R1  produces Arg1:T3 Arg2:T4
A1  Speculated E1
```

This encodes:
- 4 named entities (genes/proteins)
- 2 biological processes (events) with arguments
- 1 binary relation
- 1 attribute (speculated event)

### BRAT → Training Data Pipeline

```
BRAT .ann files
    ↓
Parse standoff annotations
    ↓
Align with token boundaries       ← tricky: char offsets ↔ token offsets
    ↓
Convert to IOB/BIO tags           ← for sequence labeling
    or
Convert to (span_start, span_end, label) triples   ← for span-based models
    ↓
Train NER / RE / Event model
```

**Tools**: `brateval`, `standoff2conll`, or custom parsers.

---

## How It Connects to ML / NLP

| BRAT Concept | NLP/ML Application |
|---|---|
| Standoff format | Preserves original text; enables multiple annotation layers |
| Entity spans | Training data for NER, EL |
| Relations | Training data for RE, dependency-like tasks |
| Events | Training data for event extraction, SRL |
| Attributes | Negation/speculation detection |
| IAA metrics | Data quality assessment; annotation guideline refinement |
| `annotation.conf` | Defines the label ontology for a task |

**Why BRAT is widely used in biomedical NLP:**
Biomedical NLP (BioNLP shared tasks, CRAFT corpus, n2c2 challenges) relies heavily on BRAT because:
1. Biomedical entities overlap and nest (gene inside protein complex)
2. Events have complex argument structures
3. Standoff format handles all of these cleanly
4. Annotators are domain experts (biologists) who need a GUI

**From BRAT to model input — common Python parsing:**
```python
def parse_brat(txt_path, ann_path):
    text = open(txt_path).read()
    entities, relations = [], []
    for line in open(ann_path):
        if line.startswith('T'):
            parts = line.strip().split('\t')
            tid = parts[0]
            info = parts[1].split()
            label, start, end = info[0], int(info[1]), int(info[2])
            surface = parts[2]
            entities.append({'id': tid, 'label': label,
                              'start': start, 'end': end, 'text': surface})
        elif line.startswith('R'):
            parts = line.strip().split('\t')[1].split()
            rel_type = parts[0]
            arg1 = parts[1].split(':')[1]
            arg2 = parts[2].split(':')[1]
            relations.append({'type': rel_type, 'arg1': arg1, 'arg2': arg2})
    return text, entities, relations
```

**Cross-links:**
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — BRAT produces the labeled data
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Labels.md]] — BRAT annotations = the label source
- [[IOB and BIO Tagging]] — convert BRAT spans to IOB for sequence labeling

---

## Common Interview Questions

**Q: What is standoff annotation and why is it preferred over inline annotation for NLP?**
A: Standoff annotation stores annotations separately from the source text, referencing positions via character offsets. Inline annotation (XML/HTML tags) embeds markup directly in the text. Standoff is preferred because: (1) the original text is never modified — essential for character-level model training; (2) overlapping/nested spans are represented naturally; (3) multiple independent annotation layers can be added without conflict; (4) easier to version-control the annotation file separately from the text.

**Q: How do you handle overlapping or nested entities in BRAT annotations when converting to BIO format?**
A: Standard BIO cannot represent overlapping or nested spans — each token gets one label. Solutions: (1) choose the innermost entity for each token (losing outer entity information); (2) use multiple BIO label sequences (one per nesting level); (3) use a span-based model that directly scores (start, end, type) triples, bypassing BIO entirely. In practice, most production NER systems use approach (3) when nested entities matter.

**Q: What is inter-annotator agreement and why does it matter for training data quality?**
A: IAA measures how consistently different human annotators label the same data. Low IAA means the annotation guidelines are ambiguous, the task is inherently subjective, or annotators need more training. IAA is important because: (1) it upper-bounds model performance — if humans agree at 80% F1, a model at 85% might be "superhuman" or might be exploiting annotation artifacts; (2) low-IAA annotations produce noisy training labels that degrade model generalization.

---

## Common Mistakes / Gotchas

- **Off-by-one in character offsets**: BRAT uses half-open intervals `[start, end)`. Some parsers use closed intervals `[start, end]`. Always verify: `text[start:end]` should equal the surface form in the annotation.
- **Whitespace tokenization vs. character offsets**: When converting BRAT to BIO, character offset of the annotation may fall in the middle of a subword token (BPE). Need to handle boundary misalignment — usually by snapping to the nearest token boundary.
- **Forgetting to validate `annotation.conf`**: Invalid entity types or relation arguments not listed in `annotation.conf` silently fail in BRAT. Always validate annotations programmatically after export.
- **Treating BRAT format as universal**: BRAT `.ann` is not the same as CoNLL, IOB, or any other format. Always write an explicit converter for your downstream pipeline.

---

## Further Reading / Paper References

- Stenetorp, P. et al. (2012). *brat: a Web-based Tool for NLP-Assisted Text Annotation.* EACL Demo. — original BRAT paper
- BRAT documentation: https://brat.nlplab.org/standoff.html
- Kim, J.D. et al. (2009). *Overview of BioNLP'09 Shared Task on Event Extraction.* — major BRAT-based biomedical task
- Neves, M. & Seva, J. (2021). *An extensive review of tools for manual annotation of documents.* Briefings in Bioinformatics. — comparison of annotation tools
- Uzuner, O. et al. (2011). *2010 i2b2/VA Challenge on Concepts, Assertions, and Relations in Clinical Text.* JAMIA. — clinical NLP annotation with BRAT
