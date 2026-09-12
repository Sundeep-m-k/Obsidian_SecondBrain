# Transfer Learning and Fine-Tuning

## What is it?

**Transfer learning** reuses a model already trained on a large, general dataset as the starting point for a new, related task, instead of training from randomly-initialized weights. **Fine-tuning** is the specific mechanism: continue training that pretrained model's weights (all of them, or a subset) on the new task's smaller dataset.

---

## Why This Works: Feature Reuse

A model trained on a large, diverse dataset (millions of images, or a large text corpus) learns general-purpose features in its earlier/middle layers — edge and texture detectors for images, syntactic and semantic structure for text — that are broadly useful across many downstream tasks, not just the one it was originally trained on. Reusing those learned features means a new task needs far less data and far less compute than training an equivalent network from scratch, since the model doesn't need to relearn "what an edge looks like" or "how syntax works" — only the task-specific mapping on top.

## Feature Extraction vs. Full Fine-Tuning

**Feature extraction** — freeze the pretrained model's weights entirely, and only train a new, small task-specific head on top (e.g. a new classification layer). Cheapest, fastest, needs the least data; appropriate when the new task is closely related to the pretraining task and data is scarce.

**Full fine-tuning** — update all (or most) of the pretrained model's weights, typically with a small [[Learning Rate]] to avoid destroying the useful pretrained representations. Needs more data and compute than feature extraction but can adapt the model more thoroughly when the new task differs more from the original pretraining objective.

**Layer-wise/partial fine-tuning** — a middle ground: freeze early layers (most general, most transferable), fine-tune later layers (more task-specific in what they've learned).

---

## Why Fine-Tuning Uses a Small Learning Rate

A large [[Learning Rate]] applied to already-good pretrained weights can rapidly destroy the useful structure that pretraining spent enormous compute learning — a failure mode called **catastrophic forgetting**. Fine-tuning learning rates are typically 10–100x smaller than the rate used for training from scratch, making small, careful adjustments rather than large, disruptive ones.

## Parameter-Efficient Fine-Tuning (a modern LLM-specific note)

For very large models, updating every parameter during fine-tuning is expensive to store and serve (a separate full copy of a multi-billion-parameter model per fine-tuned task). Techniques like **LoRA** (Low-Rank Adaptation) freeze the original weights and train small, low-rank additive updates instead — dramatically cheaper to store and swap between tasks, at a small cost in fine-tuning flexibility relative to full fine-tuning. Worth knowing by name when discussing how modern LLMs get adapted to specific tasks or domains without full retraining.

---

## Interview Questions

**Why is transfer learning effective — what's actually being reused?** The general-purpose features learned from a large, diverse pretraining dataset (edges/textures for vision, syntax/semantics for language) are broadly useful across related tasks, so a new task can build on those already-learned representations instead of relearning them from random initialization with far less data.

**When would you freeze the pretrained weights entirely (feature extraction) vs. fine-tune the whole model?** Freeze and only train a new head when the new task is closely related to pretraining and data is scarce; fine-tune more (or all) of the weights when the new task differs more substantially and there's enough data to adapt the representations without overfitting.

**Why does fine-tuning use a much smaller learning rate than training from scratch?** A large learning rate can rapidly overwrite the useful structure the pretrained weights already encode — catastrophic forgetting — so fine-tuning takes small, careful steps instead of the larger steps appropriate for random initialization.

## Connections

- [[Learning Rate]], [[Learning Rate Scheduling]] — fine-tuning's smaller rate is a direct application of these ideas
- [[BERT Embeddings]] — a standard example of a pretrained model commonly fine-tuned for downstream NLP tasks
- LLM Fundamentals (Applied AI & LLM Systems, module 19) — fine-tuning vs. prompting is a central practical decision for adapting an LLM to a task

## One-line Summary

> Transfer learning reuses a pretrained model's general features instead of starting from random weights — fine-tune with a small learning rate to adapt without catastrophic forgetting, freeze more layers when data is scarce, and consider parameter-efficient methods like LoRA when full fine-tuning is too expensive to store per task.
