# Content-Based Filtering

## What is it?

**Content-Based Filtering** recommends items similar to those a user has liked before, based on **item features** — not on other users' behaviour.

> "Because you liked Action Movie A, here are other Action/Thriller movies with similar features..."

---

## How It Works

**Item representation:** Each item $i$ has a feature vector $x_i \in \mathbb{R}^d$ describing its content.
- Movie: genre (action/comedy/drama), actors, director, year, plot keywords
- Song: tempo, key, instrumentation, genre, lyrics sentiment
- Product: category, brand, price, description TF-IDF

**User profile:** For each user $u$, build a profile vector $\theta_u \in \mathbb{R}^d$ representing their preferences.

**Prediction:**
$$\hat{r}_{ui} = \theta_u^T x_i$$

**Learning $\theta_u$:** For each user $u$, minimise the squared error on their rated items:

$$\min_{\theta_u} \frac{1}{2}\sum_{i: r_{ui}\text{ known}}(\theta_u^T x_i - r_{ui})^2 + \frac{\lambda}{2}\|\theta_u\|^2$$

This is a Ridge regression problem per user — closed-form solution exists.

---

## Advantages and Limitations

| Advantage | Limitation |
|---|---|
| No cold start for new items (if features are available) | Cold start for new users (no history) |
| No need for other users' data | Requires good item features |
| Transparent recommendations (explainable) | Cannot recommend outside known item types |
| | Diversity problem: over-specialises to past preferences |

---

## Connections

- [[Recommender Systems]] — content-based is one approach
- [[Collaborative Filtering]] — the user-behaviour-based alternative
- [[Matrix Factorization]] — learns both user and item latent features

---

## One-line Summary

> Content-based filtering recommends items similar in feature space to those a user has liked — learning a user preference vector via regression on item features — avoiding dependency on other users but limited by the quality of item features.
