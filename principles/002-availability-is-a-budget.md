---
id: PRIN-002
title: Availability is a budget, not a trophy
status: draft
date: 2026-07-11
tags: [reliability, availability]
related: [ADR-002]
---

# Availability is a budget, not a trophy

**Choose the number of nines the business actually needs; more is not better, it's more expensive.**

Each nine is roughly 10× the cost and complexity of the last. A target you don't need spends
redundancy, operational load, and feature velocity the business would never knowingly buy.

*Ignore it and:* you over-provision an internal tool to five nines and tax every release and every
on-call shift for reliability no customer asked for — or under-provision what mattered and learn it
in an outage. See [[ADR-002]].
