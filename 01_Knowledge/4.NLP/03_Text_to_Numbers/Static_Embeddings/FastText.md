---
tags: [nlp, static-embeddings, fasttext, subword, oov, morphology, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Word2Vec]] [[GloVe]] [[Character Embeddings]] [[Subword Embeddings]] [[Agglutinative Languages]]"
---

# FastText

## Definition + Intuition

**FastText** (Bojanowski et al., 2017) is an extension of Word2Vec's skip-gram model that represents each word as the **sum of its character n-gram embeddings**. Instead of learning one vector per word, it learns vectors for character n-grams and composes them to form word representations.

**Intuition**: Word2Vec treats "run", "running", "runner", "ran" as completely unrelated — four separate entries in the vocabulary with four independent embeddings. FastText recognises that these words share the n-gram `run` and learns a shared component. This gives FastText two critical advantages:

1. **OOV handling**: A word never seen during training can still be represented from its character n-grams
2. **Morphological awareness**: Related words (inflected forms, derivations) share embedding components → similar representations

This is especially powerful for morphologically rich languages (Turkish, Finnish, Arabic) where a word stem may appear in hundreds of inflected forms, most of which are rare or unseen.

---

## Key Properties / Types

**Character n-gram representation**:

For a word $w$, FastText uses:
- The word itself (as a special token): `<run>`
- All character n-grams of length $n_{\min}$ to $n_{\max}$ (default: 3–6)

Example for "running":
```
n=3: <ru, run, unn, nni, nin, ing, ng>
n=4: <run, runn, unni, nnin, ning, ing>
n=5: <runn, runni, unnin, nning, ning>
n=6: <runni, runnin, unning, nning>
special: <running>
```

**Word vector** = sum of all its n-gram embeddings:
$$v_w = \frac{1}{|G_w|}\sum_{g \in G_w} z_g$$

where $G_w$ is the set of n-grams for word $w$ and $z_g$ is the embedding for n-gram $g$.

**OOV word representation**: A word never seen during training can be represented by the mean of its n-gram embeddings (all of which may have been seen in other words). This is a key advantage over Word2Vec and GloVe.

**Hashing trick**: To avoid maintaining a separate vocabulary of all n-grams (can be millions), FastText hashes n-grams into a fixed-size bucket table (default: 2 million buckets), trading exact n-gram identity for memory efficiency.

**FastText for text classification**: Facebook also released FastText for classification — a shallow architecture (bag of word/n-gram embeddings → linear classifier) that achieves near state-of-the-art on many classification tasks while training in seconds.

---

## Math / Formal Notation

**N-gram set for word $w$**:
$$G_w = \{g : g \text{ is a character n-gram of } w, n_{\min} \leq |g| \leq n_{\max}\} \cup \{\langle w \rangle\}$$

where $\langle w \rangle$ denotes the special whole-word token with boundary markers.

**Skip-gram with subword objective**:

Replace the single word vector $v_w$ in the standard skip-gram with the subword-composed vector:

$$\mathcal{L} = -\sum_{(w,c)} \left[ \log \sigma\left(\sum_{g \in G_w} z_g \cdot v_c\right) + \sum_{j=1}^{k} \mathbb{E}_{c_j \sim P_n} \log \sigma\left(-\sum_{g \in G_w} z_g \cdot v_{c_j}\right) \right]$$

All other training details (negative sampling, subsampling, window size) are inherited from skip-gram.

**Gradient update**: When updating for word $w$, *all* n-gram embeddings in $G_w$ are updated simultaneously. This means frequent n-grams (shared across many words) receive many gradient updates → well-trained representations.

**OOV representation at inference**:
$$v_{\text{OOV}} = \frac{1}{|G_{\text{OOV}}|} \sum_{g \in G_{\text{OOV}}} z_g$$

Only n-grams seen during training have embeddings. Unknown n-grams (from truly novel character sequences) fall back to their hash bucket, which may collide with another n-gram.

---

## Examples (Concrete)

**OOV handling** (Word2Vec vs FastText):

Word "photosynthetically" not seen in training:
- Word2Vec → `<UNK>` (zero or random vector)
- FastText → compose from `<ph, pho, hot, oto, tos, osy, syn, ynt, nth, the, het, eti, tic, ica, cal, all, lly, ly>` → meaningful vector close to "photosynthesis", "photosynthetic", "synthetically"

**Morphological generalisation**:
```python
import fasttext

model = fasttext.load_model('cc.en.300.bin')

# Standard words
model.get_word_vector("running")   # 300-dim, good representation

# OOV — misspelling
model.get_word_vector("runnning")  # still useful (shares n-grams with "running")

# OOV — technical term
model.get_word_vector("immunoprecipitation")  # composed from bio n-grams
```

**Language comparison** (fastText pre-trained models cover 157 languages):
```python
# Turkish agglutination: "evlerindekilerden" = "from those in their houses"
# Word2Vec: OOV (almost certainly)
# FastText: composed from character n-grams → reasonable vector

model_tr = fasttext.load_model('cc.tr.300.bin')
model_tr.get_word_vector("evlerindekilerden")  # works
```

**FastText classification**:
```python
import fasttext

# Train
model = fasttext.train_supervised(
    input="train.txt",         # format: "__label__sport text..."
    lr=0.5, epoch=25, wordNgrams=2
)

# Predict
model.predict("the match was intense", k=1)
# (('__label__sport',), array([0.98]))
```

---

## How It Connects to ML / NLP

| FastText Concept | ML/NLP Link |
|----------------|-------------|
| Subword composition = feature aggregation | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Engineering.md]] — n-grams are hand-designed sub-features automatically learned |
| Sum of n-gram embeddings | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Feature Vector.md]] — word vector is a sum (linear combination) of n-gram feature vectors |
| OOV via n-gram composition | [[3.ML & DL/1.Concepts/1.Foundations/Generalization.md]] — FastText generalises to unseen words via subword structure |
| Morphological awareness | [[Agglutinative Languages]] — FastText was explicitly designed to handle morphologically rich languages |
| Hashing trick for n-gram vocab | [[3.ML & DL/1.Concepts/6.Feature Engineering & Data Preparation/Vectorization.md]] — hash trick is a fixed-size feature representation |
| Negative sampling objective | [[3.ML & DL/1.Concepts/4.Loss and Cost/Logistic Loss.md]] — same binary cross-entropy as Word2Vec SGNS |
| N-gram features + linear classifier | [[3.ML & DL/1.Concepts/7.Logistic Regression/Classification Pipeline.md]] — FastText classifier is logistic regression over n-gram embeddings |
| Shared n-gram parameters = regularisation | [[3.ML & DL/1.Concepts/9.Regularization/Regularization.md]] — sharing n-gram embeddings across words acts as morphological regularisation |

---

## Common Interview Questions

**Q: What is FastText's key innovation over Word2Vec?**
A: FastText represents words as the sum of their character n-gram embeddings, rather than a single whole-word vector. This enables: (1) OOV handling — any word can be represented from its n-grams even if unseen during training; (2) morphological awareness — "run", "running", "runner" share n-gram components and therefore have related embeddings.

**Q: How does FastText handle OOV words?**
A: At inference, the embeddings of a word's character n-grams are averaged to form its representation. Since n-grams are character-level, they are likely to have been seen during training even if the full word hasn't. This means FastText degrades gracefully — a rare or OOV word still gets a reasonable embedding based on its morphological components.

**Q: When would you prefer FastText over BERT?**
A: (1) When computational resources are limited — FastText inference is microseconds vs. milliseconds for BERT. (2) When the downstream task is simple classification — FastText's bag-of-ngrams classifier is extremely fast. (3) For morphologically rich languages where subword composition is important. (4) When you need static embeddings for downstream models that can't afford Transformer encoding.

**Q: What n-gram range does FastText use by default?**
A: Character n-grams of length 3 to 6. Length 3 captures common prefixes/suffixes; length 6 captures longer morphological units. The whole-word token `<word>` is also always included. These are tunable hyperparameters.

**Q: What is the hashing trick in FastText?**
A: All character n-grams are hashed into a fixed-size bucket table (default 2M buckets). Multiple n-grams may hash to the same bucket (collision), but in practice the bucket table is large enough that collisions are rare and don't significantly degrade performance. This avoids storing a separate vocabulary for all possible n-grams.

---

## Common Mistakes / Gotchas

- **Using FastText when context matters**: FastText is static — "bank" in "river bank" and "bank account" have the same embedding. For polysemy, use BERT.
- **Forgetting boundary markers**: FastText wraps words with `<` and `>` boundary markers before extracting n-grams. "run" → `<run>`. The trigrams are `<ru, run, un>`. Don't forget these when implementing from scratch.
- **Hash collisions in small models**: If you train with a small bucket count (e.g., 100k), collision rate is high and representation quality suffers. Use at least 1M buckets (2M is the default).
- **Language-specific n-gram range**: For agglutinative languages (Turkish, Finnish), longer n-grams (4–6) capture more meaningful morphemes. For isolating languages (Chinese, Vietnamese), character n-grams don't capture morphology — use character embeddings or SentencePiece instead.
- **Pre-trained model mismatch**: Facebook's pre-trained models are trained on Common Crawl (web text). For formal/scientific text, consider retraining. 157-language models are available at https://fasttext.cc/docs/en/crawl-vectors.html

---

## Further Reading / Paper References

- Bojanowski et al. (2017). Enriching Word Vectors with Subword Information — original FastText embedding paper arxiv:1607.04606
- Joulin et al. (2017). Bag of Tricks for Efficient Text Classification — FastText classifier arxiv:1607.01759
- Mikolov et al. (2018). Advances in Pre-Training Distributed Word Representations — FastText improvements arxiv:1712.09405
- FastText official site: https://fasttext.cc — pre-trained models for 157 languages
- Jurafsky & Martin, SLP Ch. 6.7 — subword embeddings, including FastText
