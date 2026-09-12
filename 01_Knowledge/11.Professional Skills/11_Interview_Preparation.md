# Professional Skills: Interview Preparation

## What This Note Covers

This is your **interview playbook** — structured approaches to both behavioral and technical questions that determine 50% of your hiring outcome. The other 50% is your technical depth, but no amount of leetcode skill saves you if you can't communicate it clearly under pressure.

> Interviews test three things: can you code? can you think systematically? can you communicate that thinking? All three matter.

---

## The Three Interview Pillars

1. **Behavioral / Soft Skills** (40% of outcome)
   - "Tell me about yourself"
   - "Why do you want this role?"
   - "Describe a time you failed"
   - Rapport building, communication clarity

2. **Technical Problem-Solving** (40% of outcome)
   - Algorithm design (LeetCode-style)
   - Data structures
   - System design (for mid-level+ roles)
   - "Walk me through your approach"

3. **Domain & Project Knowledge** (20% of outcome)
   - Your specific projects (Drug Pipeline, etc.)
   - What you learned
   - What you'd do differently
   - Connections to the role

---

## Section 1: Behavioral Interview Prep

### The STAR Method (Structure Your Stories)

**STAR = Situation, Task, Action, Result**

Every behavioral question is answered with a **concrete story** that demonstrates a skill. Interviewers want evidence, not claims.

#### STAR Template

```
SITUATION (30 seconds):
  "I was working on [project], where we needed to [challenge]."
  
TASK (20 seconds):
  "My responsibility was [your specific role]."
  "The constraint was [deadline/resource/complexity]."
  
ACTION (60 seconds):
  "Here's what I did: [specific steps, choices, reasoning]."
  "When [obstacle] came up, I [overcame it by]."
  
RESULT (20 seconds):
  "The outcome was [quantified result if possible]."
  "I learned [what you learned]; I'd do [differently] next time."
```

**Total: ~2 minutes per story**

---

### Essential STAR Stories (Prepare 4-5 of These)

#### Story 1: Technical Challenge (Shows Problem-Solving)

**Template:**
```
SITUATION:
  "I was working on [project—use Drug Pipeline if strongest example].
   We had [technical problem]: [describe clearly]."

TASK:
  "I was responsible for [solving/debugging/optimizing] this."

ACTION:
  "I broke it down: first I [step 1], then I [step 2].
   The key insight was [key technical decision].
   I used [tools/techniques] because [reasoning].
   When [setback] happened, I [adapted by]."

RESULT:
  "We achieved [outcome]: [performance improvement/feature shipped/deadline met].
   In retrospect, I'd [improvement/lesson learned]."
```

**Examples from Drug Pipeline:**
- Debugging schema exploration script → schema issues discovered
- Handling label ambiguity → defined confidence tiers
- Optimizing data queries → joined multiple tables efficiently

#### Story 2: Failure / Setback (Shows Humility + Growth)

**Template:**
```
SITUATION:
  "Early in [project/internship], I [made mistake/misjudged approach]."

TASK:
  "I was responsible for [what you were working on]."

ACTION:
  "I initially [wrong approach]. When [how you discovered it was wrong],
   I realized [what I missed]. Then I [how you fixed it].
   It taught me [the lesson]."

RESULT:
  "The project still shipped on time / delivered results.
   Now I [how you apply that lesson differently]."
```

**Good candidates for this:**
- Early inefficient approach → optimized after feedback
- Misunderstood requirements → clarified and reframed
- Overestimated complexity → learned to scope better

#### Story 3: Collaboration / Teamwork (Shows Communication)

**Template:**
```
SITUATION:
  "I was working with [team/mentor] on [project].
   We had [communication challenge / disagreement / dependency]."

TASK:
  "We needed to [align on approach / unblock each other / deliver together]."

ACTION:
  "I listened to their perspective: [what they wanted].
   I shared my thinking: [your reasoning].
   We landed on [compromise/synthesis] by [how you resolved it].
   I made sure [how you enabled the collaboration]."

RESULT:
  "We delivered [result]. The collaboration strengthened [what changed].
   I learned [what the interaction taught you]."
```

#### Story 4: Initiative / Ownership (Shows Drive)

**Template:**
```
SITUATION:
  "I noticed [gap/inefficiency/opportunity] on the project."

TASK:
  "No one had been assigned to fix it yet."

ACTION:
  "I took the initiative to [what you did]. I didn't wait for
   [permission/full clarity], but [how you handled uncertainty].
   I [specific steps] to [outcome]."

RESULT:
  "The result was [improvement/time saved/new capability].
   It also [side benefit: built relationships / learned new skill / impressed team]."
```

#### Story 5: Learning / Rapid Skill Growth (Shows Adaptability)

**Template:**
```
SITUATION:
  "I faced [new technology/domain/challenge] I'd never done before."

TASK:
  "I needed to become productive in [X] by [deadline]."

ACTION:
  "I [learning approach]: [specific resources used].
   I [built something small to learn]. I [asked good questions of].
   Within [timeframe], I [what you could do]."

RESULT:
  "I delivered [what you built/contributed].
   Now [how this skill benefits your next role]."
```

---

### Common Behavioral Questions & How to Answer

| Question | What They Want | STAR Focus |
|----------|----------------|----|
| **"Tell me about yourself"** | 60-second summary: background → why you're here | Use Story 5 (rapid learning) as bridge |
| **"Why do you want this role?"** | Genuine interest + fit | Role-specific: what excites you about THIS company/role |
| **"Why are you leaving?"** | Stability + ambition | Frame as "seeking growth" not "running from problem" |
| **"Describe your greatest weakness"** | Self-awareness + growth mindset | Pick real weakness you're working on + concrete steps |
| **"Tell me about a time you failed"** | Resilience + learning | Use Story 2 (Failure); emphasize recovery |
| **"Tell me about a time you disagreed with someone"** | Respectful communication; can challenge constructively | Use Story 3 (Collaboration) but with respectful disagreement |
| **"Tell me about a time you had to learn something quickly"** | Adaptability + resourcefulness | Use Story 5 (Learning) |
| **"What's your greatest strength"** | Confidence + evidence | Use Story 1 (Technical Challenge) if relevant to role |
| **"How do you handle pressure?"** | Calm, systematic thinking under stress | Reference incident with deadline / high stakes |

---

## Section 2: Technical Interview Workflow

### The Template: How to Approach a Coding Interview

#### Phase 1: Clarification (2 minutes)

**What you do:**
- Read problem twice
- Identify input/output types, constraints
- Ask clarifying questions

**Questions to ask:**
```
"Is the input sorted?"
"Can there be duplicates?"
"What's the range of n? (10, 1M, 1B?)"
"Should I optimize for time or space?"
"Can I modify the input?"
"What if there's no valid answer?"
```

#### Phase 2: Approach (3 minutes)

**What you do:**
- Say your approach out loud
- State the time/space complexity
- Walk through an example

**Template:**
```
"I'll solve this with [technique] because [reasoning].
 First, I [step 1: complexity].
 Then, I [step 2: complexity].
 Overall: O(time) time, O(space) space.
 Let me trace through the example..."
```

#### Phase 3: Code (10-15 minutes)

**What you do:**
- Write clean, readable code
- Think out loud
- Handle edge cases

**Best practices:**
- Use clear variable names (`left` not `l`, `sorted_arr` not `sa`)
- Comment complex logic
- Avoid off-by-one errors
- Leave blank lines for readability

#### Phase 4: Test (5 minutes)

**What you do:**
- Trace through your code on the given example
- Test edge cases:
  - Empty input
  - Single element
  - All duplicates
  - Very large input (does it time out?)

**Say it out loud:**
```
"Let me trace through: arr = [1, 3, 2].
 i=0: [we do X]
 i=1: [we do Y]
 Result: [correct? yes/no]"
```

#### Phase 5: Optimize (5 minutes, if time)

**What you do:**
- "Can we do better?"
- Identify bottleneck
- Suggest improvement
- Estimate new complexity

**Template:**
```
"Current solution is O(n²). The bottleneck is [operation].
 We could optimize to O(n log n) by [using technique].
 Would you like me to code that?"
```

---

### How to Explain Your Solution (The "Why")

**Interviewers care less about code, more about:**
- Can you think through problems systematically?
- Can you defend your choices?
- Do you recognize tradeoffs?

**Always explain:**
1. **Why this data structure?** "I'm using a hash set because O(1) lookup is critical here; array would be O(n)."
2. **Why this algorithm?** "Binary search because the array is sorted, reducing $O(n)$ to $O(\log n)$."
3. **Why this complexity?** "We must iterate through all n elements, so O(n) is a lower bound."

---

## Section 3: System Design (For Mid-Level+ Roles)

### Scale: What You're Designing For

| Scale | RPS | Storage | Example Role |
|-------|-----|---------|--------------|
| **Small** | 100 RPS | <100 GB | Junior → Senior IC |
| **Medium** | 1K-10K RPS | 100 GB - 1 TB | Senior → Staff |
| **Large** | 100K+ RPS | 1+ TB | Staff → Principal |

For **ML Engineer** roles, system design often covers:
- **Training pipeline:** data ingestion → preprocessing → model training → versioning
- **Serving pipeline:** model load → preprocessing → inference → response
- **Monitoring:** model performance drift, data drift, serving latency

---

### ML System Design Template

#### 1. Requirements

"Let me clarify the requirements:
- What's the target metric? (accuracy, latency, cost?)
- What's the SLA? (99.99% uptime?)
- What scale? (1K predictions/day? 1M/sec?)"

#### 2. High-Level Architecture

```
Data Pipeline
  ├─ Collect raw data
  ├─ Store in data lake (S3, Parquet)
  └─ Preprocess & feature engineering

Training Pipeline
  ├─ Load training data in batches
  ├─ Train model (local or distributed)
  ├─ Validate on holdout
  └─ Version model & artifacts

Serving Pipeline
  ├─ Load model to inference server
  ├─ Preprocess incoming request (same as training!)
  ├─ Run inference
  └─ Return prediction

Monitoring
  ├─ Track model accuracy over time
  ├─ Alert if drift detected
  └─ Log predictions for analysis
```

#### 3. Tradeoffs

"We could use [simpler approach] but that sacrifices [what].
 Instead, [our approach] trades [cost] for [benefit]."

---

## Section 4: Domain Knowledge — Your Projects

### How to Frame Your Drug Pipeline Project

**Generic answer (weak):**
```
"I built a model to predict drug advancement."
```

**Strong answer (use this):**
```
"I worked on Drug Pipeline Advancement Forecasting, a machine learning system
that predicts whether a drug program advances to the next clinical phase.

Here's the impact:
- Formulated the problem: define what 'advance' means (phase transition);
  handle data ambiguity (multiple indications, trial designs)
- Built the pipeline: loaded 192 tables, joined molecule/trial/phase data,
  created phase transition labels from clinical outcomes
- Found key features: phase history, therapeutic area, trial status
- Result: could rank drugs by advancement probability, helping prioritize

Key technical challenges I solved:
1. Schema complexity: 192 tables; had to understand which were relevant
2. Label ambiguity: what defines success? Resolved through domain logic
3. Data quality: missing molecule IDs; built a mapping table

What I'd do differently:
- Build more sophisticated outcome prediction first (before phase prediction)
- Incorporate regulatory pathway info earlier
- Validate assumptions with a domain expert
```

**Interview follow-ups to expect:**
- "What was the hardest part?" → Label definition + schema navigation
- "What did you learn?" → Domain (clinical trials) + systems (large data warehouse)
- "How would you evaluate?" → Precision for expensive false positives; recall for discovery
- "Scale it up?" → Distributed computing, real-time serving

---

## Section 5: Questions to Ask Them (Shows Interest)

Ask 2-3 questions at the end. Shows you're genuinely interested, not just seeking a job.

**Good questions:**
- "What's the biggest technical challenge the team is facing right now?"
- "How is success measured in this role? (What does a great first 90 days look like?)"
- "What's the career trajectory from this position?"
- "How does the team handle technical debt?"
- "What's the culture like around code review and learning?"

**Avoid:**
- "How much vacation do I get?" (save for offer stage)
- "When would I get a raise?" (save for offer stage)
- Questions they already answered

---

## Section 6: Interview Day Logistics

### Before the Interview

- **Night before:** Sleep well; review 1-2 past problems
- **30 min before:** Review your STAR stories and resume
- **5 min before:** Deep breath; technical skills are locked in; confidence is what changes now

### During the Interview

| Do | Don't |
|----|-------|
| Speak clearly and slowly | Rush to coding |
| Think out loud ("I'm checking...") | Code in silence then explain |
| Ask clarifying questions | Assume you understand |
| Test your code | Submit untested code |
| Acknowledge mistakes | Pretend errors aren't there |
| Explain your tradeoffs | Act like there's one "right" answer |
| Show enthusiasm | Interview like it's a chore |

### After the Interview

- Thank them (email within 24 hours if they provided address)
- Mention something specific from the conversation
- Reiterate genuine interest in the role

---

## Red Flags: What Kills An Otherwise Good Interview

| Red Flag | How to Avoid |
|----------|--------------|
| Can't explain complexity of your own code | Practice explaining beforehand |
| Gives up quickly on hard problem | Say "let me think through this" + stay engaged |
| No questions for them | Prepare 3 questions; shows genuine interest |
| Vague STAR stories ("I did some stuff") | Practice stories with specifics + numbers |
| Dismissive of feedback ("Actually, you're wrong") | Listen; integrate feedback gracefully |
| Negative about past experiences/companies | Stay professional; frame as learning |

---

## Practice Plan (Next 4 Weeks to Interview Day)

### Week 1-2: Behavioral Prep
- [ ] Write out 5 STAR stories with specifics
- [ ] Practice 2 min "tell me about yourself"
- [ ] Record yourself; listen for clarity/filler words
- [ ] List 3 questions to ask interviewers

### Week 3: Technical Practice
- [ ] Solve 10-15 LeetCode problems (mix DSA topics)
- [ ] Practice explaining approach before coding
- [ ] Time yourself; aim for 25 min total per problem
- [ ] Record a mock interview; watch it back

### Week 4: Integration
- [ ] 2-3 mock interviews with real feedback
- [ ] Refine STAR stories based on feedback
- [ ] Memorize 1-2 system design templates
- [ ] Review company + role specifics

---

## Interview Checklist (Day Before)

- [ ] Resume reviewed; you can defend every line
- [ ] 5 STAR stories memorized (not recited; conversational)
- [ ] 3 questions for them prepared
- [ ] 5-10 DSA solutions reviewed
- [ ] One mock interview done in past week
- [ ] Tech setup tested (webcam, audio, screen share)
- [ ] Quiet space arranged for interview
- [ ] Clothes selected (professional but comfortable)

---

## Key Takeaways

1. **Behavioral >> Technical for first interview** — You can't code your way out of bad communication
2. **Tell stories, not claims** — "I improved efficiency" vs. "I analyzed the bottleneck, recognized $O(n^2)$ was the issue, refactored to $O(n \log n)$ using binary search"
3. **Practice out loud** — Your brain doesn't know what your mouth can communicate until you say it
4. **Mistakes are okay; giving up isn't** — "I made an off-by-one error; let me trace through... ah, I see it"
5. **Interview is a conversation** — They want to hire someone they can work with

---

## Related Notes

- [[1.CS Foundations]] — Complexity to explain in interviews
- [[2.DSA]] — Problem patterns and solutions
- [[8.Systems, Cloud & System Design]] — System design deep dives
- [[11.Professional Skills Overview]] — Broader professional development

#category/professional-skills #topic/interviews #topic/behavioral #topic/technical-interviews

