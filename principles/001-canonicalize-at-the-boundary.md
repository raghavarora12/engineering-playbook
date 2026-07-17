---
id: PRIN-001
title: Canonicalize at the boundary
status: draft
date: 2026-07-11
tags: [data-strategy, governance]
related: [ADR-001]
---

# Standardize and canonicalize at the boundary

**Standardize values and canonicalize the model once, where data enters the platform — as far left
as you can, never in every consumer.**

The differences between sources get reconciled *somewhere*. Doing it once, at the boundary — and
into the source itself via data contracts where you own it — makes lineage, governance, and audit
provable. Pushing it downstream means every consumer reinvents the same reconciliation, slightly
differently. (Standardizing left is near-universal; canonicalizing left is the judgment call —
see [[ADR-001]].)

*Ignore it and:* the numbers stop agreeing across systems, and no one can answer an auditor's "what
does this figure actually mean?" — because it means four things. See [[ADR-001]].
