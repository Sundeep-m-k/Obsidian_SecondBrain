# Parse Trees

tags: #nlp #syntax #parsing #tree-structures #cfg
links: [[Constituency vs Dependency]] [[Grammar Formalisms]] [[CFG and PCFG]] [[Head-Driven Phrase Structure]]

---

## Definition + Intuition

A **parse tree** (also called a **syntax tree** or **phrase structure tree**) is a rooted, labeled tree that represents the syntactic structure of a sentence according to a grammar.

- **Constituency parse tree**: nodes are phrases and words; shows grouping structure.
- **Dependency parse tree**: nodes are words; directed arcs are grammatical relations.

> **Intuition**: A parse tree is to a sentence what a folder hierarchy is to a file system. Just as files are grouped into folders, grouped into top-level directories, words are grouped into phrases, grouped into larger phrases, up to the root node (S). The tree makes explicit which words "belong together" and what role each group plays.

---

## Key Properties / Types

### Anatomy of a Constituency Parse Tree

```
            S
          /   \
        NP     VP
       / \    /  \
     DT  NN VBD   NP
     |   |   |   / \
    The dog bit DT  NN
                |   |
               the  cat
```

**Nodes:**
- **Root node** (`S`): represents the full sentence
- **Internal nodes**: non-terminal labels (`NP`, `VP`, `PP`, etc.)
- **Leaf nodes**: terminal words (actual tokens)
- **Pre-terminal nodes**: POS tags (directly above words)

**Relationships:**
- **Parent**: the node immediately above
- **Children**: nodes immediately below
- **Dominance**: node A dominates B if there is a path from A to B
- **Immediate dominance**: A immediately dominates B if A is the parent of B
- **Precedence**: word A precedes word B if A is to the left in the tree

### Tree Properties
- **Depth**: length of the longest root-to-leaf path
- **Branching factor**: number of children per node
- **Span**: the set of words dominated by a node
  - Span of root = whole sentence
  - Span of NP "the dog" = {The, dog}

### Special Tree Types
- **Binary tree**: each internal node has exactly 2 children (Chomsky Normal Form)
- **Right-branching**: most structure is on the right (English coordination: "a and b and c")
- **Left-branching**: most structure is on the left (Japanese)
- **Flat tree**: all children of S in one level (no internal structure)
- **Binarized tree**: non-binary tree converted to binary for parsing efficiency

---

## Math / Formal Notation

### Formal Definition

A parse tree $T$ for sentence $w = w_1 w_2 \ldots w_n$ under CFG $G = (N, \Sigma, R, S)$ is:

$$T = (V_T, E_T, r, \ell)$$

where:
- $V_T$ = nodes
- $E_T \subseteq V_T \times V_T$ = edges (parent → child)
- $r \in V_T$ = root node (labeled $S$)
- $\ell: V_T \rightarrow N \cup \Sigma$ = labeling function

Validity conditions:
1. $\ell(r) = S$ (root is start symbol)
2. Each internal node $v$ with label $A$ and children $v_1, \ldots, v_k$ corresponds to a rule $A \rightarrow \ell(v_1) \ldots \ell(v_k) \in R$
3. Leaf nodes are labeled with terminals: $\ell(v) \in \Sigma$
4. Left-to-right ordering of leaves = $w_1 w_2 \ldots w_n$

### CYK Algorithm (Chart Parsing)

The **Cocke-Younger-Kasami (CYK)** algorithm fills a triangular chart to find all parses:

For grammar in **Chomsky Normal Form (CNF)**: $A \rightarrow BC$ or $A \rightarrow w$

Chart entry: $\text{chart}[i][j][A]$ = True if nonterminal $A$ can generate span $w_i \ldots w_j$

**Base case** (span of length 1):
$$\text{chart}[i][i][A] = \text{True} \iff (A \rightarrow w_i) \in R$$

**Recursive case** (span $[i, j]$, length $j - i + 1 \geq 2$):
$$\text{chart}[i][j][A] = \bigvee_{k=i}^{j-1} \bigvee_{B,C : (A \rightarrow BC) \in R} \text{chart}[i][k][B] \wedge \text{chart}[k+1][j][C]$$

**Complexity**: $O(n^3 |G|)$ where $n$ = sentence length, $|G|$ = grammar size.

### PCFG: Probability of a Parse Tree

$$P(T) = \prod_{(A \rightarrow \alpha) \in T} P(A \rightarrow \alpha \mid A)$$

The **most likely parse** (Viterbi parse):
$$T^* = \arg\max_{T : \text{yield}(T) = w} P(T)$$

---

## Examples (Concrete)

### Example 1: Simple sentence
```
Sentence: "I saw the man"

         S
        / \
       NP   VP
       |   /  \
      PRP VBD   NP
       |   |   / \
       I  saw DT  NN
              |   |
             the  man
```

### Example 2: Ambiguous sentence (PP attachment ambiguity)
```
"I saw the man with the telescope"

Parse A: I [saw [the man] [with the telescope]]  -- I used telescope to see
         VP
        /  \
      VBD   NP
       |   /   \
      saw NP    PP
         / \    
        ...  "with the telescope"

Parse B: I [saw [the man [with the telescope]]]  -- man has telescope
         VP
        /  \
      VBD   NP
       |   /  \
      saw DT   N'
          |   /  \
         the  NN   PP
              |    |
             man  "with the telescope"
```

**Both are valid parses under the grammar** — structural ambiguity. The model must assign probabilities (PCFG) to select the more likely one, or use semantics to disambiguate.

### Example 3: Treebank notation (Penn Treebank)
```
(S (NP (DT The) (NN dog)) 
   (VP (VBD bit) 
       (NP (DT the) (NN cat))))
```
This bracket notation is how parse trees are stored in the Penn Treebank — the standard training/evaluation resource for constituency parsers.

---

## How It Connects to ML / NLP

| Parse Tree Concept | NLP/ML Application |
|---|---|
| Constituency span | Constituent-based coreference, syntactic features |
| Tree structure | Input to tree-structured neural networks (Tree-LSTM) |
| PCFG probabilities | Maximum likelihood estimation from treebanks |
| CYK algorithm | Dynamic programming — same principle as sequence alignment (Viterbi) |
| Penn Treebank | Standard training data for parser evaluation |
| Binarization (CNF) | Required for CYK; analogous to normalization in ML |

**Tree-structured neural networks:**
Tree-LSTMs process sentences via their parse tree structure instead of left-to-right:
$$h_v = \tanh\left(W x_v + \sum_{c \in \text{children}(v)} U h_c + b\right)$$

This captures compositional structure explicitly — useful for tasks like sentiment analysis (knowing that "not bad" is a negated positive phrase).

**Syntax in transformers:**
Transformers don't use explicit parse trees, but:
- Attention patterns in early layers of BERT correlate with syntactic dependencies
- Syntax-aware pretraining (e.g. BERT with constituency labels) improves some tasks
- Linearized parse trees can be used as additional input to seq2seq models

**Cross-links:**
- [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] — PCFG inside-outside algorithm is analogous to forward-backward
- [[3.ML & DL/1.Concepts/1.Foundations/Model.md]] — the grammar is the model; parse tree is the output
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — cross-entropy over tree productions for PCFG training

---

## Common Interview Questions

**Q: What is the time complexity of CYK parsing and why?**
A: O(n³ · |G|) where n is sentence length. We fill an n×n triangle of chart cells; each cell [i,j] requires considering all split points k (O(n)), and all grammar rules (O(|G|)). The cubic dependence on length is the bottleneck for long sentences — modern neural parsers (shift-reduce) achieve O(n) or O(n log n).

**Q: What is structural ambiguity? Give an example.**
A: A sentence has structural ambiguity when there are multiple valid parse trees. Classic example: "I saw the man with the telescope" — either I used a telescope (PP modifies VP) or the man has a telescope (PP modifies NP). Both are syntactically valid. Humans resolve this using world knowledge; NLP systems use PCFG probabilities or semantic models.

**Q: How are parse trees used in modern transformer-based NLP?**
A: Mostly implicitly. Transformers learn some syntactic structure through self-supervised training. Explicit parse trees are used in: (1) tree-structured models (Tree-LSTM), (2) syntax-augmented pretraining, (3) syntax features for specific tasks (SRL, coreference), and (4) evaluation — comparing predicted syntactic structure to gold treebank parses.

---

## Common Mistakes / Gotchas

- **Assuming one parse = one meaning**: A syntactically correct parse does not guarantee semantic coherence. "Colorless green ideas sleep furiously" is perfectly parseable but semantically nonsensical (Chomsky's famous example).
- **CNF conversion changes the tree**: Chomsky Normal Form requires binarization — internal nodes may be added that have no linguistic interpretation. These `X-bar` intermediate nodes need to be removed when reporting results.
- **Penn Treebank is English-specific**: PTB tagset (NN, VBD, DT...) and phrase labels are not universal. Do not use PTB-style parsers for non-English text without adaptation.
- **Confusing POS tags and phrase labels**: `NN` (noun, singular) is a POS tag at the pre-terminal level. `NP` is a phrase label at the internal node level. They occupy different levels of the tree.

---

## Further Reading / Paper References

- Chomsky, N. (1965). *Aspects of the Theory of Syntax.* — formal phrase structure theory
- Marcus, M.P. et al. (1993). *Building a Large Annotated Corpus of English: The Penn Treebank.* Computational Linguistics, 19(2). — standard treebank
- Klein, D. & Manning, C.D. (2003). *Accurate Unlexicalized Parsing.* ACL. — PCFG parsing
- Tai, K.S., Socher, R. & Manning, C.D. (2015). *Improved Semantic Representations from Tree-Structured LSTM.* [[arxiv:1503.00075]]
- Kitaev, N. & Klein, D. (2018). *Constituency Parsing with a Self-Attentive Encoder.* [[arxiv:1805.01052]] — BERT-based parser
- Jurafsky & Martin, *Speech and Language Processing* Ch. 12–13
