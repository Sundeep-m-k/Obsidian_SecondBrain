# Topic 17: Statistical Validation — Complete Index

## Overview

Statistical validation ensures estimates are reliable and conclusions are sound. Three focused files cover confidence intervals, hypothesis testing, and multiple testing correction.

**Total Content:** 3 files, ~15 KB | 20+ Questions | 15+ Code Examples

---

## Files at a Glance

| File | Focus | Key Concepts |
|------|-------|--------------|
| [[17.1_Confidence_Intervals_and_Bootstrap]] | CI construction, bootstrap resampling, interpretation | Uncertainty quantification |
| [[17.2_Hypothesis_Testing_and_P_Values]] | T-tests, p-value interpretation, Type I/II errors, power | Statistical significance |
| [[17.3_Multiple_Testing_Correction]] | Bonferroni, Benjamini-Hochberg FDR, when to apply | False positive inflation |

---

## Quick Reference

**Confidence Intervals:**
- Bootstrap: Resample with replacement; use percentiles as CI bounds
- Interpret: "If repeated 100 times, true value in range ~95 times" (not "95% probability")
- Python: `np.percentile(bootstrap_samples, [2.5, 97.5])`

**Hypothesis Testing:**
- Null ($H_0$): No difference / No effect
- Alternative ($H_1$): There is a difference / There is an effect
- P-value: Probability of data if $H_0$ true
- Decision: p < 0.05 → reject $H_0$

**Multiple Testing:**
- Bonferroni: $\alpha_{\text{adj}} = \alpha / m$ (strict)
- Benjamini-Hochberg: FDR control (less strict)
- 100 tests with α=0.05 → ~99% false positive rate without correction

---

## Interview Scenarios

### Scenario 1: "Model A has 85% accuracy, Model B has 87%. Are they significantly different?"

Use t-test on cross-validated scores:
```python
scores_a = cross_val_score(ModelA, X, y, cv=5)
scores_b = cross_val_score(ModelB, X, y, cv=5)
t_stat, p_value = ttest_rel(scores_a, scores_b)

# If p < 0.05: Significantly different
# Also report 95% CI on difference
```

See [[17.2_Hypothesis_Testing_and_P_Values]], [[17.1_Confidence_Intervals_and_Bootstrap]]

### Scenario 2: "You selected 50 significant features from 1000. How many are false positives?"

Without correction: ~50 false positives expected (50 × 0.05).
With Benjamini-Hochberg FDR: Reduced to ~2-3 false positives (FDR = 0.05).
Validate selected features on held-out test set.

See [[17.3_Multiple_Testing_Correction]]

---

## Study Order

**1-hour:** 17.1 + 17.2
**2-hour:** All 3 files
**Practice:** 5-10 Q&A per file

---

## One-line Summaries

| File | Summary |
|------|---------|
| 17.1 | CI quantifies uncertainty; bootstrap resamples to estimate without assumptions; interpret as repeated sampling |
| 17.2 | P-value = probability of data if null true; p<0.05 rejects null; power = ability to detect real effects |
| 17.3 | Multiple tests inflate false positives; correct with Bonferroni (strict) or FDR (balanced); CV doesn't need correction |

---

**Total Study Time:** 2-3 hours (deep dive)
**Interview Readiness:** 85% after 2 files, 95% after all 3
