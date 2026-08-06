---
id: ADR-005
title: Coupling across domains — when a caller may wait, and when it must not
status: draft
tier: 2
date: 2026-07-11
tags: [architecture, messaging, coupling, event-driven, ai-agents]
supersedes: null
superseded_by: null
related: [ADR-004, ADR-006, ADR-008]
---

# ADR-005 — Coupling across domains: when a caller may wait, and when it must not

> Choose by coupling and failure semantics, not fashion. Request/response when the caller genuinely
> can't proceed without an answer. Event-driven when domains should evolve independently and the
> producer shouldn't know — or wait for — its consumers. Async is a liability precisely where you
> need an immediate, consistent answer. This question is fifteen years old and newly urgent: an
> agent chain is the same fan-out with a far worse tail, and the industry is relearning the answer
> at model latency.

## Context

By 2011, Netflix's API was taking in more than **1 billion calls a day**, fanning out at roughly
**1:6** to dozens of backing subsystems, with peaks over **100,000 dependency requests per
second**. Every one of those fan-out calls was synchronous — the API had to wait for an answer
before it could respond to the device asking for a home screen or a stream URL. That's a
defensible design until one dependency goes slow instead of cleanly down. A clean failure is easy;
a *slow* one is what actually happens, and because all six calls shared the same caller's thread
pool, one latent dependency was enough to exhaust it in seconds — silently blocking requests to
the other five dependencies, which were perfectly healthy. Netflix's fix, published in 2012 and
later open-sourced as **Hystrix**, was to isolate every dependency behind its own circuit breaker,
so one slow call could only ever cost its own slice of capacity.

Don't reach for Hystrix itself — Netflix put it in maintenance mode in 2018 and moved to adaptive
concurrency limits that infer a safe ceiling from live latency rather than from a threshold someone
tuned by hand a year ago. The library aged out; the isolation pattern didn't. That distinction is
worth holding onto, because the pattern is about to matter more than it ever did.

![Diagram: Netflix's API in 2011 fans out roughly 1-to-6 from a single caller to dozens of dependencies at up to 100,000 requests per second at peak. One dependency goes slow. Because all six share the caller's thread pool, that single slow dependency exhausts it within seconds, blocking requests to the other five healthy dependencies too. Fix: isolate each dependency in its own circuit breaker — the design that became Hystrix.](assets/005-netflix-cascading-failure.svg)

When one domain needs something from another, the communication style is a coupling decision, not a
transport detail. Request/response couples the caller's availability to the callee's. Event-driven
decouples them in time but trades away the immediate, consistent answer. Picking the wrong one
shows up later as either cascading failures (sync where you needed async) or impossible-to-reason-
about eventual consistency (async where you needed a straight answer).

The physics matter. A synchronous call is not free: an in-datacenter round trip is ~**0.5 ms**, but
a cross-continent round trip is ~**150 ms** (Dean; "Latency Numbers Every Programmer Should Know").
Chain several synchronous calls across domains and those numbers add — and, worse, the whole chain
inherits the **tail latency** of its slowest dependency ("The Tail at Scale," Dean & Barroso): a
service fanning out to many synchronous dependencies is as slow as the slowest one on any given
request, and its p99 degrades fast. Bronson et al. (formerly Meta/Facebook) gave this pattern a
name in 2021 — **metastable failure**: a system that is stable under normal load can get trapped in
a self-sustaining degraded state after a small trigger, typically because retries and synchronous
fan-out amplify a load spike instead of absorbing it ("Metastable Failures in Distributed Systems,"
HotOS 2021). Circuit breakers and backoff aren't defensive-coding hygiene — they're the mechanism
that keeps a synchronous chain from tipping into exactly that trap.

### The same argument, at model latency

All of that was settled engineering by about 2015, which is why this reads like a dated debate. It
isn't. Agentic systems have reintroduced the identical failure mode with the constants moved by
three orders of magnitude, and they have done it to teams who never had to learn the first lesson.

A model call is a synchronous dependency measured in seconds, not milliseconds, and its latency
varies wildly between two requests with the same shape. An agent step that plans, retrieves, calls
two tools and verifies is a fan-out of exactly the Netflix shape — except every leg is thousands of
times slower and none of them has a predictable tail. Chain five agents that call each other
directly and you have multiplied five availabilities and summed five tails, which is the arithmetic
Dean & Barroso described, now applied to dependencies that were never fast to begin with.

The industry is converging on the answer it reached last time: put a bus between them. Microsoft
rebuilt AutoGen from scratch in v0.4 (full release early 2025) around an asynchronous, event-driven
actor model, on the grounds that it "decouples how the messages are delivered between the agents from
how the agents handle them." Confluent's guidance is blunter — "in a production-grade multi-agent
system, agents never call each other directly," because "direct API or function calls between agents
create tight coupling, synchronous failure propagation, and implicit dependencies," and "over time,
the system collapses into a distributed monolith."

![Diagram: the same fan-out shape drawn twice, side by side. On the left, Netflix in 2011 — one caller fanning out roughly one to six across dozens of backing subsystems, at more than a billion calls a day and over 100,000 dependency requests per second at peak, where a round trip inside the datacentre is about half a millisecond and across a continent about 150 milliseconds, and the tail is predictable. One dependency goes slow and exhausts the shared thread pool within seconds, blocking the healthy five. On the right, an agent chain in 2026 — one agent fanning out across plan, retrieve, two tool calls and verify, where every leg is a model call measured in seconds rather than milliseconds and none of them has a dependable tail, so five availabilities multiply and five tails sum. A badge between the two panels reads: same shape. Along the bottom, the fix is the same both times — put a bus between them: Microsoft rebuilt AutoGen v0.4 on an asynchronous event-driven actor model that decouples how messages are delivered from how agents handle them, and Confluent states that in a production-grade multi-agent system agents never call each other directly, because over time the system collapses into a distributed monolith. The closing caption reads: the coupling question didn't age — the callees just got slower.](assets/005-same-shape-new-constants.svg)

"Distributed monolith" is the tell — it is the failure microservices spent a decade learning to
name, arrived at again from a completely different direction. The coupling question didn't age. The
callees just got slower, and the cost of getting it wrong went up.

[AUTHOR: the real cross-domain decision, the domains involved, and what the rejected option cost.]

## Options Considered

```mermaid
sequenceDiagram
    participant A as Domain A
    participant B as Domain B
    Note over A,B: Request/response (synchronous)
    A->>B: request
    rect rgb(28, 21, 12)
    B-->>A: response (A blocks until it returns)
    end
```

```mermaid
sequenceDiagram
    participant A as Domain A
    participant Bus as Event bus
    participant B as Domain B
    participant C as Domain C
    Note over A,C: Event-driven (asynchronous)
    A->>Bus: publish event
    rect rgb(13, 26, 28)
    Bus-->>B: deliver
    Bus-->>C: deliver
    Note over A: A does not wait or know who consumes
    end
```

| Option | Coupling | Failure behavior | Cost / when right |
|--------|----------|------------------|-------------------|
| **Request/response** (REST/gRPC) | Temporal — caller needs callee up *now* | Callee down → caller blocked; risk of cascading failure without timeouts/circuit breakers; tail-latency amplification across a fan-out | Simple to reason about; right when the caller truly cannot proceed without the answer (e.g. authorize this payment). |
| **Event-driven** (pub/sub) | Decoupled — producer doesn't know consumers | Consumer down → events buffer; producer unaffected | Resilience and independent evolution, paid for with eventual consistency, ordering/idempotency burden, and harder tracing/debugging. Right when domains should evolve and fail independently. |

The rows don't change for agents; only the numbers do. A synchronous agent-to-agent call is the
first row with a callee that takes seconds and has no dependable tail — which is why the row's
failure column, not its simplicity column, should decide it.

The decision aid:

```mermaid
flowchart TD
    Q1{"Does the caller need the answer<br/>to make its very next move?"}
    Q1 -- "Yes — e.g. authorize this payment" --> RR["✓ Request/response<br/>+ timeouts, retries with backoff,<br/>circuit breakers"]
    Q1 -- "No — other domains just<br/>need to know it happened" --> EV["✓ Event-driven<br/>— publish and let consumers react"]
    RR -.-> WARN["⚠ Fan out sync calls unguarded<br/>= metastable failure risk"]

    classDef decision fill:#11151c,stroke:#c7cfe0,stroke-width:1.5px,color:#e7ecf7
    classDef good fill:#0d1a1c,stroke:#2dd4bf,stroke-width:2px,color:#9ff3e6
    classDef warn fill:#1c150c,stroke:#ffa53d,stroke-width:2px,color:#ffd9a0
    class Q1 decision
    class RR,EV good
    class WARN warn
```

## Decision

[AUTHOR: which you chose, for which interaction.] The rule that should decide it: **does the caller
need the answer to make its next move?** If yes — an authorization, a balance check, a synchronous
validation — use request/response and protect it with timeouts, retries with backoff, and circuit
breakers. If no — a state change other domains merely need to *know about* — publish an event and
let them react on their own schedule.

Most cross-domain communication in a payments platform is the second kind (something happened),
which is why event-driven is the backbone — but the moments that are the first kind (is this
authorized?) must stay synchronous, and getting that boundary wrong is the expensive mistake.

The same rule decides agent topology, and it decides it harder. When the callee is a model, "does
the caller need the answer to make its next move?" has to be asked per step rather than per service,
and the honest answer is *no* far more often than agent frameworks encourage — a plan step, an
enrichment, a verification, a downstream notification are all things other components need to
*know happened*, not things the caller must block on. Reserve synchronous agent calls for the step
whose result genuinely gates the next one, and give every one of them a timeout you chose
deliberately, because the default is usually the model provider's, not yours.

## Consequences

**What it buys (event-driven).** Domains that fail and evolve independently; natural buffering under
load; new consumers added without touching the producer.

**What it costs — and when it's the wrong call.**

- **Eventual consistency and debugging cost.** "Where did this state come from?" spans services and
  time. You take on ordering, idempotency, and distributed tracing as standing obligations.
- **Async is wrong when you need an answer now.** Modeling an authorization as fire-and-forget to
  "decouple" it is how you get a system that can't tell you whether a payment went through.
- **Request/response is wrong** as the default for everything — sync-calling a chain of domains turns
  one slow dependency into a system-wide outage. [AUTHOR: an instance where the coupling choice
  caused, or prevented, a cascading failure.]
- **Async costs more in an agent system than in a service one.** A bus between agents buys the
  isolation above, but you now have to reconstruct a single logical "turn" from events scattered
  across topics and time — and the thing you are debugging is non-deterministic. Distributed tracing
  stops being a nice-to-have the moment a run you cannot reproduce spans four agents.

## Status

draft — awaiting author specifics and review. The event-driven patterns here are demonstrated in the
**[Payments Resiliency Simulator](https://github.com/raghavarora12/payments-resiliency-simulator)**.
Once that event-driven path has to survive the loss of a region, the log's own failure semantics take
over — that is [ADR-006](006-multi-region-kafka-high-availability.md). This ADR takes a position on
how agents should be *wired*;
[ADR-008](008-ai-assisted-engineering-adoption.md) takes one on how their adoption should be
*measured* — the architectural and organizational halves of the same arrival.
Related principle: [[choose-coupling-on-purpose]].

## References

- Dean & Barroso — [The Tail at Scale](https://research.google/pubs/the-tail-at-scale/), CACM 2013.
- [Latency Numbers Every Programmer Should Know](https://gist.github.com/jboner/2841832).
- Newman — *Building Microservices* (temporal vs. implementation coupling).
- Netflix Technology Blog (Ben Christensen) — [Fault Tolerance in a High Volume, Distributed System](https://netflixtechblog.com/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a) (2012) — the 1B+ calls/day, ~1:6 fan-out, 100k+ req/sec cascading-failure problem that led to Hystrix.
- Bronson, Aghayev, Charapko, Zhu (formerly Meta/Facebook) — [Metastable Failures in Distributed Systems](https://sigops.org/s/conferences/hotos/2021/papers/hotos21-s11-bronson.pdf), HotOS 2021 — how retries and synchronous fan-out turn a small trigger into a self-sustaining, hard-to-predict outage.
- [Netflix/Hystrix](https://github.com/Netflix/Hystrix) — in maintenance mode since November 2018; Netflix moved to adaptive concurrency limits that react to live performance instead of hand-tuned thresholds. Cited here for the pattern, not as a recommendation.
- Microsoft Research — [AutoGen v0.4: Reimagining the foundation of agentic AI](https://www.microsoft.com/en-us/research/articles/autogen-v0-4-reimagining-the-foundation-of-agentic-ai-for-scale-extensibility-and-robustness/) (preview Fall 2024, full release early 2025) — the rewrite onto an asynchronous, event-driven actor model, and why message delivery was decoupled from message handling.
- Confluent (Mohtasham Sayeed Mohiuddin) — [Autonomous agentic event-driven systems architecture](https://www.confluent.io/blog/autonomous-agentic-event-driven-systems-architecture/) (25 May 2026) — why agents should never call each other directly, and how direct calls collapse a multi-agent system into a distributed monolith.

---

*Coupling decisions look obvious on a whiteboard and rarely stay that way once the failure
semantics show up. If you're weighing one, I'd genuinely enjoy comparing notes —
[get in touch](../README.md#get-in-touch).*
