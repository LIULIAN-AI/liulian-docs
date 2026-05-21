# Specs catalog (cross-repo)

Every active feature spec across the federation. Each spec lives with
the repo whose code it governs; this page is the index.

## Active specs

| Date | Spec | Repo | Status |
|---|---|---|---|
| 2026-04-10 | [Entity identifier forecasting design](../../../liulian-python/docs/superpowers/specs/2026-04-10-entity-identifier-forecasting-design.md) | liulian-python | accepted |
| 2026-04-14 | [Entity-ID deep-audit design](../../../liulian-python/docs/superpowers/specs/2026-04-14-entity-id-deep-audit-design.md) | liulian-python | accepted |
| 2026-05-20 | [/forecast redesign (R5 lock)](../../../liulian-web/docs/specs/2026-05-20-forecast-redesign-design.md) | liulian-web | accepted |
| 2026-05-21 | [/forecast dashboard (R8 lock)](../../../liulian-web/docs/specs/2026-05-21-forecast-dashboard-design.md) | liulian-web | accepted |

## How specs are named

`docs/specs/YYYY-MM-DD-<topic>-design.md` (or `docs/superpowers/specs/`
for legacy locations in `liulian-python`).

## How to write one

Use the [design-doc template](../agile/essentials.md#52-design-doc-template).
Four sections are non-negotiable:

1. Goal
2. Information architecture / data flow
3. Acceptance criteria
4. Out-of-scope

Everything else is included on demand. Specs > 2 000 lines should be
decomposed.

## Cross-reference

- Agile essentials + templates: [`agile/essentials.md`](../agile/essentials.md)
- User stories that drive these specs: [`user-stories/`](../user-stories/index.md)
