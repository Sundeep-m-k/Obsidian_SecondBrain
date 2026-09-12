# Value Function

## What is it?

The **value function** estimates the **expected cumulative future reward** from a given state (or state-action pair), under a given [[Policy]] $\pi$. It tells the agent how "good" it is to be in a particular situation.

---

## Two Types

### State Value Function $V^\pi(s)$
Expected return starting from state $s$, then following policy $\pi$:

$$V^\pi(s) = \mathbb{E}_\pi\left[G_t \mid S_t = s\right] = \mathbb{E}_\pi\left[\sum_{k=0}^\infty \gamma^k r_{t+k} \mid S_t = s\right]$$

### Action-Value Function $Q^\pi(s,a)$ (Q-function)
Expected return starting from state $s$, **first** taking action $a$, then following $\pi$:

$$Q^\pi(s,a) = \mathbb{E}_\pi\left[G_t \mid S_t = s, A_t = a\right]$$

**Relationship:**
$$V^\pi(s) = \sum_a \pi(a\mid s) Q^\pi(s,a) = \mathbb{E}_{a\sim\pi}[Q^\pi(s,a)]$$

---

## Bellman Equations

The value functions satisfy recursive equations — this is the key to computing them.

**Bellman expectation equation for $V^\pi$:**
$$V^\pi(s) = \sum_a \pi(a\mid s)\sum_{s'} P(s'\mid s,a)\left[R(s,a) + \gamma V^\pi(s')\right]$$

This says: the value of a state = immediate reward + discounted value of the next state, averaged over all possible actions and transitions.

**Bellman expectation equation for $Q^\pi$:**
$$Q^\pi(s,a) = \sum_{s'} P(s'\mid s,a)\left[R(s,a) + \gamma\sum_{a'}\pi(a'\mid s')Q^\pi(s',a')\right]$$

**Bellman optimality equations** ($\pi^*$):
$$V^*(s) = \max_a\sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma V^*(s')\right]$$

$$Q^*(s,a) = \sum_{s'}P(s'\mid s,a)\left[R(s,a) + \gamma\max_{a'}Q^*(s',a')\right]$$

**Optimal policy from $Q^*$:** $\pi^*(s) = \arg\max_a Q^*(s,a)$.

---

## Advantage Function

$$A^\pi(s,a) = Q^\pi(s,a) - V^\pi(s)$$

Measures how much **better** action $a$ is compared to the average action from state $s$ under policy $\pi$.
- $A > 0$: action $a$ is better than average → increase its probability
- $A < 0$: action $a$ is worse than average → decrease its probability
- Used in Actor-Critic methods to reduce variance in policy gradient estimates

---

## Learning the Value Function: TD Learning

**Temporal Difference (TD) learning** updates $V$ using bootstrapped estimates:

$$V(s_t) \leftarrow V(s_t) + \alpha\underbrace{\left[r_t + \gamma V(s_{t+1}) - V(s_t)\right]}_{\text{TD error }\delta_t}$$

The **TD error** $\delta_t = r_t + \gamma V(s_{t+1}) - V(s_t)$ is the difference between:
- **Target:** $r_t + \gamma V(s_{t+1})$ (bootstrapped estimate of true value)
- **Current estimate:** $V(s_t)$

TD learning is online — it updates after each step, not after a full episode.

**Q-Learning update (off-policy):**
$$Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha\left[r_t + \gamma\max_{a'}Q(s_{t+1},a') - Q(s_t,a_t)\right]$$

---

## Deep Value Functions (DQN)

For large or continuous state spaces, represent $Q(s,a;\theta)$ as a neural network.

**DQN loss:**
$$\mathcal{L}(\theta) = \mathbb{E}_{(s,a,r,s')\sim\mathcal{B}}\left[\left(r + \gamma\max_{a'}Q(s',a';\theta^-) - Q(s,a;\theta)\right)^2\right]$$

Where $\mathcal{B}$ = replay buffer, $\theta^-$ = target network (slowly updated copy of $\theta$).

**Key tricks:**
- **Experience replay:** Store transitions $(s,a,r,s')$ in buffer; sample random mini-batches to break correlations.
- **Target network:** Use a frozen copy $\theta^-$ for the TD target; prevents moving-target instability.

---

## Connections

- [[Reinforcement Learning]] — value functions are a core component
- [[Policy]] — optimal policy derived from $Q^*$
- [[Reward]] — value functions integrate reward over time
- [[Agent]] — the agent stores and updates value functions

---

## One-line Summary

> The value function estimates expected cumulative reward from a state (or state-action pair) under a policy — defined recursively via Bellman equations, learned via TD updates, and central to both value-based methods (Q-Learning, DQN) and actor-critic methods that use it to reduce policy gradient variance.
