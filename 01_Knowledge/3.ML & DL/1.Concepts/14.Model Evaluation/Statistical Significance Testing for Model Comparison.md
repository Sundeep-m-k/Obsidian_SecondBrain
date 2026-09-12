# Statistical Significance Testing for Model Comparison

## What is it?

When Model B scores higher than Model A on a metric, this note covers how to tell whether that gap is a real effect or noise from the particular sample/split — i.e. how to apply [[Hypothesis Test]] and [[P-Value]] machinery specifically to comparing two models, rather than the general theory of hypothesis testing (which lives in [[Hypothesis Test]], [[P-Value]], and [[Permutation Test]] under Statistics & Experimental Design — read those first if the underlying concepts of $H_0$/$H_1$/Type I/II error/power are unfamiliar).

---

## Common Tests for Comparing Models

**Paired t-test on repeated scores (e.g. per-CV-fold accuracy):**
$$t = \frac{\bar{x}_1 - \bar{x}_2}{SE}$$

```python
from scipy import stats
scores_a = [0.85, 0.87, 0.88, 0.86, 0.84]   # Model A, per fold
scores_b = [0.82, 0.84, 0.83, 0.85, 0.81]   # Model B, per fold
t_stat, p_value = stats.ttest_rel(scores_a, scores_b)  # paired, since same folds
```
Assumes the differences are approximately normal. This is the default choice for comparing two models' cross-validated scores.

**Chi-square test (comparing error rates on categorical outcomes):**
$$\chi^2 = \sum \frac{(O-E)^2}{E}$$
```python
from scipy.stats import chi2_contingency
table = [[45, 5], [8, 42]]  # [correct, wrong] counts per model
chi2, p_value, dof, expected = chi2_contingency(table)
```

**Mann-Whitney U test** — the non-parametric alternative when scores aren't approximately normal or the sample is small; makes no distributional assumption, at some cost in power.
```python
from scipy.stats import mannwhitneyu
u_stat, p_value = mannwhitneyu(scores_a, scores_b, alternative="two-sided")
```

A non-parametric resampling alternative to all of the above, when you don't want to assume a test statistic's distribution at all, is [[Permutation Test]].

---

## Multiple Model Comparisons Need Correction — Single CV Comparisons Don't

**A single paired test on one pair of models' CV scores does not need multiple-testing correction** — it's one question, one test, and cross-validation itself already gives a stability estimate (mean ± std across folds) that guards against a lucky split.

**Comparing many models/variants against a baseline does need it.** Running one test per candidate model and reporting whichever clears $p<0.05$ is exactly the [[Multiple Comparisons Problem]] this vault already covers formally — at 20+ candidates, a spurious "win" becomes likely even if none of them are real improvements. Apply [[Benjamini-Hochberg Procedure]] (or Bonferroni if the candidate count is small and you want to be conservative) to the resulting p-values before trusting any of them, exactly as you would for any other batch of hypothesis tests.

```python
# One comparison — no correction needed
t_stat, p_value = stats.ttest_rel(cv_scores_model, cv_scores_baseline)

# Many comparisons — correction needed
p_values = [stats.ttest_rel(cross_val_score(m, X, y), baseline_scores)[1] for m in candidate_models]
# apply Benjamini-Hochberg to p_values before accepting any "win"
```

---

## Interview Questions

**Model A scores 87%, Model B scores 85% on the same test set — are they different?** Not necessarily — run a paired test on per-fold or per-bootstrap-sample scores rather than comparing two single numbers; report a p-value or, better, a confidence interval on the difference (via [[Confidence Intervals and Bootstrap]]) alongside it. With enough data, a trivially small difference can still be "significant"; always weigh statistical significance against practical significance.

**You test 15 candidate models against a baseline and 2 clear $p<0.05$ uncorrected — do you ship them?** Not without correcting for the fact that 15 tests were run — apply Benjamini-Hochberg (or Bonferroni for a small, high-stakes candidate set) first; at 15 uncorrected tests a false positive is a real possibility, not a corner case.

**Do you need to correct p-values across cross-validation folds of the *same* model?** No — the folds aren't independent hypothesis tests of different claims, they're repeated measurements feeding one paired comparison; correction applies across *different* comparisons/hypotheses, not across folds of the same one.

---

## Connections

- [[Hypothesis Test]], [[P-Value]] — the underlying theory this note applies
- [[Permutation Test]] — non-parametric alternative when no test statistic's distribution can be assumed
- [[Multiple Comparisons Problem]], [[Benjamini-Hochberg Procedure]] — required once more than one comparison is being made
- [[Confidence Intervals and Bootstrap]] — report alongside a p-value, not instead of one
- [[Cross Validation Strategy]] — where the per-fold scores being compared come from

## Prerequisites

[[Hypothesis Test]], [[P-Value]]

## One-line Summary

> Comparing two models statistically means applying ordinary hypothesis-testing machinery (paired t-test, chi-square, Mann-Whitney, or a permutation test) to their per-fold or per-sample scores — a single such comparison needs no multiple-testing correction, but comparing many candidates against a baseline does.
