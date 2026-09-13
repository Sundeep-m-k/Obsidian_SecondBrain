# Deployment Strategies

## What is it?

**Deployment** moves a model from development into production, where it actually serves predictions. The **strategy** chosen determines how much risk a bad model can do before it's caught, how fast rollback is if something goes wrong, and how much infrastructure/cost the safety buys.

---

## Strategies

### Rolling Deployment

**Mechanism**: replace old model instances with new ones incrementally, a few servers/pods at a time, until every instance runs the new version — at any moment during the rollout, some fraction of traffic hits the old version and the rest hits the new one, simply as a function of which instances have been updated so far.

**Advantages**: no need to run two full parallel environments (unlike blue-green); infrastructure-efficient.

**Disadvantages**: rollback means rolling instances back the same incremental way, which is slower than blue-green's instant switch; during the rollout window, two model versions are live simultaneously and users can get inconsistent results depending on which instance handles their request — a real problem if predictions need to be consistent for the same user across requests.

**Rollback behavior**: incremental, in reverse — not instant.

**Cost**: low — no duplicate full-scale infrastructure required.

**When to use**: routine, low-risk updates where brief version inconsistency across instances is acceptable. **When not to use**: when instant rollback matters most, or when a user seeing different model versions on different requests would itself be a problem (e.g. a recommendation system where inconsistent behavior looks buggy to the user).

### Blue-Green Deployment

**Mechanism**: two full production environments — Blue (current) and Green (new, fully deployed but not yet receiving traffic). Traffic switches from Blue to Green all at once (instant cutover, no gradual ramp).

**Advantages**: rollback is just switching traffic back to Blue — as fast as the cutover itself, and Blue is left fully intact and ready the whole time. Simple to reason about.

**Disadvantages**: all-or-nothing — no gradual exposure to catch a problem before it hits 100% of traffic; requires running two complete production-scale environments simultaneously, which roughly doubles infrastructure cost for the duration of the deployment.

**Rollback behavior**: instant (a traffic-routing switch, not a redeploy).

**Cost**: high — duplicate full-scale environment, even if only briefly.

**When to use**: when instant, guaranteed rollback matters more than the cost of double infrastructure, and when the team is confident enough in pre-production validation that a sudden full cutover is an acceptable risk. **When not to use**: when a gradual, monitored rollout is preferred specifically *because* pre-production validation can't fully predict real production behavior — which is common for ML models, more so than for ordinary code (see below), and is why canary is often preferred for models specifically even where blue-green is standard for other software.

### Canary Deployment

**Mechanism**: route a small percentage of real traffic to the new model, monitor it closely, and *gradually* increase that percentage only as confidence grows — e.g. 1% → 5% → 20% → 50% → 100%, at each stage checking that nothing has gotten worse before proceeding to the next.

**Advantages**: gradual, real-traffic exposure catches problems while they only affect a small fraction of users; each stage is a checkpoint where a bad model can be caught and rolled back before wider exposure.

**Disadvantages**: slower to reach full rollout than blue-green; needs real traffic-splitting/routing infrastructure and the monitoring discipline to actually act on each stage's results rather than rubber-stamping progression.

**Rollback behavior**: route the affected traffic back to the previous version — fast, and only ever affects the (small) fraction that was on the canary at the time of rollback, which is the core safety property canary is chosen for.

**Cost**: moderate — no full duplicate environment required, but real engineering investment in traffic-splitting and per-stage monitoring.

**When to use**: for models where pre-production validation is a genuinely imperfect predictor of real production behavior — the default recommendation for most model rollouts specifically because of the differences between ML and ordinary software deployment covered below. **When not to use**: when the model change is trivial/low-risk enough that the overhead of staged rollout and monitoring isn't worth the added deployment time, or when instant full rollback (blue-green) is a harder requirement than gradual exposure.

### Shadow Deployment

**Mechanism**: the new model runs in production *in parallel* with the current one, receiving a copy of real traffic, but its predictions are never actually shown to users or acted on — only logged and compared against the current model's live predictions.

**Advantages**: genuinely zero production risk, since no real decision is ever based on the new model's output; surfaces latency, error rate, and prediction-divergence issues under real traffic patterns before any user is exposed at all.

**Disadvantages**: cannot measure real business impact (conversion, revenue, engagement) — a shadow model's predictions were never actually acted on, so there's no way to observe how users would have responded to them; doubles compute cost (both models process every request); may not fully reflect production latency under real load if the shadow path isn't given the exact same resource contention as the live path.

**Rollback behavior**: not applicable in the traditional sense — nothing was ever live to roll back.

**Cost**: high compute (running two models on all traffic), but zero user-facing risk.

**When shadow deployment is specifically useful**: as the step *before* canary, for a model change significant or unproven enough that even canary's small live-traffic exposure feels premature — shadow answers "does this behave sanely at all under real traffic and load" before any real user sees its output; it does not answer "do users respond better to it," which requires canary/A-B testing instead.

### A/B Deployment / Testing

**Mechanism**: route a defined split of traffic (commonly 50/50, though not required) to the current and new models simultaneously, and measure differences in business/product metrics directly — this is [[A-B Testing]]'s controlled-experiment methodology applied to comparing model versions specifically, and it's the only strategy in this list actually designed to answer "does the new model produce a better *business* outcome," not just "does it behave correctly."

**When to use**: when the deployment decision genuinely depends on a business metric a canary's technical monitoring wouldn't directly capture (e.g. does a new ranking model actually increase engagement, not just "does it run without errors"). See [[Online vs Offline Evaluation]] for how this complements offline validation.

---

## Worked Canary Example

Model v1 currently serves 100% of traffic. Deploying v2:

$$1\% \to 5\% \to 20\% \to 50\% \to 100\%$$

At each stage, track **latency** (p95), **error rate**, a **business metric** (e.g. conversion rate), and a **model-quality metric** (e.g. precision on labeled feedback where available in near-real-time). Example rollback thresholds (illustrative, not universal — the right numbers depend on the specific system and its normal variance):

$$\text{rollback if: } \Delta p95\text{ latency} > 20\% \quad \text{OR} \quad \text{error rate} > 2\% \quad \text{OR} \quad \text{critical model metric degrades beyond tolerance}$$

**Walkthrough with a failure at the 20% stage**: 1% and 5% stages both pass every threshold — latency and error rate are stable, the business metric is flat-to-positive. At **20%**, p95 latency has increased 28% over baseline — this crosses the $>20\%$ rollback threshold. **Rollback**: traffic is routed back to v1 for the affected 20% cohort immediately; the deployment does not proceed to 50%; the specific cause (a slow code path in v2, an infrastructure issue, a data-dependent slowdown) is investigated *before* any second attempt, rather than simply retrying the same rollout and hoping the latency spike doesn't recur.

The exact percentages and thresholds above are examples, not universal rules — a system with high normal latency variance needs a looser threshold or it will falsely trigger rollback on ordinary noise; a system with strict latency SLAs needs a tighter one.

---

## Why ML Deployment Differs From Ordinary Software Deployment

**The central difference**: ordinary software deployment can verify correctness with tests — if the code passes its tests and the service responds with expected status codes, it's healthy. **A model can be perfectly "healthy" by every infrastructure metric (low latency, zero errors, normal CPU/memory) while its predictions are simply wrong** — a healthy service returning bad predictions looks identical, from an infrastructure dashboard's perspective, to a healthy service returning good ones. This is why model deployment must monitor **all three** layers together, not infrastructure health alone:

$$\text{system health} \quad \text{AND} \quad \text{model behavior} \quad \text{AND} \quad \text{business outcomes}$$

(This is the same three-layer reasoning — plus a fourth, data-layer check — formalized in [[Model Monitoring in Production]]'s four-layer framework.)

## Compatibility: What Has to Match Across a Deployment

A model version doesn't deploy in isolation — it implicitly depends on a specific **feature pipeline** (the exact transformations that produced its training features), a specific **schema** (feature names, types, expected ranges), a specific **preprocessing** implementation (the exact scaling/encoding logic — see [[Preprocessing Pipelines]]), and specific **dependency versions** (a library version change can silently change numerical behavior). Deploying a new model version without verifying these all still match what the model was actually trained against is a specific, common cause of "the model was accurate in evaluation but wrong in production" — the model itself may be unchanged and correct, while the *pipeline feeding it* silently drifted out of sync.

---

## Interview Questions

**Blue-green vs. canary — when would you choose each?** Blue-green gives instant, guaranteed rollback at the cost of double infrastructure and no gradual exposure — appropriate when a sudden full cutover is an acceptable risk. Canary trades slower rollout for staged, real-traffic exposure that catches problems while they only affect a small fraction of users — the more common default for ML models specifically, since pre-production validation is a weaker guarantee of production behavior for a model than for ordinary code.

**Canary vs. shadow — what's the actual difference in what each can tell you?** Shadow never affects real users or real decisions, so it can validate technical behavior (does it run, is latency acceptable, do predictions look sane) under real traffic without any risk — but it cannot measure real business impact, since nothing was ever acted on. Canary does affect real (a small fraction of) users and real decisions, so it's the one that can actually answer whether the new model performs better in the ways that matter, at the cost of some real exposure.

**When is shadow deployment specifically worth the doubled compute cost?** When a model change is significant or unproven enough that even canary's small live-traffic exposure feels premature — shadow is the step that answers "does this behave sanely under real traffic and load at all" before committing to any real user exposure, however small.

**How does rollback actually work for each strategy?** Rolling: incremental, reversing the same gradual instance-by-instance process (slow). Blue-green: an instant traffic-routing switch back to the untouched previous environment. Canary: route the currently-canaried traffic fraction back to the previous version, which by construction only ever affects a small fraction of users. Shadow: not applicable — nothing was ever live.

**What metrics determine whether a canary should be promoted to the next stage?** At minimum, all of: infrastructure health (latency, error rate), model-behavior metrics (prediction distribution, any available near-real-time quality signal), and business metrics where measurable in the time window — promoting only on infrastructure health while ignoring model behavior is exactly the failure mode the "system health AND model behavior AND business outcomes" framing above is meant to prevent.

**How would you deploy a genuinely risky new model safely?** Shadow first (validate technical behavior with zero user exposure), then canary with a conservative initial percentage and explicit, pre-agreed rollback thresholds (not decided ad hoc mid-rollout), verifying feature-pipeline/schema/dependency compatibility before any of it, and only reaching for an A/B test once the model has cleared the technical bar and the remaining question is genuinely about business impact rather than correctness.

## Connections

- [[Model Monitoring in Production]] — the four-layer monitoring framework this note's "system + model + business" reasoning is one instance of
- [[Model Versioning and Reproducibility]] — a deployable model version depends on exact code/data/config being tracked, which is what makes safe rollback possible at all
- [[A-B Testing]], [[Online vs Offline Evaluation]] — the methodology behind the A/B deployment strategy specifically
- [[Preprocessing Pipelines]], [[Feature Stores]] — the feature-pipeline compatibility concern above, and the infrastructure that specifically prevents it from silently breaking
- [[Data Drift and Concept Drift]] — why a model that passed deployment validation can still degrade later, a separate concern from the deployment strategy itself

## One-line Summary

> Rolling/blue-green/canary/shadow/A-B each trade rollout speed, infrastructure cost, and risk exposure differently — canary is the common ML default specifically because a model can be perfectly healthy by every infrastructure metric while its predictions are simply wrong, which is why deployment must monitor system health, model behavior, and business outcomes together, not infrastructure health alone.
