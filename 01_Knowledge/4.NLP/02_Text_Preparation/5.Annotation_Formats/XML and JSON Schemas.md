# XML and JSON Schemas for NLP

tags: #nlp #annotation #data-formats #xml #json #corpus
links: [[CoNLL Format]] [[BRAT Annotation]] [[IOB and BIO Tagging]] [[Dataset Cards]]

---

## Definition + Intuition

XML and JSON are two of the most common structured data formats used to store NLP annotations, datasets, and model inputs/outputs. They differ in verbosity, readability, and ecosystem fit:

- **XML** (eXtensible Markup Language): hierarchical, tag-based, verbose, supports attributes and namespaces; dominant in older NLP corpora and linguistic annotation (TEI, GATE, XCES).
- **JSON** (JavaScript Object Notation): key-value pairs and arrays, lightweight, human-readable, dominant in modern NLP APIs and datasets (Hugging Face, SQuAD, CoNLL-U JSON).

> **Intuition**: XML is like a legal contract — very precise, deeply structured, and a bit verbose. JSON is like a text message — concise, easy to read, and modern APIs speak it natively. Both encode the same information; which you use depends on the ecosystem, tooling, and age of the project.

---

## Key Properties / Types

### XML in NLP

**Inline annotation** — markup embedded in text:
```xml
<document id="doc001">
  <sentence id="s1">
    <token id="t1" pos="NNP" ner="B-PER">Barack</token>
    <token id="t2" pos="NNP" ner="I-PER">Obama</token>
    <token id="t3" pos="VBD" ner="O">visited</token>
    <token id="t4" pos="NNP" ner="B-LOC">Paris</token>
  </sentence>
</document>
```

**Key XML features for NLP:**
- **Attributes** (`pos="NNP"`): per-token metadata
- **Nesting**: natural representation of syntax trees, discourse structure
- **Namespaces**: multiple annotation layers in one file (`<tok:token>`, `<dep:edge>`)
- **DTD/Schema validation**: enforce annotation structure correctness

**Major XML-based NLP formats:**
| Format | Use |
|--------|-----|
| **TEI (Text Encoding Initiative)** | Literary/historical corpus annotation |
| **GATE XML** | General NLP pipeline output |
| **XCES** | Corpus encoding standard |
| **Tiger XML** | German treebank format |
| **PAULA XML** | Multi-layer linguistic annotation |
| **FoLiA** | Dutch NLP; multi-layer |

### JSON in NLP

**SQuAD format** (canonical QA dataset structure):
```json
{
  "data": [{
    "title": "Super Bowl 50",
    "paragraphs": [{
      "context": "Super Bowl 50 was an American football game...",
      "qas": [{
        "id": "56be4db0acb8001400a502ec",
        "question": "Which NFL team represented the AFC?",
        "answers": [{
          "text": "Denver Broncos",
          "answer_start": 177
        }],
        "is_impossible": false
      }]
    }]
  }]
}
```

**CoNLL-U JSON** (Universal Dependencies in JSON):
```json
{
  "sent_id": "en-ud-train-00001",
  "text": "They buy and sell books.",
  "tokens": [
    {"id": 1, "form": "They",  "lemma": "they", "upos": "PRON",
     "feats": {"Case": "Nom", "Number": "Plur"}, "head": 2, "deprel": "nsubj"},
    {"id": 2, "form": "buy",   "lemma": "buy",  "upos": "VERB",
     "head": 0, "deprel": "root"},
    ...
  ]
}
```

**Hugging Face `datasets` format** (JSONL — one JSON object per line):
```jsonl
{"text": "I love this movie!", "label": 1}
{"text": "Terrible film, waste of time.", "label": 0}
{"text": "Decent but not memorable.", "label": 2}
```

JSONL (newline-delimited JSON) is the de facto standard for large NLP datasets — each line is independently parseable, enabling streaming and parallel processing.

### JSONL vs JSON Array

```
JSON array (all in memory):
[{"text": "...", "label": 0}, {"text": "...", "label": 1}, ...]
→ Must load entire file to start processing
→ Bad for large datasets (100GB+)

JSONL (streamable):
{"text": "...", "label": 0}
{"text": "...", "label": 1}
...
→ Process line by line; never load full dataset
→ Used by: Hugging Face, CommonCrawl, The Pile
```

---

## Math / Formal Notation

**Schema validation** — checking an annotation file against a schema:

For JSON, **JSON Schema** defines the expected structure:
```json
{
  "$schema": "http://json-schema.org/draft-07/schema",
  "type": "object",
  "required": ["text", "entities"],
  "properties": {
    "text": {"type": "string"},
    "entities": {
      "type": "array",
      "items": {
        "type": "object",
        "required": ["start", "end", "label"],
        "properties": {
          "start": {"type": "integer", "minimum": 0},
          "end":   {"type": "integer", "minimum": 1},
          "label": {"type": "string", "enum": ["PER", "LOC", "ORG"]}
        }
      }
    }
  }
}
```

For XML, **DTD** or **XML Schema (XSD)** serves the same purpose.

**Span representation in JSON — two conventions:**

Character offsets (inclusive start, exclusive end):
$$\text{span} = \{\text{"start"}: i,\ \text{"end"}: j\}\ \text{ where text}[i:j] = \text{surface}$$

Token offsets (index into token list):
$$\text{span} = \{\text{"token\_start"}: i,\ \text{"token\_end"}: j\}$$

Character offsets are more general (model-agnostic); token offsets are faster for training.

---

## Examples (Concrete)

### NER Dataset in JSONL (spaCy-style)
```jsonl
{"text": "Apple is looking at buying U.K. startup for $1 billion",
 "entities": [
   {"start": 0,  "end": 5,  "label": "ORG"},
   {"start": 27, "end": 31, "label": "GPE"},
   {"start": 44, "end": 54, "label": "MONEY"}
 ]}
{"text": "San Francisco considers banning sidewalk delivery robots",
 "entities": [
   {"start": 0, "end": 13, "label": "GPE"}
 ]}
```

### Annotation Comparison: Same Data, Three Formats

**CoNLL (tab-separated)**:
```
Barack  NNP  B-PER
Obama   NNP  I-PER
visited VBD  O
Paris   NNP  B-LOC
```

**XML (inline)**:
```xml
<s><PER>Barack Obama</PER> visited <LOC>Paris</LOC></s>
```

**JSON (standoff)**:
```json
{"text": "Barack Obama visited Paris",
 "entities": [{"start": 0, "end": 12, "label": "PER"},
              {"start": 21, "end": 26, "label": "LOC"}]}
```

All three encode the same information — format choice depends on tooling.

### Loading Formats in Python

```python
# JSONL (Hugging Face style)
import json
with open("data.jsonl") as f:
    for line in f:
        example = json.loads(line)

# Hugging Face datasets (handles many formats automatically)
from datasets import load_dataset
ds = load_dataset("conll2003")
ds = load_dataset("json", data_files="data.jsonl")

# XML with lxml
from lxml import etree
tree = etree.parse("corpus.xml")
for token in tree.findall(".//token"):
    print(token.text, token.get("pos"), token.get("ner"))
```

---

## How It Connects to ML / NLP

| Format Concept | ML/NLP Application |
|---|---|
| JSONL streaming | Large dataset training (The Pile, C4, RedPajama) |
| SQuAD JSON schema | Standard QA benchmark format; model input format |
| Character offsets | Span-based NER/RE model targets |
| JSON Schema validation | Dataset quality assurance |
| XML treebanks | Penn Treebank, Tiger (German), historical corpora |
| JSONL sharding | Distributed training data pipelines |

**Why Hugging Face standardized on JSONL:**
1. Streamable — can iterate a 1TB dataset without loading it
2. Shardable — split into N files for parallel preprocessing
3. Language-agnostic — any language can read JSON
4. Self-describing — column names visible in every line

**Cross-links:**
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — these formats are how training data is stored
- [[3.ML & DL/1.Concepts/6.Features and Representation/Data Representation.md]] — JSON/XML are the serialization layer

---

## Common Interview Questions

**Q: Why is JSONL preferred over JSON arrays for large NLP datasets?**
A: JSONL (newline-delimited JSON) is streamable — each line is a complete JSON object that can be parsed independently. This means you can process a 500GB dataset with constant memory by reading one line at a time. A JSON array requires the entire file to be loaded into memory before any processing can begin. For datasets like The Pile (825GB), JSONL is the only practical choice.

**Q: What is the difference between character-level and token-level span offsets?**
A: Character offsets reference positions in the raw text string. Token offsets reference positions in a tokenized sequence. Character offsets are model-agnostic (they work regardless of tokenizer), but require offset alignment when training on subword tokens. Token offsets are convenient for sequence models but become invalid if the text is re-tokenized with a different tokenizer. SQuAD uses character offsets; CoNLL-U uses token offsets.

**Q: How do you validate an NLP dataset schema programmatically?**
A: Use JSON Schema validation (`jsonschema` Python library) to enforce that every record has required fields, correct types, and valid label values. For character offset annotations, additionally validate: (1) `0 <= start < end <= len(text)`, (2) `text[start:end]` matches the stored surface form, (3) no two entities have the same span. Automated validation catches annotation errors before training.

---

## Common Mistakes / Gotchas

- **Unicode vs byte offsets**: Python strings are Unicode (character) indexed. Some tools use byte offsets (important for multi-byte characters like Chinese, Arabic, emoji). Always check which convention a dataset uses — mismatched offsets corrupt annotations silently.
- **Trailing whitespace in surface forms**: `text[start:end]` sometimes includes a trailing space if offsets are off by one. Always strip and verify surface forms match the annotation's `text` field.
- **JSON vs JSONL**: loading JSONL with `json.load()` (for JSON arrays) fails because JSONL is not valid JSON. Use `json.loads(line)` in a loop, or use `pandas.read_json(..., lines=True)`.
- **XML namespace clutter**: Large XML corpora with multiple annotation layers have complex namespace declarations. Use `etree.QName(element.tag).localname` to strip namespaces when parsing.

---

## Further Reading / Paper References

- Rajpurkar, P. et al. (2016). *SQuAD: 100,000+ Questions for Machine Comprehension of Text.* [[arxiv:1606.05250]] — canonical JSON NLP dataset format
- Nivre, J. et al. (2020). *Universal Dependencies v2.* [[arxiv:2004.10643]] — CoNLL-U format (tab-separated + JSON-compatible)
- JSON Schema specification: https://json-schema.org
- TEI Consortium. *TEI P5: Guidelines for Electronic Text Encoding and Interchange.* https://tei-c.org/guidelines/p5/
- Hugging Face `datasets` documentation: https://huggingface.co/docs/datasets — JSONL/Arrow data loading
- Loper, E. & Bird, S. (2002). *NLTK: The Natural Language Toolkit.* — XML corpus readers in NLTK
