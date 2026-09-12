# Feature Importance

## What is it?

**Feature importance** quantifies how much each feature contributes to a model's predictions — the first, cheapest tool for answering "why did the model decide this?" before reaching for something heavier like [[SHAP]].

---

## Three Different Things People Call "Feature Importance"

**Impurity-based (tree models only)** — sums how much each feature reduces impurity (Gini/entropy) across all splits in a [[Decision Tree]] or [[Random Forest]]. Fast (free byproduct of training) but biased toward high-cardinality features (a feature with many unique values gets more opportunities to produce a good-looking split, inflating its apparent importance even if it's not truly more predictive).

**Permutation importance** — shuffle one feature's values (breaking its relationship with the target) and measure how much the model's performance drops. A larger drop means the model relied on that feature more heavily. Model-agnostic (works on any model, not just trees), and directly measures impact on actual predictive performance rather than a training-time proxy like impurity reduction — but it's expensive (requires re-scoring the whole validation set once per feature) and can be misleading for two highly correlated features, since permuting one doesn't hurt performance much when the other still carries the same signal.

```python
from sklearn.inspection import permutation_importance
result = permutation_importance(model, X_val, y_val, n_repeats=10, random_state=0)
```

**Coefficient magnitude (linear/logistic models only)** — for [[Linear Regression]] or [[Logistic Regression]], the fitted weight $\theta_j$ directly measures each feature's contribution, *but only if features are scaled first* ([[Feature Scaling]]) — otherwise a feature's raw scale, not its predictive value, determines its coefficient's magnitude.

---

## Global vs. Local Importance

Everything above is **global** — one importance score per feature, describing the model's behavior *on average* across the whole dataset. This is a genuinely different question from **local** interpretability — "why did the model predict this specific outcome for this specific example?" — which is what [[SHAP]] and [[LIME]] answer. A feature can have high global importance while being irrelevant to one particular prediction, or vice versa.

---

## Common Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| Trusting impurity importance with mixed-cardinality features | High-cardinality features look artificially important | Use permutation importance instead |
| Reading raw linear coefficients on unscaled features | Magnitude reflects feature scale, not importance | Scale features first |
| Treating high importance as "this feature causes the outcome" | Importance measures predictive association, not causation | See [[Hypothesis Test]] and experimental design for actual causal claims |
| Trusting importance under strong feature correlation | Correlated features "share" and dilute each other's apparent importance | Check correlation first; consider grouping correlated features before interpreting |

## When to Use / When Not To

**Use** as the first, cheap pass at understanding a model — feature selection, sanity-checking that the model is using sensible signal, communicating model behavior to a non-technical stakeholder at a high level. **Reach for [[SHAP]] or [[LIME]] instead** when the question is about one specific prediction rather than overall model behavior, or when correlated features make global importance scores untrustworthy.

---

## Interview Questions

**Why is impurity-based feature importance from a random forest sometimes misleading?** It's biased toward high-cardinality/continuous features, which get more candidate split points and therefore more chances to produce an impurity-reducing split, inflating their apparent importance independent of true predictive value.

**How does permutation importance work, and what's its main cost?** Shuffle one feature at a time and measure the resulting performance drop on held-out data — model-agnostic and tied directly to real predictive performance, but expensive (one full re-scoring pass per feature) and unreliable when features are correlated.

**A model has 100 features, two of which are near-duplicates of each other. What happens to their importance scores?** Both will show artificially low importance under permutation importance — since permuting one still leaves the other carrying the same signal, neither shuffle hurts performance much even though the *pair* is jointly important.

## Connections

- [[SHAP]], [[LIME]] — the local-interpretability counterparts to this global view
- [[Random Forest]], [[Gradient Boosting]] — where impurity-based importance is a free training byproduct
- [[Feature Scaling]] — required before linear-coefficient importance means anything
- Feature Selection (a direct downstream use of importance scores — no dedicated note yet in this vault)

## One-line Summary

> Feature importance answers "which features does the model rely on overall" — start with permutation importance for a model-agnostic, performance-grounded answer, and don't trust impurity-based or unscaled-coefficient importance without checking for the biases each carries.
