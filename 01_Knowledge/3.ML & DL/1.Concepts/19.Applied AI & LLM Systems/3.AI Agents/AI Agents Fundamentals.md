# AI Agents Fundamentals

## What is it?

An **AI agent** is an LLM-driven system that operates in a loop — observe the current state, decide on an action (respond directly, or call a tool), execute that action, observe the result, and repeat — until the task is complete, rather than producing one response to one prompt and stopping. The core mechanism is exactly [[LLM Inference Fundamentals|tool/function calling]], run repeatedly rather than once.

---

## The Agent Loop

```
1. Observe: current conversation history + any tool results so far
2. Think/Decide: the LLM decides — respond directly, or call a tool?
3. Act: if a tool call, the application executes it (search, code execution,
   API call, database query...) and returns the result
4. Observe: the tool's result is added to the context
5. Repeat from step 2, until the LLM decides the task is done and
   responds directly instead of calling another tool
```

This loop is sometimes called **ReAct** (Reason + Act) — interleaving explicit reasoning about what to do next with actually taking actions and observing their results, rather than trying to plan the entire task in one shot before taking any action.

## Why an Agent Loop Instead of One Prompt

A single LLM call is bounded by what the model already knows and can reason about in one pass — it has no way to look something up, run code to check its own arithmetic, or react to information it only learns partway through a task (e.g. a search result that changes what should happen next). The agent loop lets the model gather information, take an action, see what actually happened, and adjust — closer to how a person would actually work through a multi-step task with uncertain intermediate outcomes, rather than trying to solve everything from prior knowledge in one shot.

---

## Interview Questions

**What's the fundamental difference between a single LLM call and an agent?** A single call produces one response from the context it's given and stops; an agent runs in a loop, taking actions (tool calls), observing their real results, and using those results to decide its next action — it can react to information it didn't have when the task started.

**What is ReAct, and why interleave reasoning with acting rather than planning everything upfront?** ReAct interleaves explicit reasoning steps with actions and their observed results; planning an entire multi-step task upfront assumes the outcome of every intermediate step is predictable, which often isn't true (a search might return unexpected results, an API call might fail) — interleaving lets the agent adapt its plan based on what actually happens at each step.

**Why can an agent solve tasks a single LLM call can't, even using the exact same underlying model?** Because the agent can take real actions — search, compute, query a database — and incorporate their actual results into its reasoning, rather than being limited to whatever the model already knows or can derive purely from its own parametric memory in one pass.

## Connections

- [[LLM Inference Fundamentals]] — tool/function calling is the mechanism every step of the agent loop is built on
- [[Planning and Memory]] — the two capabilities that make longer, more complex agent tasks tractable
- [[Workflows vs Agents]] — when this loop is actually the right architecture, vs. simpler alternatives
- [[Agent Failure Modes and Guardrails]] — what goes wrong when this loop runs unchecked

## One-line Summary

> An agent repeatedly decides whether to act (call a tool) or respond, observing each action's real result before deciding the next step — the ReAct pattern of interleaving reasoning and acting is what lets it handle tasks a single, one-shot LLM call fundamentally cannot.
