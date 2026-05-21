# User stories — current cycle

- **Date:** 2026-05-21
- **Owner:** Linlin Jia (jajupmochi)
- **Cycle:** post-R8 implementation / pre-pre-seed demo cycle
- **Status:** drafts for review (revision 2 — re-scoped per 2026-05-21 review)

## Why these six

R8 ships the canvas; the next leverage is *the full user journey from
landing to dashboard*. These six stories cover everything between
"open the URL" and "save a shared dashboard", in the order a real
user encounters them.

| # | Codename | Story | Why first / next |
|---|---|---|---|
| 1 | *The Doorway* (门厅) | [first-time visitor lands and chooses a path](01-home-onboarding.md) | First impression. If this fails, nothing else matters. |
| 2 | *The Annotated Drop* (落档) | [import data with AI-annotated schema](02-import-data.md) | The only way to get from "I have a CSV" to "I have a dashboard". |
| 3 | *First Light* (亮起) | [data lands, the dashboard lights up](03-auto-bi.md) | The "wow" moment — what you see when data lands. |
| 4 | *The Pencil* (改笔) | [edit a cell, watch the chart follow](04-edit-data.md) | Trust comes from being able to fix things. |
| 5 | *The Graft* (嫁接) | [add new rows, columns, or external joins](05-add-data.md) | The platform earns repeat use when it grows with the user. |
| 6 | *The Easel* (画板) | [author a custom chart by code, canvas, or sketch](06-author-chart.md) | Power-user differentiator; pitch demo gold. |

Each story has a memorable codename so the team can refer to it in
one word ("we'll demo The Doorway and First Light on Friday"). The
codename also marks the dominant interaction shape — Doorway is
routing, Drop is import, Light is reveal, Pencil is edit, Graft is
extension, Easel is creation.

## Three modes per story

Every story is presented in three modes — same goal, three radically
different interaction shapes:

| Mode | Description |
|---|---|
| **A · No-bot (classical)** | Pure controls / forms / buttons. Zero AI dialogue. Matches a mature, conservative BI tool (Power BI / Tableau). The fallback when the model is offline; the muscle-memory path for hands-on users. |
| **B · All-bot (radical)** | Pure natural-language conversation. Controls are hidden or minimised. The "AI-first" future surface. Demos well; assumes the user trusts the agent. |
| **C · Fused (innovative)** | Controls + chatbot collaborate. This is the **differentiator** — neither a BI tool nor a chatbot, but a new interaction shape. The fused mode follows six design principles ([below](#fused-mode-design-principles)). |

The three modes co-exist in the product. A power user might live in
mode A; a casual user might live in mode B; an investor demo wants
mode C. Mode C is the **headline mode** — what the product is for.

## Fused-mode design principles

Every fused-mode interaction in these stories obeys at least three of
the following six principles. Together they define what makes the
fused mode genuinely *different* from "chatbot in a sidebar plus
controls in the middle".

| # | Principle | One-liner |
|---|---|---|
| **F1** | **Spatial anchoring** | Chatbot output anchored to a specific canvas location (a cell, a chart, a data point), not in a side rail. Where you look = where the AI speaks. |
| **F2** | **Predictive prefetch** | Chatbot watches `hover`, `dwell`, `scroll` and pre-computes suggestions *before* the user finishes the gesture. Latency at action time ≈ 0. |
| **F3** | **Bidirectional spec sync** | Editing a control updates the spec text; editing the spec text updates the control. Either side is canonical; neither is "second class". |
| **F4** | **Visible reasoning** | Every AI suggestion comes with a one-sentence *why* + clickable data evidence (footnotes that scroll to the actual data point). Never "the AI just decided." |
| **F5** | **Reversible by default** | Every AI-triggered change shows a diff + a one-click undo. The user can refuse an AI action *after* it has happened, not only before. |
| **F6** | **Shared cursor** | Chatbot has a *ghost cursor* / pointer + annotation rights on the canvas. The user and the chatbot can both touch the same artefact; they're not in separate panes. |

Stories cite which principles they use in their *Innovation note*
section.

## Personas

| Persona | Role | Default mode |
|---|---|---|
| **Researcher-Linlin** | PhD-track ML researcher; pandas-fluent; comparing models | A or C |
| **PO-Linlin** | Founder; uses in pitches | C |
| **Operator-Marc** | Hydrology operator on rota | A (will tolerate C if it earns trust) |
| **Advisor-Sophia** | Reviewer; tech-literate non-coder | C |
| **Investor (pre-seed)** | One-time demo viewer on phone | C |
| **First-time visitor** | Lands from a link; unknown background | C (with auto-detection toward A or B) |

## Cross-reference

- Agile essentials + 4 templates: [`agile/essentials.md`](../agile/essentials.md)
- Fused-mode interactions reference design system: `liulian-design-system/docs/PLATFORM_DESIGN.md`
- Forecast canvas spec (R8 lock): `liulian-web/docs/specs/2026-05-21-forecast-dashboard-design.md`
- ADRs that constrain interaction design: [`strategy/adr/0002-custom-agent-not-langgraph.md`](../strategy/adr/0002-custom-agent-not-langgraph.md), [`0004-unibe-red-as-anchor.md`](../strategy/adr/0004-unibe-red-as-anchor.md), [`0010-hybrid-shadcn-antd-stack.md`](../strategy/adr/0010-hybrid-shadcn-antd-stack.md)
