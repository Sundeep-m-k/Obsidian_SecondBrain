# Learning Rate Scheduling

## What is it?

**Learning rate scheduling** changes the [[Learning Rate]] during training rather than holding it fixed — typically starting higher for fast early progress and decreasing it as training proceeds, to allow fine convergence near a minimum without a large fixed step size overshooting it.

---

## Why a Fixed Learning Rate Is a Compromise

A learning rate large enough for fast early progress is often too large to settle precisely into a minimum later in training (it keeps overshooting, oscillating around the minimum rather than reaching it — see [[Convergence]], [[Local Minimum]]). A learning rate small enough for stable late-stage convergence wastes enormous time early in training when much larger steps would make faster progress. Scheduling resolves this by using different rates at different times instead of picking one fixed value.

## Step Decay

Drop the learning rate by a fixed factor at set intervals (e.g. every 10 epochs, or when validation loss plateaus):

$$\eta_t = \eta_0 \cdot \gamma^{\lfloor t/s \rfloor}$$

Simple and effective; the drawback is having to choose the schedule's timing and factor somewhat manually.

## Exponential / Cosine Decay

Smoothly decrease the learning rate every step rather than in discrete jumps:

$$\eta_t = \eta_0 \cdot e^{-kt} \quad \text{(exponential)}, \qquad \eta_t = \eta_0 \cdot \frac{1}{2}\left(1+\cos\frac{\pi t}{T}\right) \quad \text{(cosine)}$$

Cosine decay is common in modern deep learning (including large Transformer pretraining runs) because the smooth, gradual decrease avoids the abrupt jumps of step decay while still driving the rate down to near-zero by the end of training.

## Warmup

Start the learning rate *very small* and increase it over the first several hundred/thousand steps, before switching to a decay schedule. Early in training, weights are near their random [[Weight Initialization]] and gradients can be large and unreliable; a large learning rate applied immediately can push the model into a bad region it never recovers from. Warmup lets the model take small, safe initial steps while gradients are least trustworthy, then ramp up once the loss landscape being navigated is more sensible. **Warmup followed by decay is the standard schedule for training Transformers.**

## Reduce-on-Plateau

Monitor validation loss and reduce the learning rate by a factor whenever it stops improving for a set number of epochs — an adaptive alternative to a fixed schedule, responding to the model's actual training dynamics rather than a pre-committed timetable.

```python
from torch.optim.lr_scheduler import CosineAnnealingLR, ReduceLROnPlateau
scheduler = CosineAnnealingLR(optimizer, T_max=100)
# or:
scheduler = ReduceLROnPlateau(optimizer, mode="min", factor=0.5, patience=5)
```

---

## Interview Questions

**Why not just pick one good learning rate and use it for the whole training run?** The rate that's ideal for fast early progress is usually too large for precise late-stage convergence (it overshoots near the minimum), while a rate small enough for stable convergence wastes time early on — scheduling gets the benefit of both regimes at their respective stages of training.

**What problem does learning-rate warmup solve?** Early in training, weights are close to their random initialization and gradients can be large or unreliable; taking a large step immediately risks pushing the model into a bad region of the loss landscape it can't recover from — warmup takes small, safe steps first, then increases the rate once training has stabilized somewhat.

**Why is warmup + cosine decay the standard schedule for training Transformers specifically?** Transformers are deep, sensitive to early training instability especially with [[Batch and Layer Normalization|LayerNorm]] in the mix, so warmup is important for stability; cosine decay then provides a smooth, well-understood way to anneal the rate down over a long pretraining run without the abrupt jumps of step decay.

## Connections

- [[Learning Rate]] — the quantity being scheduled
- [[Optimizers in Deep Learning]] — schedules are typically applied on top of Adam/SGD, not instead of them
- [[Convergence]] — the goal scheduling is trying to reach more reliably
- [[Weight Initialization]] — why warmup matters most right at the start of training

## One-line Summary

> A fixed learning rate is a compromise between fast early progress and precise late-stage convergence — scheduling (step/exponential/cosine decay, often preceded by warmup) resolves that tension by using a high rate early and a low rate late, with warmup+cosine-decay the standard choice for training Transformers.
