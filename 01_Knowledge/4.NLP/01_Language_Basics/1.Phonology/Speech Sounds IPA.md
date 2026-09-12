# Speech Sounds — IPA

tags: #nlp #phonology #ipa #speech #asr #tts
links: [[Phonemes and Allophones]] [[Prosody and Stress]] [[Tokenization — BPE]]

---

## Definition + Intuition

The **International Phonetic Alphabet (IPA)** is a standardized notation system for representing the sounds of any human language using a unique symbol per distinct sound. It decouples *sound* from *spelling* — each IPA symbol maps to exactly one phonetic sound, regardless of language.

**Speech sounds** are produced by modulating airflow from the lungs through the vocal tract (larynx, pharynx, mouth, nose). They are classified along three dimensions: **voicing**, **place of articulation**, and **manner of articulation**.

> **Intuition**: IPA is the ASCII of human speech. Just as ASCII gives a unique code to every printable character regardless of font, IPA gives a unique symbol to every producible sound regardless of language. It's the universal encoding layer for the "tokens" of spoken language.

---

## Key Properties / Types

### Consonant Classification (3 dimensions)

**1. Voicing** — are the vocal folds vibrating?
- **Voiced**: `/b/`, `/d/`, `/g/`, `/v/`, `/z/` — cords vibrate
- **Voiceless**: `/p/`, `/t/`, `/k/`, `/f/`, `/s/` — cords not vibrating

**2. Place of Articulation** — where in the vocal tract is the constriction?

| Place | Description | Examples |
|-------|-------------|---------|
| Bilabial | Both lips | `/p/`, `/b/`, `/m/` |
| Labiodental | Lip + teeth | `/f/`, `/v/` |
| Dental | Tongue + teeth | `/θ/` (think), `/ð/` (this) |
| Alveolar | Tongue + alveolar ridge | `/t/`, `/d/`, `/n/`, `/s/`, `/z/` |
| Palatal | Tongue + hard palate | `/j/` (yes) |
| Velar | Tongue + soft palate | `/k/`, `/g/`, `/ŋ/` (sing) |
| Glottal | At the glottis | `/h/`, `/ʔ/` (glottal stop) |

**3. Manner of Articulation** — how is the airflow modified?

| Manner | Description | Examples |
|--------|-------------|---------|
| Stop/Plosive | Full closure, then release | `/p/`, `/t/`, `/k/`, `/b/`, `/d/`, `/g/` |
| Fricative | Narrow constriction, turbulent air | `/f/`, `/s/`, `/ʃ/` (sh), `/v/`, `/z/` |
| Affricate | Stop + fricative | `/tʃ/` (ch), `/dʒ/` (j) |
| Nasal | Velum lowered, air through nose | `/m/`, `/n/`, `/ŋ/` |
| Approximant | Partial constriction, no turbulence | `/w/`, `/j/`, `/l/`, `/r/` |
| Lateral | Air around sides of tongue | `/l/` |

### Vowel Classification (3 dimensions)
Vowels are classified by **tongue height**, **tongue backness**, and **lip rounding**.

```
         Front   Central   Back
High      /i/      /ɨ/      /u/
Mid       /e/      /ə/      /o/
Low       /æ/      /a/      /ɑ/
```

- `/i/` = "beet" — high front unrounded
- `/u/` = "boot" — high back rounded
- `/ə/` = schwa ("about") — mid central unrounded (most common English vowel)
- `/æ/` = "cat" — low front unrounded

---

## Math / Formal Notation

A speech sound can be represented as a **feature vector** of binary distinctive features:

$$\vec{f}(\text{/p/}) = [−\text{voiced},\ +\text{labial},\ −\text{nasal},\ +\text{stop},\ −\text{continuant},\ \ldots]$$

This maps naturally to ML feature representations. The acoustic signal itself is modeled as:

**Spectrogram**: a 2D representation of frequency content over time.
$$S(t, f) = \left| \int_{-\infty}^{\infty} x(\tau) \cdot w(\tau - t) \cdot e^{-j2\pi f \tau} d\tau \right|^2$$

where $x(\tau)$ is the raw audio waveform and $w$ is a window function (Hann, Hamming).

**Mel-filterbank features** (used in most modern ASR):
$$\text{mel}(f) = 2595 \cdot \log_{10}\left(1 + \frac{f}{700}\right)$$

The mel scale approximates human auditory perception — equal perceptual distances map to equal mel intervals. This is the standard input to ASR systems (MFCCs or mel spectrograms).

---

## Examples (Concrete)

**IPA transcriptions of English words:**
```
"cat"     → /kæt/
"phone"   → /foʊn/
"thought" → /θɔːt/
"through" → /θruː/
"knight"  → /naɪt/
"gnome"   → /noʊm/
```

Notice: spelling is irrelevant in IPA. "phone", "thought", and "through" all look different but phonetically they each map cleanly.

**Same IPA symbol, different orthographies:**
- `/f/` → `f` (food), `ph` (phone), `gh` (tough)
- `/ʃ/` → `sh` (ship), `ti` (nation), `ci` (special), `ch` (chef)

**Sounds English lacks but other languages use:**
- `/x/` — velar fricative: German *Bach*, Scottish *loch*, Spanish *jota*
- `/ɣ/` — voiced velar fricative: Greek, Arabic
- `/ɾ/` — alveolar tap: Spanish *pero* (but not *perro*)
- `/ŋ/` — exists in English (*sing*) but never word-initial; word-initial in many African languages

---

## How It Connects to ML / NLP

| IPA Concept | NLP/ML Application |
|---|---|
| Phoneme set | Vocabulary / token set for speech models |
| Feature vectors | Input features for phone classifiers in ASR |
| Mel spectrogram | Standard input representation for all neural speech models |
| G2P (grapheme-to-phoneme) | Sequence-to-sequence prediction task; needed for TTS |
| Phone duration | Regression target in TTS acoustic models |

**G2P as a seq2seq problem:**
$$P(\phi_1 \ldots \phi_n \mid c_1 \ldots c_m)$$

where $c_i$ are characters and $\phi_j$ are IPA phonemes. This is exactly [[3.ML & DL/1.Concepts/3.Linear Regression/Regression Pipeline.md]]-style mapping but over sequences.

**ASR acoustic model:**
The acoustic model computes $P(\phi \mid \text{mel frame})$ — a classification problem over the phoneme inventory at each time frame. [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]]

**Phoneme embeddings:**
Phonemes can be embedded using their feature vectors as initializations — a form of linguistically-informed [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]].

---

## Common Interview Questions

**Q: Why does NLP mostly work at the character/subword level rather than the phoneme level?**
A: Because NLP primarily operates on *written* text, where the basic unit is a character or subword token. Phonemes are the unit of spoken language. For written NLP, phoneme-level processing is not necessary (and not always possible without a G2P model). For speech NLP (ASR, TTS), phonemes are central.

**Q: What are MFCCs and why were they used in classical ASR?**
A: Mel-Frequency Cepstral Coefficients are a compact, perceptually-motivated representation of the spectral envelope of speech. They reduce a high-dimensional spectrogram to ~13 coefficients that correlate well with phonetic identity. HMM-based ASR used MFCCs as input features for decades. Neural ASR (wav2vec, Whisper) now learns its own features from raw waveforms or mel spectrograms directly.

**Q: How many phonemes does English have?**
A: Approximately 44 (varies slightly by dialect and analysis): ~24 consonants and ~20 vowels/diphthongs. Compare to the 26 letters of the alphabet — the mismatch is why English spelling is notoriously irregular.

---

## Common Mistakes / Gotchas

- **Using orthography as a phonetic proxy**: English spelling is not phonetic. Never assume letter count = phoneme count, or that spelling reflects pronunciation.
- **Language-universal phoneme assumption**: There is no universal phoneme inventory. Every language has its own set. Multilingual ASR must handle different phoneme sets or use a universal phone set (like IPA-based ones).
- **Ignoring the mel scale**: Linear frequency spectrograms are less useful for speech than mel spectrograms. Human perception is logarithmic in frequency — the mel transformation reflects this.
- **Confusing phones and phonemes**: A phone is a physical sound described articulatorily/acoustically. A phoneme is a cognitive category. The same phone can be two different phonemes in one language and one phoneme in another.

---

## Further Reading / Paper References

- Ladefoged, P. & Johnson, K. *A Course in Phonetics* (7th ed.) — the definitive textbook
- IPA Chart: https://www.internationalphoneticassociation.org/content/ipa-chart
- Davis, S. & Mermelstein, P. (1980). *Comparison of Parametric Representations for Monosyllabic Word Recognition.* — original MFCC paper
- Baevski, A. et al. (2020). *wav2vec 2.0: A Framework for Self-Supervised Learning of Speech Representations.* [[arxiv:2006.11477]]
- Radford, A. et al. (2022). *Robust Speech Recognition via Large-Scale Weak Supervision.* (Whisper) [[arxiv:2212.04356]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 26
