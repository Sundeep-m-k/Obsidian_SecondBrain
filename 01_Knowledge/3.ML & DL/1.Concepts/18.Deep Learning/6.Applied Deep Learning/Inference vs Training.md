# Inference vs Training

## What is it?

**Training** is the process of fitting a model's parameters to data via [[Forward Propagation]], [[Backpropagation]], and repeated optimizer steps. **Inference** is using an already-trained, fixed model to produce predictions on new data. They have fundamentally different computational profiles, and confusing the two — or leaving training-only behavior active during inference — is a common source of subtle production bugs.

---

## Key Differences

| | Training | Inference |
|---|---|---|
| Gradients | Computed and used to update weights | Not needed at all |
| [[Dropout]] | Active | Disabled |
| [[Batch and Layer Normalization\|BatchNorm]] | Uses current batch's statistics | Uses running average from training |
| Compute pattern | Forward + backward pass, every batch | Forward pass only |
| Typical batch size | As large as memory allows, for stable gradient estimates | Often 1 (a single real-time request) or small |
| Memory needs | Must cache every layer's intermediate activations (for backprop) | No caching needed — activations can be discarded immediately after use |

Because inference needs no gradient computation, wrapping it in `torch.no_grad()` (see [[Training Loops and PyTorch Fundamentals]]) both saves memory (no cached activations for a backward pass that will never happen) and speeds up execution.

---

## GPU Batching at Inference

Training almost always processes large batches, since GPUs are efficient at large parallel matrix operations and a bigger batch amortizes fixed overhead. **Inference is often the opposite** — a production system frequently needs to serve one request at a time, in real time, which under-utilizes a GPU's parallelism (a batch of size 1 doesn't benefit from a GPU's ability to process many examples simultaneously). Production inference systems commonly use **dynamic batching** — briefly queueing incoming requests and grouping them into a batch before running the model — trading a small amount of added latency per request for much higher total throughput, a real engineering tradeoff relevant to Forward Deployed Engineer and Applied AI Engineer roles serving models in production.

## Latency vs. Throughput

**Latency** — time to get one prediction back. **Throughput** — total predictions served per unit time across many requests. Larger batches improve throughput (more predictions per GPU-second) but can worsen latency for any individual request (it has to wait for the batch to fill, or for the whole larger batch to finish processing). Production serving systems explicitly tune this tradeoff — real-time user-facing applications typically prioritize low latency (small or no batching, accepting lower throughput); offline/batch-processing pipelines prioritize throughput.

---

## Interview Questions

**Why is `torch.no_grad()` used during inference, and what does it actually save?** Inference never calls `.backward()`, so there's no need for autograd to build a computational graph or cache intermediate activations for one — disabling gradient tracking saves both memory and compute that would otherwise be spent maintaining state nothing will ever use.

**Why might a production inference system batch requests even though each user wants their result immediately?** A GPU is most efficient processing many examples in one large parallel operation; batching several incoming requests together (even at the cost of each waiting slightly for the batch to fill) can dramatically increase total throughput, which matters when serving many users — the tradeoff is a small latency cost per request for a large throughput gain overall.

**What's the practical difference between optimizing for latency and optimizing for throughput at inference time?** Latency-focused serving minimizes the time to answer any single request, typically using small or no batching; throughput-focused serving maximizes total requests handled per unit time, typically using larger batches — real-time user-facing systems usually need the former, offline processing pipelines the latter.

## Connections

- [[Training Loops and PyTorch Fundamentals]] — the mechanics that differ between train and eval mode
- [[Dropout]], [[Batch and Layer Normalization]] — the specific layers whose behavior changes between the two modes
- Production AI Systems (Applied AI & LLM Systems, module 19) — latency/throughput tradeoffs are central to serving any model (LLM or otherwise) in production

## One-line Summary

> Training computes and applies gradients over large batches; inference is a forward-only pass, often on small or single-example batches in real time — production systems explicitly trade latency against throughput via batching strategy, and forgetting to disable training-only behavior (dropout, batch statistics) at inference silently corrupts predictions.
