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

## Worked Numerical Example: Continuing the 2→2→1 Network

Picking up exactly where [[Forward Propagation]] left off — same network, same numbers. Forward pass gave: $\text{out}_{h1}{=}0.5933$, $\text{out}_{h2}{=}0.5968$, $\hat y{=}0.7514$, target $y{=}0.01$, loss $E{=}0.2749$. Learning rate $\eta{=}0.5$.

### Step 6 — Derivative of the loss w.r.t. the prediction

$$\frac{\partial E}{\partial \hat y} = -(y-\hat y) = -(0.01-0.7514) = 0.7414$$

Intuition: the loss is a squared error, so its slope with respect to $\hat y$ is proportional to how wrong the prediction is, in the direction that *increasing* $\hat y$ further would make the error worse (since $\hat y$ is already too high relative to $y=0.01$).

### Step 7 — Gradient through the output layer (chain rule, step 1 of 2)

The sigmoid's own derivative is $\sigma'(z) = \sigma(z)(1-\sigma(z))$ — evaluated at the output layer:
$$\frac{\partial \hat y}{\partial \text{net}_o} = \hat y(1-\hat y) = 0.7514(0.2486) \approx 0.1868$$

Multiply the two pieces (chain rule) to get $\delta_o$ — "how much the output layer's pre-activation should change to reduce the loss":
$$\delta_o = \frac{\partial E}{\partial \hat y}\cdot\frac{\partial \hat y}{\partial \text{net}_o} = 0.7414 \times 0.1868 \approx 0.1385$$

Now get the actual weight gradients — $\delta_o$ times whatever fed into that weight (exactly $\frac{\partial L}{\partial W^{(l)}}=\delta^{(l)}(a^{(l-1)})^\top$ from above, applied to two scalar weights instead of a matrix):
$$\frac{\partial E}{\partial w_5} = \delta_o \cdot \text{out}_{h1} = 0.1385 \times 0.5933 \approx 0.0822, \qquad \frac{\partial E}{\partial w_6} = \delta_o \cdot \text{out}_{h2} \approx 0.0827$$

### Step 8 — Gradient through the hidden layer (chain rule, step 2 of 2 — the actual "back" propagation)

This is the step that makes backprop *backprop*: $\delta_o$'s influence has to flow backward through $w_5$/$w_6$ to reach the hidden layer, then get multiplied by the hidden layer's *own* sigmoid derivative — exactly $\delta^{(l)} = (W^{(l+1)\top}\delta^{(l+1)})\odot\sigma'(z^{(l)})$ from the general formula above:

$$\delta_{h1} = \delta_o \cdot w_5 \cdot \text{out}_{h1}(1-\text{out}_{h1}) = 0.1385 \times 0.40 \times 0.2413 \approx 0.01337$$
$$\delta_{h2} = \delta_o \cdot w_6 \cdot \text{out}_{h2}(1-\text{out}_{h2}) = 0.1385 \times 0.45 \times 0.2406 \approx 0.01500$$

Notice these hidden-layer gradients ($\approx0.013$–$0.015$) are roughly **10x smaller** than the output layer's gradient ($0.1385$) — a first-hand look at *why* vanishing gradients happen: each layer further back multiplies in another sigmoid derivative (always $\le 0.25$) and another weight, shrinking the signal every step backward. In a 2-layer network this is barely noticeable; in a 50-layer network the same multiplication, repeated 50 times, is what makes early layers nearly untrainable without the fixes in the section below.

Then the hidden-layer weight gradients, same pattern as step 7:
$$\frac{\partial E}{\partial w_1} = \delta_{h1}\cdot i_1 \approx 0.000669, \quad \frac{\partial E}{\partial w_2} = \delta_{h1}\cdot i_2 \approx 0.001337$$
$$\frac{\partial E}{\partial w_3} = \delta_{h2}\cdot i_1 \approx 0.000750, \quad \frac{\partial E}{\partial w_4} = \delta_{h2}\cdot i_2 \approx 0.001500$$

### Step 9 — Weight update (gradient descent)

$$w \leftarrow w - \eta \frac{\partial E}{\partial w}$$

$$w_5^{\text{new}} = 0.40 - 0.5(0.0822) \approx 0.3589, \qquad w_6^{\text{new}} = 0.45 - 0.5(0.0827) \approx 0.4086$$
$$w_1^{\text{new}} \approx 0.14967, \quad w_2^{\text{new}} \approx 0.19933, \quad w_3^{\text{new}} \approx 0.24963, \quad w_4^{\text{new}} \approx 0.29925$$

Notice $w_5, w_6$ moved far more (by ~0.04) than $w_1$–$w_4$ (by ~0.0007–0.0014) — a direct, numerical illustration of the same shrinking-gradient effect from step 8: later layers get bigger updates, earlier layers get much smaller ones, from a single backward pass.

### Step 10 — Second forward pass: did the loss actually go down?

Re-running [[Forward Propagation]]'s steps 1-5 with the updated weights (biases held fixed for simplicity):

$$\text{out}_{h1} \approx 0.5933 \ (\text{barely changed}), \quad \text{out}_{h2} \approx 0.5968 \ (\text{barely changed})$$
$$\text{net}_o^{\text{new}} = 0.3589(0.5933) + 0.4086(0.5968) + 0.60 \approx 1.0568, \qquad \hat y^{\text{new}} = \sigma(1.0568) \approx 0.7421$$
$$E^{\text{new}} = \tfrac{1}{2}(0.01-0.7421)^2 \approx 0.2680$$

**Loss dropped from $0.2749 \to 0.2680$** after exactly one gradient step. Not much on its own — this is what thousands of repeated forward/backward/update cycles, each nudging the loss down a little further, actually look like in aggregate. This is literally what a training loop does, one iteration at a time.

### Python: the Full Manual Backward Pass and Update

Continuing directly from [[Forward Propagation]]'s forward-pass code:

```python
# ... continuing from the forward pass code in Forward Propagation.md ...
eta = 0.5

# Step 6-7: output layer
dE_dyhat = -(target - y_hat)
dyhat_dnet_o = y_hat * (1 - y_hat)
delta_o = dE_dyhat * dyhat_dnet_o

dE_dw5 = delta_o * out_h1
dE_dw6 = delta_o * out_h2

# Step 8: hidden layer (using the ORIGINAL w5, w6 — not yet updated)
delta_h1 = delta_o * w5 * out_h1 * (1 - out_h1)
delta_h2 = delta_o * w6 * out_h2 * (1 - out_h2)

dE_dw1 = delta_h1 * i1
dE_dw2 = delta_h1 * i2
dE_dw3 = delta_h2 * i1
dE_dw4 = delta_h2 * i2

# Step 9: update
w1 -= eta * dE_dw1; w2 -= eta * dE_dw2
w3 -= eta * dE_dw3; w4 -= eta * dE_dw4
w5 -= eta * dE_dw5; w6 -= eta * dE_dw6

# Step 10: second forward pass
net_h1 = w1*i1 + w2*i2 + b1
net_h2 = w3*i1 + w4*i2 + b1
out_h1, out_h2 = sigmoid(net_h1), sigmoid(net_h2)
net_o = w5*out_h1 + w6*out_h2 + b2
y_hat_new = sigmoid(net_o)
loss_new = 0.5 * (target - y_hat_new)**2
print(f"new prediction={y_hat_new:.4f}  new loss={loss_new:.4f}  (was {loss:.4f})")
# new prediction=0.7421  new loss=0.2680  (was 0.2749) — the loss went down
```

**A critical implementation detail this code makes concrete**: `delta_h1`/`delta_h2` use the *original* `w5`/`w6`, not the already-updated ones — every gradient in one backward pass must be computed using the weight values from *before* any update in that same step, or the chain rule's math is simply wrong. This is why frameworks compute the entire backward pass first, then apply all updates together, never interleaving the two.

### The Same Thing, in PyTorch — After Understanding the Manual Version

```python
import torch

i = torch.tensor([[0.05, 0.10]])
target = torch.tensor([[0.01]])

W1 = torch.tensor([[0.15, 0.25], [0.20, 0.30]], requires_grad=True)  # columns = h1, h2
b1 = torch.tensor([0.35, 0.35], requires_grad=True)
W2 = torch.tensor([[0.40], [0.45]], requires_grad=True)
b2 = torch.tensor([0.60], requires_grad=True)

hidden = torch.sigmoid(i @ W1 + b1)
y_hat = torch.sigmoid(hidden @ W2 + b2)
loss = 0.5 * (target - y_hat).pow(2).sum()

loss.backward()          # computes every gradient above automatically
print(W2.grad)            # tensor([[0.0822], [0.0827]]) — matches dE/dw5, dE/dw6 by hand

with torch.no_grad():
    W1 -= 0.5 * W1.grad; W2 -= 0.5 * W2.grad
    b1 -= 0.5 * b1.grad; b2 -= 0.5 * b2.grad
```

`loss.backward()` is doing *exactly* steps 6-8 above — walking the same [[Computational Graphs|computational graph]], applying the same chain rule, computing the same numbers — just without anyone writing out `delta_o`/`delta_h1`/`delta_h2` by hand. Nothing about the underlying math changes; only who does the bookkeeping does.

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

**In the worked 2→2→1 example above, why did $w_5$/$w_6$ (hidden→output) get a much larger update than $w_1$–$w_4$ (input→hidden)?** The hidden-layer gradients ($\delta_{h1},\delta_{h2}$) are the output-layer gradient ($\delta_o$) multiplied by an additional weight and an additional sigmoid derivative (always $\le 0.25$) — one extra multiplicative shrinking factor per layer further back — which is the exact mechanism, seen in miniature on two layers, that becomes severe vanishing gradients across dozens of layers.

**What would happen numerically if the learning rate in that example were 5.0 instead of 0.5?** The much larger update to $w_5$/$w_6$ specifically (whose raw gradients were already ~60x larger than $w_1$-$w_4$'s) could overshoot past a good value entirely, potentially increasing the loss on the next forward pass rather than decreasing it — illustrating why [[Learning Rate]] choice interacts directly with how gradient magnitudes already differ across layers.

## Connections

- [[Computational Graphs]], [[Forward Propagation]] — the structure and forward pass backprop reverses
- [[Gradient Descent]] — what the resulting gradients are used for
- [[Vanishing and Exploding Gradients in RNNs]] — the same failure mode, most visible in recurrent networks but present in any deep network
- [[Weight Initialization]], [[Batch and Layer Normalization]] — the standard fixes

## One-line Summary

> Backpropagation computes every parameter's gradient in one efficient backward pass by propagating a single per-layer error term $\delta^{(l)}$ via the chain rule — mechanically just reverse-mode automatic differentiation on the network's computational graph, and the reason gradients can vanish or explode across many layers.
