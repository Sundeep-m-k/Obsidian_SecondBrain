# Paragraph Segmentation

tags: #nlp #segmentation #chunking #discourse #rag
links: [[Sentence Boundary Detection]] [[Discourse Structure]] [[RAG Overview]] [[Chunking Strategies]]

---

## Definition + Intuition

**Paragraph segmentation** identifies paragraph boundaries in text — grouping sentences into coherent topical or discourse units. More broadly, **text segmentation** covers any task of dividing a text into coherent sections: paragraphs, topics, chapters, or scenes.

> **Intuition**: A document is not just a sequence of sentences — it's a sequence of topics. Paragraph boundaries mark shifts in the discourse. For retrieval and summarisation, knowing which sentences belong to the same paragraph/topic is crucial — you want to retrieve coherent passages, not randomly split sentences.

---

## Key Properties / Types

### Why Paragraph Segmentation Matters

- **RAG chunking**: splitting documents into chunks for vector retrieval — bad chunking breaks coherent passages mid-thought
- **Summarisation**: summarise by paragraph rather than arbitrary windows
- **Topic modelling**: each paragraph is a natural document unit
- **Discourse analysis**: paragraph = rhetorical unit

### Approaches

**1. White-line detection**: paragraphs in plain text are often separated by blank lines. Simple but effective for structured documents.

```python
paragraphs = text.split('\n\n')
```

**2. Topic segmentation (TextTiling, Hearst 1997)**:
- Compute word similarity between adjacent sentence windows
- Paragraph boundaries = local minima in similarity curve

**3. Neural topic segmentation**:
- Models like BERT-based segmenters (e.g. Cross-Segment BERT)
- Each sentence pair classified as same or different segment

**4. Fixed-size chunking** (common in RAG):
```python
def chunk_text(text, chunk_size=200, overlap=50):
    tokens = tokenizer.encode(text)
    chunks = []
    for i in range(0, len(tokens), chunk_size - overlap):
        chunks.append(tokens[i:i + chunk_size])
    return chunks
```

**5. Semantic chunking** (advanced RAG):
- Embed each sentence
- Split where cosine similarity between adjacent sentences drops below threshold
- Produces semantically coherent chunks regardless of document structure

---

## Math / Formal Notation

### TextTiling Algorithm

1. Tokenize text into sentences; compute TF vector per sentence: $\vec{v}_i$
2. For each window boundary at position $k$, compute similarity between left and right windows:

$$\text{sim}(k) = \cos\left(\sum_{i \in W_\text{left}(k)} \vec{v}_i,\ \sum_{j \in W_\text{right}(k)} \vec{v}_j\right)$$

3. Compute depth score at valley $k$:

$$\text{depth}(k) = \frac{(\text{sim}(k_L) - \text{sim}(k)) + (\text{sim}(k_R) - \text{sim}(k))}{2}$$

where $k_L$, $k_R$ are the nearest peaks to the left and right.

4. Boundaries are placed at valleys where $\text{depth}(k) > \mu - \sigma$ (threshold based on mean and std of depths).

---

## Examples (Concrete)

**RAG chunking strategy comparison:**
```
Document: 500-word article about climate change

Fixed (200 token, 50 overlap):
  Chunk 1: "...carbon emissions have increased... Rising temperatures affect..."
  → May split "A new study shows that [CHUNK BOUNDARY] according to scientists..."
  → Bad: splits coherent sentence

Sentence-aware chunking:
  Chunk 1: [sentences 1-5] — ~150 tokens
  Chunk 2: [sentences 6-10] — ~160 tokens
  → Better: never splits mid-sentence

Semantic chunking:
  Paragraph 1: "Human activities and emissions..." (high internal similarity)
  Paragraph 2: "Policy responses include..." (topic shift detected)
  → Best: semantically coherent chunks
```

---

## How It Connects to ML / NLP

- **RAG systems**: chunking strategy is one of the most impactful hyperparameters for retrieval quality
- **Summarisation**: paragraph-level extractive summarisation selects high-scoring paragraphs
- **Long document processing**: splitting into paragraphs before encoding with BERT (max 512 tokens)

**Cross-links:**
- [[RAG Overview]] — chunking is a core RAG design decision
- [[Discourse Structure]] — paragraphs are the surface realisation of discourse segments
- [[3.ML & DL/1.Concepts/7.Search_Retrieval/Dense Retrieval]] — retrieval quality depends on chunk quality

---

## Further Reading

- Hearst, M.A. (1997). *TextTiling: Segmenting Text into Multi-Paragraph Subtopic Passages.* Computational Linguistics.
- Shi, T. & Keneshloo, Y. (2023). *Semantic Chunking for RAG.* — blog/technical report
