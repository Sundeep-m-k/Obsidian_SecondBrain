# Matrix Factorization

## What is it?

**Matrix Factorization** is the dominant model-based approach to [[Collaborative Filtering]]. It decomposes the user-item rating matrix $R \in \mathbb{R}^{m\times n}$ into two low-rank matrices:

$$R \approx P Q^T$$

Where:
- $P \in \mathbb{R}^{m\times k}$ — user latent factor matrix (row $u$ = user $u$'s latent factors)
- $Q \in \mathbb{R}^{n\times k}$ — item latent factor matrix (row $i$ = item $i$'s latent factors)
- $k \ll \min(m,n)$ — the number of latent factors (hyperparameter, typically 20–200)

**Predicted rating:**
$$\hat{r}_{ui} = p_u^T q_i = \sum_{f=1}^k p_{uf} q_{if}$$

The latent factors capture hidden features: for movies, they might represent genre preferences, preference for blockbusters vs. art films, etc. — automatically discovered, not hand-labelled.

---

## Objective Function

Minimise the regularised squared error over all observed ratings:

$$J(P,Q) = \frac{1}{2}\sum_{(u,i): r_{ui}\text{ known}}\left(r_{ui} - p_u^T q_i\right)^2 + \frac{\lambda}{2}\left(\|P\|_F^2 + \|Q\|_F^2\right)$$

Where $\|P\|_F^2 = \sum_u\sum_f p_{uf}^2$ is the Frobenius norm (sum of all squared entries).

---

## Gradient Descent (SGD for MF)

For each observed rating $(u,i)$:

**Prediction error:**
$$e_{ui} = r_{ui} - p_u^T q_i$$

**Gradient updates:**
$$p_u \leftarrow p_u + \eta(e_{ui} q_i - \lambda p_u)$$
$$q_i \leftarrow q_i + \eta(e_{ui} p_u - \lambda q_i)$$

Repeat for all observed ratings (shuffle each epoch). This is **SGD** — update after each $(u,i)$ pair.

---

## Biases

Real ratings have systematic biases:
- User $u$ tends to rate higher/lower on average
- Item $i$ is generally popular/unpopular

**Rating with biases:**
$$\hat{r}_{ui} = \mu + b_u + b_i + p_u^T q_i$$

Where $\mu$ = global mean rating, $b_u$ = user bias, $b_i$ = item bias.

**Objective with biases:**
$$J = \frac{1}{2}\sum_{(u,i)}(r_{ui} - \mu - b_u - b_i - p_u^T q_i)^2 + \frac{\lambda}{2}(\|P\|_F^2 + \|Q\|_F^2 + \sum_u b_u^2 + \sum_i b_i^2)$$

This is the **SVD++ model** (Netflix Prize winner foundation).

---

## Alternating Least Squares (ALS)

Alternative to SGD: fix $Q$, solve for optimal $P$ in closed form; then fix $P$, solve for $Q$; repeat.

For fixed $Q$, each row $p_u$ solves:
$$\min_{p_u}\sum_{i: r_{ui}\text{ known}}(r_{ui} - p_u^T q_i)^2 + \lambda\|p_u\|^2$$

This is Ridge regression with analytical solution:
$$p_u = (Q_u^T Q_u + \lambda I)^{-1} Q_u^T r_u$$

Where $Q_u$ = rows of $Q$ for items rated by user $u$, $r_u$ = their ratings.

**ALS advantages:**
- Parallelisable across users/items
- Preferred for implicit feedback (clicks, views)
- More stable than SGD for sparse matrices

---

## Implicit Feedback

Often we don't have explicit ratings (1–5 stars) — only **implicit signals** (clicks, purchases, time spent).

**Model:** Binary preference $p_{ui} = \mathbf{1}[r_{ui} > 0]$ with confidence $c_{ui} = 1 + \alpha r_{ui}$ (higher interaction = higher confidence).

**Weighted objective (Hu et al., 2008):**
$$J = \sum_{u,i} c_{ui}(p_{ui} - p_u^T q_i)^2 + \lambda(\|P\|_F^2 + \|Q\|_F^2)$$

Solved efficiently with ALS (closed-form per user/item).

---

## Connections

- [[Collaborative Filtering]] — matrix factorization is model-based CF
- [[Recommender Systems]] — core algorithm
- [[Regularization]] — L2 penalty on latent factors

---

## One-line Summary

> Matrix factorization decomposes the sparse user-item rating matrix into low-rank user and item latent factor matrices, predicting ratings as dot products of latent vectors — trained by minimising regularised squared error via SGD or ALS, and extended with biases for state-of-the-art performance.
