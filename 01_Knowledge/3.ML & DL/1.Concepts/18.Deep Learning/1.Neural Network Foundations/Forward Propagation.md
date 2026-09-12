# Forward Propagation

## What is it?

**Forward propagation** is the process of computing a neural network's output by pushing an input through every layer in order — each layer's weighted sum and activation feeding directly into the next layer's input — ending in a prediction and, during training, a loss value.

$$z^{(l)} = W^{(l)}a^{(l-1)} + b^{(l)}, \quad a^{(l)} = \sigma(z^{(l)}), \quad a^{(0)}=x$$

$z^{(l)}$ is layer $l$'s pre-activation (weighted sum), $a^{(l)}$ its post-activation output. This is exactly the [[Multi-Layer Perceptron]] equation applied one layer at a time, and it's the forward traversal of the network's [[Computational Graphs|computational graph]].

---

## Why It's Called "Forward" (and Why That Matters for Training)

The name distinguishes it from [[Backpropagation]], which traverses the same graph in reverse. During training, both passes happen every iteration: forward propagation computes the prediction and loss; backpropagation then computes how each parameter should change to reduce that loss. **Every intermediate $z^{(l)}$ and $a^{(l)}$ computed during the forward pass must be cached**, because backpropagation's chain-rule computation needs them — this is a genuine memory cost of training deep networks, and it's why memory usage (not just compute) grows with depth and batch size.

## Vectorized, Batched Computation

In practice, forward propagation runs on a **batch** of examples simultaneously, not one at a time — $X$ becomes a matrix (batch size $\times$ input dimension) rather than a single vector, and $z^{(l)}=W^{(l)}A^{(l-1)}+b^{(l)}$ becomes a matrix multiplication. This is what makes GPU acceleration effective: modern GPUs are built for exactly this kind of large, parallel matrix multiplication, and processing a batch at once amortizes the fixed overhead of a GPU kernel launch across many examples instead of paying it per-example.

---

## Interview Questions

**What has to be cached during the forward pass, and why?** Every layer's pre-activation and post-activation values — backpropagation's chain-rule computation at each layer needs the values that were actually produced during the forward pass, not just the final output, so recomputing them from scratch during the backward pass would be wasteful (and in practice, isn't how it's done).

**Why is forward propagation implemented as batched matrix multiplication rather than looping over examples one at a time?** GPUs are optimized for large parallel matrix operations; batching amortizes fixed per-operation overhead across many examples at once and lets the whole batch's forward pass execute as a small number of large matrix multiplications instead of many small ones.

## Connections

- [[Multi-Layer Perceptron]] — the architecture this equation describes, layer by layer
- [[Computational Graphs]] — the forward traversal of the graph
- [[Backpropagation]] — the reverse pass that consumes what forward propagation caches
- [[Activation Functions]] — $\sigma$ in the equation above

## One-line Summary

> Forward propagation computes a network's output layer by layer via $z^{(l)}=W^{(l)}a^{(l-1)}+b^{(l)}$, $a^{(l)}=\sigma(z^{(l)})$, run as batched matrix multiplication for GPU efficiency, caching every intermediate value that backpropagation will need on the way back.
