# Variance

## What is it?

**Variance** measures how much a model's predictions change when trained on different training sets drawn from the same distribution.

$$\text{Var}(\hat{f}(x)) = \mathbb{E}_{\mathcal{D}}\left[\left(\hat{f}(x) - \mathbb{E}_{\mathcal{D}}[\hat{f}(x)]\right)^2\right]$$

High variance = the model is very sensitive to which specific training examples it saw. Small changes in the training data cause large changes in the learned function.

---

## Intuition

A high-variance model "remembers" the training data rather than learning its underlying pattern.

**Example:** A degree-9 polynomial fit to 10 points. Change one point and the entire curve changes drastically. The model's predictions are highly sensitive to the training set.

---

## Variance in the Error Decomposition

$$\mathbb{E}\left[(y - \hat{f}(x))^2\right] = \text{Bias}^2 + \underbrace{\text{Var}(\hat{f}(x))}_{\text{model too complex}} + \sigma_\epsilon^2$$

Variance is the error due to the model being too sensitive to training data. It can be reduced.

---

## Sources of Variance

- Model too complex (high-capacity)
- Too little training data relative to model capacity
- No regularisation
- Noisy features

---

## Reducing Variance

| Method | Effect |
|---|---|
| More training data | Average out idiosyncratic patterns |
| Regularisation (L1/L2) | Forces simpler models |
| Dropout | Neural net ensemble effect |
| Ensemble methods | Average $M$ models: $\text{Var}/M$ |
| Feature selection | Fewer features → less sensitivity |

**Ensemble variance reduction:** If $M$ independent models each have variance $\sigma^2$, their average has variance $\sigma^2/M$:
$$\text{Var}\left(\frac{1}{M}\sum_{m=1}^M \hat{f}_m(x)\right) = \frac{\sigma^2}{M}$$

This is why Random Forests and Boosting work so well.

---

## Connections

- [[Bias]] — the other error component
- [[Bias Variance Tradeoff]] — variance and bias trade off
- [[Overfitting]] — high variance causes overfitting
- [[Regularization]] — reduces variance

---

## One-line Summary

> Variance is the sensitivity of a model's predictions to the specific training data used — high variance means the model overfits, and it is reduced by regularisation, more data, or ensemble methods that average out individual model instability.
