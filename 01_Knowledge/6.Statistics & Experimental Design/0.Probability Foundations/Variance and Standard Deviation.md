# Variance and Standard Deviation

## What is it?

**Variance** measures how spread out a random variable's values are around its mean — the expected squared deviation from the mean.

$$\text{Var}(X) = \mathbb{E}[(X-\mathbb{E}[X])^2] = \mathbb{E}[X^2] - (\mathbb{E}[X])^2$$

The second form (expectation of the square, minus the square of the expectation) is the standard computational shortcut — usually easier to compute than the definitional form directly.

$$\text{SD}(X) = \sqrt{\text{Var}(X)}$$

**Standard deviation** is variance's square root, used far more often in reporting because it's in the *same units* as $X$ itself (variance of a variable measured in dollars is in dollars², which isn't directly interpretable; standard deviation is back in dollars).

---

## Worked Numerical Example

Two models' per-fold accuracy on 5-fold CV:
- Model A: $[0.84, 0.85, 0.86, 0.84, 0.85]$ → mean $= 0.848$
- Model B: $[0.70, 0.95, 0.75, 0.90, 0.85]$ → mean $= 0.83$

Model B has *higher* mean but look at spread:
$$\text{Var}(A) = \frac{(0.84{-}0.848)^2+\cdots}{5} \approx 0.00006, \quad \text{Var}(B) = \frac{(0.70{-}0.83)^2+\cdots}{5} \approx 0.0084$$

Model B's variance is roughly 140x larger — Model A is far more *reliable* despite the lower average. This is exactly why [[Cross Validation Strategy]] reports mean ± standard deviation across folds, not just the mean: a model whose performance swings wildly across folds is riskier to deploy than a slightly-lower-but-stable one, even before any test set is touched.

---

## Key Properties

**Variance does NOT add under a sum in general** — $\text{Var}(X+Y) = \text{Var}(X)+\text{Var}(Y)+2\,\text{Cov}(X,Y)$. It only simplifies to $\text{Var}(X)+\text{Var}(Y)$ when $X$ and $Y$ are **independent** (making [[Covariance and Correlation]] zero). This is a common point of confusion with [[Expectation]]'s linearity, which holds unconditionally — variance's additivity requires independence, expectation's does not.

**Variance is always non-negative** and equals zero only when $X$ is a constant (no randomness at all).

**Scaling**: $\text{Var}(aX) = a^2\text{Var}(X)$ — doubling a variable's scale quadruples its variance (but only doubles its standard deviation), which is why standard deviation, not variance, is the more intuitive "spread" measure to reason about directly.

## Common Misconception

**"Low variance always means a better estimator/model."** Not quite — an estimator that always outputs a fixed, wrong number has *zero* variance but is completely useless (it's maximally *biased*). Variance only matters in combination with bias — which is exactly the [[Bias Variance Tradeoff]]'s point: minimizing variance alone, ignoring bias, produces a stable but wrong model.

---

## Relationship to ML/DS

**The [[Bias Variance Tradeoff]] decomposition is literally a variance decomposition** — a model's expected squared error splits into (bias)² + variance + irreducible noise, derived by expanding $\mathbb{E}[(y-\hat{f}(x))^2]$ using the definition of variance above. **[[Cross Validation Strategy]]'s mean ± std reporting** is a direct, practical application: the standard deviation across folds is an empirical estimate of how much the model's true performance would vary if trained on a different sample. **[[Confidence Intervals and Bootstrap|Standard error]]** — the standard deviation of a *sample mean* specifically (not of individual observations) — is $\sigma/\sqrt{n}$, directly derived from variance's scaling property applied to a sum of $n$ independent observations, and it's the quantity every confidence interval width is built from.

---

## Interview-Ready Explanation

"Variance measures spread — the average squared distance from the mean. It's foundational because two things can have the same average but wildly different reliability, and variance is what distinguishes them. In ML specifically, it's one half of the bias-variance tradeoff — a model with high variance is unstable across different training samples, and it's also directly what's reported (as standard deviation) alongside a cross-validation mean, because a model's average CV score alone hides how much you should trust that number."

---

## Interview Questions

**Why does standard deviation get reported more often than variance?** It's in the same units as the original variable (variance is in squared units, which usually isn't directly interpretable — e.g. "dollars squared" means nothing intuitively, while "$50" does).

**When does $\text{Var}(X+Y) = \text{Var}(X) + \text{Var}(Y)$, and when does it not?** Only when $X$ and $Y$ are independent (or at least uncorrelated, $\text{Cov}(X,Y)=0$); in general there's an extra $2\,\text{Cov}(X,Y)$ term, meaning positively correlated variables produce *more* combined variance than either alone, and negatively correlated variables can partially cancel out.

**Two models have the same mean cross-validation accuracy, but one has much higher variance across folds — which would you deploy, and why?** Generally the lower-variance model, all else equal — a model whose performance swings wildly across different training samples is a signal of instability that's likely to show up as inconsistent real-world performance, even though its *average* looks identical on paper; report both mean and variance, never mean alone.

**Why isn't "low variance" alone a sign of a good estimator?** A constant, always-wrong prediction has zero variance but is maximally biased — variance only measures consistency, not correctness; the two must be weighed together, which is exactly the bias-variance tradeoff's framing.

**How does variance relate to the standard error of a sample mean?** The standard error is $\sigma/\sqrt{n}$ — the standard deviation of the *sampling distribution of the mean*, derived from variance's scaling property applied to an average of $n$ independent observations; it's the quantity that directly determines how wide a confidence interval needs to be.

## Connections

- [[Expectation]] — variance is defined in terms of it ($\mathbb{E}[(X-\mathbb{E}[X])^2]$)
- [[Covariance and Correlation]] — governs whether variance is additive under summation
- [[Bias Variance Tradeoff]] — the direct ML application of this decomposition
- [[Cross Validation Strategy]] — mean ± std reporting is a practical variance application
- [[Confidence Intervals and Bootstrap]] — standard error (derived from variance) determines CI width
- [[Central Limit Theorem]] — describes how a sample mean's variance shrinks as $1/n$ with sample size

## One-line Summary

> Variance measures spread around the mean and only adds across a sum when the variables are independent (unlike expectation, which is always linear) — in ML, it's half of the bias-variance tradeoff and the direct source of a cross-validation standard deviation and a confidence interval's width.
