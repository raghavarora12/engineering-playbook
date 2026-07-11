---
id: ADR-003
title: Multi-region active-active vs. active-passive
status: draft
tier: 1
date: 2026-07-11
tags: [reliability, multi-region, disaster-recovery, availability]
supersedes: null
superseded_by: null
related: [ADR-002]
---

# ADR-003 — Multi-region active-active vs. active-passive

> Active-active is sold as maximum availability, but it protects against a **narrower** set of
> failures than its price implies — and introduces its own. Active-passive with fast, tested
> failover is often the honest choice. Active-active earns its cost only when you need
> near-zero-RTO regional survivability, or you're already multi-region for latency.

## Context

Once a system needs to survive the loss of a whole region, the question is whether both regions
serve traffic at once (active-active) or one serves while the other stands ready (active-passive).
The instinct is that active-active is strictly better because "both are live." It isn't strictly
better — it's a different set of tradeoffs, and the expensive one.

[AUTHOR: the real system and topology, and the specific failure mode that forced this decision.]

## Options Considered

```mermaid
flowchart LR
    subgraph AP["Active-passive — one region serves"]
        direction TB
        lb1["Traffic"] --> ra["Region A (active)"]
        ra -. "replicate" .-> rb["Region B (standby)"]
    end
    subgraph AA["Active-active — both serve"]
        direction TB
        lb2["Traffic"] --> rc["Region C"]
        lb2 --> rd["Region D"]
        rc <-. "bidirectional<br/>replication" .-> rd
    end
```

| Option | Cost / complexity | Protects against | Introduces / when right |
|--------|-------------------|------------------|-------------------------|
| **Active-passive** (warm standby) | ~1.5× infra; failover is the hard part to keep tested | Total region loss, with a short, bounded failover gap (RTO minutes, RPO near-zero with sync replication) | Failover that silently rots if never exercised. Right when you need region survivability but can tolerate a minutes-long, controlled cutover. |
| **Active-active** | ~2×+ infra, *plus* the real cost: cross-region data consistency | Total region loss with near-zero RTO; also serves latency from both regions | Split-brain, conflict resolution, and the fact that it **replicates your bad deploy or data corruption to both regions instantly**. Right when zero-RTO regional survivability is mandatory or you're already multi-region for latency. |

The sharp point: most outages are not clean region losses. They are bad deploys, dependency
failures, and data corruption — and active-active faithfully propagates all three to every region.
It buys you resilience to the failure mode that is *least* common and asks you to solve
cross-region consistency to get it.

| | Active-passive | Active-active |
|---|---|---|
| Typical RTO | Minutes (failover) | Seconds / near-zero |
| Typical RPO | Near-zero (sync) to seconds (async) | Depends on conflict model |
| Hard problem | Keeping failover tested | Cross-region consistency |
| Steady-state cost | ~1.5× | ~2×+ |

## Decision

[AUTHOR: which you chose, for which system, and the failure mode that decided it.] The reasoning
that should drive it: choose active-active only if the RTO gap of active-passive is genuinely
unacceptable *and* you can solve — or sidestep, via partitioned regional ownership — cross-region
data consistency. Otherwise active-passive with a **failover you actually rehearse** delivers most
of the protection at a fraction of the cost and complexity.

## Consequences

**What it buys.** Survival of a region-level failure, at an RTO matched to the target set in
[ADR-002](002-availability-target.md).

**What it costs — and when it's the wrong call.**

- **Active-active** makes cross-region data consistency your permanent problem, and its cost is
  paid every day, not just during a disaster. It is the wrong call when a bounded failover gap is
  acceptable — which is more often than its reputation suggests.
- **Active-passive** is only as good as the last time you tested failover. An untested standby is a
  false sense of security, not a DR strategy. [AUTHOR: your failover test cadence / a time it was
  exercised for real.]
- **Both** are the wrong call for systems that don't need to survive region loss at all — single-
  region with multi-AZ is the honest stop for most services (see [ADR-002](002-availability-target.md)).

## Status

draft — awaiting author specifics and review. Multi-region failover behavior is demonstrated in the
**[Payments Resiliency Simulator](https://github.com/raghavarora12/payments-resiliency-simulator)**.
Related: [ADR-002](002-availability-target.md).
