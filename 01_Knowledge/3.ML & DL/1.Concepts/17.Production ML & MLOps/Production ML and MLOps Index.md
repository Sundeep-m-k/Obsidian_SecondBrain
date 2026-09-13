# Topic 18: Production ML & MLOps — Complete Index

## Overview

Production ML covers feature infrastructure, drift detection, monitoring, evaluation, versioning/registry, deployment, and retraining. Seven files guide building reliable, maintainable systems.

**2026-09 update (P1 pass)**: Deployment Strategies and Model Monitoring in Production were substantially expanded (rolling deployment, worked canary/incident walkthroughs, the four-layer monitoring framework, drift taxonomy, alert design); Model Versioning and Reproducibility gained a full Model Registry section (previously one unexplained clause); Feature Stores was added as a new note (previously zero coverage vault-wide).

---

## Files at a Glance

| File | Focus | Key Concepts |
|------|-------|--------------|
| [[Feature Stores]] | Training-serving skew, offline/online stores, point-in-time correctness | Feature definitions, point-in-time joins |
| [[Data Drift and Concept Drift]] | Distribution shift detection, handling | KS test, retraining |
| [[Model Monitoring in Production]] | Four-layer framework, worked incident, delayed ground truth, drift taxonomy, alert design | Infrastructure/Data/Model/Business |
| [[Online vs Offline Evaluation]] | A/B testing, shadow, canary, hybrid | Production validation |
| [[Model Versioning and Reproducibility]] | Git, MLflow, experiment tracking, model registry | Reproducible ML, registry vs. tracker |
| [[Deployment Strategies]] | Rolling, blue-green, canary, shadow, A/B, worked canary example | Safe rollouts |
| [[Retraining and Model Maintenance]] | Cold/warm start, triggers, workflow | Keeping models fresh |

---

## Quick Workflow

```
Train model offline
        ↓
Shadow test (no real users)
        ↓
Canary deploy (5% users)
        ↓
Monitor accuracy, latency, data quality daily
        ↓
Detect drift via KS test or accuracy drop
        ↓
Retrain on recent data
        ↓
Validate on held-out test set
        ↓
Deploy (A/B test or canary)
        ↓
Repeat
```

---

## Interview Scenarios

### Scenario 1: "Model worked great in dev (91% accuracy) but degraded in prod (84% accuracy). Why?"

Likely data drift or concept drift. Check: (1) Input distribution (feature values shifted)? (2) Target distribution (label proportion changed)? (3) Relationship (fraud tactics changed)? Use KS test for univariate drift. Monitor ground truth labels. Retrain on recent data if drift confirmed. See [[Data Drift and Concept Drift]], [[Model Monitoring in Production]].

### Scenario 2: "How do you deploy a new model safely to 1M users?"

Canary strategy: (1) Shadow test (no real traffic). (2) Canary 5% (monitor for bugs, latency). (3) Gradual rollout (10% → 25% → 100%). (4) Full A/B test if business metrics critical (CTR, revenue). Rollback instantly if issues. See [[Deployment Strategies]].

---

## Study Order

**2-hour:** All 7 files
**Practice:** 5 Q&A per file

---

## One-line Summaries

| File | Summary |
|------|---------|
| Feature Stores | One feature definition serves both training and inference, preventing training-serving skew; point-in-time joins prevent leakage into training data |
| 18.1 | Data/concept drift degrade model performance; detect via KS test, handle via retraining |
| 18.2 | Monitor accuracy, latency, data quality daily; alert on SLA breach; use proxy metrics for delayed labels |
| 18.3 | Offline fast/cheap but incomplete; online A/B test measures real impact but slower; use both |
| 18.4 | Version models semantically; track code, data, hyperparams; use MLflow for reproducibility |
| 18.5 | Deploy via canary (5%→25%→100%), blue-green (instant rollback), shadow (no risk) |
| 18.6 | Retrain on performance drop or schedule; cold start (from scratch) vs. warm start (fine-tune) |

---

**Total Study Time:** 3-4 hours (deep dive)
**Interview Readiness:** 85% after 4 files, 95% after all 7
