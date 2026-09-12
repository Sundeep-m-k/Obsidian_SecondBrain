# Low-Resource Languages

tags: #nlp #typology #low-resource #multilingual #transfer-learning #data-augmentation
links: [[Language Families]] [[Morphological Typology]] [[Agglutinative Languages]] [[XLM-R]] [[Cross-Lingual Transfer]] [[Data Augmentation]]

---

## Definition + Intuition

A **low-resource language** is one for which NLP tools and labeled training data are scarce — typically fewer than a few hundred thousand sentences of digital text, and little to no annotated data for tasks like NER, parsing, or translation.

The "resource" in low-resource refers to:
- **Data**: parallel corpora, monolingual text, annotated datasets
- **Tools**: tokenizers, morphological analyzers, POS taggers, parsers
- **Models**: pretrained language models, word embeddings

> **Intuition**: Building an NLP system for English is like cooking with a fully stocked kitchen. Building one for Yoruba or Quechua is like cooking with a handful of ingredients, no recipe book, no kitchen equipment, and instructions in a language you barely speak. Everything requires 10× more effort for worse results.

---

## Key Properties / Types

### The Resource Spectrum

```
High-resource:     English, Chinese, German, French, Spanish
                   → Billions of tokens, hundreds of annotated datasets
                   → Full model zoo: BERT, GPT, parsers, NER, MT, ASR

Mid-resource:      Indonesian, Turkish, Polish, Vietnamese, Hindi
                   → Hundreds of millions of tokens
                   → Some annotated data; multilingual models cover these

Low-resource:      Swahili, Yoruba, Haitian Creole, Basque, Welsh
                   → Tens of millions of tokens
                   → Little annotated data; multilingual models give partial coverage

Very low-resource: Most of the 7,000 world languages
                   → < 1 million tokens, often < 100k
                   → No NLP tools; endangered languages
```

### What Makes a Language Low-Resource?

1. **Few native speakers** (absolute size): fewer speakers → less text produced
2. **Digital divide**: speakers exist but little online presence
3. **Script**: unique or undigitized script (some languages lack standard orthography)
4. **Colonial legacy**: dominant languages displaced indigenous ones in formal contexts
5. **Research bias**: most NLP researchers work on their own languages
6. **Chicken-and-egg**: no data → no models → no tools to collect data

### The 7,000 Language Problem
```
~7,000 languages spoken today
~2,000 have writing systems
~500 have Wikipedia pages
~100 have any NLP research
~10 have high-quality large-scale models
```

**The top 10 languages represent < 10% of the world's languages but > 90% of NLP research.**

---

## Math / Formal Notation

### Transfer Learning for Low-Resource NLP

**Zero-shot cross-lingual transfer**: train on high-resource language $L_H$, evaluate on low-resource $L_L$ with no fine-tuning data:

$$\theta^* = \arg\min_\theta \mathcal{L}(\theta; \mathcal{D}_{L_H})$$
$$\text{eval on } \mathcal{D}_{L_L}$$

**Few-shot fine-tuning**: small labeled set $\mathcal{D}_{L_L}^{small}$ ($|D| \approx 10$–$1000$):
$$\theta^{**} = \arg\min_\theta \mathcal{L}(\theta^*; \mathcal{D}_{L_L}^{small})$$

**Expected transfer gap**:
$$\Delta = \text{Acc}(L_H) - \text{Acc}(L_L^{\text{zero-shot}})$$

This gap correlates with:
- Typological distance between $L_H$ and $L_L$
- Script similarity (shared subword units)
- Amount of $L_L$ data in multilingual pretraining

### Data Augmentation for Low-Resource Settings

**Back-translation**: translate $L_L$ text to $L_H$, generate pseudo-parallel data:

$$\tilde{\mathcal{D}} = \{(x_{L_L}, \text{MT}(x_{L_L} \rightarrow L_H))\}$$

This synthetically creates parallel data. MT quality directly limits augmentation quality.

**Lexical substitution**: replace words with synonyms/translations to multiply training examples:

$$P(\tilde{x} \mid x) = \prod_{i} P(\tilde{w}_i \mid w_i, \text{context})$$

**Self-training**: train a model on labeled data, apply to unlabeled data, add high-confidence predictions to training set:

$$\mathcal{D}^{t+1} = \mathcal{D}^t \cup \{(x, \hat{y}) : P(\hat{y} \mid x, \theta^t) > \tau\}$$

This is semi-supervised learning — very common in low-resource NLP.

---

## Examples (Concrete)

### Yoruba — A Canonical Low-Resource Language

Yoruba is spoken by ~45 million people in Nigeria — not small! But NLP resources are scarce:
- **Text**: ~500k sentences of web-scraped text (vs billions for English)
- **Annotated data**: YorùbáTwi Twitter NER dataset (11k tokens), some MT pairs
- **Challenges**: tonal language (4 tones, often unmarked in informal text), complex morphology, code-switching with English

Performance comparison on NER:
```
English (high-resource, fine-tuned):  91 F1
Yoruba  (zero-shot from English):     ~50 F1
Yoruba  (100 Yoruba examples):        ~62 F1
Yoruba  (full Yoruba fine-tune):      ~73 F1
```

The gap reflects resource scarcity, not linguistic difficulty.

### Masakhane — Community-Led African NLP

Masakhane is a grassroots project that created MT models for 38 African languages:
```
Language pairs with Masakhane:
  English ↔ Zulu (isiZulu)
  English ↔ Igbo
  English ↔ Luo
  English ↔ Wolof
  ... (38 total)

Method: community participants translate their own text
Result: first MT models for many of these language pairs
```

### Endangered Language Documentation

Many of the world's ~3,000 endangered languages are being documented with:
- Field recordings (audio)
- Interlinear glosses (morpheme-by-morpheme translations)
- Small text archives

NLP tools like forced alignment, automatic morphological analysis, and speech recognition are being adapted to support language documentation — but very few labeled examples exist (sometimes < 100 sentences).

---

## How It Connects to ML / NLP

| Low-Resource Challenge | NLP/ML Solution |
|----------------------|----------------|
| No labeled training data | Zero-shot transfer; cross-lingual models (mBERT, XLM-R) |
| Small labeled sets | Few-shot learning; meta-learning (MAML) |
| No unlabeled text | Multilingual pretraining; transfer from related languages |
| Morphological complexity | Character/byte-level models; morphological analysis |
| No tokenizer | SentencePiece (learns from raw text); character/byte level |
| No evaluation data | Universal Dependencies; XTREME; AmericasNLP benchmarks |
| Domain mismatch | Cross-domain transfer + low-resource fine-tuning |

### Key Techniques

**1. Multilingual pretraining (mBERT, XLM-R)**
Pretrain on 104 languages simultaneously. Low-resource languages get some representation even without fine-tuning data.

**2. Language-adaptive fine-tuning (LAFT)**
Continue pretraining on low-resource language text before task fine-tuning — improves downstream task performance by ~5–10 F1.

**3. Cross-lingual data projection**
Project annotations from high-resource language via word alignments:
```
English NER: [John]_PER went to [Paris]_LOC
Align EN→FR: [Jean]_PER est allé à [Paris]_LOC
```
Projected NER annotations are noisy but can bootstrap a low-resource NER system.

**4. Universal Dependencies for parsing**
UD provides a consistent annotation scheme across 100+ languages — allowing parsers trained on high-resource languages to transfer to low-resource ones.

**5. Active learning**
Select the most informative examples to annotate — get maximum value from limited annotation budget:
$$x^* = \arg\max_{x \in \mathcal{U}} H(P(y \mid x, \theta))$$
Label the examples with highest prediction entropy (most uncertain). Related to [[3.ML & DL/1.Concepts/12.Reinforcement Learning/Exploration vs Exploitation.md]].

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — generalizing with limited data
- [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — preventing overfitting on tiny labeled sets
- [[3.ML & DL/1.Concepts/1.Foundations/Training Data.md]] — data scarcity is the core challenge

---

## Common Interview Questions

**Q: What is the difference between zero-shot and few-shot cross-lingual transfer?**
A: Zero-shot transfer trains a model on a high-resource language (or multilingual data) and directly evaluates on a target language without any target-language fine-tuning. Few-shot transfer uses a small number ($k$ = 10–1000) of labeled examples in the target language for fine-tuning. Zero-shot is useful when no annotation budget exists; few-shot is usually significantly better and should be used if even small annotation is possible.

**Q: Why don't large multilingual models solve the low-resource problem?**
A: Three reasons: (1) **Curse of multilinguality** — adding more languages to a fixed-capacity model dilutes capacity per language; low-resource languages get little representation. (2) **Data imbalance** — even with oversampling, low-resource languages contribute far fewer tokens to pretraining, so the model learns less about them. (3) **Typological distance** — if the low-resource language is typologically distant from all high-resource languages in training (e.g., a polysynthetic Amerindian language), transfer is minimal.

**Q: What is the Masakhane project and why does it matter?**
A: Masakhane is a community-driven NLP research initiative focused on African languages. It has produced MT benchmarks, evaluation datasets, and pretrained models for 38+ African languages — most of which had no prior NLP resources. It demonstrates that community participation from native speakers can rapidly bootstrap resources for underserved languages. It also highlights that most NLP research serves less than 10% of the world's speakers.

---

## Common Mistakes / Gotchas

- **Confusing low-resource with low-speaker-count**: Yoruba has 45 million speakers — it's low-resource due to digital divide and research bias, not because of few speakers. Resource level ≠ language vitality.
- **Assuming English-pretrained models work for low-resource**: A model pretrained only on English will perform very poorly on Yoruba or Tamil even if fine-tuned on a small labeled set. Start with multilingual pretrained models (mBERT, XLM-R, mT5).
- **Ignoring orthographic issues**: many low-resource languages have non-standard orthographies, tonal markings that are inconsistently written, or multiple competing writing systems. Text cleaning and normalization is crucial and language-specific.
- **Evaluating only on lab benchmarks**: Low-resource NLP benchmarks are often created by non-native speakers or via machine translation — they may not represent the actual needs of the language community. Community engagement is essential.

---

## Further Reading / Paper References

- Joshi, P. et al. (2020). *The State and Fate of Linguistic Diversity and Inclusion in the NLP World.* ACL. [[arxiv:2004.09095]] — must-read survey
- Adelani, D.I. et al. (2021). *MasakhaNER: Named Entity Recognition for African Languages.* EACL. [[arxiv:2103.11811]]
- Conneau, A. et al. (2020). *Unsupervised Cross-lingual Representation Learning at Scale.* (XLM-R) [[arxiv:1911.02116]]
- Hu, J. et al. (2020). *XTREME: A Massively Multilingual Multi-task Benchmark.* [[arxiv:2003.11080]]
- Ruder, S. et al. (2019). *A Survey of Cross-lingual Word Embedding Models.* JAIR. [[arxiv:1706.04902]]
- Lauscher, A. et al. (2020). *From Zero to Hero: On the Limitations of Zero-Shot Language Transfer with Multilingual Transformers.* EMNLP.
- Nekoto, W. et al. (2020). *Participatory Research for Low-resourced Machine Translation: A Case Study in African Languages.* (Masakhane) EMNLP Findings.
