# LLM Inference Fundamentals

## What is it?

A **Large Language Model (LLM)** is a [[Transformer Architecture|decoder-only Transformer]] trained to predict the next token given everything before it. Everything covered here is about *using* an already-trained LLM at inference time — the practical layer between the Deep Learning architecture (module 18) and building an actual application (RAG, agents — modules 2–3 of this one).

**Tokenization is not covered here** — it's already built in depth under `4.NLP/02_Text_Preparation/3.Tokenization/` ([[Tokenization — BPE]], [[WordPiece and Unigram LM]], [[SentencePiece]]); the mechanics are identical for an LLM's input/output, so this note links there rather than duplicating it.

---

## Context Window

The **context window** is the maximum number of tokens (input + generated output combined) a model can process in one call — everything outside it is simply invisible to the model. This is a hard architectural limit tied to the Transformer's [[Attention Mechanism]]: self-attention's compute and memory cost grows roughly quadratically with sequence length ($O(n^2)$ for $n$ tokens), which is *why* context windows have a limit at all rather than being unbounded.

**Practical consequence**: a long conversation, a large document, or a big set of retrieved chunks (see [[RAG Architecture]]) can silently exceed the context window, causing older content to be truncated or the call to fail outright — a common, easy-to-miss production bug. Cost also scales with tokens processed, so a larger context window isn't free even when it fits — every extra token in the prompt is billed and adds latency.

## Sampling Parameters

An LLM's raw output at each step is a probability distribution over the vocabulary (via [[Activation Functions|softmax]] over the final layer's logits). How that distribution gets turned into an actual chosen token — greedy decoding, temperature, top-k, top-p, and how they interact — is covered in full, with a worked numerical example, in [[Sampling and Decoding Strategies]]. Short version: low temperature (or greedy) for tasks needing consistency (code generation, structured extraction); higher temperature with top-p for tasks wanting variety (creative writing, brainstorming).

---

## Structured Outputs

Getting an LLM to reliably return output matching a fixed schema (valid JSON with specific fields, rather than free-form prose) is a common practical need — an application consuming the output programmatically can't tolerate the model occasionally wrapping the JSON in explanatory text or getting a field name wrong. Modern approaches include **constrained decoding** (the API restricts which tokens are even eligible at each step, so only schema-valid output can be generated at all — the strongest guarantee) and **schema-guided prompting** (describing the desired schema in the prompt and hoping the model complies, weaker but works with any model). Prefer constrained decoding/function-calling-style structured output APIs when the calling application requires strict validity, since prompting alone can still fail on edge cases.

## Tool/Function Calling

Rather than only producing text, a modern LLM can be given a set of available **tools** (functions with a name, description, and parameter schema) and choose to output a structured request to call one, with arguments, instead of a direct answer — e.g. asked "what's the weather in Boston," the model outputs a call to a `get_weather(location="Boston")` tool rather than guessing. The calling application executes the actual function and returns its result to the model, which then uses it to produce a final answer. This is the foundational mechanism behind everything in [[AI Agents Fundamentals|AI Agents]] — an agent is, at its core, an LLM repeatedly deciding whether to call a tool or respond directly.

## Prompting Fundamentals

**Zero-shot** — ask directly, no examples. **Few-shot** — include a small number of example input-output pairs in the prompt before the real query, which measurably improves performance on tasks the model hasn't been explicitly fine-tuned for, by demonstrating the desired format/reasoning pattern in-context. **Chain-of-thought** — explicitly ask the model to reason step by step before giving a final answer, which improves performance on multi-step reasoning tasks by giving the model "space" to work through intermediate steps rather than jumping straight to an answer (this also produces more tokens, at proportionally higher cost and latency). **System prompts** — instructions given a privileged, persistent role in the conversation (behavior rules, persona, constraints) distinct from the user's actual query, letting an application's behavior be shaped independently of what any given user types.

---

## Interview Questions

**Why does an LLM have a fixed context window instead of being able to process arbitrarily long input?** Self-attention's compute and memory cost grows roughly quadratically with sequence length, so there's a hard architectural and practical limit on how many tokens can be processed in one call — an application has to actively manage what fits (truncation, summarization, retrieval) rather than assuming unlimited context.

**What's the difference between temperature and top-p, and would you ever use both together?** Temperature rescales the whole probability distribution's sharpness; top-p restricts sampling to a dynamically-sized set of the most probable tokens, adapting to how confident the model already is. They're commonly combined — temperature controlling overall randomness, top-p trimming away an unlikely long tail regardless of temperature.

**How does tool/function calling actually work, mechanically?** The model is given a schema describing available functions; instead of only generating natural-language text, it can generate a structured call to one of those functions with specific arguments, which the calling application then executes and feeds the result of back to the model for it to use in a final response — the model never executes anything itself.

**When would you use few-shot prompting over zero-shot?** When the model needs to see the exact desired output format or reasoning pattern demonstrated rather than described — few-shot examples reliably improve performance on tasks where the "shape" of a correct answer isn't obvious from the instruction alone.

## Connections

- [[Transformer Architecture]], [[Attention Mechanism]] — the architecture underlying every LLM discussed here
- [[Transformer End-to-End Walkthrough]] — a worked numerical example spanning tokenization through sampling
- [[Sampling and Decoding Strategies]] — full treatment of temperature/top-k/top-p, with a worked example and their interactions
- [[Tokenization — BPE]], [[WordPiece and Unigram LM]] — canonical tokenization coverage, linked not duplicated
- [[AI Agents Fundamentals]] — built directly on tool/function calling
- [[RAG Architecture]] — context window limits are a central practical constraint on how much retrieved content can be included
- Production AI Systems (this module, section 4) — cost and latency both scale with tokens processed

## One-line Summary

> An LLM is a decoder-only Transformer with a hard context-window limit driven by attention's quadratic cost; temperature and top-p control sampling randomness, structured outputs and tool/function calling let it interact with real systems reliably, and few-shot/chain-of-thought prompting are the cheapest levers for improving output quality without any fine-tuning.
