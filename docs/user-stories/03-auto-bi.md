# Story 3 · *First Light* — data lands, the dashboard lights up

**Codename:** *First Light* (亮起) — the moment after import when the
canvas reveals itself for the first time.

**Persona:** *Researcher-Linlin* (just imported a dataset, story 2)
or *Investor* (just opened a shared link, story 5 of an earlier
session). The user has done one of:

- imported a CSV moments ago
- loaded a demo (swiss-river-1990) from the home page
- opened a saved dashboard link

What they want next is for the canvas to *not be empty*. A useful
default dashboard should appear automatically — with the panels a
reasonable analyst would pick for this *shape* of data — without
demanding configuration.

**Goal:** On dataset landing, render a default 4–6-panel dashboard
that's *opinionated, correct, and inspectable*. Let the user click
sub-pages / panels to drill in; let them swap panels at will.

**Motivation:** Power BI hits the user with an empty canvas and a
"drag a field" hint. Tableau the same. The empty-canvas problem
costs every new user about 5 minutes of friction. LIULIAN can
*choose for them*: time-series of the target field, a station-map
when there's `station_id`, a multi-station overlay, a quality KPI
strip. If the choice is wrong, the user changes it; if it's right,
they're already analysing.

---

## Mode A · No-bot (classical)

A deterministic auto-layout based on schema inference. Five panels
chosen by rules; no AI.

### Auto-layout rules

| Detected schema | Panels chosen |
|---|---|
| time series + 1 target, no id | `#fan` (target over time), `#kpi` (recent stats), `#table` (last 60 rows), `#picker` (chart-type swap) |
| time series + 1 target + 1 id (multi-station) | above + `#map` (if id has lat/lon meta) + `#mm` (multi-id overlay), `#sev` (anomaly ribbon) |
| time series + N targets | `#fan` (first target) + `#picker`, target dropdown above `#fan` |
| categorical only | `#bar` (counts by category), `#table`, `#picker` |
| flat tabular, no time | `#table` (full table), `#kpi` (numeric column stats), `#picker` |

### Scenario

- **Given** I just committed a CSV import (story 2)
- **And** the dataset is inferred as `time series + 1 target + 1 id`
- **When** the browser navigates to `/forecast?dataset=<id>`
- **Then** within 1.5 s the canvas renders the 6-panel default
- **And** the run-header shows `<dataset-name> · <stamp> · default
  dashboard · unsaved`
- **And** the left rail (`#stations`) lists every id in the dataset
- **And** the hero `#fan` shows the first id's target over time
- **And** the chart-type picker above `#fan` is available
- **When** I click another id in the rail
- **Then** the URL updates `?station=<id>` and all panels cross-filter
- **When** I click a chart-type tile in `#picker`
- **Then** the hero panel re-renders as that type
- **When** I navigate to `/forecast?dataset=<id>&page=quality`
- **Then** the canvas swaps to a quality-focused sub-page (`#kpi`
  table is hero; `#fan` becomes a residual histogram; `#table` is
  the dirty-rows list)
- **And** the URL parameter `page=` controls the layout family
- **When** there's no `id` column, only a time series
- **Then** `#stations`, `#map`, `#mm` are absent; the layout
  re-flows to fill the space with `#table` wider and `#kpi` more
  prominent

### Acceptance criteria

- [ ] On dataset land, default dashboard renders ≤ 1.5 s for a
      typical < 50 k-row dataset
- [ ] Auto-layout decision matrix has 5 entries covering 95 % of
      schema shapes (the four listed + a fallback "we can't tell
      → table-only")
- [ ] Each chosen panel is sized via the existing 12-col bento; no
      hard-coded pixel sizes
- [ ] Sub-pages (`?page=quality`, `?page=compare`, `?page=ops`)
      route to canonical layouts
- [ ] Station rail driven by the dataset's id column (whichever
      column inherited the `id` semantic tag)
- [ ] Cross-filter on id click updates all panels in < 200 ms
- [ ] Chart-type picker behaves as in story 6 / spec R8 §5
- [ ] If schema inference returned ambiguous results, panels render
      a placeholder "configure this panel" CTA instead of garbage
- [ ] Sub-page navigation is preserved when sharing the URL (story 5
      mode A also depends on this)
- [ ] Reds-per-viewport ≤ 2 still honoured
- [ ] All panels have edit-toolbar `[</>]` `[⌗]` (still pending
      story 6 wiring)

---

## Mode B · All-bot (radical)

Instead of an auto-layout, the agent asks "what do you want to see"
and constructs the dashboard panel by panel from the conversation.

### Scenario

- **Given** the user is in mode "AI-first"
- **And** the dataset just committed
- **When** the browser navigates to `/forecast?dataset=<id>`
- **Then** the canvas is empty except for the agent rail and a
  run-header
- **And** the agent immediately greets:
    > "Loaded `swiss-temp-2025`: 5 240 rows, 3 stations,
    > 2010-01-01 → 2024-12-31. What do you want to see first?
    > [the time series for one station] [all stations overlaid]
    > [a quality summary] [something else]"
- **When** I tap [all stations overlaid]
- **Then** the agent renders a multi-station overlay in the centre
  of the canvas (one big panel) and confirms:
    > "Overlay shown. What next? [add a station map] [show a quality
    > KPI strip] [drill into one station]"
- **When** I tap [add a station map]
- **Then** a map panel appears to the right of the overlay
- **And** the agent continues offering next options
- **When** I type "this is fine, save it"
- **Then** the agent says "Saved as `dashboard_a4f2`. Share link
  copied to clipboard." and the run-header updates accordingly

### Acceptance criteria

- [ ] On dataset land, canvas is empty except for agent rail
- [ ] Agent greets within 500 ms with a context-aware sentence
- [ ] Each agent suggestion creates one panel via a `panel_create`
      SSE event
- [ ] Each panel has the same affordances as a mode-A panel (chart
      picker, edit toolbar)
- [ ] User can take over at any point — clicking `[ + add panel ]`
      at the bottom opens the chart picker (mode A) without leaving
      mode B
- [ ] Save flow integrates with story 5

### Risks

- An empty canvas with a chatbot can feel intimidating. The
  *agent's first greeting must offer concrete starting points*, not
  open the floor. The four suggestion chips are mandatory.

---

## Mode C · Fused (innovative) · **headline mode**

Mode A's auto-layout *plus* the agent narrates each panel as it
lands, calls out anomalies, and pre-fetches the most likely
drill-down. The agent's narration is **spatially anchored to each
panel**, and when the user hovers a panel the narration shifts
focus to that panel.

### Innovation note

Uses **F1** (anchored narration), **F2** (predictive prefetch of
drill-downs), **F4** (visible reasoning — every callout has a
data-evidence link), **F6** (shared cursor — agent can highlight
data points across panels).

The headline interaction is **the canvas narration choreography**:
as panels render one by one, the agent's commentary streams next to
each panel in sequence — like a museum audio guide. The first 8
seconds of post-import experience are *narrated*. Then the agent
falls silent and waits for the user to lead.

The second headline interaction is **predictive drill-down**: when
the user hovers a panel for ≥ 1 s, the agent prefetches the
relevant drill-down so the click feels instant.

### Scenario

- **Given** I am in fused mode
- **And** the dataset just committed
- **When** the browser navigates to `/forecast?dataset=<id>`
- **Then** panels render in *staggered* sequence over ~1.5 s
  (stations rail at 0.2 s, `#fan` at 0.4 s, `#table` at 0.6 s,
  `#mm` at 0.8 s, `#map` at 1.0 s, `#kpi` at 1.2 s)
- **And** as each panel appears the agent emits a one-sentence
  anchored callout next to it:
    1. by `#stations`: "3 stations in this dataset. Station 2091 has
       the longest series — using it as default."
    2. by `#fan`: "Station 2091 trends downward in winter; current
       value 5.83 °C is 1.4σ below the seasonal mean.<sup>1</sup>"
    3. by `#table`: "Last 60 days shown. 23 NaN rows interpolated
       during import.<sup>2</sup>"
    4. by `#mm`: "All 3 stations plotted; station 2034 lags by ~24 h.
       <sup>3</sup>"
    5. by `#map`: "Topology inferred from station-id prefixes. Tap a
       station to focus."
    6. by `#kpi`: "Forecast backtest on the last 14 days: MAE 0.62 °C
       — within the platform's healthy-band."
- **And** each `<sup>` is clickable and scroll-anchors to the source
  data point — **F4**
- **And** after 8 s of narration the agent goes silent and the rail
  collapses to a tight chip
- **When** I hover `#fan` for 1 s
- **Then** the agent silently prefetches the drill-down (the 14
  worst-residual windows, the seasonal residual histogram, the
  scenario diff for +18 % precip) — **F2**
- **When** I click `#fan`
- **Then** the drill-down opens in < 200 ms (because of the prefetch)
- **And** the agent re-emerges with: "Top 3 residual windows for
  PatchTST on 2091. Want me to compare with chronos-2?"
- **When** I drag a date range on `#fan`
- **Then** the agent prefetches the cross-filtered version of every
  other panel; release → all panels cross-filter in < 200 ms
- **When** the dataset has an obvious anomaly (e.g. a 4σ spike)
- **Then** the agent's narration *re-orders* to start with that
  anomaly: "Station 2034 has a 4σ spike at 2020-12-15. Inspecting
  first.<sup>★</sup>"
- **And** the agent's pointer-cursor (a 4 px UniBe-red ring) highlights the
  spike point on `#fan` — **F6**

### Acceptance criteria

- [ ] On dataset land, panels render in staggered sequence (0.2 s
      apart) to give the narration time to anchor
- [ ] Each panel callout: anchored bubble (F1) + one sentence + at
      least one `<sup>` footnote linking to the source data
      (F4)
- [ ] Narration ends after the last panel (≤ 8 s total)
- [ ] Predictive prefetch: hover ≥ 1 s on a panel → drill-down
      query fires; user click renders the drill-down in < 200 ms
      from cache (F2)
- [ ] Cross-filter (drag a range on `#fan`): all panels update in
      < 200 ms thanks to prefetch
- [ ] Anomaly-aware re-ordering: if the dataset has an outlier
      with z > 3, the narration starts with that panel
- [ ] Agent's pointer-cursor: 4 px red ring rendered atop a data
      point when the agent's `highlight` event fires (F6)
- [ ] Narration is dismissible: user clicks any panel before the
      narration finishes → narration silences immediately
- [ ] If the agent is unavailable, fall back to mode A (no
      narration, no prefetch)
- [ ] Telemetry: narration_completed / narration_interrupted /
      drill-down_click_after_prefetch counters

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| Auto-dashboard generators (e.g. Sweetviz) | LIULIAN's auto-layout is *narrated* — each panel arrives with a one-sentence why and a data-anchored callout |
| Onboarding tour / tooltips | Tour is one-time and generic. Narration is *data-specific* and only fires on the first ~8 s after landing |
| Predictive autocomplete | LIULIAN's prefetch isn't on text input — it's on *spatial dwell*. The chart computes its drill-down before the click |
| AI chat in a side rail | Agent voice is anchored to the panel its sentence applies to; the user's eye and the agent's words share a location (F1) |

### Non-goals (mode C, v1)

- Voice / audio narration (text-only callouts for v1)
- Narration in languages other than English / Chinese (i18n later)
- "Narrate again" replay button (user can refresh the page)
- Narration of every column individually in `#table` (too noisy)
- Pre-computed drill-downs for every panel — only the hovered one

---

## Dependencies

- `liulian-api` (NEW):
  - `GET /datasets/{id}/auto-layout` — returns the chosen panel set
    given the schema (mode A logic)
  - `GET /datasets/{id}/highlights` — returns 1-3 callout-worthy
    facts about the dataset (used for anomaly-aware re-ordering)
  - `GET /panels/<id>/drill?range=…` — drill-down precomputed
- `liulian-agent` events:
  - `panel_create` event for mode B
  - `panel_callout` event with `{ panel_id, message, sup_anchors[] }`
  - `highlight` event with `{ panel_id, x, y, color }`
  - `prefetch` event (no UI change; client-side cache write)
- `liulian-web` (NEW):
  - Mode-aware orchestration: which mode is active determines whether
    auto-layout fires, callouts stream, prefetch runs
  - Staggered panel render (animation timing per panel)
  - Anchored-callout primitive (shared with story 1, 2)
  - Drag-range cross-filter on `#fan`

## Open questions

- When the dataset is large enough that auto-layout takes > 3 s, do
  we render placeholder panels and stream content in, or wait?
  (Author leans: placeholders, fade in as data arrives.)
- How do we A/B-test the narration's effect on engagement? (Author
  leans: a `?narrate=0` flag toggles it off; compare retention.)
- "Anomaly-aware reordering" — what threshold defines an anomaly?
  (Author leans: z > 3 on the target column; configurable.)
- Drill-down panels: replace `#fan` or open in a new bento slot?
  (Author leans: new slot if there's grid room; replace `#fan` with
  a back chip otherwise.)

## Linked specs / ADRs

- Story 2 (import) — what produces the dataset
- Story 4 (edit) — what happens when the user edits a value during
  exploration
- Story 5 (add) — extending the dataset
- ADR 0008 (canvas orchestrator reuse) — the auto-layout engine is
  one of those orchestrators
- Forecast dashboard spec R8 — defines the 8 canonical panels
