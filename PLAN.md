# PLAN — Engineering Playbook Build Plan

> The ordered build steps for `engineering-playbook`. `SPEC.md` says *what* and *why*;
> this says *in what order* and *done when*. `SPEC.md` wins any conflict.
>
> **Status:** DRAFT — awaiting author review. No content is written until this plan is agreed.
> Update the status column of each step as it lands; don't jump phases (per `CLAUDE.md`).

---

## Open questions resolved (SPEC §6)

| # | Question | Resolution | Source |
|---|----------|------------|--------|
| 1 | Final ADR shortlist & order | 8 ADRs: 4 Tier 1 then 4 Tier 2, ordered strongest/most experience-backed first (see Phases 2–3). ADR-008 (AI adoption) added after the initial shortlist. | Shortlist **carried from SPEC**; order is my **recommendation**. |
| 2 | Platform/ways-of-working in v1? | **Deferred to v2.** *Reason: v1's edge is scar-backed ADRs; a broad platform section risks the generic-best-practice filler SPEC §4 exists to prevent, and dilutes shipping focus.* | My **recommendation** — override if you have real golden-path/IDP scars ready to write. |
| 3 | Repo structure & front-matter | `/adrs`, `/principles`, `/templates` (+ deferred `/platform`); front-matter schema fixed in Phase 0. | **Carried from CLAUDE.md**, extended with machine-readable `id`/`related` fields (my recommendation). |
| 4 | README design | Positioning line → the gap it closes → curated ADR index table → reader-map → provenance → Simulator link. Skeleton early, finalized last. | My **recommendation** (Phase 1). |
| 5 | What "done" means for v1 | All 8 ADRs + 8–12 principles + 5 templates + README, every `[AUTHOR: …]` filled, statuses left at `draft` for author promotion. | My **recommendation** (see Definition of Done). |

**One line on ADR order (question 1):** SPEC calls data-strategy *the* v1 differentiator, so the
repo opens with its rarest signal — canonical data model — then leads into the most
number-dense story (availability). *Alternative if you'd rather lead with concrete figures:
swap ADR-001 and ADR-002.*

---

## Author inputs to gather up front

These are the load-bearing `[AUTHOR: …]` specifics. Nothing in Phase 2–4 can be marked done
until the relevant row is supplied. **I can draft the industry reasoning around each; I cannot
invent any of these** (`CLAUDE.md` §"never fabricate").

| ADR / artifact | What I need from you |
|---|---|
| ADR-001 Canonical data model | Which platform/system; the concrete lineage/reconciliation/governance/audit cost you paid (or avoided); reference-data & metadata-management specifics; any counts (consumers, reconciliation effort). |
| ADR-002 Availability target | The 99.999% system & context; the 40→<5 monthly-incident figures + timeframe; the SLO/error-budget mechanism as you ran it; the feature-velocity you traded; **your call: split error-budget into its own ADR or keep it inline** (SPEC leaves this open — do not pre-commit to two). |
| ADR-003 Multi-region | The real active-active vs active-passive decision & system; the failure mode that drove it; cost / RTO / RPO figures; region topology. |
| ADR-004 Op/analytical planes | The real converge-or-separate decision; the latency-sensitive use case (e.g. fraud/risk) if any; why you chose as you did; any latency/cost numbers. |
| ADR-005 Coupling across domains | The cross-domain decision; the domains involved; what the rejected option actually cost. Plus, if you have it: a real agent-topology call, since the ADR now carries the model-latency argument. |
| ADR-006 Multi-region Kafka | The payments platform & its region topology; the RPO/RTO the business actually set; what a failover rehearsal revealed; **the duplicate/reconciliation window you actually operated to** and what it cost. |
| ADR-007 DB-selection framework | The real selections across DB families and the criteria you actually applied. |
| ADR-008 AI-assisted engineering adoption | The org and its size; what you actually rolled out and to whom; **what you chose to measure and what you refused to measure**; the instability or quality effect you saw (or didn't); what adoption cost that the tooling bill didn't show. I can supply the DORA/METR evidence base — the org call, the measurement decision, and its cost are yours. |
| Principles | Confirm the 8–12 that reflect how you *actually* lead — I can distill candidates from the ADRs, you confirm/cut. |

---

## Phase 0 — Scaffolding *(blocks everything)* — ✅ DONE

Everything downstream depends on a fixed template and front-matter, so this lands first and
alone.

**0.1 — Directory structure**
```
engineering-playbook/
├── README.md          # Phase 1
├── SPEC.md  CLAUDE.md  PLAN.md   # exist
├── .gitignore
├── adrs/              # NNN-short-title.md, zero-padded, sequential
├── principles/        # NNN-short-title.md
├── templates/
└── platform/          # deferred to v2 — created empty or omitted until then
```
- *Done when:* folders exist, committed.  *Depends on:* nothing.  *Drafts independently.*

**0.2 — Front-matter convention** (fix once, applies to every ADR & principle; enables the
future architecture-review agent to parse the corpus)

ADR front-matter:
```yaml
---
id: ADR-001
title: Canonical data model at ingestion vs. transform-on-read
status: draft            # draft | accepted | superseded
tier: 1                  # 1 | 2 | 3
date: 2026-07-11
tags: [data-strategy, governance, lineage]
supersedes: null
superseded_by: null
related: [ADR-004, PRIN-003]
---
```
Principle front-matter: same minus `tier`/`supersedes` (`id: PRIN-00N`).
- *Done when:* schema written into the ADR template and this plan; agreed.  *Needs author sign-off* on the schema.

**0.3 — ADR template** (`templates/adr-template.md`)
- Encodes front-matter + the fixed sections **Context → Options Considered → Decision →
  Consequences → Status** (never improvised per-ADR — `CLAUDE.md`).
- Inline guidance comments: where a comparison table / Mermaid diagram is expected, the
  `[AUTHOR: …]` convention, and the "state when this would be the *wrong* call" requirement.
- Must read cleanly with the guidance stripped (templates are fillable, not example-bound).
- *Done when:* a contributor could author an ADR from it with no other reference.  *Depends on:* 0.2.  *Drafts independently.*

**0.4 — `.gitignore`**
- OS/editor cruft (`.DS_Store`, `Thumbs.db`), `.vscode/`, `.idea/`, `.env*`. Minimal.
- *Done when:* committed.  *Drafts independently.*

---

## Phase 1 — README *(its own deliverable, not an afterthought)* — ◑ SKELETON DONE (finalize after Phases 2–4)

The README carries the 30-second skim (SPEC §4.5) and must land reader 1 (leaders) and reader 3
(recruiters) on structure alone. **Skeleton built now; finalized last** once ADRs exist to link.

Must contain, in this order:
1. **Positioning line** — one sentence that signals altitude, e.g. *"How I make architecture
   decisions — the tradeoffs made explicit and owned."* Not a description of contents.
2. **The gap it closes** — 2–3 lines: most repos prove *what* you used; this proves *why*, with
   the cost owned (SPEC §1).
3. **Curated ADR index** — a table (tier · decision-in-one-line · status · link). This is the
   skimmable centerpiece; a leader should read the table and see the judgment.
4. **Reader-map** — "Recruiter → read this page. Staff/principal → read ADR-001, 002, 003." Routes
   each audience to proof fast (SPEC §2).
5. **Principles & templates** — one line each + links.
6. **Provenance note** — AI assisted the drafting; the judgment and opinions are the author's
   (SPEC §4.7). Stated plainly, as a strength.
7. **Cross-link to the Payments Resiliency Simulator** — the working system these ADRs describe.

- *Done when:* skeleton passes the skim test (sections 1–2, 4, 6–7); **finalize when** the index
  table (3) links every shipped ADR/principle.
- *Depends on:* Phase 0 for structure; **finalization depends on Phases 2–4** (index needs targets).  *Drafts independently* except the Simulator URL (confirm the repo link).

---

## Phase 2 — Tier 1 ADRs *(the differentiators — ship first, strongest first)* — ◑ DRAFTED (awaiting author specifics)

Each: fixed template, front-matter complete, `status: draft`, at least one *earned* visual, honest
"wrong call" section, quantified tradeoffs where real numbers exist. *Depends on:* Phase 0 + the
matching author-input row. *I draft the industry reasoning; author supplies the lived specifics.*

**2.1 — ADR-001 · Canonical data model at ingestion vs. transform-on-read**
- Why canonicalize once at the boundary vs. N consumers reshaping; lineage / reconciliation /
  governance / audit cost of not doing so.
- *Visuals:* Mermaid — boundary-canonicalization vs. consumer fan-out; table — cost per approach
  across lineage/reconciliation/governance/audit.
- *Author inputs:* ADR-001 row.  *Done when:* draft complete, no open `[AUTHOR: …]`, wrong-call section present.

**2.2 — ADR-002 · Choosing the right availability target (why 99.999% is sometimes wrong)**
- Each nine ≈ 10× cost; five nines ≈ 5 min/yr — justified for a payment network, wrong for many
  systems. The 40→<5 reduction as lived evidence via SLO/error-budget discipline.
- *Visuals:* table of nines (downtime/yr · cost multiplier · when-appropriate); before/after on
  40→<5; optional Mermaid failover sequence.  *Cross-link:* Payments Resiliency Simulator (working demo).
- *Author inputs:* ADR-002 row **incl. the split-or-inline error-budget decision**.

**2.3 — ADR-003 · Multi-region active-active vs. active-passive**
- Cost, complexity, and the specific failure modes each actually protects against.
- *Visuals:* Mermaid topology per option; failure-mode table; cost / complexity / RTO / RPO table.
  *Cross-link:* Simulator (multi-region patterns).
- *Author inputs:* ADR-003 row.

**2.4 — ADR-004 · Converging operational & analytical data planes vs. keeping them separate**
- When unified (HTAP / streaming lakehouse) earns its complexity vs. when OLTP/OLAP-with-a-pipeline
  is still right. Honest live-debate framing, not a convergence hot take.
- *Visuals:* Mermaid — split-with-pipeline vs. unified store; table — latency / complexity / cost /
  when-to-use.
- *Author inputs:* ADR-004 row.

---

## Phase 3 — Tier 2 ADRs *(technology judgment — solid, supporting)* — ◑ DRAFTED (awaiting author specifics)

Same bar as Phase 2. General → specific → framework. *Depends on:* Phase 0 + author rows.

**3.1 — ADR-005 · Coupling across domains — when a caller may wait, and when it must not**
- Reframed from the event-driven-vs-REST debate onto coupling and failure semantics, because agent
  chains have made the same question urgent again at model latency. File renamed to match.
- *Visuals:* Mermaid sequence per model; tradeoff table; the same-shape/new-constants comparison
  carrying the agent section.  *Cross-link:* Simulator (event-driven); ADR-008 as its org-side counterpart.
- *Author inputs:* ADR-005 row.

**3.2 — ADR-006 · Multi-region Kafka — and the duplicate window you can't design away**
- Stretched cluster vs. cross-region mirroring vs. producer broadcast vs. active-passive — compared
  on write ownership and duplicate window, *not* on the RPO/RTO table every vendor already publishes.
  Follows through to publishers, consumers, replay, reconciliation, and DLQ.
- Must stay clear of ADR-003: that one decides the data-plane topology, this one decides what happens
  to the event log and the consumer's position in it.
- *Visuals:* the duplicate-window anatomy (the diagram nobody publishes); the four topologies on a
  duplicate-window spectrum; decision tree keyed on the window the ledger can absorb.  *Cross-link:* Simulator.
- *Author inputs:* ADR-006 row.

**3.3 — ADR-007 · Database-selection framework (relational / document / wide-column / key-value)**
- A decision *framework*, not a ranking.
- *Visuals:* Mermaid decision tree; comparison table across DB families.
- *Author inputs:* ADR-007 row.

**3.4 — ADR-008 · AI-assisted engineering adoption — a measurement decision, not a tooling one**
- Added to scope after the initial 7-ADR shortlist (SPEC §3.2 updated to match). Earns its place on
  the author's org-adoption experience, **not** on the topic being current — if the author input
  doesn't materialise, cut it rather than ship an industry summary.
- The position: AI raises throughput and instability together and amplifies whatever discipline the
  org already had, so the decision is which outcome you commit to measuring — and developer
  perception is disqualified as the instrument.
- Must state honestly where the evidence is contested and where it has already dated (METR
  themselves now label their result historical).
- *Visuals:* an SVG on the perception-vs-measurement gap; a table of what to measure vs. the
  adoption metrics most orgs reach for.  *Cross-links:* ADR-002 (error budget as the mechanism that
  catches the instability), PRIN-010 (measure outcomes, not activity), PRIN-007 (novelty earns its place).
- *Author inputs:* ADR-008 row.

---

## Phase 4 — Principles & templates — ◑ DRAFTED (templates final; principles await author confirm)

**4.1 — Principles (8–12)** — one file each: one-line statement + short rationale + a real
consequence of ignoring it. Specific enough to disagree with (SPEC §3.1). `related:` links to the
ADR each distills.
- *Depends on:* Phase 2 (principles distill Tier 1 positions — this is why principles follow, not
  precede, the ADRs).  *Author confirms* the final set reflects how they lead.

**4.2 — Templates (5)** — `rfc-template.md`, `microservice-readiness-checklist.md`,
`tech-debt-framework.md`, `engineering-metrics.md` (what to measure, what *not* to, why vanity
metrics mislead). Each genuinely fillable, no example left inside (CLAUDE.md). ADR template already
shipped in 0.3.
- *Depends on:* Phase 0 only — **no content dependency; can be pulled forward** to run parallel with
  Phases 2–3 if useful.  *Drafts independently* (metrics doc may want an author scar to anchor it).

---

## Phase 5 — Platform & ways-of-working *(DEFERRED to v2)*

Not in v1. *Reason: the golden-path / IDP / GitOps material is only worth shipping if it's as
scar-backed as the ADRs; drafted generically it becomes the exact best-practice filler SPEC §4
forbids, and it competes for focus with the eight ADRs that are the actual differentiator.*
Revisit once v1 ships — or pull one sharp "paved-road / golden-path" principle into Phase 4 if you
have a concrete story for it now.

---

## Cross-cutting: visuals & cross-links

- **Visuals (SPEC §4.8 — earn their place or cut):** Mermaid for every topology / sequence /
  decision tree; a comparison table for every A-vs-B-vs-C (the core shape of an ADR); real numbers
  wherever a claim can be made concrete. Never fabricate a figure to fill a gap.
- **Simulator cross-links** belong in **ADR-002 (availability), ADR-003 (multi-region), ADR-005 &
  ADR-006 (event-driven / multi-region Kafka)** — the resiliency, multi-region, and event-driven patterns the
  Simulator actually demonstrates (SPEC §5). Confirm the Simulator repo URL before finalizing the README.

---

## Definition of Done — v1

- All **8 ADRs** written to the fixed template, front-matter complete, `status: draft` (author
  promotes to `accepted` — Claude does not self-certify).
- **8–12 principles** and **5 templates** shipped; every template fillable with no example inside.
- **README** wins the skim, index links every artifact, provenance note and Simulator cross-link present.
- **Zero unfilled `[AUTHOR: …]`** placeholders remain.
- Every ADR carries at least one earned visual and an honest "wrong call" section.
- Platform explicitly deferred and noted as such.

---

## What v1 is explicitly NOT

- Not the platform/ways-of-working sections (v2).
- Not Tier 3 ADRs (REST-vs-gRPC, serverless, k8s-vs-managed) — cut unless genuinely scar-backed;
  thin coverage dilutes the sharp ADRs around it (SPEC §3.2).
- Not comprehensive — 8 ADRs, not 15. Curation is the demonstration (SPEC §4.6).
- Not code — no review agent, no automation yet; that's a later phase with its own CLAUDE.md.
- Not self-promoted to `accepted` — the author reviews and promotes.
```
