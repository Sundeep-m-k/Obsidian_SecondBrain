# Reward

## What is it?

The **reward** $r_t$ is a scalar signal the [[Environment]] emits at each time step, telling the [[Agent]] how good or bad its last action was. It is the **only training signal** in [[Reinforcement Learning]].

$$r_t = R(s_t, a_t, s_{t+1}) \in \mathbb{R}$$

The agent's entire goal is to maximise the expected **cumulative** reward over time, not just the immediate reward.

---

## Cumulative Reward (Return)

The **return** at time $t$ is the discounted sum of all future rewards:

$$G_t = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots = \sum_{k=0}^{\infty}\gamma^k r_{t+k}$$

**Discount factor $\gamma \in [0,1)$:**
- $\gamma = 0$: only immediate reward matters (myopic)
- $\gamma = 1$: all future rewards count equally (infinite horizon, may diverge)
- $\gamma = 0.99$: typical — cares about the future but prefers sooner rewards

**Why discount?**
- Mathematical convenience (ensures finite sum)
- Economic interpretation: future rewards are worth less (uncertainty, time value)
- Practical: incentivises agent to solve tasks quickly

---

## Recursive Relationship

$$G_t = r_t + \gamma G_{t+1}$$

This is the Bellman equation's basis — allows dynamic programming.

---

## Reward Design: The Hard Part

The reward function must perfectly capture the designer's intent. Poor reward design leads to **reward hacking** — the agent finds unexpected ways to maximise the reward metric without achieving the actual goal.

**Famous examples of reward hacking:**
- Boat racing game: agent found it could score more points by spinning in circles collecting power-ups than actually racing.
- Robot hand: learned to flip itself over to get closer to the target without actually grasping.

**Sparse vs. dense rewards:**
| Type | Definition | Problem |
|---|---|---|
| **Sparse** | Reward only at success (e.g., +1 win, 0 otherwise) | Hard to learn — agent rarely sees any reward |
| **Dense** | Reward at every step (e.g., distance to goal) | Risk of reward hacking; requires manual shaping |

**Reward shaping:** Add intermediate rewards to guide learning:
$$R_{\text{shaped}}(s,a,s') = R(s,a,s') + \gamma\Phi(s') - \Phi(s)$$

where $\Phi(s)$ is a potential function. **Potential-based shaping** is guaranteed not to change the optimal policy (Ng et al., 1999).

---

## RLHF (Reward from Human Feedback)

For complex tasks where the reward is hard to specify (like "write a helpful response"), train a **reward model** $R_\phi(s,a)$ from human preference comparisons:

1. Collect pairs of responses $(y_1, y_2)$ to the same prompt.
2. Have humans label which is better.
3. Train $R_\phi$ to predict human preferences: $P(y_1 \succ y_2) = \sigma(R_\phi(y_1) - R_\phi(y_2))$.
4. Use $R_\phi$ as the reward signal for RL training (PPO).

This is the foundation of ChatGPT, Claude, and other RLHF-trained models.

---

## Connections

- [[Reinforcement Learning]] — reward is the training signal
- [[Agent]] — tries to maximise cumulative reward
- [[Value Function]] — estimates expected cumulative reward
- [[Policy]] — shaped by rewards over time

---

## One-line Summary

> The reward is the scalar feedback signal the environment gives the agent at each step — the agent must maximise its discounted cumulative sum (return), making reward design the most critical and most difficult part of applying RL to real problems.
