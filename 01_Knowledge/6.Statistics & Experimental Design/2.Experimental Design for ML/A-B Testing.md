# A/B Testing

## What is it?

An **A/B test** is a controlled experiment that randomly assigns users (or sessions, or any other unit) to one of two (or more) variants — a control (A, the existing experience) and a treatment (B, the change being evaluated) — and compares an outcome metric between groups to decide whether the change causes a real improvement. It's the direct application of [[Hypothesis Test]] machinery to a specific, extremely common business decision: "did this change actually help?"

$$H_0: \mu_B = \mu_A \quad \text{vs.} \quad H_1: \mu_B \neq \mu_A$$

---

## Why Randomization Is the Whole Point

Comparing metrics for users who *chose* to use a new feature against users who didn't is comparing biased, self-selected groups — users who opt into a new feature may differ systematically from those who don't (more engaged, more tech-savvy, etc.), and any difference in outcome could be caused by that pre-existing difference rather than the feature itself. This is exactly the [[Correlation vs Causation|correlation-vs-causation]] trap. **Random assignment** is what breaks this: by construction, randomization makes the two groups statistically identical in expectation on every dimension — observed and unobserved — except which variant they received, so any systematic difference in outcome can be attributed to the variant itself.

---

## Designing the Test

**Choosing a metric**: pick a primary metric *before* running the test, ideally one directly tied to the business question ("does this increase conversion") rather than a vanity metric that's easy to move without actually mattering. Also track guardrail metrics (e.g. page load time, unsubscribe rate) to catch a "win" on the primary metric that comes at an unacceptable cost elsewhere.

**Sample size / statistical power**: an underpowered test (too few users) can fail to detect a real effect that exists, wasting the experiment — see [[Hypothesis Test]]'s treatment of power and effect size; sample size should be calculated *before* running the test based on the minimum effect size worth detecting, not decided arbitrarily.

**Test duration**: run long enough to capture a full natural cycle of user behavior (typically at least one full week, to average over day-of-week effects) and to reach the pre-calculated sample size — stopping early because results look promising is a specific and common mistake, covered next.

## The Peeking Problem

Checking a test's results repeatedly and stopping as soon as $p<0.05$ is observed inflates the false-positive rate dramatically — this is a specific instance of the [[Multiple Comparisons Problem]], where each "peek" at an in-progress test is effectively another hypothesis test, and stopping at the first one that clears significance is exactly the p-hacking pattern that problem describes. **Fix**: decide the sample size and stopping point in advance and only look at the final result (a "fixed-horizon" test), or use a sequential testing method specifically designed to allow valid peeking (e.g. sequential probability ratio tests, or Bayesian methods with different guarantees) — but naive repeated peeking with a standard fixed-sample test is not valid.

---

## Diagnosing Metric Changes

When a metric moves after a launch (an A/B test result, or any observed change in a live metric), the DA/DS-specific skill is distinguishing genuine causal effect from confounding factors: was there a simultaneous external event (seasonality, a marketing campaign, a competitor's action)? Did the population being measured shift (a change in traffic mix, not user behavior)? Is the change within normal metric variance, or genuinely outside it? An A/B test's randomization is precisely what makes this diagnosis tractable for the specific change being tested — a metric move observed *without* a controlled experiment behind it requires much more careful, and much less certain, causal reasoning.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| Peeking and stopping early | Inflated false-positive rate (multiple implicit tests) | Fixed sample size decided upfront, or valid sequential testing methods |
| No pre-registered primary metric | Post-hoc metric selection is p-hacking in disguise | Choose the primary metric before launching the test |
| Ignoring guardrail metrics | A "win" that hides an unacceptable cost elsewhere | Track guardrails alongside the primary metric |
| Underpowered test | Real effects go undetected, test wrongly concluded "no effect" | Calculate required sample size from the minimum effect size worth detecting, before running |
| Novelty effect | Short-term lift from users noticing something new, not a durable improvement | Run long enough, and consider a holdout to check effect persistence |

---

## Interview Questions

**Why is randomization necessary rather than just comparing users who adopted a feature against those who didn't?** Self-selected adopters differ systematically from non-adopters on both observed and unobserved dimensions, so any outcome difference could be caused by those pre-existing differences rather than the feature — randomization makes the groups statistically identical in expectation on everything except the variant assigned, isolating the causal effect of the variant itself.

**What's wrong with checking an A/B test's results every day and stopping as soon as it hits significance?** Each check is effectively another hypothesis test on the same accumulating data, and stopping at the first one that clears $p<0.05$ dramatically inflates the true false-positive rate — the same multiple-comparisons problem as running many tests and reporting only the one that happened to be significant.

**A test shows a statistically significant 0.3% lift in conversion — do you ship it?** Statistical significance alone doesn't mean the effect is practically meaningful — check the effect size against the cost of shipping and maintaining the change, check guardrail metrics for hidden costs, and consider whether 0.3% is large enough to matter at the given traffic volume before treating "significant" as synonymous with "worth shipping."

## Connections

- [[Hypothesis Test]], [[P-Value]] — the statistical machinery this note applies to a specific business decision
- [[Multiple Comparisons Problem]] — the peeking problem is a specific instance of this
- [[Confidence Intervals and Bootstrap]] — report an effect size with a confidence interval, not just a p-value
- [[Correlation vs Causation]] — why randomization, not just observation, is required to draw a causal conclusion
- [[Metric Selection and Diagnosing Change]] (module 20, ML & DL) — choosing the right metric and diagnosing a metric change are core product-sense skills this note's statistical machinery supports

## One-line Summary

> A/B testing applies hypothesis testing to a controlled, randomized comparison between variants — randomization is what licenses a causal conclusion, and the most common way to invalidate that conclusion in practice is peeking at results and stopping early, which is p-hacking in disguise.
