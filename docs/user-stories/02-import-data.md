# Story 2 · *The Annotated Drop* — import data with AI-annotated schema

**Codename:** *The Annotated Drop* (落档) — the moment a CSV lands
and the schema is born with annotations attached.

**Persona:** *Researcher-Linlin*. Has a CSV on her laptop (or a URL,
or a remote sensor feed). Wants to get it into LIULIAN and verify
it imported correctly — schema, types, row count, date range,
obvious data quality issues — before doing anything else.

**Goal:** From the home page (story 1) or from `/datasets/new`,
load a dataset by drop-zone, URL paste, or pre-built demo selector;
see the schema + a sample + a quality flag panel; commit the import
so the dataset is available to the rest of the platform.

**Motivation:** Data import is the platform's first technical
proof. If schema inference is wrong, types are wrong, or the
import silently truncates rows, every downstream dashboard /
forecast / alert is poisoned. The import surface must show what it
*thinks* about the data before the user commits — and let the user
correct it.

---

## Mode A · No-bot (classical)

A standard data-import wizard. Three input methods (drop / paste /
demo), then a parse + preview + confirm step.

### Scenario

- **Given** I am on `/datasets/new`
- **Then** the page shows three method tabs at the top: **Upload**
  (active) · **URL** · **Demo dataset**
- **And** the active panel shows a large drop-zone (with `<input
  type="file" accept=".csv,.parquet,.arrow,.json">`), a body copy
  describing accepted formats and size limit (200 MB), and a
  secondary button "browse files"
- **When** I drop a CSV file into the drop-zone
- **Then** the file is uploaded via `POST /datasets/upload` with a
  progress bar
- **And** when upload completes the page transitions to a 4-section
  preview:
    1. **Detected schema** — table of column name · inferred type ·
       semantic tag (target / id / time / feature) · null %
    2. **Sample rows** — first 10 rows of the parsed data
    3. **Quality summary** — row count · date range · % nulls · # of
       duplicates · # of out-of-range values per column
    4. **Action bar** — primary `[import]` button, secondary `[edit
       schema]` button, tertiary `[discard]` button
- **When** I notice the time column is inferred as `string` instead
  of `datetime`
- **Then** I click `[edit schema]`
- **And** an inline schema editor lets me change types per column
- **When** I click `[import]`
- **Then** `POST /datasets/commit` finalises the import and the page
  navigates to `/forecast?dataset=<id>` (which triggers story 3 —
  auto-BI)
- **When** I switch to the **URL** tab and paste
  `https://storage.example.com/swiss-temp-2025.csv`
- **Then** the page fetches it server-side (with a 30 s timeout) and
  shows the same preview flow
- **When** I switch to the **Demo dataset** tab
- **Then** the page shows a 2x2 grid of preset datasets:
  swiss-river-1990 · electricity-uci · etth1 · pems-traffic
- **And** each card has a one-line description and a
  `[load]` button
- **When** I click `[load]` on a demo card
- **Then** the dataset is registered without an upload step and
  navigation proceeds as above

### Acceptance criteria

- [ ] `/datasets/new` is a server-rendered route with three tabs
- [ ] Drop-zone accepts `.csv`, `.parquet`, `.arrow`, `.json`; max
      size 200 MB
- [ ] Upload progress: bytes / total, percentage, ETA
- [ ] Schema inference uses pandas-style dtype rules + LIULIAN
      semantic-tag guesses (column names like `time`, `t`, `date`,
      `epoch_*` → `datetime`; `id`, `station_id` → `id`; etc.)
- [ ] Preview shows max 10 rows
- [ ] Quality summary computed server-side at parse time
- [ ] `[edit schema]` opens an inline editor with per-column type
      dropdown + semantic-tag dropdown
- [ ] `[import]` commits and navigates to `/forecast?dataset=<id>`
- [ ] URL tab supports https URLs only; rejects http with a one-line
      copy "https required for security"
- [ ] Demo tab shows 4 datasets with a one-line description each
- [ ] Errors handled: bad CSV (malformed) → red banner with line
      number; too large → clear "200 MB max" copy; URL timeout →
      "couldn't reach that URL within 30 s, try again or upload
      directly"
- [ ] Drop-zone keyboard-accessible (`<input type="file">` focusable)
- [ ] Files stored in api-side scratch with a TTL of 7 days for
      uncommitted; permanent on commit

---

## Mode B · All-bot (radical)

No drop-zone, no tabs, no schema editor. The agent walks the user
through the import in conversation. The drop-zone exists but only
appears when the agent asks for the file.

### Scenario

- **Given** the user has selected mode "AI-first"
- **And** I am on `/` (or in the agent rail anywhere)
- **When** I type "I want to import data"
- **Then** the agent replies:
    > "Sure — what kind of data? Tell me about it (CSV file, public
    > URL, sensor feed, demo dataset, …)?"
- **When** I reply "I have a CSV"
- **Then** the agent says:
    > "Drop it here." (an inline drop-zone widget renders within the
    > chat thread)
- **When** I drop `swiss-temp-2025.csv`
- **Then** the agent uploads + parses + replies:
    > "Got it. 5 240 rows, 4 columns, 2010-01-01 → 2024-12-31. The
    > columns look like:
    > - `date` (datetime, 0 % nulls) ✓ used as time
    > - `station` (string, 0 % nulls, 3 unique) ✓ used as id
    > - `wt_C` (float, 0.4 % nulls) ✓ used as target (water-temperature)
    > - `flag` (bool, 0 % nulls) ✓ used as quality flag
    >
    > Looks like a daily water-temperature multi-station series.
    > Any corrections, or should I import it as-is?"
- **And** the agent renders 4 line-items as a checklist (each can be
  toggled via a click)
- **When** I reply "looks right, import"
- **Then** the agent says "Importing…" then "Done. Opening the
  dashboard."
- **And** the agent navigates the user to
  `/forecast?dataset=<id>`
- **Alternate path:** when I reply "demo dataset"
- **Then** the agent says: "Pick one: swiss-river-1990 ·
  electricity-uci · etth1 · pems-traffic"
- **When** I reply "swiss-river-1990"
- **Then** the agent loads it and navigates

### Acceptance criteria

- [ ] Agent intent `import_data` parses to a tool call that opens
      the import flow inline
- [ ] Inline drop-zone widget renders in chat thread when the agent
      emits `widget: dropzone` event
- [ ] After upload, agent emits schema as a *checklist* widget
      (clickable items)
- [ ] User can correct the schema via natural language ("`station`
      should be category, not string"); agent updates the checklist
- [ ] Final confirmation via `import as-is` or `yes` keyword
- [ ] Agent emits `navigate` event after import succeeds
- [ ] No drop-zone, tab, or wizard chrome visible — only chat +
      inline widgets
- [ ] Error handling: bad CSV → agent says "this CSV has a problem
      on line N: <message>; want to fix the source and try again?"

### Risks

- The agent has to faithfully report quality issues — over-summarising
  ("looks fine") risks data corruption downstream. The mode-B prompt
  must require enumeration of issues, not a summary.
- Type inference disagreements are awkward in conversation. The
  fallback is "let me show you the editor" → opens mode A's schema
  editor inline.

---

## Mode C · Fused (innovative)

The drop-zone, tabs, and schema editor are present (mode A) **and**
the agent is co-acting on the data during import. The agent narrates
its own analysis next to each schema row, anchors warnings to specific
column cells, and proposes corrections that the user can accept with
one click.

### Innovation note

Uses **F1** (spatial anchoring of agent comments to schema rows),
**F4** (visible reasoning for every type inference), **F5**
(reversible by default — every agent-applied correction shows the
old → new and a one-click undo), **F6** (agent has a "pencil cursor"
that can mark cells in the sample preview).

The headline interaction is **schema co-authoring**: the schema table
on the left and the agent's commentary on the right share the
*same row index*. When the agent says "I think `wt_C` should be a
float, not a string — values look like `5.83`", that comment is
*anchored to the wt_C row in the schema table*. Click the comment and
the row highlights; click the row and the comment scrolls into view.

### Scenario

- **Given** I am on `/datasets/new` in fused mode
- **Then** the page renders the mode-A layout (three tabs + drop-zone)
- **And** a slim agent persona chip sits at the right edge of the
  page (collapsed)
- **When** I drop `swiss-temp-2025.csv` into the drop-zone
- **Then** the upload + parse runs as mode A
- **And** the schema preview renders on the left side of the page
- **And** simultaneously the agent expands the right rail (300 px)
  and starts streaming inline annotations *as the schema is being
  parsed*:
    > Column-by-column:
    > - **`date`** → datetime ✓ — values match `YYYY-MM-DD`, 0
    >   parse failures
    > - **`station`** → string ⚠ — only 3 unique values; consider
    >   `category` to save memory
    > - **`wt_C`** → float ✓ — range −2.4 to 28.1 °C plausible for
    >   surface water
    > - **`flag`** → bool ✓ — values are `True`/`False`
- **And** each annotation is anchored to the matching row in the
  schema table (1 px dashed UniBe-red line from the comment to the
  row) — **F1**
- **When** I hover the `station` annotation
- **Then** the matching schema row pulses softly (focus highlight)
- **When** I click the `station` annotation's [apply: category]
  inline button
- **Then** the schema editor row updates from `string` → `category`
- **And** a diff strip appears at the bottom: "Applied 1 agent
  suggestion. [undo]" — **F5**
- **When** the parser detects 23 rows where `wt_C` is `NaN`
- **Then** the agent marks those rows with a tiny red dot in the
  sample preview (the agent's "pencil cursor" is allowed to
  annotate cells) — **F6**
- **And** the agent's commentary adds: "23 rows with `NaN` `wt_C`
  — 0.4 %. Want to drop them, fill with linear interpolation, or
  keep as missing?"
- **And** three follow-up buttons appear: [drop] [interpolate] [keep]
- **When** I click [interpolate]
- **Then** the agent shows a preview of what the interpolation
  would do (e.g. "row 2 145: NaN → 6.31 °C, interpolated from
  rows 2 144 and 2 146") and asks "preview looks good? [apply]
  [adjust]"
- **When** I click [apply]
- **Then** the schema editor records "interpolate NaN in wt_C" as a
  pre-import transform; diff strip updates: "2 agent suggestions
  applied"
- **When** I click `[import]`
- **Then** the import commits with the agent transforms baked in
- **And** the next page (`/forecast?dataset=<id>`) shows a small
  "imported with 2 agent transforms — [see history]" pill in the
  header

### Acceptance criteria

- [ ] Right rail (300 px) renders agent commentary anchored to
      schema rows — **F1**
- [ ] Anchored line is 1 px dashed UniBe-red; hides on smaller
      viewport (< 1200 px wide) and falls back to inline annotation
- [ ] Every agent annotation has a one-sentence rationale + a data
      preview link (click → highlights the source rows in the sample)
       — **F4**
- [ ] Every agent-applied correction creates a diff strip entry
      with [undo]; undo reverts to the previous state — **F5**
- [ ] Agent's pencil cursor: a 3 px red dot rendered inline within
      a sample cell when the agent flags it; cell tooltip explains
      why — **F6**
- [ ] Pre-import transforms (drop / interpolate / type-cast) are
      recorded as a transform list on the committed dataset; visible
      via the dataset's `[see history]` pill
- [ ] Transforms can be re-played: opening the dataset later shows
      "imported with transforms: 1) cast station → category; 2)
      interpolate wt_C NaN". Each is undoable from the dataset detail
      page.
- [ ] User can refuse an agent transform: clicking the annotation's
      [ignore] dismisses without applying
- [ ] Mobile portrait < 480 px: agent commentary collapses to a
      bottom sheet; anchored lines become inline `[!]` badges
- [ ] Agent unavailable (api timeout): the import flow falls back
      to mode A; a one-time toast says "AI assistant unavailable;
      manual import mode"
- [ ] Performance: agent commentary streams via SSE; first
      annotation visible < 500 ms after upload completes

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| Schema editor (Tableau / Power BI) | Schema *plus* AI commentary anchored to each row — not "in a help panel" |
| AI chatbot for import (vendor X) | Agent does not *replace* the schema editor; it *augments* it. Manual override stays first-class. |
| Auto-imports with no preview | Every agent action is visible, reversible, and bears a rationale (F4 + F5) |
| Lint-style warnings in a list | Warnings are *spatially anchored* to the schema row — your eye doesn't have to translate "warning #3 → which column" (F1) |

### Non-goals (mode C, v1)

- AI-suggested joins with external datasets (deferred to story 5)
- Multi-file import in a single pass (one file at a time for v1)
- Inferring temporal sampling frequency automatically (manual choice
  in the schema editor)
- Custom transform DSL (only the 3 built-ins: drop / interpolate /
  cast)
- Persistent agent memory across imports (each import is a fresh
  conversation)

---

## Dependencies

- `liulian-api` endpoints (NEW):
  - `POST /datasets/upload` (existing as stub; needs real impl)
  - `POST /datasets/parse-preview` — returns schema + sample +
    quality summary
  - `POST /datasets/commit` — finalises the import
  - `GET /datasets/demo-presets` — returns the 4 demo cards
- `liulian-agent` events (NEW):
  - `widget` event for inline widgets (drop-zone, checklist) in chat
    thread
  - `annotation` event with `{ anchor: <row|cell|id>, message, actions[], rationale }`
  - `pencil` event with `{ cell_id, mark_type, why }`
  - `navigate` event with `{ to: <route>, in: <ms> }`
- `liulian-web` (NEW):
  - `/datasets/new` route with the three-tab + preview layout
  - Anchored-annotation primitive (shared with story 1 mode C)
  - Mode preference flag (shared with story 1)
- `liulian-design-system` annotation token (line colour, dash
  pattern, anchor connector)

## Open questions

- Should the AI's type inference be powered by a real LLM (better
  quality, latency 1-3 s) or a heuristic Python module (instant,
  uses pandas dtype rules)? Author leans: hybrid — heuristic first,
  LLM only for ambiguous columns (semantic tag guesses).
- Versioned datasets: when the same user re-imports a file with the
  same name + schema, do we overwrite, fork, or warn? Author leans:
  warn + prompt for a new version label.
- For the demo presets, do we copy the data into the user's
  workspace or just reference it? Reference is faster but couples
  the user to upstream changes.
- "Drop a folder" — supported for v1? Probably no; one file at a
  time keeps the UX clean.

## Linked specs / ADRs

- Story 1 (home onboarding) — entry path
- Story 3 (auto-BI) — what happens after import
- `liulian-python` manifest spec: defines the schema we infer
  toward
- ADR 0005 (three-entity unified tracker table) — informs how the
  imported dataset registers
