# Story 1 · *The Doorway* — first-time visitor lands and chooses a path in under 20 s

**Codename:** *The Doorway* (门厅) — the entrance hall where every
visitor is routed to the right room.

**Persona:** First-time visitor. Reached `liulian.app/` from an
advisor's WhatsApp link, a HackerNews post, or the LinkedIn "Try the
demo" button. They've never seen the product; they have 20–60 seconds
before they bounce. Half are Researcher-Linlin types (data people);
half are Operator-Marc types (domain experts); a few are investors.

**Goal:** Within 20 seconds of landing, the visitor has chosen one of
three paths and is on their way: **create a project**, **import data
straight away**, or **run a model on existing data**. The home page
is a separate surface from the BI canvas — its job is to *route*, not
to display data.

**Motivation:** A SaaS without a real landing page either dumps the
user into an empty canvas (confusing) or drops them on a marketing
page (zero affordance). LIULIAN needs a *third thing* — a curated
home that turns "I want to try this" into a specific next click in
under 20 seconds.

The home page is also the canonical entry for every flavour of
visitor: researcher, operator, investor. They each want a different
first action, so the three paths must be equally legible and equally
fast.

---

## Mode A · No-bot (classical)

A traditional three-card onboarding screen. Zero AI dialogue. Matches
what a Power-BI-coming user expects.

### Scenario

- **Given** I open `http://liulian.app/` for the first time (no
  recent sessions, no cookies)
- **Then** the page renders a centred container with the LIULIAN
  wordmark up top and three large cards below
- **And** each card has: a Lucide icon (Plus / Upload / Activity), a
  Fraunces-italic title, a one-sentence description, a primary
  button
- **And** below the cards: a thin "recent" strip — empty for a
  first-time user, otherwise three small chips of recent projects /
  datasets
- **When** I click **Create project**
- **Then** the page transitions to `/projects/new` with a 3-field
  form (name, description, default dataset)
- **When** I click **Import data**
- **Then** the page transitions to `/datasets/new` with the
  drag-and-drop import surface (Story 2 Mode A)
- **When** I click **Run a model**
- **Then** the page transitions to `/forecast/new` with a model
  picker (which dataset · which model · which station)
- **When** I click the LIULIAN wordmark
- **Then** I stay on `/`
- **When** I open the page on mobile portrait (< 480 px)
- **Then** the three cards stack vertically with the same affordances

### Acceptance criteria

- [ ] `/` is a server-rendered Next.js route, **not** the BI canvas
- [ ] Three primary cards, equal width on desktop (33% each, gap 24 px)
- [ ] Each card has: icon (24 px Lucide) · Fraunces title · body
      copy ≤ 80 chars · primary `<button>`
- [ ] Cards are keyboard-navigable (`Tab` cycles; `↵` activates)
- [ ] "Recent" strip below the cards: max 3 chips; empty on
      first visit (no placeholder copy beyond a single dash)
- [ ] Mobile breakpoint at < 480 px stacks vertically
- [ ] Routes: `/projects/new`, `/datasets/new`, `/forecast/new` all
      exist as separate pages
- [ ] No chat agent visible in this mode (the chat icon in the
      run-header is hidden when the mode preference is "classical")
- [ ] First contentful paint < 600 ms on broadband; Lighthouse
      perf ≥ 90
- [ ] Empty-state copy on the cards is concrete (e.g. "Drop a CSV"
      not "Get started")
- [ ] LIULIAN wordmark uses the WONK-italic U brand mark (from
      design-system tokens)

---

## Mode B · All-bot (radical)

A full-screen conversational onboarding. The three cards are gone;
the chatbot is the entire surface.

### Scenario

- **Given** the user toggles their mode preference to "AI-first"
  (settings → mode), and reloads `/`
- **Then** the page renders a near-empty canvas: LIULIAN wordmark at
  the top, a single Fraunces sentence in the middle reading "What
  brings you here today?", and a single input field below
- **And** below the input field: three example prompts as suggestion
  chips ("Show me a demo", "I have a CSV", "Train a model on
  swiss-river-1990")
- **When** I type "I have a CSV with daily water-temperature for
  three stations going back to 2010"
- **Then** the agent replies (SSE-streamed): "Nice — let's load it.
  Drag the file here, or paste a public URL, or pick it from your
  device. I'll inspect the header and tell you what I find."
- **And** a slim drop-zone appears in the middle of the page (single
  affordance — drop, paste, or click)
- **When** I drop the file
- **Then** the agent streams: "Got it. 4 columns: `date`, `station`,
  `wt_C`, `flag`. 3 stations, 5 240 rows, 2010-01-01 → 2024-12-31.
  Looks like daily water-temperature time series. Want me to set
  this up as a forecasting project, or just open it for inspection?"
- **And** the agent offers two follow-up chips: [forecasting project]
  [just inspect]
- **When** I tap [forecasting project]
- **Then** the agent navigates me to `/forecast?project=<id>` with a
  default canvas built from the uploaded data

### Acceptance criteria

- [ ] Mode preference flag `mode=ai-first` exists in user prefs
- [ ] When the flag is on, `/` renders the conversational layout
      instead of the three-card layout
- [ ] Centered input field with 3 suggestion chips
- [ ] Suggestion chips fire pre-baked agent prompts when clicked
- [ ] Drop / paste / click affordance appears when the agent
      requests data
- [ ] Agent streams via existing SSE protocol; no new endpoints needed
- [ ] Agent can route to `/projects/new`, `/datasets/new`,
      `/forecast/new` via a `navigate` SSE event
- [ ] Empty mode B (no message) shows the welcome sentence; no
      placeholder noise
- [ ] No three-card layout visible anywhere in mode B
- [ ] Keyboard: `/` focuses the input; `↑` recalls last prompt;
      `esc` blurs

### Risks (this mode)

- The agent can mis-route the user if intent is ambiguous. Fallback:
  if the user hesitates > 8 s after sending a message, show a "or
  click one of: [create project] [import data] [run model]" row.
- The conversational onboarding is *not* the default — it's a
  preference. Default is mode C (fused), because mode B alone is
  intimidating for first-time visitors.

---

## Mode C · Fused (innovative)

The **default** home page. Three cards *and* the agent are both
present, but they don't sit in separate panes — the agent is *spatially
anchored to the cards* and *predictively pre-fetches* the next step as
soon as the user shows interest in one card. This is the headline
interaction.

### Innovation note

Uses fused-mode principles **F1** (spatial anchoring), **F2**
(predictive prefetch), **F4** (visible reasoning), **F6** (shared
cursor).

The chatbot is not in a side rail. It manifests as **anchored
speech bubbles** that pop out *from the card the user is hovering*.
Before the user clicks anything, the agent has already loaded the
preview data for the hovered card so the next click is instant. The
agent also has a **ghost cursor** — a faint 4 px UniBe-red ring that
can outline a card to say "I recommend this one" without being
intrusive.

### Scenario

- **Given** I open `http://liulian.app/` (mode default = fused, no
  prefs touched)
- **Then** the page renders the three-card layout from Mode A …
- **And** a 32 px circular chip in the bottom-right corner shows the
  agent persona (Fraunces italic U) — collapsed by default
- **And** the agent is silently watching pointer events; no bubble
  yet
- **When** I hover the **Import data** card and stay for 1.5 s
- **Then** an anchored speech bubble pops out from the card's
  top-right corner reading:
    > "Got a CSV? Drop it on this card.
    > Or — I can pull `swiss-river-1990` (28 stations, 30 years,
    > daily water-temperature) and you'll have a dashboard in 6 s.
    > [load swiss-river-1990] [other ideas]"
- **And** in parallel, the agent *prefetches* the demo dataset's
  station list + the most-recent observations (so click → render
  is instant) — **F2**
- **And** the speech bubble has a faint UniBe-red border anchored to
  the card (line connecting bubble to card edge) — **F1**
- **When** I click **[load swiss-river-1990]** in the bubble
- **Then** the dataset is already prefetched; the page transitions
  to `/forecast?dataset=swiss-river-1990` in < 300 ms
- **And** the agent persona chip migrates from the home page corner
  to the canvas's agent rail
- **And** the agent's first canvas message says: "Loaded
  swiss-river-1990. I'm watching the first observation — station
  2091 — and the canvas is ready. What do you want to look at?"
- **When** (alternate path) I instead hover **Create project** for
  1.5 s
- **Then** the bubble there reads:
    > "Empty project, or fork an existing one? I have 3 templates:
    > river-monitoring · electricity-load · multi-tenant-demo. Tap
    > one to scaffold."
- **And** the same prefetch logic kicks in for the templates
- **When** (alternate path) I instead hover **Run a model**
- **Then** the bubble reads:
    > "Run PatchTST on a station? I can do that in 8 s on
    > swiss-river-1990. Or pick another model: [chronos-2]
    > [timesnet] [iTransformer]."
- **When** I dismiss a bubble by clicking outside or pressing `esc`
- **Then** the bubble retracts; the agent chip stays in the corner
- **When** I open the agent chip in the corner directly (without
  hovering any card)
- **Then** the agent rail expands into a centred conversation panel
  (mode-B behaviour available on demand)
- **When** I press `?` anywhere on the home page
- **Then** a help overlay shows the three keyboard shortcuts:
  `1/2/3` → activate card 1/2/3; `a` → open agent; `esc` → dismiss
  bubble

### Acceptance criteria

- [ ] **F1 · spatial anchoring**: anchored bubble has a 1 px UniBe-red
      line connecting it to the card; bubble follows card position on
      resize/scroll
- [ ] **F2 · predictive prefetch**: on `pointerenter` of a card
      followed by 1.5 s dwell, the agent issues the relevant prefetch
      requests (datasets list, templates, models); on click, the next
      page renders in < 300 ms from the cache
- [ ] **F4 · visible reasoning**: every CTA in a bubble has a
      one-sentence rationale (e.g. "swiss-river-1990 because it's our
      most-tested demo, 28 stations, real data, ready in 6 s")
- [ ] **F6 · shared cursor**: an `aria-hidden` ghost ring of 4 px
      UniBe-red appears on a card when the agent's `recommend` event
      fires; visible but not blocking
- [ ] No bubble appears unless dwell ≥ 1.5 s, to avoid noise on
      casual scrolling
- [ ] Only one bubble visible at a time (switching cards swaps the
      bubble; doesn't stack)
- [ ] Bubbles are dismissible by `esc`, outside-click, or another
      card's hover
- [ ] Agent persona chip in corner is 32 px; expanded conversation
      mode is a centred panel (not side rail) on the home page
- [ ] Keyboard shortcuts `1`/`2`/`3` activate the matching card;
      `a` opens agent; `?` shows help
- [ ] Mobile portrait < 480 px: bubbles appear as a bottom-sheet
      instead of anchored bubble (no room for anchoring); the rest of
      the F1-F6 logic continues server-side
- [ ] If the prefetch fails (api offline), the bubble copy degrades
      gracefully to "I can't reach the api right now; click anyway and
      I'll retry on the next page"
- [ ] Telemetry: bubble shown / bubble accepted / bubble dismissed
      counters, per card

### Why this is genuinely different

| Existing pattern | What fused mode does instead |
|---|---|
| Onboarding tour (Intercom-style) | No tour. The agent doesn't *explain* the cards — it *fills in the next step* before you click. |
| Chatbot in a side rail | No side rail on the home page. The agent speaks **from the card**, in the user's eyeline. |
| Suggestion chips on a chat input | Suggestions are **outcomes** ("[load swiss-river-1990]"), not topics ("[try a chat]"). |
| Tooltips on hover | Tooltips show static text; bubbles **prefetch data + offer concrete next steps with rationale**. |

### Non-goals (mode C, v1)

- Voice input on the home page (deferred)
- Multi-card simultaneous prefetch (only the hovered card; would
  cost too much api traffic)
- Persistent "remember my last hover" between sessions
- Bubbles for sub-controls inside cards (only the cards themselves)

---

## Dependencies

- `liulian-web` route `/` (NEW — currently `/` is a marketing-style
  landing page; needs to become the onboarding home)
- `liulian-web` mode preference flag in user settings (NEW)
- `liulian-agent` `prefetch` and `recommend` SSE events (NEW —
  extending the existing event schema)
- `liulian-api` `/datasets/swiss-river-1990` already exists (LANDED)
- `liulian-design-system` anchored-bubble primitive (NEW component;
  pattern is reusable across stories 3, 4, 5)
- Keyboard shortcut layer (already in `/forecast`; needs lift into
  shared layout)

## Open questions

- What's the default agent persona on the home page — same as on
  `/forecast` (BI agent), or a "guide" persona? (Author leans: same
  agent, but with a "guide" hat — first-time priors.)
- Should the home page auto-redirect to `/forecast?dataset=<recent>`
  if the user has a recent project? (Author leans: yes, with a
  3 s overlay "you have a recent project — opening it; press `esc` to
  stay here".)
- Telemetry for bubble effectiveness: how do we measure "the bubble
  saved the user from bouncing"? (A/B test mode A vs. mode C bounce
  rate.)
- What if the agent is in maintenance? (Bubble silently doesn't
  appear; mode A behaviour is intact.)

## Linked specs / ADRs

- ADR 0002 (custom agent, not LangGraph) — informs SSE event extension
- ADR 0010 (hybrid shadcn + antd stack) — informs which primitive
  library hosts the anchored bubble
- Forecast dashboard spec R8 — once the user lands on the canvas,
  story 3+ kick in
- Design-system PLATFORM_DESIGN.md — anchored-bubble component spec
  needs to be added to the system
