# Model Monitoring in Production

## What is it?

**Monitoring** tracks a deployed model's health over time so degradation is caught and acted on — retraining, rollback, or investigation — before it causes real damage. A model that looked great at evaluation time is not guaranteed to stay that way, and the specific reason it stops working is rarely obvious from any single metric alone, which is why monitoring is organized into layers rather than one dashboard.

---

## The Four-Layer Framework

**1. Infrastructure** — latency, throughput, error rate, CPU/GPU utilization, memory, queue depth. Standard software observability, necessary but not sufficient: a model can be infrastructure-healthy and still be producing wrong predictions (see [[Deployment Strategies]]'s treatment of exactly this gap).

**2. Data** — schema violations (a field arrives with the wrong type or an unexpected value), missingness (a feature that's normally populated starts arriving null), range violations (a value outside its historically observed bounds), distribution changes and feature drift (the incoming data's statistical shape shifts away from what the model was trained on).

**3. Model** — the prediction distribution itself (is the model suddenly predicting one class far more or less often than usual), confidence/calibration (see [[Calibration and Probability Evaluation]]), accuracy/F1/AUC once labels actually arrive, and [[Data Drift and Concept Drift|concept drift]] (the relationship between inputs and the true outcome itself has changed, not just the inputs).

**4. Business** — conversion, fraud actually caught, user engagement, revenue, downstream operational impact. The layer that ultimately matters, and the one most likely to lag the other three in visibility.

**Why model metrics alone can be misleading**: a model can maintain stable accuracy on a metric that's itself becoming less relevant (e.g. accuracy on a majority class that's growing as a fraction of traffic, masking real degradation on the minority class that matters more — see [[Class Imbalance Evaluation]]), or infrastructure and data can look fine while the actual relationship the model was trained on has shifted in a way accuracy hasn't caught up to reflecting yet, precisely because of delayed ground truth (below). No single layer is sufficient on its own — the four together are what actually localize a real problem.

---

## Worked Incident: A Fraud Model's Recall Gradually Drops

**Alert**: recall (measured on the delayed-but-now-available ground truth for transactions from ~2 weeks ago) has dropped from 91% to 84% over the past month — a real, non-noise decline (see Alert Design below for how "real" is judged).

**Inspect infrastructure**: latency, error rate, and throughput are all normal — this isn't an infrastructure problem, ruling out layer 1 quickly.

**Inspect input distributions**: a feature-level distribution check (layer 2) shows `transaction_amount`'s distribution is essentially unchanged, but `merchant_category`'s distribution has shifted — a new category value, `"digital_goods"`, has grown from under 1% to 8% of traffic over the same window.

**Identify an upstream feature change**: tracing `merchant_category`'s pipeline reveals a recent change in how the upstream payments system categorizes transactions — a new category was introduced that didn't exist when the model was trained, so the model has effectively never seen this category's real fraud patterns.

**Compare model versions**: re-running the *current* model against a frozen historical evaluation set shows recall unchanged there — the model itself hasn't degraded on the data it knows; it's encountering *new* data it was never trained to handle. This step is what distinguishes "the model got worse" from "the world changed and the model didn't."

**Determine whether drift occurred, and which kind**: this is **data drift** — specifically the appearance of a new, previously-unseen category in an existing feature — not concept drift, since there's no evidence the fraud-vs-legitimate relationship *for categories the model has seen before* has changed; the model is simply blind to a category it never learned from.

**Mitigation**: in the short term, route `"digital_goods"` transactions to a stricter manual-review rule (a [[Model vs Rule Decisions|rule-based]] safety net) rather than trusting the model's predictions on data it has no real basis for; in parallel, begin collecting labeled examples of this new category for retraining.

**Rollback/retraining**: since this isn't a regression caused by the current model version itself (rollback to a prior version wouldn't help — the prior version has the identical blind spot), the correct fix is **retraining** on data that includes the new category, not a version rollback.

**Postmortem**: the actual process gap — an upstream schema/category change shipped without notifying the ML team or triggering any automated check — is the thing to fix structurally (e.g. an alert specifically on new/unseen categorical values appearing in a monitored feature), not just this one incident.

---

## Delayed Ground Truth

One of the hardest production-ML monitoring problems: for many real systems, **true labels arrive days or weeks after the prediction was made** (fraud confirmed only after a chargeback dispute resolves; a loan's default status known only months later) — a live accuracy/recall number for *today's* predictions is often simply unavailable today.

**Proxy metrics and delayed evaluation, used together**: sample a subset of predictions for expedited manual labeling rather than waiting for the full natural delay on every case; use available proxy signals that correlate with the true outcome but arrive sooner (clicks, chargebacks disputed vs. not, short-term conversion) while being explicit that a proxy is not the same as ground truth and can itself drift independently; monitor the **input distribution** (layer 2) and **prediction distribution** (layer 3) in the meantime, since both are available immediately and can surface a problem before delayed labels confirm it, as in the worked incident above where the feature-distribution shift was visible well before the recall drop was fully confirmed; and recompute the *real* metric once labels do arrive, treating the proxy-based read as a leading indicator, not a final answer.

---

## Drift Taxonomy

Four related but distinct terms, worth being precise about in an interview:

- **Data drift** — the distribution of input features changes over time, regardless of whether the true relationship between inputs and outcome has changed at all.
- **Covariate shift** — the specific, formal case of data drift where $P(X)$ (the input distribution) changes but $P(Y \mid X)$ (the true relationship) stays the same — the worked incident above is a covariate shift: a new category appeared, but fraud-vs-legitimate behavior *given* a category hasn't changed for the categories the model already knew.
- **Label shift** — the distribution of the *target* $P(Y)$ changes (e.g. the true fraud rate genuinely rises or falls) while $P(X \mid Y)$ (what fraudulent/legitimate transactions look like, given their true label) stays the same — a different failure mode from covariate shift, since here it's the base rate that moved, not the input features' own distribution independent of outcome.
- **Concept drift** — $P(Y \mid X)$ itself changes — the same inputs now genuinely predict a different outcome than they used to (e.g. a fraud pattern that used to be a reliable signal stops being predictive because fraudsters adapted their behavior).

See [[Data Drift and Concept Drift]] for the full formal treatment of data drift/covariate shift and concept drift specifically, including detection methods and handling strategies — not repeated here. Label shift is the one of the four not covered there: it matters distinctly because a model can remain well-calibrated *conditional on the true label* while still making systematically worse predictions simply because the label's base rate moved and the model's decision threshold was tuned for the old rate (see [[Thresholding]]).

---

## Alert Design

**Why alerting on every statistical difference creates noise**: with enough monitored metrics and enough data, some metric will cross *some* threshold purely by chance on any given day — alerting on every statistically detectable difference, however small, produces constant false alarms that teams learn to ignore, which is the same [[Multiple Comparisons Problem|multiple-comparisons]] pattern that shows up everywhere many things are being checked at once, just applied to ongoing monitoring rather than a one-time experiment.

**Statistical significance vs. practical significance**: a metric can move by a statistically detectable amount (e.g. accuracy dropping from 91.0% to 90.7%, confirmed via a proper test to not be noise) that is nonetheless too small to matter for any actual decision — alerting should be built around **practical** significance (does this change actually matter to the business or the user), not merely statistical detectability.

**A more useful operational framing than a bare threshold**: combine **threshold** (how far a metric has moved), **duration** (has it stayed moved, or was it a single noisy data point — the same "don't act on one bad measurement" discipline as [[Statistical Significance Testing for Model Comparison]]), and **business impact** (does this specific metric, at this specific magnitude, actually translate into a consequence worth interrupting someone for) — alerting on all three together, rather than a threshold crossing alone, is what keeps alerts meaningful rather than noisy.

---

## Interview Questions

**What are the four layers of production ML monitoring, and why do you need all of them?** Infrastructure, data, model, and business — because a problem can be invisible at any one layer while fully present at another (infrastructure-healthy but wrong predictions; stable accuracy but eroding business impact), so localizing a real incident requires checking all four, not assuming the most convenient one tells the whole story.

**Ground truth labels arrive two weeks late — how do you monitor daily?** Use proxy metrics and delayed evaluation together: monitor input/prediction distributions (available immediately) as leading indicators, use faster-arriving correlated signals as proxies while being explicit they aren't ground truth, sample a subset for expedited labeling, and recompute the real metric once full labels arrive to confirm or correct what the proxies suggested.

**What's the difference between covariate shift, label shift, and concept drift?** Covariate shift: the input distribution changes, but inputs still mean the same thing (a new category of legitimate traffic, unrelated to true outcome rates). Label shift: the target's base rate changes, but what inputs look like for a given true label doesn't. Concept drift: the actual relationship between inputs and the true outcome changes — the same inputs now predict something different than they used to.

**Why shouldn't you alert on every statistically significant metric change?** With enough metrics monitored continuously, some will cross a significance threshold by chance alone — alerting on every one produces alert fatigue; a more useful framing combines threshold, sustained duration, and actual business impact, since statistical significance alone doesn't imply the change is large enough to matter.

**In the fraud-recall worked incident, why was retraining the right fix rather than rolling back to a previous model version?** The current model's performance on data it had actually seen before was unchanged — the problem was a new category it had never been trained on at all, a blind spot every prior version would share equally; rolling back wouldn't fix a gap that predates the current version.

## Connections

- [[Deployment Strategies]] — the "system health AND model behavior AND business outcomes" framing this note's four layers formalize
- [[Data Drift and Concept Drift]] — the full treatment of covariate shift and concept drift detection/handling this note's Drift Taxonomy section points to rather than repeats
- [[Retraining and Model Maintenance]] — what happens once monitoring identifies a real, retraining-worthy problem
- [[Multiple Comparisons Problem]], [[Statistical Significance Testing for Model Comparison]] — the statistical reasoning behind avoiding alert noise
- [[Class Imbalance Evaluation]] — why an aggregate metric like accuracy can mask real degradation on a minority class
- [[Thresholding]] — why label shift specifically can silently break a fixed decision threshold

## One-line Summary

> Monitor infrastructure, data, model, and business layers together, since a real problem is rarely visible at just one; use proxy metrics and leading indicators to bridge delayed ground truth; distinguish covariate shift, label shift, and concept drift precisely rather than lumping them as "drift"; and alert on threshold-plus-duration-plus-business-impact together, not statistical significance alone, or the alerts stop being trusted.
