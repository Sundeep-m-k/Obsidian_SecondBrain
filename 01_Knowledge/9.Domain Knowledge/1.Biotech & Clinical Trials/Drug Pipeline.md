# Drug Pipeline (Program)

## What is it?

A **drug pipeline**, at the company level, is the full set of drug candidates ("programs") in development. A **program** is a specific molecule developed for a specific indication:

$$\text{program} := (\text{molecule\_id}, \text{indication\_id})$$

The same molecule tested for two different diseases counts as two distinct programs, since success/failure in one carries no guaranteed information about the other.

## Why "Program" Isn't a Free Column

Databases often record phase/status/trial detail cleanly but lack one clean ID for "this molecule, for this indication, tracked over time." That identity has to be *constructed* — via [[Entity Resolution]]'s tiered key over molecule and indication identifiers — before asking "did this program advance?"

## Why the Unit-of-Prediction Choice Is Load-Bearing

Whatever "one program" means becomes the unit every downstream label, feature, and evaluation is built around. Formally, if the true resolution function is $\text{program}^*(i)$ but the constructed key $\widehat{\text{program}}(i)$ fragments some true programs into $k>1$ pieces (or merges $k>1$ true programs into one), every rate computed by grouping on $\widehat{\text{program}}$ is a biased estimate of the rate computed on $\text{program}^*$ — see [[Indication vs Therapeutic Area]] for a measured example of exactly this bias (−5.1pp/−2.5pp from indication fragmentation).

## Why Therapeutic Area ≠ Program Identity

Therapeutic area is a useful **feature** but too coarse for **identity**: the same disease can scatter across multiple TA labels due to inconsistent historical categorization (see [[Indication vs Therapeutic Area]]'s worked HIV example — 3 different TA buckets for one disease). Using TA as an identity key would incorrectly merge unrelated programs that happen to share a TA, or split one real program's records across TA-inconsistent rows.

## Common Mistakes

- Using a broad category (TA) as a stand-in for program identity instead of a properly resolved molecule + indication key
- Assuming a company's internal "program" concept lines up cleanly with how the data source records trials — check the actual grain first (see [[Spine Table Pattern]])
- Not tracking sponsor changes over a program's life (academic-to-industry handoffs — see [[Sponsor Type]]) that affect scoping

## Interview / discussion questions

- Why is "molecule + indication" a better program definition than "molecule" alone?
- Formally, why does a fragmented identity key bias every rate computed by grouping on it?
- Why can't therapeutic area substitute for a properly resolved program identity?

## Prerequisites

[[Clinical Trial Phases]], [[Entity Resolution]]

## Related concepts

[[Indication vs Therapeutic Area]], [[Sponsor Type]], [[Confidence-Tiered Matching]]

## Tags

#category/domain-knowledge #topic/clinical-trials

## One-line summary

> A program is formally (molecule, indication), an identity that has to be constructed via entity resolution — and any fragmentation of that identity key is a measurable source of bias in every downstream rate.
