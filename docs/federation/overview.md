# Federation overview

LIULIAN is a 9-repo federation. The split keeps each repo focused on
one concern and one technology stack. This page is the one-screen map.

## Repo responsibilities

| Repo | Stack | Owns |
|---|---|---|
| `liulian-python` | Python | the ML library (datasets, models, adapters, runtime, experiments) and its research notes |
| `liulian-api` | Python · FastAPI | REST over the ML library; canonical wire contract |
| `liulian-agent` | Python · FastAPI · SSE | LLM gateway, tool registry, conversation loop |
| `liulian-web` | Next.js · TypeScript | the BI canvas, project home, redesign rounds |
| `liulian-mobile` | Swift · Kotlin · ArkTS | three native apps, snapshot-tested visual parity |
| `liulian-design-system` | TS / JS tokens + component lib | tokens, components, brand references, UI audit checklist |
| `liulian-dev-env` | shell · docker · slurm scripts | local 3-tier stack, dev containers, cluster setup |
| `liulian-ingest` | Python · FastAPI | scheduled data crawlers, manifests |
| `liulian-ops` | Python · click · paramiko · helm | deploy CLI, runbooks, cost ledger |
| **`liulian-docs`** | mkdocs-material | this hub — platform-level docs |

## Data flow (run-time)

```
                    ┌──────────────────────┐
                    │  liulian-ingest      │ (cron crawlers)
                    └──────────┬───────────┘
                               ▼ writes datasets
            ┌──────────────────────────────────┐
            │       liulian-python             │ datasets · models · runtime
            └──────────┬───────────────────────┘
                       ▼ imported by
            ┌──────────────────────────────────┐
            │       liulian-api  (:8000)       │ REST → /forecasts /datasets /alerts ...
            └──────────┬───────────────────────┘
                       ▲ HTTP
                       │
   ┌───────────────────┼──────────────────────────┐
   │                   │                          │
   ▼                   ▼                          ▼
liulian-web    liulian-mobile           liulian-agent  (:8001)
(Next.js)      (3 native apps)          SSE chat · tools
   ▲                                         ▲
   │ SSE                                     │ HTTP
   └─────────────────────────────────────────┘
                       │
                       └─→ design-system tokens injected into web + mobile

         liulian-ops orchestrates deploys; liulian-dev-env stands up local stack
```

## Doc ownership rule

| Question | Where to put the doc |
|---|---|
| Is it about the seam between repos? | `liulian-docs` (here) |
| Is it about the ML library's internals (Python)? | `liulian-python/docs/` |
| Is it about how the API is shaped? | `liulian-api/docs/` |
| Is it about how a chart looks? | `liulian-design-system/docs/` |
| Is it about how the canvas behaves? | `liulian-web/docs/` |
| Is it about how to deploy? | `liulian-ops/docs/` |
| Is it about how to run locally? | `liulian-dev-env/docs/` |

If two repos both have a claim, the *seam* doc lives here (federation
hub) and each repo gets a thin link in its own `docs/`. Never duplicate
content across repos.

## Sprint history (current state)

- **M0 (2026-04-09 → 2026-05-12):** Entity-identifier research +
  swiss-river-1990 baseline experiments. Single repo (`liulian-python`).
- **M1 (2026-05-12 → 2026-05-19):** Multi-repo split. `liulian-api`,
  `liulian-agent`, `liulian-web` forked from neobanker. ADR 0001-0011.
- **M2 (2026-05-19 → 2026-05-21):** /forecast canvas redesign R1-R8;
  real swiss-river-1990 data wired through api → web; doc split into
  this federation hub.
- **M3 (current):** user-story-driven implementation. See
  [`user-stories/`](../user-stories/index.md).

## Cross-references

- [Docs map](docs-map.md) — every doc, where it lives
- [Platform blueprint](../strategy/PLATFORM_BLUEPRINT.md)
- [ADRs](../strategy/adr/index.md)
- [User stories (current cycle)](../user-stories/index.md)
