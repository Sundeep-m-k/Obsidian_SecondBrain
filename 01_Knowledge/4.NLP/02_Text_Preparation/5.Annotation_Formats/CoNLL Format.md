# CoNLL Format

tags: #nlp #annotation #data-format #ner #pos-tagging #sequence-labeling
links: [[IOB and BIO Tagging]] [[BRAT Annotation]] [[XML and JSON Schemas]] [[NER — Named Entity Recognition]] [[POS Tagging]]

---

## Definition + Intuition

**CoNLL format** (Conference on Natural Language Learning) is the standard plain-text column format for sequence labeling tasks in NLP — POS tagging, NER, chunking, and dependency parsing. Each token occupies one row; columns contain the token and its annotations; sentences are separated by blank lines.

> **Intuition**: CoNLL format is the TSV (tab-separated values) of NLP annotations. One word per row, one column per annotation layer, blank line between sentences. It's human-readable, easy to parse, and has been the standard for two decades.

---

## Key Properties / Types

### CoNLL-2003 Format (NER)

The most widely used variant — four columns:

```
WORD        POS   CHUNK  NER
EU          NNP   B-NP   B-ORG
rejects     VBZ   B-VP   O
German      JJ    B-NP   B-MISC
call        NN    I-NP   O
to          TO    B-VP   O
boycott     VB    I-VP   O
British     JJ    B-NP   B-MISC
lamb        NN    I-NP   O
.           .     O      O

Peter       NNP   B-NP   B-PER
Blackburn   NNP   I-NP   I-PER
```

Columns: word | POS tag | Chunk tag (BIO) | NER tag (BIO)

### CoNLL-2009 Format (Dependency Parsing)

Extended format with 10 columns (CoNLL-X / Universal Dependencies):

```
ID  FORM     LEMMA   UPOS   XPOS   FEATS           HEAD  DEPREL  DEPS  MISC
1   John     John    PROPN  NNP    Number=Sing     2     nsubj   _     _
2   went     go      VERB   VBD    Tense=Past|...  0     root    _     _
3   to       to      ADP    IN     _               4     case    _     _
4   school   school  NOUN   NN     Number=Sing     2     obl     _     _
5   .        .       PUNCT  .      _               2     punct   _     SpaceAfter=No
```

Column meanings: index | word | lemma | universal POS | language POS | morphological features | head index | dependency relation | enhanced deps | misc.

### Parsing CoNLL

```python
def read_conll(filepath):
    sentences = []
    current = []
    with open(filepath) as f:
        for line in f:
            line = line.rstrip('\n')
            if line.startswith('-DOCSTART-') or line == '':
                if current:
                    sentences.append(current)
                    current = []
            else:
                parts = line.split()
                current.append(parts)
    if current:
        sentences.append(current)
    return sentences

# sentences[0] = [['EU', 'NNP', 'B-NP', 'B-ORG'], ['rejects', ...], ...]
```

---

## How It Connects to ML / NLP

CoNLL format is the **universal input/output format** for sequence labeling in NLP:
- Training data for NER: CoNLL-2003 (English/German), CoNLL-2002 (Spanish/Dutch)
- Parsing: Universal Dependencies uses a CoNLL-U variant
- Evaluation: `conlleval.pl` is the standard NER evaluation script

**Cross-links:**
- [[IOB and BIO Tagging]] — the tagging scheme used in CoNLL NER labels
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — NER is sequence labeling

---

## Common Interview Questions

**Q: Why use CoNLL format instead of JSON for sequence labeling data?**
A: CoNLL is simpler to parse line by line (no nested structures), natural to inspect manually, and aligns directly with sequence labeling — one token per line mirrors the model's input/output structure. JSON is better for complex hierarchical annotations (coreference, AMR). CoNLL is the standard because the CoNLL shared tasks (2000–2012) established it as the benchmark format.

---

## Further Reading

- CoNLL-2003 NER Shared Task: https://www.clips.uantwerpen.be/conll2003/ner/
- Universal Dependencies: https://universaldependencies.org — CoNLL-U format
