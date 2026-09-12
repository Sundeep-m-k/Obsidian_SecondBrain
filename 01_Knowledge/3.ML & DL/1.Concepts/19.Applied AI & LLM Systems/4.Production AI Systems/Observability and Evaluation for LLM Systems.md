# Observability and Evaluation for LLM Systems

## What is it?

**Observability** for an LLM system means being able to see, after the fact, exactly what happened on any given request — the full prompt sent, the model's raw response, any tool calls and their results, latency and token counts — so a failure or a quality regression can actually be diagnosed rather than guessed at. **Evaluation** is the ongoing, systematic measurement of whether the system is performing well, not just whether it's technically running.

---

## Why LLM Systems Need Different Observability Than Traditional Software

Traditional software observability (logs, metrics, traces) mostly tracks *whether* something ran and *how fast* — it doesn't capture *what the system actually said* or *why*. An LLM system's failures are frequently about output *quality* (a subtly wrong answer, an unfaithful RAG response, an agent that took a reasonable-looking but ultimately unhelpful action) rather than a crash or an error code — failures that traditional infrastructure monitoring is structurally blind to. LLM observability specifically needs to capture the full input/output content of every call, not just whether the call succeeded.

## What to Log

- The exact prompt sent (including any retrieved context, system instructions, few-shot examples) and the exact response received
- For agents: every tool call, its arguments, and its result, in order
- Token counts and latency, per call (feeds directly into [[Latency and Cost Optimization]])
- Model/version used, and any sampling parameters (temperature, top-p — see [[LLM Inference Fundamentals]])
- User feedback where available (thumbs up/down, corrections) — the highest-signal evaluation data available, since it reflects real usage rather than a synthetic test set

## Evaluation in Production: Online vs. Offline

This is the same [[Online vs Offline Evaluation]] distinction covered under classical MLOps, applied to LLM systems: **offline evaluation** runs a fixed evaluation set (see [[RAG Evaluation]] for the RAG-specific version) against a system before deploying a change, to catch regressions before real users see them. **Online evaluation** monitors live production behavior — user feedback signals, downstream task success rates, flagged/escalated conversations — since an offline evaluation set, however well constructed, can't anticipate every real query distribution a live system will actually encounter.

## Regression Detection

Because LLM outputs are non-deterministic and a small prompt or model change can shift behavior in ways that are hard to predict from first principles, changes to prompts, retrieval logic, or the underlying model **must be evaluated against a fixed regression test set before shipping** — the same discipline as unit tests in traditional software, applied to a system whose "correctness" is graded, not just pass/fail. Skipping this and shipping changes based on a few manually-spot-checked examples is a common, easy-to-fall-into anti-pattern that reliably produces silent quality regressions.

---

## Interview Questions

**Why is traditional application observability (uptime, latency, error rates) insufficient for an LLM system?** Most LLM failures are quality failures — a subtly wrong or unhelpful answer — not crashes or errors, and traditional monitoring only tracks whether something ran and how fast, not what the system actually produced or whether that output was good, so it's structurally blind to the most common real failure mode.

**What's the difference between offline and online evaluation for an LLM system, and why do you need both?** Offline evaluation runs a fixed test set before deploying a change to catch known regressions ahead of time; online evaluation monitors live production behavior (user feedback, task success rates) because no offline test set can fully anticipate the real distribution of queries a live system encounters — offline catches known risks before shipping, online catches what offline evaluation didn't anticipate.

**Why should every prompt or model change be run against a regression test set before shipping?** LLM outputs are non-deterministic and behavior can shift in ways that aren't obvious from reading the change itself — a prompt tweak intended to fix one case can silently degrade performance on others; a fixed regression set catches this the same way unit tests catch regressions in traditional software, rather than relying on a few manually spot-checked examples that may not represent the full range of real usage.

## Connections

- [[Latency and Cost Optimization]] — token/latency metrics are part of what gets logged and monitored
- [[RAG Evaluation]] — the RAG-specific version of building an evaluation set and running offline evaluation
- [[Online vs Offline Evaluation]] — the same distinction as classical MLOps, applied to LLM systems
- [[Hallucination Mitigation]] — a specific quality failure mode that evaluation needs to be designed to catch

## One-line Summary

> LLM observability must log full prompt/response content (not just success/failure), because most real failures are quality problems traditional monitoring is blind to — offline evaluation against a regression set catches known risks before shipping, online evaluation catches what offline testing couldn't anticipate, and every change needs both.
