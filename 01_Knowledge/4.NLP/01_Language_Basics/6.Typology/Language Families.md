# Language Families

tags: #nlp #typology #multilingual #cross-lingual #transfer-learning
links: [[Morphological Typology]] [[Low-Resource Languages]] [[Agglutinative Languages]] [[XLM-R]] [[mBERT]]

---

## Definition + Intuition

A **language family** is a group of languages that descended from a common ancestor (proto-language). Languages within a family share vocabulary, phonological patterns, and grammatical structures due to common historical origin.

> **Intuition**: Language families are like evolutionary trees — just as primates share common ancestors with humans, Spanish and French share Latin as a proto-language. For NLP, family membership predicts how well models transfer across languages: models transfer better within families than across them.

---

## Key Language Families (NLP-Relevant)

| Family | Languages | Key NLP Features |
|--------|-----------|-----------------|
| **Indo-European** | English, Spanish, French, German, Russian, Hindi, Bengali | Most NLP resources; diverse morphology |
| **Sino-Tibetan** | Mandarin, Cantonese, Tibetan | Tonal, mostly isolating; character-based writing |
| **Afro-Asiatic** | Arabic, Hebrew, Amharic, Somali | Root-and-pattern morphology; right-to-left scripts |
| **Niger-Congo** | Swahili, Yoruba, Zulu, Igbo | Agglutinative; noun class systems; low resource |
| **Dravidian** | Tamil, Telugu, Kannada, Malayalam | Agglutinative; SOV; very different from Indo-European |
| **Turkic** | Turkish, Uzbek, Kazakh, Azerbaijani | Agglutinative; SOV; vowel harmony |
| **Uralic** | Finnish, Hungarian, Estonian | Agglutinative; extensive case systems |
| **Austronesian** | Indonesian, Tagalog, Malay, Javanese | Focus system; large geographic spread |
| **Japonic** | Japanese | Agglutinative; SOV; complex writing system |
| **Koreanic** | Korean | Agglutinative; SOV; honorifics |

### NLP Resource Distribution
```
~7,000 languages in the world
~2,000 have a writing system
~500 have some digital text
~100 have substantial NLP research
~10 have truly large-scale resources (English >> all others)
```

---

## How It Connects to ML / NLP

**Transfer within vs across families:**
- English → German: +15 F1 on NER (within Indo-European)
- English → Turkish: +3 F1 on NER (cross-family, very different morphology)

**mBERT / XLM-R**: multilingual models benefit from family proximity. Fine-tuning on English data and evaluating on Spanish (same family) shows ~70% of supervised performance. Cross-family transfer is much weaker.

**Cross-links:**
- [[3.ML & DL/1.Concepts/8.Model Behavior/Generalization.md]] — cross-lingual generalization
- [[Agglutinative Languages]] — morphological typology by family
- [[Low-Resource Languages]] — most language families are low-resource

---

## Further Reading

- Ethnologue: https://www.ethnologue.com — comprehensive language family reference
- Joshi, P. et al. (2020). *The State and Fate of Linguistic Diversity and Inclusion in the NLP World.* ACL. — NLP resource distribution
- Conneau, A. et al. (2020). *Unsupervised Cross-lingual Representation Learning at Scale.* [[arxiv:1911.02116]]
