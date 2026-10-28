# What Metrics Matter Most

## 1. Advancement (north star)

- **Goal progress %** = `(current match score − baseline score) / (target score − baseline score) × 100`
- **Advancement rate** = `users with at least X points of gain within 30 days / activated users × 100`
- **Time to progress** = median days from first CV upload to first meaningful gain (for example, +10 points)

**Caveat:** the match score is produced by your own system, so if the matching is wrong, this metric is circular. Validate it against external outcomes: the share of advanced users who report interviews or offers.

## 2. Tracking completion

- **Completion rate** = `users who completed the tracking cycle / users who started it × 100`
- Define "completed" up front, such as reaching the target or finishing every milestone in the plan.
- Also track **milestone drop-off** = `users reaching milestone n+1 / users reaching milestone n`

## 3. Feedback quality

- **Positive share** = `positive responses / (positive + negative) × 100`
  - This is better than a good-to-bad ratio, which is unbounded and unstable when the numbers are small.
- **Net satisfaction** = `(positive − negative) / total responses`
- **Response rate** = `users who gave feedback / users who were asked`
  - If this is low, treat the other two numbers with caution.

## 4. Workflow feedback (diagnostic)

For each of the five workflows:

- **Workflow satisfaction** = `positive responses / total responses for that workflow`
- **Coverage** = `responses / users who used that workflow`
- **Friction index** = `(1 − satisfaction) × reach`, where `reach = users who used the workflow / all active users`
