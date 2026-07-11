---
id: PRIN-004
title: Design for the common failure, not the cinematic one
status: draft
date: 2026-07-11
tags: [reliability, resilience]
related: [ADR-003]
---

# Design for the common failure, not the cinematic one

**Optimize first for the failures you actually get — bad deploys, dependency timeouts, data
corruption — not the rare total-region loss.**

Teams over-invest in dramatic failure modes and under-invest in the mundane ones that cause most
downtime. Active-active survives a region loss and then faithfully replicates your bad deploy to
every region.

*Ignore it and:* you spend the disaster-recovery budget on the 1% failure and keep getting paged
for the 99%. See [[ADR-003]].
