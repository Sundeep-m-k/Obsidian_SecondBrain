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

## Worked Numerical Example: A 2→2→1 Network

Used consistently here and in [[Backpropagation]] (which continues this exact example through the backward pass and a weight update) — small enough to check by hand, real enough to show every mechanic.

**Architecture**: 2 inputs → 2 hidden neurons (sigmoid) → 1 output neuron (sigmoid).

**Initial weights and biases** (arbitrary starting values, exactly as a network would be handed after [[Weight Initialization]]):
$$w_1{=}0.15,\ w_2{=}0.20,\ w_3{=}0.25,\ w_4{=}0.30 \ \text{(input→hidden)}, \quad b_1{=}0.35 \ \text{(shared hidden bias)}$$
$$w_5{=}0.40,\ w_6{=}0.45 \ \text{(hidden→output)}, \quad b_2{=}0.60 \ \text{(output bias)}$$

**Input**: $i_1{=}0.05,\ i_2{=}0.10$. **Target**: $y{=}0.01$.

### Step 1 — Weighted sum into the hidden layer

$$\text{net}_{h1} = w_1 i_1 + w_2 i_2 + b_1 = 0.15(0.05)+0.20(0.10)+0.35 = 0.3775$$
$$\text{net}_{h2} = w_3 i_1 + w_4 i_2 + b_1 = 0.25(0.05)+0.30(0.10)+0.35 = 0.3925$$

### Step 2 — Activation (sigmoid) of the hidden layer

$$\text{out}_{h1} = \sigma(0.3775) = \frac{1}{1+e^{-0.3775}} \approx 0.5933, \qquad \text{out}_{h2} = \sigma(0.3925) \approx 0.5968$$

### Step 3 — Weighted sum into the output layer

$$\text{net}_{o} = w_5\,\text{out}_{h1} + w_6\,\text{out}_{h2} + b_2 = 0.40(0.5933)+0.45(0.5968)+0.60 \approx 1.1059$$

### Step 4 — Prediction (activation of the output)

$$\hat y = \sigma(1.1059) \approx 0.7514$$

### Step 5 — Loss (squared error)

$$E = \tfrac{1}{2}(y-\hat y)^2 = \tfrac{1}{2}(0.01-0.7514)^2 \approx 0.2749$$

The network currently predicts 0.7514 for a target of 0.01 — badly wrong, as expected before any training. [[Backpropagation]] picks up here: computing exactly how much each of $w_1$ through $w_6$ contributed to this error, and updating them to reduce it. The same numbers are reused there — nothing here is thrown away, everything computed in steps 1-4 (the cached $\text{net}$ and $\text{out}$ values) is exactly what the backward pass needs, which is precisely the point made below about why the forward pass must be cached.

### Minimal Python (the same 5 steps, in code)

```python
import math

def sigmoid(z): return 1 / (1 + math.exp(-z))

i1, i2 = 0.05, 0.10
w1, w2, w3, w4, b1 = 0.15, 0.20, 0.25, 0.30, 0.35
w5, w6, b2 = 0.40, 0.45, 0.60
target = 0.01

net_h1 = w1*i1 + w2*i2 + b1
net_h2 = w3*i1 + w4*i2 + b1
out_h1, out_h2 = sigmoid(net_h1), sigmoid(net_h2)          # Step 2

net_o = w5*out_h1 + w6*out_h2 + b2
y_hat = sigmoid(net_o)                                      # Step 4

loss = 0.5 * (target - y_hat)**2                             # Step 5
print(f"prediction={y_hat:.4f}  loss={loss:.4f}")
# prediction=0.7514  loss=0.2749 — matches the hand computation above
```

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
