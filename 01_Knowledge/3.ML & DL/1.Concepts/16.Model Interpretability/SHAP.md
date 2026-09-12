# SHAP

## What is it?

**SHAP (SHapley Additive exPlanations)** explains one individual prediction by fairly distributing the credit for it among the input features, using **Shapley values** from cooperative game theory — each feature is treated as a "player" contributing to the "payout" (the prediction), and Shapley values are the unique way to split that payout fairly given each feature's marginal contribution across every possible subset of the other features.

$$f(x) = \phi_0 + \sum_{j=1}^{d} \phi_j$$

$\phi_0$ is the average prediction over the whole dataset (the baseline); each $\phi_j$ is feature $j$'s contribution — how much it pushed this specific prediction above or below that baseline.

---

## Why This Is Different From Feature Importance

[[Feature Importance]] answers "which features matter overall, on average." SHAP answers "why did the model predict *this* value for *this* example" — a local, per-prediction explanation rather than a global summary. A feature can have low global importance but a large SHAP value for one particular unusual example, and vice versa.

## The Core Idea: Marginal Contribution Across Coalitions

For feature $j$, its Shapley value averages its marginal contribution across every possible subset ("coalition") of the other features:

$$\phi_j = \sum_{S \subseteq F\setminus\{j\}} \frac{|S|!(|F|-|S|-1)!}{|F|!}\big[f(S\cup\{j\}) - f(S)\big]$$

This is expensive to compute exactly ($2^{|F|}$ subsets), which is why practical SHAP implementations use approximations tuned to the model type — **TreeSHAP** computes exact Shapley values efficiently for tree-based models ([[Random Forest]], [[Gradient Boosting]]) by exploiting tree structure; **KernelSHAP** is a slower, model-agnostic approximation that works for any model, including neural networks.

```python
import shap
explainer = shap.TreeExplainer(model)     # fast, exact, for tree models
shap_values = explainer.shap_values(X_val)
shap.summary_plot(shap_values, X_val)      # global view: distribution of every feature's SHAP values
shap.force_plot(explainer.expected_value, shap_values[0], X_val.iloc[0])  # one prediction, explained
```

## Properties That Make It Trustworthy

SHAP values satisfy **local accuracy** (they sum exactly to the actual prediction minus the baseline — no unexplained residual), **consistency** (if a model changes so that a feature's marginal contribution never decreases, that feature's SHAP value never decreases either), and **missingness** (a feature that isn't used gets a SHAP value of exactly zero). These guarantees are what distinguish SHAP from ad hoc explanation heuristics — they're provably the *unique* attribution method satisfying all three simultaneously.

---

## SHAP vs. LIME

Both produce local, per-prediction explanations, but SHAP is grounded in a game-theoretic guarantee of fairness and consistency; [[LIME]] fits a cheap local surrogate model and has no such guarantee, trading theoretical rigor for speed and simplicity. In practice, SHAP is preferred when the explanation needs to be defensible (regulatory, high-stakes decisions); LIME is preferred when a fast, rough explanation is enough and the model type isn't well-supported by an efficient SHAP variant.

## When to Use / When Not To

**Use** SHAP when a *specific* prediction needs to be explained and defended — why was this loan denied, why was this transaction flagged — especially with tree-based models where TreeSHAP is fast and exact. **Avoid** it as a first pass on a large dataset (start with global [[Feature Importance]] instead) or when compute budget can't absorb KernelSHAP's cost on a non-tree model.

---

## Interview Questions

**What guarantee does SHAP provide that ad hoc feature-attribution methods don't?** Local accuracy — the SHAP values for one prediction sum exactly to that prediction minus the baseline, so nothing is left unexplained — plus consistency and missingness, properties that Shapley values are provably the unique attribution satisfying together.

**Why is exact SHAP computation intractable in general, and how is it made practical?** The exact formula sums over all $2^{|F|}$ feature subsets; TreeSHAP exploits tree structure to compute exact values efficiently for tree-based models, while KernelSHAP approximates the computation for arbitrary models at higher cost.

**A stakeholder asks why the model denied a specific loan application — what tool do you reach for and why?** SHAP's force plot for that one example — it gives a mathematically grounded, per-feature breakdown of exactly how much each input pushed the prediction away from the baseline, which is what a defensible individual explanation requires.

## Connections

- [[Feature Importance]] — the global counterpart this note complements
- [[LIME]] — the faster, less theoretically grounded alternative for local explanations
- [[Random Forest]], [[Gradient Boosting]] — where TreeSHAP makes exact computation fast
- [[Model Interpretability Index]] — where this sits in the broader interpretability toolkit

## One-line Summary

> SHAP explains one prediction by fairly splitting credit among features using game-theoretically unique Shapley values that provably sum to the actual prediction — use TreeSHAP for fast, exact explanations on tree models, and KernelSHAP as a slower model-agnostic fallback.
