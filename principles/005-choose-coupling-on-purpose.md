---
id: PRIN-005
title: Choose coupling on purpose
status: draft
date: 2026-07-11
tags: [architecture, coupling]
related: [ADR-005]
---

# Choose coupling on purpose

**Use request/response when the caller needs the answer to proceed; use events when domains should
evolve and fail independently.**

Coupling is a design decision, not a transport default. Pick it deliberately from the failure
semantics you want, not from whatever style the last service used.

*Ignore it and:* sync-by-default turns one slow dependency into a system-wide outage; async-by-
default gives you a system that can't tell you whether the payment actually went through. See
[[ADR-005]].
