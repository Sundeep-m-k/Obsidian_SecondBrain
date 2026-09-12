# Artificial Intelligence

## What is it?

**Artificial Intelligence (AI)** is the broad field of computer science concerned with building systems that can perform tasks which, if done by a human, would require **intelligence** — perception, reasoning, learning, planning, language understanding, and decision-making.

> Think of AI as the *goal*, and everything else (ML, deep learning, etc.) as different approaches to reach that goal.

---

## The Hierarchy (Know This Cold)

```
Artificial Intelligence  ← broadest goal
    └── Machine Learning     ← learns from data
            └── Deep Learning    ← learns via neural networks
                    └── Generative AI / LLMs / etc.
```

AI contains ML. ML contains Deep Learning. Not all AI is ML (e.g., rule-based expert systems are AI but not ML).

---

## A Brief History (Why It Matters)

| Era | Period | Key Idea |
|---|---|---|
| Symbolic AI / GOFAI | 1950s–1980s | Hand-coded rules and logic |
| First AI Winter | 1974–1980 | Funding dried up; rules couldn't scale |
| Expert Systems | 1980s | Domain-specific rule databases |
| Second AI Winter | 1987–1993 | Expert systems too brittle |
| Statistical / ML era | 1990s–2010s | Learn patterns from data |
| Deep Learning era | 2012–present | Neural nets + big data + GPUs |

---

## Core Sub-fields of AI

| Sub-field | What it studies |
|---|---|
| **Machine Learning** | Systems that improve from experience/data |
| **Computer Vision** | Understanding images and video |
| **Natural Language Processing (NLP)** | Understanding and generating text/speech |
| **Robotics** | Perception + action in the physical world |
| **Planning & Search** | Finding sequences of actions to reach goals |
| **Knowledge Representation** | Encoding facts and reasoning over them |
| **Multi-agent Systems** | Multiple AI agents interacting |

---

## Types of AI by Capability (Conceptual)

### Narrow AI (ANI — Artificial Narrow Intelligence)
- Does **one specific task** extremely well.
- All AI that exists today is narrow AI.
- Examples: chess engines, recommendation systems, image classifiers, ChatGPT.

### General AI (AGI — Artificial General Intelligence)
- Can perform **any intellectual task a human can**.
- Does not exist yet (as of 2024). Active research area.
- Would require reasoning, common sense, transfer learning across all domains.

### Superintelligence (ASI)
- Hypothetical AI that surpasses human intelligence in every domain.
- Subject of serious safety research (Bostrom, Russell, Anthropic, etc.).

---

## The Turing Test

Proposed by Alan Turing (1950) in *"Computing Machinery and Intelligence"*.

**Setup:** A human judge chats (in text) with a human and a machine. If the judge cannot reliably tell which is which, the machine is said to exhibit intelligent behaviour.

**Criticism:** Passing the Turing Test doesn't mean *understanding* — it may only mean *imitation* (see: Chinese Room argument by Searle).

---

## Intelligent Agent Framework

Most modern AI is formalised as an **intelligent agent**:

```
Environment → [Sensors] → Agent → [Actuators] → Environment
                              ↑
                         (Perception → Reasoning → Action)
```

An agent is anything that:
1. **Perceives** its environment through sensors.
2. **Acts** upon the environment through actuators.
3. Does so to **maximise a performance measure** (its goal).

**PEAS description** — how to specify any AI agent:

| Letter | Stands for | Example (self-driving car) |
|---|---|---|
| P | Performance measure | Safety, speed, legality, comfort |
| E | Environment | Roads, traffic, pedestrians, weather |
| A | Actuators | Steering, brakes, accelerator, horn |
| S | Sensors | Camera, LIDAR, GPS, speedometer |

---

## Approaches to AI

### 1. Symbolic / Rule-based AI
- Knowledge is explicitly encoded as **if-then rules**.
- Example: `IF temperature > 38°C AND cough = true THEN diagnose = flu`
- Strengths: Interpretable, no data needed.
- Weaknesses: Brittle, doesn't generalise, doesn't learn.

### 2. Machine Learning
- System learns rules **from data** rather than being explicitly programmed.
- See: [[Machine Learning]] note.

### 3. Neural / Connectionist AI
- Loosely inspired by the brain.
- Large networks of simple units (neurons) that learn distributed representations.
- Powers modern deep learning.

### 4. Evolutionary / Search-based AI
- Uses search algorithms (A\*, minimax, MCTS) or evolutionary algorithms (genetic algorithms) to find solutions.

---

## Why AI is Hard: Key Challenges

| Challenge | Description |
|---|---|
| **Brittleness** | AI often fails on inputs slightly outside its training distribution |
| **Data hunger** | Many methods need enormous labelled datasets |
| **Interpretability** | Hard to understand *why* a model made a decision |
| **Reward hacking** | Agent maximises the metric in unintended ways |
| **Common sense** | Humans effortlessly reason about causality, physics, social norms — AI struggles |
| **Transfer learning** | Knowledge learned in one domain rarely transfers cleanly to another |

---

## Key Notation You'll See Everywhere

| Symbol | Meaning |
|---|---|
| $x$ | Input (a single data point or feature vector) |
| $y$ | Output / target label |
| $f$ | A function (the model maps $x \to y$) |
| $\hat{y}$ | Predicted output (what the model thinks $y$ is) |
| $\theta$ or $w$ | Parameters/weights of the model |
| $\mathcal{D}$ | Dataset |
| $\mathcal{L}$ | Loss function (measures error) |

---

## Connections

- [[Machine Learning]] — the dominant approach to AI today
- [[Supervised Learning]] — the most common ML paradigm
- [[Unsupervised Learning]] — finding structure without labels
- [[Model]] — the mathematical object an AI system produces
- [[Inference]] — using a trained model to make predictions

---

## One-line Summary

> AI is the science of making machines that act intelligently; today, nearly all practical AI is built using Machine Learning — systems that learn patterns from data rather than following hand-written rules.
