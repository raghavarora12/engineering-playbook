<!--
HOW TO USE (delete this block when adopting):
- This is a gate, not a wish list. A service is not production-ready until every ★ item is a
  genuine "yes" — not "planned," not "mostly."
- If an item doesn't apply, strike it and say why in review. Silence is not an exemption.
-->

# Microservice Production-Readiness Checklist

A service is production-ready when you can answer these honestly — most of them *before* the
incident, not during it. The test for each item is "could you demonstrate this right now," not
"do we intend to."

## Ownership & contract
- [ ] ★ A single team owns this service, named in the service catalog / CODEOWNERS.
- [ ] ★ Its API contract is versioned and documented; breaking changes have a deprecation path.
- [ ] Upstream and downstream dependencies are known and documented.

## Observability
- [ ] ★ Structured logs with a request/correlation ID that crosses service boundaries.
- [ ] ★ RED metrics exposed (Rate, Errors, Duration) and on a dashboard.
- [ ] ★ Distributed tracing wired through the critical paths.
- [ ] Alerts fire on symptoms users feel (SLO burn), not on causes no one acts on.

## SLOs & reliability
- [ ] ★ An SLO exists, is owned, and matches the availability target this service actually needs
      (not a reflexive five nines — see ADR-002).
- [ ] An error budget is tracked and has a stated policy for when it's exhausted.
- [ ] Failure modes of each dependency are handled: timeouts, retries with backoff, circuit
      breakers. No unbounded waits.
- [ ] Load/capacity is understood: known limits and a scaling story.

## On-call & operations
- [ ] ★ There is an on-call rotation with a reachable escalation path.
- [ ] ★ A runbook exists for the top alerts — first responder can act without tribal knowledge.
- [ ] Deploys are automated, and rollback is one tested step.
- [ ] Config and secrets are managed outside the code; no secrets in the repo.

## Data ownership
- [ ] ★ The service owns its data; no other service writes its store directly.
- [ ] Backup and restore are defined *and restore has actually been tested*.
- [ ] Data retention and PII handling meet policy; sensitive fields are classified.

## Security & compliance
- [ ] AuthN/AuthZ enforced on every entry point; no implicitly trusted callers.
- [ ] Dependencies are scanned; a patching path exists for CVEs.
- [ ] Audit logging where the domain requires provable history.

## Delivery
- [ ] CI runs tests and blocks merge on failure.
- [ ] The service can be stood up in a non-prod environment from scratch, reproducibly.
