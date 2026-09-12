# Vectorization

## What is it?

**Vectorization** is the practice of rewriting loops over training examples as **matrix/vector operations**, enabling modern hardware (CPUs with SIMD, GPUs) to process all examples simultaneously rather than one at a time.

> Vectorization is what makes ML practical at scale. A vectorised operation on 10,000 examples runs as fast as an operation on 1 — because the hardware does it in parallel.

---

## The Core Idea: Replace Loops With Matrix Ops

**Non-vectorised (slow):**
```python
predictions = []
for i in range(n):
    y_hat = 0
    for j in range(d+1):
        y_hat += theta[j] * x[i][j]
    predictions.append(y_hat)
```

**Vectorised (fast):**
```python
predictions = X @ theta   # Matrix multiplication: (n × d+1) @ (d+1,) = (n,)
```

The second version uses a single matrix multiply — hardware-optimised, runs in parallel.

---

## Key Vectorised Formulas

| Operation | Non-vectorised | Vectorised |
|---|---|---|
| Linear regression predictions | $\hat{y}^{(i)} = \theta^T x^{(i)}$ for each $i$ | $\hat{y} = X\theta$ |
| MSE cost | $J = \frac{1}{2n}\sum_i(\hat{y}^{(i)}-y^{(i)})^2$ | $J = \frac{1}{2n}\|X\theta - y\|^2$ |
| Gradient | $\nabla J_j = \frac{1}{n}\sum_i (\hat{y}^{(i)}-y^{(i)})x_j^{(i)}$ | $\nabla J = \frac{1}{n}X^T(X\theta - y)$ |
| Logistic predictions | $\hat{p}^{(i)} = \sigma(\theta^T x^{(i)})$ for each $i$ | $\hat{p} = \sigma(X\theta)$ |
| Euclidean distances | Loop over pairs | $D_{ij} = \|x^{(i)} - x^{(j)}\|^2$ via broadcasting |

---

## Why It's Fast

**Hardware parallelism:**
- **SIMD (Single Instruction, Multiple Data):** Modern CPUs can apply one instruction to 8–32 floats simultaneously.
- **GPUs:** Thousands of cores, each performing one float multiply-add. A matrix multiply on a GPU runs thousands of operations simultaneously.

**Memory locality:**
- Contiguous memory access (arrays, matrices) is much faster than pointer-chasing (loops with conditionals).
- Vectorised operations are optimised for cache efficiency.

**Speedups:** Vectorisation can make code 10×–1000× faster than Python loops.

---

## Broadcasting

**Broadcasting** allows operations between arrays of different shapes without explicit looping.

Example: subtract the mean from every row of a matrix:
```python
X_centred = X - X.mean(axis=0)   # mean shape (d,) broadcast over (n, d)
```

**Broadcasting rules (NumPy/PyTorch):**
1. If shapes differ in number of dimensions, pad with 1s on the left.
2. Dimensions of size 1 are stretched to match the other array.
3. If neither size is 1 and they differ: error.

Example: `(n, 1) + (1, d)` → broadcasts to `(n, d)`.

---

## Connections

- [[Feature Vector]] — the vector that vectorisation operates on
- [[Multiple Features]] — multiple features require matrix operations
- [[Gradient Descent]] — gradients are computed in vectorised form
- [[Linear Regression]] — all operations vectorised via $X\theta$

---

## One-line Summary

> Vectorization replaces slow Python loops with fast matrix operations, enabling ML training on millions of examples by leveraging CPU/GPU hardware parallelism — it is the reason deep learning is computationally feasible.
