# Framing Ambiguous Business Problems

## What is it?

Data Scientist, Data Analyst, and Forward Deployed Engineer interviews frequently open with a deliberately vague prompt — "our engagement is down, what would you do?" or "design a system to detect fraud" — specifically to test whether a candidate can turn an ambiguous business question into a well-scoped technical problem *before* jumping to a modeling technique. This is a distinct skill from anything covered in the algorithm-focused modules of this vault, and it's frequently the actual bar being evaluated in these interviews, more than the specific technique eventually proposed.

---

## The Core Skill: Clarify Before Solving

Jumping straight to "I'd build a random forest for this" without first clarifying the actual question being asked is the single most common failure mode in this style of interview. Before proposing any technique:

**Clarify the actual goal.** "Engagement is down" — down for whom, over what time window, compared to what baseline? Is this a genuine business problem or a measurement artifact (a broken tracking pipeline, a change in how a metric is computed)?

**Ask what decision this analysis will inform.** A model or analysis exists to support a decision someone will actually make — if the output wouldn't change what anyone does, the analysis isn't worth the effort regardless of how technically sound it is. This reframes "build a model to predict X" into the more useful "what decision needs X to be predicted, and does knowing X actually change that decision."

**Identify constraints early.** Available data, latency requirements (a real-time fraud check has very different constraints than a monthly report), interpretability needs (a regulator-facing decision needs an explainable model — see [[Model Interpretability Index]] — far more than an internal recommendation does), and team/infrastructure maturity all shape which solution is actually appropriate, independent of which is most accurate in isolation.

---

## Translating to a Technical Problem

Once the actual question is clarified, translate it into one of the vault's established problem shapes: is this [[Classification]] or [[Regression]]? Is there a labeled target at all, or is this genuinely [[Unsupervised Learning]]? Is the right output a prediction, or is it [[Model Interpretability Index|an explanation]] of an already-observed pattern — a genuinely different deliverable that a predictive model alone doesn't provide?

**Worked example**: "engagement is down" could translate to *any* of: a time series anomaly-detection problem (is this drop statistically unusual — see [[Time Series Fundamentals]]), a causal-inference problem (did a specific recent change cause this — see [[Correlation vs Causation]], [[A-B Testing]]), a segmentation problem (which user segment specifically dropped, via [[Clustering]]), or purely a data-quality investigation (is the metric itself broken). Naming *which* of these the actual question is asking is the interview's real test — most candidates who struggle here skip straight to proposing a model without first establishing which of these fundamentally different problems is even being asked.

---

## Interview Questions

**"Our app's daily active users dropped 10% last week — walk me through your approach."** Before proposing any analysis: clarify what "dropped" means precisely (vs. last week, vs. the same week last year, accounting for known seasonality), rule out measurement issues first (a broken tracking event looks identical to a real drop from a dashboard's perspective), check for an identifiable single cause (an outage, a release, a marketing campaign ending) before assuming a gradual behavioral shift, and only then decide whether this needs a causal investigation, a segmentation breakdown, or is explained by a known one-time event.

**Why is "what decision will this analysis inform" a better starting question than "what model should I build"?** Because an analysis or model only has value insofar as it changes what someone actually does — starting from the decision forces scoping the actual deliverable (a prediction, an explanation, a monitoring alert) around what's genuinely needed, rather than defaulting to building a predictive model because that's the most familiar tool, when the real need might be a simpler diagnostic or a data-quality check.

## Connections

- [[Metric Selection and Diagnosing Change]] — the natural next step once the problem is framed
- [[Correlation vs Causation]], [[A-B Testing]] — often the right technical translation of "did X cause this"
- [[Model Interpretability Index]] — relevant when the actual deliverable is an explanation, not a prediction
- [[Model vs Rule Decisions]] — part of choosing the right *kind* of solution once the problem is scoped

## One-line Summary

> The interview skill being tested by an ambiguous business prompt is clarifying the actual question and the decision it needs to inform *before* proposing any technique — most candidates who struggle here jump straight to a model instead of first establishing what kind of problem is actually being asked.
