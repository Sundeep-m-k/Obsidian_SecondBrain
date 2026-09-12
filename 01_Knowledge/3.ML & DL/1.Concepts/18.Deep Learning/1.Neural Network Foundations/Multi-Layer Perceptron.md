# Multi-Layer Perceptron

## What is it?

A **Multi-Layer Perceptron (MLP)**, also called a **feedforward neural network**, stacks layers of [[Perceptron|perceptron]]-like units, where each layer's output feeds into the next layer's input, with a non-linear [[Activation Functions|activation function]] applied at each hidden layer. This stacking is precisely what escapes the single perceptron's linear-only limitation.

$$h^{(1)} = \sigma(W^{(1)}x + b^{(1)}), \quad h^{(2)} = \sigma(W^{(2)}h^{(1)} + b^{(2)}), \quad \dots, \quad \hat{y} = f(W^{(L)}h^{(L-1)} + b^{(L)})$$

Each $W^{(l)}$ is a weight matrix, $\sigma$ a non-linear activation, and $f$ the output layer's activation (sigmoid for binary classification, softmax for multi-class, identity for regression).

---

## Why Non-Linearity Between Layers Is Essential

If every layer used a linear (identity) activation, stacking layers would be pointless: a composition of linear functions is still just one linear function, $W^{(2)}(W^{(1)}x) = (W^{(2)}W^{(1)})x$, collapsing the entire network back to a single linear transform no more expressive than one perceptron. The non-linear activation between layers is what actually gives depth its expressive power — see [[Activation Functions]] for why the specific choice of non-linearity matters too.

## The Universal Approximation Theorem

An MLP with a single hidden layer, given enough hidden units, can approximate any continuous function on a bounded domain to arbitrary precision. This is a genuinely important and often-misunderstood result: it says such a network *exists*, not that gradient descent will *find* it, and not that a wide shallow network is practically as good as a deep one — in practice, depth tends to represent complex functions far more parameter-efficiently than width alone.

---

## Architecture Choices

**Depth vs. width**: a deeper network can represent hierarchical structure (each layer building on features from the last) more efficiently than a wider, shallower one, but is harder to train (see [[Vanishing and Exploding Gradients in RNNs]] — the same gradient-through-many-layers problem that plagues RNNs plagues very deep MLPs too, addressed by the techniques in [[Training Deep Networks]]).

**Hidden layer size**: too few units underfits (not enough capacity to represent the pattern — see [[Underfitting]]); too many risks overfitting without [[Regularization]] and increases compute cost for often-marginal gains.

**Output layer**: matches the task — sigmoid + [[Logistic Loss]] for binary classification, softmax + cross-entropy for multi-class, identity + [[Mean Squared Error]] for regression.

---

## How It's Trained

An MLP is trained exactly like every other model in this vault — define a loss ([[Loss Function]]), compute its gradient with respect to every parameter, and take a step with [[Gradient Descent]]. The only new machinery needed is *how* to compute that gradient efficiently through many stacked layers, which is [[Backpropagation]] — covered as its own note because the mechanism (the chain rule applied systematically through a [[Computational Graphs|computational graph]]) is the one genuinely new idea an MLP introduces beyond linear/logistic regression.

---

## Interview Questions

**Why does stacking linear layers without non-linear activations accomplish nothing?** The composition of any number of linear transformations is itself a single linear transformation, so a network of purely linear layers has exactly the representational power of one layer — the activation function between layers is what makes depth meaningful.

**What does the universal approximation theorem actually guarantee, and what does it not guarantee?** It guarantees that a wide-enough single-hidden-layer network *can* approximate any continuous function — it says nothing about whether gradient descent will find such a network, how many hidden units are needed in practice, or whether a shallow network is as efficient as a deep one for a given function.

**Why go deeper instead of just wider?** Depth lets each layer build hierarchical features on top of the previous layer's representation, which tends to be dramatically more parameter-efficient for complex, structured functions than trying to represent the same function with one very wide layer — at the cost of harder optimization (vanishing/exploding gradients).

## Connections

- [[Perceptron]] — the single unit this stacks
- [[Activation Functions]] — the non-linearity that makes stacking meaningful
- [[Backpropagation]], [[Computational Graphs]] — how the gradient is actually computed through the stack
- [[Bias Variance Tradeoff]] — hidden layer width/depth is this network's primary complexity knob
- [[Training Deep Networks]] (module) — the techniques that make training a deep MLP actually work in practice

## One-line Summary

> An MLP stacks linear transformations with non-linear activations between them, which is the entire source of its expressive power over a single perceptron — trained the same way as any other model, via gradient descent, with backpropagation supplying the gradient efficiently through every layer.
