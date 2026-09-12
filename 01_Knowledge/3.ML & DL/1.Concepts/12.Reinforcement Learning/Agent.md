## What is it?

The **agent** is the learner and decision-maker in [[Reinforcement Learning]]. It observes the environment's state, selects actions, and updates its behaviour based on received rewards.

$$\text{Agent} = \text{Perception} + \text{Decision-making} + \text{Learning}$$

---

## What the Agent Contains

| Component | Symbol | Role |
|---|---|---|
| **Policy** | $\pi(a\mid s)$ | Decision rule: what action to take in each state |
| **Value function** | $V(s)$ or $Q(s,a)$ | Estimate of future rewards from a state |
| **Model** (optional) | $\hat{P}(s'\mid s,a)$ | Learned model of the environment |

An agent may have all three, or just a policy (policy-based), or just a value function (value-based).

---

## Agent Types

| Type | Has policy? | Has value fn? | Has model? |
|---|---|---|---|
| Value-based (Q-Learning) | Implicit (greedy) | ✅ | ❌ |
| Policy-based (REINFORCE) | ✅ (explicit) | ❌ | ❌ |
| Actor-Critic (A2C, PPO) | ✅ | ✅ | ❌ |
| Model-Based (Dyna) | ✅ | ✅ | ✅ |

---

## Connections

- [[Reinforcement Learning]] — the framework
- [[Policy]] — the agent's strategy
- [[Value Function]] — the agent's predictions
- [[Environment]] — what the agent interacts with

---

## One-line Summary

> The agent is the RL system's decision-maker — it perceives states, selects actions via its policy, estimates future rewards via its value function, and updates all components through trial-and-error interaction with the environment.
