# Task Ledgers

Federation-wide record of **completed / in-progress / future** work,
organised along three axes:

1. **Repo** — one sub-directory per repository
   (`liulian-python/`, `liulian-agent/`, `liulian-web/`, …; created on
   first use).
2. **Time** — one file per month inside each repo directory
   (`YYYY-MM.md`). The newest file is the *live* ledger; older files are
   frozen history.
3. **Topic** — entries inside a monthly file carry a topic tag, e.g.
   `[id-exp]` (entity-identifier experiments), `[tsl-align]`
   (TSL alignment study), `[optim]` (HPO / search spaces), `[data]`
   (data layer), `[infra]` (CI / branches / cluster / cost), `[results]`
   (reporting / artifacts). Add tags as needed — keep them short and
   reuse existing ones.

## Conventions

- Every entry names its **artifacts** (commits, files, run-tags, job ids)
  so it can be audited later. Agent session task IDs (`#N`) are mirrored
  for traceability.
- **Update the ledger in the same commit/PR** that finishes a work item or
  decides a new one (rule mirrored in each code repo's `CLAUDE.md`).
- Items that move across months: carry the open items forward into the new
  month's file under *In progress* / *Future*; do not edit frozen months
  except to add a "→ moved to YYYY-MM" note.
- Code repos keep a pointer stub (e.g. `liulian-python/docs/tasks.md`)
  linking here, so the ledger is discoverable from inside each repo.

## Ledgers

- **liulian-python** — [2026-06](liulian-python/2026-06.md) ·
  [2026-05](liulian-python/2026-05.md)
