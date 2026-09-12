# Multi-Agent Systems

## What is it?

A **multi-agent system** splits a complex task across several specialized [[AI Agents Fundamentals|agents]], each with a narrower role, coordinated by some structure (a fixed pipeline between them, a designated orchestrator agent, or peer-to-peer delegation) — rather than one general-purpose agent trying to handle every aspect of the task itself.

---

## Why Split Into Multiple Agents

**Specialization** — a narrowly-scoped agent (e.g. "research agent," "code-writing agent," "code-review agent") can have a more focused prompt, a smaller and more relevant tool set, and clearer success criteria than one agent trying to do everything — the same logic as why a human team specializes rather than every member doing every part of a project.

**Context management** — each sub-agent's context window only needs to hold information relevant to *its* piece of the task, rather than one agent accumulating the entire task's history, which helps against the [[LLM Inference Fundamentals|context window]] filling up with increasingly irrelevant accumulated history as a complex task progresses.

**Parallelism** — independent sub-tasks can run concurrently across separate agents rather than serially within one agent's loop, reducing overall latency for tasks that genuinely decompose into independent pieces.

## Common Coordination Patterns

**Orchestrator-worker** — one orchestrator agent decomposes the task and delegates sub-tasks to specialized worker agents, then synthesizes their results. Centralizes control and makes the overall flow easier to reason about; the orchestrator itself becomes a bottleneck/single point of failure if it makes a bad decomposition.

**Sequential pipeline** — agents run in a fixed order, each consuming the previous one's output (e.g. research agent → drafting agent → review agent). Predictable and debuggable, at the cost of no ability to loop back if a later agent discovers the earlier one's output was flawed, unless that's explicitly built in.

**Debate/critique** — multiple agents (or one agent playing multiple roles) generate and critique each other's outputs before a final answer is produced, aiming to catch errors a single pass wouldn't — at the cost of significantly more LLM calls (and therefore cost and latency) per task.

---

## The Real Cost: Compounding Failure and Compounding Latency/Cost

Every additional agent in a pipeline is another opportunity for something to go wrong, and errors can compound — an early agent's mistake propagates into every downstream agent's input, and by the time it surfaces, the actual root cause may be several agents upstream and hard to trace, an even sharper version of [[Agent Failure Modes and Guardrails|single-agent failure debugging]] difficulty. Cost and latency also compound directly — a 4-agent pipeline is, at minimum, roughly 4x the LLM calls of a single well-designed agent handling the same task, which is why multi-agent architectures should be justified by genuine task decomposition needs (true specialization or parallelism benefit), not reached for by default.

---

## Interview Questions

**Why split a task across multiple specialized agents instead of using one general-purpose agent with all the necessary tools?** Specialization gives each agent a narrower, more focused context and tool set with clearer success criteria, which tends to produce more reliable behavior per sub-task than one agent juggling everything — and independent sub-tasks can run in parallel across agents rather than serially within one agent's loop.

**What's the main risk specific to multi-agent systems that doesn't apply to a single agent?** Compounding failure — an early agent's mistake propagates as flawed input to every downstream agent, and by the time the error surfaces it can be difficult to trace back to which agent actually caused it, especially in a pipeline without an explicit way to loop back and correct an earlier stage.

**When would an orchestrator-worker pattern be preferable to a fixed sequential pipeline of agents?** When the actual decomposition of the task into sub-tasks can't be fully determined in advance and needs to be decided based on the specific input — the orchestrator dynamically delegates based on what the task actually requires, whereas a fixed pipeline assumes the same sequence of agents and handoffs every time.

## Connections

- [[AI Agents Fundamentals]] — the single-agent building block this composes
- [[Workflows vs Agents]] — the same autonomy-vs-predictability tradeoff, one level up in complexity
- [[Agent Failure Modes and Guardrails]] — failure modes here compound across agents rather than occurring once
- Latency and Cost Optimization (module 19, Production AI Systems) — multi-agent systems directly multiply per-task LLM call costs

## One-line Summary

> Multi-agent systems trade the complexity and compounding cost/latency/failure risk of coordinating several specialized agents for genuine benefits in specialization, context management, and parallelism — justified when a task truly decomposes that way, not reached for by default over a single well-scoped agent.
