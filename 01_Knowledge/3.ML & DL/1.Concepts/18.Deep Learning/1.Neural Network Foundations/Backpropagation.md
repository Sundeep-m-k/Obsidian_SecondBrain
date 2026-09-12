# Backpropagation

## What is it?

**Backpropagation** computes the gradient of the loss with respect to *every* parameter in a neural network by applying the chain rule systematically, backward through the network, reusing intermediate results at every layer instead of recomputing them from scratch for each parameter. It's what makes [[Gradient Descent]] tractable for a network with millions of parameters.

---

## The Core Idea: Reusing Work via the Chain Rule

Naively, computing $\frac{\partial L}{\partial W^{(1)}}$ (the gradient with respect to the *first* layer's weights) requires differentiating through every layer between it and the loss. Doing this separately for every parameter in every layer would recompute the same intermediate derivatives over and over. Backpropagation instead computes one quantity per layer — the gradient of the loss with respect to that layer's *output* — and passes it backward, so each layer's gradient computation reuses the layer after it's already-computed result:

$$\delta^{(l)} = \frac{\partial L}{\partial z^{(l)}} = \left(W^{(l+1)\top}\delta^{(l+1)}\right)\odot \sigma'(z^{(l)})$$

$$\frac{\partial L}{\partial W^{(l)}} = \delta^{(l)}(a^{(l-1)})^\top, \quad \frac{\partial L}{\partial b^{(l)}} = \delta^{(l)}$$

$\delta^{(l)}$ ("the error at layer $l$") is computed once per layer, in a single backward pass, and directly gives that layer's weight and bias gradients. This turns what would be an exponential-in-depth amount of redundant computation into work that's linear in the number of layers.

---

## Worked Intuition: A Two-Layer Network

For $a^{(1)} = \sigma(W^{(1)}x)$, $\hat{y} = \sigma(W^{(2)}a^{(1)})$, $L = \text{loss}(\hat{y}, y)$:

1. Forward pass computes and caches $a^{(1)}$, $\hat{y}$ (see [[Forward Propagation]]).
2. $\delta^{(2)} = \frac{\partial L}{\partial \hat{y}}\cdot\sigma'(z^{(2)})$ — how much the output layer's pre-activation should change.
3. $\frac{\partial L}{\partial W^{(2)}} = \delta^{(2)}(a^{(1)})^\top$ — the output layer's gradient, using cached $a^{(1)}$.
4. $\delta^{(1)} = (W^{(2)\top}\delta^{(2)})\odot\sigma'(z^{(1)})$ — propagate the error one layer back.
5. $\frac{\partial L}{\partial W^{(1)}} = \delta^{(1)}x^\top$ — the first layer's gradient, computed without ever re-deriving anything already used in step 3.

---

## Why Backprop Is "Just" Reverse-Mode Automatic Differentiation

This entire procedure is a special case of reverse-mode autodiff applied to the network's [[Computational Graphs|computational graph]] — there's no separate "neural network gradient theory," just the chain rule applied efficiently and systematically. This is why frameworks like PyTorch implement `.backward()` once, generically, for any graph a user builds, rather than hand-coding a gradient formula per architecture.

## Why It Can Fail: Vanishing and Exploding Gradients

Each layer's $\delta^{(l)}$ is multiplied by that layer's activation derivative $\sigma'(z^{(l)})$ and weight matrix as it propagates backward. Across many layers, this is a repeated product — if the typical factor is less than 1 (e.g. sigmoid's derivative, which maxes at $0.25$), the gradient shrinks exponentially with depth (**vanishing**); if greater than 1, it grows exponentially (**exploding**). This is the exact same mechanism covered for RNNs in [[Vanishing and Exploding Gradients in RNNs]] — RNNs just make it especially visible because "depth" there means "number of time steps," which can be very large. [[Activation Functions|ReLU]], careful [[Weight Initialization]], and [[Batch and Layer Normalization]] all exist specifically to keep this product well-behaved.

---

## Interview Questions

**Why is backpropagation more efficient than computing each parameter's gradient independently?** It computes one error term $\delta^{(l)}$ per layer and reuses it for every parameter in that layer and every layer before it, rather than re-deriving shared intermediate derivatives from scratch for each of potentially millions of individual parameters.

**How does backpropagation relate to the chain rule?** It *is* the chain rule, applied systematically backward through a computational graph — $\delta^{(l)}$ at each layer is exactly the chain-rule product of the upstream gradient and that layer's local derivative, computed once and passed further back.

**Why do deep networks suffer from vanishing gradients, and how is backpropagation implicated?** Backpropagation multiplies gradients by each layer's activation derivative and weight matrix as it moves backward; across many layers this is a long repeated product, and if the typical factor is below 1 (as with saturating activations like sigmoid) the product shrinks toward zero, leaving early layers with almost no gradient signal to learn from.

## Connections

- [[Computational Graphs]], [[Forward Propagation]] — the structure and forward pass backprop reverses
- [[Gradient Descent]] — what the resulting gradients are used for
- [[Vanishing and Exploding Gradients in RNNs]] — the same failure mode, most visible in recurrent networks but present in any deep network
- [[Weight Initialization]], [[Batch and Layer Normalization]] — the standard fixes

## One-line Summary

> Backpropagation computes every parameter's gradient in one efficient backward pass by propagating a single per-layer error term $\delta^{(l)}$ via the chain rule — mechanically just reverse-mode automatic differentiation on the network's computational graph, and the reason gradients can vanish or explode across many layers.
