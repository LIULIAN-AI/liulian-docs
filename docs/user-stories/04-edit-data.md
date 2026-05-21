# Story 4 · *The Pencil* — edit a cell, watch the chart follow

**Codename:** *The Pencil* (改笔) — the smallest user action with the
loudest downstream effect.

**Persona:** *Researcher-Linlin*. She's looking at the dashboard and
notices a suspect value — a wt °C reading of 28.4 °C in winter for a
mountain station, almost certainly a sensor glitch. She wants to fix
or null it, see the chart and KPIs update, and (if she trusts the
edit) commit it as a transform.

**Goal:** Edit one cell (or many) in `#table`; every chart and KPI
that reads that cell re-renders within 200 ms. Track every edit in a
visible diff strip with one-click undo. Optionally upgrade the edit
into a *transform* (a rule that applies to similar future imports).

**Motivation:** Trust comes from being able to fix things. Read-only
dashboards generate distrust on first contact with bad data —
"there's a hole in the data; if I can't fix it here, why am I
here?" Sub-200 ms feedback after a fix turns the canvas from a
report into a *workspace*.

---

## Mode A · No-bot (classical)

The standard spreadsheet edit-commit-re-render flow.

### Scenario

- **Given** I am on `/forecast?station=2091` with `#table` and
  `#fan` visible
- **And** I notice the row for `2020-11-06` shows `28.40` °C, an
  obvious outlier in November
- **When** I double-click that cell
- **Then** the cell enters edit mode (UniBe-red border, value
  selected, autofocus)
- **When** I type `7.92` and press `↵`
- **Then** the cell commits with bg `#FAFFEA` (dirty marker)
- **And** `#fan` re-renders within 200 ms with the new observed
  series
- **And** `#kpi` re-computes MAE / RMSE / coverage for the affected
  window
- **And** a diff strip slides up from the bottom of `#table` reading
  "1 edit · [undo] · [view diff]"
- **When** I click `[view diff]`
- **Then** a modal lists every edit: `(2020-11-06) wt: 28.40 → 7.92`
- **When** I click `[undo]`
- **Then** the cell reverts; the diff strip closes
- **When** I make multiple edits (5 rows)
- **Then** the diff strip shows "5 edits"; the undo affects them
  as a batch (last-in-first-out)
- **When** I press `Ctrl+S` (or click [save] in the strip)
- **Then** the edits commit as a permanent transform on the
  dataset version; the run-header pill flips to `dashboard v2 ·
  saved`

### Acceptance criteria

- [ ] Double-click on a numeric cell in `#table` enters edit mode
- [ ] `↵` commits, `esc` cancels
- [ ] Dirty cell bg `#FAFFEA`
- [ ] Edit dispatches `liulian:table-edit` CustomEvent
- [ ] `<FanPanel/>` subscribes to the event and re-renders < 200 ms
- [ ] `<KpiStrip/>` recomputes its metrics on edit; updates the
      sparkline
- [ ] Diff strip docks at the bottom of `#table`; visible only when
      dirty count > 0
- [ ] Multi-cell edits supported; per-cell undo via `[view diff]`
- [ ] `Ctrl+S` saves; `Ctrl+Z` undoes last edit; `Ctrl+Shift+Z`
      redoes
- [ ] Edit on a row that no chart reads (e.g. `flag` column) marks
      dirty without re-rendering charts
- [ ] Edit history is part of the dashboard state for sharing
      (story 5)
- [ ] No agent involvement in this mode (mode preference = "classical"
      hides the agent rail entirely)

---

## Mode B · All-bot (radical)

The user does not directly click cells; they describe the edits in
conversation. The agent identifies the cell(s) and applies the edit.

### Scenario

- **Given** I am on `/forecast?station=2091`
- **And** I am in mode "AI-first"
- **When** I type into the agent: "set 2020-11-06 to 7.92"
- **Then** the agent parses to a cell-edit and replies:
    > "Updating `2020-11-06` `wt_C` from 28.40 → 7.92.
    > Reason for the change? [outlier removal] [sensor recalibration]
    > [other]"
- **When** I tap [outlier removal]
- **Then** the edit applies; the agent confirms:
    > "Done. `#fan` and `#kpi` updated. (1 dirty cell)"
- **When** I type "show me other suspect cells"
- **Then** the agent runs a quality scan and replies:
    > "I found 4 more cells with z > 3 in 2020-2024.
    > 2034 / 2020-08-14 → 28.10 (z = +3.4)
    > 2009 / 2021-06-22 → 27.90 (z = +3.1)
    > 2143 / 2022-09-03 → 27.40 (z = +3.0)
    > 2018 / 2023-01-15 → 28.60 (z = +4.0)
    > Want me to flag, null, or replace each?"
- **When** I type "null all of them"
- **Then** the agent applies 4 edits; diff: "4 edits"; confirms

### Acceptance criteria

- [ ] Agent intent `edit_cell` parses to a tool call `cell_edit(date,
      station, column, new_value, reason)`
- [ ] Agent intent `find_suspects` parses to a quality scan
- [ ] Agent edit follows the same diff-strip pipeline as mode A
- [ ] Agent edits are tagged in the diff with `via_agent: true`
- [ ] Edit confirmations include the panels that re-rendered
- [ ] No spreadsheet edit UI in mode B — cells are not directly
      clickable; if the user clicks one, the agent intercepts:
      "you're trying to edit `2020-11-06`. Tell me the new value or
      the rule."

### Risks

- The agent can mistake what cell the user means (date format
  ambiguity, station ambiguity). Mitigation: when ambiguity is
  detected, the agent asks before applying.
- Bulk edits via the agent can chain destructively. Mitigation: any
  edit of > 5 cells in one message requires a confirmation tap.

---

## Mode C · Fused (innovative) · **headline mode**

The user clicks cells like in mode A *and* the agent watches every
edit, infers the *intent*, and offers to extend the edit pattern
across the dataset. The user can accept, refuse, or tweak the
extension.

### Innovation note

Uses **F1** (anchored intent-inference bubble on the edited cell),
**F2** (predictive prefetch — agent already computed the candidate
expansion set before the user finishes typing), **F4** (visible
reasoning — "I found 23 more cells with z > 2.5; here they are"),
**F5** (reversible — extension applies to a *preview* layer first;
user confirms to commit), **F6** (agent can mark the candidate
cells in `#table` with a dotted outline, separately from the user's
own dirty marker).

The headline interaction is **intent-inference from a single edit**:
the user edits one cell, the agent immediately asks "is this pattern
something you want to apply elsewhere?" If yes, the agent marks
candidate cells with a dotted outline (its "pencil cursor", F6) and
the user can accept individually or all-at-once. This collapses what
used to be a 30-minute "find all the outliers and fix them" pandas
session into a 30-second loop.

The reverse interaction also works: the user describes the *intent*
to the agent, the agent renders the candidate cells as outlines, the
user clicks individual outlines to commit or refines the rule in chat.

### Scenario · the canonical edit → infer → extend loop

- **Given** I am on `/forecast?station=2091` in fused mode
- **And** I notice `2020-11-06` shows 28.40 °C
- **When** I double-click the cell and change it to `7.92`
- **Then** the edit commits per mode A
- **And** in parallel (within 300 ms) the agent identifies the edit
  as an outlier removal (heuristic: |Δ| > 3 σ of surrounding 7-day
  median; or LLM classification)
- **And** an anchored bubble pops out from the edited cell saying:
    > "Looks like an outlier removal. I found **23** other cells
    > with the same pattern (z > 2.5 against a 7-day window).
    > Want to flag, null, or replace each? [show them]"
- **And** the bubble has a `[ignore]` button to dismiss without action
- **When** I click [show them]
- **Then** the agent marks 23 cells in `#table` with a dotted
  UniBe-red 1 px outline — the agent's "pencil cursor" — and
  scrolls the table to the first one — **F6**
- **And** the agent's bubble morphs into a 23-cell action row at
  the top of `#table`:
    > "23 candidate cells. Action for all: [null] [interpolate]
    > [keep]. Or click each one to decide."
- **When** I click [interpolate]
- **Then** the agent applies the transform in a **preview layer**
  (cells highlighted with bg `#FBF8E8`, italic) — not committed yet
- **And** `#fan` and `#kpi` render with the previewed values
- **And** a confirm-bar at the top of the canvas reads:
    > "Preview: 23 cells interpolated. KPIs improved (MAE 0.62 →
    > 0.58). [commit] [revert] [refine rule]"
- **When** I click [commit]
- **Then** the preview layer becomes the dirty layer (bg
  `#FAFFEA`); diff strip records "23 agent-suggested edits applied,
  rule: outlier (z > 2.5)"
- **When** instead I click [refine rule]
- **Then** an inline rule editor opens with three sliders
  (`z-threshold`, `window-days`, `action`) and the agent says:
    > "Tune the rule. Candidate count updates live."
- **And** as I move the `z-threshold` slider the candidate count
  re-computes in real time and the marked cells in `#table` add /
  remove their outline — bidirectional spec sync — **F3**

### Scenario · the reverse — chat-led, table-confirmed

- **Given** I am on `/forecast?station=2091` in fused mode
- **When** I type into the agent: "find all winter readings above
  20 °C and null them"
- **Then** the agent emits a `pencil` event marking N matching
  cells with dotted outlines
- **And** the agent's reply quotes the rule it parsed:
    > "Rule: month ∈ {12, 1, 2} ∧ wt_C > 20. Found **6** cells. They
    > are marked in `#table`. Want me to null all? [yes] [refine]"
- **And** the canvas auto-scrolls `#table` to the first marked cell
- **When** I scroll past one of the marked cells, I can click it
  to deselect (the outline becomes ghosted)
- **And** the agent's tally updates: "5 selected (1 deselected)"
- **When** I click [yes] in the agent reply
- **Then** the 5 remaining cells null in preview mode
- **And** the same commit / revert / refine flow applies

### Acceptance criteria

- [ ] Single edit → intent inference within 300 ms; bubble anchored
      to the edited cell (F1)
- [ ] Intent classifier covers ≥ 3 patterns initially: outlier
      removal, threshold replacement, NaN backfill
- [ ] Agent marks candidate cells in `#table` with a 1 px dotted
      UniBe-red outline (distinct from the dirty bg) (F6)
- [ ] Action row above `#table` for batch actions (null /
      interpolate / keep)
- [ ] Preview layer: cells render with `#FBF8E8` bg + italic; not
      committed until [commit]
- [ ] Charts (`#fan`, `#kpi`) re-render against the preview layer
- [ ] Confirm-bar at top of canvas shows preview's downstream KPI
      impact (e.g. MAE delta)
- [ ] Rule editor: 3 sliders + live re-computation of candidate
      count + live re-rendering of outlines (F3)
- [ ] Reverse flow: NL → pencil event → cell outlines + tally;
      click cells to opt out
- [ ] Refused edits get a [ignore] state; agent doesn't re-suggest
      the same pattern within the session
- [ ] All agent edits show `via_agent: true` in the diff; one-click
      undo (F5)
- [ ] Performance: 23-cell preview render ≤ 200 ms total
- [ ] A11y: dotted outline pattern is colour-blind safe (test under
      protanopia simulator)

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| Excel "find similar" | Find Similar is generic. The agent infers the *semantic intent* (outlier vs. unit conversion vs. backfill) and offers a domain-aware extension |
| pandas + a notebook | Agent renders the candidate set *spatially in the table*, not as a print of N indices. User can opt in/out per cell. |
| AI suggestions in a side panel | Suggestions are anchored to the edit (F1) and to the candidate cells (F6); user doesn't context-switch |
| Macro recording / templating | Rules are *editable as parameters* (F3 sliders + spec sync), not opaque recordings |

### Non-goals (mode C, v1)

- Predicting the next edit based on *time-of-day patterns* or the
  user's history across sessions (deferred)
- Cross-dataset rule transfer (a rule learned on dataset A applied
  on dataset B)
- Voice-driven edits
- Rules that depend on external API calls (e.g. "compare to
  MeteoSwiss" — that's story 5 territory)

---

## Dependencies

- `liulian-api` (NEW):
  - `POST /datasets/{id}/transforms/preview` — returns the preview
    layer for a given rule
  - `POST /datasets/{id}/transforms/commit` — finalises a transform
  - `POST /datasets/{id}/quality-scan` — agent-callable; returns
    candidate cells for a given rule
- `liulian-agent` events / tools (NEW):
  - `cell_edit(date, station, column, new_value, reason?)` tool
  - `find_suspects(rule)` tool returning marked cells
  - `pencil` event with `{ cell_ids[], outline: 'dotted'|'solid' }`
  - `preview_layer` event with `{ cells: [{id, new_value, why}] }`
  - `intent_classified` event from a hooked classifier
- `liulian-web` (NEW):
  - Preview-layer rendering in `<DataTablePanel/>` and `<FanPanel/>`
  - Anchored intent bubble (shared primitive with story 1, 2, 3)
  - Rule-editor inline drawer (3 sliders + live candidate count)
  - Multi-cell outline render with cell opt-out
- `liulian-design-system`:
  - Tokens for preview-layer bg, dotted-outline pattern
  - Confirm-bar primitive

## Open questions

- Intent classifier: heuristic only, or LLM? Heuristic is instant
  but misses semantic patterns. Author leans: heuristic for first
  pass, LLM enrichment within 1 s.
- When the same cell is edited by user AND by agent in the same
  session, who wins? (Author leans: last-write-wins; both visible
  in diff history.)
- Rule persistence: should a confirmed rule become a *named
  transform* available on future imports? (Author leans: yes, as
  an optional checkbox at commit time; ties into story 2.)
- Edit on a cell that drives a forecast (e.g. an observation row in
  the recent window): does the agent re-run the forecast with the
  edit baked in? (Author leans: yes for sandbox preview; only on
  commit for the canonical forecast.)

## Linked specs / ADRs

- Story 2 (import) — transforms registered at commit
- Story 3 (auto-BI) — what re-renders on edit
- Story 5 (add data) — adds rows; same diff strip
- Forecast dashboard spec R8 §10 (a11y) — colour-blind constraints
- ADR 0005 (three-entity unified tracker table) — transforms are an
  entity in the tracker
