# Planning and Memory

## What is it?

**Planning** is how an agent decomposes a complex task into smaller steps before or during execution. **Memory** is how an agent retains and uses information across steps of a task — and, for longer-lived agents, across separate sessions. Both are what let an [[AI Agents Fundamentals|agent loop]] handle tasks too complex to solve in a single reasoning step.

---

## Planning Approaches

**Single-step reasoning (ReAct-style)** — decide the very next action given the current state, without committing to a full multi-step plan upfront (as in [[AI Agents Fundamentals]]). Adapts naturally to unexpected results, at the cost of potentially wandering inefficiently if the task genuinely benefits from upfront structure.

**Explicit upfront decomposition** — ask the model to first produce a full task breakdown ("to answer this, I need to: 1... 2... 3...") before executing any step. Gives more structure and a checkable plan, but risks committing to a plan that doesn't survive contact with the first step's actual result — a plan step assuming information that turns out to be wrong requires either rigid execution of a now-flawed plan or explicit re-planning.

**Re-planning** — after each step, check whether the original plan still makes sense given what was just learned, and revise it if not. The practical middle ground: enough upfront structure to avoid pure step-by-step wandering, enough adaptability to not be locked into an early plan invalidated by new information.

---

## Types of Memory

**Short-term (working) memory** — the current conversation/task context, held in the LLM's [[LLM Inference Fundamentals|context window]] for the duration of one session. Bounded by context-window size — a very long task can exhaust it, requiring summarization or selective retention of only the most relevant history.

**Long-term memory** — information persisted *across* sessions (user preferences learned over many interactions, facts established in a previous conversation) — implemented as an external store (often a [[Vector Search and Databases|vector database]]) that the agent explicitly retrieves from, structurally identical to [[RAG Architecture|RAG]]'s retrieval step, just applied to the agent's own history/knowledge rather than a static document corpus.

**Episodic vs. semantic memory** (a useful distinction borrowed from cognitive science) — episodic memory retains specific past events/interactions ("the user asked about X last Tuesday"); semantic memory retains generalized facts extracted from those events ("the user prefers concise answers"). Production agent memory systems often maintain both, since a specific episode and a generalized preference serve different purposes.

---

## Why Context-Window Limits Force These Design Decisions

Both planning and memory exist as engineering problems specifically because the [[LLM Inference Fundamentals|context window]] is finite — an agent can't simply keep every observation and every intermediate reasoning step in context forever for a long-running task. Planning limits how much needs to stay "live" at once by breaking work into steps; memory decides what's worth persisting outside the context window and retrieving back in only when relevant, rather than trying to keep everything present at all times.

---

## Interview Questions

**Why might an agent re-plan mid-task instead of following its original plan rigidly?** Because an early step's actual result can invalidate assumptions the original plan was built on — rigidly executing a plan based on outdated assumptions produces worse outcomes than checking, after each step, whether the plan still makes sense given what's actually been learned so far.

**What's the difference between short-term and long-term agent memory, and why can't everything just live in the context window?** Short-term memory is the current session's context, bounded by the context window's finite size; long-term memory persists information across sessions in an external store, retrieved back in only when relevant — everything can't live in context indefinitely because the context window is finite and every token in it costs latency and money regardless of whether it's currently useful.

**How is long-term agent memory architecturally similar to RAG?** Both retrieve relevant stored information (often from a vector database) based on the current query/context and inject it into the prompt — long-term memory is essentially RAG applied to the agent's own accumulated history and learned facts rather than a static external document corpus.

## Connections

- [[AI Agent End-to-End Execution Trace]] — the state-vs-memory distinction made concrete in a worked trace
- [[AI Agents Fundamentals]] — the loop planning and memory both support
- [[LLM Inference Fundamentals]] — the context-window constraint driving both design problems
- [[RAG Architecture]], [[Vector Search and Databases]] — the retrieval mechanism long-term memory typically reuses
- [[Workflows vs Agents]] — simpler workflows often need no planning/memory machinery at all

## One-line Summary

> Planning decomposes a task into manageable steps (with re-planning as the practical middle ground between rigid upfront plans and pure step-by-step reactivity), and memory decides what information persists beyond the current context window — both exist because the context window is finite, not by design choice alone.
