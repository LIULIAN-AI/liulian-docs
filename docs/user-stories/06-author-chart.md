# Story 6 · *The Easel* — author a custom chart by code, canvas, or sketch

**Codename:** *The Easel* (画板) — three ways to make a chart that
doesn't exist on the picker yet.

**Persona:** *Researcher-Linlin* needs a chart no template covers —
a polar plot of seasonal water-temperature variance per station, or a
sankey of station-pair correlations, or a custom small-multiples
grid she's seen in a paper. *Advisor-Sophia* wants to see "can an AI
really build a chart from a sketch?" as her vote-of-confidence test.
*Investor-on-phone* wants the demo to end with "and look, I just
drew this and the AI made it real".

**Goal:** Author a non-template chart through one of three routes
— writing code / API spec; building it with a canvas-style chart
editor (paint by selection); or sketching on a blank canvas and
letting the agent translate the sketch into a chart.

**Motivation:** The chart-picker covers the common 10. The 11th
chart is the one that makes a paper publishable, an audit credible,
or a pitch memorable. A platform that *only* offers a picker dies on
contact with a researcher's actual need. The three authoring routes
let the user pick the route that matches their fluency: code
(researcher), canvas builder (designer), sketch (everyone else).

---

## Mode A · No-bot (classical)

Three sub-modes within mode A itself, matching the three authoring
routes. No agent involvement.

### A.1 · Write a LiulianChartSpec (code)

- **Given** I am on `/forecast` and click `[+ author chart]`
- **Then** a modal opens with three tabs: **Code** · **Canvas** ·
  **Sketch**
- **When** I select **Code**
- **Then** a monaco-style editor opens with a starter LiulianChartSpec
  JSON template
- **When** I edit the JSON and click [preview]
- **Then** the spec is sent to a `/charts/render?spec=…` endpoint and
  the rendered chart appears in a preview pane
- **When** I click [commit]
- **Then** the chart becomes a new panel on the canvas; spec stored
  in the dashboard

### A.2 · Canvas-style chart builder

- **When** I select **Canvas** tab
- **Then** a *chart workbench* opens: a blank rectangular area on the
  left ("the easel") and a properties panel on the right
- **And** the right panel has 4 sections:
    1. **Marks** — point, line, area, bar, arc, rule (drag onto easel)
    2. **Encodings** — x, y, color, size, shape (drag a column into
       each slot)
    3. **Style** — palette, axes, title, legend
    4. **Data** — which dataset / which columns
- **When** I drag the `line` mark onto the easel
- **Then** an empty line chart appears
- **When** I drag column `date` to the `x` slot, `wt_C` to `y`, and
  `station` to `color`
- **Then** the easel renders the multi-line chart
- **When** I tweak the palette
- **Then** the easel updates
- **When** I click [commit]
- **Then** the chart becomes a panel; the underlying
  `LiulianChartSpec` is auto-derived from the workbench's state

### A.3 · Sketch (no agent yet)

- **When** I select **Sketch** tab in mode A
- **Then** a blank canvas appears with a single message: "Sketching
  requires the AI assistant — please switch on the agent or use the
  Canvas tab"
- **(rationale: pure sketch-to-chart without an agent is not
  feasible — sketch interpretation requires the LLM. Mode A's
  Sketch tab is intentionally a stub that nudges the user.)**

### Acceptance criteria

- [ ] `[+ author chart]` opens the 3-tab modal
- [ ] Code tab: monaco editor, JSON validation against
      `LiulianChartSpec` schema, preview button, commit button
- [ ] Canvas tab: easel + 4-section property panel; drag-drop
      column-to-encoding; live preview on every change
- [ ] Sketch tab is present but in mode A shows the redirect message
- [ ] Committed chart becomes a panel with its own panel-id
      (`#custom-<short-id>`)
- [ ] The chart's spec is editable later via the existing spec
      drawer (`[</>]`)
- [ ] Errors in JSON spec / unsupported encoding combos surface
      inline (the field with the error highlighted)
- [ ] Editor undo / redo (`Ctrl+Z`/`Ctrl+Shift+Z`)

---

## Mode B · All-bot (radical)

The user describes the chart they want; the agent emits a spec; the
chart renders. No editor surfaces — only conversation.

### Scenario

- **Given** I am in mode "AI-first"
- **When** I type to the agent: "build me a heatmap of station by
  month showing average water-temperature"
- **Then** the agent emits a `panel_create` SSE event with a
  LiulianChartSpec:
    > chart_type: `heatmap` · x: `month(date)` · y: `station` ·
    > value: `mean(wt_C)`
- **And** the agent's reply explains the spec:
    > "Rendering a heatmap. Rows = stations, columns = months,
    > cell colour = average wt_C. Palette UniBe-red gradient. Want a
    > different palette or aggregation? [median] [max] [ocean
    > palette]"
- **When** I tap [median]
- **Then** the agent re-emits the spec with `value: median(wt_C)`
- **And** the chart re-renders
- **When** I type "now make a small-multiples version, one chart
  per station, showing wt_C over years"
- **Then** the agent emits a new chart with chart_type `multiples`
  + faceting on `station`

### Acceptance criteria

- [ ] Agent intent `build_chart` parses NL → LiulianChartSpec
- [ ] Agent's reply must enumerate the spec choices (chart_type,
      encodings, aggregation) in plain English
- [ ] User can refine via NL ("change the aggregation to max",
      "rotate axes", "add a colour legend")
- [ ] Each refinement creates a new spec version (visible in story 3
      diff history)
- [ ] No editor visible in mode B — pure conversation + rendered
      chart

### Risks

- The agent can over-render (build a chart from an ambiguous
  prompt that wasn't quite what the user wanted). Mitigation: when
  ambiguity is high, the agent asks for clarification before
  rendering ("did you mean per station or per basin?").
- Some chart types are computationally expensive (sankey on 30 k
  rows). Agent must surface "this will take ~5 s" pre-render.

---

## Mode C · Fused (innovative) · **headline mode** · the sketch-to-chart loop

The third tab — **Sketch** — comes alive in fused mode. The user
draws a rough shape on a blank canvas; the agent interprets it as a
chart-type + encoding; the chart renders; the user redlines /
strokes on the rendered chart; the agent updates the spec; loop.

### Innovation note

Uses **F1** (agent's interpretation appears anchored to the user's
last stroke), **F3** (bidirectional sync — sketch updates the spec,
spec updates the sketch interpretation), **F4** (every
interpretation cites the stroke it came from), **F5** (every
interpretation has [accept] / [reject] / [refine]), **F6** (agent
draws its *interpretation strokes* on the same canvas, in a distinct
color, so the user sees what the agent thinks of each stroke).

The headline interaction is the **sketch chain**: the user paints,
the agent paints back its interpretation in a distinct color, the
user refines, the agent updates. The same canvas hosts both the
user's rough strokes and the rendered chart, plus the agent's
interpretation strokes. By the end of the loop, the canvas *is* the
chart.

### Scenario · the sketch chain

- **Given** I click `[+ author chart]` and select **Sketch** tab
- **Then** a blank rectangular canvas opens (with a faint grid)
- **And** the toolbar above the canvas has 4 tools: ✎ stroke ·
  ⬚ box · ┄ axis · 🗲 erase
- **And** the right rail shows an agent persona chip (collapsed)
- **When** I draw a downward-trending curve from top-left to
  bottom-right using the stroke tool
- **Then** within 600 ms the agent interprets the stroke as "line
  chart, time-on-x, declining-y"
- **And** the agent's *ghost stroke* (a 1 px dashed UniBe-red line)
  overlays the user's stroke
- **And** an anchored bubble appears at the end of the user's
  stroke:
    > "A line chart? I read your stroke as a downward trend over time.
    > [yes — render] [no — adjust] [it's a curve, not a line]"
- **When** I tap [yes — render]
- **Then** the canvas's user-stroke fades; a real chart renders in
  its place using the active dataset's first time-series target
- **And** the spec drawer slides over showing the auto-derived spec
- **When** I draw a vertical bracket on the rendered chart
- **Then** the agent interprets the bracket as "select this date
  range" and emits an annotation that focuses the chart on that
  range
- **And** the agent asks: "Focus on this range? [yes] [no]
  [or — split the chart into two by this break]"
- **When** I tap [split]
- **Then** the canvas re-renders as two side-by-side panels (before /
  after the break) — bidirectional spec sync (F3)
- **When** I draw a small grid icon (3×3 dots) in the corner
- **Then** the agent interprets it as "convert to small-multiples"
  and offers: "Group by which field? [station] [year] [basin]"
- **When** I tap [station]
- **Then** the chart becomes a 3-cell small-multiples by station
- **When** I am satisfied I click [commit]
- **Then** the new chart becomes a panel; the spec drawer shows the
  spec; the canvas closes

### Scenario · code + sketch hybrid

- **Given** I drew a sketch and accepted the agent's chart
- **And** the spec drawer is open
- **When** I edit one key in the JSON ("change `chart_type` to
  `area`")
- **Then** the chart re-renders as an area chart
- **And** the agent simultaneously updates its sketch interpretation
  (the ghost stroke morphs to suggest "fill area under curve") — F3
- **When** I then sketch a small "shadow" under the curve
- **Then** the agent interprets that as a Q05-Q95 band request and
  emits: "Add a fan? [yes — Q05-Q95] [no, just gradient fill]"

### Scenario · code-side authoring with agent narration

- **Given** I am in the **Code** tab in fused mode
- **When** I type the start of a JSON spec
- **Then** the agent's anchored bubble next to the editor reads:
    > "Building a `chart_type: 'heatmap'` — you'll need `x`, `y`,
    > and a `value` encoding. Try `month(date)` on x?" (F2)
- **And** the agent prefetches the available aggregation functions
- **When** I make a typo (`charrt_type`)
- **Then** the agent says: "Typo — `charrt_type` should be
  `chart_type`. Fix it?" with a one-click fix button (F5)

### Acceptance criteria

- [ ] Sketch tab opens a blank canvas with 4 tools
- [ ] User strokes captured as SVG path data
- [ ] Each stroke (or stroke cluster within 800 ms of each other)
      sent to the agent for interpretation
- [ ] Agent's ghost stroke (1 px dashed UniBe-red) overlays the
      user's stroke (F6)
- [ ] Anchored interpretation bubble at the stroke's end point
      with [accept] / [reject] / [refine] (F4, F5)
- [ ] On accept, the canvas transitions from sketch to rendered
      chart (user stroke fades)
- [ ] Subsequent strokes on the rendered chart are
      interpreted-and-applied (vertical bracket → focus; grid icon →
      small-multiples; curve → second series; etc.)
- [ ] Spec drawer can be opened anytime; bidirectional sync
      (sketch → spec, spec → sketch interpretation) (F3)
- [ ] Code-tab fused: anchored agent suggestions next to the editor;
      typo corrections; aggregation suggestions; available-fields
      autocomplete (F2)
- [ ] Mobile portrait < 480 px: sketch tab uses touch strokes;
      smaller toolbar; interpretation bubble becomes inline
- [ ] Sketch + agent fallback: if agent is offline, sketch tab
      degrades to the Canvas tab (no interpretation, only
      drag-drop)

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| draw.io / Lucidchart | Drawing a *diagram*, not a chart. LIULIAN draws strokes that *become bound data charts*. |
| Tableau "show me" recommended chart | Recommendation is a button. LIULIAN's interpretation is a *bidirectional collaboration* on the same canvas (F6). |
| Excalidraw with AI export | Excalidraw exports the drawing; LIULIAN replaces the drawing with a real chart bound to data. |
| Code-only chart authoring (matplotlib + Jupyter) | Code stays available but is *augmented* with anchored agent hints, not replaced. |

### Non-goals (mode C, v1)

- Pen-pressure sensitivity (use simple line strokes for v1)
- Multi-stroke gestures (interpret each stroke independently for
  v1; chain via the loop, not via gesture detection)
- 3D charts (Three.js integration is a separate story)
- Hand-drawn typography on the chart (text labels via spec)
- Sketch-to-Plotly / Vega-Lite export (LIULIAN spec is the only
  output format for v1)

---

## Dependencies

- `liulian-api` (NEW):
  - `POST /charts/render` — renders a `LiulianChartSpec` to SVG
  - `POST /charts/interpret-sketch` — takes SVG path data + dataset
    context, returns a candidate spec (calls the agent under the
    hood)
- `liulian-agent` tools (NEW):
  - `build_chart(description)` — for mode B
  - `interpret_sketch(svg_paths, dataset_ref)` — for mode C
  - `refine_chart(current_spec, user_intent)`
- `liulian-web` (NEW):
  - `[+ author chart]` modal with 3 tabs
  - `<ChartCodeEditor/>` (monaco-style; spec validation)
  - `<ChartCanvasBuilder/>` (the easel + property panel)
  - `<ChartSketchTab/>` (sketch canvas + agent interpretation overlay)
  - LiulianChartSpec type lives in `liulian-web/lib/chartSpec.ts`
- `liulian-design-system`:
  - LiulianChartSpec JSON schema (canonical)
  - Tokens for sketch ghost-stroke colour, dashed pattern
  - 4-section property panel primitive

## Open questions

- Sketch interpretation latency target: must be ≤ 1.5 s to feel
  responsive; what's the floor on agent infra? (Author leans: pre-
  warmed model; sketch is rare enough that occasional slow
  interpretations are acceptable if rare.)
- Should the agent learn the user's stroke patterns over time
  (e.g. their personal "small-multiples icon")? (Deferred to a
  future personalisation story.)
- Code-tab fused mode: should the agent be allowed to *auto-fix*
  typos or only suggest? (Author leans: suggest, never auto-fix
  without one click.)
- Persistence of sketches: should the original sketch be stored with
  the chart for later reference, or discarded once the chart is
  committed? (Author leans: stored as metadata on the spec, viewable
  via a "see the sketch" toggle.)

## Linked specs / ADRs

- Story 3 (auto-BI) — author-chart panels join the bento with the
  auto-layout
- Story 4 (edit) — sketch interpretation must respect dirty cells
- Story 5 (add data) — sketches that reference unavailable columns
  prompt a graft-extend hint
- ADR 0008 (canvas orchestrator reuse) — sketch interpretation is
  an orchestrator pattern
- LiulianChartSpec (defined in spec R8 §6)
