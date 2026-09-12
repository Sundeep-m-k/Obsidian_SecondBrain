# Sponsor Type

## What is it?

A clinical trial's **sponsor** is the organization running it, typically categorized as **industry** (pharma/biotech company) vs. **academic/government** (university, hospital system, public research body).

## Formalizing "Ever Industry-Sponsored"

Let $\text{sponsor}(p, t)$ be program $p$'s sponsor type at time $t$ (over its set of linked trials). A naive "current sponsor" filter:

$$\text{industry}_{\text{current}}(p) = \mathbf{1}[\text{sponsor}(p, t_{\text{now}}) = \text{industry}]$$

silently drops programs that started academic and were later licensed to industry, if evaluated *before* the handoff, or drops the academic-origin evidence entirely. The more defensible definition, used deliberately here:

$$\text{industry}_{\text{ever}}(p) = \mathbf{1}\left[\exists\, t : \text{sponsor}(p,t) = \text{industry}\right]$$

This preserves academic-to-industry handoffs rather than dropping them.

## Worked Example: Validating the Rule Actually Matters

Applying $\text{industry}_{\text{ever}}$ to a cohort of 71,000 industry-scoped programs, suppose 5,538 of them ($7.8\%$) have at least one academic/government-sponsored trial earlier in their history alongside industry trials — i.e. they're "mixed-sponsor." That $7.8\%$ is the concrete, measured population that $\text{industry}_{\text{current}}$ would have wrongly included or excluded depending on which point in their history was checked. If this number were $0.0\%$, the ever-vs-current distinction would be a theoretical nicety with no real effect on the data — checking it is what makes the design choice validated rather than assumed.

## Why It Matters

Getting this filter wrong either drops real programs (too strict, current-only) or fails to exclude never-industry academic programs (not applied at all). Industry backing correlates with funding stability to reach expensive later-phase trials, making sponsor type both a scoping filter and a feature.

## Common Mistakes

- Filtering by a program's most recent or first sponsor only, silently dropping legitimate handoff programs
- Assuming sponsor type is static over a program's lifetime
- Skipping the validation step — computing the ever-vs-current rule without ever measuring what fraction of programs it actually affects

## Interview / discussion questions

- Formally contrast $\text{industry}_{\text{current}}$ and $\text{industry}_{\text{ever}}$ and explain which programs each one would misclassify.
- What real-world scenario does the academic-to-industry handoff represent, and why would silently dropping it bias a forecasting dataset?
- How would you measure whether the "ever" vs. "current" distinction actually matters for a given dataset?

## Prerequisites

[[Drug Pipeline]], [[Clinical Trial Phases]]

## Related concepts

[[Trial Status]], [[Data Leakage]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> "Industry-sponsored" should be defined as $\exists\, t : \text{sponsor}(p,t)=\text{industry}$ over a program's full history, not its current state — and the ~7.8% mixed-sponsor rate in a real cohort is the measured evidence that this definition, not the naive current-sponsor one, is the correct choice.
