# Environment

## What is it?

The **environment** is everything outside the [[Agent]] in a [[Reinforcement Learning]] system. It receives actions from the agent, transitions to new states, and emits rewards.

$$\text{Environment}: (s_t, a_t) \mapsto (s_{t+1}, r_t)$$

---

## The Environment's Components

**State transition function:**
$$s_{t+1} \sim P(s'\mid s_t, a_t)$$

**Reward function:**
$$r_t = R(s_t, a_t, s_{t+1})$$

These may be deterministic (chess, classic games) or stochastic (real world, noisy sensors).

---

## Fully Observable vs. Partially Observable

| Type | Definition | Formalisation |
|---|---|---|
| **Fully observable (MDP)** | Agent observes the full state $s_t$ | Markov Decision Process |
| **Partially observable (POMDP)** | Agent only observes $o_t$, a noisy function of $s_t$ | Partially Observable MDP |

Real environments are almost always partially observable.

---

## Connections

- [[Reinforcement Learning]] — the framework
- [[Agent]] — interacts with environment
- [[Reward]] — environment-provided signal

---

## One-line Summary

> The environment is everything outside the agent — it receives actions, transitions to a new state according to its dynamics, and emits a reward signal that drives the agent's learning.
