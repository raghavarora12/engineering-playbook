---
id: PRIN-001
title: Canonicalize at the boundary
status: draft
date: 2026-07-11
tags: [data-strategy, governance]
related: [ADR-001]
---

# Canonicalize at the boundary

**Standardize and canonicalize data once, where it enters the platform — never in every consumer.**

The differences between sources get reconciled *somewhere*. Doing it once, at the boundary, makes
lineage, governance, and audit provable. Pushing it downstream means every consumer reinvents the
same reconciliation, slightly differently.

*Ignore it and:* the numbers stop agreeing across systems, and no one can answer an auditor's "what
does this figure actually mean?" — because it means four things. See [[ADR-001]].
