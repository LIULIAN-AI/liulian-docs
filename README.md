# liulian-docs

Federation-level documentation hub for the LIULIAN platform (9 repos).
This repo holds the cross-cutting docs that don't naturally belong to
any single repo: platform blueprint, ADRs, sprint history, user
stories, agile templates, advisor briefings.

Per-repo documentation lives inside each repo's own `docs/` folder;
this hub indexes and links to them.

## Quick start

```bash
uv pip install mkdocs-material  # or pip install
mkdocs serve                    # local preview at http://localhost:8000
mkdocs build                    # static site → ./site/
```

## Structure

```
docs/
├── index.md                    # platform overview
├── federation/                 # cross-repo map + ownership
├── strategy/                   # blueprint, reuse map, sprint history, audit
├── strategy/adr/               # architecture decision records (cross-repo)
├── advisor-report-2026-05-19/  # historical briefings
├── agile/                      # agile essentials + templates
├── user-stories/               # canonical user stories
└── specs-index/                # cross-repo spec catalog (pointers)
```

## Why a separate docs repo?

`liulian-python` is the ML library. Stuffing platform-level docs into
its `docs/` made the ML-developer experience noisy. Each sibling
repo's `docs/` covers its own concerns; this hub covers everything
that crosses the seams.
