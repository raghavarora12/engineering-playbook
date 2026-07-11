---
id: PRIN-006
title: Match the datastore to the access pattern, not the trend
status: draft
date: 2026-07-11
tags: [databases, data-modeling]
related: [ADR-007, ADR-006]
---

# Match the datastore to the access pattern, not the trend

**Pick a database from how the data is read and written, defaulting to relational until a real
pressure forces a move.**

Access pattern determines fit; fashion doesn't. "NoSQL by default" — or "Kafka for everything" —
just rebuilds joins, transactions, and queues in application code, badly.

*Ignore it and:* the wrong store shows up as query gymnastics, hand-rolled consistency, and a
migration you'll pay for with interest later. See [[ADR-007]] and [[ADR-006]].
