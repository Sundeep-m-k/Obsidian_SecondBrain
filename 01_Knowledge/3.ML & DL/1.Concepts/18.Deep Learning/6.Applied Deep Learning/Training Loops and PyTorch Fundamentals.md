# Training Loops and PyTorch Fundamentals

## What is it?

A **training loop** is the concrete code structure that repeatedly executes [[Forward Propagation]], computes a loss, runs [[Backpropagation]], and applies an optimizer step — the practical implementation of everything covered conceptually elsewhere in this module. PyTorch is the dominant framework for expressing this, and its core mechanics are worth knowing explicitly rather than only conceptually.

---

## The Anatomy of a Training Loop

```python
for epoch in range(num_epochs):
    for batch_x, batch_y in dataloader:
        optimizer.zero_grad()          # clear gradients from the previous step
        predictions = model(batch_x)   # forward pass
        loss = loss_fn(predictions, batch_y)
        loss.backward()                # backward pass — populates .grad on every parameter
        optimizer.step()               # apply the update rule (SGD/Adam/etc.) using those gradients
```

**Why `zero_grad()` is required**: PyTorch *accumulates* gradients into `.grad` by default on every `.backward()` call, rather than overwriting them — a design choice that supports use cases like gradient accumulation across multiple mini-batches. Forgetting `zero_grad()` silently adds each batch's gradient on top of the last, corrupting every update after the first — one of the most common real bugs in a from-scratch training loop.

## Autograd: What `.backward()` Actually Does

Every operation on a tensor with `requires_grad=True` is recorded into a dynamic [[Computational Graphs|computational graph]] as it happens (PyTorch's "define-by-run" design). Calling `.backward()` on the final loss walks that graph in reverse — exactly the mechanism described in [[Backpropagation]] — populating each parameter's `.grad` attribute with $\partial \text{loss}/\partial \theta$, without any of it needing to be derived by hand.

## Train Mode vs. Eval Mode

`model.train()` and `model.eval()` toggle behavior that must differ between training and inference — most importantly [[Dropout]] (active only in train mode) and [[Batch and Layer Normalization|BatchNorm]] (uses batch statistics in train mode, a running average in eval mode). Forgetting `model.eval()` before inference silently leaves dropout active and BatchNorm using per-batch statistics, corrupting predictions in a way that's easy to miss since the code still runs without error.

```python
model.eval()
with torch.no_grad():          # disables gradient tracking — inference doesn't need it
    predictions = model(X_test)
```

`torch.no_grad()` is a separate, complementary step: it stops autograd from building a computational graph for these operations at all, saving memory and compute during inference where no `.backward()` will ever be called.

---

## Gradient Accumulation

When a batch large enough for stable training doesn't fit in GPU memory, accumulate gradients across several smaller mini-batches before calling `optimizer.step()` (and only then `zero_grad()`) — this simulates a larger effective batch size without needing the memory to hold it all at once, at the cost of more forward/backward passes for the same effective update.

---

## Interview Questions

**Why does PyTorch require calling `optimizer.zero_grad()` before every backward pass?** Because `.grad` accumulates by default rather than being overwritten on each `.backward()` call — this supports gradient accumulation across mini-batches, but forgetting to clear it means each batch's gradient silently adds to the last, corrupting training.

**What's the practical difference between `model.eval()` and `torch.no_grad()`?** `model.eval()` changes layer *behavior* (disables dropout, switches BatchNorm to running-average statistics); `torch.no_grad()` disables gradient *tracking* entirely for memory/compute savings during inference — they're complementary and both typically needed together at inference time.

**What is gradient accumulation and when would you use it?** Running several forward/backward passes on smaller sub-batches, summing their gradients, before taking one optimizer step — used when the desired effective batch size doesn't fit in available GPU memory, simulating a larger batch at the cost of more passes per update.

## Connections

- [[Forward Propagation]], [[Backpropagation]], [[Computational Graphs]] — the concepts this loop implements concretely
- [[Dropout]], [[Batch and Layer Normalization]] — the layers whose behavior train/eval mode actually toggles
- [[Optimizers in Deep Learning]] — what `optimizer.step()` executes
- Inference vs Training (this module) — the broader distinction this note's train/eval-mode section is one instance of

## One-line Summary

> A training loop is forward pass → loss → `zero_grad()` → `backward()` → `optimizer.step()`, repeated per batch — PyTorch's autograd builds the computational graph dynamically and walks it backward automatically, and `model.eval()` plus `torch.no_grad()` are both required at inference to get correct, efficient predictions.
