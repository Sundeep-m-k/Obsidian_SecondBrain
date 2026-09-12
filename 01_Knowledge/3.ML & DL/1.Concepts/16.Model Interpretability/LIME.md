# LIME

## What is it?

**LIME (Local Interpretable Model-agnostic Explanations)** explains one prediction by fitting a simple, interpretable model (usually a sparse linear model) to approximate the complex model's behavior *only in the local neighborhood* of that one prediction — the complex model can be a total black box (any classifier or regressor), since LIME never looks inside it, only at its input-output behavior.

---

## How It Works

1. Take the instance $x$ to be explained.
2. Generate perturbed samples around $x$ (e.g. for tabular data, vary feature values; for text, remove words; for images, occlude superpixels).
3. Get the black-box model's prediction for each perturbed sample.
4. Fit a simple, interpretable model (weighted by proximity to $x$ — closer perturbations count more) to these (perturbed sample, black-box prediction) pairs.
5. The simple model's coefficients, valid only in this local neighborhood, are the explanation.

$$\xi(x) = \arg\min_{g \in G} \ \mathcal{L}(f, g, \pi_x) + \Omega(g)$$

$f$ is the black-box model, $g$ is the interpretable surrogate (drawn from a simple model class $G$, e.g. sparse linear models), $\pi_x$ weights samples by proximity to $x$, and $\Omega(g)$ penalizes surrogate complexity (favoring a short, readable explanation — a handful of features, not hundreds).

---

## Why "Local" Is the Whole Point

A complex model's overall decision surface may be wildly non-linear, but *locally*, near one specific point, a linear approximation is often good enough to explain that one prediction — even though the same linear surrogate would be a terrible global summary of the model. This is the same intuition as a first-order Taylor approximation: any smooth function looks approximately linear if you zoom in close enough.

---

## LIME vs. SHAP

LIME is model-agnostic and fast (perturb, fit a cheap local model), but its explanations aren't provably unique or consistent — running it twice on the same prediction can give somewhat different results depending on the random perturbation sample, and there's no guarantee analogous to [[SHAP]]'s local-accuracy property. SHAP is slower (or requires a model-specific fast path like TreeSHAP) but comes with formal guarantees. In practice: reach for LIME for a quick, model-agnostic explanation (especially for image/text models where SHAP's exact variants don't apply cleanly); reach for SHAP when the explanation needs to be defensible or consistent.

```python
import lime.lime_tabular
explainer = lime.lime_tabular.LimeTabularExplainer(X_train.values, feature_names=X_train.columns, class_names=["neg","pos"])
exp = explainer.explain_instance(X_val.iloc[0].values, model.predict_proba, num_features=5)
```

## Common Pitfalls

| Pitfall | Problem | Fix |
|---|---|---|
| Trusting LIME's explanation as globally valid | It's only valid in the local neighborhood of one instance | Never generalize a LIME explanation across the dataset — use [[Feature Importance]] for that |
| Ignoring instability across re-runs | Random perturbation sampling gives slightly different explanations each run | Average over multiple runs, or use a fixed seed when reproducibility matters |
| Using a perturbation strategy that doesn't reflect realistic inputs | Explanation is based on synthetic, off-distribution samples | Check that the perturbation method matches the data type sensibly (word-removal for text, superpixel-occlusion for images) |

## When to Use / When Not To

**Use** as a fast, model-agnostic way to explain individual predictions, especially for image or text models where a fast exact-SHAP variant doesn't exist. **Avoid** when the explanation needs to be provably consistent or defensible in a high-stakes/regulatory context — [[SHAP]]'s guarantees exist precisely for that case.

---

## Interview Questions

**How does LIME explain a black-box model's prediction?** By perturbing the input around that one instance, querying the black box for each perturbation, and fitting a simple, locally-weighted interpretable model to the resulting (perturbation, prediction) pairs — the surrogate's coefficients become the explanation, valid only near that instance.

**Why can't a LIME explanation for one prediction be generalized to describe the whole model?** It's a local linear approximation, valid only in a small neighborhood around the explained instance — the same complex model can behave completely differently a short distance away, which is exactly why the underlying model needed explaining in the first place.

**LIME vs. SHAP — what's the actual tradeoff?** LIME is faster and works on essentially any model or data type but gives no formal consistency guarantee and can vary run-to-run; SHAP is grounded in a unique, provably fair attribution but is slower without a model-specific fast path like TreeSHAP.

## Connections

- [[SHAP]] — the more theoretically grounded alternative for local explanations
- [[Feature Importance]] — the global counterpart; never substitute one LIME explanation for this
- [[Model Interpretability]] (index) — where this fits in the broader toolkit

## One-line Summary

> LIME explains one prediction by fitting a simple, locally-weighted surrogate model around it — fast and model-agnostic, but only valid in that local neighborhood and without SHAP's formal fairness/consistency guarantees.
