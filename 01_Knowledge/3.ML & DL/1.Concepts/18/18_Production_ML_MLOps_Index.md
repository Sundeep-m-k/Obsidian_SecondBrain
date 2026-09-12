# Topic 18: Production ML & MLOps — Complete Index

## Overview

Production ML covers drift detection, monitoring, evaluation, versioning, deployment, and retraining. Six files guide building reliable, maintainable systems.

**Total Content:** 6 files, ~20 KB | 25+ Questions | 20+ Code Examples

---

## Files at a Glance

| File | Focus | Key Concepts |
|------|-------|--------------|
| [[18.1_Data_Drift_and_Concept_Drift]] | Distribution shift detection, handling | KS test, retraining |
| [[18.2_Model_Monitoring_in_Production]] | Dashboards, SLAs, alerts, ground truth | Real-time tracking |
| [[18.3_Online_vs_Offline_Evaluation]] | A/B testing, shadow, canary, hybrid | Production validation |
| [[18.4_Model_Versioning_and_Reproducibility]] | Git, MLflow, experiment tracking | Reproducible ML |
| [[18.5_Deployment_Strategies]] | Blue-green, canary, shadow, A/B | Safe rollouts |
| [[18.6_Retraining_and_Model_Maintenance]] | Cold/warm start, triggers, workflow | Keeping models fresh |

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

Likely data drift or concept drift. Check: (1) Input distribution (feature values shifted)? (2) Target distribution (label proportion changed)? (3) Relationship (fraud tactics changed)? Use KS test for univariate drift. Monitor ground truth labels. Retrain on recent data if drift confirmed. See [[18.1_Data_Drift_and_Concept_Drift]], [[18.2_Model_Monitoring_in_Production]].

### Scenario 2: "How do you deploy a new model safely to 1M users?"

Canary strategy: (1) Shadow test (no real traffic). (2) Canary 5% (monitor for bugs, latency). (3) Gradual rollout (10% → 25% → 100%). (4) Full A/B test if business metrics critical (CTR, revenue). Rollback instantly if issues. See [[18.5_Deployment_Strategies]].

---

## Study Order

**2-hour:** All 6 files
**Practice:** 5 Q&A per file

---

## One-line Summaries

| File | Summary |
|------|---------|
| 18.1 | Data/concept drift degrade model performance; detect via KS test, handle via retraining |
| 18.2 | Monitor accuracy, latency, data quality daily; alert on SLA breach; use proxy metrics for delayed labels |
| 18.3 | Offline fast/cheap but incomplete; online A/B test measures real impact but slower; use both |
| 18.4 | Version models semantically; track code, data, hyperparams; use MLflow for reproducibility |
| 18.5 | Deploy via canary (5%→25%→100%), blue-green (instant rollback), shadow (no risk) |
| 18.6 | Retrain on performance drop or schedule; cold start (from scratch) vs. warm start (fine-tune) |

---

**Total Study Time:** 3-4 hours (deep dive)
**Interview Readiness:** 85% after 3 files, 95% after all 6
