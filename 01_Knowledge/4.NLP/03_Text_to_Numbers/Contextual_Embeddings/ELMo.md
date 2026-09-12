---
tags: [nlp, contextual-embeddings, elmo, lstm, language-model, transfer-learning, text-to-numbers]
links: "[[03_Text_to_Numbers/Index]] [[BERT Embeddings]] [[Word2Vec]] [[GloVe]] [[Sequential Models]]"
---

# ELMo

## Definition + Intuition

**ELMo (Embeddings from Language Models)** (Peters et al., 2018) generates word representations that are functions of the **entire input sentence**, not the word in isolation. Unlike Word2Vec or GloVe where "bank" always has the same vector, ELMo produces a different vector for "bank" in "river bank" vs. "bank account" — because it runs a deep bidirectional LSTM over the entire sentence and uses all layers' outputs as the embedding.

**Intuition**: Imagine reading a sentence. When you encounter the word "bank", you immediately use all surrounding words to disambiguate its meaning. ELMo does the same — it reads the whole sentence left-to-right and right-to-left using LSTM language models, then combines the internal states at each layer for each token as its contextual representation.

**Historical significance**: ELMo (2018) was the first demonstration that **pre-trained language model representations** transfer broadly across NLP tasks. It achieved new state-of-the-art on 6 NLP benchmarks simultaneously, launching the era of contextual pre-training. BERT (2018) superseded it but follows the same paradigm.

---

## Key Properties / Types

**Architecture**:

ELMo uses a **2-layer biLSTM** language model:

```
Input character CNN → Token embedding
     ↓
Forward LSTM layer 1 → hidden states h→₁
Backward LSTM layer 1 → hidden states h←₁
     ↓
Forward LSTM layer 2 → hidden states h→₂
Backward LSTM layer 2 → hidden states h←₂
```

**Three layers of representations** for each token $t_k$:
- Layer 0: Character CNN token embedding (context-independent)
- Layer 1: First biLSTM layer (syntax-sensitive)
- Layer 2: Second biLSTM layer (semantics-sensitive — solves WSD)

**ELMo vector for token $k$**:
$$\text{ELMo}_k^{\text{task}} = \gamma^{\text{task}} \sum_{j=0}^{L} s_j^{\text{task}} \cdot h_{k,j}^{LM}$$

where:
- $h_{k,j}^{LM}$ = concatenated [forward, backward] LSTM hidden state at layer $j$ for token $k$
- $s_j^{\text{task}}$ = learned **softmax-normalised weights** per task (which layer matters most?)
- $\gamma^{\text{task}}$ = learned scalar scaling the entire ELMo vector

The task-specific weights $s_j$ are learned during fine-tuning — different NLP tasks weight the layers differently.

**Character CNN input** (key detail): ELMo uses a character-level CNN to produce the initial token embedding. This gives ELMo natural OOV handling — any word, however rare, can be embedded from its characters. This is unlike Word2Vec/GloVe which fail on OOV.

---

## Math / Formal Notation

**Forward language model objective**:
$$\mathcal{L}_{\text{fwd}} = -\sum_{k=1}^{N} \log P(t_k \mid t_1, \ldots, t_{k-1}; \Theta_e, \overrightarrow{\Theta}_{LSTM}, \Theta_s)$$

**Backward language model objective**:
$$\mathcal{L}_{\text{bwd}} = -\sum_{k=1}^{N} \log P(t_k \mid t_{k+1}, \ldots, t_N; \Theta_e, \overleftarrow{\Theta}_{LSTM}, \Theta_s)$$

**Combined biLM objective** (parameters $\Theta_e$, $\Theta_s$ shared; LSTM weights separate):
$$\mathcal{L} = \mathcal{L}_{\text{fwd}} + \mathcal{L}_{\text{bwd}}$$

**ELMo representation** for token $k$, $L$ biLSTM layers:

For each layer $j$, the representation is the concatenation of forward and backward states:
$$h_{k,j}^{LM} = [\overrightarrow{h}_{k,j}^{LM}; \overleftarrow{h}_{k,j}^{LM}]$$

The task-conditioned ELMo vector:
$$\text{ELMo}_k = \gamma \sum_{j=0}^{L} \text{softmax}(\mathbf{s})_j \cdot h_{k,j}^{LM}$$

**Key insight from probing experiments**:
- Lower layers (layer 1) encode **syntax** — POS tags, dependency structure
- Higher layers (layer 2) encode **semantics** — word sense, coreference
- Tasks requiring syntax (NER, POS) weight layer 1 more; tasks requiring semantics (WSD, NLI) weight layer 2 more

---

## Examples (Concrete)

**Polysemy resolution**:

Sentence A: "I went to the **bank** to deposit money."
Sentence B: "We sat on the **bank** of the river."

Word2Vec/GloVe: `embed("bank")` = same vector for both → cannot distinguish.

ELMo: 
- Sentence A → `bank` embedding is close to "account", "deposit", "finance"
- Sentence B → `bank` embedding is close to "river", "shore", "water"

The biLSTM context propagates surrounding words through the hidden states, changing the "bank" representation.

**Using ELMo (AllenNLP)**:
```python
from allennlp.modules.elmo import Elmo, batch_to_ids

options_file = "https://allennlp.s3.amazonaws.com/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_options.json"
weight_file = "https://allennlp.s3.amazonaws.com/models/elmo/2x4096_512_2048cnn_2xhighway/elmo_2x4096_512_2048cnn_2xhighway_weights.hdf5"

# num_output_representations=1: one ELMo vector per token
elmo = Elmo(options_file, weight_file, num_output_representations=1)

sentences = [["I", "went", "to", "the", "bank"],
             ["The", "river", "bank", "was", "muddy"]]
character_ids = batch_to_ids(sentences)

embeddings = elmo(character_ids)
# embeddings['elmo_representations'][0].shape = (2, 5, 1024)
# 2 sentences, 5 tokens, 1024-dim ELMo vector
```

**Task-specific ELMo integration** (pseudocode):
```python
# Freeze ELMo, learn task weights s_j and scale γ during fine-tuning
elmo_layer_0 = elmo_encoder(sentence, layer=0)  # char CNN
elmo_layer_1 = elmo_encoder(sentence, layer=1)  # biLSTM 1
elmo_layer_2 = elmo_encoder(sentence, layer=2)  # biLSTM 2

# s = softmax([s0, s1, s2]) — learned per task
elmo_vector = gamma * (s[0]*elmo_layer_0 + s[1]*elmo_layer_1 + s[2]*elmo_layer_2)

# Feed to task model (e.g., CRF for NER)
output = task_model(elmo_vector)
```

---

## How It Connects to ML / NLP

| ELMo Concept | ML/NLP Link |
|-------------|-------------|
| Pre-training on language modelling | [[3.ML & DL/1.Concepts/1.Foundations/Unsupervised Learning.md]] — self-supervised training signal from raw text |
| Transfer learning to downstream tasks | [[3.ML & DL/1.Concepts/1.Foundations/Generalization.md]] — representations learned on one task generalise to others |
| Feature extraction (frozen ELMo) | [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — ELMo as feature extractor, task model trained on top |
| Learned layer weights $s_j$ | [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — $s_j$ and $\gamma$ learned via task-specific backprop |
| Softmax over layer weights | [[3.ML & DL/1.Concepts/7.Logistic Regression/Sigmoid Function.md]] — softmax is multi-class generalisation of sigmoid |
| biLSTM = Sequential model | [[Sequential Models]] — ELMo's backbone is a stacked bidirectional LSTM |
| Character CNN input | [[Character Embeddings]] — OOV-robust token representation |
| Layer 1 ≈ syntax, Layer 2 ≈ semantics | [[3.ML & DL/1.Concepts/8.Model Behavior/Model Complexity.md]] — depth encodes different levels of abstraction |

---

## Common Interview Questions

**Q: How does ELMo solve the polysemy problem that Word2Vec cannot?**
A: ELMo runs a biLSTM over the entire sentence, producing hidden states that integrate contextual information from all surrounding words. The same word token gets a different ELMo embedding in different sentences because the LSTM hidden states reflect the full sequence context. Word2Vec maps each word type to one fixed vector — polysemy is invisible.

**Q: What does "feature extraction" mean in the context of ELMo?**
A: ELMo parameters are frozen after pre-training. The pre-trained ELMo embeddings are extracted and used as input features to a task-specific model (CRF for NER, MLP for classification). The task model is trained; ELMo is not updated. This contrasts with BERT fine-tuning where the entire pre-trained model is updated.

**Q: Why are there task-specific layer weights in ELMo?**
A: Different NLP tasks use different levels of linguistic abstraction. Syntax-heavy tasks (POS, NER) benefit more from lower LSTM layers which encode local word patterns. Semantics-heavy tasks (WSD, NLI) benefit more from higher layers which integrate broader context. The learned softmax weights $s_j$ let the model automatically select the most informative layer combination for each task.

**Q: How does ELMo handle OOV words?**
A: ELMo uses a character-level CNN to produce the initial token embedding (layer 0). This means any word, regardless of whether it was seen during training, can be embedded from its character n-grams. This is a major advantage over Word2Vec and GloVe.

**Q: How does ELMo differ from BERT?**
A: (1) Architecture: ELMo uses stacked biLSTMs; BERT uses Transformers. (2) Bidirectionality: ELMo concatenates separate forward and backward LSTMs — they don't attend to each other. BERT's Transformer uses full self-attention — every token attends to all others simultaneously. (3) Pre-training: ELMo uses standard LM (predict next/previous word); BERT uses Masked LM (predict masked tokens). (4) Usage: ELMo is typically used as a feature extractor; BERT is typically fine-tuned end-to-end.

---

## Common Mistakes / Gotchas

- **Using ELMo as a drop-in for static embeddings**: ELMo requires a full sentence as input — you cannot embed isolated words without context. Always pass complete sentences.
- **Forgetting that ELMo is slow**: Running biLSTMs over sequences is significantly slower than lookup in a static embedding table. For latency-sensitive applications, cache ELMo representations or use simpler embeddings.
- **Not using all layers**: Using only the top layer (layer 2) loses the syntactic information captured in layer 1. Always compute the weighted combination unless you have a strong reason to use a single layer.
- **BERT has largely superseded ELMo**: For new projects, BERT/RoBERTa/DeBERTa are strongly preferred. ELMo is primarily relevant for understanding the history of contextual embeddings and for resource-constrained settings where Transformers are too slow.
- **ELMo is bidirectional but not jointly**: The forward and backward LSTMs are trained separately, concatenated. BERT's full self-attention is jointly bidirectional — fundamentally more expressive.

---

## Further Reading / Paper References

- Peters et al. (2018). Deep Contextualized Word Representations (ELMo) arxiv:1802.05365 — the paper
- Peters et al. (2018). Dissecting Contextual Word Embeddings: Architecture and Representation — probing analysis of ELMo layers
- Devlin et al. (2018). BERT: Pre-training of Deep Bidirectional Transformers — successor to ELMo arxiv:1810.04805
- AllenNLP ELMo documentation: https://allennlp.org/elmo
- Jurafsky & Martin, SLP Ch. 11 — Transfer Learning with Pre-trained Language Models
