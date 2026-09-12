# Speech Acts

tags: #nlp #pragmatics #dialogue #intent-detection #dialogue-acts
links: [[Implicature and Grice]] [[Discourse Structure]] [[Coreference]] [[Intent Detection]] [[Dialogue State Tracking]]

---

## Definition + Intuition

**Speech act theory** (Austin 1962, Searle 1969) holds that utterances are not just descriptions of the world — they are *actions*. When you say "I promise to call you," you are not describing a promise; you are *performing* one.

Every utterance has three levels:
1. **Locutionary act**: the literal meaning of what is said
2. **Illocutionary act**: the speaker's intention / social action being performed
3. **Perlocutionary act**: the effect on the listener

> **Intuition**: "Can you pass the salt?" is literally a yes/no question about your ability. But everyone knows it's a *request*. The locutionary act (a question) is totally different from the illocutionary act (a request). Understanding speech means going beyond literal meaning — this is pragmatics.

---

## Key Properties / Types

### Searle's Taxonomy of Illocutionary Acts

| Type | Description | Examples |
|------|-------------|---------|
| **Assertives** | Commit speaker to truth of proposition | state, assert, claim, believe, report |
| **Directives** | Attempt to get listener to do something | ask, order, request, invite, permit |
| **Commissives** | Commit speaker to future action | promise, offer, threaten, vow |
| **Expressives** | Express psychological state | thank, apologize, congratulate, deplore |
| **Declarations** | Change state of affairs by utterance | "You're fired." "I pronounce you..." "I hereby declare..." |

### Direct vs Indirect Speech Acts

**Direct**: illocutionary force matches sentence type
- Interrogative → question: "What time is it?"
- Imperative → order: "Close the door."
- Declarative → assertion: "The door is open."

**Indirect**: illocutionary force is different from literal sentence type
- "It's cold in here." → (request to close a window)
- "Can you pass the salt?" → (request, not a capability question)
- "Would you mind...?" → (polite request, not a question about mental state)

Indirect speech acts are the *norm* in polite conversation — understanding them requires world knowledge, context, and social conventions.

### Dialogue Acts
In dialogue systems, speech acts are operationalized as **dialogue acts** (DA):
```
Greeting:    "Hello! How can I help?"
Inform:      "The next train leaves at 3pm."
Request:     "What time does the last train leave?"
Confirm:     "Yes, that's right."
Deny:        "No, the train is at 4pm, not 3pm."
Clarify:     "Do you mean the express or local?"
Promise:     "I'll look that up for you."
Goodbye:     "Have a nice trip!"
```

---

## Math / Formal Notation

**Dialogue act classification** as sequence labeling:

Given utterance $u = w_1 w_2 \ldots w_n$, predict dialogue act $d \in \mathcal{D}$:

$$d^* = \arg\max_{d \in \mathcal{D}} P(d \mid u, h)$$

where $h$ is the conversation history. This is a [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] problem.

**Pragmatic inference** can be modeled with **Rational Speech Act (RSA) framework**:

**Literal listener** $L_0$: interprets utterance at face value
$$P_{L_0}(s \mid u) \propto P(u \mid s) \cdot P(s)$$

**Pragmatic speaker** $S_1$: chooses utterance considering how listener will interpret it
$$P_{S_1}(u \mid s) \propto \exp(\alpha \cdot \log P_{L_0}(s \mid u)) \cdot P(u)$$

**Pragmatic listener** $L_1$: reasons about speaker's communicative intent
$$P_{L_1}(s \mid u) \propto P_{S_1}(u \mid s) \cdot P(s)$$

RSA has been used to model pragmatic inference in neural NLP models — including why "some students passed" pragmatically implies "not all students passed."

---

## Examples (Concrete)

**The same string, different speech acts depending on context:**
```
"It's a beautiful day."
  → Assertive (stating a fact)
  → Expressive (expressing pleasure)
  → Implicit directive (let's go outside)
  → Sarcasm (expressing frustration) — prosody/context dependent
```

**Declarations — reality-changing utterances:**
```
"I now pronounce you husband and wife." → creates marriage
"You're under arrest."                 → changes legal status
"The meeting is adjourned."            → ends the meeting
"I name this ship the Enterprise."     → names the ship
These only work if speaker has the right institutional role.
```

**Indirect request strategies:**
```
Direct:    "Give me the report."
Indirect 1: "Could you give me the report?"     (ability framing)
Indirect 2: "I need the report."                (need framing)
Indirect 3: "Is the report ready?"             (hint)
Indirect 4: "I was hoping to see the report."  (desire framing)
```
All are requests; politeness increases as directness decreases.

---

## How It Connects to ML / NLP

| Speech Act Concept | NLP/ML Application |
|---|---|
| Illocutionary act classification | Dialogue act detection; intent classification |
| Indirect speech acts | Pragmatic inference; politeness detection |
| Declarations | Domain-specific: legal documents, medical orders |
| RSA model | Pragmatic NLG; reference games |
| DA sequences | Dialogue state tracking; conversation modeling |

**In task-oriented dialogue systems:**
The NLU component performs intent classification + slot filling — this is exactly dialogue act recognition. "Book a flight to Paris on Monday" → intent: `book_flight`, slots: `{destination: Paris, date: Monday}`.

**Cross-links:**
- [[3.ML & DL/1.Concepts/2.Supervised Learning/Classification.md]] — DA classification
- [[3.ML & DL/1.Concepts/6.Features and Representation/Feature Engineering.md]] — context features for indirect speech act detection

---

## Common Interview Questions

**Q: Why is understanding intent harder than understanding literal meaning?**
A: Literal meaning is recoverable from the sentence alone (compositionally). Intent requires reasoning about the speaker's goals, the conversational context, social norms, and world knowledge. "Can you pass the salt?" — the literal meaning is trivially recoverable; the intended meaning (request) requires knowing that this phrasing is a conventional request form in English.

**Q: How do LLMs handle speech acts?**
A: LLMs learn speech act patterns implicitly from training data. They generate appropriate responses to indirect requests because they've seen millions of such exchanges. However, they don't have an explicit model of illocutionary force — they can fail on unusual indirect speech acts or when the social/institutional context is important (e.g., whether a declaration is performed by someone with the right authority).

---

## Common Mistakes / Gotchas

- **Treating all speech as assertive**: most dialogue is not factual assertion. Failing to model other speech act types (requests, questions, expressives) causes dialogue systems to misinterpret user turns.
- **Ignoring indirectness**: a model trained only on direct speech acts will fail on "It's getting warm in here" as a request to open a window.
- **Conflating intent and sentiment**: "I hate waiting" has negative sentiment but is an expressive, not a complaint directed at anyone. Intent ≠ sentiment.

---

## Further Reading / Paper References

- Austin, J.L. (1962). *How To Do Things With Words.* Oxford University Press.
- Searle, J.R. (1969). *Speech Acts.* Cambridge University Press.
- Stolcke, A. et al. (2000). *Dialogue Act Modeling for Automatic Tagging and Recognition of Conversational Speech.* Computational Linguistics.
- Frank, M.C. & Goodman, N.D. (2012). *Predicting Pragmatic Reasoning in Language Games.* Science. — RSA model
- Chen, D. & Mooney, R.J. (2011). *Learning to Interpret Natural Language Navigation Instructions from Observations.* AAAI.
