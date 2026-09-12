---
tags: [nlp, subword-char, character-embeddings, cnn, oov, morphology, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[Subword Embeddings]] [[FastText]] [[Tokenization — BPE]] [[Agglutinative Languages]]"
---

# Character Embeddings

## Definition + Intuition

**Character embeddings** represent words by learning over their individual characters rather than looking them up in a word-level vocabulary. Each character is embedded into a dense vector, and then a composition function (typically a CNN or LSTM) combines the character vectors into a single word-level representation.

**Intuition**: The word "unbelievable" is not in the dictionary → Word2Vec returns `<UNK>`. But a character-level model sees `u-n-b-e-l-i-e-v-a-b-l-e` — it recognises the prefix `un-`, the root `believ-`, and the suffix `-able`, all of which it has seen in other words. The result is a meaningful representation even for OOV words.

**Why characters?** Characters are a closed, finite set (~100 characters in English; ~10,000 Unicode code points for multilingual). Any word — no matter how rare, misspelled, or technical — can be represented from characters. This provides a natural solution to the OOV problem without engineering subword vocabularies.

---

## Key Properties / Types

**CNN over Characters** (most common architecture — Kim 2016):

```
Word: "playing"
Characters: [p, l, a, y, i, n, g] → character embeddings: (7, char_dim)
Conv filters of widths [2, 3, 4, 5] → feature maps
Max-over-time pooling → fixed-size word vector
```

Properties:
- Local: CNN captures character n-gram features (morphological affixes)
- Fixed output size regardless of word length
- Fast: parallelisable across all positions
- Used in: ELMo input layer, CoVe, many NER models

**BiLSTM over Characters**:
```
Characters → char embeddings → BiLSTM → [h_fwd_last; h_bwd_last] → word vector
```

Properties:
- Captures longer-range dependencies within the word
- Slower than CNN (sequential)
- Better for languages with complex long-range morphology (Finnish, Turkish)

**Character n-gram embeddings** (FastText-style):
- Hash character n-grams into a fixed bucket table
- Sum n-gram embeddings → word vector
- No explicit composition model — simpler and faster
- See: [[FastText]]

**Byte-level representations**:
- Operate on raw UTF-8 bytes rather than Unicode characters
- True universality: handles any language, emoji, code
- Used in: ByT5, Charformer, GPT-4 tokeniser edge cases

---

## Math / Formal Notation

**Character CNN (Kim 2016)**:

For word $w = c_1 c_2 \ldots c_m$ (sequence of character indices):

1. Look up character embeddings: $C = [e_{c_1}, e_{c_2}, \ldots, e_{c_m}] \in \mathbb{R}^{m \times d_c}$

2. Apply convolution with filter $H \in \mathbb{R}^{w \times d_c}$ (width $w$, output 1 feature):
$$f_i = \tanh(H \cdot C[i:i+w] + b) \quad \text{for } i = 1, \ldots, m-w+1$$

3. Max-over-time pooling:
$$y = \max_i f_i$$

4. Multiple filters of different widths $\{2, 3, 4, 5\}$, each with $k$ feature maps → concatenate all pooled values → word vector $\in \mathbb{R}^{|widths| \times k}$

**Highway network** (Kim 2016 adds these on top of char CNN output to enable deeper transformations):
$$z = t \odot g(Wx + b) + (1-t) \odot x$$

where $t = \sigma(W_T x + b_T)$ is the transform gate. Allows gradient to flow through unchanged (residual connection precursor).

**BiLSTM character model**:
$$h_t^{\rightarrow} = \text{LSTM}(e_{c_t}, h_{t-1}^{\rightarrow}), \quad h_t^{\leftarrow} = \text{LSTM}(e_{c_{m+1-t}}, h_{t+1}^{\leftarrow})$$
$$v_w = [h_m^{\rightarrow}; h_1^{\leftarrow}]$$

The final forward and backward hidden states capture prefix (suffix) morphology respectively.

---

## Examples (Concrete)

**OOV handling in NER**:

Sentence: "The patient was prescribed Paracetamoxyfrusebendroneomycin."

Word-level embedding: `<UNK>` — no information.

Character CNN: Sees `Para-`, `-mycin`, `-one`, etc. — familiar drug morpheme patterns. Returns a vector close to other drug names in the training set. NER system correctly predicts DRUG entity.

**Misspelling robustness**:
```
"recieve" (misspelled) → char CNN → vector close to "receive"
because character n-grams [rec, eci, cie, iev, eve] overlap heavily
```

**Architecture in a neural NER model**:
```python
import torch
import torch.nn as nn

class CharCNN(nn.Module):
    def __init__(self, char_vocab_size=100, char_embed_dim=30, 
                 num_filters=50, filter_widths=[2,3,4,5]):
        super().__init__()
        self.char_embed = nn.Embedding(char_vocab_size, char_embed_dim, padding_idx=0)
        self.convs = nn.ModuleList([
            nn.Conv1d(char_embed_dim, num_filters, w) 
            for w in filter_widths
        ])
    
    def forward(self, chars):
        # chars: (batch, seq_len, max_word_len)
        B, S, L = chars.shape
        chars = chars.view(B*S, L)
        x = self.char_embed(chars)           # (B*S, L, char_dim)
        x = x.transpose(1, 2)               # (B*S, char_dim, L) for Conv1d
        pooled = []
        for conv in self.convs:
            h = torch.relu(conv(x))         # (B*S, num_filters, L-w+1)
            h = h.max(dim=-1).values        # (B*S, num_filters)
            pooled.append(h)
        out = torch.cat(pooled, dim=-1)     # (B*S, num_filters * len(filter_widths))
        return out.view(B, S, -1)           # (batch, seq_len, word_repr_dim)

# In the full NER model: concatenate char_repr with word_embedding before biLSTM/Transformer
```

---

## How It Connects to ML / NLP

| Character Embedding Concept | ML/NLP Link |
|---------------------------|-------------|
| Character vocab is closed set | [[3.ML & DL/1.Concepts/1.Foundations/Features.md]] — characters are a fixed feature alphabet |
| CNN over character sequence | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — CNN automatically learns character n-gram features |
| Max pooling = feature selection | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Vector.md]] — max-over-time picks the most activated feature regardless of position |
| OOV via character composition | [[3.ML & DL/1.Concepts/1.Foundations/Generalization.md]] — character model generalises to words outside training vocabulary |
| Highway network | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — highway gates control gradient flow, enabling deep character CNNs |
| Concatenating char + word embeddings | [[3.ML & DL/1.Concepts/6.Features and Representation/Multiple Features.md]] — combining information sources, standard in NER |
| Char model for morphology | [[Morphemes — Free and Bound]] — character n-grams approximate morpheme boundaries |

**When to use character embeddings**:
- NER, POS tagging — morphological cues (capitalization, suffixes) are highly predictive
- Morphologically rich languages — Turkish, Finnish, Arabic
- Noisy text — social media, OCR output, clinical notes with spelling variation
- When OOV rate is high — technical domains, named entities

---

## Common Interview Questions

**Q: Why are character embeddings useful for NER?**
A: NER benefits from morphological cues: capitalisation signals proper nouns, suffixes like `-tion`, `-ness`, `-ity` signal specific entity types, and medical/legal named entities often share morphological patterns (drug suffixes: `-mycin`, `-pril`, `-olol`). Character CNNs automatically learn these features. Additionally, character models handle OOV entities gracefully — a new drug name will still produce a meaningful representation from its character n-grams.

**Q: What is the difference between character embeddings and FastText?**
A: Both use character-level information. FastText uses a bag of character n-grams (summed without position-awareness), which is fast but loses character order. Character CNNs and biLSTMs explicitly process the character sequence — the CNN captures local n-gram features with positional awareness; the biLSTM captures the full sequence. In practice, CNN character models outperform FastText for sequence labelling tasks because position and morphological structure matter.

**Q: Why use CNN rather than LSTM for character composition?**
A: CNNs are faster (parallelisable across character positions) and have been shown to work as well or better than LSTMs for character composition in most NLP tasks. LSTMs have an advantage for very long words or languages where long-range character dependencies matter. In practice, CNNs are the default choice.

**Q: How do you combine character embeddings with word embeddings?**
A: Concatenate the character-level word vector with the pre-trained word embedding (e.g., GloVe or BERT subword embedding) before feeding to the sequence model. The two representations are complementary: word embeddings capture distributional semantics; character embeddings capture morphology and handle OOV. This combination consistently outperforms either alone for NER and POS tasks.

---

## Common Mistakes / Gotchas

- **Using characters alone for semantic tasks**: Character models capture morphology well but not semantics (meaning). For semantic similarity or classification, word or sentence embeddings are far better. Characters are most useful as a *supplement* to word embeddings, not a replacement.
- **Ignoring max word length**: Character CNN/LSTM requires a fixed maximum word length for batching. Words longer than this are truncated. Set max length to the 99th percentile of word lengths in your corpus (typically 25–30 for English, 50+ for agglutinative languages).
- **Not handling padding properly**: Padding characters must have a separate `padding_idx` in the embedding layer so their gradients are zeroed. Forgetting this means padding positions contaminate the word representation.
- **Forgetting uppercase as a feature**: Capitalisation is a powerful NER feature (beginning of proper noun). Character embeddings preserve case — don't lowercase before character encoding for NER tasks.
- **Slow training with biLSTM chars**: Character biLSTMs process every character of every word sequentially. For a 20-word sentence averaging 6 chars each = 120 LSTM steps just for character encoding. Use CNN unless you have strong reasons to prefer LSTM.

---

## Further Reading / Paper References

- Kim, Y. et al. (2016). Character-Aware Neural Language Models — character CNN for word-level language modelling arxiv:1508.06615
- Lample et al. (2016). Neural Architectures for Named Entity Recognition — biLSTM-CRF with character embeddings; foundational NER paper arxiv:1603.01360
- Ma & Hovy (2016). End-to-end Sequence Labeling via Bi-directional LSTM-CNNs-CRF — character CNN in NER arxiv:1603.01354
- Xin et al. (2018). Learning Better Internal Structure of Words for Sequence Labeling — analysis of character models for NER
- Peters et al. (2018). ELMo — uses character CNN as the token embedding layer arxiv:1802.05365
