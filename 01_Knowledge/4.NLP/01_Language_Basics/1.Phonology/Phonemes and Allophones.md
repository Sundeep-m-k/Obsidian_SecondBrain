# Phonemes and Allophones

tags: #nlp #phonology #linguistics #speech
links: [[Prosody and Stress]] [[Speech Sounds IPA]] [[Tokenization — BPE]] [[Features]]

---

## Definition + Intuition

A **phoneme** is the smallest unit of sound in a language that can distinguish meaning. It is an *abstract* mental category — not a physical sound, but a cognitive unit that speakers of a language treat as "the same sound."

An **allophone** is a concrete, physical realization of a phoneme. One phoneme can have multiple allophones depending on phonetic context, but native speakers perceive them as the same sound.

> **Intuition**: Think of a phoneme as a class label, and allophones as instances of that class. The phoneme `/t/` in English is a class. The aspirated `[tʰ]` in *top* and the unaspirated `[t]` in *stop* are two instances (allophones) of the same class. A native English speaker doesn't notice the difference; a learner of English from a language where this distinction is meaningful (e.g. Hindi) does.

---

## Key Properties / Types

### Phoneme Properties
- **Contrastive**: changing one phoneme changes meaning. `bat` vs `cat` — `/b/` and `/k/` are distinct phonemes.
- **Abstract**: phonemes exist in the mind of the speaker, not in physical air pressure waves.
- **Language-specific**: what counts as one phoneme vs two differs across languages. `/l/` and `/r/` are separate phonemes in English but allophones of the same phoneme in some Japanese dialects.

### Types of Allophones
| Type | Description | Example |
|------|-------------|---------|
| **Complementary distribution** | Allophones appear in mutually exclusive environments | Aspirated `[tʰ]` only at start of stressed syllable |
| **Free variation** | Allophones can substitute freely without meaning change | Some speakers say `[ɛ]` or `[e]` in unstressed syllables |
| **Contextual allophones** | Triggered by neighboring sounds (coarticulation) | Nasalization of vowels before nasal consonants |

### Minimal Pairs
The standard test for phonemic status: two words that differ in exactly one sound and have different meanings.

```
/p/ vs /b/: "pat" vs "bat"
/s/ vs /z/: "sip" vs "zip"
/i:/ vs /ɪ/: "sheep" vs "ship"
```

---

## Math / Formal Notation

In formal phonology, a language's phoneme inventory is modeled as a set:

$$\Phi = \{\phi_1, \phi_2, \ldots, \phi_n\}$$

Each phoneme $\phi_i$ is defined as a **bundle of distinctive features** (binary-valued):

$$\phi_i = [±\text{voiced},\ ±\text{nasal},\ ±\text{continuant},\ ±\text{labial},\ \ldots]$$

For example, English `/p/` = `[−voiced, −nasal, −continuant, +labial]`

An allophone mapping can be expressed as a phonological rule:

$$\phi \rightarrow [\text{allophone}] / \text{context}\_\_\text{context}$$

Example: `/t/ → [tʰ] / #\_V` (t becomes aspirated at the start of a word before a vowel)

The **phoneme-to-grapheme** mapping problem central to spelling is:

$$P(g_1, g_2, \ldots | \phi_1, \phi_2, \ldots)$$

This is a sequence-to-sequence problem — directly relevant to G2P (grapheme-to-phoneme) models in TTS.

---

## Examples (Concrete)

**English `/t/` allophones:**
- `[tʰ]` — aspirated, in *top*, *time* (syllable-initial, stressed)
- `[t]` — unaspirated, in *stop*, *sting*
- `[ɾ]` — flap, in American English *butter*, *water* (intervocalic)
- `[ʔ]` — glottal stop, in *button* (pre-nasal, British)
- `[t̚]` — unreleased, in *cat*, *bat* (word-final)

All five are allophones of one phoneme `/t/` — English speakers hear them as "the same."

**Cross-linguistic contrast:**
- Thai has three-way distinction: `/p/`, `/pʰ/`, `/b/` — aspirated, unaspirated voiceless, voiced
- English only has two: `/p/` (covers both aspirated and unaspirated) and `/b/`
- This is why English speakers struggle to hear/produce the Thai distinction

---

## How It Connects to ML / NLP

| Phonology Concept    | NLP/ML Application                                       |
| -------------------- | -------------------------------------------------------- |
| Phoneme inventory    | Vocabulary size in speech models; HMM state space in ASR |
| Allophonic variation | Data augmentation in TTS; robustness in ASR              |
| Distinctive features | Feature vectors for phoneme embeddings                   |
| Phoneme sequences    | Input tokens for G2P models, TTS front-ends              |
| Minimal pairs        | Adversarial examples in speech models                    |
|                      |                                                          |

**Cross-links:**
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — phonemes as feature bundles parallel ML feature vectors
- [[3.ML & DL/1.Concepts/1.Foundations/Features.md]] — distinctive features are binary features
- The phoneme inventory is a **finite vocabulary** — the same concept as a token vocabulary in [[Tokenization — BPE]]

**In ASR (Automatic Speech Recognition):**
Classical ASR pipelines explicitly model phonemes as HMM states. End-to-end models (wav2vec 2.0, Whisper) implicitly learn phoneme-like representations without explicit phoneme labels.

---

## Common Interview Questions

**Q: What is the difference between a phoneme and a phone?**
A: A *phone* is any distinct speech sound, described in purely phonetic terms (IPA). A *phoneme* is an abstract, language-specific unit — the cognitive category. Multiple phones can be realizations (allophones) of one phoneme.

**Q: Why does phoneme awareness matter for NLP?**
A: It explains why text-to-speech and speech-to-text are hard. The same written word can have multiple pronunciations (heteronyms: *lead*, *read*), and the same sound can map to multiple spellings. Phoneme-level modeling is the classical solution.

**Q: How does phonemic contrast differ across languages?**
A: What is one phoneme in one language can be two in another. This causes systematic pronunciation errors when people learn a second language and matters for multilingual ASR/TTS.

---

## Common Mistakes / Gotchas

- **Confusing phoneme with letter**: English has 26 letters but ~44 phonemes. Letters and phonemes do not correspond 1:1. `/f/` is spelled *f*, *ph*, *gh* (enough).
- **Assuming allophones are free to swap**: Swapping allophones can make speech sound foreign/accented even if meaning is preserved, because the context rules are violated.
- **Language-transferring phoneme intuitions**: Do not assume two languages have the same phoneme inventory. English `/v/` does not exist in Japanese; `/r/` sounds very different across languages.
- **Ignoring phonology in multilingual NLP**: Tokenizers trained on text ignore phonology entirely. This is fine for written NLP but breaks for TTS, ASR, and rhyme/pun detection tasks.

---

## Further Reading / Paper References

- Chomsky, N. & Halle, M. (1968). *The Sound Pattern of English.* — foundational distinctive feature theory
- Ladefoged, P. & Johnson, K. *A Course in Phonetics* — best accessible reference
- [[Baevski et al. (2020). wav2vec 2.0 — learns latent phoneme-like units without supervision]] [[arxiv:2006.11477]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 26 — phonetics/phonology for NLP practitioners
