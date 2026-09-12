# Hypothesis Test

## What is it?

A **hypothesis test** is a procedure for deciding whether an observed effect in data is likely real or could plausibly be explained by chance, by computing how surprising the observation would be under a "nothing is happening" assumption.

$$H_0: \theta = \theta_0 \quad \text{vs.} \quad H_1: \theta \neq \theta_0 \text{ (or } \theta > \theta_0 \text{, or } \theta < \theta_0\text{)}$$

Where $\theta$ is the parameter of interest (e.g. the true difference in advancement rate between model B and the baseline) and $\theta_0$ is its value under "no effect" (typically 0).

---

## Formal Setup

1. **Null hypothesis $H_0$:** the boring explanation — $\theta = \theta_0$.
2. **Alternative hypothesis $H_1$:** the effect you care about — $\theta \neq \theta_0$ (two-sided) or $\theta > \theta_0$ / $\theta < \theta_0$ (one-sided).
3. **Test statistic** $T = g(\mathcal{D})$: a function of the data that has a known distribution under $H_0$.
4. **[[P-Value]]:** $p = P(T \geq t_{\text{obs}} \mid H_0)$ (one-sided) or $p = P(|T| \geq |t_{\text{obs}}| \mid H_0)$ (two-sided).
5. **Decision rule:** reject $H_0$ if $p \leq \alpha$, where $\alpha$ is the significance level (commonly 0.05), fixed *before* looking at the data.

---

## Two Kinds of Error

|  | $H_0$ actually true | $H_0$ actually false |
|---|---|---|
| **Reject $H_0$** | Type I error, rate $= \alpha$ | Correct — true positive |
| **Fail to reject $H_0$** | Correct — true negative | Type II error, rate $= \beta$ |

$$\text{Power} = 1 - \beta = P(\text{reject } H_0 \mid H_1 \text{ true})$$

Power depends on three things you can control at design time: sample size $n$ (up), effect size $\theta - \theta_0$ (bigger effects are easier to detect), and significance level $\alpha$ (looser $\alpha$ trades more false positives for more power).

---

## Worked Example: Comparing Two Advancement Rates

Suppose baseline advances 42% of programs ($\hat p_1 = 0.42$, $n_1 = 500$) and a candidate model advances 47% ($\hat p_2 = 0.47$, $n_2 = 500$). A two-proportion z-test:

$$\hat p = \frac{n_1 \hat p_1 + n_2 \hat p_2}{n_1+n_2} = 0.445, \qquad SE = \sqrt{\hat p(1-\hat p)\left(\frac{1}{n_1}+\frac{1}{n_2}\right)} \approx 0.0314$$

$$z = \frac{\hat p_2 - \hat p_1}{SE} = \frac{0.05}{0.0314} \approx 1.59 \;\Rightarrow\; p \approx 0.11 \text{ (two-sided)}$$

At $\alpha = 0.05$, this does **not** clear the bar — a 5-point gap on 500 examples per arm isn't yet distinguishable from noise. This is exactly why comparing a candidate model to a baseline needs a real test, not just "the number looked bigger."

---

## Why It Matters Even More With Many Tests

Testing one hypothesis at $\alpha=0.05$ risks a 5% false-positive rate. Testing **hundreds** of hypotheses (e.g. one per market segment) means dozens of false positives are *expected* by chance — the [[Multiple Comparisons Problem]], corrected via [[Benjamini-Hochberg Procedure]]. A [[Permutation Test]] is the non-parametric version of exactly this machinery, useful when the test statistic's distribution under $H_0$ isn't analytically known.

---

## Common Mistakes

- Treating $p < 0.05$ as "the effect is real and large" — a p-value measures surprise, not magnitude
- Setting $\alpha$ *after* seeing the p-value (moving the goalposts)
- Running many tests and reporting only the significant ones without correction (p-hacking)
- Confusing "fail to reject $H_0$" with "$H_0$ is true" — absence of evidence isn't evidence of absence
- Ignoring power — a study with low power can fail to detect a real, important effect and get misread as "no effect"

## Interview / discussion questions

- What exactly does a p-value measure, and what's a common misinterpretation?
- Walk through computing a two-proportion z-test by hand and interpreting the result.
- Why does testing many hypotheses inflate the false-positive rate, and what's the fix?
- What's the difference between statistical significance and practical significance?

## Prerequisites

[[Random Variables]], [[Probability Distributions]], [[Central Limit Theorem]]

## Related concepts

[[P-Value]], [[Permutation Test]], [[Multiple Comparisons Problem]], [[Benjamini-Hochberg Procedure]], [[A-B Testing]], [[Correlation vs Causation]]

## Tags

#category/statistics #topic/hypothesis-testing #math/probability

## One-line summary

> A hypothesis test converts "does this effect look real" into a precise probability statement — how surprising the observed data would be if nothing were actually happening — and power/error-rate math tells you how much to trust the answer.
