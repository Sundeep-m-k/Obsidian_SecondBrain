# Density Estimation

## What is it?

**Density Estimation** is the task of estimating the **probability distribution** $p(x)$ from which the data was sampled — without any labels.

$$\mathcal{D} = \{x^{(1)},\ldots,x^{(n)}\} \overset{\text{density estimation}}{\longrightarrow} \hat{p}(x)$$

Once you have $\hat{p}(x)$, you can:
- Generate new samples from the estimated distribution
- Detect anomalies (points where $\hat{p}(x) \ll 1$)
- Understand data structure and modes

---

## Parametric Methods

Assume data comes from a known family of distributions; estimate parameters from data.

### Univariate Gaussian MLE

$$p(x) = \mathcal{N}(x; \mu, \sigma^2) = \frac{1}{\sqrt{2\pi\sigma^2}}\exp\!\left(-\frac{(x-\mu)^2}{2\sigma^2}\right)$$

MLE estimates:
$$\hat{\mu} = \frac{1}{n}\sum_{i=1}^n x^{(i)}, \qquad \hat{\sigma}^2 = \frac{1}{n}\sum_{i=1}^n(x^{(i)} - \hat{\mu})^2$$

### Multivariate Gaussian MLE

$$p(x) = \frac{1}{(2\pi)^{d/2}|\Sigma|^{1/2}}\exp\!\left(-\frac{1}{2}(x-\mu)^T\Sigma^{-1}(x-\mu)\right)$$

MLE estimates:
$$\hat{\mu} = \frac{1}{n}\sum_i x^{(i)}, \qquad \hat{\Sigma} = \frac{1}{n}\sum_i(x^{(i)}-\hat{\mu})(x^{(i)}-\hat{\mu})^T$$

### Gaussian Mixture Model (GMM)

Data is modelled as a mixture of $K$ Gaussians:
$$p(x) = \sum_{k=1}^K \pi_k \mathcal{N}(x; \mu_k, \Sigma_k)$$

Constraints: $\pi_k \geq 0$, $\sum_k \pi_k = 1$.

Trained via the **EM Algorithm**:

**E-step** (soft assignments):
$$r_{ik} = \frac{\pi_k\mathcal{N}(x^{(i)};\mu_k,\Sigma_k)}{\sum_j\pi_j\mathcal{N}(x^{(i)};\mu_j,\Sigma_j)}$$

**M-step** (update parameters):
$$\pi_k = \frac{1}{n}\sum_i r_{ik}$$
$$\mu_k = \frac{\sum_i r_{ik}x^{(i)}}{\sum_i r_{ik}}$$
$$\Sigma_k = \frac{\sum_i r_{ik}(x^{(i)}-\mu_k)(x^{(i)}-\mu_k)^T}{\sum_i r_{ik}}$$

EM is guaranteed to increase the log-likelihood at each iteration and converges to a local maximum.

---

## Non-Parametric Methods

Make no assumption about the distribution's form.

### Kernel Density Estimation (KDE)

Place a kernel (smooth bump) at each data point, sum them up:

$$\hat{p}(x) = \frac{1}{n}\sum_{i=1}^n K_h(x - x^{(i)}) = \frac{1}{nh^d}\sum_{i=1}^n K\!\left(\frac{x-x^{(i)}}{h}\right)$$

**Gaussian kernel:**
$$K_h(u) = \frac{1}{(2\pi h^2)^{d/2}}\exp\!\left(-\frac{\|u\|^2}{2h^2}\right)$$

**Bandwidth $h$:** Controls smoothness.
- $h$ too small: noisy, spiky estimate (overfits)
- $h$ too large: oversmoothed, loses modes (underfits)
- Optimal $h$ (Silverman's rule of thumb for Gaussian KDE): $h = 1.06\hat{\sigma}n^{-1/5}$

---

## Connections

- [[Anomaly Detection]] — uses density estimation to flag low-density points
- [[Unsupervised Learning]] — density estimation is unsupervised
- [[Clustering]] — GMM is a soft clustering via density

---

## One-line Summary

> Density estimation learns the probability distribution of the data from unlabelled examples — using parametric models (Gaussian, GMM) or non-parametric methods (KDE) — enabling anomaly detection, data generation, and understanding the structure of the data.
