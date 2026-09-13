---
tags: [category/ml-dl, index, moc]
---

# Linear Regression — Index

> **Position in vault**: `3.ML & DL/1.Concepts/3.Linear Regression/`
> **Purpose**: The first concrete model — everything here (hypothesis function, parameters, prediction) is the template every later model (logistic regression, neural networks) reuses with a different function shape.
> **Prerequisite**: [[Supervised Learning Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Linear Regression]] | The model itself — fitting a linear function to continuous targets |
| [[Model Representation]] | How a model is written as a parameterized function |
| [[Hypothesis Function]] | The function form being fit |
| [[Prediction Function]] | Using fitted parameters to predict on new inputs |
| [[Parameters]] | What's actually being learned |
| [[Multiple Linear Regression]] | Extending to more than one feature |
| [[Polynomial Regression]] | Fitting non-linear relationships with a linear model over transformed features |
| [[Regression Pipeline]] | The end-to-end workflow — from raw data to prediction |
| [[Regression Diagnostics]] | Detecting and fixing violations of the Gauss-Markov assumptions (linearity, homoscedasticity, normality, multicollinearity, influential points) |

## One-line Summary

> Linear regression is the simplest instance of "define a hypothesis function, define a cost, minimize it" — the pattern every model in modules 4, 5, 7, and 18 repeats with different pieces.
