<!-- SKELETON (Phase 1). Index statuses read "planned" until each ADR lands; finalize the
     table and links in Phase 1-finalize once content exists. Confirm the Simulator URL. -->

# Engineering Playbook

**How I make architecture decisions — the tradeoffs made explicit, and owned.**

Most senior engineering repos prove *what* someone used: Kafka, Kubernetes, a cloud logo.
Very few prove *why* they chose it over the alternatives, what that choice cost, and when it
would have been wrong. This repository is the second thing. It is a curated set of
architecture decision records, principles, and reusable standards — the reasoning, not the
résumé.

It is deliberately small. Seven decision records, not fifty. Curation is the point.

---

## Decision records

The centerpiece. Each ADR states a position you could disagree with, lays the options side by
side, and names the cost of the call. Data-strategy and reliability decisions come first —
they are the ones backed by the most scar tissue.

> *Index is provisional while the repo is built out; statuses move to `accepted` as each is
> reviewed and promoted.*

| # | Decision | The position | Tier | Status |
|---|----------|--------------|------|--------|
| [001](adrs/001-canonical-data-model-at-ingestion.md) | Standardize & canonicalize at ingestion vs. transform-on-read | Reconcile once at the boundary — shift data quality left — instead of paying every consumer to reshape raw data forever. | 1 | planned |
| [002](adrs/002-availability-target.md) | Choosing the right availability target | Five nines is the right call for a payment network and the wrong one for most systems — each nine is ~10× the cost. | 1 | planned |
| [003](adrs/003-multi-region-active-active-vs-active-passive.md) | Multi-region active-active vs. active-passive | Active-active protects against a different, narrower set of failures than its price implies; passive is often the honest choice. | 1 | planned |
| [004](adrs/004-operational-vs-analytical-data-planes.md) | Converging operational & analytical data planes | Convergence earns its complexity only under live latency pressure; the OLTP/OLAP split with a pipeline is still usually right. | 1 | planned |
| [005](adrs/005-coupling-across-domains.md) | Coupling across domains — the question agent chains made urgent again | Choose by coupling and failure semantics, not fashion; async is a liability where you need a synchronous answer — and agent chains have made the old question urgent again. | 2 | planned |
| [006](adrs/006-multi-region-kafka-high-availability.md) | Multi-region Kafka — the duplicate window you can't design away | Cross-cluster replication is at-least-once by construction; the topology only sets how wide the window is, and the ledger closes it, not the broker. | 2 | planned |
| [008](adrs/008-ai-assisted-engineering-adoption.md) | AI-assisted engineering adoption **(WIP)** | Buying the licences isn't the decision — the decision is what you agree to be measured on, because AI raises throughput and instability together and self-report is the one instrument it breaks. | 2 | in progress |

---

## Where to start

- **Engineering leader (VP / Director / CTO)** — read ADR-001, ADR-002, ADR-003. If the
  tradeoffs don't ring true, nothing else here will.
- **Staff / principal engineer** — pick the decision you disagree with and read it in full.
  The "Consequences" section is where the honesty lives.

---

## Principles

A small set (8–12) of opinionated principles — one-line statement, short rationale, and the
real consequence of ignoring it. Specific enough to argue with.

*Drafted and in review — they publish here once they've earned their place.*

## Standards & templates

Genuinely fillable artifacts, not decoration: the [ADR template](templates/adr-template.md),
plus an RFC/design-doc template, a microservice-readiness checklist, a tech-debt
classification framework, and an engineering-metrics guide (what to measure, what not to, and
why vanity metrics mislead).

→ [`/templates`](templates/)

---

## Related work

**[Payments Resiliency Simulator](https://github.com/raghavarora12/payments-resiliency-simulator)**
— the working system. Several ADRs here (availability, multi-region, event-driven) describe
patterns that repo demonstrates in running code. This playbook is the reasoning; the Simulator
is the proof it runs.

## Get in touch

These are positions, not conclusions — they are meant to be argued with. If you're weighing one of
these tradeoffs in your own landscape, or you think a call here is plainly wrong, I'd genuinely
enjoy the conversation. Availability targets, multi-region failover, and the dependency chains that
quietly cap both are where I've spent the most time.

**LinkedIn** — https://www.linkedin.com/in/raghav12/

---

## Provenance

AI assisted the drafting of this repository. The judgment, the positions, and the tradeoffs are
mine — drawn from real systems, not generated to sound plausible. Making that boundary explicit
is itself the point: this is what AI-enabled engineering leadership looks like when it's done
with integrity.
