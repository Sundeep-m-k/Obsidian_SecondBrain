# CNN Fundamentals

## What is it?

A **Convolutional Neural Network (CNN)** processes grid-structured data (images, most commonly) using **convolutional layers** — small learned filters slid across the input, each producing one value per position by computing a local weighted sum, rather than the fully-connected layers of an [[Multi-Layer Perceptron|MLP]] connecting every input to every output.

$$(I * K)(i,j) = \sum_m \sum_n I(i+m, j+n)\,K(m,n)$$

$I$ is the input, $K$ the filter ("kernel") — typically small (e.g. $3\times3$), slid across every position of $I$.

---

## Why Convolution Instead of a Fully-Connected Layer

**Parameter sharing**: the same small filter is applied at every position, so a filter that detects a vertical edge in the top-left of an image uses exactly the same weights to detect one anywhere else — dramatically fewer parameters than a fully-connected layer would need to learn the same pattern independently at every location.

**Local connectivity**: each output value depends only on a small local neighborhood of the input (the filter's size), matching the reasonable prior that nearby pixels are the ones with meaningful local structure (edges, textures) — an MLP's every-input-to-every-output connectivity has no such built-in assumption.

**Translation invariance**: because the same filter scans every position, a pattern is detected regardless of where in the image it appears — a cat in the top-left is recognized by the same filters as a cat in the bottom-right.

## Pooling

**Pooling** (typically max-pooling: take the maximum value in each small region) downsamples the feature map, reducing spatial resolution and parameter count in later layers while adding a small amount of local translation invariance (a feature slightly shifted within the pooling window still produces the same pooled output).

## Typical Architecture

Stack `Conv → Activation (ReLU) → Pool` blocks, with the number of filters typically increasing with depth (early layers detect simple local patterns like edges; later layers, operating on the outputs of earlier layers, detect increasingly complex/abstract combinations of them) and spatial resolution decreasing. The final feature maps are flattened and passed through one or more fully-connected layers to produce the final classification or regression output.

---

## Interview Questions

**Why do CNNs use far fewer parameters than an equivalent fully-connected network on the same image?** Parameter sharing — each filter's weights are reused at every spatial position, rather than a fully-connected layer needing an independent weight for every input-output pixel pair.

**What does pooling accomplish?** It reduces spatial resolution (and thus computation and parameter count in later layers) while adding a degree of local translation invariance, since a small shift in a detected feature's position still falls within the same pooling window.

**Why do early CNN layers typically detect simple features and later layers detect complex ones?** Each layer's receptive field (the region of the original input it depends on) grows with depth, since it's built on top of already-aggregated information from the previous layer — early layers see small, simple local regions (edges); later layers, several convolutions removed from the raw input, effectively see much larger, more complex combinations of what earlier layers already detected.

## Connections

- [[Multi-Layer Perceptron]] — the fully-connected alternative CNNs improve on for grid-structured data
- [[Activation Functions]] — ReLU is standard between convolutional layers, for the same reasons as in any deep network
- [[Attention Mechanism]] — Vision Transformers (an alternative to CNNs for images) replace convolution with self-attention entirely, trading CNN's built-in locality/translation-invariance priors for attention's flexibility, given enough data

## One-line Summary

> CNNs exploit grid structure via small, position-shared filters (parameter sharing, local connectivity, translation invariance) instead of an MLP's unstructured full connectivity — the standard architecture for image data, though Transformers are an increasingly common alternative given enough training data.
