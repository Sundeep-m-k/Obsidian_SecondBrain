# Model Versioning and Reproducibility

## What is it?

**Model versioning:** Track which model version is deployed; maintain history of all models.

**Reproducibility:** Rebuild same model given code, data, hyperparameters. Essential for debugging, auditing, compliance.

---

## Why Versioning Matters

### Problem Without Versioning

```
Production crashes. What model is deployed?
Did you remember what hyperparams were used?
Can you rebuild it?
Can you rollback to previous version?
```

### Solution: Version Everything

```
Model V2.3.1 (SHA: abc123)
  - Training data: 2024-01-15
  - Features: 25 (see feature_list.json)
  - Hyperparams: lr=0.01, regularization=0.001
  - Performance: 91.2% accuracy (trained on 2024-01-10 data)
  - Dependencies: sklearn==1.0.2, numpy==1.21.0
  - Deployed: 2024-01-20
```

---

## Versioning Strategies

### 1. Semantic Versioning

Format: `MAJOR.MINOR.PATCH`

- **MAJOR:** Breaking changes (new features, retraining)
- **MINOR:** Non-breaking improvements
- **PATCH:** Bug fixes

**Example:** 1.2.3 → 1.2.4 (bug fix) → 1.3.0 (new feature) → 2.0.0 (breaking change)

### 2. Timestamp-Based

Format: `model_2024_01_20_v1`

Ordered by time; obvious when deployed.

---

## Experiment Tracking

**MLflow, Weights & Biases, Neptune**

```python
import mlflow

mlflow.log_param("learning_rate", 0.01)
mlflow.log_param("regularization", 0.001)
mlflow.log_metric("accuracy", 0.912)
mlflow.log_model(model, "model")  # Save model

# Later: Load experiment
best_run = mlflow.search_runs(filter_string="metrics.accuracy > 0.90")[0]
best_model = mlflow.pyfunc.load_model(best_run.info.artifact_uri + "/model")
```

**Benefits:**
- Centralized experiment tracking
- Reproducible (code + params + data logged)
- Dashboards to compare models
- Easy rollback

---

## Model Registry

**Three distinct things, often conflated**: a **model artifact** is just the serialized model file itself — weights and architecture, nothing more. An **experiment tracker** (MLflow, W&B, above) records the *process* of getting there — every run's parameters, metrics, and code version, across potentially hundreds of experiments, most of which were never deployed. A **model registry** sits downstream of both: it's the system of record for which specific artifacts are candidates for, or currently in, production — not every experiment, only the ones that graduated.

**A registry should be able to answer, for any deployed model, on demand**: Which model is in production right now? Who created it? Which exact dataset/code/config produced it (traceable back into the experiment tracker's record)? What metrics did it achieve, and against which evaluation set? Has it been formally approved for production? What model preceded it? Can it be rolled back to, and how quickly?

**What a registry tracks**: **versioning** (which specific artifact this is, uniquely identified); **metadata** (training data version, hyperparameters, code commit — the same information the Reproducibility Checklist below requires, but tied specifically to registry entries rather than experiment-tracker runs in general); **lineage** (the traceable path from raw training data through to this specific artifact); **stages/aliases** (e.g. `staging`, `production`, `archived` — a named pointer to whichever specific version currently holds that role, so "what's in production" is always one lookup, not a search through deployment logs); **approvals** (a formal sign-off step before a model can be promoted to a production-facing stage, appropriate for regulated or high-stakes domains); **promotion** (moving a version from one stage to the next, e.g. staging → production, ideally gated by the deployment strategy's own validation stages — see [[Deployment Strategies]]); and **rollback** (repointing the `production` alias back to the prior version — the registry is precisely what makes this a single lookup-and-repoint operation rather than "scramble to find the old artifact," as the Interview Questions below already describe).

**The full lifecycle, end to end**:

```
Training → Evaluation → Registration (artifact + metadata logged to the registry,
                                       not yet in any production-facing stage)
        → Validation (does it clear the bar to even be considered for promotion?)
        → Staging/canary (see Deployment Strategies' staged-rollout process)
        → Production (the registry's "production" alias now points here)
        → Monitoring (see Model Monitoring in Production)
        → Rollback/replacement (repoint the alias; the registry makes this fast and auditable)
```

The registry is the thread connecting every stage above — an experiment tracker alone can tell you a model scored well in training; only a registry can tell you, unambiguously and at any later time, exactly which artifact is serving production traffic right now and what specifically preceded it.

---

## Reproducibility Checklist

- ✓ Code versioned (git commit SHA)
- ✓ Data version (train data date, hash)
- ✓ Hyperparameters logged
- ✓ Dependencies frozen (requirements.txt)
- ✓ Random seed set
- ✓ Preprocessing logged
- ✓ Model artifact saved
- ✓ Test metrics recorded

---

## Interview Questions

**Q: Production model crashes and you need to rollback. How do you quickly deploy the previous version?**

Maintain model versioning + registry: (1) Each model version tagged (git SHA, timestamp). (2) Registry tracks: V2.3 deployed 2024-01-20, V2.2 deployed 2024-01-15. (3) Keep old model artifacts. Rollback: Update registry to point to V2.2, redeploy (< 5 min). Without versioning: Scramble to find old code, recompile, deploy (1+ hour). See [[Model Monitoring in Production]].

---

**Q: Can you rebuild your production model from scratch?**

Should be yes. Reproducible means: Given code (git SHA), data version, hyperparams, rebuild identical model. Requires: (1) Code versioned (git). (2) Data version (train data date, labeled). (3) Hyperparams logged. (4) Dependencies frozen (requirements.txt). (5) Random seed fixed. If yes → robust, auditable, compliant. If no → risky; model is black box. Use MLflow or similar to automate. See [[Model Versioning and Reproducibility]].

---

**Q: What's the actual difference between an experiment tracker and a model registry?** An experiment tracker records the process — every run's parameters and metrics across potentially hundreds of experiments, most never deployed; a registry is downstream of that, tracking only the artifacts that became production candidates, with stages/aliases (staging/production/archived), approval gates, and a lineage trail back to the exact data/code/config that produced each one. Confusing the two means treating "logged in MLflow" as equivalent to "known to be safe to roll back to" — it isn't.

**Q: How does a model registry make rollback fast?** Rollback becomes repointing a named alias (e.g. `production`) from the current version back to the prior one — a single, auditable operation — rather than reconstructing which artifact was previously deployed from deployment logs or institutional memory, which is exactly the "scramble to find old code" failure mode versioning is meant to prevent in the first place.

## Connections

- [[Deployment Strategies]] — promotion through registry stages should be gated by the deployment strategy's own staged-rollout validation, not a separate ungoverned process
- [[Model Monitoring in Production]] — what happens to a registered, deployed model after promotion
- [[Feature Stores]] — a model's reproducibility depends on the exact feature definitions that produced its training data, which a feature store versions on the data-pipeline side of this same problem

## One-line Summary

> Version every model (semver or timestamp); track code (git SHA), data, hyperparams, dependencies; use an experiment tracker (MLflow or similar) for the process of getting to a good model, and a model registry — a distinct system with stages, approvals, and lineage — for knowing unambiguously which specific artifact is in production and being able to roll back to the prior one in one auditable step.
