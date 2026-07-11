---
id: PRIN-003
title: Reliability is spent, not wished
status: draft
date: 2026-07-11
tags: [reliability, slo, error-budget]
related: [ADR-002]
---

# Reliability is spent, not wished

**Run an explicit error budget; trade feature velocity for reliability only when the budget says so.**

"Be more reliable" is a wish. An error budget makes reliability a quantity you spend on purpose,
turning a recurring argument into arithmetic everyone can see.

*Ignore it and:* reliability loses every prioritization argument to the next feature — right up
until an incident makes it win all of them at once, on the worst possible day. See [[ADR-002]].
