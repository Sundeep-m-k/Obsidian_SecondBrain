# Model Complexity

## What is it?

**Model complexity** refers to the capacity of a model to represent diverse functions — how expressive or flexible the model is. Higher complexity = more functions it can represent.

---

## Measures of Complexity

| Measure | Definition | Example |
|---|---|---|
| **Number of parameters** | Total learnable weights | Linear regression: $d+1$; ResNet-50: 25M |
| **VC dimension** $d_{VC}$ | Max number of points the model can shatter | Linear classifier in $\mathbb{R}^d$: $d+1$ |
| **Rademacher complexity** | Ability to fit random labels | More complex model = higher value |
| **Polynomial degree** | Degree of polynomial features | Degree 1 → line; degree 9 → highly complex curve |
| **Tree depth** | Max depth of a decision tree | Depth 1: one split; unlimited: memorises data |
| **Number of layers/neurons** | NN architecture | Deeper/wider = more complex |

---

## The Complexity-Error Relationship

```
Error
  │
  │ ╲ Bias²          ╱ Variance
  │  ╲              ╱
  │   ╲    Total   ╱
  │    ╲──────────╱
  │     ╲   *    ╱      * = optimal
  └─────────────────── Model Complexity
```

- Low complexity → high bias → underfitting
- High complexity → high variance → overfitting
- Optimal complexity → lowest test error

---

## Regularisation Changes the Effective Complexity

Even a highly parameterised model can have low **effective complexity** with strong regularisation:
- L2: drives weights toward zero → effectively fewer degrees of freedom
- L1: drives some weights to exactly zero → sparse model with fewer active features
- Dropout: equivalent to ensemble of smaller networks

Regularisation shifts the optimal point on the complexity axis without changing the architecture.

---

## Connections

- [[Bias Variance Tradeoff]] — complexity determines bias/variance balance
- [[Overfitting]] — too much complexity
- [[Underfitting]] — too little complexity
- [[Regularization]] — controls effective complexity
- [[Model]] — the architectural choices that determine complexity

---

## One-line Summary

> Model complexity is the expressive capacity of a model — how many different functions it can represent — and finding the right complexity (via architecture choice, regularisation, or both) is the central challenge of model selection.
