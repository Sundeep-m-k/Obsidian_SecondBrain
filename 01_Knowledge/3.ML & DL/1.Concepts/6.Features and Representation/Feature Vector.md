# Feature Vector

## What is it?

A **feature vector** is the numerical representation of a single data point — a column vector containing all the features of that example.

$$x = \begin{bmatrix}x_1 \\ x_2 \\ \vdots \\ x_d\end{bmatrix} \in \mathbb{R}^d$$

Every data point in ML must be converted to a feature vector before a model can process it.

---

## The Augmented Feature Vector

For linear models, a bias term $x_0 = 1$ is prepended:

$$\tilde{x} = \begin{bmatrix}1 \\ x_1 \\ x_2 \\ \vdots \\ x_d\end{bmatrix} \in \mathbb{R}^{d+1}$$

This allows the bias parameter $\theta_0$ to be absorbed into the dot product:
$$\hat{y} = \theta^T \tilde{x} = \theta_0 \cdot 1 + \theta_1 x_1 + \cdots + \theta_d x_d$$

---

## From Raw Data to Feature Vector

| Data type | Raw form | Feature vector |
|---|---|---|
| Tabular (numeric) | Row in a table | Direct concatenation |
| Categorical | "Red", "Blue", "Green" | One-hot: $[1,0,0]$, $[0,1,0]$, $[0,0,1]$ |
| Image ($H \times W \times C$) | Pixel grid | Flatten: vector of $H \cdot W \cdot C$ floats |
| Text | Sentence | TF-IDF vector, or word embedding average |
| Audio | Waveform | MFCC features, spectral features |

---

## Connections

- [[Features]] — the components of the feature vector
- [[Multiple Features]] — multiple components
- [[Vectorization]] — computing over feature vectors efficiently
- [[Feature Engineering]] — how feature vectors are constructed

---

## One-line Summary

> A feature vector is a single data point compressed into a fixed-length numerical column vector — the universal input format that every ML model operates on.
