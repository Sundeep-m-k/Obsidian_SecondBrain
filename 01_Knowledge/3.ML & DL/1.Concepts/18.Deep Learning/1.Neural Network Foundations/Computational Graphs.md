# Computational Graphs

## What is it?

A **computational graph** represents a function as a directed acyclic graph — each node is either an input/parameter or an operation (add, multiply, matrix-multiply, apply an activation), and each edge carries a value forward from one operation to the next. It's the data structure that makes [[Backpropagation]] a mechanical, general-purpose algorithm rather than a hand-derived formula for each specific network architecture.

---

## Forward Pass

Evaluating the graph left-to-right — from inputs through every operation node to the final output (the loss) — is exactly [[Forward Propagation]]. Each node's output is cached, because those cached intermediate values are exactly what the backward pass needs.

**Example**: for $f(x,y) = (x+y) \cdot x$, the graph has an `add` node computing $s=x+y$, then a `multiply` node computing $f = s \cdot x$. Forward: given $x=2, y=3$: $s=5$, $f=10$.

## Backward Pass: The Chain Rule, Automated

Every operation node knows its own **local gradient** — how its output changes with respect to its own inputs, holding everything else fixed. Backpropagation walks the graph in reverse, at each node multiplying the local gradient by the gradient flowing in from downstream (the chain rule), and passing the result further upstream.

$$\frac{\partial f}{\partial x} = \frac{\partial f}{\partial s}\cdot\frac{\partial s}{\partial x} + \frac{\partial f}{\partial x}\Big|_{\text{direct}}$$

For the example above: $\frac{\partial f}{\partial s} = x = 2$, $\frac{\partial s}{\partial x}=1$, and $f=s\cdot x$ also depends on $x$ directly, giving $\frac{\partial f}{\partial x} = 2\cdot 1 + s = 2+5=7$. Every gradient in the entire graph — however deep, however many parameters — is computed this same mechanical way, one node at a time, in a single backward traversal.

---

## Why This Matters: Backprop Is Just Graph Traversal

The genuinely important insight computational graphs make concrete: backpropagation is **not** a special algorithm invented separately for neural networks — it's reverse-mode automatic differentiation applied to *any* computational graph, whether it represents a two-layer MLP or a hundred-layer Transformer. This is exactly what PyTorch's `autograd` and TensorFlow's `GradientTape` implement: they build the graph automatically as the forward pass executes, then walk it backward on `.backward()`, so a practitioner never derives gradients by hand for any real architecture.

```python
import torch
x = torch.tensor(2.0, requires_grad=True)
y = torch.tensor(3.0, requires_grad=True)
s = x + y
f = s * x
f.backward()      # walks the graph backward automatically
print(x.grad)      # 7.0 — matches the hand computation above
```

---

## Static vs. Dynamic Graphs

Older frameworks (early TensorFlow) built the graph once, ahead of time, then ran data through it repeatedly — efficient but rigid (control flow like loops/conditionals inside the model is awkward to express). Modern frameworks (PyTorch, and TensorFlow 2's eager mode) build the graph dynamically on every forward pass, which makes debugging and variable-length/conditional architectures far more natural at a small runtime cost. This distinction is worth knowing by name in an interview even without needing to build a graph engine from scratch.

---

## Interview Questions

**Why is a computational graph the right abstraction for understanding backpropagation?** Because it reduces gradient computation to one mechanical rule — multiply the local gradient by the incoming upstream gradient, at every node, walking backward — which generalizes identically to any architecture, however complex, without needing a bespoke derivation per model.

**What's cached during the forward pass, and why does the backward pass need it?** Each node's output value (and often its local gradient, computable from its inputs) — the backward pass needs these cached values to compute each node's contribution to the final gradient without recomputing the entire forward pass from scratch.

**What's the practical difference between a static and dynamic computational graph?** A static graph is built once and reused across many forward passes (efficient, but awkward for architectures with data-dependent control flow); a dynamic graph is rebuilt on every forward pass (more flexible, natural for debugging and variable-length inputs, at some runtime overhead) — this is why PyTorch's eager, dynamic-graph design became the research-community default.

## Connections

- [[Backpropagation]] — the specific backward-traversal algorithm this graph structure enables
- [[Forward Propagation]] — the forward traversal that populates the graph's cached values
- [[Gradient Descent]] — what the resulting gradients are actually used for
- [[Multi-Layer Perceptron]] — the simplest architecture whose graph is worth tracing by hand once

## One-line Summary

> A computational graph turns gradient computation into a single mechanical rule (multiply local gradients by upstream gradients, walking backward) that applies identically to any architecture — this is precisely what backpropagation is, and precisely what PyTorch/TensorFlow automate under the hood.
