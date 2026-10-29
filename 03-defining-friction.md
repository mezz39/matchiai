# How to Define Friction in the Product

Use this method to answer: **where is the biggest friction?**

## 1. Define the user and the goal first

Friction only exists relative to a goal. Write one sentence:

> A job seeker with [CV state] wants to reach [target role] within [timeframe].

Without this, every step looks frictionless or equally broken.

## 2. Map the journey and the effort at each step

For each feature, write down:

- **Input required:** CV, target job, time budget.
- **Output received:** what the user gets back.
- **Assumption:** what must be true for this step to work. Examples:
  - The CV is parsed correctly.
  - The job posting is clear.
  - The country the user is looking in is identified.

Friction hides in the assumptions. A step with many fragile assumptions is a prime suspect.

## 3. Score each step on three questions (1 to 5)

| Question | Meaning |
|---|---|
| **Reach** | What share of users hit this friction? |
| **Severity** | Does it make them quit, or just annoy them? |
| **Proximity to value** | How close is it to the moment the user feels the product helped? |

## 4. Find the root cause with "5 whys"

Take the top candidate and keep asking why. Example:

1. Users do not trust the score.
2. Because they cannot see the calculation.
3. Because the matching logic is hidden.
4. Because the product shows a number without evidence.

## 5. State the evidence level honestly

Label each finding as one of:

- **Observed**
- **Inferred**
- **Assumed**

---

## Application to Matchlai

**Goal sentence:** a job seeker with a CV that does not match their target wants to reach a specific role within a few months.

Scores are 1 to 5 and are judgment, not user data. Evidence level for all of them: **inferred**, until tested on real CVs.

| Step | Reach | Severity | Proximity to value | Product |
|---|---|---|---|---|
| Foundation: CV upload and parsing | 5 | 4 | 2 | 40 |
| 2 Career target | 5 | 3 | 3 | 45 |
| 1 Gap analysis | 4 | 3 | 4 | 48 |
| **4 Job match score** | 4 | 5 | 5 | **100** |
| 6 Progress tracking | 3 | 5 | 4 | 60 |
| 7 Learning plan | 3 | 3 | 3 | 27 |
| 5 Q/A test | 2 | 2 | 2 | 8 |

**Result:** the biggest friction is the job match score (step 4).

**5 whys on the score**

1. Users do not trust the score.
2. Because they cannot see why it is 62% and not 80%.
3. Because the LLM's judgments are shown as one number, without evidence per requirement.
4. Because the score depends on three upstream pieces (the parsed CV, the normalized skills, the researched requirements baseline), and none of them is validated.
5. Root cause: there is no way yet to check the score against reality.

**Why tracking progress (step 6) is still a major risk:** it assumes users keep returning, report skill updates honestly, and that the baseline stays stable while job requirements change. It also inherits any error in the score.

**What to do about it**

- Show the evidence behind every requirement in the score.
- Add a confidence label to each score.
- Check scores against real CVs and job postings before building step 6.
- Count a skill as improved only when it is verified (for example, by the Q/A test), not just self-reported.
