## What is it?

**Data representation** is how raw, heterogeneous real-world data is converted into the numerical [[Feature Vector]] format that a machine learning model can process. Every ML system begins with a representation decision.

> "The choice of representation can make a problem easy or impossible for an algorithm."

---

## Representation by Data Type

### Tabular / Structured Data

The simplest case — each row is an example, each column is a feature.

**Numerical features:** Use directly, after scaling.

**Categorical features:**
- **Ordinal** (ordered): integer encode → `{low:0, medium:1, high:2}`
- **Nominal** (unordered): one-hot encode → `{red:[1,0,0], green:[0,1,0], blue:[0,0,1]}`
- **High-cardinality** (e.g., zip codes, user IDs): learned embeddings

**Missing values:**
- Mean/median imputation for continuous features
- Mode imputation or "Unknown" category for categoricals
- Add a binary missingness indicator column

---

### Images

A greyscale image of height $H$, width $W$ is a matrix of pixel values $\in [0, 255]^{H \times W}$.

A colour image (RGB) is a 3D tensor $\in [0, 255]^{H \times W \times 3}$.

**Preprocessing:**
1. Resize to a fixed size (e.g., $224 \times 224$).
2. Normalise pixels: divide by 255 → $[0, 1]$, or standardise using dataset mean/std.

**For traditional ML** (not deep learning): flatten to a vector of $H \times W \times C$ features, or extract handcrafted features (HOG, SIFT, colour histograms).

**For deep learning:** pass raw pixel tensor to a CNN — it learns its own features.

---

### Text

**Bag of Words (BoW):** Build a vocabulary $V$ of all unique words. Each document is a vector of length $|V|$, where entry $j$ = count of word $j$ in the document.

$$x_j = \text{count of word}_j \text{ in document}$$

Sparse (most entries zero), loses word order.

**TF-IDF (Term Frequency-Inverse Document Frequency):**
$$\text{TF-IDF}(t, d) = \underbrace{\frac{f_{t,d}}{\sum_{t'} f_{t',d}}}_{\text{TF}} \times \underbrace{\log\frac{N}{1 + n_t}}_{\text{IDF}}$$

Where:
- $f_{t,d}$ = count of term $t$ in document $d$
- $N$ = total documents
- $n_t$ = documents containing term $t$

Upweights rare, informative words. Downweights common words like "the", "is".

**Word Embeddings (Word2Vec, GloVe):**
Each word maps to a dense vector $\in \mathbb{R}^{300}$ (typically). Document = average of its word vectors.

Captures semantic similarity: $\cos(\text{king} - \text{man} + \text{woman}, \text{queen}) \approx 1$.

**Contextual Embeddings (BERT, GPT):**
Each word's representation depends on its context — "bank" in "river bank" vs. "bank account" gets different vectors. State of the art for NLP.

---

### Time Series

A time series is a sequence $x_1, x_2, \ldots, x_T$ of values over time.

**Feature extraction approaches:**
- **Lag features:** $[x_{t-1}, x_{t-2}, \ldots, x_{t-k}]$ (last $k$ values as features)
- **Rolling statistics:** mean, std, min, max over a window
- **Trend / seasonality decomposition:** $x_t = \text{trend}_t + \text{seasonal}_t + \text{residual}_t$
- **FFT features:** power spectrum for periodic patterns

**Sequential models:** RNN, LSTM, Transformer — process the sequence directly without feature engineering.

---

### Audio

Raw audio: a 1D waveform sampled at, e.g., 44,100 Hz (44,100 values per second).

**Standard representation: MFCC (Mel-Frequency Cepstral Coefficients)**
1. Frame the signal into short windows (e.g., 25ms with 10ms hop).
2. Apply FFT to each frame → power spectrum.
3. Apply mel-scale filterbank → mel spectrum.
4. Take log.
5. Apply DCT → 13–40 cepstral coefficients per frame.

Result: a 2D matrix (frames × coefficients) — the standard input for speech models.

---

### Graphs

Nodes (entities) and edges (relationships). Examples: molecules, social networks, citation networks.

**Adjacency matrix:** $A \in \{0,1\}^{n \times n}$, $A_{ij} = 1$ if edge $(i,j)$ exists.

**Node feature matrix:** $X \in \mathbb{R}^{n \times d}$, row $i$ = feature vector of node $i$.

**Graph Neural Networks (GNNs)** learn representations by aggregating neighbour features:
$$h_v^{(l+1)} = \sigma\!\left(W^{(l)}\text{AGG}\!\left(\{h_u^{(l)} : u \in \mathcal{N}(v)\}\right)\right)$$

---

## The Representation Learning Revolution

Deep learning learns representations **automatically** from raw data:

| Input | Traditional Features | Deep Learning Features |
|---|---|---|
| Image | HOG, SIFT, colour histograms | CNN activations |
| Text | TF-IDF, n-grams | Transformer embeddings |
| Audio | MFCC | CNN/RNN on spectrogram |
| Graph | Handcrafted graph features | GNN node embeddings |

The learned representations are often far more powerful than hand-engineered ones because they are optimised end-to-end for the task.

---

## Connections

- [[Features]] — the result of representation
- [[Feature Vector]] — the final numerical form
- [[Feature Engineering]] — the manual version of creating good representations
- [[Training Data]] — the raw data being represented

---

## One-line Summary

> Data representation is the conversion of raw, heterogeneous data into numerical feature vectors — the quality of representation determines the ceiling of model performance, and the modern trend is to learn representations automatically via deep learning rather than engineering them by hand.
