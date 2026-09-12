# Random Variables

## What is it?

A **random variable** is a function that assigns a number to every possible outcome of a random process. It's the bridge that lets probability theory — which starts by talking about abstract "outcomes" — actually talk about numbers you can compute an average or variance of.

$$X: \Omega \to \mathbb{R}$$

$\Omega$ is the sample space (the set of all possible outcomes); $X$ maps each outcome $\omega \in \Omega$ to a real number $X(\omega)$. "The random variable $X$" really means "the numeric summary $X$ assigns to whatever actually happens."

---

## Why This Definition Matters (Not Just Formalism)

Before random variables, "probability" only assigns numbers to *events* ("probability it rains"). A random variable lets you ask numeric questions — what's the *average* rainfall, what's the *variance* in delivery time — by converting outcomes into numbers first. Every distribution, expectation, and variance calculation in this vault is built on top of this one conversion step.

**Concrete example**: flip a coin 3 times. $\Omega = \{HHH, HHT, HTH, THH, HTT, THT, TTH, TTT\}$ — 8 equally likely outcomes. Let $X$ = number of heads. Then $X(HHH)=3$, $X(HHT)=2$, ..., $X(TTT)=0$. $X$ turned an abstract outcome ("THT happened") into a number (1) you can average across repeated experiments.

---

## Discrete vs. Continuous

**Discrete random variable** — takes countably many values (often integers): number of heads in 3 flips, number of customer churns this month, number of retries before success. Described by a **probability mass function (PMF)**: $p(x) = P(X=x)$, with $\sum_x p(x) = 1$.

**Continuous random variable** — takes any value in an interval (or the whole real line): a person's height, time until a server crashes, a model's predicted probability. Described by a **probability density function (PDF)**: $f(x)$, where $P(a \le X \le b) = \int_a^b f(x)\,dx$, and $\int_{-\infty}^{\infty} f(x)\,dx = 1$.

**Crucial distinction that trips people up**: for a continuous random variable, $f(x)$ is *not* a probability — it can exceed 1. $P(X=x)$ for any single exact value $x$ is exactly $0$ (there are infinitely many possible values, so any one has zero mass). Only an *interval's* probability (the area under $f$ over that interval) is meaningful. This is why continuous distributions are described in terms of $P(a \le X \le b)$, never $P(X=x)$.

---

## The Cumulative Distribution Function (CDF)

$$F(x) = P(X \le x)$$

Works identically for discrete and continuous random variables (unlike PMF vs. PDF, which are different objects) — always between 0 and 1, always non-decreasing. For continuous $X$, $f(x) = F'(x)$ — the PDF is the derivative of the CDF. This is the object used to compute percentiles: the median is the $x$ where $F(x)=0.5$.

---

## Common Misconception

**"A random variable is random — it doesn't have a fixed value."** More precisely: a random variable is a *function*, fully determined once you specify $\Omega$ and how $X$ maps outcomes to numbers. What's random is *which outcome occurs*; the mapping itself is fixed. This distinction matters once you get to estimators in [[Hypothesis Test]] and [[Confidence Intervals and Bootstrap]] — an estimator (like a sample mean) is itself a random variable (it varies with which sample you happened to draw), even though the *formula* computing it is fixed.

---

## Relationship to ML/DS

Every model output is a random variable in disguise: a classifier's predicted probability, a regression model's residual, a metric computed on a randomly-drawn test set — all are random variables whose behavior across repeated sampling is what [[Confidence Intervals and Bootstrap]] and [[Hypothesis Test]] are built to characterize. "The model's accuracy is 85%" is really a statement about the *expected value* of a random variable (correct/incorrect per test example), not a fixed fact about the model.

---

## Interview-Ready Explanation (Say This Out Loud)

"A random variable maps outcomes of a random process to numbers, so we can do math — averages, variances — on things that are inherently uncertain. It's discrete if it takes countable values (like a count), continuous if it takes any value in a range (like a measurement). The core subtlety is that for continuous variables, individual points have zero probability; only ranges have positive probability."

---

## Interview Questions

**What's the difference between a PMF and a PDF?** PMF gives $P(X=x)$ directly for discrete $X$; PDF's value $f(x)$ is *not* a probability for continuous $X$ — only the area under $f$ over an interval is a probability, and $P(X=x)=0$ for any single value.

**Why is $P(X=x)=0$ for a continuous random variable, and does that mean it's impossible?** It's not impossible — it happens, just like a dart can land at an exact point on a continuous dartboard — but with infinitely many possible exact values, each individual one has zero probability mass; only intervals accumulate positive probability.

**Is a sample mean a random variable? Why does that matter?** Yes — it's a function of the randomly drawn sample, so it varies from sample to sample. This is exactly why a single sample mean needs a [[Confidence Intervals and Bootstrap|confidence interval]] around it rather than being reported as if it were the fixed true population value.

**Give an example of a discrete and a continuous random variable in an ML context.** Discrete: the number of misclassified examples in a batch. Continuous: a model's predicted probability score, or the time (latency) to serve one inference request.

**What's the relationship between the PDF and the CDF?** The CDF $F(x)=P(X\le x)$ is the running cumulative probability; the PDF is its derivative, $f(x)=F'(x)$ — the PDF describes the local "density" of probability, the CDF accumulates it.

---

## Connections

- [[Probability Distributions]] — the specific PMF/PDF shapes random variables commonly follow
- [[Expectation]], [[Variance and Standard Deviation]] — the two most important numeric summaries of a random variable
- [[Confidence Intervals and Bootstrap]] — built on the idea that an estimator is itself a random variable
- [[Central Limit Theorem]] — describes the distribution of a sum/average of random variables

## One-line Summary

> A random variable converts the outcome of a random process into a number, described by a PMF (discrete) or PDF (continuous) — and every statistic computed on sampled data is itself a random variable, which is the entire reason confidence intervals and hypothesis tests exist.
