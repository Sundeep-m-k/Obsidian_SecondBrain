# Workflows vs Agents

## What is it?

A **workflow** is a fixed, predetermined sequence of LLM calls and/or tool calls — the control flow (what happens after what) is written by the developer in code. An **agent** lets the LLM itself decide the control flow at runtime — which tool to call, in what order, how many times, whether to stop — via the [[AI Agents Fundamentals|agent loop]]. This is the single most consequential architectural decision in building an LLM-powered application, and it's a genuinely important distinction for Forward Deployed Engineer and Applied AI Engineer interviews specifically, since it's a decision made on essentially every real project.

---

## Why Default to a Workflow, Not an Agent

A full autonomous agent is more flexible but also less predictable, harder to debug, slower (multiple LLM round-trips instead of a fixed number), and more expensive (every loop iteration is another LLM call) than a workflow that already encodes the known-correct sequence of steps for a well-understood task. **If the task's structure is known in advance — the steps and their order don't actually need to be decided at runtime — a workflow gets the same result more reliably, more cheaply, and more debuggably than delegating that decision to the model.** Reach for full agent autonomy specifically when the task's required steps genuinely can't be known ahead of time — an open-ended research task, a customer support query that could need any of dozens of different tools depending on what the user actually needs, debugging where the right diagnostic step depends on what previous steps revealed.

## A Spectrum, Not a Binary

**Fixed pipeline** — a hardcoded sequence (e.g. RAG's retrieve → rerank → generate): no runtime decision-making at all, maximally predictable and cheap.

**Routing** — an LLM call classifies the input and directs it to one of several fixed downstream workflows (e.g. "is this a billing question or a technical question," each handled by a different fixed pipeline): a small amount of runtime decision, still highly predictable per branch.

**Tool-augmented workflow** — a fixed sequence of steps, but one or more steps let the LLM choose *which* of a constrained set of tools to call, without open-ended looping: more flexibility than a fixed pipeline, still bounded.

**Full autonomous agent** — the LLM decides the entire sequence of actions at runtime, including when to stop: maximum flexibility, minimum predictability and cost control.

Most production LLM systems live somewhere in the middle of this spectrum, not at either extreme — a well-designed system uses exactly as much runtime autonomy as the task genuinely requires and no more.

---

## Interview Questions

**When would you choose a fixed workflow over a full autonomous agent for a new LLM feature?** When the required steps and their order are known in advance and don't depend on information only available at runtime — a workflow achieves the same outcome more predictably, cheaply, and debuggably, since there's no risk of the model choosing a wrong or unnecessary tool call, or looping longer than needed.

**Why is "workflow vs. agent" often a spectrum rather than a binary choice in practice?** Real systems commonly combine fixed pipeline stages with a bounded amount of LLM-driven decision-making (e.g. routing to one of several fixed downstream flows, or choosing among a small constrained tool set at one specific step) — using full runtime autonomy only for the specific parts of a task that genuinely can't be predetermined, while keeping everything else predictable and cheap.

**What are the concrete costs of choosing an agent when a workflow would have sufficed?** More LLM calls per task (each loop iteration is a round-trip, adding latency and cost), less predictable behavior (the model might choose a different sequence of tools on a similar-looking input), and harder debugging (the actual sequence of steps taken varies at runtime, rather than being fixed and traceable in code).

## Connections

- [[AI Agent End-to-End Execution Trace]] — includes this decision as a row in a worked planning-approach table
- [[AI Agents Fundamentals]] — the full-autonomy end of the spectrum
- [[RAG Architecture]] — a canonical example of a fixed workflow, not an agent
- Latency and Cost Optimization (module 19, Production AI Systems) — a direct practical consequence of this choice
- [[Agent Failure Modes and Guardrails]] — the risks that increase as more autonomy is granted

## One-line Summary

> A workflow encodes the control flow in code; an agent lets the LLM decide it at runtime — default to a workflow whenever the task's steps are knowable in advance, and reach for agent autonomy only for the specific parts of a task that genuinely can't be predetermined, since autonomy trades predictability, cost, and debuggability for flexibility.
