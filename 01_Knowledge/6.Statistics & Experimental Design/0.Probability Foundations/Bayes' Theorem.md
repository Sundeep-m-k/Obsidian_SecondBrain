# Bayes' Theorem

## What is it?

**Bayes' theorem** is a rule for reversing the direction of a [[Conditional Probability|conditional probability]] — computing $P(A\mid B)$ from $P(B\mid A)$ (the direction that's usually easier to know or measure directly).

$$P(A\mid B) = \frac{P(B\mid A)\,P(A)}{P(B)}$$

Derived in one line from the [[Conditional Probability|chain rule]]: $P(A\cap B) = P(A\mid B)P(B) = P(B\mid A)P(A)$, then divide both sides by $P(B)$.

---

## Prior, Likelihood, Posterior — The Vocabulary That Matters in Interviews

$$\underbrace{P(A\mid B)}_{\text{posterior}} = \frac{\overbrace{P(B\mid A)}^{\text{likelihood}} \cdot \overbrace{P(A)}^{\text{prior}}}{\underbrace{P(B)}_{\text{evidence}}}$$

- **Prior** $P(A)$ — what you believed about $A$ *before* seeing evidence $B$.
- **Likelihood** $P(B\mid A)$ — how probable the observed evidence $B$ is, *if* $A$ were true.
- **Posterior** $P(A\mid B)$ — your updated belief about $A$ *after* accounting for the evidence.
- **Evidence** $P(B)$ — the overall probability of observing $B$ at all, across every possible state of $A$ (a normalizing constant, computed via the law of total probability: $P(B) = \sum_a P(B\mid A=a)P(A=a)$).

**The one-sentence version worth memorizing for an interview**: *posterior is proportional to likelihood times prior* — new belief = how well the evidence fits each hypothesis, weighted by how plausible each hypothesis was beforehand.

---

## Worked Numerical Example (Same Disease Test as Conditional Probability)

Disease prevalence (prior): $P(\text{disease}) = 0.01$. Test sensitivity (likelihood of a positive test given disease): $P(+\mid\text{disease}) = 0.95$. False positive rate: $P(+\mid\text{no disease}) = 0.05$.

$$P(+) = P(+\mid D)P(D) + P(+\mid \neg D)P(\neg D) = 0.95(0.01) + 0.05(0.99) = 0.059$$

$$P(D\mid +) = \frac{P(+\mid D)P(D)}{P(+)} = \frac{0.95 \times 0.01}{0.059} \approx 0.161$$

Same 16.1% answer as [[Conditional Probability]]'s worked example — Bayes' theorem is just the formal machine that produces it from the prior (1% prevalence) and likelihood (95% sensitivity) directly, rather than building a frequency table by hand.

---

## Why the Prior Matters So Much (and Is So Often Ignored)

**Base rate neglect** is the single most common real-world reasoning error Bayes' theorem corrects: people intuitively weight the likelihood ($P(+\mid D)=95\%$, "the test is accurate") far more than the prior ($P(D)=1\%$, "the disease is rare"), producing a wildly overconfident gut estimate ("95% likely I have it") versus the correct 16%. **The same error shows up in ML**: a model evaluated as "95% accurate" on a rare-event problem can still be nearly useless in the positive class specifically, for exactly the reason [[Class Imbalance Evaluation]] exists as its own topic — the base rate (class prevalence) dominates what a raw accuracy or even a single conditional probability actually implies.

---

## Relationship to ML/DS

**[[Naive Bayes]]** applies this formula directly to classification: $P(y\mid x) \propto P(x\mid y)P(y)$ — the "naive" part is assuming the features in $P(x\mid y)$ are conditionally independent given $y$, making the likelihood a simple product. **Bayesian vs. frequentist interpretation of a [[P-Value]]**: a p-value is $P(\text{data}\mid H_0)$ — a *likelihood*, not the posterior $P(H_0\mid\text{data})$ people often mistakenly read it as; getting this backwards is one of the most common statistical misinterpretations, covered in [[P-Value]]. **Bayesian A/B testing** treats the true conversion rate as a random variable with a prior, updated by observed data into a posterior — a genuine alternative to the frequentist [[A-B Testing]] approach, trading a need to specify a prior for more directly interpretable statements ("there's an 87% probability variant B is better," rather than a p-value).

---

## Interview-Ready Explanation

"Bayes' theorem flips the direction of a conditional probability — it lets you go from 'how likely is this evidence, given a hypothesis' (usually easy to know) to 'how likely is the hypothesis, given this evidence' (usually what you actually want to know). The formula is posterior ∝ likelihood × prior. The single biggest practical lesson: the prior/base rate matters enormously, and ignoring it (base rate neglect) is why 'the test is 95% accurate' gets misread as '95% chance I have the disease' when the true answer, accounting for how rare the disease is, is much lower."

---

## Interview Questions

**What's the difference between $P(B\mid A)$ and $P(A\mid B)$, and why does confusing them matter?** They're generally different numbers connected by Bayes' theorem — confusing them is exactly the "prosecutor's fallacy"/p-value misinterpretation: mistaking $P(\text{data}\mid H_0)$ (a p-value) for $P(H_0\mid\text{data})$ (what people often wrongly think a p-value tells them) can lead to dramatically overconfident conclusions.

**Why does a rare disease make a "95% accurate" test's positive result much less conclusive than intuition suggests?** Because the prior (disease prevalence) is very low, the likelihood alone doesn't determine the posterior — Bayes' theorem weights the likelihood by the prior, and with a rare condition, false positives from the enormous healthy population outnumber true positives even under a highly accurate test; this is base rate neglect, the most common real-world Bayesian reasoning error.

**How does Naive Bayes use Bayes' theorem, and where does the "naive" assumption enter?** It computes $P(y\mid x) \propto P(x\mid y)P(y)$ directly — the naive part is assuming the features composing $x$ are conditionally independent given $y$, which turns the otherwise-intractable joint likelihood $P(x\mid y)$ into a simple product of per-feature likelihoods.

**What's the practical difference between a frequentist A/B test and a Bayesian one?** A frequentist test reports a p-value — the probability of the observed data (or more extreme) under the null hypothesis — and requires a decision rule around a fixed significance threshold; a Bayesian approach places a prior on the true effect and computes a posterior probability directly (e.g. "87% probability B is better than A"), trading the need to specify a prior for a more directly interpretable output.

**Why is "evidence" $P(B)$ in Bayes' theorem computed as a sum over all hypotheses?** Because it's the total probability of observing $B$ regardless of which hypothesis is actually true — the law of total probability, $P(B)=\sum_a P(B\mid A=a)P(A=a)$ — and it acts as the normalizing constant ensuring the posterior probabilities across all hypotheses sum to 1.

## Connections

- [[Conditional Probability]] — Bayes' theorem is derived directly from its chain rule
- [[Naive Bayes]] — the direct ML application of this formula to classification
- [[P-Value]] — a p-value is a likelihood, not a posterior; conflating the two is a version of the same error base rate neglect produces
- [[Class Imbalance Evaluation]] — the ML manifestation of base rate neglect
- [[A-B Testing]] — the frequentist alternative to a Bayesian updating approach

## One-line Summary

> Bayes' theorem reverses a conditional probability's direction via posterior ∝ likelihood × prior — its most important practical lesson is that ignoring the prior (base rate neglect) turns an accurate-sounding test or model into a badly overconfident conclusion whenever the thing being detected is rare.
