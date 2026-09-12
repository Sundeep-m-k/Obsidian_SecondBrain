# Frame Semantics

tags: #nlp #semantics #framenet #knowledge-representation #srl
links: [[Semantic Roles]] [[Lexical Semantics]] [[Compositional Semantics]] [[Word Sense and Polysemy]]

---

## Definition + Intuition

**Frame semantics** (Fillmore 1976, 1982) is a theory of linguistic meaning based on the idea that words evoke *conceptual frames* — structured knowledge about stereotypical situations, events, and relations.

A **frame** is a background knowledge structure that provides the context for interpreting a word's meaning. Each frame has **frame elements** (FEs) — the participants and props in the scenario.

> **Intuition**: The word "buy" doesn't just mean a transaction — it evokes an entire commercial transaction frame: there's a Buyer, a Seller, Goods, a Price, and a Place. When you say "I bought a book," the word "bought" activates all of this background knowledge, even though most of it is unstated. Frame semantics says: **you cannot understand the meaning of a word without knowing the frame it belongs to**.

---

## Key Properties / Types

### Frame Elements (Roles within a Frame)

**Commercial Transaction frame:**
```
Frame: COMMERCE_BUY
Core FEs:
  Buyer:  the one who pays
  Seller: the one who receives payment
  Goods:  what is bought
  Money:  the price paid
Non-core FEs:
  Place:  where the transaction occurs
  Time:   when it occurred
  Means:  how payment was made

Lexical Units evoking this frame:
  buy, purchase, acquire, pick up, get, obtain, procure
```

Each lexical unit that evokes the frame is a **lexical unit (LU)**. Different LUs emphasize different perspectives:
- `buy` profiles the Buyer's perspective
- `sell` profiles the Seller's perspective
- `cost` profiles the Money/Goods relation
- Same frame, different profiling

### Frame Inheritance

Frames organize into inheritance hierarchies:
```
EVENT
  └── TRANSITIVE_ACTION
         └── CAUSE_CHANGE
               └── COMMERCE
                     └── COMMERCE_BUY
                     └── COMMERCE_SELL
                     └── COMMERCE_PAY
```

Subframes inherit FEs from parent frames and add specific ones.

### FrameNet
FrameNet is the lexical database implementing frame semantics:
- ~1,200 frames
- ~13,000 lexical units
- ~200,000 annotated example sentences
- Available at https://framenet.icsi.berkeley.edu

---

## Math / Formal Notation

**Frame assignment** as a classification problem:

Given target word $w$ in context $c$:
$$f^* = \arg\max_{f \in \mathcal{F}} P(f \mid w, c)$$

This is similar to WSD but at the frame level.

**Frame element labeling** (conditional on frame):
$$P(\text{FE}_1, \ldots, \text{FE}_k \mid f^*, w_1, \ldots, w_n)$$

This is a span labeling problem — exactly like SRL with PropBank, but with frame-specific role labels instead of Arg0/Arg1/...

---

## Examples (Concrete)

**Annotated FrameNet sentence:**
```
"[The company]_Buyer bought [three firms]_Goods [last year]_Time."
Frame: COMMERCE_BUY
LU: bought.v
FEs: Buyer=The company, Goods=three firms, Time=last year
```

**Same frame, different perspectives:**
```
"[I]_Buyer bought [this car]_Goods from [the dealer]_Seller for [$20k]_Money."
"[The dealer]_Seller sold [this car]_Goods to [me]_Buyer for [$20k]_Money."
"[This car]_Goods cost [me]_Buyer [$20k]_Money."
"[I]_Buyer paid [$20k]_Money for [this car]_Goods."
All evoke COMMERCE frame; different LUs profile different participants.
```

---

## How It Connects to ML / NLP

| Frame Semantics Concept | NLP/ML Application |
|---|---|
| Frame | Knowledge representation for QA, IE |
| Frame element labeling | Fine-grained SRL; event extraction |
| FrameNet | Training/evaluation resource for semantic parsing |
| Frame inheritance | Transfer between related frames |
| Profiling (perspective) | Sentiment analysis (bias detection); framing analysis in media |

**"Framing" in NLP:**
Frame semantics inspired media framing analysis — how the same event is described differently depending on which frame (perspective) is activated. "Terrorists killed civilians" vs "Freedom fighters targeted military infrastructure" — same event, different frames. NLP systems that detect framing bias draw on this theory.

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — frame classification
- [[Semantic Roles]] — PropBank is a simplified frame-semantic annotation

---

## Further Reading / Paper References

- Fillmore, C.J. (1982). "Frame Semantics." In *Linguistics in the Morning Calm.*
- Baker, C.F. et al. (1998). *The Berkeley FrameNet Project.* COLING-ACL.
- Das, D. et al. (2014). *Frame-Semantic Parsing.* Computational Linguistics.
- Jurafsky & Martin, *Speech and Language Processing* Ch. 19
