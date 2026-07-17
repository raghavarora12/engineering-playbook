---
id: ADR-004
title: Converging operational and analytical data planes vs. keeping them separate
status: draft
tier: 1
date: 2026-07-11
tags: [data-strategy, htap, oltp-olap, streaming]
supersedes: null
superseded_by: null
related: [ADR-001]
---

# ADR-004 — Converging operational & analytical data planes vs. keeping them separate

> Convergence (HTAP / streaming lakehouse) earns its complexity only under genuine live-latency
> pressure — decisioning on data fresher than a pipeline can deliver. For most workloads the
> classic OLTP/OLAP split with a pipeline between them is still the right, cheaper, more operable
> call. "Convergence is just the modern way" is a hot take, not an architecture.

## Context

In November 2024, Snowflake shipped the industry's most ambitious bet on convergence yet: Unistore
hybrid tables, general availability, one engine meant to serve transactional and analytical
workloads at once. Within months the real ceiling showed up — throughput throttled to roughly
**1,000 operations per second**, active data capped at **500 GB**, and the system shipped without
materialized views, cloning, streams, or cross-region replication. The verdict came from the
vendors themselves: in May 2025, Databricks acquired **Neon**, a dedicated Postgres provider; a
month later, in June 2025, Snowflake acquired **Crunchy Data**, another dedicated Postgres
provider. Two of the industry's biggest analytical platforms independently concluded that
extending a columnar analytical engine into transactional territory does not work, and paired a
separate, purpose-built OLTP engine instead of doubling down on one engine doing both.

![Timeline diagram: in November 2024 Snowflake's Unistore hybrid tables reach general availability, the industry's most ambitious single-engine HTAP bet yet; within months real limits surface — throughput throttled to roughly 1,000 operations per second, a 500 gigabyte active-data cap, and no materialized views, cloning, streams, or cross-region replication; in May 2025 Databricks acquires Neon, a dedicated Postgres provider; in June 2025 Snowflake acquires Crunchy Data, another dedicated Postgres provider. Verdict: compose two engines, don't converge into one.](assets/004-htap-decade-verdict.svg)

Operational systems (OLTP) are tuned for many small, consistent transactions; analytical systems
(OLAP) for large scans over history. The long-standing pattern keeps them separate and moves data
between them with a pipeline. The newer pitch is convergence: one store that serves both, so
analytics run on live transactional data with no pipeline lag.

The question is not which is more modern. It is whether the freshness convergence buys is worth
the isolation it gives up. The honest read after a decade of HTAP is sobering, and Snowflake's
Unistore is only the latest data point: unifying both workloads *in one engine* tends to collapse
workload isolation and compromise performance for both. The lakehouse response (Zaharia et al.,
CIDR 2021) is telling — it unifies at the **storage** layer and keeps **compute** separate,
precisely so analytical scans can't starve transactions. That distinction — unify storage, isolate
compute — is the crux of this decision, and it's exactly the shape every major platform has
converged on: ClickHouse, Snowflake, Databricks, AWS, Google, and Microsoft all now ship composed
architectures — dedicated OLTP and OLAP engines joined by CDC or zero-ETL replication — rather than
a single engine doing both. As Mooncake Labs co-founder Zhou Sun put it in 2025: "it's still HTAP;
but through composition instead of consolidation of databases."

[AUTHOR: the real converge-or-separate decision, the system, and the latency-sensitive use case
(e.g. fraud/risk decisioning) if one drove it.]

## Options Considered

```mermaid
flowchart LR
    subgraph SEP["✓ Separate planes + pipeline"]
        direction LR
        o1["OLTP"] -->|"CDC / stream / ETL"| p1[("Pipeline")]
        p1 --> a1["OLAP / warehouse"]
    end
    subgraph LAKE["~ Unified storage, separate compute (lakehouse)"]
        direction LR
        o2["OLTP"] --> st[("Open storage<br/>table format")]
        st --> ac1["Txn compute"]
        st --> ac2["Analytics compute"]
    end
    subgraph CON["⚠ Converged engine (HTAP)"]
        direction LR
        o3["One engine serves<br/>txn + analytics"]
    end

    classDef good fill:#0d1a1c,stroke:#2dd4bf,stroke-width:2px,color:#e7ecf7
    classDef warn fill:#1c150c,stroke:#ffa53d,stroke-width:2px,color:#e7ecf7
    classDef bad fill:#1c0f0c,stroke:#ff5c4d,stroke-width:2px,color:#e7ecf7
    class o1,p1,a1 good
    class o2,st,ac1,ac2 warn
    class o3 bad
    style SEP fill:#0d1a1c,stroke:#2dd4bf,stroke-width:2px,color:#9ff3e6
    style LAKE fill:#1c150c,stroke:#ffa53d,stroke-width:2px,color:#ffd9a0
    style CON fill:#1c0f0c,stroke:#ff5c4d,stroke-width:2px,color:#ffb4a8
```

| Option | Data freshness | Complexity / cost | Isolation & blast radius | When it's right |
|--------|----------------|-------------------|--------------------------|-----------------|
| **Separate + pipeline** | Minutes to hours (pipeline lag) | Two mature systems + a pipeline to operate | Strong: analytical scans can't starve OLTP; scale each independently | The default. Analytics tolerate lag; you want isolation and mature tooling. |
| **Unified storage, separate compute** | Seconds to minutes | One storage layer, multiple engines | Strong on compute, shared on storage — the pragmatic middle | You want one governed copy of data without coupling the two workloads' compute. |
| **Converged engine (HTAP)** | Live / sub-second | One system, less mature, more expensive | Weak: mixed workload contends on one engine; analytics share OLTP's blast radius | Decisions must be made on data fresher than the pipeline delivers, *and* that freshness is worth more than the isolation lost. |

The decision aid:

```mermaid
flowchart TD
    Q1{"Must a decision be made on data<br/>fresher than your pipeline latency?"}
    Q1 -- No --> SEP2["✓ Separate planes + pipeline<br/>— the default, cheapest, most isolated"]
    Q1 -- Yes --> Q2{"Does that freshness's value<br/>exceed the isolation you'd give up?"}
    Q2 -- No --> SEP2
    Q2 -- Yes --> Q3{"Can unified storage / separate<br/>compute (lakehouse) deliver the<br/>freshness you actually need?"}
    Q3 -- Yes --> LAKE2["~ Unified storage, separate compute"]
    Q3 -- No --> CON2["⚠ Converged engine (HTAP) —<br/>earn it deliberately, eyes open"]

    classDef decision fill:#11151c,stroke:#c7cfe0,stroke-width:1.5px,color:#e7ecf7
    classDef good fill:#0d1a1c,stroke:#2dd4bf,stroke-width:2px,color:#9ff3e6
    classDef warn fill:#1c150c,stroke:#ffa53d,stroke-width:2px,color:#ffd9a0
    classDef bad fill:#1c0f0c,stroke:#ff5c4d,stroke-width:2px,color:#ffb4a8
    class Q1,Q2,Q3 decision
    class SEP2 good
    class LAKE2 warn
    class CON2 bad
```

## Decision

[AUTHOR: which you chose, for which system.] The test that should decide it: convergence into one
engine wins only when **the decision has to be made on data fresher than your pipeline latency, and
the value of that freshness exceeds the isolation you give up.** Live fraud/risk decisioning on
in-flight transactions is the canonical case that clears that bar. A daily revenue dashboard is not
— paying HTAP's contention and operational cost to make it "live" is complexity with no buyer.

For most "we want fresher analytics" needs, the honest answer is not a converged engine but either
a tighter pipeline (CDC/streaming, cutting lag from hours to seconds) or the unified-storage /
separate-compute shape — freshness without handing your transactional path a noisy neighbor.

## Consequences

**What it buys (convergence).** Decisions on live data without a pipeline in the critical path —
genuinely valuable where milliseconds of staleness change the answer.

**What it costs — and when it's the wrong call.**

- **Contention and blast radius.** In a single engine, a heavy analytical scan can now hurt your
  transactional path. You've coupled two workloads that separation kept apart.
- **Operational maturity and cost.** Converged HTAP engines are younger and pricier than the
  battle-tested OLTP and OLAP stacks they replace.
- **When it's simply wrong:** any workload whose analytics tolerate minutes of lag — which is most
  of them. Convergence there buys freshness nobody needs at the price of isolation everybody wants.
- **Keeping them separate** costs you freshness and a pipeline to run — the right price to pay unless
  live latency is genuinely load-bearing. [AUTHOR: the freshness SLA that actually mattered for your
  case.]

## Status

draft — awaiting author specifics and review. Builds on
[ADR-001](001-canonical-data-model-at-ingestion.md) — canonicalized data is what makes either plane
trustworthy. Related: [ADR-001](001-canonical-data-model-at-ingestion.md).

## References

- Zaharia, Ghodsi, Xin, Armbrust — [Lakehouse: A New Generation of Open Platforms](https://people.eecs.berkeley.edu/~matei/papers/2021/cidr_lakehouse.pdf), CIDR 2021.
- [HTAP: Still the Dream, a Decade Later](https://medium.com/@danthelion/htap-still-the-dream-a-decade-later-9d168f07c759) (why one-engine convergence keeps disappointing).
- ClickHouse — [Unifying OLTP and OLAP: HTAP databases, zero-ETL, and best-of-breed architectures](https://clickhouse.com/resources/engineering/unifying-oltp-and-olap) (2025) — Snowflake Unistore's real limits (~1,000 ops/sec, 500GB cap, missing features); Databricks acquiring Neon (May 2025) and Snowflake acquiring Crunchy Data (June 2025) instead of extending HTAP further.
