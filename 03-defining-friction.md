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
