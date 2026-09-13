---
tags: [category/statistics, topic/probability, index, moc]
---

# Probability Foundations — Index

> **Position in vault**: `6.Statistics & Experimental Design/0.Probability Foundations/`
> **Purpose**: The probability theory that [[Hypothesis Test]], [[P-Value]], [[A-B Testing]], and everything else in this Statistics module assume as a prerequisite without building. Added 2026-09 after an interview-readiness audit found zero probability coverage anywhere in the vault. Numbered `0` (not `1`) so it sorts before Hypothesis Testing without renumbering the existing four Statistics modules.
> **Prerequisite**: none — this is the foundation everything else in Statistics builds on.

## Section Map

| Note | Covers |
|---|---|
| [[Random Variables]] | Mapping outcomes to numbers; PMF vs. PDF; discrete vs. continuous |
| [[Probability Distributions]] | Binomial, Poisson, Normal, Exponential; when normal assumptions fail |
| [[Expectation]] | Probability-weighted average; linearity (always holds) |
| [[Variance and Standard Deviation]] | Spread around the mean; additivity requires independence |
| [[Covariance and Correlation]] | Linear association between two variables; correlation's unit-free normalization |
| [[Conditional Probability]] | Updating probability given evidence; independence vs. mutual exclusivity |
| [[Bayes' Theorem]] | Reversing a conditional probability; prior/likelihood/posterior |
| [[Sampling and Sampling Distributions]] | Population vs. sample vs. sampling distribution; sampling bias vs. sample size |
| [[Law of Large Numbers]] | Why the sample average converges to the true mean as $n$ grows — convergence, not shape |
| [[Central Limit Theorem]] | Why sample means trend normal regardless of the population's shape — shape, not just convergence |

## Reading Order

Random Variables → Probability Distributions → Expectation → Variance → Covariance and Correlation → Conditional Probability → Bayes' Theorem → Sampling and Sampling Distributions → Law of Large Numbers → Central Limit Theorem (read these last two together — they're the pair most often confused) → then [[Hypothesis Test]] and onward into the rest of this Statistics module.

## Worked Synthesis: Explaining a P-Value to a Non-Technical Stakeholder

A frequent Data Analyst/Data Scientist interview exercise, and a genuine on-the-job skill this module's formal notes don't rehearse directly — here's the compressed version, built from [[Random Variables]], [[Sampling and Sampling Distributions]], and [[P-Value]]:

*"We ran a test comparing the new checkout flow against the old one. A p-value of 0.03 means: if the new flow genuinely made no difference at all, there'd only be a 3% chance we'd see a gap this large or larger just from random noise in who happened to land in each group. That's small enough that we don't think it's just luck — but it doesn't tell us the new flow definitely works, and it doesn't tell us *how much* better it is. For that we'd also look at the actual size of the improvement and its confidence interval, and weigh whether that size is big enough to matter for the business, not just statistically detectable."*

The three things this compressed explanation deliberately does that a purely formal p-value definition doesn't: (1) frames it in terms of "if nothing were happening" rather than jargon like $H_0$, (2) explicitly says what it does *not* claim ("doesn't mean it definitely works"), (3) redirects to effect size/practical significance, since a stakeholder's actual question is almost always "should we ship this," not "is p<0.05."

## Quick Reference: Prior vs. Likelihood vs. Posterior

| Term | Question it answers | Symbol |
|---|---|---|
| Prior | What did we believe before seeing this evidence? | $P(A)$ |
| Likelihood | How probable is this evidence, if the hypothesis were true? | $P(B\mid A)$ |
| Posterior | What should we believe now, after the evidence? | $P(A\mid B)$ |

A [[P-Value]] is a **likelihood** ($P(\text{data}\mid H_0)$) — confusing it for a **posterior** ($P(H_0\mid\text{data})$) is the single most common statistical misinterpretation, and it's the same underlying mistake as [[Bayes' Theorem]]'s "base rate neglect."

## Common Exam / Interview Questions

1. Explain covariance vs. correlation — why does correlation exist as a separate concept?
2. A test is 95% accurate for a disease affecting 1% of people — why is the true positive-predictive-value so much lower than 95%?
3. Why does the Central Limit Theorem matter in practice, in plain language?
4. When do normal-distribution assumptions fail, and what would you do instead?
5. What's the difference between a population distribution, a sample distribution, and a sampling distribution?

## One-line Summary

> This module builds the probability machinery — random variables, distributions, expectation/variance/covariance, conditional probability, Bayes' theorem, sampling, and the CLT — that every hypothesis test, confidence interval, and A/B test in the rest of this Statistics subject assumes as already understood.
