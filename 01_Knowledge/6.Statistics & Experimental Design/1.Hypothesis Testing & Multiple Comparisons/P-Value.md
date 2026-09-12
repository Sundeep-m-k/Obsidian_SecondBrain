# P-Value

## What is it?

The **p-value** is the probability, computed *under the null hypothesis*, of observing a test statistic at least as extreme as the one actually observed.

$$p = P(T \geq t_{\text{obs}} \mid H_0) = \int_{t_{\text{obs}}}^{\infty} f_{T|H_0}(t)\, dt$$

for a one-sided test where large $T$ favors $H_1$; for a two-sided test, $p = P(|T| \geq |t_{\text{obs}}| \mid H_0)$.

---

## What It Is NOT

- **Not** $P(H_0 \text{ true} \mid \text{data})$ — that would require Bayes' theorem and a prior on $H_0$, which a p-value doesn't use
- **Not** the probability the alternative is true
- **Not** a measure of effect size — with a large enough $n$, even a trivial effect produces $p \to 0$, since the standard error shrinks as $1/\sqrt{n}$
- **Not** the probability you'd get the same result on replication

---

## Worked Example

Suppose $H_0: \mu = 0$, and a z-test gives $z_{\text{obs}} = 2.1$. Under $H_0$, $Z \sim \mathcal{N}(0,1)$:

$$p = 2 \cdot P(Z \geq 2.1) = 2 \cdot (1 - \Phi(2.1)) \approx 2 \cdot 0.0179 = 0.0357$$

This is below $\alpha = 0.05$, so the result is "statistically significant" at that threshold — but note the p-value itself ($0.0357$) says nothing about how large $\mu$ actually is; that requires reporting $\hat\mu$ and a confidence interval alongside it.

## Why Large $n$ Makes P-Values Misleading Alone

For a one-sample z-test, $z = \frac{\hat\mu - \mu_0}{\sigma/\sqrt n}$. As $n \to \infty$, the denominator $\to 0$, so *any* nonzero $\hat\mu - \mu_0$ eventually produces an arbitrarily small p-value — even a practically meaningless difference. This is why effect size (e.g. $\hat\mu - \mu_0$, or a standardized version like Cohen's $d$) must always be reported alongside $p$.

---

## Why It Matters

Every downstream statistical decision here — [[Permutation Test]]s comparing a model against a baseline, [[Multiple Comparisons Problem]] corrections via [[Benjamini-Hochberg Procedure]] — is built on p-values as the raw currency being thresholded and corrected.

## Common Mistakes

- Reporting "$p<0.05$" as proof of a real, important effect without checking effect size
- Comparing p-values across studies with very different $n$ as if they're on the same scale of importance
- Cherry-picking the significant tests out of many run (see [[Multiple Comparisons Problem]])

## Interview / discussion questions

- Why can a p-value be tiny even for a practically meaningless effect? Show this with the z-test formula.
- If $H_0$ is true and you run the same experiment 100 times, roughly how many trials show $p<0.05$ by chance?
- Why is a p-value not the probability that $H_0$ is true?

## Prerequisites

[[Hypothesis Test]], normal distribution / CDF

## Related concepts

[[Permutation Test]], [[Multiple Comparisons Problem]], [[Benjamini-Hochberg Procedure]]

## Tags

#category/statistics #topic/hypothesis-testing #math/probability

## One-line summary

> A p-value is $P(\text{data this extreme or more} \mid H_0)$ — it shrinks toward 0 as sample size grows even for trivial effects, which is why it must always be read alongside an effect size, never alone.
