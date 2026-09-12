# Conditional Probability

## What is it?

**Conditional probability** is the probability of an event, given that another event is already known to have happened — updating a probability in light of new information.

$$P(A \mid B) = \frac{P(A \cap B)}{P(B)}, \quad P(B) > 0$$

Read: "the probability of $A$, given $B$" equals the probability both happen, divided by the probability $B$ happens at all — restricting attention to only the world where $B$ is true, then asking how much of *that* world also has $A$.

---

## Worked Numerical Example

A disease affects 1% of a population. A test is 95% accurate for both true positives and true negatives (sensitivity = specificity = 95%). A random person tests positive — what's the probability they actually have the disease?

Using a table per 10,000 people:
- 100 actually have the disease (1%): 95 test positive (true positives), 5 test negative (false negatives)
- 9,900 don't have the disease: 495 test positive anyway (5% false positive rate), 9,405 test negative correctly

$$P(\text{disease}\mid\text{positive}) = \frac{95}{95+495} = \frac{95}{590} \approx 16.1\%$$

Despite a "95% accurate" test, a positive result only means ~16% actual disease probability — because the disease is rare, false positives from the large healthy population (495 people) vastly outnumber true positives (95 people) in absolute terms. This is the single most common conditional-probability trap in both medicine and ML (e.g. fraud/anomaly detection with a rare positive class) — see [[Class Imbalance Evaluation]] for the ML-specific version of exactly this problem.

---

## Independence

$A$ and $B$ are **independent** if knowing $B$ happened tells you nothing about $A$: $P(A\mid B) = P(A)$, equivalently $P(A\cap B) = P(A)P(B)$. This is the assumption [[Naive Bayes]] makes about features given the class — usually false in the strict sense, but often close enough to be useful anyway.

**Common misconception**: independent and mutually exclusive are opposite ideas, often confused. Mutually exclusive events ($A$ and $B$ can't both happen, $P(A\cap B)=0$) are actually maximally *dependent* — if $B$ happens, $A$ definitely didn't, which is about as much information as $B$ could possibly give about $A$. Independence means *zero* information transfer; mutual exclusivity means *total* information transfer (just in the negative direction).

---

## The Chain Rule of Probability

$$P(A \cap B) = P(A\mid B)P(B) = P(B\mid A)P(A)$$

Generalizes to any number of events: $P(A_1,\dots,A_n) = P(A_1)P(A_2\mid A_1)P(A_3\mid A_1,A_2)\cdots$ — decomposing a joint probability into a product of conditionals. This is exactly how a language model computes the probability of a whole sentence: $P(w_1,\dots,w_n) = \prod_i P(w_i \mid w_1,\dots,w_{i-1})$, the mathematical basis of autoregressive generation covered in [[LLM Inference Fundamentals]].

---

## Relationship to ML/DS

**[[Naive Bayes]]** is conditional probability plus an independence assumption, applied to classification. **[[Data Leakage]]** is, at its core, a conditional-probability violation — a feature is leaky precisely when it encodes information that shouldn't be "conditionally available" at prediction time. **Precision** ($P(\text{actually positive}\mid\text{predicted positive})$) and **recall** ($P(\text{predicted positive}\mid\text{actually positive})$) from [[Classification Metrics]] are themselves a conditional-probability pair read in opposite directions — exactly analogous to the disease-test example above, which is why a model can have high accuracy and still have poor precision on a rare positive class.

---

## Interview-Ready Explanation

"Conditional probability updates a probability given new evidence — $P(A|B)$ restricts attention to the world where $B$ is true and asks how much of that world also satisfies $A$. The classic gotcha: even a highly accurate test for a rare condition gives a low conditional probability of actually having the condition after a positive result, because false positives from the much larger healthy population outnumber true positives in absolute count. This exact same math is why accuracy misleads on imbalanced classification."

---

## Interview Questions

**A test is 95% accurate for a disease affecting 1% of the population — a patient tests positive, what's the probability they have the disease, and why is the answer so much lower than 95%?** About 16% — because the disease is rare, the much larger healthy population generates enough false positives (5% of 9,900 ≈ 495) to outnumber the true positives (95), even at a 95% accurate test; the answer is dominated by the base rate, not just the test's accuracy.

**What's the difference between independent events and mutually exclusive events?** Independent means knowing one event happened tells you nothing about the other ($P(A|B)=P(A)$); mutually exclusive means they can't both happen ($P(A\cap B)=0$), which actually means knowing one happened tells you the *other definitely didn't* — the opposite of independence, not a variant of it.

**How does the chain rule of probability relate to how a language model generates text?** An LLM computes the probability of a full sequence as a product of conditional probabilities, one token given all previous tokens — exactly the chain-rule decomposition of a joint probability, computed autoregressively one factor at a time.

**Precision and recall — how are they both conditional probabilities, just conditioned in opposite directions?** Precision is $P(\text{actually positive} \mid \text{predicted positive})$; recall is $P(\text{predicted positive} \mid \text{actually positive})$ — same two events, opposite conditioning direction, which is why they can diverge sharply and why neither alone tells the full story (see [[Confusion Matrix]]).

**Why is data leakage fundamentally a conditional-probability problem?** A leaky feature is one whose value is only knowable conditional on information from after the prediction point — it encodes $P(\text{feature} \mid \text{future outcome})$ rather than being available purely conditional on $\mathcal{F}_t$ (information at decision time), so training on it teaches the model a relationship that won't hold at actual prediction time.

## Connections

- [[Bayes' Theorem]] — built directly on the definition of conditional probability, just rearranged
- [[Naive Bayes]] — conditional probability plus a feature-independence assumption
- [[Classification Metrics]], [[Confusion Matrix]] — precision/recall as a conditional-probability pair
- [[Class Imbalance Evaluation]] — the ML version of the disease-test paradox above
- [[Data Leakage]] — a conditional-probability violation at its core
- [[LLM Inference Fundamentals]] — the chain rule is how autoregressive generation is defined mathematically

## One-line Summary

> Conditional probability updates a probability given evidence via $P(A|B)=P(A\cap B)/P(B)$ — the classic trap (a "95% accurate" test giving only ~16% true probability after a positive result on a rare condition) is driven by base rates, not the test's own accuracy, and is mathematically identical to why accuracy misleads on imbalanced classification.
