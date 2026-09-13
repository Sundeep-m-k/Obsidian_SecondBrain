# Agent Failure Modes and Guardrails

## What is it?

Because an [[AI Agents Fundamentals|agent]] decides its own actions at runtime rather than following a fixed script, it can fail in ways a plain LLM call or a fixed [[Workflows vs Agents|workflow]] simply cannot — **guardrails** are the explicit constraints and checks built around an agent to catch or prevent these failures before they cause real damage.

---

## Common Agent Failure Modes

**Infinite/excessive looping** — the agent keeps calling tools without making genuine progress toward the goal (e.g. repeatedly searching with slightly reworded queries, never satisfied with any result), consuming cost and latency without ever completing the task. Left unchecked, this can run indefinitely.

**Tool misuse** — calling a tool with malformed, wrong, or dangerous arguments (e.g. a database-write tool called with a query that deletes more than intended), or calling an entirely inappropriate tool for the situation.

**Goal drift** — over a long agent run, the agent's actions gradually stop serving the original task as intended, especially as accumulated context grows and earlier instructions become relatively less prominent in what the model is currently attending to.

**Compounding errors** — an early wrong action or misread tool result feeds forward as (now-incorrect) context for every subsequent decision, and the agent has no inherent way to recognize this unless something explicitly checks for it.

**Prompt injection** — a tool's *output* (a webpage the agent fetched, a document it retrieved) contains text specifically crafted to be interpreted by the model as new instructions rather than as data to reason about — potentially hijacking the agent into ignoring its original task or taking unintended, attacker-directed actions. This is a genuinely serious, actively-exploited failure mode wherever an agent processes untrusted external content, and it's covered in more depth in [[Prompt Injection and Production Reliability]].

---

## Guardrails

**Iteration limits** — a hard cap on how many loop iterations/tool calls an agent can make before being forced to stop and return whatever it has, directly bounding the cost and latency of the infinite-looping failure mode.

**Tool permission scoping** — give the agent access only to the tools and permissions actually required for its task (least privilege) — an agent that can only *read* a database can't be tricked into deleting data from it, regardless of what any prompt injection or misfired tool call tries to get it to do.

**Human-in-the-loop checkpoints** — require explicit human approval before executing high-stakes or irreversible actions (sending an email, making a purchase, deleting data), rather than letting the agent execute every action autonomously — the appropriate response scales with the actual cost of a wrong action, not a blanket policy either way.

**Output validation** — check a tool call's arguments (or a final answer) against expected constraints before executing/returning it — e.g. reject a database write whose scope looks unexpectedly broad, or a generated answer that fails a schema check — rather than trusting every model output unconditionally.

**Explicit stopping criteria** — give the agent a clear, checkable definition of "done" rather than leaving completion entirely to the model's own judgment, reducing both premature stopping and unnecessary continued looping.

---

## Interview Questions

**Why does an agent need an explicit iteration limit when a single LLM call doesn't?** A single call always produces exactly one response and terminates; an agent's loop has no inherent termination guarantee — without an explicit cap, a model that isn't converging on a solution (or is stuck reformulating the same failed approach) can loop indefinitely, consuming cost and latency without bound.

**What's the "least privilege" principle as applied to agent tool access, and why does it matter?** Give an agent only the tools and permissions its specific task actually requires, nothing broader — this way, even if a prompt injection or a bad decision causes the agent to attempt something harmful, the damage is bounded by what its restricted permissions actually allow it to do.

**When would you require a human-in-the-loop checkpoint rather than letting an agent act fully autonomously?** For actions that are high-stakes, costly to reverse, or hard to detect if wrong (sending a real email, making a financial transaction, deleting production data) — the appropriate amount of human oversight scales with the actual cost of the agent getting that specific action wrong, not a uniform policy applied to every action regardless of stakes.

## Connections

- [[AI Agent End-to-End Execution Trace]] — a worked failure trace (prompt injection via a tool result) showing these guardrails firing in context
- [[AI Agents Fundamentals]] — the loop structure that makes these failure modes possible in the first place
- [[Multi-Agent Systems]] — these same failure modes compound across multiple agents
- [[Prompt Injection and Production Reliability]] — the deepest treatment of the prompt-injection failure mode specifically
- [[Hallucination Mitigation]] — a related but distinct reliability concern (wrong content vs. wrong/unsafe action)

## One-line Summary

> Agents fail in ways fixed workflows can't — infinite loops, tool misuse, goal drift, compounding errors, prompt injection — and guardrails (iteration limits, least-privilege tool scoping, human-in-the-loop for high-stakes actions, output validation) exist specifically to bound the damage any of these can cause.
