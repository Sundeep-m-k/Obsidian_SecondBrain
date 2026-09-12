# Hallucination Mitigation

## What is it?

**Hallucination** is an LLM generating output that's fluent and confident-sounding but factually wrong, unsupported by any given source, or entirely fabricated (invented citations, invented function names, invented facts). It's not a bug in the traditional sense — it's a direct consequence of how these models work: an LLM generates the statistically most plausible next token given context, with no built-in mechanism distinguishing "this is something I actually know to be true" from "this is a plausible-sounding continuation."

---

## Why Hallucination Happens

The model has no explicit notion of "I don't know this" separate from generating fluent text — when asked something outside (or at the edge of) its training data, or with insufficient context to answer confidently, it still generates *some* continuation, and that continuation can be fluent and plausible while being false. This is fundamentally different from a traditional software bug with a specific reproducible cause; it's an inherent property of next-token-prediction models that must be managed, not a defect that can be fully eliminated.

---

## Mitigation Strategies

**Grounding via [[RAG Architecture|RAG]]** — provide the actual relevant source text in context so the model has something concrete to answer from, rather than relying entirely on parametric memory. Reduces but does not eliminate hallucination — a model can still ignore or misread provided context, especially when it conflicts with something the model "believes" strongly from pretraining.

**Asking the model to cite its sources** — explicitly require the model to reference which part of the provided context supports each claim, which both discourages ungrounded claims (the model has to point to something) and makes verification easier for a human or an automated checker.

**Lower temperature / more deterministic sampling** (see [[LLM Inference Fundamentals]]) — reduces the randomness contributing to occasionally producing a low-probability, unsupported continuation, though it doesn't address the underlying lack of a "don't know" mechanism.

**Self-consistency checks** — generate multiple independent responses to the same query and check for agreement; large disagreement across samples is a signal (not a guarantee) that the model is uncertain or hallucinating rather than confidently correct.

**Explicit "I don't know" instruction** — directly instruct the model to say it doesn't know rather than guess when the provided context is insufficient. Helps, but isn't reliable on its own — models can still confidently answer beyond what their context actually supports, especially under ambiguous instructions.

**Fact-checking / verification pass** — a separate step (another LLM call, a retrieval-based check, or a human review for high-stakes outputs) that specifically verifies claims in the generated output against a trusted source before it's shown to a user, rather than trusting the first-pass generation directly.

**Fine-tuning on factuality-focused data** — a heavier-weight approach ([[Transfer Learning and Fine-Tuning]]) that can improve a model's calibration around what it actually knows, though it requires more investment than the prompt/retrieval-level techniques above and doesn't eliminate the fundamental issue either.

---

## No Single Technique Is Sufficient

Every mitigation above reduces hallucination risk to some degree; none eliminates it. Production systems handling high-stakes factual content (medical, legal, financial) typically combine several of these (grounding + citation requirements + a verification pass + human review for the highest-stakes cases) rather than relying on any one technique, and treat hallucination as an ongoing risk to be monitored (see [[Observability and Evaluation for LLM Systems]]) rather than a problem that gets permanently solved once.

---

## Interview Questions

**Why can't hallucination be fully "fixed" the way a software bug can be?** It's a direct consequence of how LLMs generate text — predicting the most statistically plausible next token with no built-in mechanism for distinguishing known-true content from plausible-sounding fabrication — rather than a specific, reproducible defect with a targeted fix; every mitigation reduces the risk, none removes the underlying mechanism.

**Why does RAG reduce hallucination but not eliminate it?** Providing actual source text gives the model something concrete to ground its answer in instead of relying purely on parametric memory, which measurably helps — but the model can still ignore, misread, or contradict the provided context, especially when it conflicts with something it "believes" strongly from pretraining, so grounding reduces risk without removing it entirely.

**A production system needs to minimize hallucination for high-stakes factual answers — what would you actually build, beyond just using RAG?** Combine RAG-based grounding with an explicit citation requirement (forcing the model to point to supporting context for each claim), a separate verification/fact-checking pass before the answer reaches the user, and human review for the highest-stakes outputs — no single technique is sufficient on its own, so layering several addresses different aspects of the risk.

## Connections

- [[RAG Architecture]] — the primary grounding-based mitigation
- [[LLM Inference Fundamentals]] — sampling parameters (temperature) as a partial lever
- [[Observability and Evaluation for LLM Systems]] — how ongoing hallucination risk gets monitored rather than assumed solved
- [[Prompt Injection and Production Reliability]] — a related but distinct reliability risk (attacker-manipulated input vs. inherent model uncertainty)

## One-line Summary

> Hallucination is an inherent consequence of next-token prediction having no built-in "I don't know," not a fixable bug — grounding via RAG, citation requirements, verification passes, and human review for high-stakes cases each reduce the risk, and production systems layer several together rather than trusting any single technique.
