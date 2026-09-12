# Gradient Boosting

## What is it?

**Gradient Boosting** is [[Boosting]] where each new model is trained to predict the **negative gradient** of the loss w.r.t. the current ensemble's predictions — for squared error, that gradient is just the residual, so each tree learns to predict "how wrong am I still."

## Algorithm

1. Initialize $F_0(x) = \arg\min_c \sum_i \mathcal{L}(y_i, c)$ (e.g. the mean, for squared error).
2. For $m=1$ to $M$: compute pseudo-residuals $r_i = -\left[\frac{\partial \mathcal{L}(y_i, F(x_i))}{\partial F(x_i)}\right]_{F=F_{m-1}}$.
3. Fit a shallow tree $h_m$ to predict $r_i$ from $x_i$.
4. Update: $F_m(x) = F_{m-1}(x) + \eta \cdot h_m(x)$.

For squared-error loss $\mathcal{L}=\frac12(y-F)^2$: $r_i = -\frac{\partial}{\partial F}\frac12(y_i-F)^2 = y_i - F_{m-1}(x_i)$ — literally the residual. For log-loss (binary classification), $r_i = y_i - \sigma(F_{m-1}(x_i))$ — the difference between the true label and the current predicted probability.

This is **gradient descent in function space**: instead of updating parameters $\theta$ (as in [[Gradient Descent]]), each step adds a whole new function (tree) pointing in the loss-reducing direction.

## Worked Example: One Round, Squared Error

3 examples with $y = [10, 15, 8]$, $F_0(x) = \bar y = 11$ for all (mean). Pseudo-residuals: $r = [10-11, 15-11, 8-11] = [-1, 4, -3]$. A shallow tree $h_1$ is fit to predict these residuals from features; say it predicts $\hat r = [-0.8, 3.5, -2.9]$ (imperfect, since it's shallow). With $\eta=0.1$:

$$F_1(x) = F_0(x) + 0.1 \cdot h_1(x) = [11-0.08,\ 11+0.35,\ 11-0.29] = [10.92,\ 11.35,\ 10.71]$$

Errors shrink from $[1, 4, 3]$ (round 0) toward $[0.92, 3.65, 2.71]$ (round 1) — slow, deliberate progress by design; $\eta=0.1$ trades speed for generalization, requiring many more rounds to fully fit but avoiding overfitting any single round's noisy residual estimate.

## Why It Matters

Gradient Boosting (and fast implementations — [[HistGradientBoostingClassifier]], [[Gradient Boosting Libraries (XGBoost & LightGBM)]]) is usually the strongest off-the-shelf method for tabular data — the benchmark any custom approach has to beat.

## Key Hyperparameters

| Hyperparameter | Effect |
|---|---|
| Number of trees (rounds $M$) | More = more capacity, risk of overfitting without early stopping |
| Learning rate $\eta$ | Smaller = more conservative, usually better generalization, needs more rounds |
| Max tree depth | Controls how much interaction each tree captures |
| Subsampling (rows/columns per round) | Adds bagging-style randomness, reduces overfitting |

## Advantages

Typically best accuracy on tabular data; handles mixed types, non-linearities, interactions automatically; feature importance available (not causal).

## Limitations

More hyperparameters than [[Random Forest]], easier to overfit if mistuned; sequential training limits parallelism across rounds; poor at extrapolating beyond training range; needs validation-based early stopping, never test-set tuning.

## Common Mistakes

- Confusing "reduces bias" with "can't overfit" — it absolutely can, especially with too many rounds and high $\eta$
- Not using early stopping on a validation set to pick $M$
- Treating output probabilities as calibrated without checking — see [[Calibration]], [[Brier Score]]

## Interview / discussion questions

- Derive the pseudo-residual formula for squared-error loss and for log-loss.
- Why is gradient boosting described as "gradient descent in function space"?
- How do learning rate and number of rounds trade off, and how would you choose both properly?

## Prerequisites

[[Boosting]], [[Gradient Descent]], [[Loss Function]], [[Decision Tree]]

## Related concepts

[[HistGradientBoostingClassifier]], [[Gradient Boosting Libraries (XGBoost & LightGBM)]], [[Bagging]], [[Random Forest]]

## Tags

#category/ml-dl #topic/tree-based-methods #math/calculus

## One-line summary

> Gradient Boosting fits each new shallow tree to the negative gradient of the loss ($r_i = y_i - F_{m-1}(x_i)$ for squared error) and adds it in with a small learning rate — literally gradient descent performed in function space, tree by tree.
