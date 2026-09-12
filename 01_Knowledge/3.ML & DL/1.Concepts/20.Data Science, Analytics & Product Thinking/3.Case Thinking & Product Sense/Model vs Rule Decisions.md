# Model vs Rule Decisions

## What is it?

Not every problem that *could* be solved with a machine learning model *should* be — a large part of practical judgment in DS/DA/FDE roles is recognizing when a simple rule (a hardcoded threshold, an `if` statement, a lookup table) achieves nearly the same outcome as a model, at a fraction of the complexity, maintenance burden, and interpretability cost.

---

## When a Rule Beats a Model

**The pattern is simple and well-understood.** If domain knowledge already specifies the decision boundary clearly ("flag any transaction over $10,000 for manual review"), a model adds complexity without adding predictive value — it would just be learning to approximate a threshold a person could specify directly.

**Data is scarce.** A model needs enough examples to learn a reliable pattern; a rule based on domain expertise works from day one with zero training data, and can be deployed before enough labeled data exists to train anything.

**Interpretability/auditability is a hard requirement.** A rule's logic is fully transparent and directly explainable to a regulator, a customer, or an internal stakeholder — no [[Model Interpretability Index|SHAP or LIME]] needed to explain a decision that's already a readable if-statement.

**The cost of a wrong decision is asymmetric and severe**, and a rule provides a guaranteed hard boundary a probabilistic model cannot promise (a model can always be wrong with some probability; a rule like "never allow X above threshold Y" is a hard constraint, not a tendency).

## When a Model Beats a Rule

**The pattern is genuinely complex or high-dimensional** — too many interacting factors for a person to hand-specify a rule that captures them well, which is exactly the situation [[Feature Engineering]] and the algorithms throughout modules 1–18 of this vault are built for.

**The pattern shifts over time** in a way a static rule can't track, but a model can be retrained to follow (see [[Data Drift and Concept Drift]], [[Retraining and Model Maintenance]]).

**Enough labeled data and engineering maturity exist** to build, validate, and — critically — *monitor* a model in production ([[Model Monitoring in Production]]), since an unmonitored model degrading silently is often worse than a rule that, while less accurate, degrades predictably and visibly.

## The Practical Middle Ground: Rules as a Baseline and a Safety Net

A simple rule is frequently the right **first version** of a system — establishing a [[Baseline Estimator|baseline]] that any subsequent model must actually beat to justify its added complexity, exactly the evaluation discipline in [[Evaluation Workflow and Baselines]]. Rules and models also commonly coexist in production: a rule-based hard constraint (a guardrail — "never approve above $X regardless of model output") wrapped around a model's prediction, so the model handles the nuanced majority of cases while the rule prevents a worst-case failure the model might otherwise be fooled into.

---

## Communicating This Tradeoff to Stakeholders

Recommending a simpler rule over a more sophisticated model to a stakeholder expecting "the AI solution" requires explaining the tradeoff in terms of *their* priorities, not model architecture: maintenance cost, time-to-deploy, interpretability for their own compliance/audit needs, and the actual, measured performance gap (if any) a model would provide over the rule — a stakeholder is much more receptive to "the rule gets 95% of the model's accuracy at a fraction of the ongoing cost and full transparency" than to an unqualified claim that a model is unnecessary. Conversely, recommending a model when a rule would suffice, purely because it's the more sophisticated-sounding option, is a real anti-pattern this judgment call is meant to guard against — the right call is whichever approach actually serves the business need, not whichever is more technically interesting to build.

---

## Interview Questions

**When would you recommend a simple rule over a machine learning model, even if the model would be slightly more accurate?** When the pattern is simple enough to specify directly from domain knowledge, data is too scarce to train a reliable model, interpretability/auditability is a hard requirement, or the maintenance and monitoring cost of a model isn't justified by a marginal accuracy gain — the right comparison is total cost and risk, not accuracy alone.

**How would you explain to a non-technical stakeholder why you're recommending a rule-based approach instead of "the AI solution" they expected?** Frame it around their actual priorities — faster time to deployment, full transparency for compliance needs, no ongoing retraining/monitoring burden — and back it with the measured (or reasonably estimated) performance gap a model would provide, so the recommendation is grounded in an honest tradeoff rather than "it isn't fancy enough."

**Why might a production system use both a rule and a model together, rather than choosing one?** A rule can serve as a hard safety guardrail around a model's output (e.g. "never approve above $X regardless of what the model predicts"), letting the model handle the nuanced majority of cases while the rule prevents a worst-case failure the model could otherwise be fooled or wrong about — combining the model's flexibility with the rule's guaranteed boundary.

## Connections

- [[Baseline Estimator]], [[Evaluation Workflow and Baselines]] — a rule is frequently the correct baseline a model must actually beat
- [[Model Interpretability Index]] — the interpretability gap a rule doesn't have and a model does
- [[Data Drift and Concept Drift]], [[Model Monitoring in Production]] — the ongoing cost a model carries that a static rule doesn't
- [[Framing Ambiguous Business Problems]] — this decision is part of translating a scoped problem into an actual solution

## One-line Summary

> A rule wins when the pattern is simple, data is scarce, interpretability is required, or a hard guaranteed boundary matters more than marginal accuracy; a model wins when the pattern is genuinely complex, shifts over time, and the team has the data and maturity to monitor it — and the two frequently coexist, with a rule acting as a guardrail around a model's output.
