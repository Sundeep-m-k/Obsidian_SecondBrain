# Hyperparameter Search Methods

## What is it?

A **hyperparameter** is a value set before training (learning rate, $k$ in KNN, $C$ in SVM, tree depth) rather than learned from data. **Hyperparameter search** is the process of choosing good hyperparameter values by trying candidates and evaluating each on a validation set — never on the test set, or the final performance estimate becomes optimistic (see [[Data Leakage]]).

---

## Grid Search

Exhaustively try every combination of hyperparameter values from a specified grid.

```python
from sklearn.model_selection import GridSearchCV
param_grid = {"max_depth": [3, 5, 7], "learning_rate": [0.01, 0.1, 0.3]}
grid = GridSearchCV(model, param_grid, cv=5, scoring="f1")
grid.fit(X_train, y_train)
```

**Cost grows exponentially with the number of hyperparameters** — 3 hyperparameters with 5 values each is $5^3=125$ combinations, each evaluated with [[Cross Validation Strategy|k-fold CV]] (multiplying the cost by $k$). Exhaustive and guaranteed to find the best point *on the grid*, but wastes evaluations on combinations unlikely to matter, and doesn't explore between grid points at all.

## Random Search

Sample hyperparameter combinations randomly from specified distributions, for a fixed budget of trials, instead of trying every grid point.

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import uniform, randint
param_dist = {"max_depth": randint(2, 10), "learning_rate": uniform(0.001, 0.3)}
search = RandomizedSearchCV(model, param_dist, n_iter=50, cv=5, scoring="f1")
```

**Why random search often beats grid search at the same budget**: most hyperparameters have unequal importance — often only one or two actually drive performance. Grid search wastes evaluations varying unimportant hyperparameters at fixed values of the important ones; random search's independent sampling means every trial explores a genuinely new value of *every* hyperparameter, so the important ones get effectively explored much more densely for the same total budget.

## Bayesian Optimization

Builds a probabilistic model (typically a Gaussian Process) of "hyperparameters → validation score" from the trials run so far, and uses it to choose the *next* candidate — balancing **exploitation** (try near the best point found so far) against **exploration** (try where uncertainty is highest, in case something better is hiding there). This is the same [[Exploration vs Exploitation]] tradeoff from reinforcement learning, applied to hyperparameter search instead of an RL policy.

```python
from skopt import BayesSearchCV
opt = BayesSearchCV(model, {"max_depth": (2, 10), "learning_rate": (0.001, 0.3, "log-uniform")}, n_iter=30, cv=5)
```

Each trial is *informed* by all previous trials rather than sampled independently, so Bayesian optimization typically finds a good configuration in far fewer trials than random search — valuable when each trial is expensive (a large neural network, hours per training run). The cost is added implementation complexity and the overhead of fitting the surrogate model between trials, which isn't worth it when trials themselves are cheap.

---

## Choosing Among the Three

| Method | Best when | Cost pattern |
|---|---|---|
| Grid Search | Few hyperparameters (1–2), small discrete grids, need exhaustive guarantee on the grid | Exponential in hyperparameter count |
| Random Search | More hyperparameters, cheap trials, unequal hyperparameter importance (the common case) | Fixed budget, scales linearly |
| Bayesian Optimization | Expensive trials (deep learning), budget for only a small number of runs | Fewer trials needed, more overhead per trial |

## Nested Cross-Validation

Tuning hyperparameters using the same validation split repeatedly and then reporting that split's score is itself a mild form of overfitting to the validation set. **Nested CV** — an outer loop for the final performance estimate, an inner loop for hyperparameter selection — avoids this by never letting the outer test fold influence which hyperparameters were chosen. See [[Cross Validation Strategy]] for the full mechanics; reach for nested CV when reporting a final, publication-grade performance number, and plain train/val/test tuning otherwise.

---

## Interview Questions

**Why does random search often outperform grid search at the same compute budget?** Because most problems have a small number of hyperparameters that actually matter; grid search's combinatorial structure wastes trials varying unimportant ones at fixed values of the important ones, while random search's independent sampling explores every hyperparameter's range densely regardless of how many others there are.

**When would you use Bayesian optimization instead of random search?** When each trial is expensive enough (e.g. training a large neural network for hours) that using prior trial results to inform the next candidate — rather than sampling blindly — meaningfully reduces the total number of trials needed to find a good configuration.

**Why can't you tune hyperparameters on the test set?** The test set's entire purpose is an unbiased estimate of generalization; if hyperparameters are chosen based on test performance, the model has effectively "seen" the test set indirectly, and the reported number becomes optimistic — exactly the [[Data Leakage]] failure mode of tuning on data you'll later claim was held out.

## Connections

- [[Cross Validation Strategy]] — the validation mechanism every search method here evaluates candidates with
- [[Data Leakage]] — tuning on the test set is a specific instance of this
- [[Exploration vs Exploitation]] — the same tradeoff Bayesian optimization balances, from a different module
- [[Learning Rate]] — one of the most commonly tuned hyperparameters
- [[Bias Variance Tradeoff]] — most hyperparameters being tuned are, at heart, complexity controls

## One-line Summary

> Grid search is exhaustive but wastes budget on unimportant hyperparameters, random search fixes that by sampling independently for the same budget, and Bayesian optimization uses prior trials to pick the next candidate intelligently — reach for the last one only when trials are too expensive to spend many of them blindly.
