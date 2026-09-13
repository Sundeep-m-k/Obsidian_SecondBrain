# Law of Large Numbers

## What is it?

The **Law of Large Numbers (LLN)** says that as sample size grows, the **sample average converges to the true expected value** — the formal justification for the intuition that "more data makes the average stabilize."

$$\bar{X}_n = \frac{1}{n}\sum_{i=1}^n X_i \ \xrightarrow{\ n \to \infty\ } \ \mathbb{E}[X]$$

under the standard conditions: the $X_i$ are independent and identically distributed, with finite [[Expectation]].

---

## LLN vs. CLT — The Distinction Interviews Actually Probe

Both describe a sample average's behavior as $n$ grows, and are frequently confused precisely because they're about the same object — but they answer **different questions**:

- **LLN is concerned with *convergence*** — does the sample average actually get closer to the true population mean as $n$ increases? (Yes, under mild conditions.) It says nothing about the *shape* of the remaining fluctuation around that mean — just that it shrinks toward zero.
- **[[Central Limit Theorem|CLT]] is concerned with the *distribution* of the properly normalized sample mean around the population mean** — specifically, that $\sqrt{n}(\bar{X}_n - \mu)$ approaches a **normal distribution** as $n$ grows, regardless of the original population's own shape. CLT is a much sharper, more specific statement — it doesn't just say the average converges, it says *how* the remaining error is distributed.

**The one-line version worth memorizing**: *"more data makes the average stabilize"* is LLN; *"the sampling distribution of the mean becomes approximately normal"* is CLT. LLN is about the destination; CLT is about the shape of the journey there.

---

## Worked Example: Repeated Coin Flips

Flip a fair coin repeatedly, recording the running proportion of heads:

| Flips ($n$) | Example running proportion of heads |
|---|---|
| 10 | 0.70 (7 heads — a plausible, sizable deviation from 0.5 at small $n$) |
| 100 | 0.56 (closer to 0.5, but still a visible gap) |
| 10,000 | 0.503 (very close to 0.5) |
| 1,000,000 | 0.5001 (essentially 0.5) |

**This progression — the running proportion getting closer and closer to the true $p=0.5$ — is the Law of Large Numbers, directly observed.** It says nothing on its own about *how big* a typical deviation is at any given $n$ or what shape that deviation's distribution takes — that additional, sharper claim (that the deviation, appropriately rescaled, looks normal) is exactly what the [[Central Limit Theorem]] adds on top.

---

## Common Misconception: The "Gambler's Fallacy" Is a Misapplication of LLN

**Wrong reasoning**: "the coin has landed heads 8 times in a row, so tails is 'due' to balance things out." This misapplies LLN — the law says the *running average* converges as $n \to \infty$, **not** that any specific future outcome is influenced by the past in order to force that convergence. Each flip remains independent (probability 0.5 regardless of history); convergence happens simply because $n$ new, independent flips eventually dilute the influence of any earlier streak on the overall average — not because the coin "corrects" itself. This distinction — convergence through dilution by future independent data, not correction of past data — is precisely what separates a correct understanding of LLN from the gambler's fallacy.

---

## Relationship to ML/DS

**Why larger training/evaluation sets give more trustworthy metrics**: a model's evaluation accuracy on $n$ test examples is itself a sample average (of a correct/incorrect indicator per example); LLN is the guarantee that this average converges toward the model's *true* population accuracy as $n$ grows — the direct justification for "a bigger test set gives a more reliable accuracy estimate," independent of the separate, sharper CLT-based question of how to construct a confidence interval *around* that estimate at a given $n$ (see [[Confidence Intervals and Bootstrap]]). **Monte Carlo methods** (estimating an expectation by averaging many random samples) rely on LLN directly for their validity — the average of enough random draws converges to the true expected value being estimated.

---

## Interview-Ready Explanation

"The Law of Large Numbers says a sample average converges to the true population mean as the sample gets larger — it's why more data makes an estimate more trustworthy. It's easy to confuse with the CLT because both describe the sample mean's behavior, but LLN only promises convergence; it doesn't say anything about the *shape* of the fluctuation around the mean before you get there. The CLT is the separate, sharper claim that this fluctuation looks normal, once properly rescaled."

---

## Interview Questions

**What's the actual difference between the Law of Large Numbers and the Central Limit Theorem?** LLN says the sample average converges to the true mean as $n$ grows — a statement about convergence. CLT says the *properly normalized* sample mean's distribution approaches normal as $n$ grows — a statement about the shape of the remaining error, which is a sharper, more specific claim than LLN makes on its own.

**Why is "the coin is due for tails after 8 heads in a row" a misunderstanding of the Law of Large Numbers?** LLN guarantees the *running average* converges over many future independent flips — each individual flip remains independent and unaffected by past outcomes; convergence happens because new data dilutes the influence of any past streak on the overall average, not because any future outcome is forced to "balance" the past.

**Why does a larger test set give a more trustworthy accuracy estimate, in LLN terms specifically (as opposed to a CLT-based confidence interval)?** The test accuracy is a sample average of a correct/incorrect indicator across test examples; LLN guarantees this average converges toward the model's true population accuracy as the test set grows — a direct justification for trusting a larger test set's point estimate more, independent of the separate question (answered via CLT) of how wide a confidence interval to put around that estimate.

**Does the Law of Large Numbers say anything about how large a deviation from the true mean to expect at a specific, finite sample size?** No — LLN is an asymptotic statement about convergence as $n\to\infty$; quantifying the *typical size* of the deviation at a specific finite $n$, and its distributional shape, is what the Central Limit Theorem (and the standard-error machinery built on it) provides instead.

## Connections

- [[Central Limit Theorem]] — the sharper, complementary claim about the *shape* of convergence LLN doesn't itself address
- [[Expectation]] — the quantity the sample average is converging to
- [[Sampling and Sampling Distributions]] — the general machinery both LLN and CLT are specific statements within
- [[Confidence Intervals and Bootstrap]] — relies on CLT (or resampling) for interval width, not on LLN alone

## One-line Summary

> The Law of Large Numbers guarantees a sample average converges to the true expected value as sample size grows — "more data stabilizes the average" — while the Central Limit Theorem separately and more specifically describes the normal *shape* that convergence's remaining error takes; conflating the two, or mistaking LLN for a claim that individual future outcomes "correct" past ones, are the two most common ways this topic gets misunderstood.
