---
tags: [nlp, feature-engineering, linguistic-features, pos, ner, classical-nlp, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Dimensionality Reduction]] [[Constituency vs Dependency]] [[Semantic Roles]] [[IOB and BIO Tagging]]"
---

# Linguistic Features

## Definition + Intuition

**Linguistic features** are explicitly engineered numeric representations derived from NLP tools and linguistic knowledge — POS tags, dependency labels, named entity types, morphological properties, lexical properties, and corpus statistics. They convert linguistic annotations into input features for classical ML models (Logistic Regression, SVM, CRF) and can also augment neural models.

**Intuition**: Before neural networks, every NLP system was built on carefully crafted features. A named entity recogniser would use: "Is the word capitalised? Is the previous word 'Mr.'? Does the word end in '-burg'? Is it in a gazetteer of city names?" Each of these signals is a linguistic feature. The art of NLP before 2013 was designing the right feature template.

**Why still relevant**: Feature engineering is not obsolete. In low-resource settings (few labeled examples), neural models often overfit while feature-based models generalise. Hybrid systems (neural + explicit features) frequently outperform neural-only on structured tasks (relation extraction, clinical NLP, legal NLP). Understanding features is also essential for debugging and interpreting model errors.

---

## Key Properties / Types

### 1. Lexical Features
Properties derivable from the word itself, without external resources:

| Feature | Example | Captures |
|---------|---------|---------|
| Word form | "Running" | Identity |
| Lowercase form | "running" | Case-normalised identity |
| Prefix ($n$ chars) | "ru", "run" | Morphological prefix pattern |
| Suffix ($n$ chars) | "ng", "ing" | Morphological suffix (often highly predictive for POS) |
| Is all caps | NASDAQ → True | Acronym detection |
| Is title case | "London" → True | Proper noun signal |
| Has digit | "B2B" → True | Code/ID pattern |
| Has hyphen | "state-of-the-art" → True | Compound word |
| Word length | 7 | Short function words vs. long content words |
| Is stopword | "the" → True | Function vs. content word |

### 2. POS Features
Part-of-speech tags assigned by a tagger (spaCy, NLTK, Stanford):

- `NOUN`, `VERB`, `ADJ`, `ADV`, `PRON`, `DET`, `CONJ`, `ADP` (coarse-grained)
- Penn Treebank tagset (fine-grained): `NN`, `NNS`, `VBZ`, `VBD`, `JJ`, `RB`, ...
- Context: POS of previous and next words (window features)

POS is highly predictive for: NER (nouns are more likely entities), SRL (verbs are predicates), parsing (syntactic role assignment).

### 3. Dependency Features
Derived from dependency parsing (spaCy, UDPipe):

| Feature | Example | Use |
|---------|---------|-----|
| Dependency relation | `nsubj`, `dobj`, `amod` | SRL, RE, IE |
| Head word | "ate" is head of "John" (nsubj) | Syntactic head features |
| Children | nominal subjects, objects | Predicate-argument structure |
| Dependency path | `entity → nsubj → verb → dobj → entity` | Relation extraction |
| Depth in tree | How far from root | Syntactic embedding |

### 4. Named Entity Features
Labels from an NER system used as features for downstream tasks:

- Entity type: `PER`, `ORG`, `LOC`, `DATE`, `MONEY`, `PERCENT`, `MISC`
- Entity IOB tag: `B-PER`, `I-PER`, `O`
- In gazetteer: Is this word in a list of known person/city/organisation names?

Entity type is a strong feature for: relation extraction (linking two entities), coreference (matching pronouns to named entities), event extraction (identifying event participants).

### 5. Morphological Features
From a morphological analyser (NLTK, spaCy, stanza):

- Lemma (base form): "running" → "run"
- Number: `Sing`, `Plur`
- Tense: `Past`, `Pres`, `Fut`
- Voice: `Active`, `Passive`
- Case: `Nom`, `Acc`, `Gen` (crucial for inflected languages)
- Valency: number of arguments a verb takes

### 6. Lexical Resource Features
External knowledge bases:

| Resource | Features | Task |
|----------|---------|------|
| WordNet | Synset, hypernym, hyponym, semantic category | WSD, semantic similarity |
| SentiWordNet | Positive/negative/objective score per synset | Sentiment analysis |
| Gazetteers | In-list: cities, countries, person names | NER |
| MPQA lexicon | Subjectivity, polarity, intensity | Opinion mining |
| FrameNet | Frame membership, frame element role | SRL, event extraction |

---

## Math / Formal Notation

**Feature vector construction** for word $w_i$ in sentence:

$$\mathbf{x}_i = [f_1(w_{i-2}), f_2(w_{i-1}), f_3(w_i), f_4(w_{i+1}), f_5(w_{i+2}), \ldots]$$

Each $f_k$ is a feature function returning a real number or a one-hot categorical encoding.

**One-hot encoding of categorical features**:

POS tag vocabulary = $\{$NN, NNS, VB, VBZ, JJ, ...$\}$ (|T| tags):
$$\text{POS\_vec}(t_i) \in \{0,1\}^{|T|}, \quad \text{POS\_vec}(t_i)_j = \mathbf{1}[t_i = j]$$

**Window features** (context is critical for most sequence labelling tasks):

For NER with window $k=2$, the feature vector for token $i$ includes features of tokens $i-2, i-1, i, i+1, i+2$. This creates feature templates like:
- `word[-1]=Mr.` ∧ `word[0]=Smith` → strong B-PER signal
- `pos[0]=NNP` ∧ `word[-1]=in` → LOC signal

**Feature conjunction**: In linear models (CRF, SVM), interaction terms are added explicitly:
$$f_{conj}(w_i, w_{i+1}) = [f(w_i)] \otimes [f(w_{i+1})]$$
This is the "feature template" approach — combinatorial feature space but sparse in practice.

**CRF with linguistic features** (standard baseline for NER):
$$P(\mathbf{y} \mid \mathbf{x}) = \frac{1}{Z(\mathbf{x})} \exp\left(\sum_{i} \sum_k \lambda_k f_k(y_{i-1}, y_i, \mathbf{x}, i)\right)$$

where $f_k$ are feature functions (including all linguistic features above) and $\lambda_k$ are learned weights.

---

## Examples (Concrete)

**NER feature template** (CoNLL-style CRF):

For token "London" in "... travelled to London yesterday":

```
WORD=London
LOWER=london
PREFIX2=Lo
PREFIX3=Lon
SUFFIX2=on
SUFFIX3=don
ISTITLE=True
ISDIGIT=False
POS=NNP
PREV_WORD=to
PREV_POS=IN
NEXT_WORD=yesterday
NEXT_POS=NN
IN_CITY_GAZETTEER=True
WORDSHAPE=Xxxxx  (capital + lowercase pattern)
```

From this feature vector, the CRF learns weights like: `IN_CITY_GAZETTEER=True ∧ POS=NNP ∧ PREV_POS=IN` → strong signal for `B-LOC`.

**Relation extraction features** (SVM with dependency path):
```python
import spacy
nlp = spacy.load("en_core_web_sm")

sentence = "Apple acquired Beats Electronics in 2014."
doc = nlp(sentence)

# Feature: shortest dependency path between two entities
def dep_path(doc, ent1_head, ent2_head):
    # Traverse dependency tree to find path between entities
    # "Apple → acquired (nsubj), acquired → Beats (dobj)" → [nsubj, dobj]
    pass

# Feature: entity types
print([(ent.text, ent.label_) for ent in doc.ents])
# [('Apple', 'ORG'), ('Beats Electronics', 'ORG'), ('2014', 'DATE')]

# Feature conjunction: (ORG, nsubj→verb→dobj, ORG) → strong signal for ACQUIRED_BY
```

---

## How It Connects to ML / NLP

| Linguistic Feature Concept | ML/NLP Link |
|---------------------------|-------------|
| Feature template = feature function | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Engineering.md]] — linguistic features are the NLP instance of domain-driven feature design |
| One-hot POS encoding | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Vector.md]] — categorical features become binary indicator vectors |
| Window features = context | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Multiple Features.md]] — concatenating multiple word features creates a wide feature vector |
| CRF with linguistic features | [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — CRF is a discriminative classifier over sequences |
| L1 regularisation on CRF weights | [[3.ML & DL/1.Concepts/9.Regularization/L1 Regularization.md]] — L1 induces sparsity; only informative features have non-zero weights |
| Feature selection for high-dim features | [[3.ML & DL/1.Concepts/21.Meta & Interview Revision/How to Choose Features.md]] — which features to include is a core ML decision |
| Gazetteer = prior knowledge | [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — gazetteers inject external knowledge without labelled examples |
| Overfitting on feature conjunctions | [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — combinatorial feature templates can overfit on small datasets |

---

## Common Interview Questions

**Q: Why are linguistic features still used when neural models exist?**
A: (1) **Low-resource settings**: With <1000 labeled examples, neural models often overfit while feature-based CRFs generalise (fewer parameters). (2) **Interpretability**: A CRF's learned weights tell you exactly which features fire for each prediction — critical for debugging and regulatory requirements. (3) **Hybrid augmentation**: Adding POS or entity type features to BERT inputs consistently improves performance on structured tasks. (4) **Speed**: Feature-based models are orders of magnitude faster at inference than Transformers.

**Q: What is a feature template?**
A: A feature template is a pattern that generates features from a token and its context. Example: `WORD[-1]_POS[0]` generates a feature for every combination of previous word and current POS tag seen in training. Feature templates define the feature space — the set of all possible features. CRF systems like CRFsuite and CRFPP use feature templates directly.

**Q: What features are most important for NER?**
A: Empirically (pre-neural): (1) Capitalisation/title case — strongest single feature for English NER. (2) Gazetteer membership — known entity names. (3) Context POS tags — NNP (proper noun) is a strong entity signal. (4) Suffixes — `-burg`, `-ville` suggest locations; `-son`, `-sen` suggest person names. (5) Dependency relation — entity is `nsubj` or `dobj`. Modern neural NER replaces most of these with BERT contextual representations.

**Q: What is a word shape feature?**
A: Word shape maps a word to a pattern representing its character types: "London" → "Xxxxx" (capital followed by lowercase), "B2B" → "X0X", "1990" → "0000". Shape features are compact representations of capitalisation and digit patterns that capture entity-type regularities (all-caps → acronym, title-case → proper noun).

---

## Common Mistakes / Gotchas

- **Using gold POS/NER tags as features**: In experiments, it's tempting to use gold standard annotations as features. At test time, you only have predicted annotations (with errors). Always use predicted tags as features to avoid train/test mismatch.
- **Ignoring feature normalisation**: Binary features (0/1) and continuous features (word frequency, sentence length) have very different scales. Normalise continuous features before feeding to SVM/LR.
- **Forgetting negation in sentiment features**: A lexicon-based sentiment feature "contains_positive_word=True" fires for "not good" — but the sentiment is negative. Negation-scoping features (`is_under_negation`) are essential for accurate sentiment.
- **Over-engineering features before establishing baselines**: Always start with simple features (word identity, POS) and add complexity only if needed. Complex feature conjunctions often provide marginal gain but increase overfitting risk.
- **Applying English-centric features to multilingual text**: Capitalisation features are useless for languages without case (Arabic, Chinese, Hebrew). Suffix features for morphology are weak for agglutinative languages where suffixes are long and regular. Always adapt feature templates to the target language.

---

## Further Reading / Paper References

- Lafferty, McCallum & Pereira (2001). Conditional Random Fields: Probabilistic Models for Segmenting and Labeling Sequence Data — the CRF paper that uses linguistic features
- Finkel, Grenager & Manning (2005). Incorporating Non-local Information into Information Extraction Systems by Gibbs Sampling — Stanford NER with extensive features
- Zhou & Su (2002). Named Entity Recognition Using an HMM-based Chunk Tagger — classic feature engineering for NER
- Manning et al. (2014). The Stanford CoreNLP Natural Language Processing Toolkit — gold standard feature-based NLP pipeline
- Jurafsky & Martin, SLP Ch. 8 — Sequence Labeling for Parts of Speech and NER, feature-based approaches
