# Perceptron

## What is it?

The **perceptron** is the simplest possible neural network: a single unit that computes a weighted sum of its inputs, adds a bias, and passes the result through a step function to produce a binary output.

$$\hat{y} = \begin{cases} 1 & \text{if } w^\top x + b > 0 \\ 0 & \text{otherwise}\end{cases}$$

This is mechanically identical to [[Logistic Regression]] with a hard threshold instead of a sigmoid — the perceptron is the historical ancestor both of logistic regression and of every deep network in this module.

---

## The Perceptron Learning Rule

Unlike gradient descent on a smooth loss, the classic perceptron algorithm updates weights directly from misclassified examples:

$$w \leftarrow w + \eta(y - \hat{y})x, \quad b \leftarrow b + \eta(y-\hat{y})$$

If a point is correctly classified, $(y-\hat{y})=0$ and nothing changes. If misclassified, the weight vector shifts toward the misclassified point (for a false negative) or away from it (for a false positive). The **Perceptron Convergence Theorem** guarantees this converges to a perfect separator in finitely many steps — *if and only if* the data is linearly separable.

---

## The Fatal Limitation: Linear Separability

A single perceptron can only represent linear decision boundaries — it cannot learn XOR, the canonical example of a function with no linear separator (the two positive-class points sit on opposite corners of the input square from the two negative-class points). This limitation, publicized in Minsky and Papert's 1969 analysis, is historically why neural network research stalled for over a decade: a single layer simply isn't expressive enough for most real problems.

## Why This Matters: Motivating the MLP

The fix isn't a smarter single unit — it's *stacking* units into layers. A [[Multi-Layer Perceptron]] with even one hidden layer can represent XOR and, more generally (by the universal approximation theorem), can approximate any continuous function given enough hidden units. Understanding *why* a single perceptron fails on XOR is what makes the necessity of depth in every later architecture (MLPs, CNNs, Transformers) concrete rather than assumed.

---

## Interview Questions

**Why can't a single perceptron learn XOR?** XOR isn't linearly separable — no single straight line (or hyperplane in higher dimensions) separates the positive from the negative examples, and a perceptron can only represent linear decision boundaries.

**What does the Perceptron Convergence Theorem actually guarantee, and what's the catch?** It guarantees the perceptron learning rule finds a perfect linear separator in finitely many updates — but only if the data is linearly separable in the first place; on non-separable data, the algorithm never converges and oscillates indefinitely.

**How is a perceptron related to logistic regression?** They share the same linear scoring function $w^\top x+b$; logistic regression replaces the perceptron's hard step function with a smooth sigmoid, which makes the output a genuine probability and makes the loss differentiable, enabling gradient-based optimization instead of the mistake-driven perceptron rule.

## Connections

- [[Multi-Layer Perceptron]] — stacking perceptron-like units to escape the linear-separability limit
- [[Logistic Regression]], [[Sigmoid Function]] — the smooth, differentiable generalization
- [[Activation Functions]] — the step function is the perceptron's (non-differentiable) activation
- [[Decision Boundary]] — the perceptron's boundary is always linear

## One-line Summary

> A perceptron is a single linear unit with a hard threshold that provably learns any linearly separable pattern but nothing more (XOR being the canonical counterexample) — motivating every multi-layer architecture that follows.
