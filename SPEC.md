  # SPEC — Engineering Playbook & Decision Records

**Repository:** `engineering-playbook` (github.com/raghavarora12/engineering-playbook)

> This is a **specification of intent**, not a build plan. It defines *what* this
> repository is, *why* it exists, *who* it is for, and the *standard* it must meet.
> The ordered build steps live in a separate `PLAN.md`, authored after this SPEC is agreed.

---

## 1. Purpose & Intent

**What this is:** A public, curated engineering playbook — architecture principles,
decision records, standards, and reusable templates — that demonstrates
*director-level engineering and architecture judgment*, not tool familiarity.

**Why it exists:** Most senior engineering CVs and repos prove *what* someone used
(Kafka, Kubernetes, AWS). Very few prove *why* they chose it over the alternatives,
with the tradeoffs made explicit and owned. This repository closes that gap. It is
designed to let a recruiter grasp seniority in a 30-second README skim, and let a
technologist or engineering leader verify depth by reading the decision records.

**What success looks like:**
- A recruiter reads the README and thinks: *"This is how a Director thinks."*
- A staff/principal engineer reads three ADRs and thinks: *"These are real tradeoffs,
  made by someone who has actually been burned by the wrong choice."*
- Nothing in the repo reads as generic best-practice filler or AI-generated boilerplate.

**Explicit non-goals:**
- Not a tutorial, not a framework, not a "best practices" listicle.
- Not a comprehensive encyclopedia — curation and opinion are the point; breadth is not.
- Not a duplicate of the Payments Resiliency Simulator. This repo is domain-agnostic
  *reasoning and standards*; the Simulator is a working system. They cross-link, they
  do not overlap.

---

## 2. Audience

Written for three readers simultaneously, in priority order:

1. **Technical hiring managers / engineering leaders (VP/Director/CTO)** — assessing
   judgment, breadth of experience, and whether this person can set standards for an org.
2. **Senior technologists (staff/principal engineers)** — pressure-testing whether the
   tradeoffs are real and correctly reasoned, or superficial.
3. **Recruiters (non-technical)** — need to grasp level and relevance fast, from README
   and structure alone.

Every artifact must serve reader 1 or 2 on depth, while remaining legible to reader 3
on structure.

---

## 3. Scope — WHAT is included

### 3.1 Architecture Principles
A small set (target 8–12, not 50) of opinionated principles that reflect how the author
actually leads. Each principle: one-line statement + short rationale + a real consequence
of ignoring it. Principles must be specific enough to disagree with.

### 3.2 Architecture Decision Records (ADRs)
The centerpiece. Each ADR follows a consistent template (context → options considered →
decision → consequences → status). Target 6–10 high-quality ADRs over quantity.

ADRs are tiered by how differentiated they are. **Data-strategy ADRs are the priority
for v1** — they signal Director/Enterprise-Architect judgment (landscape-level ownership)
rather than senior-IC technology selection, and are the ones most directly backed by the
author's real data-governance and payments-platform experience.

**Tier 1 — Data-strategy & reliability (differentiators, ship first):**
- **Canonical data model at ingestion vs. transform-on-read downstream** — why
  standardising and canonicalising data at the boundary (once) beats letting each
  consumer reshape it (many times), and what lineage, reconciliation, governance, and
  audit exposure cost you when you don't. Anchored in real metadata-management /
  lineage-tracking / reference-data experience.
- **Converging operational and analytical data planes vs. keeping them separate** — when
  a unified store (HTAP / streaming lakehouse) earns its complexity (e.g. latency-
  sensitive fraud/risk decisioning on live data), and when the classic OLTP/OLAP split
  with a pipeline between them is still the right call. Must state the tradeoff
  honestly — this is a live industry debate, not a "convergence is the modern way" hot take.
- **Choosing the right availability target — why 99.999% is sometimes wrong** — each nine
  is ~10x the cost and complexity; five nines (~5 min/year downtime) is the right call
  for a payment network (regulatory, financial, and trust consequences of downtime) and
  the wrong call for many systems that should honestly stop at three or four. Frames the
  author's real 99.999% track record as a *justified decision*, not an achievement.
  The 40→<5 monthly incident reduction lives here as the lived evidence — via SLO /
  error-budget discipline, and the explicit tradeoff of feature velocity for reliability.
  Cross-links to the Payments Resiliency Simulator as the working demonstration of the
  patterns. (If this runs long, split the error-budget mechanism into its own ADR — do
  not pre-commit to two.)
- **Multi-region active-active vs. active-passive** — cost, complexity, and the specific
  failure modes each actually protects against.

**Tier 2 — Technology judgment (solid, supporting):**
- Multi-region high availability for Kafka — stretched cluster vs. mirrored active-active vs.
  mirrored active-passive vs. producer dual-write, and what each does to publishers, consumers, replay, reconciliation, and
  dead-letter handling. The position is a consequence, not a topology: cross-cluster replication
  is at-least-once by construction, so the topology only sets how wide the duplicate window is.
  Distinct from the Tier 1 multi-region ADR, which decides the *data-plane* topology; this one
  decides what happens to the *event log* and the consumer's position in it.
- Coupling across domains — when a caller may wait, and when it must not. Framed as a coupling
  and failure-semantics decision rather than an event-driven-vs-REST debate, because agent
  chains have made the same question urgent again at model latency.
- One database-selection framework ADR (relational vs. document vs. wide-column vs.
  key-value) — a decision framework, not a ranking
- **AI-assisted engineering across teams — why adoption is a measurement decision, not a tooling
  one.** The org-level counterpart to the Tier 1 reliability ADRs: DORA 2025 finds AI raises
  throughput *and* instability and acts as an amplifier of an organization's existing strengths
  and dysfunctions, while METR's randomized trial found experienced developers were measurably
  slower with AI while believing they were faster. The position that follows — you cannot steer
  this on developer perception or adoption counts, so the decision is which outcome you commit to
  measuring. Anchored in the author's real experience leading AI-assisted delivery adoption across
  teams; must state honestly where the evidence is contested and where it is already dated.

**Tier 3 — Hold for v2, only if genuinely backed by real scars (do not include on
generic best-practice grounds alone):**
- REST vs. gRPC — the decision boundary, not the dogma
- When serverless is right, and when it quietly becomes a liability
- Kubernetes vs. managed (e.g. GKE/EKS) vs. not-orchestrated-at-all
- Kafka as the right and wrong tool — the log-is-not-a-queue argument (durable replayable log vs.
  task queue vs. RPC bus). Drafted for v1 and cut: the endorsement half is well-covered ground,
  and the ADR only earns its place if the author brings the real case where reaching for Kafka
  cost more than it returned. Revive it with that scar, or not at all.

v1 ships all of Tier 1 (four ADRs) plus Tier 2 (four ADRs) — eight strong ADRs. The AI-adoption
ADR was added to Tier 2 after the initial shortlist: it is the one place the repo takes a position
on the defining platform shift of the period, and it earns its place only because the author owns
the org-adoption experience behind it — not because the topic is current. Tier 3
is deferred unless the author has strong, specific opinions to bring — thin coverage of a
Tier 3 topic dilutes the sharper ADRs around it and should be cut rather than included
for completeness.

### 3.3 Standards & Templates
Reusable, genuinely usable artifacts (not decorative):
- ADR template
- RFC / design-doc template
- Microservice readiness checklist (observability, SLOs, on-call, data ownership)
- Tech-debt classification & prioritization framework
- Engineering metrics: what to measure, what NOT to measure, and why vanity metrics mislead

### 3.4 Platform & Ways-of-Working (folded in from the "Platform Blueprint" idea)
Strategy-level thinking, as focused sections rather than a separate repo:
- Platform vision & the "golden path" concept
- Internal Developer Platform (IDP) shape and self-service boundaries
- Observability, GitOps, and self-service as first-class platform concerns

---

## 4. Standard & Quality Bar — the HOW (principles of construction)

These are the rules the build must obey. They are the difference between a portfolio
piece that impresses and one that gets quietly dismissed.

1. **Opinion over survey.** Every document takes a position. If a reader cannot disagree
   with it, it is too generic and must be cut or sharpened.
2. **Experience-anchored.** Claims are grounded in real, specific situations from the
   author's background (payments networks, core banking, 100+ engineer orgs, 99.999%
   availability, the 40→<5 incident reduction).
   Placeholders marked `[AUTHOR: …]` MUST be filled with real specifics before publish —
   they are load-bearing, not optional flavor.
3. **Tradeoffs are mandatory.** No decision record is complete without honestly stating
   what the choice costs and when it would be the wrong call.
4. **No AI tell.** No filler, no "in today's fast-paced world," no exhaustive bullet
   dumps, no hedging. Reads as written by a senior human with a point of view.
5. **Skimmable then deep.** Structure serves the 30-second skim; content rewards the
   30-minute read. Both readers must be served by the same document.
6. **Curated, not comprehensive.** Fewer, sharper artifacts beat exhaustive coverage.
   Cutting is part of the craft this repo is meant to demonstrate.
7. **Honest provenance.** The repo does not hide that AI assisted the drafting; it makes
   clear the *judgment and opinions are the author's*. This is itself a demonstration of
   AI-enabled engineering leadership done with integrity.
8. **Show, don't just tell — but only where it earns its place.** A wall of prose is
   boring and gets skimmed and closed. Diagrams, tables, and numbers make reasoning
   land faster and prove the author thinks visually, as a senior technologist does. But
   a decorative diagram added just to break up text reads as *more* AI-generated, not
   less — every visual must carry information the prose can't convey as efficiently.
   Specifically:
   - **Mermaid diagrams** for anything structural or sequential — C4-style context
     views, event flows, failure-mode/failover sequences, decision trees. Prefer a
     diagram over three paragraphs describing a topology. Mermaid (not image files) so
     it renders natively on GitHub and stays version-controllable and editable.
   - **Comparison tables** for any "option A vs. option B vs. option C" reasoning — the
     core shape of most ADRs. A tradeoff table (cost / complexity / failure modes /
     when-to-use, per option) is almost always clearer than prose for the decision
     itself; keep the prose for the *why* behind the row values.
   - **Numbers and quantified tradeoffs** wherever a claim can be made concrete —
     latency, cost multipliers, downtime-per-nine, throughput, incident counts. "Each
     nine is ~10x the cost" and "five nines ≈ 5 min/year" land harder than "higher
     availability is expensive." Use the author's real figures where supplied; never
     fabricate a statistic to fill the gap (see the never-fabricate rule above).
   - **Restraint is the skill.** If a diagram or table doesn't make the point faster
     than a sentence would, cut it. Two sharp visuals in an ADR beat six filler ones —
     the same curation discipline as §4.6.

---

## 5. Relationship to the Broader Portfolio

- **Payments Resiliency Simulator** (flagship, existing) — the working system. This
  Playbook links to it as the lived example of several ADRs (resiliency patterns,
  multi-region, event-driven).
- **Architecture-review agent** (future) — will consume this Playbook's principles and
  ADRs as its rule set, turning written standards into enforced ones. This SPEC should
  leave that door open: principles and ADRs should be structured predictably enough to
  be machine-readable later (consistent front-matter / headings).
- **RFP + response** (future) — will reference this Playbook's decision frameworks as the
  reasoning behind its technology choices.

The portfolio is one connected argument, not six separate repos.

---

## 6. Open Questions to Resolve in PLAN.md

- Final ADR shortlist (which 6–10, in what order).
- Whether platform/ways-of-working ships in v1 or a later iteration.
- Repo structure & naming, front-matter convention for future machine-readability.
- README design: how to win the 30-second skim.
- What "done" means for v1 vs. what is explicitly deferred.

---

*Status: DRAFT — for author review. Author's real decisions and specifics to be supplied
before any content is written. PLAN.md to follow once this SPEC is agreed.*
