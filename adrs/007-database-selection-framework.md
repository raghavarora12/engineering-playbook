---
id: ADR-007
title: Database-selection framework — relational vs. document vs. wide-column vs. key-value
status: draft
tier: 2
date: 2026-07-11
tags: [databases, data-modeling, selection-framework]
supersedes: null
superseded_by: null
related: []
---

# ADR-007 — Database-selection framework

> A framework, not a ranking. Pick by access pattern, consistency needs, and scale shape — not by
> what's fashionable. Default to relational until a specific pressure forces a move; "NoSQL by
> default" is how you end up re-implementing joins and transactions in application code.

## Context

Database selection is where architecture judgment is most often replaced by habit or hype. The
right output of this decision is not "use X" — it's a repeatable way to reason from a workload to a
data model. This ADR is that framework, so the choice is defensible each time it's made rather than
re-argued from scratch.

[AUTHOR: the real selections this framework is drawn from — a case where relational was kept under
pressure, and a case where a specific pressure justified moving off it.]

## Options Considered

Decide access pattern first; the data model follows from it.

```mermaid
flowchart TD
    Q1{"Multi-entity transactions +<br/>rich ad-hoc queries?"}
    Q1 -- Yes --> REL["Relational"]
    Q1 -- No --> Q2{"Document-shaped aggregate,<br/>read/written as a whole?"}
    Q2 -- Yes --> DOC["Document"]
    Q2 -- No --> Q3{"Massive write throughput,<br/>known access pattern, linear scale?"}
    Q3 -- Yes --> WIDE["Wide-column"]
    Q3 -- No --> Q4{"Simple key lookups,<br/>lowest latency?"}
    Q4 -- Yes --> KV["Key-value"]
    Q4 -- No --> REL
```

| Family | Data model | Consistency / transactions | Scale shape | Watch out for |
|--------|-----------|----------------------------|-------------|---------------|
| **Relational** | Normalized tables, joins | Strong ACID, multi-row transactions | Vertical + read replicas; sharding is manual | Write-scaling ceiling at very high volume |
| **Document** | Self-contained aggregates | Per-document atomicity; weaker cross-document | Horizontal | Cross-document joins/transactions pushed into app code |
| **Wide-column** | Partitioned wide rows | Tunable, eventual-leaning | Horizontal, linear, write-heavy | Must know your query patterns up front; ad-hoc queries are painful |
| **Key-value** | Opaque value by key | Usually none beyond single key | Horizontal, lowest latency | No querying beyond the key; it's a lookup, not a database of record |

## Decision

Apply the framework in this order, and stop at the first honest answer:

1. **Access pattern** — known and narrow, or ad-hoc and rich? This dominates everything below.
2. **Consistency & transactions** — do you need multi-entity ACID, or is per-record atomicity enough?
3. **Scale shape** — read-heavy, write-heavy, data volume, throughput. Only real pressure here
   justifies leaving relational.
4. **Data-model fit** — relational / document / wide / key-value follows from 1–3, not the reverse.
5. **Operational maturity** — the team's ability to run it is a first-class criterion, not an
   afterthought.

Default to relational and make the workload *earn* a move off it. Polyglot persistence is a cost —
every additional store is operational surface, expertise, and failure modes — so earn each one; do
not collect databases.

## Consequences

**What it buys.** Every database choice becomes a short, defensible derivation from the workload,
consistent across teams — and a future architecture-review agent can check a proposal against these
same criteria.

**What it costs — and when it's the wrong call.**

- **Discipline over reflex.** The framework only works if applied honestly; "we already use X
  everywhere" will masquerade as an access-pattern answer if you let it.
- **Relational-by-default is wrong** when a genuine scale or model pressure exists and the framework
  says so — a global, write-heavy, known-access-pattern workload forced onto a single relational
  primary is its own failure. [AUTHOR: the case where the pressure was real and you moved.]
- **Polyglot is wrong** when the second store is added for a marginal fit gain that doesn't cover
  its operational cost.

## Status

draft — awaiting author specifics and review. This framework is a candidate rule set for the future
architecture-review agent. Related: —
