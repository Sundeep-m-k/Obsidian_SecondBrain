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

## One-line Summary

> Version every model (semver or timestamp); track code (git SHA), data, hyperparams, dependencies; use MLflow for experiment tracking; maintain model registry for easy rollback.
