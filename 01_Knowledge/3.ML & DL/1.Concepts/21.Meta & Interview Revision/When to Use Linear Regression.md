# When to Use Linear Regression

## Quick Decision

Use **Linear Regression** when:

1. ✅ Your **output is a continuous number** (price, temperature, age, score)
2. ✅ The relationship between features and output is **approximately linear** (or can be made linear with feature engineering)
3. ✅ You need **interpretability** — each coefficient has a clear meaning
4. ✅ Your dataset is **small to medium** size
5. ✅ You want a **fast, simple baseline** before trying complex models

Do NOT use when:
- ❌ Output is categorical → use [[Logistic Regression]] or classifier
- ❌ Relationship is highly non-linear and you can't engineer it away → use trees, NNs
- ❌ Number of features $d \gg$ number of examples $n$ without regularisation → always regularise (Ridge/Lasso)

---

## Decision Flowchart

```
Is your output continuous (real-valued)?
    │
    ├─ NO  → Use Classification (logistic regression, etc.)
    │
    └─ YES
        │
        Is the relationship approximately linear?
        │
        ├─ YES → Linear Regression ✅
        │         (add polynomial features if mildly curved)
        │
        └─ NO  → Do you want interpretability?
                    │
                    ├─ YES → Polynomial Regression / Splines
                    │
                    └─ NO  → Random Forest / Gradient Boosting / Neural Network
```

---

## Checking the Linear Assumption

**Residual plot:** Plot $\hat{y}$ vs. residuals $(y - \hat{y})$.
- Random scatter around zero → linearity holds ✅
- Curved pattern → non-linearity → add polynomial features or use non-linear model ❌
- Funnel shape → heteroscedasticity → transform $y$ (log, sqrt) or use WLS

**Correlation check:** Compute $r(x_j, y)$ for each feature. If $|r| > 0.3$ for the raw feature, linear model can use it directly.

---

## Practical Notes

- **Always scale features** (standardise) when using gradient descent.
- **Check for multicollinearity** (VIF > 10) — use Ridge if present.
- **Handle outliers** before fitting (outliers distort MSE heavily).
- **Report $R^2$ and RMSE** as evaluation metrics — $R^2 > 0.7$ is typically considered a decent fit.
- **Baseline:** Does linear regression beat simply predicting $\bar{y}$? If not ($R^2 \approx 0$), the features have no linear relationship with $y$.

---

## Connections

- [[Linear Regression]] — the algorithm
- [[Regression]] — the task
- [[When to Use Logistic Regression]] — for classification problems
- [[How to Choose Features]] — feature quality matters most

---

## One-line Summary

> Use linear regression when your output is a continuous number and the relationship is approximately linear — it's the fastest, most interpretable baseline and always the right first attempt before more complex models.
