# Central Limit Theorem

## What is it?

The **Central Limit Theorem (CLT)** says that the [[Sampling and Sampling Distributions|sampling distribution]] of a sample mean approaches a normal distribution as sample size grows — **regardless of the shape of the original population's distribution**, as long as observations are independent and identically distributed with finite variance.

$$\bar{X}_n \approx \mathcal{N}\left(\mu, \ \frac{\sigma^2}{n}\right) \quad \text{as } n \to \infty$$

$\mu, \sigma^2$ are the *population's* mean and variance; $\bar{X}_n$ is the mean of a sample of size $n$. The standard deviation of this sampling distribution, $\sigma/\sqrt{n}$, is called the **standard error** — it shrinks as $\sqrt{n}$, meaning to halve the standard error you need to *quadruple* the sample size, not just double it.

---

## Why This Is Remarkable (Not Just a Technicality)

The population itself can be *any* shape — uniform, exponential, heavily skewed, even bimodal — and the *distribution of its sample mean* still converges toward a normal bell curve. This is what licenses using normal-distribution-based formulas (like a standard [[Confidence Intervals and Bootstrap|confidence interval]] or a [[Hypothesis Test]]'s z-test) for a *mean*, even when the underlying data obviously isn't normal — the CLT is a statement about the mean's behavior under repeated sampling, not a claim that the raw data itself becomes normal.

## Worked Numerical Example: Simulating the CLT

Population: a single die roll, uniform over $\{1,2,3,4,5,6\}$ — visibly *not* normal (it's flat). $\mu = 3.5$, $\sigma^2 \approx 2.917$.

- **$n=1$** (one roll): the sampling distribution of the mean is just the original uniform distribution — flat, not bell-shaped at all.
- **$n=2$** (average of two rolls): possible sums range 2-12; already visibly more bell-shaped and peaked around 7 (mean 3.5), since middle sums (like 7) can be reached by more combinations (1+6, 2+5, 3+4...) than extreme sums (like 2, only 1+1).
- **$n=30$** (average of 30 rolls): the distribution of that average is very close to $\mathcal{N}(3.5, \ 2.917/30) = \mathcal{N}(3.5, 0.097)$ — a tight, clearly bell-shaped curve, even though a single die roll is nowhere close to normal.

This progression — flat, to lumpy-but-symmetric, to clearly bell-shaped — *is* the Central Limit Theorem happening in front of you as $n$ grows, starting from a distribution that couldn't be less normal-looking.

---

## "How Large Does $n$ Need to Be?"

**No universal number** — it depends on how far the population distribution is from normal already. A rule of thumb often cited is $n \ge 30$, but this is genuinely just a rule of thumb: a population that's already roughly symmetric needs a much smaller $n$ for the approximation to be good; a heavily skewed or extreme-outlier-prone population needs a much larger $n$. **Common misconception**: treating $n\ge30$ as a hard, universal threshold rather than what it actually is — a rough heuristic that depends entirely on the underlying population's shape.

## When the CLT's Assumptions Break

- **Non-independent observations** — time series data, or grouped/clustered data (repeated measures on the same patient) — violates the i.i.d. assumption directly; this is exactly why [[Grouped Train-Test Split]] and [[Rolling-Origin Validation]] exist as their own topics, since standard CLT-based formulas don't apply cleanly to such data.
- **Infinite or undefined variance** — some heavy-tailed distributions (certain financial return models) don't have finite variance at all, and the CLT simply doesn't apply regardless of $n$.
- **Extremely skewed populations with small $n$** — the approximation can still be poor even at "reasonable" sample sizes; this is why [[Confidence Intervals and Bootstrap|bootstrap methods]] are preferred over normal-approximation formulas when the population's shape is unknown or clearly non-normal and $n$ isn't very large.

---

## Relationship to ML/DS

**Every standard [[Confidence Intervals and Bootstrap|confidence interval]] formula that uses $\bar{x} \pm 1.96 \cdot \text{SE}$ relies on the CLT** to justify treating the sampling distribution as normal. **[[A-B Testing]]'s standard t-tests/z-tests** on conversion rate differences lean on the CLT to justify a normal approximation to what is, per-user, actually a binary (Bernoulli) outcome — this is exactly why A/B tests generally need a reasonably large sample size before their significance tests are trustworthy. **Batch training in deep learning** — averaging gradients across a mini-batch is, informally, exploiting the same variance-reduction-through-averaging mechanism the CLT formalizes, which is part of why larger batch sizes produce more stable (lower-variance) gradient estimates per step.

---

## Interview-Ready Explanation

"The Central Limit Theorem says the sampling distribution of a mean tends toward normal as sample size grows, no matter what the underlying population looks like — even a flat or heavily skewed distribution. That's what licenses using normal-based formulas (confidence intervals, many hypothesis tests) for averages in practice, even on non-normal raw data. The catch: it needs independent observations and a large-enough sample, where 'large enough' depends on how non-normal the population already is — n≥30 is a rule of thumb, not a guarantee."

---

## Interview Questions

**What's the difference between the CLT and the Law of Large Numbers?** See [[Law of Large Numbers]] for the full comparison — briefly, LLN guarantees the sample average *converges* to the true mean as $n$ grows; CLT is the sharper, separate claim about the *normal shape* the remaining (properly rescaled) fluctuation around that mean takes — "more data stabilizes the average" is LLN, "the sampling distribution of the mean becomes approximately normal" is CLT.

**Why can you use a normal-distribution-based confidence interval for a mean, even when the underlying data is clearly not normal?** The CLT says it's the *sampling distribution of the mean* that approaches normal as $n$ grows, regardless of the population's own shape — the confidence interval formula relies on this fact about the mean's behavior under repeated sampling, not on the raw data itself being normally distributed.

**Why does the standard error shrink as $\sqrt{n}$ rather than $n$?** It follows directly from variance scaling: the variance of a sample mean is $\sigma^2/n$, so its standard deviation (the standard error) is $\sigma/\sqrt{n}$ — a consequence of averaging independent observations, and the direct reason halving your margin of error requires quadrupling your sample size, not doubling it.

**Is $n\ge30$ always enough for the CLT approximation to be valid?** No — it's a rough rule of thumb; a population that's already close to symmetric needs far fewer than 30, while a heavily skewed or outlier-prone population may need substantially more before the sampling distribution of the mean looks reasonably normal.

**Does the CLT apply to time series data collected sequentially from the same source?** Not directly — the CLT assumes independent, identically distributed observations, and time series data is typically autocorrelated (today's value depends on yesterday's), which is exactly why time series needs its own validation and inference machinery ([[Rolling-Origin Validation]], [[Time Series Fundamentals]]) rather than standard CLT-based formulas.

**How does the CLT relate to why an A/B test needs a minimum sample size before its significance test is trustworthy?** The test's p-value calculation typically assumes the sampling distribution of the difference in conversion rates is approximately normal, which the CLT justifies only once the sample is large enough — too small a sample means the normal approximation (and therefore the p-value itself) can be unreliable, independent of whether a real effect exists.

## Connections

- [[Law of Large Numbers]] — the complementary, more basic claim (convergence) that this note's *shape-of-convergence* claim builds on top of; commonly confused with this note, worth reading together
- [[Sampling and Sampling Distributions]] — the CLT is a specific, precise statement about one particular sampling distribution
- [[Confidence Intervals and Bootstrap]] — the theoretical justification for normal-approximation confidence intervals
- [[Hypothesis Test]], [[A-B Testing]] — most classical significance tests lean on the CLT for their normal-approximation validity
- [[Variance and Standard Deviation]] — the $\sigma^2/n$ scaling is a direct consequence of variance's behavior under averaging
- [[Rolling-Origin Validation]], [[Grouped Train-Test Split]] — exist because the CLT's independence assumption fails for temporal/grouped data

## One-line Summary

> The Central Limit Theorem guarantees the sampling distribution of a mean approaches normal as $n$ grows regardless of the population's own shape — the theoretical foundation under nearly every normal-approximation confidence interval and significance test, and it fails when observations aren't independent (time series, grouped data) or when $n$ is too small relative to how non-normal the population already is.
