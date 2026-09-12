# Metric Selection and Diagnosing Change

## What is it?

Two closely related, extremely common Data Analyst/Data Scientist interview and on-the-job skills: **choosing the right metric** to represent success for a given goal, and **diagnosing why a metric moved** once it has. Neither is primarily a modeling problem — both are reasoning skills about what a number actually represents and what could make it change.

---

## Choosing a Metric

**A good metric is directly tied to the actual goal, hard to game, and sensitive enough to detect real changes.** A metric that's easy to move without actually achieving the underlying goal (a **vanity metric** — e.g. total signups, when the real goal is engaged, retained users) creates an incentive to optimize the metric instead of the thing it was meant to represent — a specific instance of Goodhart's Law ("when a measure becomes a target, it ceases to be a good measure").

**North Star metrics vs. guardrail metrics**: a single primary metric usually can't capture a whole business goal without risk of being gamed or missing an important tradeoff — pairing a North Star metric (the primary success signal) with guardrail metrics (things that must *not* get worse, like latency, unsubscribe rate, cost) catches the case where an improvement on the primary metric comes at an unacceptable cost elsewhere, exactly the same discipline as [[A-B Testing]]'s guardrail-metric practice.

**Leading vs. lagging indicators**: a lagging metric (churn, quarterly revenue) confirms an outcome after the fact but is too slow to act on; a leading metric (weekly active usage, a specific engagement action known to predict retention) allows earlier intervention, at the cost of being a proxy rather than the actual outcome of interest, and proxies can decouple from the true outcome without warning if the underlying relationship changes.

---

## Diagnosing a Metric Change

When a metric moves, the systematic diagnostic sequence (extending [[Framing Ambiguous Business Problems]]'s framing skill to a specific, already-observed change):

1. **Rule out a measurement/data problem first.** A broken tracking pipeline, a change in how the metric is computed, or a reporting lag can produce an apparent change that isn't real — this is the cheapest hypothesis to check and the most common actual cause of a sudden, unexplained metric shift.
2. **Check for a known, simultaneous cause.** A recent release, an outage, a marketing campaign starting or ending, a seasonal pattern (see [[Time Series Fundamentals]]) — before assuming a genuine underlying behavioral shift, rule out an identifiable external event that coincides with the change.
3. **Segment the change.** Is the move uniform across all users/segments, or concentrated in one specific segment (a platform, a geography, a user cohort)? A [[Clustering]]-style breakdown of *where* the change is concentrated often reveals the actual cause far faster than analyzing the aggregate metric alone.
4. **Distinguish population-mix shift from behavior change.** A metric can move because the *composition* of users changed (more new users, who behave differently on average, without any existing user's behavior actually changing) rather than because any individual user's behavior shifted — this distinction changes what response, if any, is appropriate.
5. **Only then, consider a causal investigation** ([[Correlation vs Causation]], potentially a follow-up [[A-B Testing|A/B test]]) if steps 1–4 don't explain the change and a specific causal hypothesis needs testing.

---

## Interview Questions

**A company wants to measure "user happiness" — what metric would you propose, and what's the risk of getting this wrong?** Push back on "happiness" as unmeasurable directly, and propose an operational proxy tied to a concrete, gameable-resistant behavior (e.g. return usage rate, a specific completed-action rate) rather than a survey-only metric alone; the risk of choosing poorly is optimizing a metric that diverges from what it was meant to represent (Goodhart's Law) — e.g. optimizing signups when the real goal was retained, satisfied users.

**Revenue dropped 5% last month — what's your diagnostic process before proposing any model?** Rule out a measurement/reporting artifact first (a tracking or attribution bug), check for a known simultaneous cause (a pricing change, a major customer churning, a seasonal pattern), segment the drop to see if it's concentrated in one product line/region/customer cohort rather than uniform, and only pursue a deeper causal investigation once those simpler, cheaper explanations are ruled out.

**Why pair a North Star metric with guardrail metrics rather than optimizing one number?** A single metric can be improved in ways that damage something else not being measured — a guardrail metric catches the case where a genuine "win" on the primary metric comes with an unacceptable hidden cost, the same logic behind requiring guardrail metrics alongside a primary metric in any A/B test.

## Connections

- [[Framing Ambiguous Business Problems]] — the upstream skill this note's diagnostic process extends
- [[A-B Testing]] — guardrail metrics and controlled causal testing, applied to a specific proposed change
- [[Correlation vs Causation]] — required once simpler explanations for a metric change are ruled out
- [[Time Series Fundamentals]] — seasonality is a common, easy-to-miss confound in metric diagnosis
- [[Clustering]] — the technique underlying "segment the change" as a diagnostic step

## One-line Summary

> A good metric is hard to game and tied directly to the actual goal, ideally paired with guardrail metrics — and diagnosing why a metric moved should rule out measurement error and known external causes, then segment the change, before reaching for a deeper causal investigation.
