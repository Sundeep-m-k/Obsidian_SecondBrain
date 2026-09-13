# Feature Stores

## What is it?

A **feature store** is infrastructure that centralizes feature definitions and serves them consistently to both model training and real-time inference — solving one specific, painful production problem: **the feature definition used during training must match the feature definition used during inference, exactly**, or the model silently makes worse predictions in production than its evaluation numbers promised.

---

## The Core Problem: Training-Serving Skew

**Training-serving skew** is what happens when the feature computed at training time and the feature computed at inference time diverge — even subtly. Concretely: a training pipeline computes `avg_purchase_last_30_days` in a batch job, in Python, over historical data; a real-time inference service computes what's supposed to be the *same* feature in a different codebase (perhaps a different language, for latency reasons), reading from a live production database. If the two implementations round differently, handle nulls differently, or use a slightly different time window, the model at inference time is being fed a feature that doesn't match what it was actually trained on — and it can degrade the model's real-world accuracy without any code actually being "wrong" in isolation; each implementation can pass its own tests while still disagreeing with the other.

This is the single problem a feature store exists to solve: **one feature definition, computed once, served identically to both training and serving paths**, eliminating the two-implementation divergence risk entirely.

---

## Offline and Online Stores

**Offline store**: holds historical feature values, optimized for the large batch reads training and offline evaluation need — commonly backed by a data warehouse or a distributed file store. Training pulls a large historical dataset from here.

**Online store**: holds the *current* value of each feature, optimized for low-latency point lookups — commonly backed by a low-latency key-value store (Redis or similar). Real-time inference reads the current feature value for one entity (one user, one transaction) from here, in milliseconds, not from a batch warehouse query.

**Why some architectures need both**: training needs to see a feature's value *as it was at various points in the past* across potentially millions of historical examples (a batch, offline workload); serving needs a *single current value*, instantly, for one entity at inference time (a real-time, online workload) — these are different access patterns with different latency requirements, and a store optimized for one is typically a poor fit for the other, which is why a feature store's core architecture is usually a dual offline/online system with a defined, consistent path keeping both in sync from the same feature definitions.

---

## Point-in-Time Correctness

**The critical property, and the one most commonly gotten wrong**: training data must only use feature values that were **actually available at the time of the prediction being trained on** — not values that were true *by the time the training job happened to run*, which can leak future information into the training set (a specific, feature-store-relevant instance of [[Data Leakage]]).

**Fraud example**: training a model to predict whether a transaction is fraudulent, using a feature like `account_total_transaction_count`. If this feature is computed using the count **as it stands today** (when the training job runs) rather than **as it stood at the moment of each historical transaction being trained on**, an account that turned out to transact heavily *after* a fraudulent transaction gets a training-time feature value that already reflects information from the future relative to that transaction — the model learns a relationship that used information it could never have had at real prediction time, and its real-world performance will be worse than its (leaked) training/evaluation numbers suggest.

**How a feature store enforces this**: via **point-in-time joins** — when constructing a training dataset, the feature store joins each historical example to the feature values *as they existed at that example's specific timestamp*, not the feature's current value — this is a genuinely non-trivial capability to implement correctly by hand (it requires versioned, timestamped feature history, not just a current-value lookup), which is one of the concrete things a dedicated feature store provides that a naive "just query the current features table" approach does not.

---

## Feature Pipeline

```
Raw events (transactions, clicks, sessions, ...)
  → Feature computation (aggregate, transform, join)
  → Feature definitions (a named, versioned specification of how each feature is computed)
  → Offline storage (historical values, for training)
  → Training dataset (point-in-time-correct join against offline store)
  → Online materialization (the current value of each feature pushed to the online store)
  → Real-time inference (online store lookup, milliseconds, at prediction time)
```

The feature **definition** is written once and drives both the offline computation (for training data) and the online materialization (for serving) — this single-definition property is what actually prevents training-serving skew; two independently-written implementations of "the same" feature is exactly the failure mode a feature store's shared-definition architecture eliminates by construction.

---

## Feature Store vs. Database — "Why Can't I Just Use PostgreSQL?"

A plain database *can* store feature values — the question is what a feature store adds on top:

- **Feature definitions as a first-class, versioned artifact** — not just a column in a table, but a named, documented, versioned specification of how a feature is computed, reusable and discoverable across teams/models.
- **Lineage** — knowing which raw events and transformations produced a given feature value, useful for debugging exactly the kind of "training features looked correct but inference predictions were wrong" investigation this note's Phase-1-adjacent motivation is built around.
- **Point-in-time joins** — a genuinely hard capability to build correctly by hand (see above); a plain database query for "current value" is trivial, but "value as of an arbitrary past timestamp, for millions of historical examples, efficiently" is not something ordinary SQL against a mutable table gives you for free.
- **Offline/online consistency** — the dual-store architecture and the guarantee that both are fed from the same feature definition, which a single database doesn't provide on its own (a team could still build two separate pipelines feeding a shared Postgres instance and reintroduce skew regardless).
- **Discovery and reuse** — a searchable catalog of features other teams have already built and validated, reducing duplicate feature-engineering effort across models.

**When a dedicated feature store is overkill**: a single team, a small number of models, features that are simple enough to compute identically in both training and serving without much risk of divergence, or an early-stage project where the operational overhead of standing up and maintaining feature-store infrastructure isn't yet justified by the training-serving-skew risk it prevents — a well-disciplined shared feature-computation library (even without a dedicated store) can mitigate much of the risk at far lower operational cost, up to a point.

---

## Interview Questions

**What specific production problem does a feature store solve?** Training-serving skew — the risk that a feature is computed one way during training and a subtly different way during real-time inference, degrading model performance in production in a way that's invisible from the training pipeline's own perspective, since each implementation can look correct in isolation.

**Why do most feature-store architectures need both an offline and an online store?** Training/offline evaluation needs large batch reads of historical feature values across many examples (an offline access pattern); real-time inference needs a single current value for one entity in milliseconds (an online access pattern) — these have fundamentally different latency and throughput requirements that a single store optimized for one poorly serves for the other.

**Why does point-in-time correctness matter, with a concrete example of what goes wrong without it?** Training data must only reflect information available at each example's actual prediction time — using a feature's *current* value instead of its value *as of that historical timestamp* leaks future information into training (e.g. a fraud model trained on an account's current total transaction count, which for older training examples already reflects transactions that happened after the labeled event), producing training/evaluation metrics that don't hold up in real deployment.

**"We already have a database with feature tables — why do we need a feature store?"** A database alone stores values but doesn't guarantee the same feature *definition* computed the values used in training and the values served at inference — a feature store adds versioned feature definitions, lineage, point-in-time joins (non-trivial to build correctly by hand), and enforced offline/online consistency, specifically to prevent the two-implementation divergence a database alone doesn't structurally prevent.

**When would you NOT build a dedicated feature store?** Early-stage projects, a single team/small number of models, or features simple enough that a disciplined shared computation library achieves most of the same consistency benefit at far lower operational overhead — the decision should weigh the actual training-serving-skew risk being prevented against the real cost of standing up and maintaining the infrastructure.

## Connections

- [[Data Leakage]] — point-in-time correctness violations are a specific, feature-store-relevant instance of leakage
- [[Preprocessing Pipelines]] — the general discipline (fit on train, apply consistently) that feature stores enforce at production scale specifically for training-vs-serving consistency
- [[Deployment Strategies]] — feature-pipeline compatibility is one of the things that must match across a model deployment
- [[Model Versioning and Reproducibility]] — a model version's reproducibility depends on the exact feature definitions that produced its training data, which a feature store versions explicitly
- [[Vector Search and Databases]] — a different kind of specialized store solving a different specific problem (approximate similarity search vs. training-serving feature consistency), worth contrasting when asked "why not just use a database" in either context

## One-line Summary

> A feature store exists to guarantee the exact same feature definition feeds both training and real-time inference — preventing training-serving skew via a shared definition, an offline store for batch historical access and an online store for low-latency serving, and point-in-time joins that a plain database doesn't provide for free.
