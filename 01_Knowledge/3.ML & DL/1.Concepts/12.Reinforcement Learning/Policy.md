# Policy

## What is it?

The **policy** $\pi$ is the [[Agent]]'s strategy — a mapping from states to actions (or distributions over actions). It completely defines the agent's behaviour.

$$\pi: S \rightarrow A \quad \text{(deterministic)}$$
$$\pi: S \rightarrow \Delta(A) \quad \text{(stochastic, where } \Delta(A) \text{ = probability distribution over } A\text{)}$$

The goal of [[Reinforcement Learning]] is to find the **optimal policy** $\pi^*$ that maximises expected cumulative reward.

---

## Deterministic vs. Stochastic Policies

**Deterministic policy:**
$$a_t = \pi(s_t)$$
Selects exactly one action for each state. Simple but can be exploited in adversarial settings and misses the exploration benefits of stochasticity.

**Stochastic policy:**
$$a_t \sim \pi(\cdot \mid s_t), \quad \pi(a\mid s) = P(A_t = a \mid S_t = s)$$
Selects actions probabilistically. Required for:
- Partially observable environments (stochasticity can encode uncertainty)
- Game theory settings (mixed strategies)
- Policy gradient methods (differentiable w.r.t. parameters)

---

## Policy Representation

### Tabular Policy (Small, Discrete State/Action Spaces)
Store $\pi(a\mid s)$ as a table. Exact but only feasible for small problems.

### Linear Policy
$$\pi_\theta(a\mid s) \propto \exp(\theta^T \phi(s,a))$$
Where $\phi(s,a)$ are hand-crafted state-action features. Fast but limited.

### Neural Network Policy (Deep RL)
$$\pi_\theta(a\mid s) = \text{softmax}(f_\theta(s))$$
- Input: state $s$ (raw pixels, sensor readings, etc.)
- Output: probability distribution over actions
- Parameters $\theta$ = network weights, updated by policy gradient

For continuous action spaces (robotics):
$$a \sim \mathcal{N}(\mu_\theta(s), \sigma_\theta(s)^2)$$
Policy outputs mean and variance of a Gaussian; actions are sampled.

---

## The Optimal Policy

$$\pi^*(s) = \arg\max_a Q^*(s,a)$$

The optimal policy acts greedily w.r.t. the optimal Q-function. If we know $Q^*$, we immediately know $\pi^*$.

---

## Policy Gradient Theorem

For parameterised policy $\pi_\theta$, the gradient of expected return $J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}[G_0]$:

$$\nabla_\theta J(\theta) = \mathbb{E}_{\tau\sim\pi_\theta}\left[\sum_{t=0}^T \nabla_\theta \log\pi_\theta(a_t\mid s_t) \cdot G_t\right]$$

**Intuition:** Increase $\log\pi_\theta(a_t\mid s_t)$ (probability of action $a_t$ in state $s_t$) proportionally to $G_t$ (how good the outcome was). Decrease probability of actions that led to poor outcomes.

**Variance reduction:** Replace $G_t$ with the **advantage** $A_t = G_t - V(s_t)$ (how much better than expected). Lower variance without bias.

---

## PPO (Proximal Policy Optimisation)

The dominant modern policy gradient algorithm:

$$\mathcal{L}^{\text{CLIP}}(\theta) = \mathbb{E}_t\left[\min\left(r_t(\theta)\hat{A}_t,\ \text{clip}(r_t(\theta), 1-\varepsilon, 1+\varepsilon)\hat{A}_t\right)\right]$$

Where $r_t(\theta) = \frac{\pi_\theta(a_t\mid s_t)}{\pi_{\theta_\text{old}}(a_t\mid s_t)}$ is the probability ratio (new vs. old policy).

The clipping prevents the new policy from deviating too far from the old one — stable, reliable updates. PPO is used to train ChatGPT/Claude in RLHF.

---

## Connections

- [[Reinforcement Learning]] — policy is the goal
- [[Agent]] — the agent IS its policy
- [[Value Function]] — can derive a policy from value function
- [[Exploration vs Exploitation]] — policy must balance exploring vs. exploiting

---

## One-line Summary

> The policy is the agent's complete strategy — a mapping from states to actions (or action distributions) — and the goal of RL is to find the optimal policy that maximises expected cumulative reward, learned directly via policy gradients (REINFORCE, PPO) or derived from an optimal value function.
