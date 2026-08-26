<!--
HOW TO USE THIS TEMPLATE (delete this comment block in the finished ADR):
- Copy to /adrs/NNN-short-title.md with the next zero-padded number.
- Fill the front-matter completely. `status` stays `draft` until the author promotes it.
- Replace every *(italic guidance)* line with real content, then delete the guidance.
- Every [AUTHOR: …] is load-bearing — fill it with a real specific or leave it visibly
  open and flag it. Never invent a number, incident, or anecdote (CLAUDE.md).
- Keep the fixed sections and their order — a future tool parses these. Do not improvise.
-->
---
id: ADR-NNN
title: <short decision title, e.g. Canonical data model at ingestion vs. transform-on-read>
status: draft            # draft | accepted | superseded
tier: 1                  # 1 | 2 | 3  (per SPEC §3.2)
date: YYYY-MM-DD
tags: []                 # e.g. [data-strategy, reliability]
supersedes: null         # ADR-NNN or null
superseded_by: null      # ADR-NNN or null
related: []              # [ADR-NNN, PRIN-NNN] — cross-refs for the review agent
---

# ADR-NNN — <Decision title>

> *(One sentence: what was decided and the position taken. A reader who stops here knows
> the call and could disagree with it. Lead with the position, not the background.)*

## Context

*(The forces at play: the system, the constraint, the moment this decision had to be made.
Ground it in a real situation — [AUTHOR: the specific system / scale / incident]. Only what's
needed to understand the options; not a history lesson.)*

## Options Considered

*(Two or more genuinely viable options. Use the table — the A-vs-B-vs-C tradeoff is the core
of an ADR (SPEC §4.8). Keep prose for the* why *behind the row values, not to restate the table.)*

| Option | Cost / complexity | Failure modes | When it's the right call |
|--------|-------------------|---------------|--------------------------|
| A — … | | | |
| B — … | | | |

*(Add a Mermaid diagram here only if the options differ structurally or sequentially —
topology, event flow, failover sequence, decision tree — and it lands the point faster than a
sentence would. If not, cut it: a decorative diagram reads as* more *AI-generated, not less.)*

## Decision

*(The option chosen and what tipped it — not a restatement of the table. [AUTHOR: the real
judgment call and any figures that drove it].)*

## Consequences

*(What this buys you — and, mandatory, what it costs you and when this would be the* wrong *call
(SPEC §4.3 / CLAUDE.md). An ADR with no honest downside is not done.)*

## Status

*(draft | accepted | superseded — match the front-matter. Note any supersession, and cross-link
related ADRs or the Payments Resiliency Simulator where it is the working demonstration.)*

## References

Put reference links / docs here as evidence / citations.