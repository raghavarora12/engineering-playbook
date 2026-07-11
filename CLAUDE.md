# CLAUDE.md — Operating Rules for This Repository

**Repository:** `engineering-playbook`

> This file is read automatically at the start of every Claude Code session in this
> repo. It exists because this repository will be built across many sessions, and the
> quality bar in `SPEC.md` §4 is easy to forget by session five if it isn't re-asserted
> every time. Treat this as a standing instruction, not background reading.

## Source of truth, in order

1. **`SPEC.md`** — intent, scope, and quality bar. If anything here conflicts with it,
   SPEC.md wins.
2. **`PLAN.md`** — the ordered build steps and current status. Check it before starting
   work to know what's next; update it when a step is done.
3. **This file** — the rules for *how* to write, not *what* to write.

If a request from the author conflicts with SPEC.md's intent (e.g. "just fill in
something plausible here to save time"), flag the conflict rather than silently
complying — the whole value of this repo depends on SPEC §4 holding.

## The one rule that matters most: never fabricate the author's experience

This repo's credibility rests entirely on its opinions being real, lived judgment from
the author's career (payments networks, core banking,
100+ engineer orgs, the 40→<5 monthly incident
reduction, etc.), not plausible-sounding AI prose dressed up as experience.

- Any `[AUTHOR: …]` placeholder is **load-bearing**. Do not invent a specific number,
  incident, or anecdote to fill one. If the author hasn't supplied the specific yet,
  leave the placeholder visibly open and flag it — do not paper over it with something
  generic that "could" be true.
- If a draft is heading toward a claim that sounds specific but isn't sourced from
  something the author actually told you, stop and ask rather than continue.
- General industry knowledge (how Kafka works, what HTAP means) is fine to draft from.
  Claims about *what the author decided, why, and what it cost* are not — those come
  from the author only.

## Voice and quality bar (enforcing SPEC.md §4)

Before treating any document as done, check it against these, in order:

1. **Does it take a position someone could disagree with?** If a reader couldn't argue
   back, it's too generic — sharpen or cut it.
2. **Is the tradeoff stated honestly?** Every ADR needs a real "here's what this costs
   you and when this would be the wrong call" section, not just a justification for the
   chosen option.
3. **Does it pass the AI-tell test?** Cut on sight: "in today's fast-paced world," "it's
   important to note that," exhaustive caveat-everything hedging, listicle padding,
   restating the question before answering it, or any sentence that could appear
   unchanged in literally any other company's engineering blog.
4. **Does it serve both the 30-second skim and the 30-minute read?** Lead with the
   position; let depth follow for the reader who wants it.
5. **Does it show, not just tell?** Prose-only documents get skimmed and closed. Reach
   for a Mermaid diagram (structural/sequential reasoning), a comparison table (any
   A-vs-B-vs-C tradeoff — the core of most ADRs), or concrete numbers wherever they land
   the point faster than prose. But every visual must earn its place — a decorative
   diagram added to break up text reads as *more* AI-generated, not less. If a visual
   doesn't beat a sentence, cut it. See SPEC §4.8.
6. **Is it curated, not exhaustive?** If a section is trying to cover everything, that's
   a sign it should be shorter and more opinionated instead.

## Repo conventions

- ADRs live in `/adrs/`, one file per decision, named `NNN-short-title.md` (zero-padded,
  sequential).
- Every ADR uses the template in `/templates/adr-template.md` — same structure every
  time: Context → Options Considered → Decision → Consequences → Status. Consistency
  here matters because a future tool (the architecture-review agent, planned) will parse
  these programmatically — don't improvise the structure per-ADR.
- Front-matter on every ADR and principle doc: `title`, `status` (draft/accepted/
  superseded), `tier` (1/2/3 per SPEC §3.2), `date`. This is what makes the corpus
  machine-readable later; don't skip it because it seems like overhead now.
- Principles live in `/principles/`, one short file each — one-line statement +
  rationale + a real consequence of ignoring it. Resist the urge to write more than
  that; length is not the goal.
- Templates (RFC, checklist, tech-debt framework) live in `/templates/`, and must be
  genuinely fillable — no template that only makes sense with the example still in it.

## Workflow

- Check `PLAN.md` for the current step before starting; don't jump ahead to later
  phases (e.g. the review agent) while Tier 1 ADRs are still unwritten.
- When a document is drafted, mark its status as `draft` in front-matter, not
  `accepted` — the author reviews and promotes it, Claude doesn't self-certify.
- If scope changes mid-build (a new ADR idea, a tier gets reprioritized), update
  `SPEC.md` and `PLAN.md` to reflect it before writing content — don't let the plan
  silently drift out of sync with what's actually being built.
- Commit messages describe the *decision content*, not the mechanics (`Add ADR-004:
  canonical data model at ingestion`, not `Update files`).

## What this repo is not (don't drift into these)

- Not a tutorial or how-to guide.
- Not a place to demonstrate breadth by covering every possible topic — see SPEC §4.6.
- Not code (yet) — this phase is documentation and reasoning. The architecture-review
  agent is a separate, later phase with its own CLAUDE.md when it starts.
