# Expectation

## What is it?

The **expectation** (expected value) of a random variable is its probability-weighted average — the long-run average value you'd observe if you repeated the random process infinitely many times.

$$\mathbb{E}[X] = \sum_x x \cdot p(x) \quad \text{(discrete)}, \qquad \mathbb{E}[X] = \int_{-\infty}^{\infty} x f(x)\,dx \quad \text{(continuous)}$$

---

## Worked Numerical Example

A fair 6-sided die: $\mathbb{E}[X] = \sum_{k=1}^{6} k \cdot \frac{1}{6} = \frac{1+2+3+4+5+6}{6} = 3.5$.

Notice $3.5$ is *not a value the die can ever show* — expectation is a long-run average, not a prediction of any single outcome. Roll the die 6,000 times and the *average* of all rolls converges to 3.5, even though every individual roll is an integer from 1 to 6.

**A more decision-relevant example**: a marketing campaign costs $1,000 to run. It has a 20% chance of generating $10,000 in revenue and an 80% chance of generating $0.
$$\mathbb{E}[\text{revenue}] = 0.20 \times \$10{,}000 + 0.80 \times \$0 = \$2{,}000$$
Expected profit $= \$2{,}000 - \$1{,}000 = \$1{,}000 > 0$ — worth running *on average*, even though 80% of the time it actually loses money. This is exactly the reasoning behind evaluating a single [[A-B Testing|A/B test]] variant's expected value versus its risk, not just its most likely single outcome.

---

## Key Properties

**Linearity of expectation** — $\mathbb{E}[aX+bY] = a\mathbb{E}[X]+b\mathbb{E}[Y]$, **always true, regardless of whether $X$ and $Y$ are independent**. This is one of the most useful and most under-appreciated facts in probability — it lets you compute the expectation of a complicated sum by just adding up each piece's expectation separately, with zero need to know how the pieces relate to each other.

**Expectation is not the mode or median** — for skewed distributions these three can differ substantially (e.g. income: mean pulled up by a few very high earners, median much lower, mode lower still). Reporting "average" income without specifying which of these three is used is a common source of misleading statistics.

## Common Misconception

**"The expected value is what usually happens."** It's the long-run *average*, which for many real distributions is a value the variable rarely or never actually takes (the 3.5 die example, or "the average family has 2.3 children"). Confusing expectation with "the typical/most likely outcome" is a frequent, consequential error — e.g. reading "the expected wait time is 4.7 minutes" as "most customers wait about 4.7 minutes," when a right-skewed wait-time distribution might mean most customers actually wait much less, with a long tail of occasional very long waits pulling the average up.

---

## Relationship to ML/DS

**Loss functions are expectations.** Training a model by minimizing a [[Cost Function]] is literally minimizing the *expected* loss over the training distribution — $\mathbb{E}[\ell(\hat{y},y)]$. This is why linearity of expectation matters practically: the [[Bias Variance Tradeoff]] decomposition itself is derived by expanding an expectation using exactly this linearity property. Every reported model metric (mean accuracy, mean squared error) is an empirical estimate of an expectation over the true (unknown) data distribution.

---

## Interview-Ready Explanation

"Expectation is the probability-weighted average of a random variable — multiply each possible value by its probability and sum. The single most useful fact about it is linearity: the expectation of a sum always equals the sum of expectations, no matter how the underlying variables are related, which is why it shows up as the backbone of loss functions and the bias-variance decomposition."

---

## Interview Questions

**Why is linearity of expectation true even when the variables aren't independent?** It falls directly out of the definition as a weighted sum/integral — the weighting-and-summing operation distributes over addition regardless of any relationship between the variables; independence is required for *variance* to add simply, but never for expectation.

**A lottery ticket costs $2 and has a 1-in-1,000,000 chance of winning $500,000 — what's the expected value, and does that mean you should buy one?** $\mathbb{E}[\text{payout}] = \frac{1}{1{,}000{,}000}\times\$500{,}000 = \$0.50$, versus a $2 cost — negative expected value, meaning on average you lose money buying tickets, even though a small chance of a large win feels appealing; expectation alone also ignores risk tolerance, which matters for a one-off decision more than a repeated one.

**Why can the expected value of a variable be a number the variable can never actually equal?** Because expectation is a probability-weighted *average* across all possible outcomes, not a selection among them — a die's expectation (3.5) is a mathematical summary of the whole distribution, not a claim about any achievable single roll.

**How does expectation relate to a model's loss function during training?** Training minimizes the expected loss over the data distribution — the empirical average loss on a training batch is a sample estimate of that true expectation, and generalization is fundamentally the question of whether minimizing the empirical (sample) expectation also minimizes the true (population) expectation on unseen data.

**Mean vs. median vs. mode — when do they diverge, and why does it matter for reporting a statistic?** They coincide for a perfectly symmetric, unimodal distribution and diverge under skew (e.g. income) — reporting "the average" without specifying which can mislead a stakeholder, since a mean pulled upward by a few extreme values can misrepresent the "typical" case that a median would better convey.

## Connections

- [[Random Variables]], [[Probability Distributions]] — expectation is defined over these
- [[Variance and Standard Deviation]], [[Covariance and Correlation]] — both are themselves defined as expectations of a transformed variable
- [[Bias Variance Tradeoff]] — derived by expanding an expectation using linearity
- [[Cost Function]], [[Loss Function]] — training minimizes an empirical estimate of an expectation
- [[Central Limit Theorem]] — describes how a sample average (an empirical expectation estimate) behaves as sample size grows

## One-line Summary

> Expectation is the probability-weighted average of a random variable — not the typical or most likely outcome — and its linearity property (the expectation of a sum is always the sum of expectations, regardless of dependence) is the mathematical backbone behind loss functions and the bias-variance decomposition.
