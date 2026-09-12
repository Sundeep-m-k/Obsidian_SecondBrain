# CFG and PCFG

tags: #nlp #syntax #formal-grammars #parsing #probability
links: [[Parse Trees]] [[Constituency vs Dependency]] [[Grammar Formalisms]] [[Head-Driven Phrase Structure]]

---

## Definition + Intuition

A **Context-Free Grammar (CFG)** is a formal grammar that defines a language as a set of strings generatable by applying rewriting rules to a start symbol. It is "context-free" because rules apply regardless of what surrounds the symbol being rewritten.

A **Probabilistic CFG (PCFG)** augments each rule with a probability, turning the grammar into a generative probabilistic model over parse trees (and thereby over sentences).

> **Intuition**:
> A CFG is like a recipe book where each recipe (rule) says "an NP can be made of DT + NN" — and you apply recipes recursively until you've produced all the words. A PCFG adds: "and this recipe is chosen 40% of the time vs other NP recipes." This lets you pick the most likely recipe sequence (parse) for a given sentence.

---

## Key Properties / Types

### CFG Formal Definition

$$G = (N,\ \Sigma,\ R,\ S)$$

| Component | Symbol | Description |
|-----------|--------|-------------|
| Nonterminals | $N$ | Phrase labels: {S, NP, VP, PP, DT, NN, VBD, ...} |
| Terminals | $\Sigma$ | Words: {the, cat, bit, ...} |
| Rules | $R$ | Productions: $A \rightarrow \alpha$ |
| Start symbol | $S$ | Root nonterminal |

**Rule types:**
- $A \rightarrow B\ C$ (binary branching)
- $A \rightarrow B$ (unary)
- $A \rightarrow w$ (lexical — A is a pre-terminal, w is a word)

**Example grammar:**
```
S   → NP VP
NP  → DT NN | DT JJ NN | NP PP | PRP
VP  → VBD NP | VBD NP PP | VBZ NP
PP  → IN NP
DT  → "the" | "a"
NN  → "dog" | "cat" | "man" | "telescope"
VBD → "saw" | "bit"
IN  → "with" | "on"
PRP → "I" | "he"
JJ  → "big" | "old"
```

### CFG Limitations
- **Cannot count**: CFGs can generate $a^n b^n$ but not $a^n b^n c^n$ — some cross-serial dependencies in natural language exceed CFG power (Swiss German verb clusters).
- **Context-free**: the rule for NP fires the same regardless of whether NP is a subject or object — but subjects and objects sometimes require different forms (case, agreement).
- **Structural ambiguity**: many sentences have multiple valid parse trees.

### PCFG

Each rule $A \rightarrow \alpha$ has probability $P(A \rightarrow \alpha \mid A)$, with:

$$\sum_{\alpha : (A \rightarrow \alpha) \in R} P(A \rightarrow \alpha \mid A) = 1 \quad \forall A \in N$$

(The probabilities for all expansions of a given nonterminal sum to 1.)

**PCFG generation**: to generate a sentence, start from S and at each step sample a rule according to its probability.

**Probability of a complete parse tree:**
$$P(T) = \prod_{(A \rightarrow \alpha) \in T} P(A \rightarrow \alpha \mid A)$$

---

## Math / Formal Notation

### PCFG Training (MLE from Treebank)

Given a treebank $\mathcal{T}$ (collection of annotated parse trees):

$$\hat{P}(A \rightarrow \alpha \mid A) = \frac{\text{count}(A \rightarrow \alpha)}{\text{count}(A)} = \frac{\text{count}(A \rightarrow \alpha)}{\sum_\beta \text{count}(A \rightarrow \beta)}$$

This is the standard MLE formula for multinomials — exactly the same as [[3.ML & DL/1.Concepts/4.Loss and Cost/Cost Function.md]] minimization with cross-entropy.

### Parsing with PCFG — Probabilistic CYK

A probabilistic chart, where $\pi[i][j][A]$ = the probability of the best parse of span $[i, j]$ as nonterminal $A$:

**Base case:**
$$\pi[i][i][A] = P(A \rightarrow w_i \mid A)$$

**Recursive case:**
$$\pi[i][j][A] = \max_{k, B, C} P(A \rightarrow B\ C) \cdot \pi[i][k][B] \cdot \pi[k+1][j][C]$$

**Time complexity**: $O(n^3 |N|^3)$ in the worst case (or $O(n^3 |R|)$).

The **most probable parse** is recovered by backtracing through the chart.

### Inside-Outside Algorithm (Forward-Backward for Trees)

The **inside probability** $\beta(A, i, j)$ = probability that nonterminal $A$ generates the span $[i, j]$:
$$\beta(A, i, j) = \sum_{k, B, C} P(A \rightarrow B\ C) \cdot \beta(B, i, k) \cdot \beta(C, k+1, j)$$

The **outside probability** $\alpha(A, i, j)$ = probability of the rest of the sentence given $A$ generates $[i,j]$.

Together, they enable **EM training** of PCFG parameters from unannotated text (unsupervised PCFG learning).

### Chomsky Normal Form (CNF)

CYK requires CNF: every rule is either $A \rightarrow B\ C$ or $A \rightarrow w$.

**Conversion steps:**
1. **Remove $\epsilon$-productions** (nullable rules)
2. **Remove unit rules** ($A \rightarrow B$) via closure
3. **Binarize** rules with $> 2$ RHS symbols: $A \rightarrow B\ C\ D$ becomes $A \rightarrow B\ X_{BC};\ X_{BC} \rightarrow C\ D$
4. **Factor out terminals** in mixed rules: $A \rightarrow B\ w$ becomes $A \rightarrow B\ X_w;\ X_w \rightarrow w$

---

## Examples (Concrete)

### Computing Parse Tree Probability

Grammar rules with probabilities:
```
S   → NP VP           P = 1.0
NP  → DT NN           P = 0.7
NP  → PRP             P = 0.3
VP  → VBD NP          P = 0.6
VP  → VBD             P = 0.4
DT  → "the"           P = 1.0
NN  → "dog"           P = 0.5
NN  → "cat"           P = 0.5
VBD → "bit"           P = 1.0
PRP → "I"             P = 1.0
```

Parse of "The dog bit the cat":
```
P(T) = P(S→NP VP) × P(NP→DT NN) × P(DT→the) × P(NN→dog)
       × P(VP→VBD NP) × P(VBD→bit) × P(NP→DT NN) × P(DT→the) × P(NN→cat)
     = 1.0 × 0.7 × 1.0 × 0.5 × 0.6 × 1.0 × 0.7 × 1.0 × 0.5
     = 0.0735
```

### Ambiguity Resolution with PCFG

"I saw the man with the telescope" — two parses:
- Parse A (VP attachment): $P(A) = 0.04$
- Parse B (NP attachment): $P(B) = 0.06$

PCFG selects Parse B (man has telescope) as more probable. Whether this is correct depends on context — but corpus-trained probabilities generally prefer the statistically more common attachment.

---

## How It Connects to ML / NLP

| CFG/PCFG Concept | ML Connection |
|---|---|
| Rule probabilities | Multinomial distribution; MLE from counts |
| Inside-outside | Expectation-Maximization (EM) algorithm |
| Viterbi parse | Viterbi decoding = dynamic programming for best sequence |
| PCFG training | Supervised (treebank MLE) or unsupervised (EM) |
| Ambiguity | Multiple hypotheses — like beam search in seq2seq |
| CNF binarization | Normalization step like feature scaling |

**Modern neural parsing replaces PCFG with:**
- Encoder (BERT) produces contextual representations
- Span scoring: $f(i, j, A) =$ score of span $[i,j]$ being labeled $A$
- Chart-based decoder fills the CYK chart using neural scores instead of PCFG probabilities

This retains the cubic CYK algorithm but replaces the handcrafted grammar with learned neural scores — [[3.ML & DL/1.Concepts/5.Optimization/Gradient Descent.md]] trains the encoder.

**Cross-links:**
- [[3.ML & DL/1.Concepts/4.Loss and Cost/Loss Function.md]] — PCFG training objective = MLE = cross-entropy
- [[3.ML & DL/1.Concepts/1.Foundations/Model.md]] — PCFG is a generative model
- [[3.ML & DL/1.Concepts/5.Optimization/Convergence.md]] — inside-outside EM convergence

---

## Common Interview Questions

**Q: Why did PCFG parsing fall out of favor for neural methods?**
A: PCFGs have several weaknesses: (1) independence assumption — rule probabilities don't depend on lexical content, so "dog bit cat" and "idea bit cat" have the same parse probability; (2) lack of lexicalization — head word information is crucial for PP attachment but ignored in basic PCFG; (3) grammar coverage — PCFGs require careful engineering. Neural parsers (span-based, BERT-based) learn to score all these factors jointly from data.

**Q: What is the difference between the inside algorithm and the Viterbi algorithm?**
A: Both are dynamic programming algorithms over structured outputs. Inside computes the *total probability* of a span (sum over all parses) — used for marginalizing and EM training. Viterbi computes the *maximum probability* parse — used for decoding. The inside algorithm is to PCFG what the forward algorithm is to HMMs; Viterbi is Viterbi in both cases.

**Q: What makes a grammar context-free? When is it not enough?**
A: Context-free means a rule fires regardless of context — the rewriting of A does not depend on what surrounds A. This fails for: (1) Swiss German cross-serial verb dependencies (require mildly context-sensitive grammars), (2) agreement across long distances that CFGs can't track. Tree-Adjoining Grammars (TAGs) and Minimalist Grammars extend CFGs for these cases.

---

## Common Mistakes / Gotchas

- **Rule probabilities don't sum to 1 globally**: they sum to 1 per nonterminal. $\sum_\alpha P(A \rightarrow \alpha \mid A) = 1$ for each $A$, but $\sum_{A, \alpha} P(A \rightarrow \alpha) \neq 1$.
- **Inside probability ≠ parse probability**: the inside probability sums over all parses; the tree probability is the product of rules in one specific tree.
- **CNF changes tree structure**: intermediate nodes introduced during binarization (e.g. `@NP→DT+NN`) must be collapsed before reporting F1 scores on the original tree.
- **Overfitting to treebank**: PCFG trained on Penn Treebank overfits to WSJ newswire. Parsing performance drops ~10 F1 on out-of-domain text (questions, social media).

---

## Further Reading / Paper References

- Hopcroft, J., Motwani, R. & Ullman, J. *Introduction to Automata Theory, Languages, and Computation.* — formal CFG theory
- Jelinek, F. et al. (1990). *Perplexity — A Measure of the Difficulty of Speech Recognition Tasks.* — early PCFG language modeling
- Collins, M. (1997). *Three Generative, Lexicalised Models for Statistical Parsing.* ACL. — lexicalized PCFGs
- Lari, K. & Young, S.J. (1990). *The Estimation of Stochastic Context-Free Grammars Using the Inside-Outside Algorithm.* Computer Speech and Language.
- Kitaev, N. & Klein, D. (2018). *Constituency Parsing with a Self-Attentive Encoder.* [[arxiv:1805.01052]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 12–13
