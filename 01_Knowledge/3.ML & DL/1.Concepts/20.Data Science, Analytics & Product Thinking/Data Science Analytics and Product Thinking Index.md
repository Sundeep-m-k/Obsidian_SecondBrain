---
tags: [category/ml-dl, topic/data-science, index, moc]
---

# Data Science, Analytics & Product Thinking — Index

> **Position in vault**: `3.ML & DL/1.Concepts/20.Data Science, Analytics & Product Thinking/`
> **Purpose**: The Data Analyst/Data Scientist-specific material this vault was thinnest on — statistical experimentation for business decisions, time series, and the case-study/product-sense reasoning these interviews specifically probe, distinct from algorithm knowledge.
> **Prerequisite**: [[Hypothesis Test]] (Statistics)

## Section Map

| Subfolder | Notes | Covers |
|---|---|---|
| 1. Statistical & Experimental Methods | *(link only — see below)* | A/B testing, correlation vs. causation — canonical home is `6.Statistics & Experimental Design`, not duplicated here |
| 2. Time Series | [[Time Series Fundamentals]] | Decomposition, stationarity, ARIMA, why standard CV doesn't apply |
| 3. Case Thinking & Product Sense | [[Framing Ambiguous Business Problems]], [[Metric Selection and Diagnosing Change]], [[Model vs Rule Decisions]] | Turning ambiguous prompts into scoped problems, choosing/diagnosing metrics, and when *not* to reach for ML |

## Why A/B Testing and Correlation vs. Causation Aren't Duplicated Here

Both live in `6.Statistics & Experimental Design/2.Experimental Design for ML/` as [[A-B Testing]] and [[Correlation vs Causation]] — they're general experimental-design and statistical-reasoning concepts, not ML-specific ones, and this vault's own convention ([[Databases & Data Systems Overview]], [[NLP Index]]) is to link into a concept's canonical home rather than re-explain it in every subject that uses it. This module's Case Thinking notes lean on both heavily.

## Why "Case Thinking & Product Sense" Is Distinct From Algorithm Knowledge

Everything in modules 1–19 of `3.ML & DL` answers "how does this technique work." This subfolder answers a different question entirely — "which technique, if any, actually addresses the business problem, and how do you even scope the problem before answering that" — the skill most directly tested by open-ended DA/DS/FDE case interviews, and one that no amount of algorithm depth substitutes for.

## Common Exam / Interview Questions

1. Walk through your process for an ambiguous "engagement is down, what would you do" prompt.
2. What makes a metric a poor choice even if it's easy to measure?
3. Why can't a random train/test split be used for time series data?
4. When would you recommend a simple rule over a more accurate model?
5. Ice cream sales correlate with drowning deaths — what does that tell you, and what doesn't it tell you?

## One-line Summary

> This module supplies the statistics-adjacent and product-reasoning material DA/DS/FDE interviews specifically probe — time series validation discipline and the case-thinking skill of scoping an ambiguous problem before reaching for a technique — while deliberately linking to, not duplicating, the general experimental-design concepts already built in Statistics.
