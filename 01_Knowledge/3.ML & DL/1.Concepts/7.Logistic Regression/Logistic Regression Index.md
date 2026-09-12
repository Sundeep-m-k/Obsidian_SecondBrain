---
tags: [category/ml-dl, index, moc]
---

# Logistic Regression — Index

> **Position in vault**: `3.ML & DL/1.Concepts/7.Logistic Regression/`
> **Purpose**: The default classification baseline — how the linear-regression template (hypothesis, cost, gradient descent) adapts to predicting a probability instead of a continuous value.
> **Prerequisite**: [[Linear Regression Index]], [[Loss and Cost Index]]

## Section Map

| Note | Covers |
|---|---|
| [[Logistic Regression]] | The model — a linear function squashed through a sigmoid |
| [[Sigmoid Function]] | The squashing function that turns a real number into a probability |
| [[Log Odds]] | The linear quantity logistic regression actually fits |
| [[Probability Output]] | Interpreting the sigmoid's output as $P(y=1\mid x)$ |
| [[Thresholding]] | Turning a probability into a hard class prediction |
| [[Decision Boundary in Logistic Regression]] | Why the boundary is linear in the input space |
| [[Cost Function for Logistic Regression]] | Why MSE is abandoned in favor of log loss here |
| [[Gradient Descent for Logistic Regression]] | The update rule, which turns out to look identical to linear regression's |
| [[Classification Pipeline]] | The end-to-end workflow |

## One-line Summary

> Logistic regression reuses linear regression's entire optimization machinery, swapping only the hypothesis function (sigmoid) and cost function (log loss) — everything else, including the gradient descent update rule's form, carries over unchanged.
