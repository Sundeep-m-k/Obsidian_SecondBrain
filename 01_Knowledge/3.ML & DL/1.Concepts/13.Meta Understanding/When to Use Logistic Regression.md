# When to Use Logistic Regression

## Quick Decision

Use **Logistic Regression** when:

1. ✅ Your **output is categorical** (binary or multi-class)
2. ✅ You need **probability estimates** (not just hard class labels)
3. ✅ You need **interpretability** — coefficients as log-odds ratios
4. ✅ The **decision boundary is approximately linear** (or can be made so with feature engineering)
5. ✅ You want a **fast, interpretable baseline** for classification

Do NOT use when:
- ❌ Output is continuous → use [[Linear Regression]]
- ❌ Decision boundary is highly non-linear and feature engineering won't help → use trees, SVMs, NNs
- ❌ Very high-dimensional sparse data with complex interactions → use neural networks
- ❌ You need to model complex feature interactions → use ensemble methods

---

## Decision Flowchart

```
Is your output categorical?
    │
    ├─ NO  → Use Regression
    │
    └─ YES
        │
        Do you need probabilities AND interpretability?
        │
        ├─ YES → Logistic Regression ✅
        │
        └─ NO
            │
            Is the boundary approximately linear?
            │
            ├─ YES → Logistic Regression ✅ or Linear SVM
            │
            └─ NO  → Random Forest / Gradient Boosting / Neural Network
```

---

## Checking Linear Separability

**Plot features vs. class label** (for $d \leq 3$). Do the classes look separable by a line/plane?

**Class-conditional distributions:** If class 0 and class 1 have non-overlapping or slightly overlapping distributions for key features, logistic regression will work well.

**Logistic regression R² equivalent:** Use McFadden's pseudo-$R^2$:
$$R^2_{\text{McFadden}} = 1 - \frac{\log\mathcal{L}(\hat{\theta})}{\log\mathcal{L}(\theta_0)}$$
Where $\theta_0$ = intercept-only model. Values > 0.2 indicate good fit.

---

## Practical Notes

- **Scale features** (standardise) before training.
- **Regularise by default** — use `C=1.0` in sklearn (equivalent to $\lambda=1$). Tune $C$ via CV.
- **Check class balance** — if severely imbalanced, use `class_weight='balanced'`.
- **Tune threshold** based on business cost of FP vs FN — default 0.5 is rarely optimal.
- **Interpret coefficients**: $e^{\theta_j}$ = odds ratio for feature $j$.
- **Baseline**: Does logistic regression beat always predicting the majority class? If not, features carry no classification signal.

---

## Connections

- [[Logistic Regression]] — the algorithm
- [[Classification]] — the task
- [[Binary Classification]] — most common use case
- [[When to Use Linear Regression]] — for continuous outputs
- [[How to Diagnose Overfitting]] — if val accuracy is much lower than train

---

## One-line Summary

> Use logistic regression when your output is categorical and you need calibrated probabilities with interpretable coefficients — it's the classification equivalent of linear regression: the right first attempt before more complex models.
