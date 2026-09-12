# Implicature and Grice

tags: #nlp #pragmatics #inference #grice #conversational-norms
links: [[Speech Acts]] [[Discourse Structure]] [[Coreference]] [[NLI]] [[Hallucination]]

---

## Definition + Intuition

**Implicature** (Grice 1975) is the meaning that is *conveyed but not literally said* in an utterance. It is the gap between what is said and what is communicated.

**Grice's Cooperative Principle**: speakers and listeners implicitly cooperate to make conversations work. Utterances are interpreted under the assumption that the speaker is being cooperative.

The Cooperative Principle is divided into **four Maxims**:

> **Intuition**:
> Q: "Did everyone pass the exam?"
> A: "Well, John passed."
>
> Literal content: John passed. But the implicature is: *not everyone passed* (otherwise why single out John?). This inference is drawn because of Gricean reasoning — if everyone had passed, the cooperative response would have been "Yes." The partial answer implies there's something the speaker is holding back.

---

## Key Properties / Types

### Grice's Maxims

| Maxim | Sub-maxims | Violation implicature |
|-------|-----------|----------------------|
| **Quantity** | 1. Be as informative as required. 2. Don't be more informative than required. | Saying less than needed implies there's a reason; saying more implies significance |
| **Quality** | 1. Don't say what you believe false. 2. Don't say what you lack evidence for. | Violations signal irony, sarcasm, hedging |
| **Relation** | Be relevant. | Apparent irrelevance triggers implicature (what is the relevance?) |
| **Manner** | 1. Avoid obscurity. 2. Avoid ambiguity. 3. Be brief. 4. Be orderly. | Deliberate obscurity signals meaning beyond the literal |

### Types of Implicature

**Conversational implicature**: arises from the Maxims in context; cancellable.
- "Some students passed" → implicates "not all students passed"
- Cancellable: "Some students passed — in fact, all of them did." (implicature cancelled)
- This is the *scalar implicature*: "some" implicates "not all"

**Conventional implicature**: triggered by specific words, not cancelable.
- "She was poor **but** honest" — `but` conventionally implicates contrast (poverty ↔ honesty are normally associated, here they're contrasted)
- Not cancellable: cannot say "She was poor but honest — I don't mean there's any contrast."

**Scalar implicatures**: from scales like `<some, all>`, `<warm, hot>`, `<possible, certain>`:
```
Scale: <some, all>
"I read some of the papers" → implicates "not all" (else would have said "all")

Scale: <possible, certain>
"It's possible it will rain" → implicates "it's not certain"

Scale: <or, and>
"You can have cake or ice cream" → implicates "not both"
```

### Gricean Reasoning (Abductive Inference)

The listener's pragmatic inference process:
1. The speaker said $p$
2. Assuming cooperation, why would they say $p$ in this context?
3. The most plausible explanation generates the implicature $q$

This is a form of **abductive reasoning** — inferring the most plausible explanation.

---

## Math / Formal Notation

### RSA (Rational Speech Acts) — Formal Model of Implicature

The RSA framework (Frank & Goodman, 2012) formalizes Gricean reasoning probabilistically.

**Literal listener** $L_0$ interprets literally:
$$P_{L_0}(m \mid u) \propto \mathbb{1}[\![u]\!](m) \cdot P(m)$$
where $\mathbb{1}[\![u]\!](m) = 1$ if utterance $u$ is literally true of meaning $m$.

**Pragmatic speaker** $S_1$ chooses utterance to inform listener efficiently:
$$P_{S_1}(u \mid m, \lambda) \propto \exp\left(\lambda \cdot \log P_{L_0}(m \mid u) - C(u)\right)$$
where $\lambda$ = rationality parameter, $C(u)$ = utterance cost (length etc.)

**Pragmatic listener** $L_1$ reasons about speaker's choice:
$$P_{L_1}(m \mid u) \propto P_{S_1}(u \mid m) \cdot P(m)$$

**Scalar implicature derivation via RSA:**
- Utterances: {"some", "all"}, meanings: {some-not-all, all}
- "some" is literally true of both meanings; "all" is only true of "all"
- $S_1$ would say "all" if the meaning were "all" (more informative, same cost)
- Therefore $L_1$ hearing "some" infers meaning "some-not-all"

This recovers the implicature from pure rationality reasoning.

---

## Examples (Concrete)

### Quantity Maxim Implicatures
```
Q: "Do you know what time it is?"
A: "Well, the shops are closed."
→ Implicature: it's late (relevant piece of evidence that answers the spirit of the question)

Q: "Do you know John?"
A: "I know someone called John."
→ Implicature: I'm not sure if it's the same John (hedging with quantity)

"The lecture was interesting." [said by someone who normally loves lectures]
→ Implicature: the lecture was less interesting than usual (under-informative)
```

### Scalar Implicature Examples
```
"It's warm today." → implicates: it's not hot
"I might come."   → implicates: I'm not sure; possibly I won't
"He's a competent teacher." → implicates: he's not outstanding
"Some of the students passed." → implicates: not all passed
"Or you could email them." → implicates: don't call AND email (exclusive or)
```

### Relevance Maxim (flouting)
```
Q: "What did you think of my cooking?"
A: "The weather's been nice lately."
→ Implicature: I have nothing positive to say (relevance signals evasion)

Q: "Did you sleep with her?"
A: "I'm a married man."
→ Implicature: No (being a married man = wouldn't do that)
```

---

## How It Connects to ML / NLP

| Implicature Concept | NLP/ML Application |
|---|---|
| Scalar implicature | NLI (some → not all); textual entailment |
| Gricean relevance | Dialogue coherence modeling |
| Conventional implicature | Sentiment (`but`, `though`, `yet`) |
| Implicature cancellation | Adversarial NLI examples |
| RSA framework | Neural pragmatic inference; reference game agents |
| Hallucination | LLMs violate Quality maxim (assert false/ungrounded claims) |

**Scalar implicature in NLI:**
```
Premise: "Some students passed the exam."
Hypothesis: "All students passed the exam."
Label: CONTRADICTION (via scalar implicature)
```

This is debated — the hypothesis is not *logically* contradicted (some ⊆ all is valid). But pragmatically, the premise implicates not-all. Most NLI datasets contain such scalar implicature cases.

**LLM Hallucination as Maxim violation:**
When an LLM generates a confident false statement, it violates Grice's Quality maxim. The Maxims provide a principled framework for analyzing LLM failure modes:
- **Quality violation**: hallucination (stating falsehoods)
- **Quantity violation**: over-generation (adding irrelevant facts) or under-generation (omitting needed info)
- **Manner violation**: verbosity, obscure phrasing

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — implicature detection as classification
- [[3.ML & DL/1.Concepts/8.Model Behavior/Overfitting.md]] — NLI models memorize pragmatic patterns without understanding

---

## Common Interview Questions

**Q: What is the difference between entailment and implicature?**
A: Entailment is a logical relation: $p$ entails $q$ if whenever $p$ is true, $q$ must be true in all possible worlds. Implicature is a pragmatic inference that can be cancelled: "Some students passed" implicates "not all" but doesn't entail it — adding "in fact, all of them" cancels the implicature without contradiction. This distinction is crucial for NLI system design.

**Q: How does Gricean theory explain sarcasm?**
A: Sarcasm involves *flouting* (deliberately violating) the Quality maxim. When someone says "Oh, great, another meeting!" with obvious disappointment, they are saying something they manifestly believe to be false. The listener, assuming cooperation, infers that the speaker intends to convey the opposite of the literal meaning. This requires meta-reasoning that is very hard for NLP systems.

**Q: Can LLMs perform pragmatic inference?**
A: They show partial ability. LLMs can resolve many implicatures that appear frequently in training data (scalar implicatures, conventional implicatures). They struggle with: (1) novel implicatures requiring genuine Gricean reasoning, (2) implicatures that depend on speaker identity/authority, (3) cross-cultural pragmatic conventions that differ from their training distribution.

---

## Common Mistakes / Gotchas

- **Treating implicature as entailment in NLI**: "Some students passed" does NOT entail "not all passed" — but it implicates it. NLI systems trained on human judgments will mark this as contradiction; logic-based systems won't. Know which you need.
- **Confusing cancellability**: implicatures are cancellable; entailments are not. If you can add "...in fact, more than that" without contradiction, it's an implicature.
- **Ignoring cultural variation**: Grice's maxims are idealized for Western cooperative discourse. Different cultures have different norms — what counts as "relevant" or "sufficiently informative" varies cross-culturally.
- **Assuming LLMs handle irony/sarcasm reliably**: they don't. Irony detection is still a hard NLP problem, especially in text without prosodic cues.

---

## Further Reading / Paper References

- Grice, H.P. (1975). "Logic and Conversation." In Cole & Morgan, *Syntax and Semantics Vol 3: Speech Acts.* — the original paper
- Levinson, S.C. (1983). *Pragmatics.* Cambridge University Press. — comprehensive textbook
- Horn, L.R. (1984). "Toward a New Taxonomy for Pragmatic Inference." — neo-Gricean refinements
- Frank, M.C. & Goodman, N.D. (2012). *Predicting Pragmatic Reasoning in Language Games.* Science. — RSA framework
- Goodman, N.D. & Frank, M.C. (2016). *Pragmatic Language Interpretation as Probabilistic Inference.* Trends in Cognitive Sciences.
- Jeretic, P. et al. (2020). *Are Natural Language Inference Models IMPPRESsive? Learning IMPlicature and PRESupposition.* ACL. — NLI + implicature benchmark
