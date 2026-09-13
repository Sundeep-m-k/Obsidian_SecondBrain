# Regression Diagnostics

## What is it?

[[Linear Regression]]'s Gauss-Markov assumptions are usually just *stated*. This note covers what to actually do when they might be violated: how to **detect** each violation, what it **looks like**, why it **matters**, and what's a reasonable **remediation** — the difference between listing assumptions and being able to diagnose a real dataset.

**One distinction to hold onto throughout**: whether an assumption violation matters depends on whether the goal is **prediction** or **statistical inference**. A model used purely to predict can tolerate some assumption violations that would invalidate a p-value or confidence interval computed from the same fit — this note flags exactly where that distinction changes the answer.

---

## Linearity

**How to detect**: plot residuals ($y - \hat{y}$) against fitted values ($\hat{y}$) or against each predictor. A well-specified linear model shows no pattern — just random scatter around zero.

**What failure looks like**: systematic curvature in the residual plot (a U-shape, an S-shape) — the model is missing a non-linear relationship the straight-line fit can't capture.

**Why it matters**: for both prediction and inference — a genuinely curved relationship fit with a straight line is systematically wrong in predictable regions (over-predicting in some ranges, under-predicting in others), not just noisy.

**Remediation**: add polynomial terms ([[Polynomial Regression]]), transform a predictor (log, square root) if theory or the residual shape suggests a specific functional form, or switch to a non-linear model entirely if the relationship is fundamentally not well-approximated by any transformation of a linear form.

## Independence

**How to detect**: for time-ordered or spatially-ordered data, plot residuals in that order and look for runs of same-sign residuals (a positive residual tends to be followed by another positive one); a formal test (e.g. Durbin-Watson for time series) is available.

**What failure looks like**: residuals that are correlated with each other rather than independent — common in time series (today's error predicts tomorrow's) or clustered/grouped data (repeated measurements on the same subject).

**Why it matters mainly for inference, not raw prediction**: correlated errors don't necessarily bias the coefficient estimates themselves, but they make the standard errors (and therefore p-values and confidence intervals) **wrong — typically too small**, making effects look more statistically significant than they actually are. See [[Rolling-Origin Validation]] and [[Grouped Train-Test Split]] for the closely related — but distinct — problem of independence violations breaking *evaluation* (not just inference), which is a separate failure mode from what this section addresses.

**Remediation**: use methods designed for correlated errors (generalized least squares, clustered/robust standard errors, or a time-series-appropriate model per [[Time Series Fundamentals]]) rather than plain OLS's standard errors.

## Homoscedasticity (Constant Variance)

**How to detect**: the same residuals-vs-fitted-values plot used for linearity — here, look at the *spread* of residuals across the range of fitted values, not their central pattern.

**What failure looks like** (**heteroscedasticity**): a funnel or cone shape — residual spread widens (or narrows) systematically as fitted values increase, rather than staying constant.

**Why it matters mainly for inference**: OLS coefficient estimates remain unbiased under heteroscedasticity, but the standard errors are wrong, again typically distorting p-values and confidence intervals — prediction *point estimates* aren't directly invalidated, but any prediction interval built assuming constant variance will be too narrow in the high-variance region and too wide in the low-variance region.

**Remediation**: a variance-stabilizing transformation of the target (log, Box-Cox), weighted least squares (explicitly down-weighting high-variance observations), or heteroscedasticity-robust standard errors that don't assume constant variance in the first place.

## Residual Normality

**How to detect**: a Q-Q (quantile-quantile) plot of residuals against a theoretical normal distribution — points falling on the diagonal line indicate normality; systematic deviation (especially in the tails) indicates non-normal residuals.

**What failure looks like**: points curving away from the diagonal at one or both ends — heavy tails (more extreme residuals than a normal distribution would produce) or skew.

**Why it matters only for inference, not prediction — the sharpest prediction-vs-inference distinction in this note**: OLS's coefficient estimates and point predictions don't require normal residuals to be valid — only [[Central Limit Theorem|large-sample behavior]] and correctly-specified other assumptions. Normality specifically matters for **exact, small-sample** hypothesis tests and confidence intervals on the coefficients — with a reasonably large sample, the CLT means those inferential procedures are approximately valid even without normal residuals. **A model built purely to predict, evaluated by held-out error rather than by trusting a coefficient's p-value, can have decidedly non-normal residuals and still be a perfectly good predictive model** — this is the single most commonly conflated point in this entire note.

**Remediation**: for small samples where exact inference is needed, a target transformation (log, Box-Cox) or a non-parametric/bootstrap-based inference method (see [[Confidence Intervals and Bootstrap]]) that doesn't assume normality at all.

## Multicollinearity

**How to detect**: the **Variance Inflation Factor (VIF)** for each predictor — regress that predictor on all the others and compute $VIF_j = \frac{1}{1-R_j^2}$, where $R_j^2$ is that regression's $R^2$. $VIF_j > 5$–$10$ (conventions vary) flags a predictor that's substantially explainable by the others. See [[Covariance and Correlation]] for the simpler pairwise-correlation check this generalizes to more than two variables at once.

**What failure looks like**: coefficient estimates that are unstable (large standard errors, signs that flip with small data changes, wildly different values across near-identical model refits).

**Why it matters mainly for inference and interpretation, not raw prediction accuracy**: multicollinearity doesn't necessarily hurt overall predictive accuracy — the model can still fit the data well — but it makes individual coefficients unreliable and hard to interpret, since the model can't cleanly attribute the outcome to one correlated predictor over another.

**Remediation**: drop or combine redundant predictors, use [[L2 Regularization|Ridge regression]] (which explicitly handles correlated predictors by shrinking their coefficients together rather than arbitrarily favoring one), or use [[PCA]] to reduce correlated predictors to uncorrelated components first.

## Influential Observations and Outliers

**How to detect**: **leverage** (how unusual a point's *predictor* values are, independent of its target value) identifies points with the potential to disproportionately influence the fit; **Cook's distance** combines leverage with the actual residual size to measure how much the fitted model would change if that one point were removed — a large Cook's distance flags a point that's *actually* exerting outsized influence, not just sitting in an unusual location.

**What failure looks like**: one or a few points with high Cook's distance — removing them would noticeably change the fitted coefficients, meaning the reported model is substantially shaped by a handful of observations rather than the bulk of the data.

**Why it matters for both prediction and inference**: an overly-influential point can pull the fitted line toward itself, degrading both the fit's genuine predictive accuracy on typical points and the validity of any inference drawn from the fitted coefficients.

**Remediation**: investigate *why* the point is influential first (a data-entry error should be fixed or removed; a genuine, rare-but-real extreme case may need to stay, with the analysis explicitly noting sensitivity to it) — never remove an influential point purely because it's inconvenient without first checking whether it's a legitimate part of the data-generating process.

---

## Worked Diagnostic Case

**Scenario**: a linear regression predicting house price from square footage. The residual-vs-fitted plot shows: **(1)** visible curvature (residuals dip negative in the middle range of fitted values and rise at both ends), **(2)** variance clearly increasing with fitted value (a funnel shape — small houses' predictions are tightly clustered, large houses' predictions are widely scattered), and **(3)** one point with a Cook's distance far larger than every other observation — a mansion with an unusually large lot size that also happens to be the single most expensive property in the dataset.

**What to investigate**: the curvature suggests the price-vs-square-footage relationship isn't actually linear across the full range — plausible given that price-per-square-foot often *decreases* at very large sizes (diminishing returns) — worth checking against domain knowledge, not just the plot. The increasing variance is a common and expected pattern for price data specifically (absolute price variability naturally scales with price level) rather than a surprising anomaly. The high-Cook's-distance mansion needs a targeted look: is its square footage or price value actually correct, or is it a data-entry error? If it's a legitimate, rare, high-end property, its outsized influence on a model meant to predict *typical* homes' prices is itself the problem, independent of any data error.

**Reasonable fixes, not automatic magic ones**: a log transformation of price is a standard, well-motivated response to *both* the curvature (log often linearizes a diminishing-returns relationship) and the increasing variance (log compresses the scale where variance was growing) simultaneously — but it should be checked against the residual plots again *after* transforming, not assumed to fix everything just because it's the standard move. The high-leverage mansion should be investigated on its own terms (verify the data, then make an explicit, documented decision about whether to keep, downweight, or model it separately) rather than silently dropped because the model looks better without it — dropping an inconvenient point without justification is itself a form of [[Correlation vs Causation|selection bias]] introduced into the analysis after the fact.

---

## Interview Questions

**A residual plot shows a funnel shape — what does that suggest, and does it invalidate your model's predictions?** Heteroscedasticity — non-constant error variance across the range of fitted values. It doesn't bias the point predictions or coefficient estimates themselves, but it means the standard errors (and any p-values/confidence intervals built from them) are unreliable, and any prediction interval assuming constant variance will be miscalibrated across the range.

**Why do non-normal residuals matter less if the model is being used purely for prediction rather than statistical inference?** OLS point predictions and coefficient estimates don't require normal residuals to be valid; normality specifically matters for exact small-sample hypothesis tests and confidence intervals on the coefficients — with a large-enough sample, the Central Limit Theorem makes those inferential procedures approximately valid regardless, and a model evaluated purely by held-out predictive error never relied on the normality assumption in the first place.

**What does a high VIF actually tell you, and what would you do about it?** That a predictor is substantially explainable by a linear combination of the other predictors (multicollinearity) — it doesn't necessarily hurt overall predictive accuracy, but it makes that predictor's individual coefficient unstable and hard to interpret; fixes include dropping/combining redundant predictors, Ridge regression, or PCA.

**What's the difference between leverage and Cook's distance?** Leverage measures how unusual a point's predictor values are, on their own, independent of its actual target value; Cook's distance combines leverage with the residual size to measure how much the fitted model would actually change if that point were removed — a point can have high leverage without being influential (if its target value fits the trend well), and Cook's distance is what actually confirms influence.

**Should you always drop an influential outlier once you find one?** No — investigate why it's influential first. A data-entry error should be fixed or removed; a genuine, rare extreme case is real information about the data-generating process and dropping it purely because it's inconvenient (without documented justification) introduces a form of selection bias into the analysis.

## Connections

- [[Linear Regression]] — the Gauss-Markov assumptions this note diagnoses
- [[Covariance and Correlation]] — the pairwise version of the multicollinearity check VIF generalizes
- [[L2 Regularization]], [[PCA]] — standard remediations for multicollinearity
- [[Confidence Intervals and Bootstrap]] — a normality-free alternative for inference when residuals are non-normal
- [[Central Limit Theorem]] — why normality matters less at large sample sizes for inference
- [[Rolling-Origin Validation]], [[Grouped Train-Test Split]] — the evaluation-side (not inference-side) consequence of independence violations
- [[Time Series Fundamentals]] — the standard home for genuinely autocorrelated data, rather than patching OLS's independence assumption after the fact

## One-line Summary

> Diagnose linearity and homoscedasticity via a residuals-vs-fitted plot, normality via a Q-Q plot, multicollinearity via VIF, and influence via Cook's distance — and before applying any fix, know whether the goal is prediction (many violations matter less) or statistical inference (most of them directly invalidate a p-value or confidence interval), since that answer changes what "fixed" even means.
