# 02 — Text Preparation (Index)

tags: #nlp #index #moc #text-preparation #preprocessing
links: [[NLP Index]] [[01_Knowledge/4.NLP/01_Language_Basics/Index]] [[01_Knowledge/4.NLP/03_Text_to_Numbers/Index]]

---

> **Position in vault**: `01_Knowledge/4.NLP/02_Text_Preparation/`
> **Purpose**: Everything that happens to raw text *before* it enters a model. These steps determine data quality, vocabulary coverage, and ultimately model performance. Garbage in → garbage out.
> **Prerequisite**: [[01_Knowledge/4.NLP/01_Language_Basics/Index]] — especially Morphology (for tokenization) and Typology (for CJK/multilingual preprocessing).

---

## Why Text Preparation Matters

Raw text is messy. Web-scraped corpora contain HTML artifacts, encoding errors, duplicate documents, inconsistent casing, non-standard characters, and unresolved contractions. Preprocessing decisions made here propagate through the entire pipeline:

- A bad tokenizer causes vocabulary fragmentation and poor subword coverage
- Skipping deduplication means a model memorizes repeated training examples
- Wrong encoding handling corrupts ~5% of multilingual text silently
- Over-aggressive normalization strips linguistically meaningful distinctions

> **Rule of thumb**: The quality of your preprocessing is often more impactful than the choice of model architecture — especially for low-resource settings.

---

## Section Map

### 🧹 1. Cleaning — Removing Noise
*Make the text safe and consistent before any linguistic processing.*

| Note | Key Concepts | When You Need It |
|------|-------------|-----------------|
| [[Noise Removal]] | Special chars, control chars, regex cleaning, pipeline design | Any raw corpus |
| [[Encoding and Unicode]] | UTF-8, code points, normalization forms (NFC/NFD), BOM, mojibake | Multilingual corpora; web scrapes |
| [[HTML and Markup Stripping]] | BeautifulSoup, trafilatura, boilerplate removal, tag vs text | Web-scraped data |
| [[Deduplication]] | Exact dedup, MinHash LSH, SimHash, near-dedup, memorization | LLM pretraining; any large corpus |

**Key insight**: Deduplication is the single most impactful preprocessing step for large-scale LM pretraining. The Pile, C4, and RedPajama all deduplicate aggressively. A model trained on 30% duplicate data memorizes those examples rather than learning generalizable patterns.

---

### 🔧 2. Normalization — Standardizing Form
*Collapse equivalent surface forms to reduce vocabulary and improve generalization.*

| Note | Key Concepts | When You Need It |
|------|-------------|-----------------|
| [[Lowercasing]] | Case folding, truecasing, when NOT to lowercase | Classification, IR |
| [[Accent Folding]] | Unicode decomposition, diacritic removal, language sensitivity | Cross-lingual IR, noisy social media |
| [[Contraction Expansion]] | "can't"→"cannot", lookup tables, language-specific rules | Formal NLP tasks, preprocessing for parsing |
| [[Spelling Correction]] | Edit distance, noisy channel model, context-sensitive correction, neural | Social media, OCR output, user queries |

**Key insight**: Normalization trades precision for recall. Lowercasing improves recall ("Apple" matches "apple") but loses precision (can't distinguish the company from the fruit). Always ask: does this downstream task need case/accent distinctions?

---

### ✂️ 3. Tokenization — Text to Token Sequences
*The single most consequential preprocessing decision for neural NLP.*

| Note | Key Concepts | When You Need It |
|------|-------------|-----------------|
| [[Whitespace Tokenization]] | Split on spaces/punctuation, NLTK word_tokenize, limitations | Baselines; English-only simple tasks |
| [[Rule-Based Tokenizers]] | Regex rules, MosesTokenizer, language-specific rules, apostrophes | MT preprocessing; linguistics research |
| [[Tokenization — BPE]] | Byte Pair Encoding, merge rules, vocabulary size, BPE-dropout | GPT-2, GPT-3, RoBERTa, most LLMs |
| [[WordPiece and Unigram LM]] | WordPiece (BERT), Unigram language model, EM training | BERT, ALBERT, XLNet, T5 |
| [[SentencePiece]] | Language-agnostic, BPE/Unigram on raw text, no pre-tokenization | Multilingual models, T5, mBART, LLaMA |
| [[Tokenization for CJK Languages]] | Character-based Chinese/Japanese, Jieba, MeCab, morphological segmenters | Any East Asian language NLP |

**Key insight**: The tokenizer is the contract between your text and your model. Changing the tokenizer changes the token count per sentence, the effective context length, the vocabulary size, and the OOV rate — all at once. It cannot be changed without retraining the model.

**Tokenizer comparison at a glance:**

| Tokenizer | Used by | Algorithm | Pre-tokenization |
|-----------|---------|-----------|-----------------|
| BPE | GPT-2, RoBERTa, GPT-3/4 | Frequency-based merges | Yes (whitespace) |
| WordPiece | BERT, DistilBERT | Likelihood-based merges | Yes (whitespace) |
| Unigram LM | ALBERT, T5 (via SP) | EM on unigram LM | Via SentencePiece |
| SentencePiece | T5, LLaMA, mBART | BPE or Unigram, no pre-tok | No |
| Byte-level BPE | GPT-2, RoBERTa (byte) | BPE on raw bytes | No |

---

### 📄 4. Segmentation — Finding Boundaries
*Before tokenizing words, you need to find sentences — and sometimes paragraphs.*

| Note | Key Concepts | When You Need It |
|------|-------------|-----------------|
| [[Sentence Boundary Detection]] | Punkt algorithm, spaCy sentencizer, abbreviation disambiguation | Any sentence-level task: NLI, MT, summarization |
| [[Paragraph Segmentation]] | Blank lines, indentation, topic segmentation, TextTiling | Document understanding, RAG chunking |

**Key insight**: Poor sentence boundary detection silently corrupts sequence pairs. "Dr. Smith said the results were good." — incorrectly split after "Dr." creates `["Dr."]` and `["Smith said the results were good."]` — the first "sentence" is a truncated fragment. Especially problematic in legal, medical, and scientific text with many abbreviations.

---

### 🏷️ 5. Annotation Formats — Storing Structured Labels
*How labeled data is represented on disk — the interface between human annotation and model training.*

| Note | Key Concepts | When You Need It |
|------|-------------|-----------------|
| [[CoNLL Format]] | Tab-separated, one token per line, blank-line sentence separator | NER, POS, parsing training data |
| [[IOB and BIO Tagging]] | B/I/O tags, BIOES, valid transitions, CRF decoding, span extraction | Any sequence labeling task |
| [[BRAT Annotation]] | Standoff format, `.ann` files, character offsets, IAA | Creating NER/RE/event training data |
| [[XML and JSON Schemas]] | SQuAD JSON, JSONL, CoNLL-U, TEI, schema validation | Dataset loading, API design, large corpora |

**Key insight**: Your annotation format determines what information you can represent. Standard BIO cannot represent nested entities. CoNLL cannot represent relations. BRAT can represent almost anything but requires an explicit converter to every downstream format. Choose your format based on task requirements, not habit.

---

## Decision Guide: Which Preprocessing Steps to Apply

```
Raw text corpus
      │
      ▼
Is it web-scraped? ──yes──► HTML stripping → Boilerplate removal
      │
      ▼
Is it multilingual or non-UTF-8? ──yes──► Encoding normalization (Unicode NFC)
      │
      ▼
Is it large-scale (>1M docs)? ──yes──► Deduplication (MinHash LSH)
      │
      ▼
Is the downstream task case-sensitive? ──yes──► Skip lowercasing
      │                                ──no───► Lowercase
      ▼
Is it for a neural model? ──yes──► BPE/WordPiece/SentencePiece tokenizer
                          ──no───► Rule-based tokenizer + stemming/lemmatization
      │
      ▼
Is it non-English? ──CJK──► Character segmentation + morphological analyzer
                 ──agglutinative──► SentencePiece (language-agnostic)
                 ──other──► SentencePiece or language-specific tokenizer
```

---

## Key Cross-Links to `3.ML & DL/`

| Preprocessing Concept | Links to ML Concept |
|----------------------|-------------------|
| Deduplication → reduces memorization | [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] |
| Tokenization → vocabulary = feature space | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Vector.md]] |
| Normalization → reduces vocabulary size | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Scaling.md]] |
| BPE merge rules | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] (via EM/counting) |
| IOB tagging → classification target | [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] |
| Annotation quality → training data | [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] |
| Spelling correction → noisy channel | [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] |

---

## Common Exam / Interview Questions from This Section

1. What is the difference between BPE and WordPiece tokenization? Which does BERT use?
2. How does SentencePiece differ from BPE? Why is it preferred for multilingual models?
3. What is byte-level BPE and why is it used in GPT-2?
4. Why is deduplication important for LLM pretraining? What method does the Pile use?
5. What is mojibake? How do you detect and fix encoding errors in a multilingual corpus?
6. What is the difference between BIO and BIOES tagging schemes? When would you prefer BIOES?
7. How does the Punkt algorithm detect sentence boundaries?
8. What is standoff annotation? Why is it preferred over inline annotation for NLP tasks?
9. When should you NOT lowercase text during preprocessing?
10. How do you handle tokenization for Chinese text (no whitespace)?
11. What is a tokenizer alignment problem in BERT-based NER, and how do you solve it?
12. Why does changing the tokenizer require retraining the model from scratch?

---

## Common Preprocessing Mistakes

| Mistake | Consequence | Fix |
|---------|-------------|-----|
| Skip deduplication on pretraining data | Model memorizes duplicate text; inflated benchmark scores | MinHash LSH dedup |
| Use wrong encoding | Silent corruption of 5–15% of multilingual text | Enforce UTF-8; use `chardet` |
| Lowercase before NER | Loses capitalization signal (proper nouns) | Never lowercase before NER/POS |
| Same tokenizer for all languages | Bad coverage for agglutinative/CJK | Use SentencePiece or language-specific |
| Tokenize before sentence splitting | Sentence boundaries cross document | Always sentence-split before tokenizing |
| Strip all punctuation | Loses sentence boundaries, quotes, important punctuation | Use selective noise removal |

---

## Reading Order (Recommended)

**Fast track** (most commonly needed):
BPE → WordPiece and Unigram LM → SentencePiece → Noise Removal → Encoding and Unicode → Deduplication → BIO Tagging

**Complete track**:
Follow sub-folder order: Cleaning (1) → Normalization (2) → Tokenization (3) → Segmentation (4) → Annotation Formats (5)

**Data engineering track**:
Encoding and Unicode → Deduplication → HTML Stripping → JSONL/XML → CoNLL Format

---

*Next section: [[01_Knowledge/4.NLP/03_Text_to_Numbers/Index]] — Sparse representations, embeddings, feature engineering*
*Previous section: [[01_Knowledge/4.NLP/01_Language_Basics/Index]]*
