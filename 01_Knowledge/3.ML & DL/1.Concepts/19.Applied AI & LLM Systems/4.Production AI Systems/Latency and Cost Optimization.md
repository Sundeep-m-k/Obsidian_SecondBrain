# Latency and Cost Optimization

## What is it?

Running LLM calls in production costs real money and real time per request — both scale roughly with the number of tokens processed (input and output combined), which makes token usage the central lever for optimizing either latency or cost, alongside a handful of infrastructure-level techniques.

---

## Why Cost and Latency Both Scale With Tokens

Every token in the prompt must be processed through every layer of the model (self-attention's cost grows with sequence length, per [[Attention Mechanism]]), and every generated output token requires a full additional forward pass through the model (LLMs generate one token at a time, autoregressively). A longer prompt or a longer requested output therefore directly increases both the compute cost billed and the wall-clock time to get a response — this single fact drives most practical LLM cost/latency engineering.

## Reducing Cost and Latency

**Prompt minimization** — remove unnecessary context (see [[RAG Architecture|context dilution]]), use concise instructions, and avoid re-sending large unchanging context on every call when a cheaper alternative (below) exists.

**Prompt caching** — many LLM APIs let a long, unchanging prefix of the prompt (e.g. a system prompt, or a large retrieved document reused across many queries) be cached server-side, so subsequent calls reusing that prefix are billed and processed faster than reprocessing it from scratch every time. This is a direct, practical lever whenever an application repeatedly sends the same large context with only the final query changing.

**Model selection** — smaller/cheaper models are faster and cheaper per token, and are often sufficient for simpler sub-tasks (classification, extraction, routing) — reserving the largest, most expensive model specifically for the sub-tasks that actually need its full capability, rather than routing everything through it by default.

**Streaming** — return generated tokens to the user as they're produced rather than waiting for the entire response to complete. Doesn't reduce total cost or total generation time, but dramatically improves *perceived* latency — a user sees the response starting immediately instead of waiting for the full answer, which matters enormously for user-facing latency even though the underlying compute cost is unchanged.

**Batching** (see [[Inference vs Training]]) — for high-throughput, non-real-time workloads, batching multiple requests together improves total throughput at some individual-request latency cost.

**Choosing max output length deliberately** — an unnecessarily generous max-token limit doesn't directly cost more if the model stops early, but combined with verbose prompting (e.g. encouraging long chain-of-thought reasoning where it isn't needed) directly inflates both cost and latency for no benefit.

---

## Rate Limits and Retries

LLM API providers impose rate limits (requests per minute, tokens per minute) to protect their own infrastructure — a production system must handle being rate-limited gracefully rather than treating it as an unrecoverable error. **Exponential backoff with jitter** is the standard retry pattern: on a rate-limit or transient error, wait an exponentially increasing amount of time between retries (with some randomness added, so many simultaneously-retrying clients don't all retry at exactly the same moment and immediately re-trigger the same rate limit). A well-built production LLM integration treats rate limits and transient failures as expected, routine conditions to handle, not exceptional ones to merely log and fail on.

---

## Interview Questions

**Why do both cost and latency scale with the number of tokens processed, not just the complexity of the task?** Every input token is processed through the model's attention mechanism (cost growing with sequence length), and every output token requires a separate full forward pass since generation is autoregressive — so a longer prompt or longer requested output directly increases both compute cost and wall-clock time, independent of how conceptually simple or complex the actual task is.

**What does prompt caching actually save, and when is it worth using?** It lets a long, unchanging prefix of the prompt be reprocessed faster (and billed differently) on subsequent calls that reuse it — worth using whenever an application repeatedly sends the same large context (a system prompt, a large retrieved document) with only a small trailing part of the prompt actually changing between calls.

**Why does streaming improve user experience without reducing actual cost or total generation time?** Streaming returns tokens as they're generated rather than waiting for the full response, so the user perceives the response starting immediately — but the total compute needed to generate the full response, and its total cost, are unchanged; streaming only changes when the user sees the output, not how much work was done to produce it.

**Why use exponential backoff with jitter for retries rather than retrying immediately or at a fixed interval?** Immediate or fixed-interval retries from many clients hitting the same rate limit tend to retry in sync, immediately re-triggering the same limit; exponentially increasing wait times reduce load during sustained issues, and adding random jitter prevents many simultaneously-retrying clients from synchronizing their retry attempts.

## Connections

- [[LLM Inference Fundamentals]] — context window size is the direct driver of prompt-side cost/latency
- [[Attention Mechanism]] — the quadratic-cost mechanism underlying why longer prompts cost more
- [[Inference vs Training]] — batching's latency/throughput tradeoff, applied here to LLM serving specifically
- [[Workflows vs Agents]] — an agent's multiple LLM calls per task directly multiply these costs versus a fixed workflow

## One-line Summary

> LLM cost and latency both scale with tokens processed, input and output alike — prompt minimization, caching, right-sizing the model per sub-task, and streaming are the main levers, and production systems must handle rate limits and transient failures via exponential backoff with jitter as routine, expected conditions.
