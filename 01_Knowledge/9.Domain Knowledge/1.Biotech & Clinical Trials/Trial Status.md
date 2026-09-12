# Trial Status

## What is it?

**Trial status** records a trial's current state — Active/Recruiting, Completed, Terminated, Withdrawn, Suspended. It's a key input (with phase and elapsed time) to the [[Censoring]] three-way label: success, failure, or censored.

## Why Status Alone Isn't the Full Picture

Status describes one *trial's* state, not the whole *program's* fate. Formally, using [[Censoring]]'s notation, the program-level event indicator must be computed over **all** of a program $p$'s linked trials, not any single one:

$$\delta(p) = \mathbf{1}\left[\exists\, \text{trial } j \in \text{trials}(p) : \text{phase}(j) \geq k+1\right]$$

A program can have one Terminated trial ($j_1$) but a later, still-active higher-phase trial ($j_2$) — meaning $\delta(p)=1$ (or censored, pending $j_2$'s outcome) even though $j_1$ alone looks like failure. Labeling logic that checks only the most recent trial's status, rather than aggregating over $\text{trials}(p)$, will misclassify these cases.

## Terminal vs. Non-Terminal Statuses

| Status | Contributes to |
|---|---|
| Completed (with evidence of reaching next phase) | Success ($\delta=1$) |
| Terminated / Withdrawn (no later higher-phase evidence, anywhere in $\text{trials}(p)$) | Failure ("dead-end", $\delta=0$ resolved) |
| Active / Recruiting, under timeout | Censored (excluded from training as a hard 0 — see [[Censoring]]) |

## Why It Matters

Status is one of the two mechanisms (alongside the [[Censoring]] timeout window) that converts "no future success observed for *any* linked trial" into a confident failure label — conflating a single trial's terminal status with the whole program's fate is a distinct bug from the censored-vs-failed conflation [[Censoring]] describes, but has the same root cause: checking the wrong scope (one trial instead of the whole program).

## Common Mistakes

- Treating one trial's "Terminated" status as automatically meaning the whole program failed, without checking $\text{trials}(p)$ for a later, higher-phase trial
- Not distinguishing "terminated for business reasons" from "terminated for efficacy/safety failure" when both record simply as "Terminated" — a real nuance worth flagging even when the data can't always separate it

## Interview / discussion questions

- Formally, why must $\delta(p)$ be computed as an aggregate over $\text{trials}(p)$ rather than read off the single most recent trial?
- Why can a program have a "Terminated" trial in its history but still ultimately count as a success?

## Prerequisites

[[Clinical Trial Phases]]

## Related concepts

[[Censoring]], [[Drug Pipeline]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> Trial status is per-trial, but the success/failure/censored label must be an aggregate over *all* of a program's linked trials ($\delta(p) = \mathbf{1}[\exists j \in \text{trials}(p): \ldots]$) — checking only the latest trial's status is a distinct, common labeling bug.
