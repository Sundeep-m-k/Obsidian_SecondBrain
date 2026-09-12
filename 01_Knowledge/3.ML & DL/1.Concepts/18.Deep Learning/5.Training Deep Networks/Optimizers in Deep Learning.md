# Optimizers in Deep Learning

## What is it?

Plain [[Gradient Descent]] takes a step of fixed size in the direction of the negative gradient. **Optimizers** in deep learning are refinements of this update rule that use the *history* of past gradients to take smarter steps — faster convergence, less sensitivity to the exact [[Learning Rate]] chosen, and better handling of the noisy, non-convex loss surfaces deep networks actually have.

---

## SGD with Momentum

Plain [[Batch Gradient Descent]] (or its stochastic mini-batch variant) can zigzag badly in narrow, curved valleys of the loss surface. **Momentum** adds a running average of past gradients, so the update "remembers" its recent direction and dampens oscillation:

$$v_t = \beta v_{t-1} + (1-\beta)\nabla L(\theta_{t-1}), \quad \theta_t = \theta_{t-1} - \eta v_t$$

$\beta \approx 0.9$ is typical. Intuitively: a ball rolling downhill accumulates velocity in the consistent downhill direction and damps out the side-to-side wobble from any one noisy gradient estimate.

## RMSProp

Adapts the learning rate **per parameter**, dividing by a running average of that parameter's squared recent gradients:

$$s_t = \beta s_{t-1} + (1-\beta)(\nabla L)^2, \quad \theta_t = \theta_{t-1} - \frac{\eta}{\sqrt{s_t+\epsilon}}\nabla L$$

Parameters with consistently large gradients get their effective step size shrunk; parameters with small, infrequent gradients get a relatively larger step. This matters a great deal in deep networks, where different layers' parameters can have very different natural gradient scales.

## Adam (Adaptive Moment Estimation)

Combines momentum (tracking the first moment — mean — of gradients) with RMSProp's per-parameter adaptive scaling (tracking the second moment — uncentered variance):

$$m_t = \beta_1 m_{t-1}+(1-\beta_1)\nabla L, \quad v_t = \beta_2 v_{t-1}+(1-\beta_2)(\nabla L)^2$$
$$\hat{m}_t = \frac{m_t}{1-\beta_1^t}, \quad \hat{v}_t = \frac{v_t}{1-\beta_2^t}, \quad \theta_t = \theta_{t-1} - \frac{\eta}{\sqrt{\hat{v}_t}+\epsilon}\hat{m}_t$$

$\hat{m}_t,\hat{v}_t$ are bias-corrected estimates (the raw $m_t,v_t$ are biased toward zero early in training, since they start at zero). **Adam is the default optimizer for most deep learning today** — it combines momentum's smoothing with RMSProp's per-parameter adaptivity, converges quickly, and works reasonably well with minimal tuning, which matters given how many other hyperparameters a deep network already has.

---

## Choosing an Optimizer

| Optimizer | When to reach for it |
|---|---|
| SGD + Momentum | Well-understood, often generalizes slightly better than Adam given enough tuning/training time — common in production computer-vision pipelines |
| RMSProp | Rarely chosen alone today; largely superseded by Adam |
| Adam | The practical default — fast convergence, low tuning burden, standard for Transformers and most modern architectures |

Adam's convenience has a documented cost: it sometimes generalizes slightly worse than well-tuned SGD+momentum on some vision tasks, which is why some production CV pipelines still default to SGD. For most practical purposes (and virtually all NLP/Transformer work), Adam is the reasonable starting choice.

---

## Interview Questions

**Why does momentum help gradient descent converge faster?** It accumulates a running average of past gradient directions, so it keeps moving consistently in directions the gradient has agreed on recently and damps oscillation in directions where the gradient has been noisy or flip-flopping — like a ball's accumulated velocity smoothing out a bumpy downhill path.

**What does Adam combine, and why is it the common default?** Adam combines momentum (a running average of the gradient itself, damping oscillation) with RMSProp's idea of per-parameter adaptive learning rates (dividing by a running average of squared gradients, so parameters with large gradients get smaller effective steps). It converges quickly with relatively little tuning, which is why it's the standard first choice for most deep learning.

**When might you prefer plain SGD with momentum over Adam?** When training time budget allows for more careful tuning and slightly better final generalization matters more than fast, low-effort convergence — some production computer vision pipelines still default to well-tuned SGD+momentum for this reason.

## Connections

- [[Gradient Descent]], [[Learning Rate]] — the foundation every optimizer here refines
- [[Learning Rate Scheduling]] — often combined with these optimizers rather than used as an alternative
- [[Backpropagation]] — supplies the gradients every optimizer here consumes
- [[Convergence]] — what all of these optimizers are trying to reach faster/more reliably

## One-line Summary

> Momentum smooths gradient descent's path by averaging recent gradient directions, RMSProp adapts the learning rate per parameter based on recent gradient magnitude, and Adam combines both — making it the practical default optimizer for most deep learning, with SGD+momentum still preferred in some tuned production vision pipelines.
