# Recommender Systems

## What is it?

**Recommender Systems** are ML systems that predict a user's preference for items and suggest the most relevant ones. They drive massive value in industry: Netflix (films), Spotify (music), Amazon (products), YouTube (videos).

**Core problem:** Given a user $u$ and items $I = \{i_1, i_2, \ldots\}$, predict which unrated items the user will like most, and rank them.

---

## The Rating Matrix

The central data structure is a user-item rating matrix $R \in \mathbb{R}^{m \times n}$:

- $m$ = number of users
- $n$ = number of items
- $R_{ui}$ = rating of user $u$ for item $i$ (or 0/missing if not rated)

**The problem:** $R$ is extremely sparse — users rate only a tiny fraction of all items. The goal is to fill in the missing entries (matrix completion).

---

## Types of Recommenders

| Type | Uses | No user history needed? |
|---|---|---|
| **Collaborative Filtering** | User-item interactions only | ❌ |
| **Content-Based Filtering** | Item features + user profile | ✅ for new items |
| **Hybrid** | Both signals | Partially |
| **Knowledge-Based** | Explicit user requirements | ✅ |

---

## Connections

- [[Collaborative Filtering]] — user-item interaction based
- [[Content Based Filtering]] — item feature based
- [[Matrix Factorization]] — the dominant algorithm for collaborative filtering
- [[Recommendation Pipeline]] — end-to-end system

---

## One-line Summary

> Recommender systems predict user preferences over items from sparse historical interactions — powered by collaborative filtering (who's similar to you?), content-based filtering (what's similar to items you liked?), and matrix factorisation (latent factor models).
