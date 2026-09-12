# Reinforcement Learning

## What is it?

**Reinforcement Learning (RL)** is a type of [[Machine Learning]] where an **agent** learns to make decisions by interacting with an **environment**, receiving **rewards** for good actions and penalties for bad ones.

Unlike [[Supervised Learning]] (fixed labelled dataset), RL learns from **sequential interaction** — decisions affect future states and future rewards.

---

## The RL Framework

```
        Action aₜ
Agent ────────────→ Environment
  ↑                      │
  │  Observation sₜ₊₁    │
  │  Reward rₜ           │
  └──────────────────────┘
```

At each time step $t$:
1. Agent observes state $s_t$
2. Agent selects action $a_t$ according to policy $\pi(a\mid s)$
3. Environment transitions to new state $s_{t+1} \sim p(s'\mid s_t, a_t)$
4. Agent receives reward $r_t = R(s_t, a_t)$
5. Repeat

---

## The Markov Decision Process (MDP)

RL is formally defined as an MDP: $(S, A, P, R, \gamma)$

| Symbol | Name | Meaning |
|---|---|---|
| $S$ | State space | Set of all possible states |
| $A$ | Action space | Set of all possible actions |
| $P(s'\mid s,a)$ | Transition probability | Prob. of reaching $s'$ from $s$ with action $a$ |
| $R(s,a)$ | Reward function | Immediate reward for action $a$ in state $s$ |
| $\gamma \in [0,1)$ | Discount factor | How much to value future rewards |

**Markov property:** The future depends only on the current state, not the history: $P(s_{t+1}\mid s_t, a_t, s_{t-1}, a_{t-1}, \ldots) = P(s_{t+1}\mid s_t, a_t)$.

---

## The Objective

Maximise the **expected discounted cumulative reward** (return):

$$G_t = \sum_{k=0}^{\infty}\gamma^k r_{t+k} = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + \cdots$$

$\gamma$ controls the time horizon:
- $\gamma = 0$: myopic — only care about immediate reward
- $\gamma \to 1$: far-sighted — care about all future rewards equally
- $\gamma = 0.99$: typical value

---

## Key Concepts

- **[[Policy]]** $\pi(a\mid s)$: the strategy — probability of taking action $a$ in state $s$
- **[[Value Function]]** $V^\pi(s)$: expected return starting from state $s$ under policy $\pi$
- **Q-function** $Q^\pi(s,a)$: expected return starting from state $s$, taking action $a$, then following $\pi$
- **[[Reward]]** $r_t$: the scalar feedback signal
- **[[Environment]]**: everything outside the agent

---

## Major RL Algorithm Families

| Family | Key idea | Examples |
|---|---|---|
| **Dynamic Programming** | Full model known; compute value functions exactly | Policy Iteration, Value Iteration |
| **Model-Free Value-Based** | Estimate $Q(s,a)$ from experience | Q-Learning, DQN |
| **Policy Gradient** | Directly optimise policy parameters | REINFORCE, PPO, A3C |
| **Actor-Critic** | Combine value estimation + policy gradient | A2C, SAC, TD3 |
| **Model-Based RL** | Learn environment model; plan with it | Dyna, AlphaZero, MuZero |

---

## Bellman Equations

The fundamental recursive equations relating value functions:

**Bellman expectation equation:**
$$V^\pi(s) = \sum_a \pi(a\mid s)\sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma V^\pi(s')\right]$$

$$Q^\pi(s,a) = \sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma\sum_{a'}\pi(a'\mid s')Q^\pi(s',a')\right]$$

**Bellman optimality equation** (optimal policy $\pi^*$):
$$V^*(s) = \max_a\sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma V^*(s')\right]$$

$$Q^*(s,a) = \sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma\max_{a'}Q^*(s',a')\right]$$

The optimal policy: $\pi^*(s) = \arg\max_a Q^*(s,a)$.

---

## Q-Learning (Model-Free)

Learns $Q^*(s,a)$ without knowing $P(s'\mid s,a)$:

$$Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha\underbrace{\left[r_t + \gamma\max_{a'}Q(s_{t+1},a') - Q(s_t,a_t)\right]}_{\text{TD error }\delta_t}$$

Where $\alpha$ = learning rate. The **TD error** $\delta_t$ is the difference between the estimated value and the bootstrapped target.

**DQN (Deep Q-Network):** Replace the Q-table with a neural network $Q(s,a;\theta)$, trained via:
$$\mathcal{L}(\theta) = \mathbb{E}\left[(y_t - Q(s_t,a_t;\theta))^2\right]$$
$$y_t = r_t + \gamma\max_{a'}Q(s_{t+1},a';\theta^-)$$

Key techniques: experience replay buffer, target network $\theta^-$ (frozen copy of $\theta$).

---

## Policy Gradient

Directly parameterise the policy $\pi_\theta(a\mid s)$ and optimise:
$$J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}[G_0]$$

**REINFORCE gradient:**
$$\nabla_\theta J(\theta) = \mathbb{E}_{\tau}\left[\sum_t\nabla_\theta\log\pi_\theta(a_t\mid s_t)\cdot G_t\right]$$

Intuition: increase probability of actions that led to high returns.

---

## Applications

| Domain | Application |
|---|---|
| Games | AlphaGo/AlphaZero, Atari (DQN), Dota 2 (OpenAI Five) |
| Robotics | Locomotion, manipulation, dexterous hands |
| NLP | RLHF (training ChatGPT/Claude from human feedback) |
| Finance | Portfolio optimisation, trading strategies |
| Healthcare | Treatment planning, drug discovery |
| Operations | Data centre cooling, supply chain |

---

## Connections

- [[Agent]] — the learner
- [[Environment]] — what the agent interacts with
- [[Reward]] — the training signal
- [[Policy]] — the learned strategy
- [[Value Function]] — expected cumulative reward
- [[Exploration vs Exploitation]] — the fundamental dilemma

---

## One-line Summary

> Reinforcement learning trains an agent to maximise cumulative reward through trial-and-error interaction with an environment — formalised as an MDP, solved by learning value functions (Q-learning, DQN) or directly optimising policies (policy gradient), and behind breakthroughs from AlphaGo to ChatGPT's RLHF training.
