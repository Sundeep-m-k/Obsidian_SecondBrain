# Sampling and Sampling Distributions

## What is it?

**Sampling** is the process of selecting a subset (a sample) from a larger population to estimate something about the whole population without measuring every member. A **sampling distribution** is the probability distribution of a statistic (e.g. the sample mean) computed across every possible sample of a given size — the theoretical object that makes it possible to say how much a statistic computed from *one* actual sample can be trusted.

---

## Why This Distinction Matters

There are three different distributions easy to conflate, and interview questions frequently probe exactly this confusion:

1. **Population distribution** — the distribution of the underlying variable itself, across the whole population (unknown, what we're trying to learn about).
2. **Sample distribution** — the distribution of the actual values observed in *one* drawn sample (an empirical approximation of #1, gets closer as sample size grows).
3. **Sampling distribution** — the distribution of a *statistic* (like the mean) *if you repeated the sampling process many times* — this is a distribution of a computed number, not of raw data at all, and it's a theoretical/simulated construct, not something you directly observe from one sample.

The [[Central Limit Theorem]] is a statement specifically about #3 — how the sampling distribution of the mean behaves — and it's the reason a *single* sample mean can be given a [[Confidence Intervals and Bootstrap|confidence interval]] at all.

---

## Sampling Methods

**Simple random sampling** — every member of the population has an equal chance of selection. The assumption every classical formula (standard error, confidence intervals) implicitly relies on.

**Stratified sampling** — split the population into subgroups (strata) and sample within each proportionally, guaranteeing representation of each subgroup — directly analogous to [[Cross Validation Strategy]]'s stratified CV, which preserves class proportions in every fold for exactly the same reason.

**Convenience sampling** — sample whoever's easiest to reach (survey respondents, app users who opt in). Fast and cheap, but introduces **selection bias**: whoever chooses to respond may differ systematically from the full population (see [[Correlation vs Causation]]'s treatment of selection bias), which no amount of sample *size* can fix — a huge biased sample is still biased.

## Sampling Bias vs. Sample Size

**A common misconception worth naming explicitly**: a bigger sample reduces *variance* (random sampling error) but does nothing to fix *bias* (systematic sampling error). A convenience sample of 1,000,000 self-selected survey respondents is not more trustworthy than a truly random sample of 1,000 — it's just a more precisely wrong estimate. This is a frequent, high-value interview point: "would collecting more data fix this?" often has the answer "only if the *bias*, not just the noise, is the actual problem."

---

## Worked Numerical Example: Building a Sampling Distribution by Hand

Population: the numbers $\{2, 4, 6, 8\}$, true mean $\mu = 5$. Draw all possible samples of size $n=2$ *with replacement* (16 equally likely pairs) and compute each sample's mean:

$(2,2){=}2,\ (2,4){=}3,\ (2,6){=}4,\ (2,8){=}5,\ (4,2){=}3,\ (4,4){=}4,\dots$ (16 total pairs)

Tabulating all 16 sample means gives a distribution — its own mean turns out to be exactly $5$ (matching the population mean $\mu$, illustrating that the sample mean is an **unbiased estimator**), and its spread is *smaller* than the original population's spread (illustrating that averaging reduces variance — the core mechanism behind why larger samples give tighter estimates). This is a sampling distribution built by literal enumeration on a tiny population — in practice it's either derived analytically (as the [[Central Limit Theorem]] does) or approximated by resampling (as [[Confidence Intervals and Bootstrap|bootstrap]] does).

---

## Relationship to ML/DS

**Train/test splits are a sampling problem** — [[Grouped Train-Test Split]] exists precisely because naive random sampling can violate the "samples are exchangeable" assumption when related examples (same patient, same user) belong together, producing an optimistic, biased performance estimate that no amount of data volume fixes. **A/B test assignment** ([[A-B Testing]]) is simple random sampling applied to treatment/control assignment specifically. **Model evaluation itself** is a sampling problem: a reported test-set accuracy is one sample-based estimate of the true (population) accuracy, which is exactly why [[Confidence Intervals and Bootstrap]] exists — to characterize the sampling distribution of that one number.

---

## Interview-Ready Explanation

"Sampling distribution is the distribution of a *statistic* — like a sample mean — across many hypothetical repeated samples, not the distribution of the raw data itself. It's the theoretical bridge that lets you say how much to trust one number computed from one sample. The critical practical point: sample size fixes variance/noise, not bias — a huge biased sample is still biased, just more precisely so."

---

## Interview Questions

**What's the difference between the sample distribution and the sampling distribution of the mean?** The sample distribution is the spread of the actual raw values observed in one sample; the sampling distribution of the mean is the theoretical distribution of the *sample mean itself*, across many hypothetical repeated samples — a distribution of a computed statistic, not of raw data.

**Would collecting a much larger sample fix a survey that only reached convenience-sampled respondents?** No — more data reduces sampling variance (random noise) but does nothing about sampling bias (a systematic difference between who was sampled and the true population); a larger biased sample is a more precisely wrong estimate, not a more accurate one.

**Why does stratified sampling matter, and where else in this vault does the same idea appear?** It guarantees proportional representation of subgroups rather than leaving it to chance, which matters when a subgroup is small enough that random sampling alone might badly over- or under-represent it — the identical idea appears as stratified cross-validation ([[Cross Validation Strategy]]), preserving class balance across CV folds.

**In the tiny worked example above, why does the sampling distribution of the mean have less spread than the original population?** Averaging multiple draws tends to cancel out individual extreme values — a sample containing both a low and high draw averages toward the middle — so the *distribution of averages* is naturally more concentrated than the distribution of individual raw values; this variance-reduction-through-averaging is the same underlying mechanism the Central Limit Theorem formalizes.

**Why is a reported test-set accuracy considered a sample-based estimate rather than a fixed fact about the model?** The test set is itself one particular sample from the true underlying data distribution — a different test sample (even drawn from the same distribution) would give a somewhat different accuracy number, which is precisely why a confidence interval around a reported metric matters more than the single point estimate.

## Connections

- [[Central Limit Theorem]] — describes the shape of the sampling distribution of the mean specifically
- [[Confidence Intervals and Bootstrap]] — bootstrap approximates a sampling distribution by resampling when no analytical formula is convenient
- [[Grouped Train-Test Split]], [[Cross Validation Strategy]] — sampling-integrity concerns applied directly to ML evaluation
- [[Correlation vs Causation]] — selection bias as a sampling failure mode with causal-inference consequences
- [[A-B Testing]] — random assignment is simple random sampling applied to treatment groups

## One-line Summary

> A sampling distribution describes how a statistic (like a mean) behaves across repeated hypothetical samples, not how raw data is distributed within one sample — and the single highest-value distinction to remember is that larger samples fix variance/noise but never fix bias, which requires fixing the sampling method itself.
