# Architecture Decision Records (ADRs)

These eleven ADRs capture the decisions that shape the LIULIAN
federation as a whole — they cross repo boundaries and would take a
new contributor > 30 min to derive from the code.

| # | Title | Status |
|---|---|---|
| 0001 | [Multi-repo split](0001-multi-repo-split.md) | accepted |
| 0002 | [Custom agent (not LangGraph)](0002-custom-agent-not-langgraph.md) | accepted |
| 0003 | [TimescaleDB (not TDengine)](0003-timescaledb-not-tdengine-now.md) | accepted |
| 0004 | [UniBe red as brand anchor](0004-unibe-red-as-anchor.md) | accepted |
| 0005 | [Three-entity unified tracker table](0005-tracker-three-entity-unified-table.md) | accepted |
| 0006 | [Fork & adapt from neobanker](0006-fork-and-adapt-from-neobanker.md) | accepted |
| 0007 | [Reject refine-dev-ui](0007-reject-refine-dev-ui.md) | accepted |
| 0008 | [Canvas orchestrator reuse](0008-canvas-orchestrator-reuse.md) | accepted |
| 0009 | [Spring Boot → FastAPI translation](0009-spring-boot-to-fastapi-pattern-translation.md) | accepted |
| 0010 | [Hybrid shadcn + antd UI stack](0010-hybrid-shadcn-antd-stack.md) | accepted |
| 0011 | [Dev-env separate from ops](0011-dev-env-separate-from-ops.md) | accepted |

## When to write a new ADR

Add one when:

- the decision crosses ≥ 2 repos
- reversing it later would cost > 1 day of work
- it would take a new contributor > 30 min to reverse-engineer
  the reasoning from the code

If the decision is local to one repo, put the ADR in that repo's
`docs/decisions/` instead. This hub is for federation-level
decisions only.

ADR template lives in [`../../agile/essentials.md`](../../agile/essentials.md) §5.3.
