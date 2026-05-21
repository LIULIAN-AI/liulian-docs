# Agile documentation essentials — what LIULIAN actually needs

- **Date:** 2026-05-21
- **Audience:** the LIULIAN team (Linlin Jia + AI agents, occasional
  reviewer / advisor)
- **Status:** living guide

This document answers two questions:

1. **Under agile, which docs are unavoidable?** What's the minimum
   viable doc set so the team — including a future-self picking up the
   project in three months — can ship, debug, hand over, and audit
   without re-deriving everything from code?
2. **For design specifically, which design documents are needed?**
   What shape do they take, when are they written, when updated.

The orientation here is *agile for a 1-developer-with-AI team building
a 9-repo federation*, not generic agile / Scrum / SAFe boilerplate.
What's worth writing is what stops me (or my future self, or a
collaborator) from making a mistake that costs more than the document.

## 1 · The minimum viable doc set

Six categories. Anything outside this list is optional / on-demand.

### 1.1 Always-on (one per repo, updated as code changes)

| Doc | Owner | Why it exists |
|---|---|---|
| **README.md** (+ `.zh.md`) | each repo | First door — what is this repo, how to install, how to run, how to test, link to deeper docs |
| **CLAUDE.md** (project-level) | each repo | What conventions Claude / future AI sessions must follow; supersedes generic defaults for this repo |
| **`docs/architecture.md`** | each repo | One diagram + one paragraph: what subsystems, what flows, where to look in the code |
| **`docs/api/` or `openapi.json`** | each api-producing repo | Wire-format contract; consumers code against this, not against guessed shapes |
| **CHANGELOG.md** | each repo | Reverse-chronological list of user-visible changes — what shipped, when, why |
| **LICENSE** | each repo | non-negotiable |

These are **never absent**. If a contributor (or my future self) opens
a repo and any of these is missing, the repo is broken on the docs
axis.

### 1.2 Per-feature (one per non-trivial piece of work)

These travel together as a *trio*:

| Doc | When created | When updated |
|---|---|---|
| **User story** | before any design | when scope / acceptance changes |
| **Design doc / spec** | after the story is locked, before code | when the design changes mid-implementation |
| **Test plan** | alongside the spec (or in the same file) | when the design changes |

The order matters: **user story → spec → tests → code**. A feature
that landed without a user story is a feature whose value is
unverifiable.

### 1.3 Cross-cutting governance

| Doc | Lives in |
|---|---|
| **ADRs (Architecture Decision Records)** | the seed repo's `docs/strategy/adr/`; one file per accepted decision |
| **Reference designs / memory bank** | `liulian-design-system/docs/REFERENCE_DESIGNS.md` — what we steal vs. avoid from the outside world |
| **External references index** | `liulian-design-system/docs/REFERENCES.md` — all external sites / papers / libs cited anywhere |
| **Glossary** | seed repo `docs/glossary.md` (yet-to-write); avoid term drift across repos |

ADRs are the most under-used doc type. The rule: **if a choice would
take a new contributor > 30 min to derive from the code, write an
ADR**. Title + status + context + decision + consequences. Five short
paragraphs are enough.

### 1.4 Periodic (sprint / release rhythm)

| Doc | Cadence |
|---|---|
| **Sprint goal** | start of each sprint (≤ 1 page; the one outcome you'd be proud to ship) |
| **Sprint retro** | end of each sprint (≤ 1 page; what worked, what didn't, what to change) |
| **Release notes** | each release tag (auto-generated from CHANGELOG + a handwritten "why this matters" paragraph) |
| **Quarterly audit** | quarterly (the [audit-report](../strategy/AUDIT_REPORT_2026-05-12.md) shape — checks doc honesty, claim integrity) |

Skip the sprint retro on solo sprints if nothing changed; do not skip
when something went badly.

### 1.5 Operational (post-ship)

| Doc | Purpose |
|---|---|
| **Runbook** (`docs/runbook-<topic>.md`) | step-by-step recovery for a specific failure mode; written from an incident, not pre-emptively |
| **Onboarding guide** | what to do on day 1 — install, run tests, find a "first easy issue" |
| **Deployment guide** | how to deploy each repo (the `liulian-ops` repo owns this) |

Onboarding guide doubles as the smoke-test for the README + LOCAL_STACK
docs — if onboarding fails, those docs are wrong.

### 1.6 Knowledge sediment

These are *not* docs to write up-front; they accumulate over time as
artefacts of work:

- redesign rounds (HTML mocks + notes per round — see
  `liulian-web/docs/redesign-*/`)
- audit reports
- research notes (`docs/research/entity-id-deep/*`)
- advisor briefings (`docs/strategy/advisor-report-*`)

They are time-stamped, append-only, and never re-edited after the fact.
Their value is the *trajectory* they show.

## 2 · Design docs — the design-specific subset

A design doc is a contract: it tells anyone who reads it *what the
feature does, why it has the shape it does, and how to know when it's
finished*. The minimum kit:

### 2.1 The required four sections

Every design doc — regardless of size — must have:

1. **Goal** (one paragraph): what user-visible outcome this design
   achieves. If you cannot state this in one paragraph, do not write
   the spec yet — go back to the user story.
2. **Information architecture / data flow** (one diagram + one
   paragraph): boxes and arrows; what reads what, what writes what.
3. **Acceptance criteria** (a list of specific testable items): how
   the team / future-self knows the design is satisfied.
4. **Out-of-scope** (a list): what this design will *not* do, to
   prevent scope creep mid-implementation.

If you omit any of these, the doc is incomplete — even if it has
clever architecture diagrams.

### 2.2 Optional sections (include only if the design demands them)

- **Tokens / typography / palette** — for UI specs
- **API contract** — for backend specs (request/response, error
  shapes, rate limits)
- **Data model** — when the design changes schemas
- **Sequence diagrams** — when async or multi-actor flows are
  involved
- **Migration plan** — when the design changes an existing surface
- **Performance budget** — when latency / size / throughput is a
  first-class concern
- **A11y requirements** — for any user-facing UI
- **Security model** — when authn / authz / secrets are touched
- **Implementation file map** — when ≥ 5 files will change; tells
  the reviewer where to look

### 2.3 Design-doc layers (from broad to narrow)

| Layer | Lives in | Example |
|---|---|---|
| L1 · platform-level architecture | seed repo `docs/strategy/` | `PLATFORM_BLUEPRINT.md` — the 9-repo federation |
| L2 · subsystem architecture | each repo `docs/architecture.md` | `liulian-api/docs/architecture.md` |
| L3 · feature design doc | each repo `docs/specs/` or `docs/superpowers/specs/` | `liulian-web/docs/specs/2026-05-21-forecast-dashboard-design.md` |
| L4 · component / module README | next to the code | a component's docstring + a 1-paragraph README |

L1 changes < monthly. L2 changes < quarterly. L3 spawns per-feature.
L4 lives with the code and changes whenever code changes.

### 2.4 Cadence — when to write vs. update

- **Before code:** L3 spec for any feature beyond a 1-file change.
- **After code:** update L4 (component README + docstrings).
- **Sometimes:** L2 (when the subsystem grew a new layer or removed
  one).
- **Rarely:** L1 (when the platform structure itself shifts; usually
  paired with an ADR).

A spec that exists *only* after the code is built is documentation
debt — useful for re-onboarding, useless for design review. The whole
point of L3 is the review that happens *before* code.

## 3 · LIULIAN-specific reality (1 dev + AI, 9 repos)

The standard agile playbook assumes a team of humans on a single
codebase. LIULIAN has neither. Adjustments:

- **Pair AI agents instead of code-review humans.** The "design doc
  review" gate is the LLM (Claude / GPT / DeepSeek) reading the spec
  before generating code. Better spec = better generated code.
- **9 repos amplify doc cost.** Every repo paying the full doc
  overhead is unaffordable. Solution: the *seed repo* (this one,
  `liulian-python`) holds the cross-cutting docs (ADRs,
  PLATFORM_BLUEPRINT, sprint history); each sibling repo holds only
  its own L2 + per-feature L3 docs.
- **One person, no Jira.** Tasks live in this Claude session's
  TaskList and in commit messages, not in an external tracker. The
  *spec doc itself* is the durable task record.
- **Skill / verification gates substitute for QA process.** Custom
  skills (`code-verifier`, `research-critic`,
  `verification-before-completion`) play the role a QA team would. If
  a spec is missing acceptance criteria, the verifier flags it.

## 4 · Anti-patterns — what NOT to write

- **Tutorial-shaped README.** A README is a directory + install
  guide, not a tutorial. Tutorials live in `docs/tutorials/` if at
  all.
- **Design docs > 2 000 lines.** If the spec doesn't fit in 2 000
  lines, the scope is too big — decompose into sub-specs.
- **Sprint goals that say "ship the feature".** Sprint goal is the
  *outcome*, not the *output*. "By Friday, a reviewer can run the
  Bern demo end-to-end against real data" is a goal; "ship forecast
  panel" is not.
- **Quarterly roadmap docs in the seed repo.** Roadmaps go stale
  fast. Keep them in a `roadmap-YYYY-MM.md` (date-stamped) and let
  the older ones rot.
- **Generated docs that aren't read.** mkdocs / Sphinx auto-generated
  pages are useful for the API surface but worthless as a
  *narrative*. Don't pretend the auto-generated docs replace the L2
  architecture page.

## 5 · Templates

### 5.1 User story template

```
# Story · <persona> wants <goal> · <date>

**Persona:** <one-paragraph profile — what role, what tools, what
context, what they care about>

**Goal:** <one sentence — the outcome they want>

**Motivation:** <one paragraph — why now, what unblocking value>

**Scenario (Given / When / Then):**
- Given <state of the world>
- When <they take an action>
- Then <observable outcome>
- And <additional observable outcomes>

**Acceptance criteria** (testable):
- [ ] <criterion 1>
- [ ] <criterion 2>
- ...

**Non-goals:**
- <what this story explicitly excludes>

**Dependencies:**
- <other stories / specs / repos this depends on>

**Open questions:**
- <unresolved decision>

**Linked specs / ADRs:** <hyperlinks>
```

### 5.2 Design doc template

```
# <Feature name> — design

- Date / status / owner
- Predecessor spec (if any)

## 1 · Goal
## 2 · Information architecture / data flow
## 3 · <Optional: tokens / typography / palette>
## 4 · <Optional: API contract>
## 5 · <Optional: data model>
## 6 · <Optional: keyboard / a11y>
## 7 · Acceptance criteria
## 8 · Out-of-scope
## 9 · Implementation file map (when ≥ 5 files change)
## 10 · Approval
```

### 5.3 ADR template

```
# ADR <NNNN> — <title>

- Status: proposed | accepted | superseded
- Decision date

## Context
## Decision
## Consequences
## Alternatives considered
```

### 5.4 Runbook template

```
# Runbook — <failure mode>

## Symptoms
## Diagnostic checklist
## Recovery steps
## Prevention
## When to escalate
```

## 6 · Cross-reference

- User stories for the current redesign cycle:
  [`user-stories/`](../user-stories/index.md)
- ADRs: [`strategy/adr/`](../strategy/adr/index.md)
- Federation docs map: [`federation/docs-map.md`](../federation/docs-map.md)
- Multi-round redesign protocol: `~/.claude/CLAUDE.md` §"Multi-round
  redesign protocol"
