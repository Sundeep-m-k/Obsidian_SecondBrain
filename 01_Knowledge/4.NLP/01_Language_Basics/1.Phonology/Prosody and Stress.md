# Prosody and Stress

tags: #nlp #phonology #prosody #speech #tts
links: [[Phonemes and Allophones]] [[Speech Sounds IPA]] [[Dialogue Generation]] [[Sentiment Analysis]]

---

## Definition + Intuition

**Prosody** is the study of the rhythmic and melodic aspects of speech that operate *above* the level of individual sounds — including stress, tone, intonation, rhythm, and tempo.

**Stress** is the relative emphasis placed on a syllable within a word (lexical stress) or a word within an utterance (phrasal/sentence stress). Stressed syllables are typically louder, longer, and higher in pitch.

> **Intuition**: If phonemes are the *letters* of a musical score, prosody is the *dynamics and expression markings* — the loud/soft, fast/slow, rising/falling that give speech its meaning beyond the words themselves. The sentence "I didn't steal the money" means something different depending on which word you stress:
> - "**I** didn't steal the money" (someone else did)
> - "I didn't **steal** the money" (I borrowed it)
> - "I didn't steal the **money**" (I stole something else)

---

## Key Properties / Types

### Types of Stress
| Type | Scope | Example |
|------|-------|---------|
| **Lexical stress** | Within a word | *PREsent* (noun) vs *preSENT* (verb) |
| **Phrasal stress** | Within a phrase | *HOT dog* (warm dog) vs *hot DOG* (the food) |
| **Sentence/nuclear stress** | Most prominent syllable in utterance | Marks information focus |
| **Contrastive stress** | Emphasizes contrast | "I want the RED one, not the blue one" |

### Components of Prosody
- **Pitch / F0 (fundamental frequency)**: perceived as tone or melody. Rising = question in many languages.
- **Duration**: stressed syllables are longer.
- **Intensity**: stressed syllables are louder (higher amplitude).
- **Tone**: in tonal languages (Mandarin, Thai), pitch is phonemic — it changes word meaning.
- **Intonation**: pitch contour over an utterance. Falling intonation = statement; rising = question (in English).
- **Rhythm**: alternation of stressed and unstressed syllables. English is stress-timed; Spanish is syllable-timed.

### Prosodic Hierarchy
```
Utterance
  └── Intonational Phrase
        └── Phonological Phrase
              └── Prosodic Word
                    └── Foot
                          └── Syllable
                                └── Phoneme
```

---

## Math / Formal Notation

**Fundamental frequency (F0):** the rate of vocal fold vibration, measured in Hz. Perceived as pitch.

For a harmonic sound:
$$f_0 = \frac{1}{T}$$
where $T$ is the period of one glottal cycle.

**ToBI (Tones and Break Indices):** the standard annotation system for English prosody. Labels pitch accents using a two-tone system:
- Pitch accents: `H*`, `L*`, `H*+L`, `L+H*`, etc.
- Phrase tones: `H-`, `L-`
- Boundary tones: `H%`, `L%`

A full intonational phrase label looks like: `L+H* L- H%` (rise–fall on the accented syllable, high boundary — typical for a list item)

**Prominence prediction** can be framed as a sequence labeling task:

$$P(\text{stress}_1, \ldots, \text{stress}_n \mid w_1, \ldots, w_n)$$

where $w_i$ are words and $\text{stress}_i \in \{0, 1\}$ (or a continuous prominence score). This is exactly the CRF / BiLSTM-CRF setup used in [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]]-based sequence labelers.

---

## Examples (Concrete)

**Lexical stress changing word class (English stress pairs):**
```
REcord  (noun: a vinyl record)   vs   reCORD  (verb: to record)
PERmit  (noun: a parking permit) vs   perMIT  (verb: to permit)
INcrease (noun)                  vs   inCREASE (verb)
```

**Sentence stress changing meaning:**
```
"She told me to leave"     — neutral
"SHE told me to leave"     — not him
"She TOLD me to leave"     — she explicitly said so (not hinted)
"She told ME to leave"     — not you
"She told me to LEAVE"     — not to stay
```

**Tonal contrast (Mandarin):**
```
mā (妈) — mother     [flat high tone]
má (麻) — hemp       [rising tone]
mǎ (马) — horse      [dipping tone]
mà (骂) — scold      [falling tone]
```
Same phoneme sequence, four different meanings — purely prosodic distinction.

---

## How It Connects to ML / NLP

| Prosody Concept | NLP/ML Application |
|---|---|
| Stress prediction | TTS front-end; a sequence labeling task |
| Intonation modeling | Expressive TTS (neural prosody models) |
| Prosodic features | Features for emotion/sentiment detection in speech |
| Tonal languages | Requires character-level or tone-aware tokenization |
| Rhythm | Affects WER in ASR; poem/lyric generation |

**Direct ML links:**
- Prosody modeling in TTS (Tacotron, VITS) uses attention over text to predict continuous F0 contours — essentially a regression problem: [[3.ML & DL/1.Concepts/3.Linear Regression/Regression Pipeline.md]]
- Emotion detection from speech uses prosodic features (F0 mean/variance, speech rate) as input features: [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Engineering.md]]
- Tonal language ASR needs the model to distinguish tone — a classification problem at the syllable level: [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]]

---

## Common Interview Questions

**Q: How does prosody affect NLP tasks beyond speech?**
A: In text, prosody is largely absent — punctuation and capitalization carry some signal. This is why sarcasm detection, emphasis inference, and reading-aloud systems are hard. Prosody recovery from text is an open problem.

**Q: What is the difference between stress-timed and syllable-timed languages?**
A: In stress-timed languages (English, German), the interval between stressed syllables is roughly equal — unstressed syllables are compressed. In syllable-timed languages (Spanish, French), each syllable takes roughly equal time. This affects speech rhythm modeling in TTS and ASR.

**Q: Why do TTS systems still sound unnatural?**
A: Mostly because of prosody. Phoneme generation has largely been solved by neural models. The remaining gap is in generating natural, contextually appropriate pitch contours, rhythm, and emphasis — which requires understanding *meaning*, not just text.

---

## Common Mistakes / Gotchas

- **Ignoring prosody for sentiment**: Text-based sentiment models miss sarcasm that is conveyed entirely through prosody ("Oh, *great*." said sarcastically).
- **Assuming English stress rules generalize**: Stress placement differs radically across languages. French has no lexical stress (stress always falls at phrase-final syllable). Applying English stress heuristics to French TTS produces unnatural output.
- **Confusing pitch with tone**: Pitch is a continuous acoustic property. Tone is a discrete phonemic feature (in tonal languages). Intonation is a phrase-level pitch pattern that conveys pragmatic meaning (question, statement, etc.).
- **Flat prosody in TTS**: Early neural TTS (e.g. Tacotron 1) produced correct phonemes but monotone delivery — highlighting that prosody is a separate modeling challenge.

---

## Further Reading / Paper References

- Pierrehumbert, J. (1980). *The Phonology and Phonetics of English Intonation.* MIT PhD thesis. — original ToBI theory
- Wang, Y. et al. (2017). *Tacotron: Towards End-to-End Speech Synthesis.* [[arxiv:1703.10135]] — prosody in neural TTS
- [[Li, N. & Jurafsky, D. (2017). Do Multi-Sense Embeddings Improve Natural Language Understanding?]]— prosody as disambiguation signal
- Jurafsky & Martin, *Speech and Language Processing* Ch. 25 — prosody for NLP practitioners
- Shen, J. et al. (2018). *Natural TTS Synthesis by Conditioning WaveNet on Mel Spectrogram Predictions.* (Tacotron 2) [[arxiv:1712.05884]]
