# 01 — Language Basics (Index)

tags: #nlp #index #moc #language-basics
links: [[NLP Index]] [[01_Knowledge/4.NLP/02_Text_Preparation/Index]] [[01_Knowledge/4.NLP/03_Text_to_Numbers/Index]]

---

> **Position in vault**: `01_Knowledge/4.NLP/01_Language_Basics/`
> **Purpose**: Linguistic foundations that every NLP system rests on. Understanding these concepts explains *why* NLP is hard, *why* certain design choices are made, and *where* models fail.
> **Prerequisite**: None — this is the entry point for the NLP track.

---

## Why Language Basics Matter for NLP

NLP systems operate on human language. Language has structure at every level — sounds, word forms, sentence structure, meaning, and context. Each level creates both challenges and opportunities for machine learning:

| Level | What It Is | Why It Matters for NLP |
|-------|-----------|----------------------|
| Phonology | Sound system | ASR, TTS, G2P — speech NLP |
| Morphology | Word formation | Tokenization, lemmatization, multilingual NLP |
| Syntax | Sentence structure | Parsing, dependency features, SRL |
| Semantics | Meaning | Word embeddings, NLI, QA, semantic parsing |
| Pragmatics | Language in context | Dialogue, intent detection, hallucination |
| Typology | Language diversity | Cross-lingual NLP, low-resource systems |

---

## Section Map

### 🔊 1. Phonology — Sound System
*Relevant for: ASR, TTS, G2P, multilingual speech NLP*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Phonemes and Allophones]] | Phoneme inventory, minimal pairs, distinctive features | Tokenization, Speech models |
| [[Prosody and Stress]] | F0, intonation, stress, tonal languages, ToBI | TTS, Sentiment, Emotion detection |
| [[Speech Sounds IPA]] | IPA, articulation, mel spectrograms, MFCCs | ASR, TTS, G2P models |

**Key insight**: A phoneme inventory is a finite vocabulary — the same concept as a token vocabulary. Understanding this explains why ASR is a sequence labeling problem over a fixed phoneme set.

---

### 🔤 2. Morphology — Word Formation
*Relevant for: tokenization, lemmatization, multilingual NLP, agglutinative language processing*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Morphemes — Free and Bound]] | Free/bound morphemes, roots, affixes, productivity | BPE motivation, vocabulary |
| [[Inflection vs Derivation]] | Inflectional vs derivational morphology, paradigms | Lemmatization, morphological tagging |
| [[Stemming vs Lemmatization]] | Porter stemmer, WordNet lemmatizer, when to use which | Text preprocessing, IR |
| [[Agglutinative Languages]] | Turkish, Finnish, vocabulary explosion, OOV | Subword tokenization, cross-lingual NLP |

**Key insight**: BPE tokenization approximates morpheme segmentation. Understanding morphology explains why BPE works and where it fails (irregular forms, agglutinative languages).

---

### 🌳 3. Syntax — Sentence Structure
*Relevant for: parsing, dependency features, SRL, coreference, NLI*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Constituency vs Dependency]] | Phrase structure vs head-dependent relations, UD | Parsing tasks, spaCy, IE |
| [[Parse Trees]] | CYK algorithm, PCFG probabilities, treebanks | Statistical parsing, Tree-LSTM |
| [[CFG and PCFG]] | Grammar rules, inside-outside, MLE from treebank | Probabilistic modeling, EM |
| [[Grammar Formalisms]] | Chomsky hierarchy, CCG, LFG, TAG | Theoretical NLP, semantic parsing |
| [[Head-Driven Phrase Structure]] | HPSG, feature structures, unification, ERG | Deep NLP, grammar engineering |

**Key insight**: Dependency parsing is directly applicable to modern NLP pipelines (spaCy). Constituency parsing is important for syntax-aware models and evaluation. Neural parsers replaced handcrafted grammars but output the same representations.

---

### 💡 4. Semantics — Meaning
*Relevant for: word embeddings, NLI, QA, semantic parsing, WSD, SRL*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Lexical Semantics]] | Synonymy, hyponymy, distributional hypothesis, PMI | Word2Vec, GloVe, WordNet |
| [[Compositional Semantics]] | Frege's principle, lambda calculus, AMR, vector composition | Semantic parsing, Tree-LSTM |
| [[Semantic Roles]] | Agent, patient, PropBank, Neo-Davidsonian semantics | SRL, IE, QA |
| [[Word Sense and Polysemy]] | WSD, Lesk algorithm, sense embeddings, Winograd | BERT contextual embeddings, MT |
| [[Frame Semantics]] | Frames, frame elements, FrameNet, profiling | SRL, event extraction, media analysis |

**Key insight**: The distributional hypothesis (words with similar meaning appear in similar contexts) is the theoretical justification for the entire word embedding paradigm. Polysemy explains why static embeddings (Word2Vec) are limited and why BERT's contextual representations are powerful.

---

### 💬 5. Pragmatics — Language in Context
*Relevant for: dialogue systems, intent detection, hallucination analysis, NLI*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Speech Acts]] | Illocutionary acts, directive/assertive/commissive, RSA | Intent detection, dialogue acts |
| [[Implicature and Grice]] | Gricean maxims, scalar implicature, RSA model | NLI, hallucination, sarcasm |
| [[Discourse Structure]] | RST, coherence relations, centering theory | Summarization, discourse parsing |
| [[Coreference]] | Coreference chains, pronoun resolution, Winograd | Coreference resolution, IE, QA |

**Key insight**: LLM hallucinations are violations of Grice's Quality maxim. Understanding pragmatics provides a theoretical framework for analyzing when and why language models fail to communicate coherently.

---

### 🌍 6. Typology — Language Diversity
*Relevant for: multilingual NLP, cross-lingual transfer, tokenization design, low-resource NLP*

| Note | Key Concepts | Connects To |
|------|-------------|-------------|
| [[Language Families]] | Indo-European, Sino-Tibetan, Afro-Asiatic, resource distribution | Cross-lingual transfer |
| [[Morphological Typology]] | Isolating, agglutinative, fusional, polysynthetic | Tokenization, mBERT, XLM-R |
| [[Low-Resource Languages]] | Resource spectrum, transfer learning, Masakhane | Zero-shot, few-shot NLP |

**Key insight**: English is an outlier — morphologically simple, well-resourced, and over-represented in NLP research. Building systems that work for the world's ~7,000 languages requires understanding typological diversity and explicit strategies for knowledge transfer.

---

## Key Cross-Links to `3.ML & DL/`

| Language Basics Concept | Links to ML Concept |
|------------------------|-------------------|
| Phoneme = finite vocabulary | [[3.ML & DL/1.Concepts/1.Foundations/Features.md]] |
| Distributional hypothesis | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] |
| Morphological complexity | [[Feature Engineering]] *(closest existing equivalent — a dedicated "Vocabulary Pruning" note doesn't exist yet)* |
| Parse tree probabilities (PCFG) | [[3.ML & DL/1.Concepts/4.Loss and Cost/Cost Function.md]] |
| Dependency parsing as classification | [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] |
| Cross-lingual generalization | [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] |
| Stemming/lemmatization → sparsity | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] |
| Inside-outside algorithm | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] |

---

## Common Exam / Interview Questions from This Section

1. What is the difference between a phoneme and an allophone? Why does it matter for ASR?
2. How does BPE tokenization approximate morphological segmentation?
3. What is the difference between stemming and lemmatization? When would you use each?
4. What is the difference between constituency and dependency parsing? Which is more useful for practical NLP?
5. Explain the distributional hypothesis. How does it justify word embeddings?
6. What is polysemy? How do static vs contextual embeddings handle it differently?
7. What is the Gricean Cooperative Principle? Name two ways LLMs violate it.
8. What is scalar implicature? How does it affect NLI datasets?
9. What is coreference resolution? What makes Winograd schemas hard?
10. What is the difference between agglutinative and fusional morphology? How does each affect tokenization?
11. Why are most of the world's languages low-resource despite having millions of speakers?
12. What is cross-lingual zero-shot transfer and when does it fail?

---

## Reading Order (Recommended)

**Fast track (most NLP-relevant):**
Morphemes → Stemming vs Lemmatization → Agglutinative Languages → Constituency vs Dependency → Lexical Semantics → Word Sense and Polysemy → Implicature and Grice → Morphological Typology → Low-Resource Languages

**Complete track:**
Follow the section order (1 → 2 → 3 → 4 → 5 → 6) within each sub-folder

**Speech NLP track:**
IPA → Phonemes and Allophones → Prosody and Stress → then → 02_Text_Preparation → Tokenization

---

*Next section: [[01_Knowledge/4.NLP/02_Text_Preparation/Index]] — Cleaning, normalization, and tokenization*
*Previous section: [[00_Start_Here/README]]*
