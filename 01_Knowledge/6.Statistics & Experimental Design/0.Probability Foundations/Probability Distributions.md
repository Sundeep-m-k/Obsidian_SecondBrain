# Probability Distributions

## What is it?

A **probability distribution** is the complete description of how likely every possible value of a [[Random Variables|random variable]] is. Certain shapes recur so often across nature and data that they have names and known formulas — knowing which distribution a quantity plausibly follows lets you compute probabilities, choose the right statistical test, and recognize when a modeling assumption is being violated.

---

## Discrete Distributions

**Bernoulli** — a single trial with two outcomes (success/failure), $P(X=1)=p$, $P(X=0)=1-p$. The building block of every other discrete distribution below. *ML example*: a single binary classification outcome.

**Binomial** — the number of successes in $n$ independent Bernoulli trials with the same $p$:
$$P(X=k) = \binom{n}{k}p^k(1-p)^{n-k}, \quad \mathbb{E}[X]=np, \quad \text{Var}(X)=np(1-p)$$
*Example*: number of converting users out of 1,000 shown a new feature, assuming each converts independently with the same probability.

**Poisson** — the number of events occurring in a fixed interval, when events happen independently at a constant average rate $\lambda$:
$$P(X=k) = \frac{\lambda^k e^{-\lambda}}{k!}, \quad \mathbb{E}[X]=\text{Var}(X)=\lambda$$
*Example*: number of server requests per second, number of customer support tickets per day. The distinctive property — mean equals variance — is itself a diagnostic: if a count variable's variance is much larger than its mean ("overdispersion"), Poisson is the wrong model.

---

## Continuous Distributions

**Uniform** — every value in $[a,b]$ equally likely: $f(x) = \frac{1}{b-a}$. The distribution of "no information beyond a known range" — often the natural default for something like a randomly assigned lottery number or a properly randomized A/B test's assignment mechanism itself.

**Normal (Gaussian)** — the bell curve:
$$f(x) = \frac{1}{\sigma\sqrt{2\pi}}e^{-\frac{(x-\mu)^2}{2\sigma^2}}, \quad \mathbb{E}[X]=\mu, \quad \text{Var}(X)=\sigma^2$$
Symmetric, fully described by just $\mu$ and $\sigma^2$. Ubiquitous partly because of the [[Central Limit Theorem]] — many real quantities are themselves sums of many small independent effects (height = genetics + nutrition + many small factors), and sums of independent effects trend toward normal regardless of the individual effects' own distributions.

**Exponential** — the time between events in a Poisson process (waiting time until the next event, given a constant rate $\lambda$):
$$f(x) = \lambda e^{-\lambda x}, \quad x \ge 0, \quad \mathbb{E}[X] = 1/\lambda$$
*Example*: time between consecutive server requests, time until component failure. Has the **memoryless property** — $P(X > s+t \mid X>s) = P(X>t)$: having already waited $s$ minutes doesn't change the distribution of additional waiting time, which is a strong, often-wrong assumption (real failures often *do* depend on age/wear, violating memorylessness).

---

## Why Distribution Choice Actually Matters

Choosing the wrong distribution silently breaks downstream calculations. A [[Hypothesis Test]] built on a normality assumption produces a wrong p-value if the data is actually heavily skewed. A Poisson-based forecast underestimates uncertainty if the real process is overdispersed. This isn't pedantry — it's the difference between a confidence interval that actually contains the true value ~95% of the time and one that doesn't, silently.

## When Normal-Distribution Assumptions Fail

- **Heavy skew** — income, wait times, and most "count of rare-but-large events" data are right-skewed; the normal's symmetric bell shape systematically underestimates the probability of extreme values.
- **Bounded data treated as unbounded** — a percentage or proportion is bounded in $[0,1]$; a normal approximation can nonsensically predict values outside that range, especially near 0 or 1.
- **Heavy tails** — financial returns, latency distributions, and many real-world "occasionally something extreme happens" phenomena have more extreme-value probability than a normal distribution assigns, meaning normal-based risk estimates understate tail risk.
- **Small sample sizes** — the [[Central Limit Theorem]]'s normal approximation for a sample mean requires $n$ to be reasonably large; for small $n$ from a non-normal population, the approximation can be poor, motivating t-distributions or non-parametric methods like [[Permutation Test]] instead.

---

## Interview-Ready Explanation

"A probability distribution tells you the full shape of uncertainty for a variable — not just its average, but how likely every value is. A handful of named distributions (binomial, Poisson, normal, exponential) recur because they arise naturally from simple generative processes — repeated independent trials, counts of rare events, sums of many small effects, waiting times. Picking the right one isn't cosmetic — it determines whether your p-values and confidence intervals are actually valid."

---

## Interview Questions

**Why does mean equal variance for a Poisson distribution, and why is that useful diagnostically?** It's a direct mathematical property of the Poisson model's derivation from a constant-rate, memoryless event process; observing real data where variance far exceeds the mean ("overdispersion") is a signal that the constant-rate/independence assumptions are violated and a different model (e.g. negative binomial) is more appropriate.

**When would you use a Poisson distribution vs. a binomial distribution?** Binomial counts successes out of a *fixed, known number of trials* with a per-trial probability; Poisson counts events in a *fixed interval of time/space* with no natural "number of trials" — a Poisson is actually the limiting case of a binomial as $n\to\infty$ and $p\to0$ with $np$ held constant.

**Why do so many real-world quantities appear approximately normal?** Because many observed quantities are themselves the sum of many small, roughly-independent contributing factors, and the [[Central Limit Theorem]] guarantees such sums trend toward a normal distribution regardless of each individual factor's own distribution — this is a consequence of summation, not evidence that any single underlying mechanism is inherently "normal."

**A dataset of customer wait times looks like it should be normal, but a QQ-plot shows heavy right skew — what would you do differently?** Don't apply normal-based tests/CIs directly; consider a log transformation (wait times are often log-normal), an exponential/gamma model if the process is plausibly a constant-rate arrival process, or a non-parametric method like [[Permutation Test]] or [[Confidence Intervals and Bootstrap|bootstrap]] that doesn't assume a specific distributional shape.

**What's the memoryless property of the exponential distribution, and when is it a bad assumption?** Having already waited some time doesn't change the distribution of remaining waiting time — a bad assumption whenever the underlying risk changes with age/history, e.g. mechanical wear (older components fail more often) or a customer's churn risk depending on how long they've already been a customer.

## Connections

- [[Random Variables]] — the object these distributions describe
- [[Central Limit Theorem]] — explains the normal distribution's ubiquity
- [[Expectation]], [[Variance and Standard Deviation]] — the formulas above are specific instances of these general definitions
- [[Hypothesis Test]] — most classical tests assume a specific distribution (often normal) for the test statistic
- [[Confidence Intervals and Bootstrap]] — bootstrap exists specifically to sidestep needing to name a distribution at all

## One-line Summary

> A handful of named distributions (binomial, Poisson, normal, exponential) arise from simple generative processes and recur constantly in real data — choosing the wrong one doesn't just look wrong on a plot, it silently invalidates the p-values and confidence intervals built on top of it.
