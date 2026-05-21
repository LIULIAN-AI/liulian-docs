# LIULIAN platform docs

Welcome. This is the **federation hub** — the place to read the
platform-level documents that span more than one repo: blueprint,
architecture decisions, sprint history, advisor briefings, agile
templates, user stories.

Per-repo docs live with the repo. This hub points you at them.

## What is LIULIAN

A spatio-temporal forecasting platform: dataset → model → forecast
→ dashboard. Nine repos, one product.

| Repo | What it does |
|---|---|
| [`liulian-python`](https://github.com/jajupmochi/liulian-python) | ML core — datasets, models, adapters, runtime, experiments |
| [`liulian-api`](https://github.com/jajupmochi/liulian-api) | FastAPI — REST surface over the ML core |
| [`liulian-agent`](https://github.com/jajupmochi/liulian-agent) | LLM gateway + BI agent — SSE chat, tool registry |
| [`liulian-web`](https://github.com/jajupmochi/liulian-web) | Next.js — the BI canvas, project home, dashboards |
| [`liulian-mobile`](https://github.com/jajupmochi/liulian-mobile) | Native iOS / Android / HarmonyOS — daily briefing surface |
| [`liulian-design-system`](https://github.com/jajupmochi/liulian-design-system) | Tokens, components, reference designs, UI audit |
| [`liulian-dev-env`](https://github.com/jajupmochi/liulian-dev-env) | Local 3-tier stack, dev containers, cluster setup |
| [`liulian-ingest`](https://github.com/jajupmochi/liulian-ingest) | Scheduled data ingest pipelines |
| [`liulian-ops`](https://github.com/jajupmochi/liulian-ops) | Deploy CLI, runbooks, cost ledger |
| **`liulian-docs`** (this repo) | Federation hub — platform-level docs |

## Start here

- New to the project? → **[Federation overview](federation/overview.md)**
- Want to read the architecture decisions? → **[ADRs](strategy/adr/index.md)**
- Looking at the product roadmap? → **[Platform blueprint](strategy/PLATFORM_BLUEPRINT.md)**
- Designing or implementing a feature? → **[Agile essentials](agile/essentials.md)** then **[user stories](user-stories/index.md)**
- Hunting for a per-repo doc? → **[Docs map](federation/docs-map.md)**

## Where to find what

```
liulian-docs/                    ← you are here · federation hub
├── docs/strategy/               · blueprint · reuse map · sprint · audit
├── docs/strategy/adr/           · ADRs 0001-0011 (cross-repo)
├── docs/advisor-report-2026-05-19/  · historical advisor briefing
├── docs/agile/                  · essentials + 4 doc templates
├── docs/user-stories/           · canonical user stories
└── docs/federation/             · repo map + doc map

liulian-python/docs/             · ML core only — adapter guide, manifest spec, datasets, models, research
liulian-api/docs/                · API contracts
liulian-agent/docs/              · agent architecture, API reference, demo guides
liulian-web/docs/                · UI redesign rounds + specs
liulian-mobile/                  · per-platform native code
liulian-design-system/docs/      · platform design, references, UI audit
liulian-dev-env/docs/            · local stack, cluster tiers
liulian-ops/docs/                · deploy guide, runbooks, cost ledger
liulian-ingest/specs/            · ingest pipeline specs
```
