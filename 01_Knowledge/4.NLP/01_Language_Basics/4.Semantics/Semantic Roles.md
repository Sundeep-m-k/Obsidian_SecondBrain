# Semantic Roles

tags: #nlp #semantics #srl #thematic-roles #information-extraction
links: [[Lexical Semantics]] [[Compositional Semantics]] [[Frame Semantics]] [[Semantic Role Labeling]] [[Dependency Parsing]]

---

## Definition + Intuition

**Semantic roles** (also called **thematic roles** or **theta roles**) describe the abstract relationship between a predicate (usually a verb) and its arguments — the *roles* participants play in an event.

Semantic roles answer: "In this event, who is the **doer**? Who is the **receiver**? What was **affected**? Where did it happen?"

> **Intuition**: The sentence "The wind broke the window" and "John broke the window" both have "broke" as the predicate. The window plays the same role in both (it gets broken — the PATIENT). The wind and John play the same role (they cause the breaking — the AGENT or CAUSE). Semantic roles abstract away from syntactic surface form to capture these stable participant relationships.

---

## Key Properties / Types

### Core Semantic Roles (Proto-Roles / Thematic Roles)

| Role | Description | Example |
|------|-------------|---------|
| **Agent** | Intentional doer of the action | **John** broke the window |
| **Patient / Theme** | Entity that undergoes change or is moved | John broke **the window** |
| **Experiencer** | Entity that experiences a mental state | **Mary** feared the dog |
| **Stimulus** | What causes the mental state | Mary feared **the dog** |
| **Goal** | Destination or intended recipient | Mary gave the book **to John** |
| **Source** | Origin point | Mary came **from Paris** |
| **Location** | Where the action occurs | John sat **in the garden** |
| **Instrument** | Tool used by agent | John broke the window **with a hammer** |
| **Beneficiary** | Who benefits from the action | John baked a cake **for Mary** |
| **Cause** | Non-intentional cause | **The wind** broke the window |

### Linking Syntax to Semantics

Semantic roles systematically map to syntactic positions, but not deterministically:

**Agent alternation** (same role, different syntactic positions):
```
"John broke the window."  → John = Agent (subject)
"The window was broken by John."  → John = Agent (oblique)
Same agent role, different syntactic realization
```

**Dative alternation**:
```
"John gave Mary the book."   → Mary = Goal (indirect object)
"John gave the book to Mary." → Mary = Goal (oblique PP)
Same semantic role, two syntactic frames
```

### FrameNet vs PropBank

Two major annotation schemes for semantic roles:

| Feature | PropBank | FrameNet |
|---------|----------|---------|
| Role labels | Arg0, Arg1, ..., ArgM | Frame-specific: Buyer, Seller, Goods |
| Basis | Per-verb sense | Conceptual frames |
| Granularity | Coarse | Fine-grained |
| Coverage | High (common verbs) | Lower but richer |
| Use in NLP | Standard for SRL | Knowledge-rich applications |

---

## Math / Formal Notation

**Semantic Role Labeling (SRL)** as sequence/span labeling:

Given sentence $w_1, \ldots, w_n$ and predicate at position $p$, assign each token (or span) a role label $r \in \mathcal{R} \cup \{\text{O}\}$:

$$P(r_1, \ldots, r_n \mid w_1, \ldots, w_n, p)$$

**Neo-Davidsonian event semantics** formalizes roles as relations over an event variable:

"John broke the window with a hammer":
$$\exists e [\text{break}(e) \wedge \text{Agent}(e, \text{John}) \wedge \text{Patient}(e, \text{window}) \wedge \text{Instrument}(e, \text{hammer})]$$

This representation is **modular**: each role is a separate conjunct, so optional arguments (instrument, location) can be omitted without changing the core predication.

**Levin's verb classes**: verbs with the same argument alternation patterns share semantic roles. This generalization underlies the VerbNet lexical resource and guides SRL system design.

---

## Examples (Concrete)

**PropBank annotation:**
```
"The company acquired three smaller firms last year."

        V = acquired
Arg0 = The company        (acquirer / Agent)
Arg1 = three smaller firms (thing acquired / Theme)
ArgM-TMP = last year      (temporal modifier)
```

**Same semantic roles, different syntactic forms:**
```
Active:  "The dog [Arg0/Agent] bit [V] the man [Arg1/Patient]."
Passive: "The man [Arg1/Patient] was bitten [V] by the dog [Arg0/Agent]."
```

**Instrument appearing in different positions:**
```
"John [Agent] broke [V] the window [Patient] with a hammer [Instrument]."
"A hammer [Instrument] broke [V] the window [Patient]."
"John [Agent] used a hammer [Instrument] to break [V] the window [Patient]."
```

---

## How It Connects to ML / NLP

| Semantic Role Concept | NLP/ML Application |
|---|---|
| SRL | Information extraction; QA; summarization |
| Event variable representation | Knowledge graph population; event extraction |
| PropBank | Training data for SRL systems |
| Role consistency | Coreference resolution; event tracking |
| Verb alternations | Data augmentation; paraphrase generation |

**SRL in the NLP pipeline:**
```
"Who broke what?" → SRL identifies Agent + Patient of "break" → answers "who" and "what"
```

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — SRL as span labeling
- [[Dependency Parsing]] — dependency parse provides structural features for SRL

---

## Further Reading / Paper References

- Palmer, M., Gildea, D. & Kingsbury, P. (2005). *The Proposition Bank: An Annotated Corpus of Semantic Roles.* Computational Linguistics. — PropBank
- Baker, C.F., Fillmore, C.J. & Lowe, J.B. (1998). *The Berkeley FrameNet Project.* COLING-ACL. — FrameNet
- Shi, P. & Lin, J. (2019). *Simple BERT Models for Relation Extraction and Semantic Role Labeling.* [[arxiv:1904.05255]]
- Jurafsky & Martin, *Speech and Language Processing* Ch. 19 — SRL
