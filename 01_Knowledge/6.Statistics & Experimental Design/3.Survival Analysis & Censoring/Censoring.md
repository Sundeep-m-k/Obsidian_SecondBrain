# Censoring

## What is it?

**Censoring** happens when the true outcome for an observation isn't yet known because the event of interest hasn't happened yet (or tracking stopped) by the time you're analyzing the data. Core to survival analysis, but relevant anywhere you measure "does X eventually happen" from data collected at a fixed point in time.

## Formal Framing

Let $T$ be the true (possibly unobserved) time to the event of interest (e.g. time to reaching the next phase), and $C$ be the observation cutoff time (e.g. "now," or a fixed follow-up window). Define:

$$\tilde T = \min(T, C), \qquad \delta = \mathbf{1}[T \leq C]$$

$\tilde T$ is what you actually observe, and $\delta$ ("event indicator") tells you whether $\tilde T$ is a real event time ($\delta=1$) or a **right-censored** observation ($\delta=0$, meaning you only know $T > \tilde T$, not the actual value).

The **survival function** $S(t) = P(T > t)$ describes the probability the event still hasn't happened by time $t$; censored observations contribute partial information to estimating $S(t)$ (they confirm survival up to $\tilde T$) without pinning down the exact event time.

---

## The Three-Way Outcome Split

| Outcome | Meaning | $\delta$ | Label used here |
|---|---|---|---|
| **Success** | Event observed: $T \leq C$ | 1 | 1 |
| **Failure** | $\tilde T = C_{\text{timeout}}$ with no event, or a terminal signal (status = terminated) before timeout | 0 (but treated as resolved) | 0 |
| **Censored** | Still ongoing, $\tilde T < C_{\text{timeout}}$, no event yet | 0 | null — excluded from training as a hard 0 |

Note the subtlety: standard survival analysis treats *all* $\delta=0$ cases as censored and uses them in a likelihood that properly accounts for partial information ($T > \tilde T$). Here, a stricter three-way split is used instead — only genuinely still-running, under-timeout cases are left censored; timed-out or terminated cases are converted to hard failures. This is a deliberate simplification that trades some statistical elegance for a labeling scheme simple enough to train a standard classifier on.

---

## Why Conflating Censored-With-Failed Is a Serious Bug

If $\delta=0$ (no event yet) is coded as $y=0$ (failure) without distinguishing "still running, not enough time passed" from "confirmed dead," the estimated failure rate for recent cohorts is systematically biased upward — recent cohorts haven't had time to succeed *or* fail, they just haven't finished. This corrupts labels precisely where you'd want reliability most: the newest, most actionable data.

## The Timeout Window as a Modeling Choice

Since $C$ can't be infinite in practice, a fixed timeout $C_{\text{timeout}}$ (e.g. 4 years with no success) converts "still running" into "presumed failed" once continued success becomes unlikely. This threshold has a real, measurable effect on reported rates — worth stress-testing by comparing e.g. a 3-year vs. 4-year cutoff and checking how much conclusions move, rather than picking arbitrarily.

## Common Mistakes

- Coding all $\delta=0$ rows as failures — inflates the apparent failure rate for anything recent
- Dropping all censored rows without checking whether they skew recent (if so, dropping them biases conclusions toward older, already-resolved cohorts)
- Picking $C_{\text{timeout}}$ without a sensitivity check on how much it moves the headline numbers

## Interview / discussion questions

- Define $\tilde T$ and $\delta$ formally and explain why "still running" is different from a normal missing value.
- What happens to failure-rate estimates for recent data if censored rows are mislabeled as failures?
- How would you check whether your timeout window choice (3 vs. 4 years) meaningfully changes conclusions?

## Prerequisites

Basic probability, [[Classification]]

## Related concepts

[[Data Leakage]], [[Baseline Estimator]]

## Tags

#category/statistics #topic/survival-analysis #math/probability

## One-line summary

> Censoring means $T > \tilde T$ is known but $T$ itself isn't — conflating "still running, $\delta=0$" with "failed" silently and systematically corrupts labels for exactly the most recent, most important data.
