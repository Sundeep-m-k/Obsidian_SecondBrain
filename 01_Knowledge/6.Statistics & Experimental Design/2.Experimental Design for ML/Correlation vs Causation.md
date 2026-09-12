# Correlation vs Causation

## What is it?

**Correlation** measures whether two variables move together statistically. **Causation** means one variable's change actually *produces* a change in the other. Correlation is necessary but nowhere near sufficient evidence for causation — this distinction is one of the most consistently tested concepts in Data Analyst and Data Scientist interviews precisely because it's so easy to violate in real analysis without noticing.

$$\rho_{X,Y} = \frac{\text{Cov}(X,Y)}{\sigma_X\sigma_Y}$$

A high $|\rho|$ says $X$ and $Y$ tend to move together; it says nothing at all about *why*.

---

## Why Correlation Can Appear Without Causation

**Confounding variable** — a third variable causes both $X$ and $Y$, creating a correlation between them with no direct causal link at all. Classic example: ice cream sales and drowning deaths are correlated — both are driven by hot weather (more swimming *and* more ice cream), not by ice cream causing drowning.

**Reverse causation** — the assumed direction is backwards: $Y$ actually causes $X$, not the other way around. A correlation between "using a premium feature" and "high customer satisfaction" might mean satisfied customers seek out premium features, not that the feature causes satisfaction.

**Selection bias** — the observed sample isn't representative, creating a spurious relationship that wouldn't hold in the full population (see [[Data Leakage]] and [[Grouped Train-Test Split]] for related sampling-integrity issues in an ML context specifically).

**Coincidence (spurious correlation)** — with enough variables compared against each other, some will correlate strongly purely by chance — the same underlying issue as the [[Multiple Comparisons Problem]], applied to correlation-hunting instead of hypothesis-testing.

---

## What Actually Establishes Causation

**Randomized controlled experiments** ([[A-B Testing|A/B testing]]) are the gold standard — random assignment to treatment/control breaks any link between the treatment and pre-existing confounders by construction, since both groups are statistically identical in expectation on every other dimension. This is *why* A/B testing is preferred over purely observational analysis whenever a causal claim actually matters for a decision.

**When randomization isn't possible or ethical** (e.g. analyzing historical, observational data), quasi-experimental methods exist — difference-in-differences, instrumental variables, regression discontinuity — each attempting to approximate randomization's confounder-breaking effect using structural features of the data, at the cost of resting on assumptions that must be argued for, not assumed automatically true the way randomization's guarantee is.

---

## Interview Questions

**Ice cream sales correlate strongly with drowning deaths — does ice cream cause drowning?** No — both are driven by a confounding variable, hot weather, which independently increases both swimming (and therefore drowning risk) and ice cream consumption; there's no direct causal link between the two observed variables at all.

**A company observes that users who use a specific feature have higher retention — should they invest in pushing that feature to everyone?** Not based on this correlation alone — it's plausible that already-more-engaged users (who would have retained anyway) are the ones choosing to use the feature (reverse causation/self-selection), rather than the feature itself driving retention; a randomized test that assigns the feature to a random subset of users, rather than let them self-select, is needed to establish the causal claim before investing based on it.

**Why is a randomized experiment considered stronger causal evidence than a well-controlled observational study?** Randomization guarantees, by construction, that treatment and control groups are statistically identical in expectation on *every* dimension — including confounders the analyst never thought to measure or control for — whereas an observational study can only control for confounders it explicitly identifies and measures, leaving unmeasured confounders as an unaddressed risk to the causal claim.

## Connections

- [[Covariance and Correlation]] — the mathematics of measuring correlation this note's inferential warnings apply to
- [[A-B Testing]] — the standard way to actually establish causation when it matters for a decision
- [[Hypothesis Test]] — correlation itself is typically tested for statistical significance using this machinery
- [[Multiple Comparisons Problem]] — spurious correlation from comparing many variable pairs is the same underlying issue
- [[Data Leakage]] — selection-bias-driven spurious relationships in an ML-specific context

## One-line Summary

> A correlation between two variables can arise from a real causal link, a shared confounder, reverse causation, selection bias, or pure chance — only a randomized experiment (or a carefully-justified quasi-experimental method) can distinguish which, since randomization is what breaks the link to unmeasured confounders that observational analysis alone cannot rule out.
