# Activation Functions

## What is it?

An **activation function** is the non-linearity applied to each layer's weighted sum in a neural network. It's the single ingredient that makes stacking layers in a [[Multi-Layer Perceptron]] meaningful at all — without it, any depth of stacked linear layers collapses to one linear transform.

---

## Sigmoid

$$\sigma(z) = \frac{1}{1+e^{-z}}, \quad \sigma'(z) = \sigma(z)(1-\sigma(z))$$

Already covered in full in [[Sigmoid Function]] as logistic regression's output function. As a *hidden-layer* activation, it has a serious flaw: its derivative maxes out at $0.25$ (at $z=0$) and approaches zero everywhere else, so gradients shrink every time they pass backward through a sigmoid layer — the direct cause of the [[Vanishing and Exploding Gradients in RNNs|vanishing gradient]] problem in deep networks. Rarely used in hidden layers of modern networks for this reason; still standard as an output activation for binary classification.

## Tanh

$$\tanh(z) = \frac{e^z-e^{-z}}{e^z+e^{-z}} = 2\sigma(2z)-1$$

Zero-centered (outputs in $(-1,1)$ rather than sigmoid's $(0,1)$), which helps optimization since a zero-centered activation's gradients don't all push weights in the same direction. Still saturates at both ends like sigmoid, though, so it suffers the same vanishing-gradient problem, just somewhat less severely.

## ReLU (Rectified Linear Unit)

$$\text{ReLU}(z) = \max(0,z), \quad \text{ReLU}'(z) = \begin{cases}1 & z>0 \\ 0 & z<0\end{cases}$$

The modern default for hidden layers. Its gradient is exactly $1$ for any positive input — no saturation, no shrinking gradient, which is why it enabled training much deeper networks than sigmoid/tanh ever could. Computationally trivial (just a max with zero).

**The dying ReLU problem**: if a unit's weighted input is consistently negative across training, its gradient is exactly zero and it stops updating permanently — the unit is "dead." Large learning rates or poor initialization make this more likely (see [[Weight Initialization]], [[Learning Rate]]).

## Leaky ReLU / Parametric ReLU

$$\text{LeakyReLU}(z) = \begin{cases}z & z>0\\ \alpha z & z \leq 0\end{cases}, \quad \alpha \approx 0.01$$

Gives negative inputs a small non-zero gradient instead of exactly zero, directly fixing the dying-ReLU problem at negligible extra cost. Parametric ReLU (PReLU) makes $\alpha$ itself a learned parameter rather than a fixed constant.

## Softmax

$$\text{softmax}(z)_i = \frac{e^{z_i}}{\sum_j e^{z_j}}$$

Not a hidden-layer activation — the standard *output* activation for multi-class classification, converting a vector of raw scores ("logits") into a probability distribution over $k$ classes that sums to 1. Generalizes the [[Sigmoid Function]] (binary case) to $k>2$ classes.

## GELU / Swish

$$\text{GELU}(z) \approx z \cdot \Phi(z) \quad (\Phi = \text{standard normal CDF})$$

Smooth approximations to ReLU used in modern Transformer architectures (BERT, GPT). Unlike ReLU's hard cutoff at zero, these are smooth everywhere, which empirically improves optimization in very deep Transformer stacks — worth knowing the name if discussing modern LLM internals, without needing to derive the formula from scratch.

---

## Choosing an Activation Function

| Layer | Standard choice | Why |
|---|---|---|
| Hidden layers | ReLU (or Leaky ReLU / GELU) | Non-saturating gradient, cheap, avoids vanishing gradients |
| Binary classification output | Sigmoid | Maps to a $(0,1)$ probability |
| Multi-class classification output | Softmax | Maps to a probability distribution over classes |
| Regression output | None (identity) | Output should be unbounded, matching the target's range |

---

## Interview Questions

**Why did ReLU largely replace sigmoid/tanh in hidden layers?** Sigmoid and tanh saturate at both ends, so their gradient shrinks toward zero for large-magnitude inputs — stacked across many layers this compounds into the vanishing gradient problem. ReLU's gradient is a constant $1$ for any positive input, so it doesn't shrink, which made training much deeper networks practical.

**What is the dying ReLU problem and how is it fixed?** A unit whose input is consistently negative gets a gradient of exactly zero and stops learning permanently; Leaky ReLU fixes this by giving negative inputs a small non-zero slope instead of clamping them to a flat zero gradient.

**Why use softmax instead of $k$ independent sigmoids for multi-class classification?** Softmax's outputs are coupled — they sum to exactly 1, giving a genuine probability distribution over mutually exclusive classes — whereas independent sigmoids would each ask "is it this class, yes/no" without that constraint, which is the right choice only for multi-*label* (non-exclusive) classification instead.

## Connections

- [[Sigmoid Function]] — the binary case, covered in full under Logistic Regression
- [[Multi-Layer Perceptron]] — why any non-linearity is needed here at all
- [[Vanishing and Exploding Gradients in RNNs]] — sigmoid/tanh's saturation is the textbook cause
- [[Weight Initialization]] — poor initialization compounds the dying-ReLU and vanishing-gradient problems
- [[Logistic Loss]] — softmax's multi-class generalization, cross-entropy loss

## One-line Summary

> Activation functions are what give depth its power at all — ReLU is the modern hidden-layer default because its gradient doesn't saturate, sigmoid/softmax remain standard at the output layer for binary/multi-class probabilities, and every saturating activation (sigmoid, tanh) risks the vanishing-gradient problem that motivated ReLU's adoption in the first place.
