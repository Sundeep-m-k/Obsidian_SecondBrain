# Boosting

## What is it?

**Boosting** builds an ensemble **sequentially**: each new model targets the current ensemble's mistakes, unlike [[Bagging]]'s independent, parallel models.

## Algorithm

1. Start with a simple model $F_0$ (often the mean, or a stump).
2. At round $m$: identify where $F_{m-1}$ is wrong (residuals or misclassifications).
3. Train $h_m$ specifically to correct those errors.
4. Update: $F_m(x) = F_{m-1}(x) + \eta \cdot h_m(x)$, where $\eta$ is the **learning rate**.
5. Repeat for $M$ rounds.

## Worked Example: AdaBoost-Style Reweighting

Start with equal weights $w_i = 1/n$ for $n=10$ examples. Round 1's weak learner misclassifies 3 of them. Weighted error $\epsilon_1 = 0.3$, giving the learner's vote weight:

$$\alpha_1 = \frac12 \ln\left(\frac{1-\epsilon_1}{\epsilon_1}\right) = \frac12\ln\left(\frac{0.7}{0.3}\right) = \frac12\ln(2.33) \approx 0.423$$

Misclassified examples get reweighted by $e^{\alpha_1} \approx 1.53$ (correctly-classified ones by $e^{-\alpha_1}\approx 0.65$), then weights renormalize to sum to 1. The next round's weak learner is trained on this reweighted distribution — forced to focus more on the 3 examples round 1 got wrong. Note $\alpha_1$ grows as $\epsilon_1 \to 0$ (a very accurate weak learner gets a big vote) and shrinks toward $0$ as $\epsilon_1 \to 0.5$ (a coin-flip learner gets essentially no vote).

## Bagging vs. Boosting

| | Bagging | Boosting |
|---|---|---|
| Training | Parallel, independent | Sequential, each round depends on the last |
| Fixes | Variance ($\rho\sigma^2$ floor, see [[Ensemble Learning]]) | Bias mainly, often variance too |
| Base learner | Usually deep/strong | Usually shallow/weak (stumps, depth 2-4) |
| Overfitting risk | Low — more trees rarely hurts | Higher — too many rounds can overfit |
| Example | [[Random Forest]] | [[Gradient Boosting]], [[HistGradientBoostingClassifier]] |

## Common Mistakes

- Using too many rounds with too high $\eta$ — overfits, unlike bagging where adding trees is nearly always safe
- Forgetting boosting is inherently sequential — can't parallelize across rounds the way bagging parallelizes across trees
- Skipping early stopping on a validation set — since more rounds = more capacity, unchecked training memorizes noise

## Interview / discussion questions

- Derive $\alpha_1 = \frac12\ln\left(\frac{1-\epsilon}{\epsilon}\right)$'s behavior at $\epsilon \to 0$ and $\epsilon \to 0.5$, and explain why that behavior makes sense for a voting weight.
- Why can boosting overfit with too many rounds, while bagging rarely overfits with more trees?
- Why are boosting's base learners usually shallow trees rather than deep ones?

## Prerequisites

[[Ensemble Learning]], [[Decision Tree]], [[Loss Function]]

## Related concepts

[[Bagging]], [[Gradient Boosting]], [[HistGradientBoostingClassifier]], [[Learning Rate]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/probability

## One-line summary

> Boosting reweights examples round by round ($\alpha_m = \frac12\ln\frac{1-\epsilon_m}{\epsilon_m}$ in the AdaBoost formulation) so each new weak learner focuses on the previous ensemble's mistakes — driving down bias in a way parallel bagging cannot, at the cost of needing careful control over rounds and learning rate.
