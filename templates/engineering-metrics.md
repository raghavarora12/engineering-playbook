# Engineering Metrics — What to Measure, What Not To, and Why

Metrics exist to inform decisions and surface system health — not to rank people. The moment a
metric becomes a target for individuals, it stops measuring anything real and starts manufacturing
the behavior it counts. This is Goodhart's law, and it is the single most common way engineering
measurement goes wrong.

The position of this document: **measure outcomes and system health; never measure individual
activity.**

## Measure this — outcomes and flow

The DORA metrics, because they measure the *system's* ability to deliver, and they resist gaming
because gaming one degrades another (ship faster recklessly and change-failure rate punishes you).

| Metric | What it tells you |
|--------|-------------------|
| Deployment frequency | How small and continuous your batches are |
| Lead time for changes | How long value waits between commit and production |
| Change failure rate | Whether speed is costing you stability |
| Time to restore (MTTR) | How well you recover, not just how rarely you fail |

Pair these with **reliability signals that map to users**: SLO attainment and error-budget burn
(see ADR-002). These tell you whether the system is meeting its promise, which is the only
availability number worth reporting upward.

## Don't measure this — activity and vanity

| Vanity metric | Why it misleads |
|---------------|-----------------|
| Lines of code | Rewards volume; the best change often *deletes* code |
| Commit / PR counts | Rewards splitting work to inflate the count; measures typing, not value |
| Story points as "productivity" | An estimation aid, not an output measure; turns planning into a target and corrupts both |
| Individual velocity rankings | Pits engineers against a number, kills collaboration and code review, optimizes the wrong unit (the individual, not the system) |
| Test count / coverage % as a goal | Coverage is a smoke detector, not a fire-safety rating; chasing the number breeds meaningless tests |

## Why vanity metrics mislead

1. **Goodhart's law.** "When a measure becomes a target, it ceases to be a good measure." Count
   commits and you'll get commits — smaller, more numerous, no more valuable.
2. **They measure the wrong unit.** Software value is produced by systems and teams, not by
   individuals in isolation. Metrics that rank individuals optimize a unit that doesn't ship value.
3. **They punish the right behaviors.** Deleting code, mentoring, unblocking a teammate, and
   preventing a problem all score zero on activity metrics — and they are exactly the work you most
   want from senior engineers.

## How to use metrics well

- **Trends over snapshots.** A number in isolation is noise; direction over time is signal.
- **Metrics start conversations, they don't end them.** A dip is a question ("what changed?"), not a
  verdict.
- **Team and system level, never individual performance.** The moment a metric appears in a
  performance review, assume it's now being gamed.
- **Measure to improve the system, not to rank the people in it.**
