# Covariance and Correlation

## What is it?

**Covariance** measures whether two random variables tend to move together (both above their means at the same time, or one above while the other's below).

$$\text{Cov}(X,Y) = \mathbb{E}[(X-\mathbb{E}[X])(Y-\mathbb{E}[Y])] = \mathbb{E}[XY] - \mathbb{E}[X]\mathbb{E}[Y]$$

Positive covariance: $X$ above its mean tends to coincide with $Y$ above its mean. Negative: $X$ above tends to coincide with $Y$ below. Zero: no linear tendency either way.

**Correlation** is covariance rescaled to always fall in $[-1, 1]$, making it interpretable independent of the variables' units:

$$\rho_{X,Y} = \frac{\text{Cov}(X,Y)}{\sigma_X \sigma_Y}$$

---

## Why Correlation Exists at All (Not Just Covariance)

Covariance's magnitude depends entirely on the variables' units — covariance between height-in-centimeters and weight-in-kilograms is a different number than height-in-inches and weight-in-pounds, even though the underlying relationship is identical. This makes raw covariance useless for judging "how strong" a relationship is, or for comparing relationship strength across different variable pairs. Correlation divides out the scale (via each variable's own standard deviation), producing a unit-free number where $\pm 1$ means a perfect linear relationship and $0$ means none — directly comparable across any two variable pairs, in any units.

## Worked Numerical Example

$X = [1,2,3,4,5]$, $Y = [2,4,5,4,5]$.

$\bar{X}=3, \bar{Y}=4$. Deviations: $X-\bar{X} = [-2,-1,0,1,2]$, $Y-\bar{Y}=[-2,0,1,0,1]$.

$$\text{Cov}(X,Y) = \frac{(-2)(-2)+(-1)(0)+(0)(1)+(1)(0)+(2)(1)}{5} = \frac{4+0+0+0+2}{5}=1.2$$

$\sigma_X = \sqrt{2} \approx 1.414$, $\sigma_Y \approx 1.020$. $\rho_{X,Y} = 1.2/(1.414\times1.020) \approx 0.83$ — strong positive linear relationship.

---

## The Critical Limitation: Correlation Only Captures *Linear* Relationships

$\rho=0$ does **not** mean $X$ and $Y$ are unrelated — it means there's no *linear* tendency. Classic example: $Y=X^2$ for $X$ symmetric around 0 (e.g. $X \in \{-2,-1,0,1,2\}$) has $\rho_{X,Y}=0$ exactly, despite $Y$ being *completely determined* by $X$. Always plot the data (a scatterplot), don't trust $\rho$ alone to tell you "no relationship" — it can only rule out a *linear* one.

## Covariance/Correlation vs. Causation

$\rho \ne 0$ never by itself establishes that $X$ causes $Y$ — see [[Correlation vs Causation]] for the full treatment (confounding, reverse causation, selection bias, coincidence). This note covers the *mathematics* of measuring linear association; [[Correlation vs Causation]] covers the *inferential* trap of over-interpreting it.

---

## Relationship to ML/DS

**Multicollinearity** — highly correlated features in a [[Linear Regression]] or [[Regularized Linear Regression]] make individual coefficients unstable and hard to interpret (the model can't distinguish which correlated feature is "really" driving the outcome), even though the *overall* prediction may still be fine — this is why [[L2 Regularization]] is often reached for specifically when features are correlated. **[[PCA]]** is built entirely on the covariance matrix — its principal components are the eigenvectors of the data's covariance matrix, ordered by how much variance (spread) each captures. **Feature selection** — a feature with near-zero correlation to the target is often (not always, per the $Y=X^2$ caveat above) a weak linear predictor and a candidate to drop, though tree-based models can still exploit a non-linear relationship a correlation coefficient would miss entirely.

---

## Interview-Ready Explanation

"Covariance tells you the direction two variables move together, but its magnitude is meaningless on its own because it depends on units. Correlation fixes that by normalizing to $[-1,1]$. The single most important caveat: correlation only detects *linear* relationships — a perfect non-linear relationship like $Y=X^2$ can show zero correlation, so always plot your data rather than trusting the number alone."

---

## Interview Questions

**Why can two variables have zero correlation but still be strongly related?** Correlation measures only *linear* association; a variable related through a non-linear function (e.g. $Y=X^2$ for symmetric $X$) can produce a correlation coefficient of exactly zero while being completely deterministically related — always visualize the data, don't rely on $\rho$ alone.

**Why is correlation used more often than covariance when reporting a relationship's strength?** Covariance's magnitude is tied to the variables' units and is therefore not comparable across different variable pairs or unit systems; correlation normalizes by both standard deviations, producing a unit-free number in $[-1,1]$ that's directly interpretable and comparable.

**How does correlation between features cause a problem in linear regression, and what's a common fix?** Highly correlated (multicollinear) features make individual coefficient estimates unstable and hard to interpret — small data changes can swing a coefficient's sign or magnitude — because the model can't cleanly attribute the outcome to one correlated feature over another; L2 (Ridge) regularization is a common fix, since it shrinks and stabilizes correlated coefficients together rather than picking one arbitrarily.

**Two variables have correlation 0.9 — can you conclude one causes the other?** No — correlation alone never establishes causation; a confounder, reverse causation, selection bias, or coincidence can all produce a strong observed correlation with no direct causal link (see [[Correlation vs Causation]]).

**What is PCA's mathematical relationship to covariance?** PCA finds the eigenvectors of the data's covariance matrix — the directions of maximum variance in the data — and projects onto the top few, which is why features must typically be scaled first (see [[Feature Scaling]]): an unscaled feature's larger raw variance would dominate the covariance matrix and distort which directions PCA identifies as "most important."

## Connections

- [[Correlation vs Causation]] — the inferential trap of over-interpreting correlation
- [[Variance and Standard Deviation]] — covariance generalizes variance to a pair of variables ($\text{Cov}(X,X)=\text{Var}(X)$)
- [[PCA]] — built directly on the covariance matrix's eigendecomposition
- [[Feature Scaling]] — required before covariance/correlation-based methods treat features fairly
- [[L2 Regularization]] — a standard response to multicollinearity among correlated features

## One-line Summary

> Covariance measures the direction two variables move together, correlation normalizes it to a unit-free $[-1,1]$ scale — but both capture only *linear* association, so a strong non-linear relationship can show zero correlation, and neither one, however strong, establishes causation on its own.
