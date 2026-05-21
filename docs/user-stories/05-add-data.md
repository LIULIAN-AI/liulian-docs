# Story 5 · *The Graft* — add new rows, columns, or external joins

**Codename:** *The Graft* (嫁接) — extending a dataset by joining
new material onto the existing trunk.

**Persona:** *Researcher-Linlin*. She's been working with
`swiss-river-1990`'s water-temperature for a week. She realises she
needs precipitation data (from MeteoSwiss) to explain the
cold-front anomalies. She also wants to add a *derived column* —
`wt_anomaly_z` — that's wt_C minus its 30-day rolling mean. And
when the last week of December's data drops, she wants to *append*
those new rows without re-importing the whole CSV.

**Goal:** From the same canvas, *graft* new material onto the
existing dataset — new rows (append), new columns (derived or
joined from an external source), or both — and have the existing
dashboard reflect the additions immediately.

**Motivation:** Real research datasets *grow*. A platform that
treats import as a one-shot loses every user the moment they need
to extend. Power BI handles this through "datasets are pipelines";
Excel handles it by re-typing; neither is good enough. LIULIAN must
support three kinds of grafts cleanly: **rows** (more time), **derived
columns** (transform of existing columns), and **external columns**
(join with another dataset or web source).

---

## Mode A · No-bot (classical)

A small `[+ extend dataset]` button on the dataset card opens a
3-tab editor: **Append rows** · **Derived column** · **External
join**.

### Scenario · append rows

- **Given** I am viewing `/forecast?dataset=swiss-temp-2025`
- **When** I open the dataset card menu and click `[+ append rows]`
- **Then** a drop-zone widget appears in a modal asking for a
  delta CSV with the same schema
- **When** I drop `swiss-temp-2025-w1-2026.csv` (a 7-row delta)
- **Then** the modal shows a side-by-side compatibility check:
  schema match ✓ · station match ✓ · date range continuity ✓
- **When** I click [append]
- **Then** the 7 new rows are appended; dataset version increments
- **And** `#fan` / `#table` / `#kpi` re-render with the new rows
  in their tails

### Scenario · derived column

- **Given** the dataset has `wt_C`
- **When** I click `[+ derived column]`
- **Then** a form opens with: column name input · expression input ·
  preview output
- **When** I type `wt_anomaly_z = (wt_C - rolling_mean(wt_C, 30)) /
  rolling_std(wt_C, 30)`
- **Then** the form previews the first 10 output values
- **When** I click [add column]
- **Then** the column is added; `#picker` now includes
  `wt_anomaly_z` as a y-axis option for `#fan`

### Scenario · external join

- **When** I click `[+ external join]`
- **Then** the form asks: source URL · join key on local · join key
  on remote · columns to import
- **When** I fill in MeteoSwiss URL · join key `date` ↔ `date` ·
  columns `precip_mm`
- **Then** the form runs a sample join and previews 10 rows
- **When** I click [join]
- **Then** the dataset gains a `precip_mm` column; `#picker`
  expanded; the auto-BI auto-includes a precip overlay if the column
  is added

### Acceptance criteria

- [ ] `[+ extend dataset]` opens a tabbed modal with 3 tabs
- [ ] Append-rows: enforces schema match; shows a compatibility
      report (schema · types · date continuity · duplicate detection)
- [ ] Derived-column: supports a small expression language
      (pandas-like: arithmetic, `rolling_mean`, `rolling_std`,
      `diff`, `lag(N)`, `if`, `clip`)
- [ ] Derived-column preview computes 10 sample rows before commit
- [ ] External-join: only accepts public HTTPS URLs (security);
      enforces join-key compatibility
- [ ] Sample-join preview ≤ 10 rows; rejects > 90 % null joins with
      a "this looks like a bad join" warning
- [ ] After commit, all panels re-fetch and re-render
- [ ] Each graft becomes a versioned transform on the dataset (see
      ADR 0005 — three-entity unified tracker)
- [ ] Rollback: a dataset's version history page lets the user
      revert to a prior version

---

## Mode B · All-bot (radical)

The user describes the extension in natural language; the agent
parses, executes, and reports.

### Scenario · external join

- **Given** I am on `/forecast?dataset=swiss-temp-2025` in
  mode "AI-first"
- **When** I type into the agent: "add precipitation from MeteoSwiss
  for these stations"
- **Then** the agent replies:
    > "I'll join the MeteoSwiss daily precip API on `date` per
    > station. Found `precip_mm` and `precip_intensity`. Take both
    > or just `precip_mm`?"
- **When** I reply "just precip_mm"
- **Then** the agent fires the join, previews 10 rows in chat,
  and asks "looks right? [join]"
- **When** I click [join]
- **Then** the column commits; agent confirms; panels re-render

### Scenario · append rows from sensor feed

- **When** I type "every Monday at 06:00, append the last week's
  rows from MeteoSwiss"
- **Then** the agent creates a scheduled job in `liulian-ingest`
  and confirms with the next-fire timestamp

### Acceptance criteria

- [ ] Agent supports intents `append_rows`, `add_derived_column`,
      `join_external`, `schedule_append`
- [ ] Each intent maps to a tool call that previews before commit
- [ ] Schedule-create returns a job id; managed via `liulian-ingest`
- [ ] Errors surfaced conversationally (e.g. "MeteoSwiss didn't
      respond; tried 3 times. Want to retry now or schedule?")

---

## Mode C · Fused (innovative) · **headline mode**

The graft editor and the agent collaborate on a **shared canvas
tool** for aligning two time series — the join key panel. The user
sees both the local time series and the remote candidate; the agent
auto-aligns the join keys but the user can drag to override; the
agent annotates suspect alignments in real time.

### Innovation note

Uses **F1** (anchored agent annotations on the join key panel),
**F3** (bidirectional sync — drag the alignment → the agent's
sample join updates), **F4** (visible reasoning for each
auto-alignment hint), **F6** (the agent has a magnet cursor that
can snap two time-series points together).

The headline interaction is the **graft alignment canvas**: a
two-track time-series plot. Top track = local `wt_C`; bottom track =
remote `precip_mm`. The agent has *aligned* the two tracks on `date`
by default. The user can drag points on the bottom track left/right
to shift the alignment; the agent re-checks the join quality in
real time and *annotates* mis-aligned regions with a dotted yellow
outline. When the alignment is good across the whole range, the join
becomes commit-able.

The second headline interaction is **proactive graft suggestions**:
when the dataset is `swiss-river-*` and the user opens `[+ extend
dataset]`, the agent pre-emptively says "Aare basin work usually
joins MeteoSwiss precip and ECMWF era5-land soil-moisture. Want me
to set up either?" The agent draws on the domain catalog to suggest
relevant external sources.

### Scenario · the graft alignment canvas

- **Given** I am on `/forecast?dataset=swiss-temp-2025` in fused
  mode
- **When** I click `[+ extend dataset]`
- **Then** the modal opens with 3 tabs, and the agent simultaneously
  expands a sidebar:
    > "Aare basin work usually joins MeteoSwiss `precip_mm` and
    > ECMWF `soil_moisture`. Want me to set up either? [precip]
    > [soil moisture] [other]"
- **When** I click [precip]
- **Then** the **External join** tab opens with the form pre-filled
  (source URL · key `date` ↔ `date` · column `precip_mm`)
- **And** below the form a **graft alignment canvas** renders: two
  stacked sparkline tracks across the same date axis
- **And** the agent's auto-alignment uses `date` directly; the
  canvas shows the join with a 1-day shift indicator if the agent
  detected drift
- **When** I drag a point on the precip track ±2 days
- **Then** the alignment shifts and the agent re-checks the join
  quality in real time — **F3**
- **And** mis-aligned regions (where the correlation breaks) are
  outlined in dotted yellow — **F6**
- **And** the agent annotates: "Shifted +2 d worsens correlation
  from 0.71 to 0.43; reverting suggested" (anchored to the slider)
   — **F1, F4**
- **When** I release at +1 d
- **Then** the agent confirms: "Correlation 0.73 at +1 d (vs 0.71 at
  0 d). 0 d is the default; would you commit at +1 d?"
- **When** I click [commit at +1 d]
- **Then** the join applies with the +1 d offset baked in
- **And** the new `precip_mm` column appears in the dataset; `#picker`
  updated; an overlay of `precip_mm` on `#fan` becomes available

### Scenario · derived column with chat assist

- **When** I click `[+ derived column]` instead
- **Then** the form opens; the agent simultaneously suggests:
    > "Common derived columns for water-temperature time series:
    > `wt_anomaly_z` · `wt_rolling_mean_30d` · `wt_diff_24h` ·
    > `wt_seasonal_residual`. Tap to scaffold one."
- **When** I tap `wt_anomaly_z`
- **Then** the form pre-fills the name and expression
  (`(wt_C - rolling_mean(wt_C, 30)) / rolling_std(wt_C, 30)`)
- **And** the agent re-states the formula in plain English
  "(value minus 30-day mean) divided by 30-day std" with a sentence
  on when each scaffolding is useful
- **When** I tweak the window from 30 to 60 days
- **Then** the preview updates; the agent re-states "(value minus
  60-day mean) divided by 60-day std" — **F3**

### Scenario · proactive append from sensor feed

- **Given** the dataset is registered in the tracker as
  `swiss-temp-2025` with a known upstream MeteoSwiss source
- **When** the user opens the dataset card menu
- **Then** the agent emits a quiet annotation: "MeteoSwiss has
  released data through 2026-W18. Your dataset ends at W17. Want
  to append the latest week?"
- **When** the user clicks [append]
- **Then** the append flow runs without further prompts

### Acceptance criteria

- [ ] Graft alignment canvas: two stacked sparklines, shared
      x-axis, draggable alignment
- [ ] Real-time join-quality re-check on drag (debounced 100 ms)
- [ ] Mis-aligned region detection: correlation drops > 0.2 in a
      window flagged with dotted yellow (F6)
- [ ] Anchored agent annotation explaining each alignment decision
      (F1, F4)
- [ ] Derived-column form auto-fills from agent-scaffolded
      expressions
- [ ] Plain-English re-statement of the expression updates with
      every change to the expression (F3)
- [ ] Proactive append suggestion when the agent detects an upstream
      delta on a known data source
- [ ] All grafts versioned and reversible (story 4 diff strip)
- [ ] Mobile portrait < 480 px: alignment canvas collapses to a
      vertical pair of sparklines with a single drag handle

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| Power BI Power Query / Tableau Prep | Multi-step pipeline editor in a separate surface. LIULIAN keeps the graft *on the canvas* with an inline alignment widget. |
| pandas `merge_asof` + manual inspection | Visual alignment with real-time quality feedback (F3 + F6) |
| Pre-built "data marketplace" connectors | Agent proposes connectors *from the domain context* of the active dataset (F4 — visible reasoning + domain catalog) |
| Schedule via a cron config file | Schedule via natural language ("every Monday at 06:00 from MeteoSwiss") — agent registers the job in `liulian-ingest` |

### Non-goals (mode C, v1)

- Multi-source joins in one pass (one external source per join for
  v1)
- Non-time-aligned joins (e.g. join by a fuzzy key like geographic
  proximity)
- Domain catalog as a learned model (catalog is hand-curated for
  v1: swiss-river → meteoswiss + era5; electricity → ENTSO-E; etc.)
- Real-time streaming sources (only batch / poll for v1)
- Reverse joins (modify the upstream dataset)

---

## Dependencies

- `liulian-api` (NEW):
  - `POST /datasets/{id}/append-rows` — append delta from upload
  - `POST /datasets/{id}/derived-columns` — register a derived col
  - `POST /datasets/{id}/joins` — register an external join with
    `{source_url, key_local, key_remote, columns, shift_days?}`
  - `GET /datasets/{id}/upstream-delta` — check upstream source for
    new data
- `liulian-agent` tools (NEW):
  - `append_rows(file_id|url)`
  - `add_derived_column(name, expression)`
  - `join_external(source, key, cols, shift?)`
  - `schedule_append(source, cadence)`
  - Domain catalog plugin: returns suggested external sources for a
    given dataset kind
- `liulian-ingest` (NEW):
  - Scheduled-job table; cron registration; per-source adapters
    (MeteoSwiss, ERA5, ENTSO-E to start)
- `liulian-web` (NEW):
  - `+ extend dataset` modal with 3 tabs
  - Graft alignment canvas (two-track sparkline + drag-to-align)
  - Quiet agent annotation primitive (no popup; inline text)
- Expression language (small, safe): subset of pandas; lives in
  `liulian-python` as a pure-Python evaluator + a JS mirror for
  the form preview

## Open questions

- Domain catalog: hand-curated or LLM-derived? (Author leans:
  hand-curated for v1, LLM-augmented for v2.)
- Schedule conflict: if a scheduled append runs while the user has
  uncommitted edits, who wins? (Author leans: schedule queues; the
  user's edits commit first.)
- Derived-column expression sandbox: how strict? (Author leans:
  whitelist of functions only; no `eval`; no `import`.)
- Storage for grafts: copy the joined data into the local dataset
  vs. reference upstream? (Author leans: copy for joins, reference
  + cache for scheduled appends.)

## Linked specs / ADRs

- Story 2 (import) — append rows is a delta-import
- Story 3 (auto-BI) — derived columns appear in `#picker`
- Story 4 (edit) — grafts are diffable / undoable
- ADR 0005 (three-entity unified tracker) — grafts are tracker
  entries
- ADR 0011 (dev-env separate from ops) — schedules run in `ingest`,
  not in `api`
