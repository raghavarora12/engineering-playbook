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
| [005](adrs/005-event-driven-vs-request-response.md) | Event-driven vs. request/response across domains | Choose by coupling and failure semantics, not fashion; async is a liability where you need a synchronous answer. | 2 | planned |
| [006](adrs/006-kafka-for-payment-events.md) | Kafka for payment event processing | Right for durable, replayable, high-throughput event flow — and a costly mistake used as a queue or an RPC bus. | 2 | planned |
| [007](adrs/007-database-selection-framework.md) | Database-selection framework | A framework for relational vs. document vs. wide-column vs. key-value — not a ranking. | 2 | planned |

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

→ [`/principles`](principles/) *(planned)*

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
