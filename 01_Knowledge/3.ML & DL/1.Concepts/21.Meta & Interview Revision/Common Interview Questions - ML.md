This note contains the questions that appear most frequently in ML interviews, with complete self-contained answers.

---

## Foundations

**Q: What is the bias-variance tradeoff?**

The generalisation error of a model decomposes as:
$$\text{Error} = \text{Bias}^2 + \text{Variance} + \text{Irreducible Noise}$$
Bias is the systematic error from a model too simple to capture the true pattern (underfitting). Variance is sensitivity to training data — a model so complex it memorises noise (overfitting). As you increase model complexity, bias decreases and variance increases. The optimal model minimises their sum. See [[Bias Variance Tradeoff]].

---

**Q: What is overfitting and how do you prevent it?**

Overfitting is when training error is low but validation/test error is high — the model has memorised training noise. Detect it by comparing train vs. val error. Fix: regularisation (L1/L2), more data, simpler model, early stopping, dropout, data augmentation. See [[Overfitting]].

---

**Q: What is the difference between L1 and L2 regularisation?**

Both add a penalty to the objective. L2 (Ridge) penalises $\sum\theta_j^2$ — shrinks weights toward zero but never exactly zero; closed-form solution; Gaussian prior. L1 (Lasso) penalises $\sum|\theta_j|$ — drives some weights to exactly zero; automatic feature selection; Laplace prior; no closed form. Use L1 when you want sparsity, L2 as the default. See [[L1 Regularization]], [[L2 Regularization]].

---

**Q: Why can't you use MSE for logistic regression?**

With sigmoid outputs, MSE produces a **non-convex cost surface** — gradient descent can get stuck in local minima. Gradient also vanishes when the model is confidently wrong. Binary cross-entropy is the correct loss: convex, derived from MLE, large gradients when confidently wrong. See [[Logistic Loss]].

---

**Q: What is gradient descent? What are its variants?**

An iterative algorithm: $\theta \leftarrow \theta - \eta\nabla_\theta J(\theta)$. Moves parameters toward steepest descent of the cost. **Batch GD**: exact gradient, slow for large $n$. **SGD**: one example per step, noisy but fast. **Mini-batch**: $b$ examples per step, standard in practice. **Adam**: adaptive learning rates per parameter. See [[Gradient Descent]].

---

**Q: What is cross-validation?**

A technique to estimate generalisation performance. Split training data into $k$ folds; train on $k-1$, validate on 1; rotate; average results. Gives a more reliable estimate than a single train/val split, especially for small datasets. Used for hyperparameter tuning and model selection. See [[Training Data]].

---

## Linear/Logistic Regression

**Q: What are the assumptions of linear regression?**

1. Linearity: $y = \theta^T x + \epsilon$
2. Zero-mean errors: $\mathbb{E}[\epsilon]=0$
3. Homoscedasticity: $\text{Var}(\epsilon^{(i)}) = \sigma^2$ (constant)
4. Independent errors: no correlation between $\epsilon^{(i)}$ and $\epsilon^{(j)}$
5. No perfect multicollinearity (required for $(X^TX)^{-1}$ to exist)
When these hold, OLS is BLUE (Gauss-Markov). See [[Linear Regression]].

---

**Q: What is the normal equation and when do you use it?**

$$\hat{\theta} = (X^TX)^{-1}X^Ty$$
Exact closed-form solution to linear regression. Use when $d$ is small (< ~10,000 features) — $O(d^3)$ complexity makes it infeasible for large $d$. No learning rate needed. See [[Linear Regression]].

---

**Q: How does logistic regression produce probabilities?**

The linear score $z = \theta^T x$ is passed through the sigmoid function $\sigma(z) = \frac{1}{1+e^{-z}} \in (0,1)$. This is the estimated probability of class 1. The decision boundary is where $\sigma(z) = 0.5$, i.e., $z=0$. See [[Logistic Regression]].

---

**Q: What is regularisation and why doesn't it penalise the bias term?**

Regularisation adds a penalty on parameter magnitude to prevent overfitting. The bias term $\theta_0$ is not penalised because it controls the mean of predictions — penalising it would force predictions toward zero regardless of the data's true mean, increasing bias without reducing variance. See [[Regularization]].

---

## Evaluation

**Q: When would you use precision vs. recall?**

**Precision** = of all predicted positives, how many are correct. Use when false positives are costly (e.g., spam filter — don't block legitimate email). **Recall** = of all actual positives, how many did we find. Use when false negatives are costly (e.g., cancer screening — don't miss cancer). $F_1$ = harmonic mean of both. See [[Classification]].

---

**Q: What is AUC-ROC?**

Area under the ROC (Receiver Operating Characteristic) curve. ROC plots TPR vs. FPR at all thresholds. AUC = probability that the model ranks a random positive example higher than a random negative one. AUC=1: perfect. AUC=0.5: random. Threshold-independent — measures ranking ability. See [[Binary Classification]].

---

**Q: Why is accuracy a bad metric for imbalanced data?**

With 99% class 0 and 1% class 1, predicting class 0 always gives 99% accuracy but captures 0% of the minority class. Use precision, recall, F1, or AUC-PR (precision-recall AUC) instead. See [[Binary Classification]].

---

## Advanced

**Q: What is the curse of dimensionality?**

In high dimensions, all pairwise distances become similar (distance concentration), data becomes sparse, and volume grows exponentially. kNN degrades, clustering becomes unreliable, and models need exponentially more data. Solutions: dimensionality reduction (PCA), feature selection, regularisation. See [[Features]].

---

**Q: What is data leakage?**

When information from outside the training set leaks into the model, making it appear to perform better than it actually will in production. Types: using test data to scale features, including features that encode the target, using future information in time-series. Prevention: strict train/val/test separation, temporal ordering in time-series. See [[Training Data]].

---

**Q: Explain gradient vanishing and exploding in neural networks.**

**Vanishing gradients:** In deep networks with sigmoid/tanh activations, gradients $\frac{\partial\mathcal{L}}{\partial W^{(l)}}$ shrink exponentially through layers (backprop multiplies values < 1 repeatedly). Early layers learn very slowly. Fix: ReLU activations, batch normalisation, residual connections.

**Exploding gradients:** Gradients grow exponentially in RNNs or very deep networks. Fix: gradient clipping — if $\|\nabla\| > c$, scale down: $\nabla \leftarrow c\nabla/\|\nabla\|$.

---

**Q: What is the difference between generative and discriminative models?**

**Discriminative:** Directly model $P(Y\mid X)$ — learn the decision boundary. Examples: logistic regression, SVM, neural networks. Better for classification.

**Generative:** Model the joint $P(X,Y)$ or $P(X\mid Y)$ — understand the full data distribution. Examples: Naive Bayes, GMM, VAE, GANs. Can generate new samples; handles missing data.

---

**Q: What is the kernel trick in SVMs?**

Implicitly maps features to a high-dimensional space $\phi(x)$ without explicitly computing $\phi(x)$. The SVM dual only needs inner products $\phi(x_i)^T\phi(x_j) = K(x_i,x_j)$ (the kernel function). Common kernels: RBF: $K(x,x')=\exp(-\|x-x'\|^2/2\sigma^2)$, polynomial, linear. Enables non-linear boundaries with linear-time prediction. See [[Supervised Learning]].

---

## Question Banks for Modules Added 2026-09

This note's original Q&A covers modules 1–9 in depth (the questions above). Rather than duplicate them here, each newer module carries its own "Interview Questions" section directly in its notes and index — go to the source:

- Classical algorithms (SVM/Naive Bayes/KNN/trees/ensembles): [[Classical ML Algorithms Index]], [[How to Choose a Classical ML Algorithm]]
- Evaluation/interpretability/tuning: [[Model Evaluation Index]], [[Model Interpretability Index]], [[Hyperparameter Tuning and Model Selection Index]]
- Deep Learning: [[Deep Learning Index]] and each subfolder's notes
- Applied AI/LLM: [[Applied AI and LLM Systems Index]] and each subfolder's notes
- DS/Analytics/Product: [[Data Science Analytics and Product Thinking Index]]

## Connections

- All concepts in [[Foundations Index]], [[Supervised Learning Index]], [[Linear Regression Index]], [[Loss and Cost Index]], [[Optimization Index]], [[Model Behavior Index]], [[Regularization Index]]
- [[Meta and Interview Revision Index]] — the module index this note belongs to

---

## One-line Summary

> This note is a self-contained interview preparation reference for classical ML foundations (modules 1–9) — every important question answered with the correct formal answer; newer modules (13–20) carry their own interview-question sections directly in their notes rather than duplicating them here.
