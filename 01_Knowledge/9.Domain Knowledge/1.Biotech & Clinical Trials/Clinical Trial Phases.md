# Clinical Trial Phases

## What is it?

Clinical trials proceed through numbered **phases**, each answering a different question, with the previous phase's questions needing to be answered well enough to justify moving forward.

## The Phases

| Phase | Typical size | Main question |
|---|---|---|
| **Phase 1** | Tens, often healthy volunteers | Is it safe? What dose is tolerable? |
| **Phase 2** | Hundreds of patients | Signal of efficacy, at what dose, what side effects? |
| **Phase 3** | Hundreds to thousands | Does it work vs. standard-of-care/placebo, at a scale that catches rarer side effects? |
| **Phase 4** | Post-approval, real-world | Long-term safety/effectiveness monitoring |

## Formalizing "Advancement" and the Combo-Phase Choice

Let $\text{phase}(p, t)$ be program $p$'s phase at time $t$, and define advancement to phase $k+1$ as:

$$\text{advanced}_{k \to k+1}(p) = \mathbf{1}\left[\exists\, t : \text{phase}(p,t) \geq k+1\right]$$

The ambiguity is in what counts as "$\geq k+1$" when a program runs a **combo phase** like "Phase 1/2." Two conventions:

$$\text{inclusive}: \text{Phase } k/(k{+}1) \Rightarrow \text{phase} \geq k+1 \qquad \text{strict}: \text{Phase } k/(k{+}1) \Rightarrow \text{phase} = k \text{ only}$$

## Worked Example: Quantifying the Choice

Across a cohort of Phase-1-starting programs, suppose $N=100{,}000$, with 30,000 running a dedicated Phase 2 trial and 12,600 running only a "Phase 1/2" combo trial (no dedicated Phase 2):

$$\text{rate}_{\text{strict}} = \frac{30{,}000}{100{,}000} = 30.0\% \qquad \text{rate}_{\text{inclusive}} = \frac{30{,}000+12{,}600}{100{,}000}=42.6\%$$

A $12.6$ percentage-point gap from one definitional choice — comparable in magnitude to many real modeling improvements. This is why the choice must be explicit and, ideally, both rates reported side by side rather than silently picking one.

## Why It Matters

Phase is both a feature (current stage) and the basis for the label (did it reach the next phase) — see [[Drug Pipeline]] for how this feeds the unit-of-prediction decision, and [[Censoring]] for how "not yet advanced" gets resolved into success/failure/censored.

## Common Mistakes

- Silently picking one combo-phase convention without documenting it, breaking comparability with any differently-defined dataset
- Treating Phase 4 (post-approval monitoring) as part of the "advancement" pipeline — a fundamentally different stage, usually scoped out
- Assuming phase progression is strictly linear — programs can skip, restart, or run in parallel across indications

## Interview / discussion questions

- Formally define $\text{advanced}_{k\to k+1}(p)$ under both the inclusive and strict combo-phase conventions.
- Why might "did this program reach Phase 2" give a very different answer depending on convention, and by how much in a real cohort?
- Why is Phase 4 typically excluded from a phase-advancement prediction task?

## Prerequisites

None — foundational domain vocabulary

## Related concepts

[[Drug Pipeline]], [[Trial Status]], [[Data Leakage]], [[Censoring]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> Clinical trial phases mark increasingly demanding stages of testing, and the inclusive-vs-strict choice for combo phases (like Phase 1/2) is a precise, quantifiable modeling decision that can move reported advancement rates by 10+ percentage points.
