# Advisor progress briefing — 2026-05-19

One-page slide reporting LIULIAN platform progress at the end of the Week-1 sprint.

## Files

| File | What it is |
|---|---|
| `slide.png` | **The deliverable** — 1280×720 rendered slide. Drop into email / PowerPoint / PDF / screen-share. |
| `slide.html` | Editable source. Open in any browser; edit text, re-render. |
| `forecast-canvas.png` | The embedded web-frontend screenshot (current GitHub Pages showcase). |

## How to view / present

- **Quick**: open `slide.png` — it is self-contained, no toolchain.
- **Edit then re-render**: open `slide.html` in Chrome → it is a fixed 1280×720 `.slide`. Edit the text in the HTML, refresh, then capture: Chrome DevTools → ⋮ → *Capture node screenshot* on the `.slide` element; or print to PDF (`Ctrl+P` → Save as PDF, set margins to none).
- **In a meeting**: open `slide.png` full-screen, or paste into one PowerPoint/Keynote slide.

## Why a single self-contained HTML slide (the "elegant" approach)

- **On-brand for free** — it is styled with LIULIAN's own design language (UniBe red, Fraunces / Switzer / JetBrains Mono, warm-paper canvas). The slide itself signals the design system works.
- **Version-controllable** — lives in the repo, diffs cleanly, no binary `.pptx` blob.
- **Render-anywhere** — HTML → PNG or PDF with the browser alone; no PowerPoint needed.
- **One artifact** — the advisor receives a single image; nothing to install.

If this grows into a multi-slide deck, the natural next step is **Marp** (Markdown → slides,
`npx @marp-team/marp-cli deck.md -o deck.pdf`) — same version-control benefit, scales past one page.

## Honesty note

The screenshot shows a **polished but static** web frontend. Every value (including the
on-screen "LIVE" badge) is hardcoded HTML — verified, zero API calls. The slide states this
plainly in the amber callout. Do not let the advisor mistake the showcase for a live system.
